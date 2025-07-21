# Go (Golang) Interview Hazırlık Notları

Bu dosya, Go programlama dili ile ilgili interview hazırlık notlarını içerir. Yüksek trafikli sistemlerde kullanılan Go konuları soru-cevap formatında düzenlenmiştir.

---

## 📝 Soru & Cevaplar

### `Go'da Concurrency ve Parallelism arasındaki fark nedir?`

**Cevap:** 
- **Concurrency**: Birden fazla işlemin aynı anda yönetilmesi (aynı anda başlatılması, durdurulması, devam ettirilmesi)
- **Parallelism**: Birden fazla işlemin gerçekten aynı anda çalışması (çoklu CPU çekirdekleri)

**Detaylı Açıklama:**

**Concurrency (Eşzamanlılık):**
- Birden fazla işlemin aynı anda yönetilmesi
- İşlemler aynı anda başlatılabilir, durdurulabilir, devam ettirilebilir
- Tek CPU çekirdeğinde bile concurrency mümkündür
- Go'da goroutine'ler ile sağlanır
- İşlemler arasında geçiş yapılır (context switching)

**Parallelism (Paralellik):**
- Birden fazla işlemin gerçekten aynı anda çalışması
- Çoklu CPU çekirdekleri gerektirir
- Her işlem farklı çekirdekte çalışır
- Go'da GOMAXPROCS ile kontrol edilir
- Gerçek paralel çalışma

**Farklar:**
- Concurrency: İşlemlerin yönetimi (management)
- Parallelism: İşlemlerin çalışması (execution)
- Concurrency olmadan parallelism mümkün değildir
- Concurrency tek çekirdekte, parallelism çok çekirdekte

**Go'da Nasıl Çalışır:**
- Go scheduler, goroutine'leri OS thread'leri arasında dağıtır
- M:N scheduling modeli (M goroutine, N OS thread)
- GOMAXPROCS ile kaç OS thread kullanılacağı belirlenir
- Blocking işlemlerde otomatik context switching

**Kod Örneği:**
```go
package main

import (
    "fmt"
    "runtime"
    "sync"
    "time"
)

// CPU yoğun işlem (parallelism için uygun)
func cpuIntensiveTask(id int) {
    for i := 0; i < 1000000; i++ {
        _ = i * i // CPU yoğun hesaplama
    }
    fmt.Printf("CPU Task %d tamamlandı\n", id)
}

// I/O yoğun işlem (concurrency için uygun)
func ioIntensiveTask(id int) {
    time.Sleep(100 * time.Millisecond) // I/O simülasyonu
    fmt.Printf("I/O Task %d tamamlandı\n", id)
}

func main() {
    // CPU çekirdek sayısını ayarla (parallelism kontrolü)
    fmt.Printf("CPU Çekirdek Sayısı: %d\n", runtime.NumCPU())
    runtime.GOMAXPROCS(4) // 4 OS thread kullan
    
    var wg sync.WaitGroup
    
    fmt.Println("=== Concurrency Örneği (I/O Yoğun) ===")
    start := time.Now()
    
    // 10 I/O yoğun goroutine (concurrency)
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            ioIntensiveTask(id)
        }(i)
    }
    
    wg.Wait()
    fmt.Printf("I/O işlemleri tamamlandı: %v\n", time.Since(start))
    
    fmt.Println("\n=== Parallelism Örneği (CPU Yoğun) ===")
    start = time.Now()
    
    // 4 CPU yoğun goroutine (parallelism)
    for i := 0; i < 4; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            cpuIntensiveTask(id)
        }(i)
    }
    
    wg.Wait()
    fmt.Printf("CPU işlemleri tamamlandı: %v\n", time.Since(start))
    
    // Goroutine sayısını göster
    fmt.Printf("\nAktif Goroutine Sayısı: %d\n", runtime.NumGoroutine())
}
```

**Pratik Senaryolar:**

**Concurrency Kullanım Alanları:**
- Web server'da birden fazla client isteği
- Database işlemleri
- File I/O operasyonları
- Network istekleri
- UI event handling

**Parallelism Kullanım Alanları:**
- Matematiksel hesaplamalar
- Image/video processing
- Machine learning algoritmaları
- Data analysis
- Scientific computing

