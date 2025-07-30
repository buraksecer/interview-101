# .NET Core Soru & Cevaplar

---

### 1. `.NET Core'da MemoryCache ve DistributedCache farkı nedir?`
**Cevap:**
- MemoryCache, uygulama belleğinde çalışır, instance'a özeldir. Uygulama yeniden başlatılırsa cache silinir ve birden fazla instance arasında paylaşılmaz.
- DistributedCache, Redis gibi harici bir kaynağa yazar, çoklu instance'larda paylaşılır. Yüksek erişilebilirlik ve ölçeklenebilirlik sağlar.

---

### 2. `Asenkron programlama .NET'te nasıl yapılır?`
**Cevap:**
- `async` ve `await` anahtar kelimeleri ile yapılır.
- `Task` ve `Task<T>` tipleri kullanılır.
- I/O-bound işlemlerde thread bloklanmaz, performans artar.

**Örnek:**
```csharp
public async Task<string> GetDataAsync()
{
    using (var httpClient = new HttpClient())
    {
        string result = await httpClient.GetStringAsync("https://api.example.com/data");
        return result;
    }
}
```

---

### 3. `Singleton, Scoped, Transient servis ömürleri nedir? Ne zaman hangisi kullanılır?`
**Cevap:**
- **Singleton**: Uygulama boyunca tek bir instance oluşturulur. Tüm isteklerde aynı nesne kullanılır. (Örn: cache, logger)
- **Scoped**: Her HTTP isteği için bir instance oluşturulur. Aynı istek boyunca aynı nesne kullanılır. (Örn: DbContext)
- **Transient**: Her talepte yeni bir instance oluşturulur. (Örn: hafif, durumsuz servisler)

**Ne zaman hangisi?**
- Singleton: Paylaşılan, thread-safe ve state tutmayan servislerde.
- Scoped: Request bazlı state tutan servislerde.
- Transient: Kısa ömürlü, bağımsız nesnelerde.

**Kullanım Örnekleri:**

**Singleton Kullanım Alanları:**
- Configuration servisleri (appsettings okuma)
- Logging servisleri (ILogger)
- Cache servisleri (MemoryCache)
- Database connection pool'ları
- Email servisleri (SMTP client)
- Background job scheduler'ları
- API client'ları (HttpClient)

**Scoped Kullanım Alanları:**
- Entity Framework DbContext
- User session bilgileri
- Request-specific cache
- Transaction scope'ları
- User authentication context
- Request tracking/logging

**Transient Kullanım Alanları:**
- Validation servisleri
- Mapping servisleri (AutoMapper)
- Utility sınıfları
- Business logic servisleri
- Data transfer object'leri (DTO)
- Helper sınıfları

---

## 🗄️ Entity Framework & ORM

### 4. `Entity Framework nedir ve avantajları nelerdir?`
**Cevap:**
Entity Framework (EF), Microsoft'un Object-Relational Mapping (ORM) framework'üdür. SQL veritabanları ile .NET nesneleri arasında köprü görevi görür.

**Avantajlar:**
- **Code-first Development**: C# sınıflarından veritabanı tabloları oluşturma
- **LINQ Desteği**: Tip güvenli sorgular
- **Change Tracking**: Otomatik değişiklik takibi
- **Lazy Loading**: İhtiyaç duyulduğunda veri yükleme
- **Migration**: Veritabanı şema değişikliklerini yönetme
- **Cross-platform**: .NET Core ile platform bağımsızlığı

**Örnek:**
```csharp
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public List<Order> Orders { get; set; }
}

public class ApplicationDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    public DbSet<Order> Orders { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("connectionString");
    }
}

// Kullanım
using var context = new ApplicationDbContext();
var users = await context.Users
    .Where(u => u.Name.Contains("John"))
    .ToListAsync();
```

---

### 5. `Code First vs Database First yaklaşımları nedir?`
**Cevap:**

