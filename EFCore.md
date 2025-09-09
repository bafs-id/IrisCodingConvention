# EF Core – Library Template

## 📦 NuGet

- **Core packages (choose provider):**

  * `Microsoft.EntityFrameworkCore`
  * `Microsoft.EntityFrameworkCore.Design` *(migrations tooling)*
  * `Microsoft.EntityFrameworkCore.Tools` *(CLI tooling)*
  * `Microsoft.EntityFrameworkCore.SqlServer`
- **Docs:** [https://learn.microsoft.com/ef/core/](https://learn.microsoft.com/ef/core/)

---

## 📖 Description

Entity Framework Core (EF Core) is a modern ORM for .NET. It maps your C# classes to database tables, provides LINQ queries, change tracking, and migrations for schema evolution.

---

## ⚙️ Configuration

**`appsettings.json`**

```json
{
  "ConnectionStrings": {
    "AppDbConnection": "Server=localhost;Database=AppDb;User Id=sa;Password=Your_password123;TrustServerCertificate=True;"
  }
}
```

---

## 💻 Example Usage (C#)
This example demonstrates how to implement a repository pattern using Entity Framework Core for managing configuration data (MsConfiguration) that is linked with MsBranch and MsItpAgency.
It includes

### 🧱 Entity Models

**`MsConfiguration.cs`**
```csharp
namespace IRIS.MobileBackend.Models;

public partial class MsConfiguration
{
    public int ConfigurationId { get; set; }

    public string ConfigurationKey { get; set; } = null!;

    public string ConfigurationValue { get; set; } = null!;

    public string? ConfigurationDesc { get; set; }

    public int BranchId { get; set; }

    public int ItpAgencyId { get; set; }

    public int EnableStatus { get; set; }

    public Guid? CreatedBy { get; set; }

    public DateTimeOffset? CreatedDate { get; set; }

    public Guid? UpdatedBy { get; set; }

    public DateTimeOffset? UpdatedDate { get; set; }

    public virtual MsBranch Branch { get; set; } = null!;

    public virtual MsItpAgency ItpAgency { get; set; } = null!;
}
```

**`BaseEntity.cs`**
```csharp
namespace IRIS.MobileBackend.Models
{
    public abstract class BaseEntity
    {
        public Guid? CreatedBy { get; set; }
        public DateTimeOffset? CreatedDate { get; set; }
        public Guid? UpdatedBy { get; set; }
        public DateTimeOffset? UpdatedDate { get; set; }
    }
}
```

**`MsBranch.cs`**
```csharp
namespace IRIS.MobileBackend.Models;

public partial class MsBranch : BaseEntity
{
    public int BranchId { get; set; }

    public string BranchCode { get; set; } = null!;

    public string BranchName { get; set; } = null!;

    public string? BranchDesc { get; set; }

    public DateOnly EffectiveStart { get; set; }

    public DateOnly EffectiveEnd { get; set; }

    public int EnableStatus { get; set; }

    public virtual ICollection<MsConfiguration> MsConfigurations { get; set; } = new List<MsConfiguration>();
}
```

**`MsItpAgency.cs`**
```csharp
namespace IRIS.MobileBackend.Models;

public partial class MsItpAgency : BaseEntity
{
    public int ItpAgencyId { get; set; }

    public string ItpAgencyRefCode { get; set; } = null!;

    public string ItpAgencyCode { get; set; } = null!;

    public string ItpAgencyName { get; set; } = null!;

    public string? ItpAgencyDesc { get; set; }

    public int EnableStatus { get; set; }

    public DateOnly EffectiveStart { get; set; }

    public DateOnly EffectiveEnd { get; set; }

    public virtual ICollection<MsConfiguration> MsConfigurations { get; set; } = new List<MsConfiguration>();
}
```

---

## 🗃️ DbContext

**`IrisDbContext.cs`**
```csharp
using Microsoft.EntityFrameworkCore;
using YourApp.Domain.Entities;

namespace YourApp.Infrastructure.Data;

public sealed class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
{
    public virtual DbSet<MsBranch> MsBranches { get; set; }

    public virtual DbSet<MsItpAgency> MsItpAgencies { get; set; }

    public virtual DbSet<MsConfiguration> MsConfigurations { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.UseCollation("Thai_CI_AS");

        modelBuilder.Entity<MsBranch>(entity =>
        {
            entity.HasKey(e => e.BranchId).HasName("PK_BRANCH_MS");

            entity.ToTable("MSBranch", "bom");

            entity.Property(e => e.BranchId).HasColumnName("BranchID");
            entity.Property(e => e.BranchCode).HasMaxLength(20);
            entity.Property(e => e.BranchDesc).HasMaxLength(200);
            entity.Property(e => e.BranchName).HasMaxLength(100);
        });

        modelBuilder.Entity<MsItpAgency>(entity =>
        {
            entity.HasKey(e => e.ItpAgencyId).HasName("PK_AGENCY_MS");

            entity.ToTable("MSITPAgency", "bom");

            entity.Property(e => e.ItpAgencyId).HasColumnName("ITPAgencyID");
            entity.Property(e => e.ItpAgencyCode)
                .HasMaxLength(20)
                .HasColumnName("ITPAgencyCode");
            entity.Property(e => e.ItpAgencyDesc)
                .HasMaxLength(200)
                .HasColumnName("ITPAgencyDesc");
            entity.Property(e => e.ItpAgencyName)
                .HasMaxLength(100)
                .HasColumnName("ITPAgencyName");
            entity.Property(e => e.ItpAgencyRefCode)
                .HasMaxLength(20)
                .HasColumnName("ITPAgencyRefCode");
        });

        modelBuilder.Entity<MsConfiguration>(entity =>
        {
            entity.HasKey(e => e.ConfigurationId);

            entity.ToTable("MSConfiguration", "mobile");

            entity.Property(e => e.ConfigurationId).HasColumnName("ConfigurationID");
            entity.Property(e => e.BranchId).HasColumnName("BranchID");
            entity.Property(e => e.ConfigurationKey).HasMaxLength(255);
            entity.Property(e => e.ConfigurationValue).HasMaxLength(255);
            entity.Property(e => e.ItpAgencyId).HasColumnName("ItpAgencyID");

            entity.HasOne(d => d.Branch).WithMany(p => p.MsConfigurations)
                .HasForeignKey(d => d.BranchId);

            entity.HasOne(d => d.ItpAgency).WithMany(p => p.MsConfigurations)
                .HasForeignKey(d => d.ItpAgencyId);
        });
    }
}
```

---

### 🔌 Interface & Implementation

- #### Repository Abstraction

**`IConfigurationRepository.cs`**
```csharp
using IRIS.MobileBackend.Models;

namespace IRIS.MobileBackend.Abstractions
{
    public interface IConfigurationRepository
    {
        ValueTask<List<MsConfiguration>> GetConfigurationsAsync(int agencyId, int branchId);
    }
}
```

- #### Implementation

**`ConfigurationRepository.cs`**
```csharp
using IRIS.MobileBackend.Abstractions;
using IRIS.MobileBackend.Helpers;
using IRIS.MobileBackend.Models;
using IRIS.MobileBackend.Models.AppModels;
using Microsoft.EntityFrameworkCore;

namespace IRIS.MobileBackend.Repositories.Configuration
{
    public class ConfigurationRepository(IrisDbContext irisDbContext) : IConfigurationRepository
    {
        private readonly int _enabledStatusId = 1;

        public async ValueTask<List<MsConfiguration>> GetConfigurationsAsync(
            int agencyId,
            int branchId
        )
        {
            return await irisDbContext.MsConfigurations.Where(
                config => config.ItpAgencyId == agencyId
                && config.BranchId == branchId
                && config.EnableStatus == _enabledStatusId
            ).ToListAsync();
        }
    }
}
```

---

### 🧩 DI Registration

**`Program.cs`**
```csharp
using IRIS.MobileBackend.Models;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);
string appDbConnection = builder.Configuration.GetConnectionString("AppDbConnection") ?? "";
builder.Services.AddDbContext<IrisDbContext>(options =>
{
    options.UseSqlServer(appDbConnection);
});

//--- Register application services, and custom dependencies ---

var app = builder.Build();

//--- Configure middleware pipeline and endpoints ---

app.Run();
```

---