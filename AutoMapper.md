## 什么是AutoMapper
`AutoMapper`是基于对象到对象约定的映射工具，它可以把复杂的对象模型转为`DTO`，或者其他的–那些让设计更合理更适于序列化、通信、传递消息的简单对象或者干脆就只是在领域层与应用层之间搭建一个简单的`ACL`防护层(就像`DTO`一样，用于代码的显示转换)来增加各自层的相互独立性。
## 为什么要有`DTO`
通常我们通过`DAO`获取`PO`，`PO`是和数据库映射的，但是可能包含了很多对于传输来说并不需要的属性。
比如一张表有100个字段，那么对应的`PO`可能就是100个属性，但是对于表示层而言并不需要那么多属性展示给用户，同样的也不应该把底层表结构暴露给表示层，那么中间就有一个`DTO`对象的转换，表示层需要多少属性则`DTO`的设置就定义多少属性。
## 注册
### MapperConfiguration方法注册
```
var config = new MapperConfiguration(cfg => cfg.CreateMap<Order, OrderDto>());
```
### Profile注册
```
public class AutoMapperProfile : Profile
{
    public AutoMapperProfile()
    {
        CreateMap<Order, OrderDto>();
    }
}

var config = new MapperConfiguration(cfg =>
{
    cfg.AddProfile<AutoMapperProfile>();
});

```
### 指定程序集
```
var config = new MapperConfiguration(cfg =>  
{  
    // 扫描当前程序集  
    cfg.AddMaps(System.AppDomain.CurrentDomain.GetAssemblies());  
    // 也可以传程序集名称（dll 名称）   
	cfg.AddMaps("LibCoreTest");  
});
```
## 配置
#### 命名约定
默认情况下，AutoMapper 基于相同的字段名映射，并且是 **不区分大小写** 的。
但有时，我们需要处理一些特殊的情况。
- `SourceMemberNamingConvention` 表示源类型命名规则
- `DestinationMemberNamingConvention` 表示目标类型命名规则
`LowerUnderscoreNamingConvention` 和 `PascalCaseNamingConvention` 是 AutoMapper 提供的两个命名规则。前者命名是小写并包含下划线，后者就是帕斯卡命名规则。
我的理解，如果源类型和目标类型分别采用了 **蛇形命名法** 和 **驼峰命名法**，那么就需要指定命名规则，使其能正确映射。
```
var config = new MapperConfiguration(cfg =>  
{ 
cfg.SourceMemberNamingConvention = new PascalCaseNamingConvention(); 
cfg.DestinationMemberNamingConvention = new LowerUnderscoreNamingConvention();
});
```
### 配置可见性
默认情况下，AutoMapper 仅映射 `public` 成员，但其实它是可以映射到 `private` 属性的。***需要注意private属性必须添加set***
```
var config = new MapperConfiguration(cfg =>  
{  
    cfg.ShouldMapProperty = p => p.GetMethod.IsPublic || p.SetMethod.IsPrivate;  
    cfg.CreateMap<Source, Destination>();  
});
```
## 调用构造函数
有些类，属性的 `set` 方法是私有的。AutoMapper 会自动找到相应的构造函数调用。如果在构造函数中对参数做一些改变的话，其改变会反应在映射结果中。如上例，映射后 `Price` 会乘 2。
```
public class Commodity  
{  
    public string Name { get; set; }  
  
    public int Price { get; set; }  
}  
  
public class CommodityDto  
{  
    public string Name { get; }  
  
    public int Price { get; }  
  
    public CommodityDto(string name, int price)  
    {  
        Name = name;  
        Price = price * 2;  
    }  
}

CommodityDto dto = mapper.Map<CommodityDto>(new Commodity() { Name: '123', Price: 123 });
```
可以禁用掉这个默认行为:  `var config = new MapperConfiguration(cfg => cfg.DisableConstructorMapping());`, 需要目标类要有一个无参构造函数。
## 数组和列表映射
```
public class Source  
{  
    public int Value { get; set; }  
    public int Percent { get; set; }  
}  
  
public class Destination  
{  
    public int Value { get; set; }  
}

var sources = new[]  
{  
    new Source { Value = 5, Percent = 50 },  
    new Source { Value = 6, Percent = 50 },  
    new Source { Value = 7, Percent = 50 }  
};  
  
var iEnumerableDest = mapper.Map<Source[], IEnumerable<Destination>>(sources);  
var iCollectionDest = mapper.Map<Source[], ICollection<Destination>>(sources);  
var iListDest = mapper.Map<Source[], IList<Destination>>(sources);  
var listDest = mapper.Map<Source[], List<Destination>>(sources);  
var arrayDest = mapper.Map<Source[], Destination[]>(sources);
```
支持的源集合类型包括：
- IEnumerable
- IEnumerable
- ICollection
- ICollection
- IList
- IList
- List
- Arrays
## 处理空集合
映射集合属性时，如果源值为 `null`，则 AutoMapper 会将目标字段映射为空集合，而不是 `null`。这与 Entity Framework 和 Framework Design Guidelines 的行为一致，认为 C＃ 引用，数组，List，Collection，Dictionary 和 IEnumerables 永远不应该为 `null`。
## 其他映射
### 方法到属性的映射
AutoMapper 不仅能实现属性到属性映射，还可以实现方法到属性的映射，并且不需要任何配置，方法名可以和属性名一致，也可以带有 `Get` 前缀。
例如下例的 `Employee.GetFullName()` 方法，可以映射到 `EmployeeDto.FullName` 属性。
```
public class Employee  
{  
    public int ID { get; set; }  
  
    public string FirstName { get; set; }  
  
    public string LastName { get; set; }  
  
    public string GetFullName()  
    {  
        return $"{FirstName} {LastName}";  
    }  
}  
  
public class EmployeeDto  
{  
    public int ID { get; set; }  
  
    public string FirstName { get; set; }  
  
    public string LastName { get; set; }  
  
    public string FullName { get; set; }  
}
```
### 自定义映射
```
public class Employee  
{  
    public int ID { get; set; }  
  
    public string Name { get; set; }  
  
    public DateTime JoinTime { get; set; }  
}  
  
public class EmployeeDto  
{  
    public int EmployeeID { get; set; }  
  
    public string EmployeeName { get; set; }  
  
    public int JoinYear { get; set; }  
}

var config = new MapperConfiguration(cfg =>  
{  
    cfg.CreateMap<Employee, EmployeeDto>()  
        .ForMember("EmployeeID", opt => opt.MapFrom(src => src.ID))  
        .ForMember(dest => dest.EmployeeName, opt => opt.MapFrom(src => src.Name))  
        .ForMember(dest => dest.JoinYear, opt => opt.MapFrom(src => src.JoinTime.Year));  
});
```

> - `VO`(`View Object`)：视图对象，用于展示层，它的作用是把某个指定页面(或组件)的所有数据封装起来。
> - `DTO`(`Data Transfer Object`)：数据传输对象，泛指用于展示层与服务层之间的数据传输对象。
> - `DO`(`Domain Object`)：领域对象，就是从现实世界中抽象出来的有形或无形的业务实体。
> - `PO`(`Persistent Object`)：持久化对象，它跟持久层(通常是[关系型数据库](https://cloud.tencent.com/product/cdb-overview?from_column=20065&from=20065))的数据结构形成一一对应的映射关系，如果持久层是关系型[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)，那么，数据表中的每个字段(或若干个)就对应`PO`的一个(或若干个)属性。
> - `DAO`(`Data Access Object`):数据访问对象，主要用来封装对数据库的操作。