**Code First:**
- C# sınıflarından başlayarak veritabanı oluşturma
- Model sınıfları veritabanı şemasını belirler
- Migration ile veritabanı güncellenir

**Database First:**
- Mevcut veritabanından model sınıfları oluşturma
- Veritabanı şeması model sınıflarını belirler
- Scaffold-DbContext komutu ile kod üretimi

---

### 6. `Migration nedir ve nasıl çalışır?`
**Cevap:**
Migration, veritabanı şemasındaki değişiklikleri versiyonlu bir şekilde yönetme sistemidir.

---

### 7. `Lazy Loading vs Eager Loading vs Explicit Loading nedir?`
**Cevap:**

**Lazy Loading (Tembel Yükleme):**
- İlişkili veriler sadece erişildiğinde yüklenir
- Proxy kullanır, virtual property gerekir
- N+1 query problemine neden olabilir

**Eager Loading (Hevesli Yükleme):**
- İlişkili veriler ana sorguyla birlikte yüklenir
- Include() metodu kullanılır
- Tek sorguda tüm veriler gelir

**Explicit Loading (Açık Yükleme):**
- İlişkili veriler manuel olarak yüklenir
- Load() metodu kullanılır
- Kontrollü veri yükleme

**Örnekler:**
```csharp
// Lazy Loading
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual List<Order> Orders { get; set; } // virtual gerekli
}

// Kullanım
var user = await context.Users.FirstAsync();
var orders = user.Orders; // Bu satırda SQL çalışır

// Eager Loading
var usersWithOrders = await context.Users
    .Include(u => u.Orders)
    .ThenInclude(o => o.OrderItems)
    .ToListAsync();

// Explicit Loading
var user = await context.Users.FirstAsync();
await context.Entry(user)
    .Collection(u => u.Orders)
    .LoadAsync();

// Select Loading (Projection)
var userNames = await context.Users
    .Select(u => new { u.Name, OrderCount = u.Orders.Count() })
    .ToListAsync();
```

---

### 8. `N+1 Query Problem nedir ve nasıl çözülür?`
**Cevap:**
N+1 Query Problem, ana listeyi getiren 1 sorgu + her item için 1 sorgu (N) olmak üzere toplam N+1 sorgu çalışması problemidir.

**Problem Örneği:**
```csharp
// Bu kod N+1 query problem yaratır
var users = await context.Users.ToListAsync(); // 1 query
foreach (var user in users)
{
    Console.WriteLine($"{user.Name} has {user.Orders.Count()} orders"); // Her user için 1 query
}
```

**Çözüm Yöntemleri:**

**1. Eager Loading:**
```csharp
var users = await context.Users
    .Include(u => u.Orders)
    .ToListAsync(); // Tek query ile tüm veriler
```

**2. Select/Projection:**
```csharp
var userSummaries = await context.Users
    .Select(u => new UserSummary
    {
        Name = u.Name,
        OrderCount = u.Orders.Count()
    })
    .ToListAsync();
```

**3. Batch Loading:**
```csharp
var userIds = users.Select(u => u.Id).ToList();
var orders = await context.Orders
    .Where(o => userIds.Contains(o.UserId))
    .ToListAsync();
```

**4. Split Query:**
```csharp
var users = await context.Users
    .AsSplitQuery()
    .Include(u => u.Orders)
    .ThenInclude(o => o.OrderItems)
    .ToListAsync();
```

---

### 9. `DbContext lifecycle ve best practices nelerdir?`
**Cevap:**

**DbContext Lifecycle:**
- **Short-lived**: Kısa sürelidir, işlem bitince dispose edilir
- **Unit of Work**: Bir iş birimi için kullanılır
- **Not Thread-safe**: Thread-safe değildir

**Best Practices:**

**1. Dependency Injection:**
```csharp
// Startup.cs
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));

// Controller
public class UsersController : ControllerBase
{
    private readonly ApplicationDbContext _context;
    
    public UsersController(ApplicationDbContext context)
    {
        _context = context;
    }
}
```