**Performance Karşılaştırması:**
- I/O bound işlemlerde concurrency çok etkili
- CPU bound işlemlerde parallelism gerekli
- Go'da her ikisi de goroutine'ler ile sağlanır
- Scheduler otomatik olarak en uygun dağıtımı yapar

---

### `Goroutine nedir ve nasıl çalışır?`

**Cevap:** 
Goroutine, Go runtime tarafından yönetilen hafif thread'lerdir. OS thread'lerinden çok daha az bellek kullanır (2KB başlangıç).

**Kaynak:**
- Go scheduler, goroutine'leri OS thread'leri arasında dağıtır
- M:N scheduling modeli kullanır (M goroutine, N OS thread)
- Blocking işlemlerde otomatik olarak başka goroutine'e geçer

**Kod Örneği:**
```go
package main

import (
    "fmt"
    "time"
)

func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Printf("Worker %d işlemiyor: %d\n", id, j)
        time.Sleep(time.Second)
        results <- j * 2
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)
    
    // 3 worker goroutine başlat
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }
    
    // 9 iş gönder
    for j := 1; j <= 9; j++ {
        jobs <- j
    }
    close(jobs)
    
    // Sonuçları topla
    for a := 1; a <= 9; a++ {
        <-results
    }
}
```

---

### `Channel nedir ve hangi durumlarda kullanılır?`

**Cevap:** 
Channel, goroutine'ler arasında veri paylaşımı ve senkronizasyon için kullanılan tip güvenli bir veri yapısıdır.

**Kaynak:**
- Buffered/Unbuffered channel'lar
- Send/Receive operasyonları blocking olabilir
- Close() ile channel kapatılabilir
- Select ile multiple channel dinlenebilir

**Kod Örneği:**
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Unbuffered channel
    ch1 := make(chan string)
    
    // Buffered channel
    ch2 := make(chan int, 3)
    
    go func() {
        ch1 <- "Merhaba"
        ch2 <- 1
        ch2 <- 2
        ch2 <- 3
        close(ch2)
    }()
    
    // Select ile multiple channel dinleme
    select {
    case msg := <-ch1:
        fmt.Println("ch1'den:", msg)
    case num := <-ch2:
        fmt.Println("ch2'den:", num)
    case <-time.After(2 * time.Second):
        fmt.Println("Timeout!")
    }
    
    // Range ile channel'dan okuma
    for num := range ch2 {
        fmt.Println("Range ile:", num)
    }
}
```

---

### `Deadlock, Race Condition ve Starvation nasıl önlenir?`

**Cevap:** 
- **Deadlock**: İki goroutine'in birbirini beklemesi
- **Race Condition**: Aynı veriye eşzamanlı erişim
- **Starvation**: Bir goroutine'in sürekli bekletilmesi

**Kaynak:**
- Mutex, RWMutex kullanımı
- Channel ile senkronizasyon
- WaitGroup ile bekleme
- Race detection: `go run -race`

**Kod Örneği:**
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type SafeCounter struct {
    mu    sync.RWMutex
    count int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *SafeCounter) GetCount() int {
    c.mu.RLock()
    defer c.mu.RUnlock()
    return c.count
}

func main() {
    counter := &SafeCounter{}
    var wg sync.WaitGroup
    
    // Race condition'ı önlemek için mutex kullan
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter.Increment()
        }()
    }
    
    wg.Wait()
    fmt.Println("Final count:", counter.GetCount())
}
```

---

### `Context paketi ne için kullanılır?`

**Cevap:** 
Context, işlemlerin iptal edilmesi, zaman aşımı ve değer taşınması için kullanılan bir pakettir.

**Kaynak:**
- `context.WithCancel()`: İptal sinyali
- `context.WithTimeout()`: Zaman aşımı
- `context.WithDeadline()`: Belirli zamana kadar
- `context.WithValue()`: Değer taşıma

**Kod Örneği:**
```go
package main

import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context, name string) {
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("%s: İşlem iptal edildi: %v\n", name, ctx.Err())
            return
        default:
            fmt.Printf("%s: Çalışıyor...\n", name)
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    // 3 saniye sonra iptal edilecek context
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    
    go worker(ctx, "Worker1")
    go worker(ctx, "Worker2")
    
    // Ana goroutine bekle
    time.Sleep(5 * time.Second)
}
```

