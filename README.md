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

