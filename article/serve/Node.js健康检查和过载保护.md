# Node.js健康检查和过载保护
设想一下，你有30个Node服务器与 "Nginx "服务器平衡。服务器的负载被平均分配，所以如果你有9000个用户，每个服务器有300个客户。把负载平均分配给每台服务器并不意味着你可以避免过载，因为你的工作对每个用户都可能不同。例如，对于user_1，你可能读取3个文件，但对于user_2，你可能需要读取10个（3.34倍）。根据用户的请求过程会有更多的复杂性，这是现实中需要去思考和解决的问题。在类似于这种的情况之下，开发者不得不平衡对同每一台服务器的请求，因为一旦这台服务器是Server Overload(过载)的状态，你的服务可能会挂掉。  
> 健康检查 每隔n秒向服务器发送请求，以了解服务器是否能够处理更多的请求，如果正常，服务器将其标记为OK，并继续从Balancer(平衡器)接收更多的请求，否则服务器将其标记为Error，Balancer将不会向该服务器发送任何请求，直到Balancer再次发送健康检查请求并标记为OK。

这个过程被称为健康检查。  
作为检测应用是否正常运行的机制，服务器健康检查通常在以下情况下使用：
1. 高可用性
2. 负载均衡：如果正在使用多个实例运行同一项目或者应用，则可以将负载平衡器配置为定期查询每个实例并记录其响应时间和状态。
3. 自动化部署：自动化部署过程中包括激活新版本后立即执行的测试步骤，这些测试步骤大多数都涉及到对系统组件进行基本的功能验证或心跳监控。
4. 监控和警报：当服务不再可访问或出现其他问题时，服务器健康检查可以触发警报并通知管理员或开发人员进行干预操作。

## 实际应用
以下是基于Express和Node.js实现健康检查的简单Demo：
1. 安装healthcheck模块：使用npm或yarn等包管理工具安装healthcheck模块。
2. 在Express应用中添加路由：创建一个新路由来处理/healthcheck请求，并返回200 OK响应

``` 
const healthCheck = require('express-healthcheck');

app.use('/health', healthCheck({
  healthy: function () {
    return { everything: 'is ok' };
  }
}));
```
3. 向监控系统报告：如果正在使用譬如Prometheus[2]之类的监控系统，则可以将其配置为定期查询/health接口并记录结果。此外，还可以将结果发送到其他系统进行分析和通知。
4. 添加自定义检查项：除了默认提供的基本系统信息之外，还可以编写自己的检查函数来测试特定组件或功能是否可用。这些函数必须返回承诺对象并被注册到健康检查器中：
``` 
 const checkDbConnection = async () => {
  try {
    const client = await pool.connect();
    await client.query('SELECT NOW()');
    return true;
  } catch (e) {
    console.error(e);
    return false;
  }
};

const customChecks = [checkDbConnection];

app.use('/health', healthCheck({
  healthy: function () {
    return { everything: 'is ok' };
  },
  customChecks
}));
```

## 结合着nginx和node.js做健康检查
考虑到实际项目中广泛使用Nginx的现状，所以笔者推荐使用nginx_upstream_check_module（不随Nginx源码分发)和Node.js来实现服务器健康检查：
1. 首先，在Nginx配置文件中启用upstream_check模块。例如，可以在http块中添加如下代码：
``` 
http {
  upstream myapp {
    server localhost:3000;

    check interval=3000 rise=2 fall=3 timeout=2000 type=http;
    check_http_send "HEAD /health HTTP/1.0\r\n\r\n";
    check_http_expect_alive http_2xx http_3xx;
  }
}
```
这将创建一个名为myapp的上游组，并且每隔3秒钟会向localhost：3000发送一个HTTP HEAD请求以检查其状态。  
2. 在Node.js应用中，按照前面提到的方法设置/ healthcheck路由（或任何其他自定义路由）。
``` 
const healthCheck = require('express-healthcheck');

app.use('/health', healthCheck({
  healthy: function () {
    return { everything: 'is ok' };
  }
}));
```
3. 确保应用正在侦听端口3000并运行。如果需要更改端口，请相应地更新Nginx配置文件。
4. 最后，在浏览器中访问http://localhost/_nginx_health_check/myapp（根据需要更改主机和路径），以确保Nginx正确显示了myapp上游组件的状态，并且已经能够成功连接到Node.js服务端点。