---

### `Interface nedir ve nasıl kullanılır?`

**Cevap:** 
Interface, davranış tanımlayan bir türdür. "Duck typing" prensibi ile çalışır.

**Kaynak:**
- Implicit implementation (açık implementasyon yok)
- Empty interface `interface{}` her türü kabul eder
- Type assertion ile interface'den tip çıkarılabilir

**Kod Örneği:**
```go
package main

import "fmt"

// Interface tanımı
type Animal interface {
    Speak() string
    Move() string
}

// Struct implementasyonu
type Dog struct {
    Name string
}

func (d Dog) Speak() string {
    return "Hav hav!"
}

func (d Dog) Move() string {
    return "Koşuyor"
}

// Başka bir implementasyon
type Bird struct {
    Name string
}

func (b Bird) Speak() string {
    return "Cik cik!"
}

func (b Bird) Move() string {
    return "Uçuyor"
}

// Interface kullanan fonksiyon
func DescribeAnimal(a Animal) {
    fmt.Printf("Konuşma: %s, Hareket: %s\n", a.Speak(), a.Move())
}

func main() {
    dog := Dog{Name: "Karabaş"}
    bird := Bird{Name: "Pamuk"}
    
    DescribeAnimal(dog)
    DescribeAnimal(bird)
    
    // Type assertion
    var animal Animal = dog
    if dog, ok := animal.(Dog); ok {
        fmt.Printf("Bu bir köpek: %s\n", dog.Name)
    }
}
```

---

### `gRPC nedir ve nasıl kullanılır?`

**Cevap:** 
gRPC, Google'ın geliştirdiği yüksek performanslı RPC (Remote Procedure Call) framework'üdür. Protobuf kullanır.

**Kaynak:**
- Protocol Buffers ile veri serileştirme
- HTTP/2 üzerinde çalışır
- Unary, Server/Client streaming, Bidirectional streaming
- Code generation ile client/server kodları üretilir

**Proto Dosyası Örneği:**
```protobuf
syntax = "proto3";

package user;

service UserService {
    rpc GetUser(GetUserRequest) returns (GetUserResponse);
    rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
    rpc StreamUsers(StreamUsersRequest) returns (stream User);
}

message GetUserRequest {
    int32 user_id = 1;
}

message GetUserResponse {
    User user = 1;
}

message CreateUserRequest {
    string name = 1;
    string email = 2;
}

message CreateUserResponse {
    User user = 1;
}

message User {
    int32 id = 1;
    string name = 2;
    string email = 3;
}

message StreamUsersRequest {
    int32 limit = 1;
}
```

**Go Server Örneği:**
```go
package main

import (
    "context"
    "log"
    "net"
    "google.golang.org/grpc"
    pb "path/to/proto"
)

type server struct {
    pb.UnimplementedUserServiceServer
}

func (s *server) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserResponse, error) {
    // Kullanıcı getirme işlemi
    user := &pb.User{
        Id:    req.UserId,
        Name:  "Test User",
        Email: "test@example.com",
    }
    return &pb.GetUserResponse{User: user}, nil
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("Failed to listen: %v", err)
    }
    
    s := grpc.NewServer()
    pb.RegisterUserServiceServer(s, &server{})
    
    log.Printf("Server listening at %v", lis.Addr())
    if err := s.Serve(lis); err != nil {
        log.Fatalf("Failed to serve: %v", err)
    }
}
```

---

### `Go'da error handling nasıl yapılır?`

**Cevap:** 
Go'da error handling explicit'tir. Error'lar değer olarak döndürülür ve kontrol edilmelidir.

**Kaynak:**
- Error interface: `type error interface { Error() string }`
- Custom error types
- Error wrapping (Go 1.13+)
- Panic/recover mekanizması

**Kod Örneği:**
```go
package main

import (
    "errors"
    "fmt"
    "time"
)

// Custom error type
type ValidationError struct {
    Field string
    Value string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("Validation failed for field %s with value %s", e.Field, e.Value)
}

// Error wrapping
func processUser(name string) error {
    if name == "" {
        return fmt.Errorf("processUser: %w", ValidationError{Field: "name", Value: name})
    }
    
    // İşlem başarılı
    return nil
}

// Error handling pattern
func main() {
    if err := processUser(""); err != nil {
        var valErr ValidationError
        if errors.As(err, &valErr) {
            fmt.Printf("Validation error: %v\n", valErr)
        } else {
            fmt.Printf("Other error: %v\n", err)
        }
        return
    }
    
    fmt.Println("İşlem başarılı")
}
```

