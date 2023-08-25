## 依赖注入
ASP.NET Core 支持依赖关系注入 (DI) 软件设计模式，这是一种在类及其依赖关系之间实现控制反转 (IoC)的技术。
ASP.NET Core 的依赖注入（Dependency Injection，简称 DI）是一种设计模式和技术，用于管理和组织应用程序中各个组件之间的依赖关系。通过依赖注入，你可以将组件的创建和管理责任从应用程序代码中解耦出来，实现松耦合、可测试性和可维护性。
**依赖关系：** 在应用程序中，不同的组件可能需要访问其他组件的功能或资源，这些被访问的组件称为依赖。依赖注入的目标是通过将依赖从组件的代码中分离出来，以降低耦合度。
> 软件设计原则中有一个依赖倒置原则（DIP），就是为了解耦；高层模块不应该依赖于底层模块。二者都应该依赖于抽象；抽象不应该依赖于细节，细节应该依赖于抽象；而依赖注入是实现这种原则的方式之一；

```
interface IUserInfoService  
{  
    IEnumerable<UserInfo> GetUserInfo();  
}  
  
public class UserInfo  
{  
    public int Id { get; set; }  
    public string Name { get; set; }  
}  
  
public class UserInfoService : IUserInfoService  
{  
    public IEnumerable<UserInfo> GetUserInfo()  
    {  
        // 模拟db获取数据  
        return new List<UserInfo> { new UserInfo { Id = 1, Name = "Emrys" }, new UserInfo { Id = 2, Name = "梅林" } };  
    }  
}  
  
public class UserInfoMongoService : IUserInfoService  
{  
    public IEnumerable<UserInfo> GetUserInfo()  
    {  
        // 模拟Mongodb获取数据  
        return new List<UserInfo> { new UserInfo { Id = 1, Name = "Emrys" }, new UserInfo { Id = 2, Name = "梅林" } };  
    }  
}
```

**传统方式**
```
public class ValuesController : ControllerBase  
{  
    IUserInfoService _userInfoService = new UserInfoService();  
  
    [HttpGet]
    public IEnumerable<UserInfo> Get()  
    {   
        return _userInfoService.GetUserInfo();  
    }  
}
```
在传统方式中, 获取用户的服务类直接用new的方式, 从中可以发现代码耦合度太高, 非常不利于维护，在所有使用到IUserInfoService的地方都要new出对象, 如果后期更改UserInfoService的实现, 换成UserInfoMongoService, 那么替换工作就会很繁琐也就是需要解耦.
**依赖注入方式**
```
public void ConfigureServices(IServiceCollection services)  
{  
    services.AddTransient<IUserInfoService, UserInfoService>();  
}  
  
public class ValuesController : ControllerBase  
{  
    IUserInfoService _userInfoService;  
  
    public ValuesController(IUserInfoService userInfoService)  
    {  
        _userInfoService = userInfoService;  
    }  
  
    [HttpGet]  
    public IEnumerable<UserInfo> Get()  
    {  
        return _userInfoService.GetUserInfo();  
    }  
}
```

> ## 使用扩展方法注册服务组
> 
> ASP.NET Core 框架使用一种约定来注册一组相关服务。 约定使用单个 `Add{GROUP_NAME}` 扩展方法来注册该框架功能所需的所有服务。 例如，[AddControllers](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addcontrollers) 扩展方法会注册 MVC 控制器所需的服务。
> 
> 下面的代码通过个人用户帐户由 Razor 页面模板生成，并演示如何使用扩展方法 [AddDbContext](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) 和 [AddDefaultIdentity](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.extensions.dependencyinjection.identityservicecollectionuiextensions.adddefaultidentity) 将其他服务添加到容器中：
> ```
> public void ConfigureServices(IServiceCollection services)
> {
>     services.AddDbContext<ApplicationDbContext>(options =>
>         options.UseSqlServer(
>             Configuration.GetConnectionString("DefaultConnection")));
>     services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
>         .AddEntityFrameworkStores<ApplicationDbContext>();
>     services.AddRazorPages();
> }
> ```
