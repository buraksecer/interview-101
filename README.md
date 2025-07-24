# Interview 101 - Hazırlık Notları

Bu repository, interview hazırlığı sırasında çalıştığım konuların notlarını içerir. İçerik soru-cevap formatında düzenlenmiştir.

---

## 📝 Soru & Cevaplar

### `Bir REST API'nin performansını artırmak için hangi teknikleri kullanırsın?`

**Cevap:** Caching, eTag, pagination, eager loading, lazy loading, limit-offset veya cursor-based paging.

**Kaynak:** 
- Etag: HTTP response'un içeriğini temsil eden benzersiz bir kimliktir. Amacı, istemcinin daha önce aldığı verinin değişip değişmediğini hızlıca anlamasını sağlamak. 
- Eager Loading: İlişkili veriler, ana sorguyla birlikte alınır (JOIN).
- Lazy Loading: İlişkili veriler, ancak ihtiyaç duyulunca yüklenir.
- Limit-offset: İstemci kaç kayıt istediğini ve hangi noktadan itibaren başlaması gerentiğini belirtir. Offset büyüdükçe performans düşer çünkü her seferinde kayıtları yeniden tarar.
- Cursor-based: Offset yerine bir “cursor” (işaretleyici) kullanılır. Bu cursor genellikle bir benzersiz ve sıralanabilir alan olur (örn. created_at, id, uuid). Offset taraması yapmaz daha hızlıdır.

**Örnek:**
```
request: GET /products/123 HTTP/1.1

response: HTTP/1.1 200 OK
Content-Type: application/json
ETag: "abc123"
```

---

### `REST API güvenliği için neler uygularsın?`
**Cevap:** OAuth 2.0, JWT ile kimlik doğrulama, Rate limiting, Input validation ve sanitization, CORS.

**Kaynak:** 
- OAtuh 2.0: OAuth 2.0, bir kullanıcının kimliğini doğrulamadan üçüncü taraf bir uygulamanın onun adına işlem yapmasına izin vermesini sağlayan yetkilendirme protokolüdür. Genelde Google, GitHub, Facebook gibi servislerde görürüz.
- Sanitization: Tehlikeli karakterleri güvenli hale getiririz, veri yapısını bozmadan filtreleriz. Bu işlem özellikle XSS (Cross-Site Scripting), SQL Injection, Command Injection saldırılarına karşı önemlidir.
- XSS: Kötü niyetli script kodlarının, bir web sayfası aracılığıyla başka kullanıcıların tarayıcısında çalıştırılması.
- CORS: Bir tarayıcıda çalışan frontend uygulama, farklı bir domain’e API isteği atarsa aynı kaynak politikası (Same-Origin Policy) gereği engellenir.
- Authentication: Bu kişi kim?
- Authorization: Bu kişi ne yapabilir?
---

### `API Gateway bize ne sağlar?`
**Cevap** API Gateway, client ile microservice’ler arasında tek giriş noktası görevi gören bir sunucudur.
Tüm istekler önce API Gateway’e gelir, oradan ilgili servislere yönlendirilir. Routing, Auth, Rate Limiting, Caching, Logging, Transformasyon(istek üzerinde düzenleme) gibi işlemler merkezleştirilir. Böylece microservisler bu sorumlulukları almazlar.

---

### `Concurrency ile çalışan REST API’lerde race condition nasıl önlenir?`
**Cevap** Optimistic Locking, Pessimistic Locking, Queue / Task Based Execution,Distributed Locking (Redlock, Zookeeper, etcd)
**Kaynak**
- Optimistic Locking: Veriyi versiyonlayarak başkası tarafından ezilmesini önleriz.
- Pessimistic Locking: Kaynak işlem sırasında kitlenir ve başkası tarafından değiştirilmesi önlenir.
- Distributed Locking: Birden fazla instance aynı işlemi işlemesin diye dağıtık sistemler kullanılır. Örneğin bir işlemi 10 saniyeliğine kilitlemek için redis e bir lock kaydı eklenir. `SET my_lock unique_token NX PX 10000` sadece lock sahibi unlock edebilir.

### `Relational database’lerde normalization nedir? Ne zaman kullanılır, ne zaman kullanmamak gerekir?`

**Cevap** Veriyi gereksiz tekrarları ortadan kaldırarak, anlamlı tablolara bölme işlemidir.
**Kaynak**
- 1NF: Bir alana 1 değer yazılabilir, atomic tekil değerleri temsil etmelidir. Örneği telefon alanına sadece 1 telefon eklenebilir.
- 2NF: Bir tabloda birleşik anahtar varsa tüm non-key sütunlar, anahtarın tamamına tam bağımlı olmalı. Aşağıda kötü bir örnek var.

**Kötü Örnek:**
| SiparişID | ÜrünID | ÜrünAdı | MüşteriAdı |
|-----------|--------|---------|------------|
| 1         | 101    | Ayakkabı | Ali        |
| 1         | 102    | Mont     | Ali        |

**Doğru Örnek:**

**Siparişler Tablosu:**
| SiparişID | MüşteriAdı |
|-----------|------------|
| 1         | Ali        |

**SiparişDetay Tablosu:**
| SiparişID | ÜrünID | ÜrünAdı |
|-----------|--------|---------|
| 1         | 101    | Ayakkabı |
| 1         | 102    | Mont     |
- 3NF: Hiçbir non-key alan, başka bir non-key alana transitif (dolaylı) bağımlı olmamalı. Yani özetle bir tabloda hem şehir hem posta hem müşteri adı hem müşteri id olmaz. Şehir ve posta kodu ayrı tabloda olması lazım.


### `CQRS(Command Query Responsibility Segregation) ne zaman kullanılır?`
**Cevap**
- Karmaşık domainlerde(e-ticaret-finans vb.)
- Okuma ve yazma yükleri çok farklıysa (o zaman okumaya ve yazmaya fark ORM kullanabilirsiniz!)
- Read tarafında düşük latency ihtiyacı varsa.
- Küçük, CRUD tabanlı sistemlerde kullanmaya ihtiyacın yok. Bas geç :)

**Kaynak**
CQRS: Okuma (Query) ve yazma (Command) işlemlerinin sorumluluklarını ayıran bir mimari yaklaşımdır.
Bu sayede sistemin karmaşıklığı yönetilebilir hale gelir ve farklı ölçeklenebilirlik ihtiyaçlarına göre optimize edilebilir. CQRS genellikle event sourcing ile birlikte gelir.	CommandHandler, QueryHandler patternleri ile uyumludur.

