## HttpClient的正确用法

HttpClient是C#中原生的Http请求类，但一直以来，我们没有正确的使用它，不仅平白地消耗了服务器的性能，更是降低了系统的稳定性。

随着微服务概念的流行，服务越分越细，服务间也出现穿插调用，服务间调用有很多方式，HTTP是较为流行的方式之一，那最为常用的便是HttpClient类。

通常对于HttpClient如下用法：

```c#
using(var client = new HttpClient())
{
    //do something with http client
}
```

#### 矛盾之处

``using``语句在C#中是处理``disposable ``类型的绝佳方式，一旦``disposable ``类型被``using``包裹，在包裹之外，该类型就会自动释放，无论该资源是否仍在使用还是仍未清理。因此，无论是在数据库连接访问还是流处理的情形下，经典的处理方式都是以``using``包裹。事实上我们也习惯了对于实现了``IDisposable``接口的类的使用，以``using``包裹之。

连微软官方对于``using``的表述则是

> *As a rule, when you use an IDisposable object, you should declare and instantiate it in a using statement.*

那么对于HttpClient类，我们是否也理所当然的用``using``包裹起来呢？

事实上，HttpClient是不同的，尽管它实现了``IDisposable``接口，但它实际上是一个共享对象，这意味着在它的表象之下。它是可重用的和线程安全的，因此，应该在应用的整个生命周期里使用一个HttpClient实例，而不是每次使用调用时都创建一个实例。

#### 代码回顾

下面是一个使用HttpClient的简单例子

```c#
using System;
using System.Net.Http;

namespace ConsoleApplication
{
    public class Program
    {
        public static async Task Main(string[] args) 
        {
            Console.WriteLine("Starting connections");
            for(int i = 0; i<10; i++)
            {
                using(var client = new HttpClient())
                {
                    var result = await client.GetAsync("http://aspnetmonsters.com");
                    Console.WriteLine(result.StatusCode);
                }
            }
            Console.WriteLine("Connections done");
        }
    }
}
```

上面的代码是使用``Get``方法对某站点发起连续10次请求，这里仅简单打印其状态码，输出如下：

```c#
C:\code\socket> dotnet run
Project socket (.NETCoreApp,Version=v1.0) will be compiled because inputs were modified
Compiling socket for .NETCoreApp,Version=v1.0

Compilation succeeded.
    0 Warning(s)
    0 Error(s)

Time elapsed 00:00:01.2501667


Starting connections
OK
OK
OK
OK
OK
OK
OK
OK
OK
OK
Connections done
```

#### 问题所在

上面的代码看似没问题，运行输出也正常，但使用``netstat``工具显示如下：

```c#
C:\code\socket>NETSTAT.EXE
...
  Proto  Local Address          Foreign Address        State
  TCP    10.211.55.6:12050      waws-prod-bay-017:http  TIME_WAIT
  TCP    10.211.55.6:12051      waws-prod-bay-017:http  TIME_WAIT
  TCP    10.211.55.6:12053      waws-prod-bay-017:http  TIME_WAIT
  TCP    10.211.55.6:12054      waws-prod-bay-017:http  TIME_WAIT
  TCP    10.211.55.6:12055      waws-prod-bay-017:http  TIME_WAIT
  TCP    10.211.55.6:12056      waws-prod-bay-017:http  TIME_WAIT
  TCP    10.211.55.6:12057      waws-prod-bay-017:http  TIME_WAIT
  TCP    10.211.55.6:12058      waws-prod-bay-017:http  TIME_WAIT
  TCP    10.211.55.6:12059      waws-prod-bay-017:http  TIME_WAIT
  TCP    10.211.55.6:12060      waws-prod-bay-017:http  TIME_WAIT
  TCP    10.211.55.6:12061      waws-prod-bay-017:http  TIME_WAIT
  TCP    10.211.55.6:12062      waws-prod-bay-017:http  TIME_WAIT
  TCP    127.0.0.1:1695         SIMONTIMMS742B:1696    ESTABLISHED
...
```

可以看到，程序虽然已经退出，但与站点间仍然存在大量的连接，状态是``TIME_WAIT``意味着连接在一方面已经断开，但我们仍然在等待其是否还有额外的数据包传来，TCP/IP状态图：

![tcpstate](https://www4.cs.fau.de/Projects/JX/Projects/TCP/tcpstate.gif)

Windows会将连接保持这种状态长达240s，时长可在``[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\TcpTimedWaitDelay]``中设置，如果将连接池用尽，将会看到如下的错误：

> Unable to connect to the remote server
> System.Net.Sockets.SocketException: Only one usage of each socket address (protocol/network address/port) is normally permitted.

如果就这个错误在网上搜索，会搜出一大堆糟糕的建议，教你如何调低连接的超时时间，但这样会带来一系列新的问题，因此，我们应探索如何让正确的处理这一错误。

#### 正确方式

如果我们使用单一的共享类，则可以减少socket的浪费

```c#
using System;
using System.Net.Http;

namespace ConsoleApplication
{
    public class Program
    {
        private static HttpClient Client = new HttpClient();
        public static async Task Main(string[] args) 
        {
            Console.WriteLine("Starting connections");
            for(int i = 0; i<10; i++)
            {
                var result = await Client.GetAsync("http://aspnetmonsters.com");
                Console.WriteLine(result.StatusCode);
            }
            Console.WriteLine("Connections done");
            Console.ReadLine();
        }
    }
}
```

这次我们在整个应用程序中使用单个HttpClient实例，``netstat``显示如下

```c#
TCP    10.211.55.6:12254      waws-prod-bay-017:http  ESTABLISHED
```

#### 写在最后

总而言之，我们需要谨记两件事：

1. 使你的 `HttpClient` 类静态化.
2. 除非有特殊的需求，*不要* 轻易释放或使用using语句包裹 `HttpClient` 类