## 创建 Web 应用项目

### 创建Web应用
```
dotnet new webapp -o aspnetcoreapp
```
### 信任开发证书
```
dotnet dev-certs https --trust
```
## 生命周期
ASP.NET Core 支持依赖关系注入（DI）软件设计模式，该模式允许我们注册服务、控制如何实例化这些服务并将其注入到不同的组件中。
一些服务可以在短周期内实例化，并且仅在特定的组件和请求中可用；一些实例仅被实例化一次，并在整个应用程序生命周期中可用。
这就是 ASP.NET Core 中可用的服务生命周期，共三种:Transient、Scoped、Singleton
![ASP.NET Core Service Lifetimes Infographic](https://www.ezzylearning.net/wp-content/uploads/ASP.NET-Core-Service-Lifetime-Infographic.png)
- Singleton: 只创建一个实例,并在所有请求之间共享. ***需要注意并发和线程问题***
- Scoped: 为每一个请求创建一个服务实例, 并在整个请求中重用.请求被当作一个范围
- Transient: 即使是同一个请求, 也会每次创建一个新的服务实例,是多线程安全问题中最安全最常见的选择
## 执行过程
### Program.cs
*(core 7创建的会默认使用minimal只有中间件注册部分)*
```
public class Program  
{  
    public static void Main(string[] args)  
    {  
        CreateHostBuilder(args).Build().Run();  
    }  
  
    public static IHostBuilder CreateHostBuilder(string[] args) =>  
		 Host.CreateDefaultBuilder(args).ConfigureWebHostDefaults(webBuilder =>  
        {  
            webBuilder.UseStartup<Startup>();  
        });  
}
```
`Program.Main` 通过 CreateHostBuilder 方法取得 Host 后，再运行Host；Host 就是 ASP.NET Core 的网站实例。
- Host.CreateDefaultBuilder  
    通过此方法建立 Builder。Builder 是用來生成 Host 的对象。  
    可以在 Host 生成之前设置一些前置动作，当 Host 建立完成时，就可以使用已准备好的物件等。
- UseStartup
    设置该 Builder 生成的 Host 启动后，要执行的类。
- Build  
    当前置准备都设置完成后，就可以调用 Builder 方法实例化 Host，并得到该实例。
- Run  
    启动 Host。
### Startup.cs
当网站启动后，WebHost会实例化 **UseStartup** 设置的_Startup_类，并且调用以下两个方法：
```
public class Startup  
{  
    private IConfiguration Configuration { get; }  
  
    public Startup(IConfiguration configuration)  
    {  
        Configuration = configuration;  
    }  
  
    public void ConfigureServices(IServiceCollection services)  
    {  
        services.AddControllers();  
        services.AddEndpointsApiExplorer();
		services.AddSwaggerGen();
	}  
  
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)  
    {  
          
        if (env.IsDevelopment())  
        {  
            app.UseSwagger();  
            app.UseSwaggerUI();  
        }  
  
        app.UseHttpsRedirection();  
  
        app.UseAuthorization();  
  
        app.UseEndpoints(endpoints =>  
        {  
            endpoints.MapControllers();  
        });  
    }  
}
```
- _**ConfigureServices**_  
    ConfigureServices 是用来将服务注册到 DI 容器用的。这个方法可不实现，并不是必要的方法。  
    
- _**Configure**_  
    这个是必要的方法，一定要实现。但 __`Configure`__ 方法的参数并不固定，参数的实例都是从 Host 注入进来，可依需求增减需要的参数。
    - IApplicationBuilder 是最重要的参数也是必要的参数，Request 进出的 Pipeline 都是通过 ApplicationBuilder 来设置。