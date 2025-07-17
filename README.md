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