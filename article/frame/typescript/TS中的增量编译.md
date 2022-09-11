# TS 中的增量编译
## TypeScript 中的增量编译
**TypeScript 中的增量编译**  
TypeScript 编译启动时，会根据是否配置 increamental 参数走入不同的编译环节。配置 increamental 表示开启增量编译，将执行增量编译行为  
``` 
if (ts.isWatchSet(configParseResult.options)) {
  if (reportWatchModeWithoutSysSupport(sys, reportDiagnostic))
    return;
  return createWatchOfConfigFile(sys, cb, reportDiagnostic, configParseResult, commandLineOptions, commandLine.watchOptions, extendedConfigCache);
}
// 配置 increamental 参数，开启增量编译
else if (ts.isIncrementalCompilation(configParseResult.options)) {
  performIncrementalCompilation(sys, cb, reportDiagnostic, configParseResult);
}
// 常规编译
else {
  performCompilation(sys, cb, reportDiagnostic, configParseResult);
}
```
**ts 中的增量编译**  
基于 buildInfo 文件生成 oldProgram  
在 ts 增量编译过程中，会读取配置项 tsBuildInfoFile 指向的 buildInfo 文件，并解析 buildInfo 文件，基于 buildInfo 文件创建旧编译程序 oldProgram  
``` 
function createIncrementalProgram(_a) {
  var rootNames = _a.rootNames, options = _a.options, configFileParsingDiagnostics = _a.configFileParsingDiagnostics, projectReferences = _a.projectReferences, host = _a.host, createProgram = _a.createProgram;
  host = host || createIncrementalCompilerHost(options);
  createProgram = createProgram || ts.createEmitAndSemanticDiagnosticsBuilderProgram;
  
  // 创建 旧的编译程序 oldProgram
  var oldProgram = readBuilderProgram(options, host);
  return createProgram(rootNames, options, host, oldProgram, configFileParsingDiagnostics, projectReferences);
}

// 基于 tsBuildInfoFile 指向的文件创建 oldProgram
function readBuilderProgram(compilerOptions, host) {
  var buildInfoPath = ts.getTsBuildInfoEmitOutputFilePath(compilerOptions);
  if (!buildInfoPath)
    return undefined;
  var buildInfo;
  if (host.getBuildInfo) {
    buildInfo = host.getBuildInfo(buildInfoPath, compilerOptions.configFilePath);
    if (!buildInfo)
      return undefined;
  }
  else {
    var content = host.readFile(buildInfoPath);
    if (!content)
      return undefined;
    buildInfo = ts.getBuildInfo(content);
  }
  if (buildInfo.version !== ts.version)
    return undefined;
  if (!buildInfo.program)
    return undefined;
  return ts.createBuilderProgramUsingProgramBuildInfo(buildInfo.program, buildInfoPath, host);
}
```
对比新老编译程序找到变更文件  
进而会基于 oldProgram 和 当次编译对应的配置信息创建当次编译对应的编译程序 (这里记为：currentProgram），此时该程序（currentProgram）将通过 oldProgram 间接持有往次编译的所有信息。  
``` 
function createIncrementalProgram(_a) {
  var rootNames = _a.rootNames, options = _a.options, configFileParsingDiagnostics = _a.configFileParsingDiagnostics, projectReferences = _a.projectReferences, host = _a.host, createProgram = _a.createProgram;
  host = host || createIncrementalCompilerHost(options);
  createProgram = createProgram || ts.createEmitAndSemanticDiagnosticsBuilderProgram;
  var oldProgram = readBuilderProgram(options, host);
  
  // 创建当次构建对应的编译程序
  return createProgram(rootNames, options, host, oldProgram, configFileParsingDiagnostics, projectReferences);
}
```
在 currentProgram 执行本次编译时，会通过对比往次编译和当次编译中的信息差异来完成增量编译。增量编译的本质在于，对往次已编译文件跳过编译、编译新增/修改内容。在 TypeScript 中会根据如下规则判断当次构建是否存在变更内容：  
- 是否存在往次构建信息
- 往次构建信息中是否包含该文件
- 该文件是否存在内容变化（在每次构建时会基于文件内容计算出 hash 值，并将该值作为文件 version ）
- 文件 format 是否存在变更
- 引用关系是否发生变化

对应代码如下：  
``` 
state.fileInfos.forEach(function (info, sourceFilePath) {
  var oldInfo;
  var newReferences;
  if (!useOldState || // 规则① 往次构建是否存在
  
    // 规则② 往次构建信息中是否包含该文件
    !(oldInfo = oldState.fileInfos.get(sourceFilePath)) || 
    
      // 规则③ 该文件是否存在内容变化
      oldInfo.version !== info.version ||
      
      // 规则④ 文件 format 是否存在变更
      oldInfo.impliedFormat !== info.impliedFormat ||
      
      // 规则⑤ 引用关系是否发生变化
      !hasSameKeys(newReferences = referencedMap && referencedMap.getValues(sourceFilePath), oldReferencedMap && oldReferencedMap.getValues(sourceFilePath)) ||
      newReferences && ts.forEachKey(newReferences, function (path) { return !state.fileInfos.has(path) && oldState.fileInfos.has(path); })) {
        state.changedFilesSet.add(sourceFilePath);
    }
    
    // other codes
 }
```
会将变更内容放入 changedFileSet 在下游产物生成环节被消费  
根据变更文件完成产物更新  
产物生成阶段会遍历 changedFileSet，对于每一个变更文件，会分别完成文件产物的更新、以及上下游引用链路上的类型文件更新  
``` 
function getNextAffectedFile(state, cancellationToken, computeHash, getCanonicalFileName, host) {
  var _a, _b;
  while (true) {
    var affectedFiles = state.affectedFiles;
    if (affectedFiles) {
      var seenAffectedFiles = state.seenAffectedFiles;
      var affectedFilesIndex = state.affectedFilesIndex;
      while (affectedFilesIndex < affectedFiles.length) {
        
          // other codes
          
          // 更新上下游引用链路上的类型
          handleDtsMayChangeOfAffectedFile(state, affectedFile, cancellationToken, computeHash, getCanonicalFileName, host);
          return affectedFile;
        }
        affectedFilesIndex++;
      }
      // other codes
    }
    // other codes
  }
}
```
**buildInfo 文件生成**  
编译完成后，会从当次编译程序中拿到所有的编译信息（buildInfo），将其写入 tsBuildInfoFile 指向的文件位置（如未定义，会将 buildInfo 文件存放于产物目录中）  
``` 
function emitBuildInfo(bundle, buildInfoPath) {

  // other codes
  
  var buildInfo = { bundle: bundle, program: program, version: version };
  ts.writeFile(host, emitterDiagnostics, buildInfoPath, getBuildInfoText(buildInfo), false, undefined, { buildInfo: buildInfo });
}
```

**如何配置 tsBuildInfoFile**  
观察 TypeScript 增量编译流程可以发现，每启动 tsc 执行源码编译都会消费上次的 buildInfo 信息，并基于此完成增量编译，编译结束后吐出该次的构建信息，供下次增量编译使用。tsBuildInfoFile 配置项可以用来指定 buildInfo 信息的保存位置  
- 通常情况下 buildInfo 信息只会被 TypeScript 消费，业务无需关注 buildInfo 文件存放位置，此时该配置项可以忽略，不用配置。
- 对于特殊场景，对 buildInfo 保存位置有强位置诉求，需要注意不同模块（尤其是在 monorepo 仓库中）的 buildInfo 的位置应该独立，否则可能会出现 buildInfo 的误消费、覆盖

参考:  
[TS 中的增量编译](https://mp.weixin.qq.com/s/830njEJhNGSSvYO7zHXg8A)
