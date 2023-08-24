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
![[Pasted image 20230824112646.png]]
可以看到两个互不依赖的方法调用的时候如果同步进行就会显示1再显示2,而异步方法会同步进行.
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
![[Pasted image 20230824114235.png]]
可以看到如果是在await运算符之后的代码会在其运算完毕之后才会执行,也就是说它是将原本异步的方法与同步的方法统一成了同步的执行顺序.