---

### `Go'da testing nasıl yapılır?`

**Cevap:** 
Go'da testing built-in `testing` paketi ile yapılır. Test dosyaları `_test.go` uzantısı ile biter.

**Kaynak:**
- Unit tests: `go test`
- Benchmark tests: `go test -bench`
- Coverage: `go test -cover`
- Testify library ile assertion'lar

**Kod Örneği:**
```go
package main

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

// Test edilecek fonksiyon
func Add(a, b int) int {
    return a + b
}

// Unit test
func TestAdd(t *testing.T) {
    result := Add(2, 3)
    assert.Equal(t, 5, result, "2 + 3 should equal 5")
}

// Table driven test
func TestAddTable(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 1, 2, 3},
        {"negative", -1, -2, -3},
        {"zero", 0, 5, 5},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            assert.Equal(t, tt.expected, result)
        })
    }
}

// Benchmark test
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(1, 2)
    }
}
```

---

### `Go'da profiling nasıl yapılır?`

**Cevap:** 
Go'da profiling `pprof` paketi ile yapılır. CPU, memory, goroutine profilleri alınabilir.

**Kaynak:**
- `go tool pprof`
- HTTP endpoint ile profiling
- CPU, memory, goroutine profilleri
- Flame graph oluşturma

**Kod Örneği:**
```go
package main

import (
    "log"
    "net/http"
    _ "net/http/pprof"
    "time"
)

func main() {
    // Profiling endpoint'ini başlat
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    
    // CPU yoğun işlem
    for {
        heavyComputation()
        time.Sleep(100 * time.Millisecond)
    }
}

func heavyComputation() {
    // CPU yoğun işlem simülasyonu
    for i := 0; i < 1000000; i++ {
        _ = i * i
    }
}
```

**Profiling Komutları:**
```bash
# CPU profiling
go tool pprof http://localhost:6060/debug/pprof/profile

# Memory profiling
go tool pprof http://localhost:6060/debug/pprof/heap

# Goroutine profiling
go tool pprof http://localhost:6060/debug/pprof/goroutine
```

---

## 📚 Önemli Go Paketleri

### Sync Paketi
- `sync.Mutex`: Mutual exclusion
- `sync.RWMutex`: Read-write mutex
- `sync.WaitGroup`: Goroutine senkronizasyonu
- `sync.Once`: Tek seferlik çalıştırma
- `sync.Pool`: Object pooling

### Context Paketi
- `context.Background()`: Root context
- `context.WithCancel()`: İptal context'i
- `context.WithTimeout()`: Zaman aşımı
- `context.WithValue()`: Değer taşıma

### Net/HTTP Paketi
- HTTP server/client
- Middleware pattern
- Request/Response handling
- JSON encoding/decoding

### Database Paketleri
- `database/sql`: Generic SQL interface
- `github.com/lib/pq`: PostgreSQL driver
- `github.com/jackc/pgx`: PostgreSQL driver
- `go.mongodb.org/mongo-driver`: MongoDB driver

---

## 🔧 Go Best Practices

1. **Error Handling**: Her error'ı kontrol et
2. **Interface Design**: Küçük interface'ler tasarla
3. **Package Structure**: Logical grouping yap
4. **Naming**: Clear ve descriptive isimler kullan
5. **Documentation**: Export edilen her şeyi dokümante et
6. **Testing**: Comprehensive test coverage
7. **Performance**: Benchmark ile ölç
8. **Security**: Input validation, sanitization
9. **Logging**: Structured logging kullan
10. **Monitoring**: Metrics ve health check'ler ekle

---

## 📖 Öğrenme Kaynakları

- [Go Official Documentation](https://golang.org/doc/)
- [Effective Go](https://golang.org/doc/effective_go.html)
- [Go by Example](https://gobyexample.com/)
- [Go Web Examples](https://gowebexamples.com/)
- [Go Concurrency Patterns](https://talks.golang.org/2012/concurrency.slide)
