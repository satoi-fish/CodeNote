### 反射(Reflection)

反射是.NET 中的重要机制，通过反射可以得到程序集内部的接口、类、方法、字段、属性、特性等信息，还可以动态创建出类型实例并执行其中的方法。

#### 通过反射获取类型

```C#
// 使用typeof
System.Type animalType = typeof(Animal);

// 使用GetType()
Animal animal = new Animal();
System.Type animalType = animal.GetType();

```

#### 这个 Type 类属性可以获取许多信息,诸如 name、fullname 等

```C#
//获取类型
System.Type animalType = typeof(Animal);
//类型名
Console.WriteLine("Name:" + animalType.Name);
//类型全名
Console.WriteLine("FullName:" + animalType.FullName);
//程序集名称
Console.WriteLine("Assembly:" + animalType.Assembly);
//程序集全名
Console.WriteLine("Ass-Name:" + animalType.AssemblyQualifiedName);
//获取父类
Console.WriteLine("BaseType:" + animalType.BaseType.BaseType);
```

#### Type 类的方法也可以获取信息类似 GetMember()、GetEvent()等

```C#
// 该类的所有成员
Console.WriteLine("BaseType:" + animalType.GetMembers());
// 该类的事件
Console.WriteLine("BaseType:" + animalType.GetEvents());
// 该类实现的接口
Console.WriteLine("BaseType:" + animalType.GetInterfaces());
// 该类的字段（成员变量）
Console.WriteLine("BaseType:" + animalType.GetFields());
// 该类的属性
Console.WriteLine("BaseType:" + animalType.GetProperties());
// 该类的方法
Console.WriteLine("BaseType:" + animalType.GetMethods());
```

其余还有 BindingFlags 按照权限获取类型、Activator 反射几种不同的构造实例化对象.

### C#命名规范

1. 在类的顶部声明所有的成员变量，静态变量声明在最前面
2. 类名和属性、集合命名为名词及名词短语,都使用 Pascal 规则,集合后面加“Collection”
3. 接口命名用 I 为前缀,后续使用 Pascal 规则
4. 派生类名称的第二个部分应当是基类的名称
5. 枚举的类型和值名称使用 Pascal 大小写
6. 变量使用 Camel 命名规则
7. 对于 bool 型属性或者变量使用 Is（is）作为前缀
8. 事件命名使用 Pascal 规则,多使用动词命名, 微软官方建议:事件参数添加后缀"EventArgs", 习惯命名:事件以 On 为前缀(比如 OnClick)

> > 针对官方的示例代码,书写习惯,智能提示,代码补全和约定俗成的 C#规范,建议 private 采用下划线+Camel 方式命名,非 Private 字段采用 Pascal 方式命名.

> > 参考微软官方命名规则,C#高级编程,Resharp 命名规范,.Net 源码命名规则,建议字段命名方式如下:

> > - public/protected/internal 以 Pascal 方式命名
> > - private 以下划线+Camel 方式命名
> >
> > ```csharp
> >        public readonly int Field1;
> >        public const int Field2 = 0;
> >
> >        internal readonly int Field3;
> >        internal const int Field4 = 0;
> >
> >        private int _field5;
> >        private static int _field6;
> >        private readonly int _field7;
> >        private const int _field8 = 0;
> > ```
> >
> > - ~~以 m\_为前缀的方式不建议(C++命名方式)~~
> > - ~~以类型为前缀的方式不建议(比如 bool 类型的字段以 b 为前缀,枚举以 e 为前缀)~~
> > - ~~以类型为后缀的方式不建议(比如单位列表 unitList,直接取名为 units)~~

| 类型            | 命名方式 |                                      示例 |
| --------------- | :------: | ----------------------------------------: |
| Namespace       |  Pascal  |       `namespace System.Security { ... }` |
| Type            |  Pascal  |       `public class StreamReader { ... }` |
| Interface       |  Pascal  |    `public interface IEnumerable { ... }` |
| Method          |  Pascal  |       `public virtual string ToString();` |
| Property        |  Pascal  |              `public int Length { get; }` |
| Delegate        |  Pascal  |    `public delegate void EvnetHandler();` |
| Event           |  Pascal  |       `public event EventHandler Exited;` |
| Public Field    |  Pascal  |            `public readonly int Min = 0;` |
| Protected Field |  Pascal  |                  `protected int Min = 0;` |
| private Field   |  Camel   |                   `private int _min = 0;` |
| Enum            |  Pascal  |                    `public enum FileMode` |
| Parameter       |  Camel   | `public static int ToInt32(string value)` |
| Local Variable  |  Camel   |         `void Method(){int number = 10;}` |