### `Middleware Yapıları Nasıl Kurgulanmalı?`
**Cevap**
Middleware, bir isteğin gelişim sürecinde araya giren yapıların tümüdür: loglama, auth, hata yönetimi, vs.
``Request → Middleware 1 → Middleware 2 → Controller → Response`` gibi.
- Küçük, bağımsız yapılara böl (Single Responsibility)
- Sıralama önemlidir! Örn: auth önce, log sonra (sırayla çalışırlar sıralamayı karıştırırsan bazı data kayıpları olabilir.)
- Ortak reusable yapı olarak yaz → tüm endpoint’lerde kullanılır.

### `Token-based Authentication vs Session-based Authentication?`
**Cevap**
Token-Based Auth(JWT): Mobile, SPA, microservices
- Avantajları: Sunucu tarafında state tutulmaz, Microservice mimarilerine uygundur, Frontend’ler arası geçiş kolaydır (Web → Mobil).
- Dezavantajlar: Token süresi boyunca yetki iptal edilemez (blacklist ile çözüm gerekir),Token payload decode edilebilir (şifrelenmeli değilse).
Session-Based Auth: Klasik web uygulamaları, Her client için benzersiz session oluşturulur.
- Avantajları: Oturum üzerinde daha fazla kontrol, Session server’da iptal edilebilir.
- Dezavantajlar: Ölçeklenebilirlik sorunları (session replication gerekir), Mobile API’lerde taşınabilir değil.

### ` Bir API Endpoint’i Saniyede 1000 İstek Almaya Başlarsa Ne Yaparsın?`
**Cevap**
CPU yükü, veritabanı baskısı, latency artışı, rate limit riski..
- Rate Limiting Uygula: Kullanıcı başına istek sınırı koy (IP ya da API key bazlı)
- Cache Layer Kullan: Sık çağrılan veri ise MemoryCache, Redis, CDN katmanı ile response’u cache’le
- Async / Queue Yapısı: ğer istek işlem gerektiriyorsa (mail gönderme, PDF üretme) → bir kuyruğa (RabbitMQ, Kafka) gönder.
- Load Balancing: Tek sunucu yanıt veremiyorsa, API servisini ölçekle → birden fazla instance. NGINX / AWS ELB gibi yük dengeleyici kullan.
- DB Performans Ayarları: Index kontrolü, Query optimizasyon, Connection pool sınırı.

### `Load Balancing Nedir?`
**Cevap**
Gelen trafiği arka planda çalışan birden fazla sunucuya dengeli bir şekilde dağıtan sistemdir.
- Performansı artırmak.
- Sunucular arası yükü dengelemek.
- Kesintisiz hizmet sağlamak (failover).
- Load Balancer Nasıl Konumlanır?
```
Kullanıcılar
     ↓
[ Load Balancer ]
   ↓     ↓     ↓
App 1   App 2  App 3  (Backend sunucuları)
```
-  Ne Tür Yükler Dengelenir?
 - HTTP istekleri (web/API trafiği)
 - TCP bağlantıları
 - Veritabanı sorguları (örneğin read-replica sistemleri)
 - Mail, FTP gibi özel servisler

- Load Balancing Yöntemleri?
 - Round Robin,  Least Connections, IP Hashing, Weighted Round Robin. Eğer size yöntem sorarlarsa güzel bir paraya işe giriyorsunuz demektir :D
-  Dikkat Edilmesi Gerekenler?
 - Session tutarlılığı gerekirse Sticky sessions veya Redis tabanlı centralized session. (sonra benim kullanıcım neden logout oldu demeyin, ya session ortak bir yerde saklayın redis vs ya da sticky session ile ip hash => sunucu da tutun ama o zaman neden load balancer yapıyoruz tek sunucudan besleneceksek?)
 - Health check yapılmazsa, Bozuk sunucuya istek gitmeye devam eder → health_check_interval, timeout ayarlanmalı.

---

### `Nesne Tabanlı Programlama (OOP) nedir ve temel prensipleri nelerdir?`

**Cevap:** 
Nesne tabanlı programlama, gerçek dünya nesnelerini modelleyerek program yazma yaklaşımıdır.

**Kaynak:** 
- **Encapsulation (Kapsülleme)**: Veri ve metodları bir arada tutma, dış dünyadan gizleme
- **Inheritance (Kalıtım)**: Bir sınıfın başka bir sınıftan özellik ve davranış alması
- **Polymorphism (Çok Biçimlilik)**: Aynı arayüzün farklı implementasyonları
- **Abstraction (Soyutlama)**: Karmaşık sistemleri basitleştirme

---

### `Encapsulation (Kapsülleme) nedir ve neden önemlidir?`

**Cevap:** 
Encapsulation, veri ve bu veri üzerinde çalışan metodları bir arada tutma ve dış dünyadan gizleme prensibidir.

**Kaynak:** 
- **Data Hiding**: Verilerin doğrudan erişimini engeller
- **Access Modifiers**: public, private, protected anahtar kelimeleri
- **Getter/Setter**: Kontrollü veri erişimi sağlar
- **Maintenance**: Kod değişikliklerini kolaylaştırır

**Örnek:**
```csharp
public class BankAccount
{
    private double balance; // private - dış dünyadan gizli
    
    public double Balance // C# property
    {
        get { return balance; }
    }
    
    public void Deposit(double amount)
    {
        if (amount > 0)
        {
            balance += amount;
        }
    }
}
```

---

### `Inheritance (Kalıtım) nedir ve hangi durumlarda kullanılır?`

**Cevap:** 
Inheritance, bir sınıfın başka bir sınıftan özellik ve davranış almasıdır. Kod tekrarını önler ve hiyerarşi oluşturur.

**Kaynak:** 
- **IS-A Relationship**: "Bir şey başka bir şeydir" ilişkisi
- **Code Reuse**: Kod tekrarını önler
- **Method Overriding**: Alt sınıflar üst sınıf metodlarını değiştirebilir
- **Single/Multiple Inheritance**: Tek veya çoklu kalıtım

**Örnek:**
```csharp
public class Animal
{
    protected string name;
    
    public virtual void MakeSound()
    {
        Console.WriteLine("Some sound");
    }
}

public class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
}
```

---

### `Polymorphism (Çok Biçimlilik) nedir ve türleri nelerdir?`

