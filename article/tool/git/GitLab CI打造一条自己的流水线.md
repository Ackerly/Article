# GitLab CI打造一条自己的流水线
**CI/CD是什么**  
CI：  
即持续集成，它是指频繁地（一天多次）将代码集成到主干，目的就为了让产品保证质量的同时快速迭代；通常它需要通过自动化测试，从而保证集成的代码的稳定性；  
CD：  
即持续交付/部署，可以看作持续集成的下一步，它指的是频繁地将软件的新版本，交付给质量团队or用户测试。如果测试通过，代码就可以部署到生产环境中   
**为什么需要CI/CD**  
过去，一个团队的开发人员可能会孤立地工作很长一段时间，只有在他们的工作完成后，才会将他们的更改合并到主分支中。这使得合并代码更改变得困难而耗时，而且还会导致错误积累很长时间而得不到纠正。这些因素导致更加难以迅速向客户交付更新。  
有了 CI/CD，可以获得以下收益：  
1. 解放了重复性劳动。自动化部署工作可以解放集成、测试、部署等重复性劳动，而机器集成的频率明显比手工高很多
2. 更快地修复问题。持续集成更早的获取变更，更早的进入测试，更早的发现问题，解决问题的成本显著下降
3. 更快的交付成果。更早发现错误减少解决错误所需的工作量。集成服务器在构建环节发现错误可以及时通知开发人员修复。集成服务器在部署环节发现错误可以回退到上一版本，服务器始终有一个可用的版本
4. 减少手工的错误。在重复性动作上，人容易犯错，而机器犯错的几率几乎为零
5. 减少了等待时间。缩短了从开发、集成、测试、部署各个环节的时间，从而也就缩短了中间可以出现的等待时机。持续集成，意味着开发、集成、测试、部署也得以持续
6. 更高的产品质量。集成服务器往往提供代码质量检测等功能，对不规范或有错误的地方会进行标志，也可以设置邮件和短信等进行警告

## GitLab CI/CD
GitLab-CI 是GitLab提供的CI工具。它可以通过指定通过如push/merge代码、打tag等行为触发CI流程；同时也可以指定不同场景要触发的不同的构建脚本（脚本可以看作是流水线中的一个操作步骤or单个任务）  
具体的使用方式是在项目根目录中配置一个 .gitlab-ci.yml 文件来启动其功能；  
**配置文件介绍**  
gitlab-ci.yml 用的是 YAML语法[1] ，可以把它理解类似 json 的格式，只不过语法方面有一些不同。比如：
```
tabitha:
    name: Tabitha Bitumen
    job: Developer
     skills:
      - lisp
      - fortran
      - erlang
```
对应到json：  
```
{
     tabitha : {
         name :  Tabitha Bitumen ,
         job :  Developer ,
         skills ： [ lisp ,  fortran ,  erlang ],
    }
}
```
缩进对应的是json对象中的key，- 对应的是数组中的一项  
**Job**  
Job可以理解为CI流程中的单个任务。  
job是一个顶级元素（相当于yml配置的一个根元素），它可以起任意的名称、并且不限数量，但必须至少包含 script 子句，用于指定当前任务要执行的脚本，如：  
```
job1:
  script:  execute-script-for-job1 
job2:
  stage: build
  script:
    - scripts/build.sh
  only:
    - master
# job n...
```
**Stages**  
Stages 用来定义一次CI有哪几个阶段，如下  
```
stages:
  - build
  - test
  - deploy
```
同时每个stage又可以与若干个job关联，即一个阶段可以并行执行多个job；如下，在每个job中使用stage关键字关联到对应stage即可：  
```
stages:
  - build
  - test
  - deploy
  
build_job:
  stage: build
  script:
    - scripts/build.sh

test_job:
  stage: test
  script:
    - scripts/test.sh

deploy_job:
  stage: deploy
  script:
    - scripts/deploy.sh
```
**Pipeline**  
Pipeline是持续集成、交付和部署的顶级组件，它可以理解为是流水线的一次完整的任务流程；  
Pipeline 可以包含若干Stage，而每个Stage又可以指定执行若干job，这样就可以把整个构建的流程串起来了。
如果Pipeline中的一个任务成功，将进入其下一个Stage的Job；反之如果中途失败，则默认会中断流水线的执行。  
## 配置
- stage 可以定义 job 在哪个阶段运行  
- script用于指定运行器要执行的脚本命令，可以指定多条
- before_script & after_script用于定义应在job 在执行脚本之前/后时要执行的内容
- allow_failure用于配置当前 job 失败时 pipeline 是否应继续运行
- cache指定缓存的文件列表，用户在不同的 job 之间共享
  - cache:key：可以给每个缓存一个唯一的标识键，如果未设置，则默认键为default；
  - cache:paths：指定要缓存的文件或目录
- only / excepto：nly 用于定义何时执行job，反之 except 用于定义何时不执行job；
  - ref：匹配 分支名称 或 分支名匹配的的正
  - variables：变量表达式
  - changes：对应路径的文件是否修改
- retry设置在 job 执行失败时候重试次数
- variables用于定义执行过程中的一些变量
- when用于配置 job 运行的条件
  - on_success（默认）：仅在之前stage的所有job都成功或配置了allow_failure: true；
  - manual：仅在手动触发时运行 job；
  - always：无论之前stage的job状态如何，都运行；
  - on_failure：仅当至少一个之前stage的job失败时才运行；
  - delayed：延迟执行job；
  - never: 永不执行 job；
- tags选择特定tag的GitLab-runner来执行

**GitLab-runner 安装&注册**  
配置好yml 文件之后，还需要配置GitLab-runner，用于执行对应的脚本  
> 注册时需要的 URL & token可以在 GitLab -> Settings -> CI/CD -> Runners 中获取

## 打造一条流水线
**预览Pipeline**  
配置好 .gitlab-ci.yml 文件、写好对应的脚本，同时配置好 GitLab-runner 后，就可以开启并体验 CI 流水线了。当提交代码后（当然也可以按上述的only关键字设置为打tag、提mr时触发），就可以触发GitLab CI的Pipeline，并执行对应的stages及其jobs啦  
**配置一条流水线**  
规划实现如下：  
- 代码编译：提供一个build.sh脚本，用于编译代码；
- 自动化测试：scripts 关键字执行测试的指令，从而运行事先编写好的自动化测试脚本；
- 人工卡点：利用上述提到的when:manual人工触发，配合allow_failure: false，即可达到卡点效果；
- 项目部署：也是利用一个脚本，将我们之前的构建产物发送到目标机器、目录下进行部署；同时使only:master指定只有在提交到master分支才执行该步骤的 job。

触发pipeline后，经过了scm编译、test自动化测试的步骤后，到了Manual-point卡点，此时流水线已经锁定执行，需要人工手动点击确认才可以继续执行；  
**定时任务**  
定时地执行Pipeline，而不是通过事件触发，可以在GitLab -> Setting -> Schedule 进行设置  
> 间隔的设置采用的是 cron 语法[7]，它是Unix和类Unix系统中设置定时任务的语法；它使用5个占位符分别代表 分钟、小时、月份的日期、月份、周几



原文:
[GitLab CI 打造一条自己的流水线](https://mp.weixin.qq.com/s/0tZ3NwIlm3WDioBLqIlKYw)