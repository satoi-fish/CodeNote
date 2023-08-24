在 .NET 编程中，`async` 和 `await` 是用于处理异步操作的关键字。它们是 C# 编程语言中引入的特性，用于简化异步代码的编写和理解。异步编程允许在执行某些操作时，不会阻塞主线程，从而提高应用程序的响应性能。
- `async`：`async` 关键字用于修饰一个方法，表示这个方法是一个异步方法。异步方法可以包含 `await` 表达式，用于在异步操作完成之前暂停方法的执行并允许主线程继续执行其他操作。
- `await`：`await` 关键字用于在异步方法内部等待一个异步操作的完成。当遇到 `await` 表达式时，方法的执行将暂停，直到异步操作完成并返回结果。
异步操作通常用于处理需要花费时间的任务，比如网络请求、文件读写、数据库查询等。通过使用 `async` 和 `await`，可以让代码在执行这些耗时操作时，不会阻塞主线程，从而保持应用程序的响应性能和用户体验。
```
static void Main(string[] args)
{
    ConsoleAsync();
    ConsoleSync();
    Console.ReadKey();
}

static async void ConsoleAsync()
{
    await Task.Run(() =>
    {
        for (int i = 0; i < 100; i++)
        {
            Console.WriteLine(" Method 1");
        }
    });
}

static void ConsoleSync()
{
    for (int i = 0; i < 25; i++)
    {
        Console.WriteLine(" Method 2");
    }
}
```
![image](https://github.com/satoi-fish/CodeNote/assets/81409285/9ec26732-06b6-4b37-a2bf-368637ef826a)
<br/>可以看到两个互不依赖的方法调用的时候如果同步进行就会显示1再显示2,而异步方法会同步进行.
如果相互依赖的两个方法执行时,就会对后续代码进行阻塞,直到执行完前面的代码.
```
static void Main(string[] args)  
{  
    RunningMethods();  
    Console.ReadKey();  
}  
  
static async void RunningMethods()  
{    
    int count = await ConsoleAsync();  
    Console.WriteLine(count);  
    ConsoleSync();
}  
  
static async Task<int> ConsoleAsync()  
{  
    int count = 0;  
    await Task.Run(() =>  
    {  
        for (int i = 0; i < 100; i++)  
        {  
            Console.WriteLine(" Method 1");  
            count++;  
        }  
    });  
    return count;  
}  
  
  
static void ConsoleSync()  
{  
    for (int i = 0; i < 25; i++)  
    {  
        Console.WriteLine(" Method 2");  
    }  
}
```
![image](https://github.com/satoi-fish/CodeNote/assets/81409285/45036c28-bcab-4b02-b0ed-effa6e61b639)
<br/>可以看到如果是在await运算符之后的代码会在其运算完毕之后才会执行,也就是说它是将原本异步的方法与同步的方法统一成了同步的执行顺序.
##   异步方法的运行机制
![异步控制流的跟踪导航](https://learn.microsoft.com/zh-cn/dotnet/csharp/asynchronous-programming/media/task-asynchronous-programming-model/navigation-trace-async-program.png)
关系图中的数字对应于以下步骤，在调用方法调用异步方法时启动。
1. 调用方法调用并等待 `GetUrlContentLengthAsync` 异步方法。
    
2. `GetUrlContentLengthAsync` 可创建 [HttpClient](https://learn.microsoft.com/zh-cn/dotnet/api/system.net.http.httpclient) 实例并调用 [GetStringAsync](https://learn.microsoft.com/zh-cn/dotnet/api/system.net.http.httpclient.getstringasync) 异步方法以下载网站内容作为字符串。
    
3. `GetStringAsync` 中发生了某种情况，该情况挂起了它的进程。 可能必须等待网站下载或一些其他阻止活动。 为避免阻止资源，`GetStringAsync` 会将控制权出让给其调用方 `GetUrlContentLengthAsync`。
    
    `GetStringAsync` 返回 [Task<TResult>](https://learn.microsoft.com/zh-cn/dotnet/api/system.threading.tasks.task-1)，其中 `TResult` 为字符串，并且 `GetUrlContentLengthAsync` 将任务分配给 `getStringTask` 变量。 该任务表示调用 `GetStringAsync` 的正在进行的进程，其中承诺当工作完成时产生实际字符串值。
    
4. 由于尚未等待 `getStringTask`，因此，`GetUrlContentLengthAsync` 可以继续执行不依赖于 `GetStringAsync` 得出的最终结果的其他工作。 该任务由对同步方法 `DoIndependentWork` 的调用表示。
    
5. `DoIndependentWork` 是完成其工作并返回其调用方的同步方法。
    
6. `GetUrlContentLengthAsync` 已运行完毕，可以不受 `getStringTask` 的结果影响。 接下来，`GetUrlContentLengthAsync` 需要计算并返回已下载的字符串的长度，但该方法只有在获得字符串的情况下才能计算该值。
	因此，`GetUrlContentLengthAsync` 使用一个 await 运算符来挂起其进度，并把控制权交给调用 `GetUrlContentLengthAsync` 的方法。 `GetUrlContentLengthAsync` 将 `Task<int>` 返回给调用方。 该任务表示对产生下载字符串长度的整数结果的一个承诺。
    
   > ***备注***<br/>
   > ***如果 `GetStringAsync`（因此 `getStringTask`）在 `GetUrlContentLengthAsync` 等待前完成，则控制会保留在 `GetUrlContentLengthAsync` 中。如果异步调用过程 `getStringTask` 已完成，并且 `GetUrlContentLengthAsync` 不必等待最终结果，则挂起然后返回到 `GetUrlContentLengthAsync` 将造成成本浪费。***
    > ***在调用方法中，处理模式会继续。 在等待结果前，调用方可以开展不依赖于 `GetUrlContentLengthAsync` 结果的其他工作，否则就需等待片刻。调用方法等待 `GetUrlContentLengthAsync`，而 `GetUrlContentLengthAsync` 等待 `GetStringAsync`。***
    
7. `GetStringAsync` 完成并生成一个字符串结果。 字符串结果不是通过按你预期的方式调用 `GetStringAsync` 所返回的。 （记住，该方法已返回步骤 3 中的一个任务）。相反，字符串结果存储在表示 `getStringTask` 方法完成的任务中。 await 运算符从 `getStringTask` 中检索结果。 赋值语句将检索到的结果赋给 `contents`。
    
8. 当 `GetUrlContentLengthAsync` 具有字符串结果时，该方法可以计算字符串长度。 然后，`GetUrlContentLengthAsync` 工作也将完成，并且等待事件处理程序可继续使用。 在此主题结尾处的完整示例中，可确认事件处理程序检索并打印长度结果的值。 如果你不熟悉异步编程，请花 1 分钟时间考虑同步行为和异步行为之间的差异。 当其工作完成时（第 5 步）会返回一个同步方法，但当其工作挂起时（第 3 步和第 6 步），异步方法会返回一个任务值。 在异步方法最终完成其工作时，任务会标记为已完成，而结果（如果有）将存储在任务中。
