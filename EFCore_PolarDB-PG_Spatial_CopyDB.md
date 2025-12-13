# Create PolarDB for PostgreSQL

See doc https://apsaradb.github.io/PolarDB-for-PostgreSQL/zh/deploying/db-localfs.html#%E5%9F%BA%E4%BA%8E-docker-%E9%95%9C%E5%83%8F%E9%83%A8%E7%BD%B2

```
docker pull registry.cn-hangzhou.aliyuncs.com/polardb_pg/polardb_pg_local_instance:15
```

```
docker run -it --rm --cap-add=SYS_PTRACE --privileged=true --env POLARDB_PORT=5432 --env POLARDB_USER=u1 --env POLARDB_PASSWORD=postgres -v C:\Users\周巍\repos\docker\polardb_pg\data:/var/polardb registry.cn-hangzhou.aliyuncs.com/polardb_pg/polardb_pg_local_instance:15 \ echo 'done'
```

```
docker run -d --cap-add=SYS_PTRACE --privileged=true -p 54320-54322:5432-5434 -v C:\Users\周巍\repos\docker\polardb_pg\data:/var/polardb registry.cn-hangzhou.aliyuncs.com/polardb_pg/polardb_pg_local_instance:15
```

# Reset Password (required)

```
psql
```

```
ALTER USER postgres WITH PASSWORD 'new_password';
```

# Entity Framework Core Code

```
using System.ComponentModel.DataAnnotations.Schema;
using Microsoft.EntityFrameworkCore;
using NetTopologySuite.Geometries;
using Npgsql;

await CreateSourceDatabase();
await CopyToTargetDatabase();

static async Task CreateSourceDatabase()
{
    using var ctx = new PolarPostgreSqlContext("source_db");
    await ctx.Database.EnsureCreatedAsync();

    ctx.Cities.Add(new()
    {
        Name = $"FooCity_{DateTime.UtcNow.ToString("yyyy-MM-dd hh:mm:ss")}",
        Center = new Point(10, 10),
        Road = new LineString(new[] { new Coordinate(10, 10), new Coordinate(20, 20) }),
        Area = new Polygon(new LinearRing(new[]
        {
        new Coordinate(10, 10),
        new Coordinate(20, 10),
        new Coordinate(20, 20),
        new Coordinate(10, 20),
        new Coordinate(10, 10)
    }))
    });
    await ctx.SaveChangesAsync();
}

static async Task CopyToTargetDatabase()
{
    // Step 1: Create target database (if not exists)
    await CreatePostgresDatabaseIfNotExists("target_db");

    // Step 2: Copy data (bulk copy for performance)
    await CopyDataBulk("source_db", "target_db");

    // Helper: Create PostgreSQL database
    async Task CreatePostgresDatabaseIfNotExists(string targetDbName)
    {
        using var ctx = new PolarPostgreSqlContext(targetDbName);
        await ctx.Database.EnsureCreatedAsync();
    }

    // Helper: Bulk copy data
    async Task CopyDataBulk(string sourceDbName, string targetDbName)
    {
        await using var sourceContext = new PolarPostgreSqlContext(sourceDbName);
        await using var targetContext = new PolarPostgreSqlContext(targetDbName);

        // Disable change tracking for performance
        sourceContext.ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking;

        // Copy source tables
        var records = await sourceContext.Cities.ToListAsync();
        targetContext.Cities.AddRange(records);
        await targetContext.SaveChangesAsync();
    }
}

public class PolarPostgreSqlContext : DbContext
{
    public PolarPostgreSqlContext(string dbName)
    {
        ArgumentNullException.ThrowIfNullOrEmpty(dbName);

        this.ConnectionString = BuildConnectionString(dbName);
    }

    public string ConnectionString { get; }

    public DbSet<City> Cities { get; set; }

    public static string BuildConnectionString(string dbName)
    {
        return $"Host=127.0.0.1;Port=54320;Username=postgres;Password=postgres;Database={dbName}";
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder) => optionsBuilder.UseNpgsql(this.ConnectionString, o => o.UseNetTopologySuite());
}

[Table("cities")]
[PrimaryKey(nameof(Id))]
public class City
{
    [Column("id")]
    public int Id { get; set; }

    [Column("name")]
    public string Name { get; set; } = null!;

    [Column("center")]
    public Point Center { get; set; } = null!;

    [Column("road")]
    public LineString Road { get; set; } = null!;

    [Column("area")]
    public Polygon Area { get; set; } = null!;
}
```