**Cevap:** 
Polymorphism, aynı arayüzün farklı implementasyonlarıdır. İki türü vardır: Compile-time (Overloading) ve Runtime (Overriding).

**Kaynak:** 
- **Method Overloading**: Aynı isimde farklı parametreli metodlar
- **Method Overriding**: Alt sınıfların üst sınıf metodlarını değiştirmesi
- **Interface Polymorphism**: Interface implementasyonları
- **Dynamic Binding**: Çalışma zamanında hangi metodun çağrılacağının belirlenmesi

**Örnek:**
```csharp
// Method Overloading
public class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
    
    public double Add(double a, double b)
    {
        return a + b;
    }
}

// Method Overriding
Animal animal = new Dog(); // Runtime polymorphism
animal.MakeSound(); // "Woof!" yazdırır
```

---

### `Abstraction (Soyutlama) nedir ve nasıl sağlanır?`

**Cevap:** 
Abstraction, karmaşık sistemleri basitleştirme ve sadece gerekli detayları gösterme prensibidir.

**Kaynak:** 
- **Abstract Classes**: Tamamlanmamış sınıflar, abstract metodlar içerir
- **Interfaces**: Sadece metod imzalarını tanımlar
- **Implementation Hiding**: Nasıl çalıştığı gizlenir, ne yaptığı gösterilir
- **Loose Coupling**: Sınıflar arası gevşek bağlantı

**Abstract Class vs Interface - Neden Abstract Class Kullandık?**

**Abstract Class'ın Avantajları:**
- **State Tutabilir**: `name`, `color` gibi ortak veriler abstract class'ta saklanır
- **Constructor**: Ortak başlatma işlemleri yapılabilir
- **Concrete Methods**: `Display()`, `ChangeColor()` gibi ortak implementasyonlar
- **Code Reuse**: Alt sınıflar ortak kodları tekrar yazmaz
- **Access Modifiers**: `protected` ile kontrollü erişim

**Interface Kullansaydık:**
- Her alt sınıf `name`, `color` alanlarını ayrı ayrı tanımlamak zorunda kalırdı
- `Display()` metodunu her sınıfta tekrar yazmak gerekirdi
- Constructor'da ortak başlatma yapılamazdı
- Kod tekrarı (duplication) olurdu

**Örnek Karşılaştırma:**
```csharp
// Interface ile (kötü yaklaşım)
public interface IShape
{
    double CalculateArea();
    double CalculatePerimeter();
    void Display(); // Her sınıf implement etmek zorunda
}

public class Circle : IShape
{
    private string name; // Tekrar!
    private string color; // Tekrar!
    private double radius;
    
    public Circle(string name, string color, double radius)
    {
        this.name = name; // Tekrar!
        this.color = color; // Tekrar!
        this.radius = radius;
    }
    
    public double CalculateArea() { return Math.PI * radius * radius; }
    public double CalculatePerimeter() { return 2 * Math.PI * radius; }
    
    public void Display() // Tekrar yazıldı!
    {
        Console.WriteLine($"Shape: {name}");
        Console.WriteLine($"Color: {color}");
        Console.WriteLine($"Area: {CalculateArea():F2}");
        Console.WriteLine($"Perimeter: {CalculatePerimeter():F2}");
    }
}

// Abstract Class ile (iyi yaklaşım)
public abstract class Shape
{
    protected string name; // Ortak!
    protected string color; // Ortak!
    
    public Shape(string name, string color) // Ortak constructor!
    {
        this.name = name;
        this.color = color;
    }
    
    public abstract double CalculateArea();
    public abstract double CalculatePerimeter();
    
    public virtual void Display() // Ortak implementasyon!
    {
        Console.WriteLine($"Shape: {name}");
        Console.WriteLine($"Color: {color}");
        Console.WriteLine($"Area: {CalculateArea():F2}");
        Console.WriteLine($"Perimeter: {CalculatePerimeter():F2}");
    }
}

public class Circle : Shape
{
    private double radius;
    
    public Circle(string name, string color, double radius) : base(name, color) // Ortak başlatma!
    {
        this.radius = radius;
    }
    
    public override double CalculateArea() { return Math.PI * radius * radius; }
    public override double CalculatePerimeter() { return 2 * Math.PI * radius; }
    // Display() metodunu yazmaya gerek yok! Abstract class'tan geliyor
}
```

**Örnek:**
```csharp
// Abstract Class - Ortak davranışları ve abstract metodları tanımlar
public abstract class Shape
{
    protected string name;
    protected string color;
    
    public Shape(string name, string color)
    {
        this.name = name;
        this.color = color;
    }
    
    // Abstract method - Alt sınıflar implement etmek zorunda
    public abstract double CalculateArea();
    
    // Abstract method - Alt sınıflar implement etmek zorunda
    public abstract double CalculatePerimeter();
    
    // Concrete method - Ortak davranış, tüm alt sınıflar kullanabilir
    public virtual void Display()
    {
        Console.WriteLine($"Shape: {name}");
        Console.WriteLine($"Color: {color}");
        Console.WriteLine($"Area: {CalculateArea():F2}");
        Console.WriteLine($"Perimeter: {CalculatePerimeter():F2}");
    }
    
    // Concrete method - Ortak davranış
    public void ChangeColor(string newColor)
    {
        color = newColor;
        Console.WriteLine($"{name} color changed to {newColor}");
    }
}

// Interface - Sadece davranış tanımlar, implementation yok
public interface IDrawable
{
    void Draw();
    void Erase();
}

public interface IResizable
{
    void Resize(double factor);
}

// Concrete implementation
public class Circle : Shape, IDrawable, IResizable
{
    private double radius;
    
    public Circle(string name, string color, double radius) : base(name, color)
    {
        this.radius = radius;
    }
    
    public override double CalculateArea()
    {
        return Math.PI * radius * radius;
    }
    
    public override double CalculatePerimeter()
    {
        return 2 * Math.PI * radius;
    }
    
    public void Draw()
    {
        Console.WriteLine($"Drawing circle with radius {radius}");
    }
    
    public void Erase()
    {
        Console.WriteLine("Erasing circle");
    }
    
    public void Resize(double factor)
    {
        radius *= factor;
        Console.WriteLine($"Circle resized by factor {factor}. New radius: {radius}");
    }
}

public class Rectangle : Shape, IDrawable
{
    private double width;
    private double height;
    
    public Rectangle(string name, string color, double width, double height) : base(name, color)
    {
        this.width = width;
        this.height = height;
    }
    
    public override double CalculateArea()
    {
        return width * height;
    }
    
    public override double CalculatePerimeter()
    {
        return 2 * (width + height);
    }
    
    public void Draw()
    {
        Console.WriteLine($"Drawing rectangle {width}x{height}");
    }
    
    public void Erase()
    {
        Console.WriteLine("Erasing rectangle");
    }
}

// Kullanım örneği
public class Program
{
    public static void Main()
    {
        // Abstraction sayesinde Shape tipinde tutabiliyoruz
        Shape circle = new Circle("MyCircle", "Red", 5.0);
        Shape rectangle = new Rectangle("MyRectangle", "Blue", 4.0, 6.0);
        
        // Polymorphism - aynı metod farklı davranışlar
        circle.Display();
        Console.WriteLine();
        rectangle.Display();
        
        // Interface abstraction
        IDrawable drawableCircle = (IDrawable)circle;
        drawableCircle.Draw();
        drawableCircle.Erase();
        
        // Multiple interface implementation
        IResizable resizableCircle = (IResizable)circle;
        resizableCircle.Resize(2.0);
    }
}
```