通过以上几个步骤，就可以将nginx_upstream_check_module和Node.js结合起来创建一个强大的健康检查机制。  
尽管ngx_http_healthcheck_module拥有着高效、零活且易于配置的优点，但是由于其仅适用于 HTTP 服务，不支持 TCP 或 UDP 健康检查：如果要检测连接到非 Web 服务器（如数据库）是否正常工作，则需要另外使用第三方组件或软件包。  

## HAProxy
HAProxy是一个开源的负载均衡工具，可以将来自客户端的请求分发到多个后端服务器，并确保这些请求能够平衡地分配给所有可用的服务器。HAProxy支持多种协议，例如HTTP、TCP和SMTP等。  
HAProxy最初是为高可用性而设计的，在其最简单的形式下，它只需配置两个或更多台服务器即可实现基本负载均衡。但随着时间推移和版本更新，它已经成为一款功能强大且广泛应用于各种场景中（包括大型企业网络）的软件。  
除了提供负载均衡之外，HAProxy还具有其他有用功能。例如：  
1. 健康检查：HAProxy可以定期检查后端服务器是否健康，并根据需要启停不健康或超时连接
2. SSL终止：HAProxy可以在前置代理层上执行SSL加密和解密操作
3. 请求重写：HAProxy允许使用ACL规则对传入请求进行修改或过滤
4. 日志记录：HAProxy生成详细日志以便于跟踪问题并进行性能优化

HAProxy可以与Nginx和NodeJS结合使用，以提供一个高性能、可扩展和可靠的网络服务基础设施。以下是将这些技术结合起来使用的一种可能方式：  
1. 使用Nginx作为一个反向代理：在这种配置中，Nginx作为前端服务器，接收所有来自客户端的传入请求，并将其转发到适当的后端服务器进行处理。开发者可以配置Nginx分别在80或443端口监听HTTP或HTTPS流量。
2. 将HAProxy配置为一个负载平衡器：HAProxy应该被配置为在运行NodeJS项目的多个后端服务器之间分配传入的流量。负载平衡算法可以根据使用者的具体需求来定制。
3. 使用PM2部署NodeJS项目：PM2是一个进程管理器，可以轻松地在多个服务器实例上管理和扩展NodeJS项目。
4. 在负载均衡器层面上设置SSL终端：如果开发者的应用需要SSL加密，可以通过在负载平衡器上安装SSL证书，在HAProxy层面上设置SSL终止。

下面是使用这些技术时需要考虑到的一些问题：
- 如果应用需要在客户端请求之间保持会话数据，可能想配置粘性会话。
- 使用ping检查或自定义脚本等工具监控服务器的健康状态。
- 通过在Nginx和HAProxy中配置缓存规则来优化性能。

总的来说，将HAProxy与Nginx和nodeJS结合起来，为建立快速、可靠的Web服务提供了一个强大的工具集，可以处理大量的流量，同时在系统故障或中断的情况下提供强大的故障转移机制。  

## 过载保护
为了检查服务器是否过载并防止过载，开发者通常需要检查一些指标。这也取决于项目代码逻辑是否完全准确无误，但在这里可以看到必须检查的通用指标。  

- 事件循环延迟
- 已用堆内存
- ...

可以借助于overload-protection这个包，可以帮助我们快速定位到这些指标。另一个值得推荐的包是event-loop-lag，接受一个代表刷新事件循环滞后测量频率的毫秒数，并返回一个可以调用的函数，以接收最新的滞后测量值，单位为毫秒。  
使用overload-protection包，我们可以指定限制，超过这个限服务器将无法处理更多的请求。当配置的最大请求限制通过时，软件包会自动发送503 SERVICE UNAVAILABLE。该包与http、express、restify和koa包一起工作。  
但是，如果负载平衡器(Banlancer)可以发送socket进行Health Check，并且你想用socket来做，那么就需要使用别的包来完成此事了。


原文:  
[Node.js健康检查和过载保护](https://mp.weixin.qq.com/s/ig3V0iwaVea9V4VQYDm3CQ)
