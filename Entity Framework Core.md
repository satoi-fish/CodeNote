Entity Framework Core (EF Core) 是 .NET 平台流行的对象关系映射（ORM）框架。
> 虽然 .NET 平台中 ORM 框架有很多，比如 Dapper、NHibernate、PetaPoco 等，并且 EF Core 的性能也不是最优的（这是由于 EF 的实体跟踪特性，将其禁用后可以大幅提升性能），但依然吸引到很多后端开发者的使用，原因如下：
> 1. EF Core 由 .NET 官方进行开发维护，出现问题解决较为及时，这是很多国产 ORM 框架不具有的优势；
> 2. EF Core 和 C# 语法高度绑定，使用 LINQ 不再需要编写复杂的数据库访问代码；
> 3. EF Core 支持大部分流行的数据库，切换数据库时只需要更改数据库访问驱动，并不需要更改业务逻辑。  

## Code First 与 Database First
Code First 和 Database First 算是 EF 中比较有特色的功能。简单来说 Code First 是先编写 C# 实体类，EF 会根据实体类之间的关系创建数据库；Database First 是先设计和创建数据库，EF 根据数据库的表结构生成 C# 实体类。

## 创建context
```csharp
public class MyContext : DbContext
{
    public MyContext(DbContextOptions<MyContext> options):base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
    }
}


public void ConfigureServices(IServiceCollection services)
{
	// ...
	services.AddDbContext<MyContext>(options =>  
	    options.UseSqlServer(Configuration.GetConnectionString("Default")));
}
        
// 需要在appsettings.json中新增一个ConnectionStrings节点，用于存放连接字符串。
"ConnectionStrings": {
    "Default": "server=(localdb);database=xxx;user=root;password=root;port=3306"
}

// 增加实体
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    public List<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public Blog Blog { get; set; }
}

public class AuditEntry
{
    public int AuditEntryId { get; set; }
    public string Username { get; set; }
    public string Action { get; set; }
}

```

使用EF CORE中添加实体，约束属性和关系，最后将其映射到数据库中的方式有两种，一种是Data Annotations，另一种是Fluent Api，相对而言，Fluent Api提供的功能更多。
### **Data Annotations**
- 在自定义的MyContext中添加以下属性信息，并在每个自定义的实体名称上部增加[Table("XXX")]，其中XXX为开发者指定的表名称。
- 虽然我们目前还没有添加任何约束，但是EF Core会自动地根据`Id/XXId`的命名方式生成自增主键，而且如果没有在实体上增加[Table]Attribute的话，表的命名也是根据属性命名而定

```csharp
//将实体在Context中映射到数据库, 可用[NotMapped]排除实体或属性
public DbSet<Blog> Blogs { get; set; }
public DbSet<Post> Posts { get; set; }
public DbSet<AuditEntry> AuditEntries { get; set; }

//添加Table特性，第一个属性代表数据库表名称
[Table("Blogs")]
public class Blog
{
    [Key]
    public int BlogId { get; set; }
    public string Url { get; set; }

    public List<Post> Posts { get; set; }
}


// 列名称和类型映射
// Column特性可用于属性上，它接收多个参数，其中比较重要的是Name和TypeName，前者表示数据库表映射的列名，后者表示数据类型和格式。假如不指定Url属性上的`[Column(TypeName="varchar(200)")]`，数据库Blog表的Url列默认数据格式将为`varchar(max)`
public class Blog
{
    [Column("blog_id")]
    public int BlogId { get; set; }
    
    [Required]
    [Column(TypeName = "varchar(200)")]
    public string Url { get; set; }
}

// 主键
// 默认情况下，EF CORE会将实体中命名为Id或者[TypeName]Id的属性映射为数据库表中的主键。当然有些开发者不喜欢将主键命名为Id，EF CORE也提供了两种方式进行主键的相关设置。
// Data Annotations方式不支持定义外键名称
class Car
{
    [Key]
    public string LicensePlate { get; set; }
    // public string LicensePlateId { get; set; }

    public string Make { get; set; }
    public string Model { get; set; }
}

// 生成值
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
    // 新增实体信息时自动添加
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
	public DateTime Inserted { get; set; }
	// 新增或更新实体时自动添加
    [DatabaseGenerated(DatabaseGeneratedOption.Computed)]
    public DateTime LastUpdated { get; set; }
}
```