---

### `Composition vs Inheritance ne zaman kullanılır?`

**Cevap:** 
"Inheritance yerine composition tercih edin" prensibi. Composition daha esnek ve güvenli bir yaklaşımdır.

**Kaynak:** 
- **Inheritance**: IS-A ilişkisi (Dog IS-A Animal)
- **Composition**: HAS-A ilişkisi (Car HAS-A Engine)
- **Favor Composition**: Daha az bağımlılık, daha kolay test
- **Diamond Problem**: Çoklu kalıtımda oluşan sorunlar

**Örnek:**
```csharp
// Inheritance
public class Car : Vehicle { }

// Composition (tercih edilen)
public class Car
{
    private Engine engine;
    private List<Wheel> wheels;
    
    public Car(Engine engine)
    {
        this.engine = engine;
    }
}
```

---

### `SOLID prensipleri nelerdir?`

**Cevap:** 
SOLID, nesne tabanlı programlama için 5 temel prensiptir.

**Kaynak:** 
- **S - Single Responsibility**: Bir sınıfın tek bir sorumluluğu olmalı
- **O - Open/Closed**: Açık genişlemeye, kapalı değişikliğe
- **L - Liskov Substitution**: Alt sınıflar üst sınıfların yerine geçebilmeli
- **I - Interface Segregation**: Küçük, özel interface'ler
- **D - Dependency Inversion**: Soyutlamalara bağımlılık, somutlamalara değil

**Detaylı Örnekler:**

**1. S - Single Responsibility Principle (SRP)**
```csharp
// KÖTÜ: Birden fazla sorumluluk
public class UserManager
{
    public void CreateUser(User user) { }
    public void UpdateUser(User user) { }
    public void DeleteUser(int id) { }
    public void SendEmail(string to, string subject) { } // Email sorumluluğu!
    public void SaveToDatabase(User user) { } // Database sorumluluğu!
    public void ValidateUser(User user) { } // Validation sorumluluğu!
}

// İYİ: Her sınıf tek sorumluluk
public class UserService
{
    private readonly IUserRepository _repository;
    private readonly IEmailService _emailService;
    private readonly IUserValidator _validator;
    
    public UserService(IUserRepository repository, IEmailService emailService, IUserValidator validator)
    {
        _repository = repository;
        _emailService = emailService;
        _validator = validator;
    }
    
    public void CreateUser(User user)
    {
        if (_validator.IsValid(user))
        {
            _repository.Save(user);
            _emailService.SendWelcomeEmail(user.Email);
        }
    }
}

public class UserRepository : IUserRepository
{
    public void Save(User user) { /* Database işlemleri */ }
}

public class EmailService : IEmailService
{
    public void SendWelcomeEmail(string email) { /* Email gönderme */ }
}

public class UserValidator : IUserValidator
{
    public bool IsValid(User user) { /* Validation kuralları */ }
}
```

**2. O - Open/Closed Principle (OCP)**
```csharp
// KÖTÜ: Yeni ödeme yöntemi eklemek için mevcut kodu değiştirmek gerekir
public class PaymentProcessor
{
    public void ProcessPayment(string paymentType, decimal amount)
    {
        if (paymentType == "CreditCard")
        {
            // Credit card işlemleri
        }
        else if (paymentType == "PayPal")
        {
            // PayPal işlemleri
        }
        // Yeni ödeme yöntemi eklemek için buraya yeni if bloğu eklemek gerekir!
    }
}

// İYİ: Yeni ödeme yöntemleri eklemek için mevcut kodu değiştirmeye gerek yok
public interface IPaymentMethod
{
    void ProcessPayment(decimal amount);
}

public class CreditCardPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing {amount:C} via Credit Card");
    }
}

public class PayPalPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing {amount:C} via PayPal");
    }
}

public class CryptoPayment : IPaymentMethod // Yeni ödeme yöntemi - mevcut kodu değiştirmeden!
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing {amount:C} via Crypto");
    }
}

public class PaymentProcessor
{
    public void ProcessPayment(IPaymentMethod paymentMethod, decimal amount)
    {
        paymentMethod.ProcessPayment(amount);
    }
}
```

**3. L - Liskov Substitution Principle (LSP)**
```csharp
// KÖTÜ: LSP ihlali
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    
    public virtual int CalculateArea()
    {
        return Width * Height;
    }
}

public class Square : Rectangle
{
    public override int Width
    {
        get => base.Width;
        set
        {
            base.Width = value;
            base.Height = value; // Square'de width ve height aynı olmalı
        }
    }
    
    public override int Height
    {
        get => base.Height;
        set
        {
            base.Width = value; // Square'de width ve height aynı olmalı
            base.Height = value;
        }
    }
}

// Kullanımda sorun:
public void TestArea(Rectangle rectangle)
{
    rectangle.Width = 5;
    rectangle.Height = 10;
    int expectedArea = 50; // 5 * 10
    int actualArea = rectangle.CalculateArea();
    
    // Eğer rectangle aslında Square ise, area 100 olur (10 * 10)!
    // Bu LSP ihlalidir çünkü Square, Rectangle'ın yerine geçemez
}

// İYİ: LSP uyumlu
public abstract class Shape
{
    public abstract int CalculateArea();
}

public class Rectangle : Shape
{
    public int Width { get; set; }
    public int Height { get; set; }
    
    public override int CalculateArea()
    {
        return Width * Height;
    }
}

public class Square : Shape
{
    public int Side { get; set; }
    
    public override int CalculateArea()
    {
        return Side * Side;
    }
}
```