**2. Using Statement:**
```csharp
using (var context = new ApplicationDbContext())
{
    var users = await context.Users.ToListAsync();
    // Context otomatik dispose edilir
}
```

**3. ConfigureAwait(false):**
```csharp
public async Task<List<User>> GetUsersAsync()
{
    return await _context.Users.ToListAsync().ConfigureAwait(false);
}
```

**ConfigureAwait(false) Ne İşe Yarar?**
- **SynchronizationContext Capture**: Varsayılan olarak async metod, mevcut SynchronizationContext'i yakalar
- **UI Thread Blocking**: ASP.NET ve WinForms'da UI thread'de deadlock oluşabilir
- **Performance**: Context switching overhead'ini azaltır
- **Library Code**: Kütüphane kodlarında mutlaka kullanılmalı

**Örnek Problem:**
```csharp
// ❌ KÖTÜ - Deadlock riski
public void ButtonClick()
{
    var result = GetDataAsync().Result; // UI thread'de bekler
}

public async Task<string> GetDataAsync()
{
    return await httpClient.GetStringAsync("api/data"); // UI context'e dönmeye çalışır
}

// ✅ İYİ - ConfigureAwait ile çözüm
public async Task<string> GetDataAsync()
{
    return await httpClient.GetStringAsync("api/data").ConfigureAwait(false);
}
```

**Ne Zaman Kullanılır?**
- ✅ **Library/Service** kodlarında her zaman
- ✅ **Background** işlemlerde
- ✅ **API Controller**'larda genellikle
- ❌ **UI Event Handler**'larda UI'ya dönüş gerekiyorsa
---

### 10. `Change Tracking nedir ve nasıl optimize edilir?`
**Cevap:**
Change Tracking, EF'nin entity'lerdeki değişiklikleri takip etme mekanizmasıdır.

**Change Tracking States:**
- **Unchanged**: Değişmemiş
- **Added**: Eklenmiş
- **Modified**: Değiştirilmiş
- **Deleted**: Silinmiş
- **Detached**: Takip edilmiyor

**Change Tracking Optimizasyonu:**

**1. AsNoTracking:**
```csharp
// Read-only queries için
var users = await context.Users
    .AsNoTracking()
    .Where(u => u.IsActive)
    .ToListAsync();
```

**2. AsNoTrackingWithIdentityResolution:**
```csharp
// Aynı entity'nin birden fazla referansı varsa
var users = await context.Users
    .AsNoTrackingWithIdentityResolution()
    .Include(u => u.Orders)
    .ToListAsync();
```

**3. Manual Change Tracking:**
```csharp
// Sadece değiştirilen property'leri güncelle
context.Entry(user).Property(u => u.Name).IsModified = true;
context.Entry(user).Property(u => u.Email).IsModified = true;
await context.SaveChangesAsync();
```

**4. Bulk Operations:**
```csharp
// EF Core 7+ ile bulk operations
await context.Users
    .Where(u => u.IsActive == false)
    .ExecuteDeleteAsync();

await context.Users
    .Where(u => u.IsActive)
    .ExecuteUpdateAsync(u => u.SetProperty(x => x.LastLoginDate, DateTime.Now));
```

---

### 11. `Repository Pattern vs DbContext hangisini kullanmalı?`
**Cevap:**

**DbContext Zaten Unit of Work + Repository:**
- DbContext hem Unit of Work hem de Repository pattern'i implement eder
- DbSet<T> repository görevi görür
- SaveChanges() Unit of Work görevi görür

**Repository Pattern Avantajları:**
- Test edilebilirlik
- Abstraction
- Business logic encapsulation
- Multiple data source desteği

