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
