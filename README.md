## ASP.NET CORE JWT 简单Demo 提供

#### Server & Client
> The project uses client and server credential identification. It's very simple and quick to try
#### MySelf Auth
> This example is the use of Authorization, Authentication in a single project, a mode for yourself to play without the Server
#### Cookie Sample
> Use cookies to verify login in ASP.NET Core, we are very simple to use it, login and logout functions
#### Policy Sample
> This example uses ASP.NET Core for custom authentication JWT. In the example, Swagger is also used for testing. The one-stop configuration is available at your convenience.

I saw a lot of senior pits, the original (context.Resource as Microsoft.AspNetCore.Routing.RouteEndpoint); In fact, it is no longer available in .NET Core 3.0. The reason is that after EndpointRouting is enabled in .NET Core 3.0, the permission filter is no longer Add to ActionDescriptor, and run permissions directly as middleware. At the same time, all filters will be added to endpoint.Metadata. If in .NET Core 2.1 & 2.2, you can usually write Handler like this:

我需要引用一段代码：  
``` c#
public class PolicyHandler : AuthorizationHandler<PolicyRequirement>
    {
        protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, PolicyRequirement requirement)
        {
            //赋值用户权限
            var userPermissions = requirement.UserPermissions;
            //从AuthorizationHandlerContext转成HttpContext，以便取出表求信息
            var httpContext = (context.Resource as Microsoft.AspNetCore.Mvc.Filters.AuthorizationFilterContext).HttpContext;
            //请求Url
            var questUrl = httpContext.Request.Path.Value.ToUpperInvariant();
            //是否经过验证
            var isAuthenticated = httpContext.User.Identity.IsAuthenticated;
            if (isAuthenticated)
            {
                if (userPermissions.GroupBy(g => g.Url).Any(w => w.Key.ToUpperInvariant() == questUrl))
                {
                    //用户名
                    var userName = httpContext.User.Claims.SingleOrDefault(s => s.Type == ClaimTypes.NameIdentifier).Value;
                    if (userPermissions.Any(w => w.UserName == userName && w.Url.ToUpperInvariant() == questUrl))
                    {
                        context.Succeed(requirement);
                    }
                    else
                    {
                        //无权限跳转到拒绝页面
                        httpContext.Response.Redirect(requirement.DeniedAction);
                    }
                }
                else
                    context.Succeed(requirement);
            }
            return Task.CompletedTask;
        }
    }
```
