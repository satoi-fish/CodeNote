## 一、终结点EndPoint
EndPoint是路由匹配的目标，当http请求到来时，路由模块就会将请求匹配到某个EndPoint。EndPoint这个类封装了action的信息，比如：Controller类型、Action方法、[Attribute]情况等。
```
namespace Microsoft.AspNetCore.Http
{
    public class Endpoint
    {
        public Endpoint(
            RequestDelegate requestDelegate,
            EndpointMetadataCollection metadata,
            string displayName)
        {
            RequestDelegate = requestDelegate;
            Metadata = metadata ?? EndpointMetadataCollection.Empty;
            DisplayName = displayName;
        }        
        public string DisplayName { get; }        
        public EndpointMetadataCollection Metadata { get; }        
        public RequestDelegate RequestDelegate { get; }
        public override string ToString() => DisplayName ?? base.ToString();
    }
}
```
## 二、终结点工作流程概述
“路由”模块一前一后注册了两个中间件，第一个中间件（EndpointRoutingMiddleware）匹配到了EndPoint，第二个中间件（EndpointMiddleware）去调用已经匹配到了的EndPoint。“授权”模块会在这两个中间件之间注册一个中间件，所以授权模块在进行授权的时候可以明确当前的请求是指向哪个Action的。
## 三、整体代码执行流程
### **这些代码的执行步骤如下：**
*程序启动阶段：*
第一步：执行services.AddControllers()
将Controller的核心服务注册到容器中去
第二步：执行app.UseRouting()
将EndpointRoutingMiddleware中间件注册到http管道中
第三步：执行app.UseAuthorization()
将AuthorizationMiddleware中间件注册到http管道中
第四步：执行app.UseEndpoints(endpoints=>endpoints.MapControllers())
有两个主要的作用：
调用endpoints.MapControllers()将本程序集定义的所有Controller和Action转换为一个个的EndPoint并放到路由中间件的配置对象RouteOptions中
将EndpointMiddleware中间件注册到http管道中

*当Http请求到来时*
第五步：收到一条http请求，此时EndpointRoutingMiddleware为它匹配一个Endpoint，并放到HttpContext中去
第六步：授权中间件进行拦截，根据Endpoint的信息对这个请求进行授权验证。
第七步：EndpointMiddleware中间件执行Endpoint中的RequestDelegate逻辑，即执行Controller的Action
## 四、ASP .Net core各模块之间关系
- “授权” 模块：Authorization
	“授权”模块注册了一个中间件进行授权拦截，拦截的对象是路由模块已匹配到的Endpoint。
- “路由” 模块：Routing
	“路由”模块前后注册了两个中间件，第一个用来寻找匹配的终结点，并将这个终结点放到HttpContext中，这之后其他的中间件可以针对匹配的结果做一些事情（比如：授权中间件），第二个用来执行终结点，也就是调用Controller中的action方法。
## 五、“路由”模块的两个中间件
**说明：**  
_在程序启动时每个action方法都会被封装成Endpoint对象交给路由模块，这样当http请求到来时，路由模块才会匹配到正确的Endpoint，进而找到Controller和Action！_  
“路由”模块注册中间件的源码如下:
```
namespace Microsoft.AspNetCore.Builder
{
    public static class EndpointRoutingApplicationBuilderExtensions
    {
        private const string EndpointRouteBuilder = "__EndpointRouteBuilder";
        public static IApplicationBuilder UseRouting(this IApplicationBuilder builder)
        {
            if (builder == null)
            {
                throw new ArgumentNullException(nameof(builder));
            }

            VerifyRoutingServicesAreRegistered(builder);

            var endpointRouteBuilder = new DefaultEndpointRouteBuilder(builder);
            builder.Properties[EndpointRouteBuilder] = endpointRouteBuilder;

            return builder.UseMiddleware<EndpointRoutingMiddleware>(endpointRouteBuilder);
        }
        public static IApplicationBuilder UseEndpoints(this IApplicationBuilder builder, Action<IEndpointRouteBuilder> configure)
        {
            if (builder == null)
            {
                throw new ArgumentNullException(nameof(builder));
            }

            if (configure == null)
            {
                throw new ArgumentNullException(nameof(configure));
            }

            VerifyRoutingServicesAreRegistered(builder);

            VerifyEndpointRoutingMiddlewareIsRegistered(builder, out var endpointRouteBuilder);

            configure(endpointRouteBuilder);
            var routeOptions = builder.ApplicationServices.GetRequiredService<IOptions<RouteOptions>>();
            foreach (var dataSource in endpointRouteBuilder.DataSources)
            {
                routeOptions.Value.EndpointDataSources.Add(dataSource);
            }

            return builder.UseMiddleware<EndpointMiddleware>();
        }
        ......
    }
}
```

从上面的源码中可以看到：
- UseRouting()这个方法很简单就是注册了EndpointRoutingMiddleware中间件；
- UseEndpoints()这个方法是先生成一个EndpointRouteBuilder，然后调用我们的代码endpoints.MapControllers()将所有的action都包装成Endpoint放进了EndpointRouteBuilder的DataSources属性中，然后又将这些Endpoint放进了RouteOptions的EndpointDataSources属性中，最后注册了EndpointMiddleware中间件。
> 至于action是怎么被封装成Endpoint的这里不做介绍（参考：.netcore入门13：aspnetcore源码之如何在程序启动时将Controller里的Action自动扫描封装成Endpoint），我们现在只要知道“路由”模块在程序启动的时候就已经将所有的action转化成了Endpoint以进行管理，并且注册了两个中间件。