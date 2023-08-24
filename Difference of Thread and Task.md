## Thread 和 Task的区别
1. **抽象级别**：
	- `Thread` 是 .NET 中的基本线程抽象，它直接映射到操作系统的底层线程。你可以通过创建和管理 Thread 实例来实现多线程编程。但是，使用 Thread 时需要手动处理线程的生命周期、同步、资源管理等问题。
	- `Task` 是 .NET 中更高级别的线程抽象，它是对 Thread 的封装和扩展。Task 是基于线程池的概念，它将工作项提交给线程池管理，允许你以异步和并行的方式执行代码。Task 提供了更高级别的控制、错误处理和取消机制。
2. **线程池使用**：
	 - `Thread` 不涉及线程池，每个 Thread 实例都会创建一个新的操作系统线程。创建和销毁线程会有较高的开销，因此过多的 Thread 实例可能会导致性能问题。
	 - `Task` 基于线程池，通过将任务提交到线程池，可以更有效地管理线程资源。线程池根据可用的线程数量来管理任务的执行，避免了频繁地创建和销毁线程。
	 - `Task` 默认使用后台线程执行，`Thread` 默认使用前台线程
3. **并行和异步**：
	- `Thread` 允许你以并行的方式执行多个任务，但需要手动同步和协调线程之间的工作。
	- `Task` 支持异步编程模型，通过 async/await 关键字可以轻松地实现异步操作，而无需显式地管理线程。Task 还可以以并行的方式执行多个任务，但它提供了更高级别的控制。
4. **异常处理**：
	- `Thread` 需要手动处理线程中的异常，否则可能会导致应用程序崩溃。
	- `Task` 提供了更好的异常处理机制，异常会被捕获并封装在 Task 对象中，可以通过调用 `await` 或 `Task.Wait` 等方法来检查和处理异常。
5. **使用特征**:
	- `Task` 可取消任务执行，`Thread` 不行
		```
		static void Main(string[] args)  
		{  
			using (var cts = new CancellationTokenSource())  
			{  
				Task task = new Task(() => { LongRunningTask(cts.Token); });  
				task.Start();  
				Console.WriteLine("Operation Performing...");  
				if(Console.ReadKey().Key == ConsoleKey.C)  
				{  
					Console.WriteLine("Cancelling..");  
					cts.Cancel();  
				}                  
				Console.Read();  
			}  
		}  
		private static void LongRunningTask(CancellationToken token)  
		{  
			for (int i = 0; i < 10000000; i++)  
			{  
				if(token.IsCancellationRequested)  
				{  
					break;  
				}  
				else  
				{                    
					Console.WriteLine(i);  
				}                 
			}            
		}
		```
	- `Task`可以执行后续操作, `Thread`不行
		```
		static void Main(string[] args)
		{
		    Task task = new Task(LongRunningTask);
		    task.Start();
		    Task childTask = task.ContinueWith(SquareOfNumber);
		    Console.WriteLine("Sqaure of number is :"+ childTask.Result);
		    Console.WriteLine("The number is :" + task.Result);
		}
		private static int LongRunningTask()
		{
		    Thread.Sleep(3000);
		    return 2;
		}
		private static int SquareOfNumber(Task obj)
		{
		    return obj.Result * obj.Result;
		}
		```
## 实践代码对比
- Task是.NET4.0加入的，跟线程池ThreadPool的功能类似，用Task开启新任务时，会从线程池中调用线程，而Thread每次实例化都会创建一个新的线程。
	```
	static void Main(string[] args)
	{
	    for (int i = 0; i < 10; i++)
	    {
	        new Thread(Run1).Start();
	    }
	    for (int i = 0; i < 10; i++)
	    {
	        Task.Run(() => { Run2(); });
	    }
	}
	static void Run1()
	{
	    Console.WriteLine("Thread Id =" + Thread.CurrentThread.ManagedThreadId);
	}
	static void Run2()
	{
	    Console.WriteLine("Task调用的Thread Id =" + Thread.CurrentThread.ManagedThreadId);
	}  
	// Thread Id =7  
	// Thread Id =6  
	// Thread Id =5  
	// Thread Id =8  
	// Thread Id =9  
	// Thread Id =10  
	// Thread Id =11  
	// Thread Id =12  
	// Thread Id =13  
	// Thread Id =14  
	// Task调用的Thread Id =21  
	// Task调用的Thread Id =15  
	// Task调用的Thread Id =17  
	// Task调用的Thread Id =18  
	// Task调用的Thread Id =19  
	// Task调用的Thread Id =20  
	// Task调用的Thread Id =21  
	// Task调用的Thread Id =17  
	// Task调用的Thread Id =20  
	// Task调用的Thread Id =22
	// Task 只用到了 15、17、18、19、20、21、22,7个线程
	// 可以看出来，直接用Thread会开启10个线程，用Task(用了线程池)只开启了7个！
	// Task的背后的实现也是使用了线程池线程，但它的性能优于ThreadPoll,因为它使用的不是线程池的全局队列，而是使用的本地队列,当前面队列任务完成并空出线程后会把未完成的任务放入已开启的线程中运行
	```