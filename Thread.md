## 线程
### 进程和线程
进程是一种正在执行的程序。 操作系统使用进程来分隔正在执行的应用程序。 线程是操作系统向其分配处理器时间的基本单元。 每个线程具有计划优先级并维护系统用于保存线程执行暂停时线程上下文的一组结构。 线程上下文包含线程顺畅继续执行所需的全部信息，包括线程的一组 CPU 寄存器和堆栈。 多个线程可在进程上下文中运行。 进程的所有线程共享其虚拟地址空间。 线程可执行任意部分的程序代码，包括其他线程正在执行的部分。
默认情况下，.NET 程序由单个线程(主线程)启动。 但它可以创建其他线程(工作线程)，以与主线程并行或同时执行代码。
### 线程生命周期中的各种状态
- **未启动状态**：当线程实例被创建但 Start 方法未被调用时的状况。
- **就绪状态**：当线程准备好运行并等待 CPU 周期时的状况。
- **不可运行状态**：下面的几种情况下线程是不可运行的：
    - 已经调用 Sleep 方法
    - 已经调用 Wait 方法
    - 通过 I/O 操作阻塞
- **死亡状态**：当线程已完成执行或已中止时的状况。
### 何时使用多个线程
一般是在程序执行可并行执行的操作的时候使用多个工作线程执行耗时的操作,此外还可以使用专用线程,提高网络或通信设备通信对传入信息的响应能力.
### 使用多线程
`System.Threading.ThreadPool` 类为应用程序提供一个受系统管理的辅助线程池，从而使开发能够专注于应用程序任务，而非线程管理。
如果有需要后台处理的短任务，托管的线程池则为利用多个线程的简便方法。 在Framework 4 及更高版本中使用线程池容易得多，因为可以创建在线程池线程上执行异步任务的`Task`和 `Task<TResult>`对象。
#### Task的基本使用
##### new Task
```
// 无参数  
var task = new Task(() =>  
{  
    Console.WriteLine($"Current ThreadId={Environment.CurrentManagedThreadId}");  
});  
task.Start(); 

// 有参数  
var task = new Task((obj) =>  
{  
    Console.WriteLine($"Current ThreadId={Environment.CurrentManagedThreadId}, Current Content={obj}");  
}, "Hello World"); 

task.Start();
```
##### Task.Factory.StartNew
```
// 无参数
var task = Task.Factory.StartNew(() =>
{
    Console.WriteLine($"Current ThreadId={Environment.CurrentManagedThreadId}");
});
// 有参
var task = Task.Factory.StartNew((obj) =>
{
    Console.WriteLine($"Current ThreadId={Environment.CurrentManagedThreadId}, Current Content={obj}");
}, "Hello World");
```
##### Task.Run
```
// 无参数
var task = Task.Run(() =>
{
    Console.WriteLine($"Current ThreadId={Environment.CurrentManagedThreadId}");
});
// 有参
var task = Task.Run((obj) =>
{
    Console.WriteLine($"Current ThreadId={Environment.CurrentManagedThreadId}, Current Content={obj}");
}, "Hello World");
```
#### 线程池特征
线程池线程是后台线程.每个线程均使用默认的堆栈大小位于多线程单元中.它是一旦有线程完成任务, 线程会返回到等待线程队列中, 这时便可以重复利用这个线程,达到无需为每个新任务创建新线程.
想要使用线程池, 最简单的就是使用任务并行库(TPL), 任务并行库 (TPL) 是 `System.Threading`和 `System.Threading.Tasks`空间中的一组公共类型和 API。
#### 何时不使用线程池线程
有几种应用场景，其中适合创建并管理自己的线程，而非使用线程池线程：
- 需要一个前台线程。
- 需要具有特定优先级的线程。
- 拥有会导致线程长时间阻塞的任务。 线程池具有最大线程数，因此大量被阻塞的线程池线程可能会阻止任务启动。
- 需将线程放入单线程单元。 所有 [ThreadPool](https://learn.microsoft.com/zh-cn/dotnet/api/system.threading.threadpool) 线程均位于多线程单元中。
- 需具有与线程关联的稳定标识，或需将一个线程专用于一项任务。

> 多个线程可能需要访问共享的资源。 要使资源保持未损坏的状态并避免争用条件，必须同步对资源的线程访问.
   请务必处理线程异常。 线程中未经处理的异常通常会终止进程。
