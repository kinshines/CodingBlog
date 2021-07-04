# Entity Framework 配置关系（1对1，1对0）

### 实体之间的关系对应数据库表的关系，有1:0，1:1，1:N，N:N这几种。这里介绍1:0、1：1这两种配置关系。

举例说明：Employee表示员工，Account表示通讯账号。有些员工使用通讯账号，但是有些员工不使用这些通讯账号。从员工的角度来观察，员工Employee与通讯账号Account之间的关系就是一个员工对应一个通讯账号（1:1），或者一个员工没有通讯账号（1:0）。从通讯账号Account的角度来观察，一个通讯账号对应一个员工（1:1）。



1.实体类：Employee

```c#
public class Employee
{
    public int Id { get; set; }

    public string Name { get; set; }

    public virtual Account Account { get; set; }
}
```

2.实体类：Account

```c#
public class Account
{
    public int Id { get; set; }

    public string UserName { get; set; }

    public string Password { get; set; }

    public virtual Employee Employee { get; set; }
}
```

3.映射Employee

```c#
public class EmployeeMap : EntityTypeConfiguration<Employee>
{
    public EmployeeMap()
    {
        this.ToTable("Employee");

        this.HasKey(p => p.Id);

        this.Property(p => p.Id).IsRequired().HasDatabaseGeneratedOption(DatabaseGeneratedOption.Identity);
        this.Property(p => p.Name).IsRequired().HasMaxLength(63);
    }
}
```

4.映射Account

```c#
public class AccountMap : EntityTypeConfiguration<Account>
{
    public AccountMap()
    {
        this.ToTable("Account");

        this.HasKey(p => p.Id);

        this.Property(p => p.Id).IsRequired().HasDatabaseGeneratedOption(DatabaseGeneratedOption.Identity);
        this.Property(p => p.UserName).IsRequired().HasMaxLength(63);
        this.Property(p => p.Password).IsRequired().HasMaxLength(63);

        this.HasRequired(m => m.Employee)
            .WithOptional(m => m.Account)
            .Map(m => m.MapKey("AccountId"))
            .WillCascadeOnDelete(true);
    }
}
```

5.DbContext

```c#
public class EntFraContext : DbContext
{
    public IDbSet<Account> AccountSet { get; set; }

    public IDbSet<Employee> EmployeeSet { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Configurations.Add(new AccountMap());
        modelBuilder.Configurations.Add(new EmployeeMap());

        base.OnModelCreating(modelBuilder);
    }
}
```