**4. I - Interface Segregation Principle (ISP)**
```csharp
// KÖTÜ: Büyük, monolitik interface
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
    void GetSalary();
    void TakeVacation();
    void AttendMeeting();
    void WriteCode();
    void TestCode();
    void DeployCode();
}

// Bu interface'i implement eden sınıflar kullanmadıkları metodları da implement etmek zorunda
public class Developer : IWorker
{
    public void Work() { }
    public void Eat() { }
    public void Sleep() { }
    public void GetSalary() { }
    public void TakeVacation() { }
    public void AttendMeeting() { }
    public void WriteCode() { }
    public void TestCode() { }
    public void DeployCode() { }
}

public class Manager : IWorker
{
    public void Work() { }
    public void Eat() { }
    public void Sleep() { }
    public void GetSalary() { }
    public void TakeVacation() { }
    public void AttendMeeting() { }
    public void WriteCode() { } // Manager kod yazmaz!
    public void TestCode() { } // Manager test yapmaz!
    public void DeployCode() { } // Manager deploy yapmaz!
}

// İYİ: Küçük, özel interface'ler
public interface IWorkable
{
    void Work();
}

public interface IEatable
{
    void Eat();
}

public interface ISleepable
{
    void Sleep();
}

public interface IPayable
{
    void GetSalary();
}

public interface IVacationable
{
    void TakeVacation();
}

public interface IMeetingAttendable
{
    void AttendMeeting();
}

public interface ICodeWritable
{
    void WriteCode();
}

public interface ICodeTestable
{
    void TestCode();
}

public interface ICodeDeployable
{
    void DeployCode();
}

// Sınıflar sadece ihtiyaç duydukları interface'leri implement eder
public class Developer : IWorkable, IEatable, ISleepable, IPayable, IVacationable, 
                        IMeetingAttendable, ICodeWritable, ICodeTestable, ICodeDeployable
{
    public void Work() { }
    public void Eat() { }
    public void Sleep() { }
    public void GetSalary() { }
    public void TakeVacation() { }
    public void AttendMeeting() { }
    public void WriteCode() { }
    public void TestCode() { }
    public void DeployCode() { }
}

public class Manager : IWorkable, IEatable, ISleepable, IPayable, IVacationable, IMeetingAttendable
{
    public void Work() { }
    public void Eat() { }
    public void Sleep() { }
    public void GetSalary() { }
    public void TakeVacation() { }
    public void AttendMeeting() { }
    // Kod yazma, test etme, deploy etme metodları yok!
}
```

**5. D - Dependency Inversion Principle (DIP)**
```csharp
// KÖTÜ: Somut sınıflara bağımlılık
public class UserService
{
    private readonly SqlServerDatabase _database; // Somut sınıfa bağımlılık
    
    public UserService()
    {
        _database = new SqlServerDatabase(); // Somut sınıf oluşturma
    }
    
    public void SaveUser(User user)
    {
        _database.Save(user);
    }
}

public class SqlServerDatabase
{
    public void Save(User user) { /* SQL Server'a kaydet */ }
}

// İYİ: Soyutlamalara bağımlılık
public interface IUserRepository
{
    void Save(User user);
    User GetById(int id);
}

public class SqlServerUserRepository : IUserRepository
{
    public void Save(User user) { /* SQL Server'a kaydet */ }
    public User GetById(int id) { /* SQL Server'dan getir */ }
}

public class MongoDbUserRepository : IUserRepository
{
    public void Save(User user) { /* MongoDB'ye kaydet */ }
    public User GetById(int id) { /* MongoDB'den getir */ }
}

public class UserService
{
    private readonly IUserRepository _repository; // Soyutlamaya bağımlılık
    
    public UserService(IUserRepository repository) // Dependency Injection
    {
        _repository = repository;
    }
    
    public void SaveUser(User user)
    {
        _repository.Save(user);
    }
}

// Kullanım - hangi database kullanılacağına dışarıdan karar verilir
var sqlService = new UserService(new SqlServerUserRepository());
var mongoService = new UserService(new MongoDbUserRepository());
```

---

### `Design Patterns nedir ve hangi kategorileri vardır?`

**Cevap:** 
Design Patterns, yaygın yazılım problemlerinin kanıtlanmış çözümleridir.

**Kaynak:** 
- **Creational Patterns**: Nesne oluşturma (Singleton, Factory, Builder)
- **Structural Patterns**: Sınıf yapıları (Adapter, Decorator, Proxy)
- **Behavioral Patterns**: Davranış kalıpları (Observer, Strategy, Command)
- **Anti-Patterns**: Kaçınılması gereken kalıplar

**Örnek:**
```csharp
// Singleton Pattern
public class DatabaseConnection
{
    private static DatabaseConnection instance;
    private static readonly object lockObject = new object();
    
    private DatabaseConnection() { }
    
    public static DatabaseConnection GetInstance()
    {
        if (instance == null)
        {
            lock (lockObject)
            {
                if (instance == null)
                {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}
```

---

### `Interface nedir ve Abstract Class'tan farkı nedir?`

**Cevap:** 
Interface, sadece metod imzalarını tanımlayan soyut yapılardır. Abstract class'lardan farklı olarak state tutamaz.

**Kaynak:** 
- **Interface**: Sadece metod imzaları, default metodlar (Java 8+)
- **Abstract Class**: Hem abstract hem concrete metodlar, state tutabilir
- **Multiple Inheritance**: Interface'ler çoklu implementasyon destekler
- **Contract**: Interface bir sözleşme gibi davranır

**Örnek:**
```csharp
public interface IFlyable
{
    void Fly();
    void Land() // C# 8.0+ default interface implementation
    {
        Console.WriteLine("Landing...");
    }
}

public abstract class Bird
{
    protected string name;
    
    public abstract void MakeSound();
    
    public void Eat()
    {
        Console.WriteLine("Eating...");
    }
}
```

---

### `Method Overriding vs Method Overloading nedir?`

**Cevap:** 
Method Overriding runtime'da çalışır, Method Overloading compile-time'da çözülür.

**Kaynak:** 
- **Method Overriding**: Alt sınıfların üst sınıf metodlarını değiştirmesi
- **Method Overloading**: Aynı sınıfta aynı isimde farklı parametreli metodlar
- **@Override Annotation**: Java'da override'ı belirtmek için
- **Dynamic vs Static Binding**: Runtime vs compile-time çözümleme