**Repository Pattern Örneği:**
```csharp
public interface IUserRepository
{
    Task<User> GetByIdAsync(int id);
    Task<List<User>> GetActiveUsersAsync();
    Task AddAsync(User user);
    Task UpdateAsync(User user);
    Task DeleteAsync(int id);
}

public class UserRepository : IUserRepository
{
    private readonly ApplicationDbContext _context;
    
    public UserRepository(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<User> GetByIdAsync(int id)
    {
        return await _context.Users
            .Include(u => u.Orders)
            .FirstOrDefaultAsync(u => u.Id == id);
    }
    
    public async Task<List<User>> GetActiveUsersAsync()
    {
        return await _context.Users
            .Where(u => u.IsActive)
            .AsNoTracking()
            .ToListAsync();
    }
}

// Generic Repository
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task AddAsync(T entity);
    void Update(T entity);
    void Remove(T entity);
}
```

**Tavsiye:**
- **Basit projeler**: Doğrudan DbContext kullan
- **Karmaşık business logic**: Repository pattern kullan
- **Test-heavy projeler**: Repository pattern daha uygun

---

### 12. `EF Core vs EF 6 farkları nelerdir?`
**Cevap:**

**EF Core Avantajları:**
- **Cross-platform**: Linux, macOS desteği
- **Performance**: Daha hızlı ve hafif
- **Modern**: Async-first design
- **Flexible**: Provider model ile farklı database'ler
- **Cloud-ready**: Container ve mikroservis uyumlu

**EF 6 Avantajları:**
- **Mature**: Uzun süreli test edilmiş
- **Full-featured**: Daha fazla özellik
- **EDMX**: Visual designer desteği
- **Complex mapping**: Karmaşık mapping senaryoları

**Feature Karşılaştırması:**

| Özellik | EF Core | EF 6 |
|---------|---------|------|
| .NET Framework | ✅ | ✅ |
| .NET Core/.NET 5+ | ✅ | ❌ |
| Cross Platform | ✅ | ❌ |
| Performance | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Lazy Loading | ✅ | ✅ |
| Many-to-Many | ✅ (5.0+) | ✅ |
| Table Splitting | ✅ | ✅ |
| Global Query Filters | ✅ | ❌ |
| Shadow Properties | ✅ | ❌ |
| Bulk Operations | ✅ (7.0+) | ❌ |

---

### 13. `IEnumerable vs IQueryable farkı nedir?`
**Cevap:**
IEnumerable ve IQueryable, veri koleksiyonları ile çalışmak için kullanılan iki farklı interface'dir, ancak çalışma şekilleri çok farklıdır.

**IEnumerable:**
- **In-Memory**: Veriler bellekte tutulur
- **Immediate Execution**: Sorgu hemen çalıştırılır
- **LINQ to Objects**: Bellekteki nesneler üzerinde çalışır
- **Client-side**: İşlemler istemci tarafında yapılır

**IQueryable:**
- **Database-side**: Veriler veritabanında kalır
- **Deferred Execution**: Sorgu geciktirilir
- **LINQ to SQL**: SQL sorgusuna çevrilir
- **Server-side**: İşlemler sunucu tarafında yapılır

**Performance Karşılaştırması:**

```csharp
// ❌ KÖTÜ - IEnumerable (All data to memory)
IEnumerable<User> users = context.Users; // Tüm kullanıcıları belleğe al
var activeUsers = users.Where(u => u.IsActive).ToList(); // Bellekte filtrele

// ✅ İYİ - IQueryable (Database filtering)
IQueryable<User> users = context.Users; // Sorgu oluştur
var activeUsers = users.Where(u => u.IsActive).ToList(); // SQL'de filtrele
```

**SQL Çıktısı Karşılaştırması:**

