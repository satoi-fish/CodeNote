## 路由

现在，我们来了解如何将请求路由到不同的控制器

首先，ASP.NET Core 中间件需要一个方法来确定给定的 HTTP 请求是否应该发送给控制器进行处理，我们将这个过程称之为路由匹配,  MVC 中间件将根据我们提供的 URL 和一些配置信息做出此决定, 这种方法通常被称为基于约定的路由。

以下代码是常规路由的代码片段

```
routeBuilder.MapRoute("Default", "{controller=Home}/{action=Index}/{id?}");
```
## 属性路由

通过基于属性的路由，我们可以在控制器类和这些类的内部方法上使用 `C#` 属性。 这些属性携带了告诉 ASP.NET Core 何时调用特定控制器的元数据

1. 属性路由是基于约定的路由的替代方案
2. 路由按照它们出现的顺序进行评估，也就是我们注册它们的顺序，映射多个路由的情况相当普遍，特别是如果我们想在 URL 中使用不同的参数或者如果要在 URL 中使用不同的文字

我们举一个简单的例子。

打开并运行 HelloWorld 项目，然后在浏览器中访问应用程序。当我们访问 `/about` 时，它会产生下面的输出

![](https://www.twle.cn/static/i/aspnetcore/aspnetcore_routing_5.png?1)

我们想要的是，当我们访问 `/about` 时，应用程序应该调用 `AboutController` 的 `Phone` 方法

针对这种情况，我们可以使用 `Microsoft.AspNetCore.Mvc` 命名空间下的 `Route` 属性为该控制器强制执行一些显式路由

下面的代码是添加了属性路由的 `AboutController` 的实现

```
using System;
using Microsoft.AspNetCore.Mvc;
namespace HelloWorld.Controllers
{
    [Route("about")]
    public class AboutController
    {
        public AboutController()
        {
        }

        [Route("")]
        public string Phone()
        {
            return "+10086"; 
        }  

        [Route("country")]
        public string Country()
        {
            return "中国"; 
        } 
    }
}
```

在这里，我们给 `Phone()` 方法添加了一个空的路由属性，这意味用户只需要访问 `/about`，而不需要指定操作就可以访问此方法。对于 `country` 方法，我们在路由属性中指定了 `country`

保存下 `AboutController.cs` ，刷新浏览器并访问 `/about`，我们可以看到正常输出了电话号码

![](https://www.twle.cn/static/i/aspnetcore/aspnetcore_attributerouting_1.png?1)

如果我们访问 `/about/country` ，那么这将访问 `AboutController` 控制器中的 `Country()` 方法

![](https://www.twle.cn/static/i/aspnetcore/aspnetcore_routing_4.png?1)

如果希望 URL 的一段包含我们的控制器的名称，那么我们可以不直接显示指定控制器的名称，取而代之的是在方括号内使用令牌控制器，这用于告诉 ASP.NET MVC 在此位置使用此控制器的名称

如以下程序中所示

```
using System;
using Microsoft.AspNetCore.Mvc;

namespace HelloWorld.Controllers
{
    [Route("[controller]")]
    public class AboutController
    {
        public AboutController()
        {
        }

        [Route("")]
        public string Phone()
        {
            return "+10086"; 
        }  

        [Route("[action]")]
        public string Country()
        {
            return "中国"; 
        } 
    }
}
```
使用这种方式，即使我们重命名路由器，也不必去更改路由。 对于一个动作也是一样，并且隐式地在控制器和动作之间有一个斜杠 ( `/` ) 。 它是控制器和动作之间的层次关系，就像它在 URL 中一样