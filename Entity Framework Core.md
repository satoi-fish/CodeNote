Entity Framework Core (EF Core) 是 .NET 平台流行的对象关系映射（ORM）框架。
> 虽然 .NET 平台中 ORM 框架有很多，比如 Dapper、NHibernate、PetaPoco 等，并且 EF Core 的性能也不是最优的（这是由于 EF 的实体跟踪特性，将其禁用后可以大幅提升性能），但依然吸引到很多后端开发者的使用，原因如下：
> 1. EF Core 由 .NET 官方进行开发维护，出现问题解决较为及时，这是很多国产 ORM 框架不具有的优势；
> 2. EF Core 和 C# 语法高度绑定，使用 LINQ 不再需要编写复杂的数据库访问代码；
> 3. EF Core 支持大部分流行的数据库，切换数据库时只需要更改数据库访问驱动，并不需要更改业务逻辑。  