```csharp
// IEnumerable örneği
IEnumerable<User> enumerable = context.Users.AsEnumerable();
var result1 = enumerable.Where(u => u.Age > 18).Take(10).ToList();
// SQL: SELECT * FROM Users (TÜM veriler gelir, bellekte filtrelenir)

// IQueryable örneği  
IQueryable<User> queryable = context.Users;
var result2 = queryable.Where(u => u.Age > 18).Take(10).ToList();
// SQL: SELECT TOP 10 * FROM Users WHERE Age > 18 (Veritabanında filtrelenir)
```

**Composition (Birleştirme):**

```csharp
// IQueryable - Dynamic query building
public IQueryable<User> GetUsers(bool includeInactive = false)
{
    var query = context.Users.AsQueryable();
    
    if (!includeInactive)
    {
        query = query.Where(u => u.IsActive);
    }
    
    return query; // Henüz execute edilmedi
}

// Kullanım
var users = GetUsers()
    .Where(u => u.Age > 18)
    .OrderBy(u => u.Name)
    .Take(10)
    .ToList(); // Burada execute edilir

// Final SQL: SELECT TOP 10 * FROM Users WHERE IsActive = 1 AND Age > 18 ORDER BY Name
```

**Expression Trees:**

```csharp
// IQueryable expression tree'ları kullanır
Expression<Func<User, bool>> predicate = u => u.Age > 18;
var users = context.Users.Where(predicate).ToList();
// Bu expression tree SQL'e çevrilir

// IEnumerable delegate kullanır
Func<User, bool> func = u => u.Age > 18;
var users2 = context.Users.AsEnumerable().Where(func).ToList();
// Bu sadece bellekte çalışır
```

**Practical Examples:**

```csharp
// ❌ Performance Problem
public List<User> GetActiveUsersWithOrders()
{
    // Tüm kullanıcıları belleğe al
    var allUsers = context.Users.AsEnumerable();
    
    // Bellekte filtrele (YAVAŞ)
    return allUsers
        .Where(u => u.IsActive && u.Orders.Any())
        .ToList();
}

// ✅ Optimized Version
public List<User> GetActiveUsersWithOrders()
{
    // Veritabanında filtrele (HIZLI)
    return context.Users
        .Where(u => u.IsActive && u.Orders.Any())
        .ToList();
}

// ✅ Best Practice - Return IQueryable for further composition
public IQueryable<User> GetActiveUsers()
{
    return context.Users.Where(u => u.IsActive);
}

// Kullanım
var recentActiveUsers = GetActiveUsers()
    .Where(u => u.CreatedDate > DateTime.Now.AddDays(-30))
    .OrderByDescending(u => u.CreatedDate)
    .Take(20)
    .ToList();
```

**Ne Zaman Hangisini Kullanmalı?**

**IQueryable Kullan:**
- ✅ Entity Framework ile çalışırken
- ✅ Database'den veri çekerken
- ✅ Büyük veri setleriyle çalışırken
- ✅ Dynamic query building için
- ✅ Performance kritik uygulamalarda

**IEnumerable Kullan:**
- ✅ In-memory koleksiyonlarla çalışırken
- ✅ LINQ to Objects işlemlerinde
- ✅ Küçük veri setlerinde
- ✅ Complex business logic uygulanırken (EF tarafından desteklenmeyen)

```csharp
// Memory'deki koleksiyonlar için IEnumerable
List<string> cities = new List<string> { "İstanbul", "Ankara", "İzmir" };
var filteredCities = cities.Where(c => c.StartsWith("İ")); // IEnumerable

// Database queries için IQueryable  
var activeUsers = context.Users.Where(u => u.IsActive); // IQueryable
```

---

### 14. `Raw SQL kullanımı ve güvenlik nasıl sağlanır?`
**Cevap:**

**Raw SQL Kullanım Senaryoları:**
- Karmaşık stored procedure'ler
- Performance kritik sorgular
- Database-specific özellikler
- Existing SQL kod migration

**Raw SQL Örnekleri:**

**1. FromSqlRaw (Entity return):**
```csharp
var users = await context.Users
    .FromSqlRaw("SELECT * FROM Users WHERE IsActive = {0}", true)
    .Include(u => u.Orders)
    .ToListAsync();
```