**Örnek:**
```csharp
// Method Overloading (Compile-time)
public class Math
{
    public int Add(int a, int b)
    {
        return a + b;
    }
    
    public double Add(double a, double b)
    {
        return a + b;
    }
}

// Method Overriding (Runtime)
public class Animal
{
    public virtual void MakeSound()
    {
        Console.WriteLine("Some sound");
    }
}

public class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
}
```


### `En Yaygın 5 Design Pattern Nedir?`

---

### `1. Saga Pattern nedir ve ne zaman kullanılır?`

**Cevap:** 
Saga Pattern, mikroservisler arası uzun süren transaction'ları yönetmek için kullanılan pattern'dir. Distributed transaction'ları choreography veya orchestration ile yönetir.

**Kaynak:** 
- **Choreography Saga**: Her servis kendi adımını bilir ve event publish eder
- **Orchestration Saga**: Merkezi orchestrator tüm süreci yönetir
- **Compensation**: Hata durumunda geri alma işlemleri
- **Eventual Consistency**: Nihai tutarlılık sağlar

**Örnek:**
```csharp
// Orchestration Saga Pattern
public class OrderSaga
{
    private readonly IPaymentService _paymentService;
    private readonly IInventoryService _inventoryService;
    private readonly IShippingService _shippingService;
    private readonly IOrderService _orderService;
    
    public async Task<SagaResult> ProcessOrderAsync(Order order)
    {
        var sagaState = new OrderSagaState { OrderId = order.Id };
        
        try
        {
            // Step 1: Reserve inventory
            sagaState.InventoryReservationId = await _inventoryService.ReserveAsync(order.Items);
            
            // Step 2: Process payment
            sagaState.PaymentId = await _paymentService.ChargeAsync(order.Amount, order.PaymentDetails);
            
            // Step 3: Create shipping
            sagaState.ShippingId = await _shippingService.CreateShipmentAsync(order.ShippingAddress);
            
            // Step 4: Confirm order
            await _orderService.ConfirmOrderAsync(order.Id);
            
            return SagaResult.Success();
        }
        catch (Exception ex)
        {
            // Compensation - geri alma işlemleri
            await CompensateAsync(sagaState);
            return SagaResult.Failure(ex.Message);
        }
    }
    
    private async Task CompensateAsync(OrderSagaState state)
    {
        if (state.ShippingId.HasValue)
            await _shippingService.CancelShipmentAsync(state.ShippingId.Value);
        
        if (state.PaymentId.HasValue)
            await _paymentService.RefundAsync(state.PaymentId.Value);
        
        if (state.InventoryReservationId.HasValue)
            await _inventoryService.ReleaseReservationAsync(state.InventoryReservationId.Value);
    }
}

// Choreography Saga Pattern
public class InventoryService
{
    public async Task ReserveInventoryAsync(Order order)
    {
        // Inventory reserve logic
        
        if (success)
        {
            await _eventBus.PublishAsync(new InventoryReservedEvent 
            { 
                OrderId = order.Id, 
                Items = order.Items 
            });
        }
        else
        {
            await _eventBus.PublishAsync(new InventoryReservationFailedEvent 
            { 
                OrderId = order.Id 
            });
        }
    }
}

public class PaymentService
{
    public async Task HandleInventoryReservedAsync(InventoryReservedEvent @event)
    {
        // Payment processing logic
        
        if (paymentSuccess)
        {
            await _eventBus.PublishAsync(new PaymentProcessedEvent 
            { 
                OrderId = @event.OrderId 
            });
        }
        else
        {
            await _eventBus.PublishAsync(new PaymentFailedEvent 
            { 
                OrderId = @event.OrderId 
            });
            
            // Compensate inventory
            await _eventBus.PublishAsync(new CompensateInventoryEvent 
            { 
                OrderId = @event.OrderId 
            });
        }
    }
}
```

---

### `2. Factory Pattern nedir ve türleri nelerdir?`

**Cevap:** 
Factory Pattern, nesne oluşturma sorumluluğunu ayrı bir sınıfa veren creational pattern'dir. Simple Factory, Factory Method ve Abstract Factory olmak üzere 3 türü vardır.

**Kaynak:** 
- **Simple Factory**: Statik metod ile nesne oluşturma
- **Factory Method**: Alt sınıfların hangi nesneyi oluşturacağına karar vermesi
- **Abstract Factory**: İlişkili nesne ailelerini oluşturma
- **Creator vs Product**: Oluşturan vs oluşturulan

**Örnek:**
```csharp
// Simple Factory
public class LoggerFactory
{
    public static ILogger CreateLogger(LoggerType type)
    {
        return type switch
        {
            LoggerType.File => new FileLogger(),
            LoggerType.Database => new DatabaseLogger(),
            LoggerType.Console => new ConsoleLogger(),
            _ => throw new ArgumentException("Invalid logger type")
        };
    }
}

// Factory Method Pattern
public abstract class DocumentCreator
{
    public abstract IDocument CreateDocument();
    
    public void ProcessDocument()
    {
        var document = CreateDocument();
        document.Open();
        document.Save();
        document.Close();
    }
}

public class WordDocumentCreator : DocumentCreator
{
    public override IDocument CreateDocument()
    {
        return new WordDocument();
    }
}

public class PdfDocumentCreator : DocumentCreator
{
    public override IDocument CreateDocument()
    {
        return new PdfDocument();
    }
}

// Abstract Factory Pattern
public interface IUIFactory
{
    IButton CreateButton();
    ITextbox CreateTextbox();
    ICheckbox CreateCheckbox();
}

public class WindowsUIFactory : IUIFactory
{
    public IButton CreateButton() => new WindowsButton();
    public ITextbox CreateTextbox() => new WindowsTextbox();
    public ICheckbox CreateCheckbox() => new WindowsCheckbox();
}

public class MacUIFactory : IUIFactory
{
    public IButton CreateButton() => new MacButton();
    public ITextbox CreateTextbox() => new MacTextbox();
    public ICheckbox CreateCheckbox() => new MacCheckbox();
}

public class Application
{
    private readonly IUIFactory _uiFactory;
    
    public Application(IUIFactory uiFactory)
    {
        _uiFactory = uiFactory;
    }
    
    public void CreateUI()
    {
        var button = _uiFactory.CreateButton();
        var textbox = _uiFactory.CreateTextbox();
        var checkbox = _uiFactory.CreateCheckbox();
    }
}
```

