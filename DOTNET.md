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

### 5. `.NET 6 ile gelen yenilikler nelerdir?`
**Cevap:**
- **Minimal API**: Daha az kod ile hızlı API geliştirme
- **Top-level statements**: Program.cs'de Main metodu olmadan çalışma
- **Hot Reload**: Kod değişikliklerini uygulamayı durdurmadan görme
- **C# 10**: Record structs, global using, file-scoped namespaces
- **Performance iyileştirmeleri**: %20-30 performans artışı
- **Unified platform**: .NET Framework ve .NET Core birleşimi

---

### 6. `.NET 7 ile gelen yenilikler nelerdir?`
**Cevap:**
- **Rate Limiting**: API rate limiting desteği
- **Output Caching**: Response caching middleware
- **C# 11**: Raw string literals, required members, generic attributes
- **Blazor United**: Server ve WebAssembly birleşimi
- **Performance iyileştirmeleri**: Daha hızlı startup ve memory kullanımı
- **Native AOT**: Ahead-of-time compilation desteği

---

### 7. `.NET 8 ile gelen yenilikler nelerdir?`
**Cevap:**
- **Blazor United**: Server ve WebAssembly arasında geçiş
- **C# 12**: Collection expressions, optional parameters, ref readonly
- **Performance iyileştirmeleri**: %20 daha hızlı JSON serialization
- **Native AOT**: Production-ready AOT compilation
- **Time abstraction**: ITimeProvider interface
- **Keyed services**: DI container'da keyed service desteği

---

### 8. `.NET 9 ile gelen yenilikler nelerdir?`
**Cevap:**
- **C# 13**: Primary constructors, collection literals
- **Performance iyileştirmeleri**: Daha hızlı startup ve lower memory usage
- **Enhanced debugging**: Daha iyi debugging deneyimi
- **Cloud-native improvements**: Container ve cloud optimizasyonları
- **Security enhancements**: Güvenlik iyileştirmeleri
- **Developer productivity**: Geliştirici verimliliği artışları

---