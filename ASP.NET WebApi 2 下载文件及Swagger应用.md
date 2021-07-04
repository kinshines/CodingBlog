# ASP.NET WebApi 2 下载文件及Swagger应用

## 文件下载
WebApi 2 实现文件下载，可在 *MemoryStream* 中处理文件后，借助 *HttpResponseMessage* 及*HttpResponseMessage* 两个类实现文件下载

```
using System.IO;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Web.Http;
using System.Web.Http.Results;

namespace WebApi2DownloadInMemoryFile.Controllers
{
    public class FileDownloadController : ApiController
    {
        public IHttpActionResult Download()
        {
            string someTextToSendAsAFile = "Hello world";
            byte[] textAsBytes = Encoding.Unicode.GetBytes(someTextToSendAsAFile);

            MemoryStream stream = new MemoryStream(textAsBytes);

            HttpResponseMessage httpResponseMessage = new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new StreamContent(stream)
            };
            httpResponseMessage.Content.Headers.ContentDisposition = new ContentDispositionHeaderValue("attachment")
            {
                FileName = "WebApi2GeneratedFile.txt"
            };
            httpResponseMessage.Content.Headers.ContentType = new MediaTypeHeaderValue("text/plain");

            ResponseMessageResult responseMessageResult = ResponseMessage(httpResponseMessage);
            return responseMessageResult;
        }
    }
}
```

## Swagger中的应用
在Swagger中使用WebApi的文件下载功能，需要 *SwaggerConfig* 类中稍加配置



1. 添加 *FileOperation* 类
```
public class FileOperation : IOperationFilter
        {
            public void Apply(Operation operation, SchemaRegistry schemaRegistry, ApiDescription apiDescription)
            {
                if (operation.operationId.ToLower().Contains("download"))
                {
                    operation.produces = new[] { "application/octet-stream" };
                    operation.responses["200"].schema = new Schema { type = "file", description = "Download file" };
                }
            }
        }
```
2.在原有的 *Register* 静态方法中添加该 *OperationFilter*

```
public static void Register()
        {
            var thisAssembly = typeof(SwaggerConfig).Assembly;

            GlobalConfiguration.Configuration
                .EnableSwagger(c =>
                    {
                        ......
                        c.OperationFilter<FileOperation>();
                    })
                        ......
        }
```