### **Fluent Api**
Fluent Api俗名流式接口，其实就是C#中的扩展接口形式
```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // 增加实体并使用haskey设置主键
    modelBuilder.Entity<Blog>().ToTable("Blogs").HasKey(c => c.BlogId);
    modelBuilder.Entity<Post>().ToTable("Posts").HasKey(c => c.PostId);
    modelBuilder.Entity<AuditEntry>().ToTable("AuditEntries").HasKey(c => c.AuditEntryId);

    // [Ignore] 排除实体和属性
    modelBuilder.Ignore<BlogMetadata>();

    // 列名称和类型映射
    modelBuilder.Entity<Blog>()
        .Property(b => b.BlogId)
        .HasColumnName("blog_id");

    modelBuilder.Entity<Blog>()
        .Property(b => b.Url)
        .HasColumnType("varchar(200)")
        .IsRequired();
    // 由于各种关系型数据库对于数据类型的名称有所区别，所以自定义数据类型时，一定要参阅目标数据库的数据类型定义。
    // 比如PostgreSql支持Json格式，那么就需要添加以下代码
    modelBuilder.Entity().Property(b => b.SomeStringProperty).HasColumnType("jsonb");

    // 复合主键  
    modelBuilder.Entity<Car>().HasKey(c => new { c.State, c.LicensePlate });
    modelBuilder.Entity<Post>().HasOne(p => p.Blog).WithMany(b => b.Posts).HasForeignKey(p => p.BlogId)
        .HasConstraintName("ForeignKey_Post_Blog");

    // 备用键
    // 第一种方法指定Post实体中的BlogUrl属性作为Blog对应Post的外键，指定Blog实体中的Url属性作为备用键（HasPrincipalKey方法
    // 将在下文的唯一标识节中讲解），此时Url将被配置为唯一列，扮演BlogId的作用。
    // 第一种方法
    modelBuilder.Entity<Post>().HasOne(p => p.Blog).WithMany(b => b.Posts).HasForeignKey(p => p.BlogUrl)
        .HasPrincipalKey(b => b.Url);
    // 第二种方法
    modelBuilder.Entity<Car>().HasAlternateKey(c => c.LicensePlate).HasName("AlternateKey_LicensePlate");

    // 生成值
    // 新增实体时自动添加
    modelBuilder.Entity<Blog>().Property(b => b.Inserted).ValueGeneratedOnAdd();
    // 新增或更新实体时添加
    modelBuilder.Entity<Blog>().Property(b => b.LastUpdated).ValueGeneratedOnAddOrUpdate();

    // 默认值
    // 默认值指的是当用户不手动输入时，使用默认值进行数据库相应列的填充
    modelBuilder.Entity<Blog>().Property(b => b.Rating).HasDefaultValue(3);

    // 索引, 为一个或多个属性手动建立索引
    modelBuilder.Entity<Blog>().HasIndex(b => b.Url).HasName("IX_Url");
    modelBuilder.Entity<Person>().HasIndex(p => new { p.FirstName, p.LastName });

    // 唯一标识
    modelBuilder.Entity<Blog>().HasIndex(b => b.Url).IsUnique();
}
```


> 继承通常被用来控制实体类接口如何映射到数据库表结构中。在EF CORE 当前版本中，TPC和TPT暂不被支持，TPH是默认且唯一的继承方式。
> 
> 那么什么是TPH（table-per-hierarchy）呢？顾名思义，一种继承结构全部映射到一张表中，比如Person父类，Student子类和Teacher子类，由EF CORE映射到数据库中时，将会只存在Person类，而Student和Teacher将以列标识的形式出现。
```csharp
class MyContext : DbContext
{
	public DbSet<Blog> Blogs { get; set; }

	protected override void OnModelCreating(ModelBuilder modelBuilder)
	{
		modelBuilder.Entity<Blog>()
			.HasDiscriminator<string>("blog_type")
			.HasValue<Blog>("blog_base")
			.HasValue<RssBlog>("blog_rss");
	}
}

public class Blog
{
	public int BlogId { get; set; }
	public string Url { get; set; }
}

public class RssBlog : Blog
{
	public string RssUrl { get; set; }
}
```
> 观察OnModelCreating方法，HasDiscriminator提供修改标识列名的功能，HasValue提供新增或修改实体时，根据实体类型将不同的标识自动写入标识列中。如新增Blog时，blog_type列将写入blog_base字符串，新增RssBlog时，blog_type列将写入blog_rss字符串。

