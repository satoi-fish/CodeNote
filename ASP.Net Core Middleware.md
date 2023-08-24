## 中间件
中间件是一种装配到应用管道以处理请求和响应的软件。
使用 `RunMap` 和 `Use`扩展方法来配置请求委托。 可将一个单独的请求委托并行指定为匿名方法（称为并行中间件），或在可重用的类中对其进行定义。 这些可重用的类和并行匿名方法即为中间件，也叫中间件组件。
请求管道中的每个组件负责调用管道中的下一个组件,如果没有调用下一个则为管道短路, 被称为‘终端中间件’.
## 使用 IApplicationBuilder 创建中间件管道
![image](https://github.com/satoi-fish/CodeNote/assets/81409285/f8b9c93c-c149-4252-af56-4db270fed577)
<br/>每个委托均可在下一个委托前后执行操作。 应尽早在管道中调用异常处理委托，这样它们就能捕获在管道的后期阶段发生的异常。
用 `Use`将多个请求委托链接在一起形成一个管道, `next` 参数表示管道中的下一个委托。 可通过不调用 next 参数使管道短路。 
```
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        app.Use(async (context, next) =>
        {
            await next.Invoke();
        });

        app.Run(async context =>
        {
            await context.Response.WriteAsync("Hello from 2nd delegate.");
        });
    }
}
```
> **警告**
> 在向客户端发送响应后，请勿调用 `next.Invoke`
> 调用 `next` 后写入响应正文：
> 
> - 可能导致违反协议。 例如，写入的长度超过规定的 `Content-Length`。
> - 可能损坏正文格式。 例如，向 CSS 文件中写入 HTML 页脚。
`Run`委托是作为终端用于终止管道, 所以其不会收到`next`参数.
```
app.Run(async context => { await context.Response.WriteAsync("Hello from 2nd delegate."); });
```
## 中间件顺序
下图显示了 ASP.NET Core MVC 和 Razor Pages 应用的完整请求处理管道。
![image](https://github.com/satoi-fish/CodeNote/assets/81409285/ec6278c4-0899-4899-86fc-d99375936300)
<br/>向 `Startup.Configure` 方法添加中间件组件的顺序定义了针对请求调用这些组件的顺序，以及响应的相反顺序。 此顺序对于安全性、性能和功能至关重要. 上述为典型的建议顺序.
```
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    // app.UseCookiePolicy();

    app.UseRouting();
    // app.UseRequestLocalization();
    // app.UseCors();

    app.UseAuthentication();
    app.UseAuthorization();
    // app.UseSession();
    // app.UseResponseCompression();
    // app.UseResponseCaching();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller=Home}/{action=Index}/{id?}");
    });
}
```
上述是建议顺序的代码实现
可以不用完全按照此顺序, 但是:
- `UseCors`、`UseAuthentication` 和 `UseAuthorization` 必须按显示的顺序出现。
- `UseCors`必须在`UseResponseCaching`之前出现
- `UseRequestLocalization` 必须在可能检查请求区域性的任何中间件之前出现
> 以下 `Startup.Configure` 方法将为常见应用方案添加中间件组件：
> 
> 1. 异常/错误处理
>     - 当应用在开发环境中运行时：
>         - 开发人员异常页中间件 ([UseDeveloperExceptionPage](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.developerexceptionpageextensions.usedeveloperexceptionpage)) 报告应用运行时错误。
>         - 数据库错误页中间件报告数据库运行时错误。
>     - 当应用在生产环境中运行时：
>         - 异常处理程序中间件 ([UseExceptionHandler](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.exceptionhandlerextensions.useexceptionhandler)) 捕获以下中间件中引发的异常。
>         - HTTP 严格传输安全协议 (HSTS) 中间件 ([UseHsts](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.hstsbuilderextensions.usehsts)) 添加 `Strict-Transport-Security` 标头。
> 2. HTTPS 重定向中间件 ([UseHttpsRedirection](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.httpspolicybuilderextensions.usehttpsredirection)) 将 HTTP 请求重定向到 HTTPS。
> 3. 静态文件中间件 ([UseStaticFiles](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles)) 返回静态文件，并简化进一步请求处理。
> 4. Cookie 策略中间件 ([UseCookiePolicy](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.cookiepolicyappbuilderextensions.usecookiepolicy)) 使应用符合欧盟一般数据保护条例 (GDPR) 规定。
> 5. 用于路由请求的路由中间件 ([UseRouting](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.endpointroutingapplicationbuilderextensions.userouting))。
> 6. 身份验证中间件 ([UseAuthentication](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication)) 尝试对用户进行身份验证，然后才会允许用户访问安全资源。
> 7. 用于授权用户访问安全资源的授权中间件 ([UseAuthorization](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.authorizationappbuilderextensions.useauthorization))。
> 8. 会话中间件 ([UseSession](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.sessionmiddlewareextensions.usesession)) 建立和维护会话状态。 如果应用使用会话状态，请在 Cookie 策略中间件之后和 MVC 中间件之前调用会话中间件。
> 9. 用于将 Razor Pages 终结点添加到请求管道的终结点路由中间件（带有 [MapRazorPages](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.razorpagesendpointroutebuilderextensions.maprazorpages) 的 [UseEndpoints](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.builder.endpointroutingapplicationbuilderextensions.useendpoints)）。
