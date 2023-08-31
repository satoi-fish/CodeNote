## Autofac
Autofac是.NET领域最为流行的IOC框架之一，传说是速度最快的一个。
> **优点：**
> - 它是C#语言联系很紧密，也就是说C#里的很多编程方式都可以为Autofac使用。
> - 较低的学习曲线，学习它非常的简单，只要你理解了IoC和DI的概念以及在何时需要使用它们。
> - XML.Json配置支持。
> - 自动装配。
> - 与Asp.Net MVC 集成。
> - 微软的Orchad开源程序使用的就是Autofac，从该源码可以看出它的方便和强大。

## IoC框架
先说说常见的Ioc框架吧。  
**Autofac：** 目前net用的比较多，好多大佬的项目比较优先选择的框架。  
**Ninject：** 已经很少用了，还时在很早的文章中见过。  
**Unity：** 比较常见的了，好多地方有用到的，  
**Core：** Core中自带的，业务逻辑不太复杂的情况下，还是比较方便的。

## AutoFac的使用
改用 `Autofac` 来实现依赖注入
```
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        // 这里加入一行
        .UseServiceProviderFactory(new AutofacServiceProviderFactory())
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });

```
添加我们自定义的 `Autofac` 注册类
```
public class AutofacModuleRegister : Autofac.Module
{
    //重写Autofac管道Load方法，在这里注册注入
    protected override void Load(ContainerBuilder builder)
    {
        builder.RegisterType<XxxxService>().As<IXxxxService>();
　　 }
 }
```
Startup类中添加方法
```
public void ConfigureContainer(ContainerBuilder builder)
{
    builder.RegisterModule(new AutofacModuleRegister());
}
```
Controller里面照常如.Net Core 自带注入框架一样写就行.

## 不同的特性
### 批量注入
如果一个项目之前有了用户 `UserService`，需求更新，加入了商品`ProductService`，有了商品那又怎么能少得了订单`OrderService`，那后面是不是还得有售后、物流、仓库、营销......

如果是`.Net Core` 自带的注入框架，那就只能不停的：
```
services.AddScoped<IProductService, ProductService>();
services.AddScoped<IOrderService, OrderService>();
......
```
这时候，`Autofac` 的好处就体现出来了：**批量注入。**
我们先回到上面的：`AutofacModuleRegister` 类，加入下面这段代码：
```
// 服务项目程序集  
Assembly service = Assembly.Load("XXX.Service");  
// 服务接口项目程序集  
Assembly iservice = Assembly.Load("XXX.IService");  
builder.RegisterAssemblyTypes(service, iservice)  
    .Where(t => t.FullName.EndsWith("Service") && !t.IsAbstract)  
    .InstancePerLifetimeScope()  
    .AsImplementedInterfaces();
```
上面的代码就是批量注入 `XXX.Service` 与 `XXX.IService` 项目下的服务与接口。

**注意：如果需要注入的服务没有 interfac ，那么 builder.RegisterAssemblyTypes 就只需要传一个程序集就OK了。如果服务与接口同在一个项目，那也是要传两个程序集。**

然后在构造函数中接收实例: 
```
private readonly ISayHelloService sayHelloService;  
private readonly IUseAutofacService useAutofacService;  
  
public HelloController(ISayHelloService _sayHelloService, IUseAutofacService _useAutofacService)  
{  
    this.sayHelloService = _sayHelloService;  
    this.useAutofacService = _useAutofacService;  
}
```

### 属性注入
虽然官方默认并建议使用构造函数注入, Autofac也是默认构造函数注入,但是会有一些场景需要用到属性注入.
对比构造函数注入，属性注入就多追加了 **PropertiesAutowired()** 函数
```
builder.RegisterAssemblyTypes(service, iservice) 
	.Where(t => t.FullName.EndsWith("Service") && !t.IsAbstract)
	.InstancePerLifetimeScope() 
	.AsImplementedInterfaces()
	.PropertiesAutowired(); // 属性注入
```
**属性注入记得将属性的访问修饰符改为注册类可访问的修饰符，否则会注入失败。**
```
public IUserService userService { get; set; }
public IXxxxService xxxxService { get; set; }
```
