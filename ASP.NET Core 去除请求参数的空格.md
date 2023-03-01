在ASP.NET Core WebApi项目中，后端接收用户输入时通常需要做去除空格处理，通过查看官网和源码后，发现继承并实现 IModelBinder和IModelBinderProvider即可。

我们知道，模型绑定有[FromQuery]、[FromRoute]、[FromForm]、[FromBody]等多种绑定方式，我们先从简单类型着手，处理[FromQuery]、[FromRoute]这些情况

### 处理简单类型

先定义一个简单类型的Trim绑定器**SimpleStringTrimModelBinder**

```c#
    /// <summary>
    /// 简单类型Trim绑定器
    /// </summary>
    public class SimpleStringTrimModelBinder : IModelBinder
    {
        private readonly Type _type;

        public SimpleStringTrimModelBinder(Type type)
        {
            _type = type;
        }

        public Task BindModelAsync(ModelBindingContext bindingContext)
        {
            if (bindingContext == null)
            {
                throw new ArgumentNullException(nameof(bindingContext));
            }
            var valueProvider = bindingContext.ValueProvider;
            var valueProviderResult = valueProvider.GetValue(bindingContext.ModelName);

            if (valueProviderResult != ValueProviderResult.None)
            {
                bindingContext.Result = ModelBindingResult.Success(valueProviderResult.FirstValue.Trim());
                return Task.CompletedTask;
            }

            //调用默认绑定
            SimpleTypeModelBinder simpleTypeModelBinder = new SimpleTypeModelBinder(_type, (ILoggerFactory)bindingContext.HttpContext.RequestServices.GetService(typeof(ILoggerFactory)));
            return simpleTypeModelBinder.BindModelAsync(bindingContext);
        }
    }
```

再定义一个简单类型Trim绑定提供器**StringTrimModelBinderProvider**

```c#
    /// <summary>
    /// 简单类型Trim绑定提供器，用于[FromQuery]参数Trim处理
    /// </summary>
    public class StringTrimModelBinderProvider : IModelBinderProvider
    {
        public IModelBinder GetBinder(ModelBinderProviderContext context)
        {
            if (context == null)
            {
                throw new ArgumentNullException(nameof(context));
            }
            if (!context.Metadata.IsComplexType && context.Metadata.ModelType == typeof(string))
            {
                //简单类型
                return new SimpleStringTrimModelBinder(context.Metadata.ModelType);
            }

            return null;
        }
    }
```

最后在**Program.cs**中注册该提供器

```c#
    ......
	builder.Services.AddControllers(option =>
    {
        option.ModelBinderProviders.Insert(0, new StringTrimModelBinderProvider());
    });
	......
```

此时，在Controller中使用 [FromQuery] 、[FromRoute] 解析请求参数时，即可默认去除空格

```c#
        [HttpGet]
		public IActionResult findAll([FromQuery] string name)
        {
            //name等参数默认去除空格
        }
```



### 处理复杂类型

那么针对[FromBody]这种绑定复杂类型的，如果继续沿用上面的思路，则需要构建一个复杂类型模型绑定器，通过反射逐个解析每一个属性，再针对string类型去除空格，虽然技术层面仍可以实现，但考虑到复杂类型的属性嵌套，就不免有些复杂了。我们不妨换一种思路：ASP.NET Core框架在将Request.Body解析到指定实体类的过程中，是通过*System.Text.Json*实现json反序列化的，那么在其json反序列的过程中，将string类型的属性去除空格即可。定义以下**StringTrimJsonConverter**

```c#
	using System.Text.Json;
	using System.Text.Json.Serialization;   

	/// <summary>
    /// 用于[FromBody]属性中的string参数Trim处理
    /// </summary>
    public class StringTrimJsonConverter : JsonConverter<string>
    {
        public override string Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
        {
            return reader.GetString().Trim();
        }

        public override void Write(Utf8JsonWriter writer, string value, JsonSerializerOptions options)
        {
            writer.WriteStringValue(value);
        }
    }
```

最后在**Program.cs**中注册该json转换器：

```c#
		......
		builder.Services.AddControllers()
        .AddJsonOptions(option =>
        {
            option.JsonSerializerOptions.Converters.Add(new StringTrimJsonConverter());
        });
		......
```

此时，在Controller中使用 [FromBody] 解析请求参数属性时，即可默认去除空格

```c#
		[HttpPost]
        public IActionResult save([FromBody] ApiDto apiDto)
        {
            //apiDto.Name属性默认去除空格
        }
```