---

### `3. Observer Pattern nedir ve ne zaman kullanılır?`

**Cevap:** 
Observer Pattern, bir nesnenin durumu değiştiğinde bağımlı nesnelerin otomatik olarak bilgilendirilmesini sağlayan behavioral pattern'dir.

**Kaynak:** 
- **Subject**: Gözlemlenen nesne (Observable)
- **Observer**: Gözlemleyen nesneler
- **Event-driven Architecture**: Olay tabanlı mimari
- **Loose Coupling**: Gevşek bağlantı sağlar

**Örnek:**
```csharp
public interface IObserver<T>
{
    void Update(T data);
}

public interface ISubject<T>
{
    void Subscribe(IObserver<T> observer);
    void Unsubscribe(IObserver<T> observer);
    void Notify(T data);
}

public class StockPrice : ISubject<decimal>
{
    private readonly List<IObserver<decimal>> _observers = new();
    private decimal _price;
    
    public decimal Price
    {
        get => _price;
        set
        {
            _price = value;
            Notify(_price);
        }
    }
    
    public void Subscribe(IObserver<decimal> observer)
    {
        _observers.Add(observer);
    }
    
    public void Unsubscribe(IObserver<decimal> observer)
    {
        _observers.Remove(observer);
    }
    
    public void Notify(decimal price)
    {
        foreach (var observer in _observers)
        {
            observer.Update(price);
        }
    }
}

public class StockDisplay : IObserver<decimal>
{
    private readonly string _displayName;
    
    public StockDisplay(string displayName)
    {
        _displayName = displayName;
    }
    
    public void Update(decimal price)
    {
        Console.WriteLine($"{_displayName}: Stock price updated to {price:C}");
    }
}

public class StockAlert : IObserver<decimal>
{
    private readonly decimal _threshold;
    
    public StockAlert(decimal threshold)
    {
        _threshold = threshold;
    }
    
    public void Update(decimal price)
    {
        if (price > _threshold)
        {
            Console.WriteLine($"ALERT: Stock price {price:C} exceeded threshold {_threshold:C}!");
        }
    }
}

// Kullanım
var stock = new StockPrice();
var display1 = new StockDisplay("Main Display");
var display2 = new StockDisplay("Mobile App");
var alert = new StockAlert(100m);

stock.Subscribe(display1);
stock.Subscribe(display2);
stock.Subscribe(alert);

stock.Price = 95m; // Tüm observer'lar bilgilendirilir
stock.Price = 105m; // Alert da tetiklenir

// C# Events ile Observer Pattern
public class Stock
{
    public event Action<decimal> PriceChanged;
    
    private decimal _price;
    public decimal Price
    {
        get => _price;
        set
        {
            _price = value;
            PriceChanged?.Invoke(_price);
        }
    }
}

// Kullanım
var stock = new Stock();
stock.PriceChanged += price => Console.WriteLine($"Price changed to {price:C}");
stock.PriceChanged += price => { if (price > 100) Console.WriteLine("High price alert!"); };
stock.Price = 105m;
```

---

### `4. Strategy Pattern nedir ve Command Pattern'dan farkı nedir?`

**Cevap:** 
Strategy Pattern, runtime'da algoritmayı değiştirmeyi sağlayan behavioral pattern'dir. Command Pattern ise istekleri nesneler olarak kapsüller.

**Kaynak:** 
- **Strategy**: Algoritma değiştirme (HOW)
- **Command**: İstek kapsülleme (WHAT)
- **Context**: Strategy kullanan sınıf
- **Encapsulation**: İş mantığını kapsülleme

**Strategy Pattern Örneği:**
```csharp
public interface IPaymentStrategy
{
    PaymentResult ProcessPayment(decimal amount, PaymentDetails details);
}

public class CreditCardStrategy : IPaymentStrategy
{
    public PaymentResult ProcessPayment(decimal amount, PaymentDetails details)
    {
        Console.WriteLine($"Processing {amount:C} via Credit Card");
        // Credit card logic
        return PaymentResult.Success();
    }
}

public class PayPalStrategy : IPaymentStrategy
{
    public PaymentResult ProcessPayment(decimal amount, PaymentDetails details)
    {
        Console.WriteLine($"Processing {amount:C} via PayPal");
        // PayPal logic
        return PaymentResult.Success();
    }
}

public class CryptoStrategy : IPaymentStrategy
{
    public PaymentResult ProcessPayment(decimal amount, PaymentDetails details)
    {
        Console.WriteLine($"Processing {amount:C} via Cryptocurrency");
        // Crypto logic
        return PaymentResult.Success();
    }
}

public class PaymentContext
{
    private IPaymentStrategy _strategy;
    
    public PaymentContext(IPaymentStrategy strategy)
    {
        _strategy = strategy;
    }
    
    public void SetStrategy(IPaymentStrategy strategy)
    {
        _strategy = strategy;
    }
    
    public PaymentResult ProcessPayment(decimal amount, PaymentDetails details)
    {
        return _strategy.ProcessPayment(amount, details);
    }
}

// Kullanım
var paymentContext = new PaymentContext(new CreditCardStrategy());
paymentContext.ProcessPayment(100m, paymentDetails);

// Runtime'da strateji değiştirme
paymentContext.SetStrategy(new PayPalStrategy());
paymentContext.ProcessPayment(200m, paymentDetails);
```

**Command Pattern Örneği:**
```csharp
public interface ICommand
{
    void Execute();
    void Undo();
}

public class CreateFileCommand : ICommand
{
    private readonly string _fileName;
    private bool _executed = false;
    
    public CreateFileCommand(string fileName)
    {
        _fileName = fileName;
    }
    
    public void Execute()
    {
        File.Create(_fileName);
        _executed = true;
        Console.WriteLine($"File {_fileName} created");
    }
    
    public void Undo()
    {
        if (_executed && File.Exists(_fileName))
        {
            File.Delete(_fileName);
            Console.WriteLine($"File {_fileName} deleted");
        }
    }
}

public class DeleteFileCommand : ICommand
{
    private readonly string _fileName;
    private string _fileContent;
    private bool _executed = false;
    
    public DeleteFileCommand(string fileName)
    {
        _fileName = fileName;
    }
    
    public void Execute()
    {
        if (File.Exists(_fileName))
        {
            _fileContent = File.ReadAllText(_fileName);
            File.Delete(_fileName);
            _executed = true;
            Console.WriteLine($"File {_fileName} deleted");
        }
    }
    
    public void Undo()
    {
        if (_executed)
        {
            File.WriteAllText(_fileName, _fileContent);
            Console.WriteLine($"File {_fileName} restored");
        }
    }
}

public class FileManager
{
    private readonly Stack<ICommand> _commandHistory = new();
    
    public void ExecuteCommand(ICommand command)
    {
        command.Execute();
        _commandHistory.Push(command);
    }
    
    public void Undo()
    {
        if (_commandHistory.Count > 0)
        {
            var command = _commandHistory.Pop();
            command.Undo();
        }
    }
}

// Kullanım
var fileManager = new FileManager();
fileManager.ExecuteCommand(new CreateFileCommand("test.txt"));
fileManager.ExecuteCommand(new DeleteFileCommand("test.txt"));
fileManager.Undo(); // Delete'i geri al
fileManager.Undo(); // Create'i geri al
```

