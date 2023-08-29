## 启动过程
ASP.NET Core应用程序拥有一个内置的**Self-Hosted（自托管）**的**Web Server（Web服务器）**，用来处理外部请求。

在ASP.NET Core应用中通过配置并启动一个Host来完成应用程序的启动和其生命周期的管理（如下图所示）。而Host的主要的职责就是Web Server的配置和**Pilpeline（请求处理管道）**的构建。

![ASP.NET Core总体启动流程](https://upload-images.jianshu.io/upload_images/2799767-5ecdfc52c288b66a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这张图描述了一个总体的启动流程，从上图中知道ASP.NET Core应用程序的启动主要包含三个步骤：
1. CreateDefaultBuilder()：创建IWebHostBuilder
2. Build()：IWebHostBuilder负责创建IWebHost
3. Run()：启动IWebHost

 ![ASP.NET Core启动流程调用堆栈](https://upload-images.jianshu.io/upload_images/2799767-956a061437103079.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 宿主构造器：IWebHostBuilder
在启动`IWebHost`宿主之前，需要完成对`IWebHost`的创建和配置。而这一项工作需要借助`IWebHostBuilder`对象来完成的，ASP.NET Core中提供了默认实现`WebHostBuilder`。而`WebHostBuilder`是由WebHost的同名工具类（Microsoft.AspNetCore命名空间下）中的`CreateDefaultBuilder`方法创建的。
从上图中可以看出`CreateDefaultBuilder()`方法主要干了六件大事：

1. UseKestrel：使用Kestrel作为Web server。
2. UseContentRoot：指定Web host使用的content root（内容根目录），比如Views。默认为当前应用程序根目录。
3. ConfigureAppConfiguration：设置当前应用程序配置。主要是读取 appsettings.json 配置文件、开发环境中配置的UserSecrets、添加环境变量和命令行参数 。
4. ConfigureLogging：读取配置文件中的Logging节点，配置日志系统。
5. UseIISIntegration：使用IISIntegration 中间件。
6. UseDefaultServiceProvider：设置默认的依赖注入容器。

创建完毕`WebHostBuilder`后，通过调用`UseStartup()`来指定启动类，来为后续服务的注册及中间件的注册提供入口。

## 宿主：IWebHost
在ASP.Net Core中定义了`IWebHost`用来表示Web应用的宿主，并提供了一个默认实现`WebHost`。宿主的创建是通过调用`IWebHostBuilder`的`Build()`方法来完成的。那该方法主要做了哪些事情呢，看上图中的黄色边框部分.
其核心主要在于WebHost的创建，又可以划分为三个部分：

1. 构建依赖注入容器，初始通用服务的注册：BuildCommonService();
2. 实例化WebHost：var host = new WebHost(...);
3. 初始化WebHost，也就是构建由中间件组成的请求处理管道：host.Initialize();

### 注册初始通用服务

`BuildBuildCommonService`方法主要做了两件事：

1. 查找`HostingStartupAttribute`特性以应用其他程序集中的启动配置
2. 注册通用服务
3. 若配置了启动程序集，则发现并以`IStartup`类型注入到IOC容器中

### 创建IWebHost
build源码
```
public IWebHost Build()
{
    //省略部分代码
 
    var host = new WebHost(
        applicationServices,
        hostingServiceProvider,
        _options,
        _config,
        hostingStartupErrors);
    }
    
    host.Initialize();
 
    return host;
}
```


### 构建请求处理管道

请求管道的构建，主要是中间件之间的衔接处理。

> 而请求处理管道的构建，又包含三个主要部分：
> 
> 1. 注册Startup中绑定的服务；
> 2. 配置IServer；
> 3. 构建管道

请求管道的构建主要是借助于`IApplicationBuilder`，相关类图如下：  
![请求管道的构建](https://upload-images.jianshu.io/upload_images/2799767-b4bced8e49659acd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 启动WebHost

> WebHost的启动主要分为两步：
> 
> 1. 再次确认请求管道正确创建
> 2. 启动Server以监听请求
> 3. 启动 HostedService

### 确认请求管道的创建

从图中可以看出，第一步调用`Initialize()`方法主要是取保请求管道的正确创建。其内部主要是对`BuildApplication()`方法的调用，与上面所讲WebHost的构建环节具有相同的调用堆栈。而最终返回的正是由中间件衔接而成的`RequestDelegate`类型代表的请求管道。

### 启动Server

先来看下类图：

![IServer类图](https://upload-images.jianshu.io/upload_images/2799767-311fe9a2b753718d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从类图中可以看出`IServer`接口主要定义了一个只读的特性集合属性、一个启动和停止的方法声明。在创建宿主构造器`IWebHostBuilder`时通过调用`UseKestrel()`方法指定了使用KestrelServer作为默认的IServer实现。其方法申明中接收了一个`IHttpApplication<TContext> application`的参数，从命名来看，它代表一个Http应用程序，看下具体的接口定义：

![IHttpApplication类图](https://upload-images.jianshu.io/upload_images/2799767-ec6da9b7c1a03061.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其主要定义了三个方法，第一个方法用来创建请求上下文；第二个方法用来处理请求；第三个方法用来释放上下文。而至于请求上下文，是用来携带请求和返回响应的核心参数，其贯穿与整个请求处理管道之中。ASP.NET Core中提供了默认的实现`HostingApplication`，其构造函数接收一个`RequestDelegate _application`（也就是链接中间件形成的处理管道）用来处理请求。

```csharp
var httpContextFactory = _applicationServices.GetRequiredService<IHttpContextFactory>();
var hostingApp = new HostingApplication(_application, _logger, diagnosticSource, httpContextFactory);
```

### 启动IHostedService

`IHostedService`接口用来定义后台任务，通过实现该接口并注册到Ioc容器中，它会随着ASP.NET Core 程序启动而启动，终止而终止。