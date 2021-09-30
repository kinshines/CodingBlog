## 使用NAutowired以类似注解的方式实现自动注入

使用过java Spring boot框架的朋友一定知道，以注解（Annotation）的方式实现基于属性的依赖注入可以带来无与伦比的开发体验，那么在.NET环境中可有类似的机制吗？微软官方给出的回复是没有，原因是基于属性的依赖注入的不安全的，无法保证非null值，因此只提供基于构造方法的注入。但没有什么能够难倒我们勤劳智慧的中国人民的，今天就来介绍一款实用的package：NAutowired，以属性（Attribute）的方式实现自动的依赖注入。

首先添加NAutowired的引用

> Install-Package NAutowired

修改`Startup`类中的`ConfigureServices`方法

```c#
public void ConfigureServices(IServiceCollection services)
        {
            ......
            services.AddControllersWithViews().AddControllersAsServices();
            services.Replace(ServiceDescriptor.Transient<IControllerActivator, NAutowiredControllerActivator>());
            //Add FooService to container.
            services.AddScoped<FooService>();
            ......
        }
```

之后，NAutowired就可以在`Controller`中自动注入`Service`

```c#
  [Route("api/[controller]")]
  [ApiController]
  public class FooController : ControllerBase {

    //Use Autowired injection.
    [Autowired]
    private readonly FooService fooService;

    [HttpGet]
    public ActionResult<string> Get() {
      return fooService == null ? "failure" : "success";
    }
  }
```

在`Filter`中引用`Service`

```c#
  public class Startup {
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services) {
      //Add Filter to container.
      services.AddScoped<AuthorizationFilter>();
    }
  }
```

```c#
  //Use ServiceFilter like ASP.NET CORE ServiceFilter.
  [NAutowired.Attributes.ServiceFilter(typeof(AuthorizationFilter))]
  public class FooController : ControllerBase {

  }
```

```c#
  public class AuthorizationFilter : IAuthorizationFilter {
    [Autowired]
    private readonly FooService fooService;

    public void OnAuthorization(AuthorizationFilterContext context) {
      System.Console.WriteLine($"{fooService.ToString()} in filter");
      return;
    }
  }
```

获取配置项`Configuration`

```c#
  public class Startup {
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services) {
      //add config to ioc container
      services.Configure<SnowflakeConfig>(Configuration.GetSection("Snowflake"));
    }
  }
```

```c#
public class FooController : ControllerBase {
  //use autowired get configuration
    [Autowired]
    private IOptions<SnowflakeConfig> options { get; set; }

    [HttpGet("snowflake")]
    public IActionResult GetSnowflakeConfig()
    {
        return Ok(options.Value);
    }
}
```

`SnowflakeConfig`

```c#
public class SnowflakeConfig
{
    public int DataCenter { get; set; }

    public int Worker { get; set; }
}
```

`appsettings.json`

```json
{
  "Snowflake": {
    "DataCenter": 1,
    "Worker": 1
  }
}
```

另外，还提供了`[Autowired(Type)]`方式注入具体类

```c#
  [Route("api/[controller]")]
  [ApiController]
  public class FooController : ControllerBase {

    //Inject a specific instance.
    [Autowired(typeof(FooService))]
    private readonly IFooService fooService;

    [HttpGet]
    public ActionResult<string> Get() {
      return fooService == null ? "failure" : "success";
    }
  }
```

最后，还贴心的提供了自动扫描依赖的快捷方式，使用`AutoRegisterDependency(assemblyName)`指定程序集名称，即可将指定程序集内添加`[Service] [Repository] [Component] [ServiceFilter]`标签的类自动注入依赖

```c#
  public class Startup {
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services) {
      //services.AddScoped<FooService>();
      //Use automatic injection.
      services.AutoRegisterDependency(new List<string> { "NAutowiredSample" });
    }
  }
```

```c#
  //The default Lifetime value is Scoped
  [Service]
  //Lifetime to choose the life cycle of dependency injection
  //[Service(Lifetime.Singleton)]
  public class FooService {
  }

  [Service(implementInterface: typeof(IService))]
  //injection interface to container like services.AddScoped(typeof(IService), typeof(FooService));
  public class FooService: IService {
  }
```

以上便是NAutowired的基本用法，但需要注意的是，官方是不推荐使用这种方式实现注入的，因此在VS中编码及编译时，都会得到类似的警告

> Warning	CS0649	Field 'FooController.fooService' is never assigned to, and will always have its default value null