---

### `5. Repository Pattern nedir ve Unit of Work ile ilişkisi nedir?`

**Cevap:** 
Repository Pattern, veri erişim katmanını soyutlayarak business logic'i database detaylarından ayıran pattern'dir. Unit of Work ile birlikte transaction yönetimi sağlar.

**Kaynak:** 
- **Data Access Abstraction**: Veri erişimini soyutlama
- **Testability**: Mock repository ile test kolaylığı
- **Unit of Work**: Transaction boundary yönetimi
- **Generic Repository**: Ortak CRUD işlemleri

**Örnek:**
```csharp
// Generic Repository Interface
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task AddAsync(T entity);
    void Update(T entity);
    void Remove(T entity);
}

// Specific Repository Interface
public interface IUserRepository : IRepository<User>
{
    Task<User> GetByEmailAsync(string email);
    Task<IEnumerable<User>> GetActiveUsersAsync();
    Task<bool> EmailExistsAsync(string email);
}

// Repository Implementation
public class UserRepository : IUserRepository
{
    private readonly DbContext _context;
    
    public UserRepository(DbContext context)
    {
        _context = context;
    }
    
    public async Task<User> GetByIdAsync(int id)
    {
        return await _context.Set<User>().FindAsync(id);
    }
    
    public async Task<IEnumerable<User>> GetAllAsync()
    {
        return await _context.Set<User>().ToListAsync();
    }
    
    public async Task<IEnumerable<User>> FindAsync(Expression<Func<User, bool>> predicate)
    {
        return await _context.Set<User>().Where(predicate).ToListAsync();
    }
    
    public async Task AddAsync(User entity)
    {
        await _context.Set<User>().AddAsync(entity);
    }
    
    public void Update(User entity)
    {
        _context.Set<User>().Update(entity);
    }
    
    public void Remove(User entity)
    {
        _context.Set<User>().Remove(entity);
    }
    
    public async Task<User> GetByEmailAsync(string email)
    {
        return await _context.Set<User>().FirstOrDefaultAsync(u => u.Email == email);
    }
    
    public async Task<IEnumerable<User>> GetActiveUsersAsync()
    {
        return await _context.Set<User>().Where(u => u.IsActive).ToListAsync();
    }
    
    public async Task<bool> EmailExistsAsync(string email)
    {
        return await _context.Set<User>().AnyAsync(u => u.Email == email);
    }
}

// Unit of Work Interface
public interface IUnitOfWork : IDisposable
{
    IUserRepository Users { get; }
    IOrderRepository Orders { get; }
    IProductRepository Products { get; }
    
    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitTransactionAsync();
    Task RollbackTransactionAsync();
}

// Unit of Work Implementation
public class UnitOfWork : IUnitOfWork
{
    private readonly DbContext _context;
    private IDbContextTransaction _transaction;
    
    public UnitOfWork(DbContext context)
    {
        _context = context;
        Users = new UserRepository(_context);
        Orders = new OrderRepository(_context);
        Products = new ProductRepository(_context);
    }
    
    public IUserRepository Users { get; }
    public IOrderRepository Orders { get; }
    public IProductRepository Products { get; }
    
    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }
    
    public async Task BeginTransactionAsync()
    {
        _transaction = await _context.Database.BeginTransactionAsync();
    }
    
    public async Task CommitTransactionAsync()
    {
        await _transaction?.CommitAsync();
    }
    
    public async Task RollbackTransactionAsync()
    {
        await _transaction?.RollbackAsync();
    }
    
    public void Dispose()
    {
        _transaction?.Dispose();
        _context?.Dispose();
    }
}

// Service Layer with Repository Pattern
public class UserService
{
    private readonly IUnitOfWork _unitOfWork;
    
    public UserService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }
    
    public async Task<User> CreateUserAsync(CreateUserRequest request)
    {
        // Business logic validation
        if (await _unitOfWork.Users.EmailExistsAsync(request.Email))
        {
            throw new BusinessException("Email already exists");
        }
        
        var user = new User
        {
            Name = request.Name,
            Email = request.Email,
            CreatedAt = DateTime.UtcNow,
            IsActive = true
        };
        
        await _unitOfWork.Users.AddAsync(user);
        await _unitOfWork.SaveChangesAsync();
        
        return user;
    }
    
    public async Task TransferUserDataAsync(int fromUserId, int toUserId)
    {
        await _unitOfWork.BeginTransactionAsync();
        
        try
        {
            var fromUser = await _unitOfWork.Users.GetByIdAsync(fromUserId);
            var toUser = await _unitOfWork.Users.GetByIdAsync(toUserId);
            
            // Transfer orders
            var orders = await _unitOfWork.Orders.FindAsync(o => o.UserId == fromUserId);
            foreach (var order in orders)
            {
                order.UserId = toUserId;
                _unitOfWork.Orders.Update(order);
            }
            
            // Deactivate old user
            fromUser.IsActive = false;
            _unitOfWork.Users.Update(fromUser);
            
            await _unitOfWork.SaveChangesAsync();
            await _unitOfWork.CommitTransactionAsync();
        }
        catch
        {
            await _unitOfWork.RollbackTransactionAsync();
            throw;
        }
    }
}

// Dependency Injection Registration
services.AddScoped<IUserRepository, UserRepository>();
services.AddScoped<IOrderRepository, OrderRepository>();
services.AddScoped<IProductRepository, ProductRepository>();
services.AddScoped<IUnitOfWork, UnitOfWork>();
services.AddScoped<UserService>();
```