以下内容用代码的方式给出了一对一，一对多和多对多的关系
> 唯一需要注意的是，关系设置请从子端（如User和Blog呈一对多对应时，从Blog开始）开始，否则配置不慎容易出现多个外键的情况。
```csharp
public class MyContext : DbContext
{
	public MyContext(DbContextOptions<MyContext> options) : base(options)
	{
	}

	public DbSet<User> Users { get; set; }
	public DbSet<UserAccount> UserAccounts { get; set; }
	public DbSet<Blog> Blogs { get; set; }
	public DbSet<Post> Posts { get; set; }
	public DbSet<PostTag> PostTags { get; set; }
	public DbSet<Tag> Tags { get; set; }

	protected override void OnModelCreating(ModelBuilder modelBuilder)
	{
		// Blog与Post之间为 1 - N 关系
		modelBuilder.Entity<Post>()
			.HasOne(p => p.Blog)
			.WithMany(b => b.Posts)
			.HasForeignKey(p => p.BlogId)
			// 使用HasConstraintName方法配置外键名称
			.HasConstraintName("ForeignKey_Post_Blog")
			.IsRequired();

		// User与UserAccount之间为 1 - 1 关系
		modelBuilder.Entity<UserAccount>()
			.HasKey(c => c.UserAccountId);

		modelBuilder.Entity<UserAccount>()
			.HasOne(c => c.User)
			.WithOne(c => c.UserAccount)
			.HasForeignKey<UserAccount>(c => c.UserAccountId)
			.IsRequired();

		// User与Blog之间为 1 - N 关系
		modelBuilder.Entity<Blog>()
			.HasOne(b => b.User)
			.WithMany(c => c.Blogs)
			.HasForeignKey(c => c.UserId)
			.IsRequired();

		// Post与Tag之间为 N - N 关系
		modelBuilder.Entity<PostTag>()
			.HasKey(t => new {t.PostId, t.TagId});

		modelBuilder.Entity<PostTag>()
			.HasOne(pt => pt.Post)
			.WithMany(p => p.PostTags)
			.HasForeignKey(pt => pt.PostId)
			.IsRequired();

		modelBuilder.Entity<PostTag>()
			.HasOne(pt => pt.Tag)
			.WithMany(t => t.PostTags)
			.HasForeignKey(pt => pt.TagId)
			.IsRequired();
	}
}

public class Blog
{
	public int BlogId { get; set; }
	public string Url { get; set; }
	public List<Post> Posts { get; set; }
	public int UserId { get; set; }
	public User User { get; set; }
}

public class User
{
	public int UserId { get; set; }
	public string UserName { get; set; }
	public List<Blog> Blogs { get; set; }
	public UserAccount UserAccount { get; set; }
}

public class UserAccount
{
	public int UserAccountId { get; set; }
	public string UserAccountName { get; set; }
	public bool IsValid { get; set; }
	public User User { get; set; }
}

public class Post
{
	public int PostId { get; set; }
	public string Title { get; set; }
	public string Content { get; set; }
	public int BlogId { get; set; }
	public Blog Blog { get; set; }
	public List<PostTag> PostTags { get; set; }
}

public class Tag
{
	public string TagId { get; set; }
	public List<PostTag> PostTags { get; set; }
}

public class PostTag
{
	public int PostId { get; set; }
	public Post Post { get; set; }
	public string TagId { get; set; }
	public Tag Tag { get; set; }
}

```

> EF CORE只支持部分类型的自动操作，详见[Default Values](https://docs.microsoft.com/zh-cn/ef/core/modeling/relational/default-values)

## 更改跟踪
每个 [DbContext](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.dbcontext) 实例跟踪对实体所做的更改。 在调用 [SaveChanges](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.dbcontext.savechanges) 时，这些跟踪的实体会相应地驱动对数据库的更改。
**跟踪实体** : 
- 从针对数据库执行的查询返回
- 通过 `Add`、`Attach`、`Update` 或类似方法显示附加到 DbContext
- 检测为连接到现有跟踪实体的新实体

**不被跟踪的实体实例** : 
- DbContext 已释放
- 清除更改跟踪器
- 显式拆离实体

根据上述而言, DbContext 的生存期应为：
1. 创建 DbContext 实例
2. 跟踪某些实体
3. 对实体进行一些更改
4. 调用 SaveChanges 以更新数据库
5. 释放 DbContext 实例