**2. ExecuteSqlRaw (Non-query):**
```csharp
var rowsAffected = await context.Database
    .ExecuteSqlRawAsync("UPDATE Users SET LastLoginDate = {0} WHERE Id = {1}", 
        DateTime.Now, userId);
```

**3. Stored Procedure:**
```csharp
var users = await context.Users
    .FromSqlRaw("EXEC GetUsersByRole {0}", "Admin")
    .ToListAsync();
```

**4. SQL Injection Korunması:**
```csharp
// ❌ YANLIŞ - SQL Injection riski
var email = "test@example.com";
var users = await context.Users
    .FromSqlRaw($"SELECT * FROM Users WHERE Email = '{email}'")
    .ToListAsync();

// ✅ DOĞRU - Parametreli sorgu
var users = await context.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Email = {0}", email)
    .ToListAsync();

// ✅ DOĞRU - FormattableString
var users = await context.Users
    .FromSqlInterpolated($"SELECT * FROM Users WHERE Email = {email}")
    .ToListAsync();
```

**5. Complex Queries:**
```csharp
// CTE (Common Table Expression) örneği
var result = await context.Database
    .SqlQueryRaw<UserOrderSummary>(@"
        WITH UserOrderCounts AS (
            SELECT UserId, COUNT(*) as OrderCount,
                   SUM(TotalAmount) as TotalSpent
            FROM Orders 
            WHERE CreatedDate >= {0}
            GROUP BY UserId
        )
        SELECT u.Name, u.Email, uoc.OrderCount, uoc.TotalSpent
        FROM Users u
        INNER JOIN UserOrderCounts uoc ON u.Id = uoc.UserId
        ORDER BY uoc.TotalSpent DESC", 
        DateTime.Now.AddYears(-1))
    .ToListAsync();
```

---

### 15. `.NET 6 ile gelen yenilikler nelerdir?`
**Cevap:**
- **Minimal API**: Daha az kod ile hızlı API geliştirme
- **Top-level statements**: Program.cs'de Main metodu olmadan çalışma
- **Hot Reload**: Kod değişikliklerini uygulamayı durdurmadan görme
- **C# 10**: Record structs, global using, file-scoped namespaces
- **Performance iyileştirmeleri**: %20-30 performans artışı
- **Unified platform**: .NET Framework ve .NET Core birleşimi

---

### 16. `.NET 7 ile gelen yenilikler nelerdir?`
**Cevap:**
- **Rate Limiting**: API rate limiting desteği
- **Output Caching**: Response caching middleware
- **C# 11**: Raw string literals, required members, generic attributes
- **Blazor United**: Server ve WebAssembly birleşimi
- **Performance iyileştirmeleri**: Daha hızlı startup ve memory kullanımı
- **Native AOT**: Ahead-of-time compilation desteği

---

### 17. `.NET 8 ile gelen yenilikler nelerdir?`
**Cevap:**
- **Blazor United**: Server ve WebAssembly arasında geçiş
- **C# 12**: Collection expressions, optional parameters, ref readonly
- **Performance iyileştirmeleri**: %20 daha hızlı JSON serialization
- **Native AOT**: Production-ready AOT compilation
- **Time abstraction**: ITimeProvider interface
- **Keyed services**: DI container'da keyed service desteği

---

### 18. `.NET 9 ile gelen yenilikler nelerdir?`
**Cevap:**
- **C# 13**: Primary constructors, collection literals
- **Performance iyileştirmeleri**: Daha hızlı startup ve lower memory usage
- **Enhanced debugging**: Daha iyi debugging deneyimi
- **Cloud-native improvements**: Container ve cloud optimizasyonları
- **Security enhancements**: Güvenlik iyileştirmeleri
- **Developer productivity**: Geliştirici verimliliği artışları

---