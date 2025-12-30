# Task { ... } Ne İşe yarar? Task-await Vs Completion Handler/@escaping Farkı ve Detayları Nedir ?

Senkron bir fonksiyon içinde asenkron kod çalıştırmayı sağlayan bir **concurrency** yapıdır.
- **Ne işe yarar?** Mevcut kodun akışını bozmadan yeni bir asenkron kapsam oluşturur ve içindeki işlemleri arka planda (çakışma olmadan) başlatır.
- **Ne zaman geldi?** 2021 yılında **iOS 15** (Swift 5.5) ile birlikte hayatımıza girdi.
- **Neyin yerine geldi?** Eskiden kullanılan karmaşık ve okunması zor olan **Completion Handler (Closure)** yapılarının ve manuel `DispatchQueue.main.async` veya `DispatchQueue.global().async` kullanımlarının modern alternatifi olarak geldi.
****

| **Özellik** | **Eski Yöntem (GCD)** | **Yeni Yöntem (Swift Concurrency)** |
| --- | --- | --- |
| **Yazım** | `DispatchQueue.global().async { ... }` | `Task { ... }` |
| **Hata Yönetimi** | `Result<T, Error>` ile manuel | `try / catch` ile doğal |
| **Okunabilirlik** | Callback Hell (İç içe bloklar) | Temiz kod |

NOT: `Task` kullandığında "ana thread'i bloklamamış" olursun. İşlem, sistemin o an en uygun gördüğü thread üzerinde (genellikle arka planda) gerçekleştirilir.

1. `Task { ... }` dediğin anda, içindeki `await` satırına gelindiğinde mevcut thread (Main Thread) "askıya alınır". Bu sayede UI donmaz, kullanıcı **butonlara basabilir** veya o meşhur **loading ikonunu** görebilir.
2. Sen arka planda API'den cevabı beklerken, sistem thread'i başka işler için kullanır.
3. API cevabı geldiğinde, `Task` kaldığı yerden devam eder ve genellikle otomatik olarak Main Thread'e geri dönerek UI'ı güvenle günceller.

### Küçük Bir Uyarı: `@MainActor`

Eğer UI güncelleyeceksen, kullandığın fonksiyonun veya sınıfın (örneğin ViewModel) başında `@MainActor` olmasına dikkat etmelisin. Bu, `Task` bittiğinde sonucun güvenli bir şekilde ana thread'e iletilmesini sağlar.

```swift
@MainActor // Sınıfın tamamı artık Main Thread garantisi altında
final class HomeViewModel: HomeViewModelProtocol {
    
    private let client: NetworkClientProtocol
    var characters: [Character] = []
    
    weak var delegate: HomeViewModelDelegate?
    
    init(client: NetworkClientProtocol = NetworkClient()) {
        self.client = client
    }
    
    // FETCH
    func fetchCharacters() {
        // 1. ADIM: Şu an Main Thread (Ana İş Parçacığı) üzerindeyiz. UI'a "Yükleniyor" bilgisini veriyoruz.
        delegate?.didChangeState(to: .loading)
        
        // 2. ADIM: Bir 'Task' (Görev) oluşturuyoruz.Bu, sistemden "bana asenkron çalışabileceğim bir alan aç" istemektir.
        Task {
            // 3. ADIM: 'await' kelimesi "Süspansiyon (Askıya Alma) Noktası"dır.Sistem burada durur, network isteğini Background Thread gönderir.
            // ÖNEMLİ: Bu sırada **Main Thread SERBEST kalır** -> Kullanıcı ekranı kaydırabilir, butonlara basabilir; UI asla DONMAZ.
            let result = await client.request(RickAndMortyEndpoint.characters, type: CharacterResponse.self)
            
            // 4. ADIM: Network cevabı geldi! @MainActor işareti sayesinde, sistem otomatik olarak bizi tekrar Main Thread'e (Ana İş Parçacığına) geri taşır.
            switch result {
            case .success(let response):
                // 5. ADIM: Veri başarıyla geldi. Ana thread'de olduğumuz için UI elemanlarını besleyecek dataları güvenle güncelliyoruz.
                self.characters = response.results
                self.delegate?.didChangeState(to: .success)
                
            case .failure(let error):
                // 6. ADIM: Hata durumunda yine Ana Thread'de kalarak kullanıcıya hata mesajını gösteriyoruz.
                self.delegate?.didChangeState(to: .error(error.localizedDescription))
            }
        }
        **// NOT: Task bloğu içindeki 'await' beklerken, kodun geri kalanı (varsa Task dışındaki alt satırlar) akmaya devam eder.**
    }
}

```

Tam olarak:

Senaryoyu teknik olarak adım adım netleştirelim:

1. **`Task { ... }` ->** "Burada asenkron bir iş başlayacak, ana thread'i serbest bırakın" der.
2. **`await apiCall()`:** Veri arka planda (background thread’de) çekilirken uygulama donmaz. 
3. **`MainActor.run { ... }`:** "İşim bitti, elimde veri var; şimdi güvenli bölgeye (Main Thread’e) dönüp ekranı güncelleyebilirim" demektir.

### Küçük Bir İpucu: Modern Yöntem

Eğer iOS 15+ ile çalışıyorsan ve bir `ViewModel` kullanıyorsan, sürekli `MainActor.run` yazmak yerine **ViewModel'ın en başına** `@MainActor` eklersin.

Böylece Swift şunu der: *"Bu sınıftaki her şey zaten Main Thread'de çalışsın, ben zahmet etmeyeyim."*

**Farkı şöyle görebilirsin:**

- **Eski/Manuel Yol:**Swift
    
    ```swift
    Task {
        let users = await api.fetch() // Background
        await MainActor.run {
            self.users = users // Main Thread'e zorla soktuk
        }
    }
    ```
    
- **Modern/Temiz Yol (@MainActor ile):**Swift
    
    ```swift
    @MainActor
    class UserViewModel: ObservableObject {
        @Published var users: [User] = []
    
        func load() {
            Task {
                // @MainActor sayesinde burası bittiğinde otomatik olarak Main Thread'de kalır.
                self.users = await api.fetch() 
            }
        }
    }
    ```
    

```swift
@MainActor // <--- BU ETİKET: "Burası benim bölgem (Main Thread)" diyor.
func verileriGetir() {
    
    Task {
        // 1. ADIM: "await" kelimesi ile sistem arka plana gider, veriyi bekler. O sırada Main Thread serbesttir, UI donmaz.
        let gelenVeri = await apiService.fetch() 
        
        // 2. ADIM Alttakı kısım: Veri geldiği an, @MainActor devreye girer:
        **// "Hop! Veri geldi, background thread'den çık, DERHAL Main Thread'e dön."**
        self.ekraniGuncelle(gelenVeri) 
    }
}
```

### Eskiden (Completion Handler):

- Veri arka planda gelir, sonra sen o veriyi bir kutuya koyup `completion(veri)` diyerek fonksiyonun dışına "fırlatırdık".
- Eğer 3 tane API isteğini sırayla yapman gerekirse, kodlar sağa doğru kayar, okunmaz hale gelirdi.
- **Thread Karmaşası:** İçeride `DispatchQueue.main.async` yazmayı unutursan uygulama patlardı.

### Şimdi (Task & await):

- **Sıralı Okuma:** Kod sanki normal, düz bir kodmuş gibi yukarıdan aşağıya okunur.
- **Bekleme:** `await` satırında kod "durur" ama uygulamayı dondurmaz. Cevap gelince alt satıra geçer.
- **Otomatik Dönüş:** `@MainActor` varsa, artık `completion` içinden ana thread'e zıplamaya çalışmakla uğraşmazsın.

| **Özellik** | **Eskiden (Completion)** | **Şimdi (Task / await)** |
| --- | --- | --- |
| **Akış** | Fonksiyon biter, cevap sonra "fırlar". | Fonksiyon "bekler", cevap gelince devam eder. |
| **Hata** | `Result<Data, Error>` ile manuel kontrol. | `try / catch` ile tertemiz yönetim. |
| **Görünüm** | İç içe geçmiş parantezler (`{ { { } } }`). | Düz, alt alta satırlar. |

### Eski Yöntem: Completion Handler (Closure) ve GCD

```swift
// ESKİ YÖNTEM
func fetchCharactersOldWay() {
    // 1. ADIM: Main Thread'de loading bilgisini veriyoruz.
    delegate?.didChangeState(to: .loading)

    // 2. ADIM: Network katmanına bir 'Escaping Closure' geçiyoruz. Fonksiyon burada biter ama request arkada devam eder.
																										// ▼▼▼ İŞTE ESCAPING CLOSURE BURASI (BAŞLANGIÇ) ▼▼▼
    client.request(RickAndMortyEndpoint.characters) { [weak self] (result: Result<CharacterResponse, Error>) in
        
        // 3. ADIM: Network isteği bitti. Şu an 'Background Thread' (Arka Plan) üzerindeyiz!
        // Burada sakın UI güncelleme yapma, uygulama çöker veya garip davranır.
        
        // 4. ADIM: Elle Main Thread'e (Ana İş Parçacığı) zıplamamız gerekiyor.
        DispatchQueue.main.async {
            
            // 5. ADIM: 'self' hala hafızada mı kontrol etmeliyiz (Memory Leak önlemi).
            guard let self = self else { return }
            
            switch result {
            case .success(let response):
                self.characters = response.results
                self.delegate?.didChangeState(to: .success)
                
            case .failure(let error):
                self.delegate?.didChangeState(to: .error(error.localizedDescription))
            }
        }
    }
    // ▲▲▲ İŞTE ESCAPING CLOSURE BURASI (BİTİŞ) ▲▲▲
}
```

1. **Escaping Closure Nerede?**: Kodun içindeki `{ [weak self] ... }` süslü parantezlerinin tamamı.
2. **Neden `@escaping` yazmıyor?**: Çünkü o kelime bu fonksiyonu **çağırdığın** yerde değil, **tanımladığın** yerde (Network katmanında) yazar.

### 1. Escaping Closure Tam Olarak Neresi?

Yazdığımız`fetchCharactersOldWay` fonksiyonundaki şu blok, **Closure**'ın ta kendisidir:

```swift
    // ▼▼▼ İŞTE ESCAPING CLOSURE BURASI (BAŞLANGIÇ) ▼▼▼
    { [weak self] (result: Result<CharacterResponse, Error>) in
        
        // Bu kodlar hemen çalışmaz! Network isteği bitip cevap gelince (belki 2 saniye sonra) çalışır.
        DispatchQueue.main.async {
            guard let self = self else { return }
            // ... işlemler ...
        }
    }
    // ▲▲▲ İŞTE ESCAPING CLOSURE BURASI (BİTİŞ) ▲▲▲
```

### 2. Neden "Escaping" Deniyor?

Olayın mantığı şu:

- Normalde bir fonksiyon (`fetchCharactersOldWay`) bittiğinde içindeki her şey hafızadan silinir.
- Ama burada sen Network katmanına diyorsun ki: **"Al bu kod bloğunu (closure), cebine koy. Ben işimi bitirip gitsem bile, sen internetten cevap gelince bunu çalıştır."**
- Yani bu kod bloğu, fonksiyonun yaşam alanından **KAÇIYOR (Escaping)** ve daha sonra çalışmak üzere hafızada hayatta kalıyor.

### 3. `@escaping` Yazısı Nerede Gizli?

Eğerki`HomeViewModel` içinde bu kelimeyi görmezsek  Bu bir yerlerde kullanmıştır.

Örnek olarak: (Kendi Network kütüphanem içinden)

```swift
protocol NetworkClientProtocol {
    func request<T: Decodable>(
        _ endpoint: Endpoint, 
        completion: @escaping (Result<T, Error>) -> Void // <-- İŞTE BURADA!
    )
}
```

### Özetle Ne Oldu?

1. `fetchCharactersOldWay` fonksiyonunu çalıştırdın.
2. `client.request` satırına geldi, isteği başlattı ve `{...}` bloğunu (Closure) network kütüphanesine teslim etti.
3. `fetchCharactersOldWay` o milisaniyede **BİTTİ**. (Stack'ten silindi).
4. **Closure →** O süslü parantez içindeki kodlar, fonksiyon bitmesine rağmen "kaçtı" ve arka planda veri gelmesini beklemeye başladı.
5. **Sonuç:** Veri gelince o kodlar çalıştı.

`async/await` (yeni yöntem) kullanınca bu "kaçma/yakalama" ve "completion" dertleri tamamen bitiyor, kod yukarıdan aşağıya dümdüz akıyor. O yüzden yeni yöntem çok daha temiz.

### Eskiden Neden Daha Zordu? (Farklar)

Eski sistem ile yeni sistem arasındaki temel farkları şu tabloda görebilirsin:

| **Özellik** | **Eski Yöntem (GCD & Completion)** | **Yeni Yöntem (@MainActor & Async)** |
| --- | --- | --- |
| **Thread Yönetimi** | `DispatchQueue.main.async` ile manuel. | `@MainActor` ile otomatik ve derleyici garantili. |
| **Hafıza Yönetimi** | `[weak self]` ve `guard let` şart. | `Task` içinde genellikle daha güvenli ve temiz. |
| **Hata Yönetimi** | Callback içinde unutulabilir, yönetmesi zor. | `try-catch` veya `Result` ile lineer akış. |
| **Okunabilirlik** | İç içe süslü parantezler | Düz bir yazı okur gibi. |

## **Daha net görmek ve özetlemek için örneğe bakalım:**

### 1. Completion Handler (Eski Usül: Kodun Zıplaması)

- Sen API isteğini başlatırsın, Swift süslü parantez `{}` içine girmez **ATLAR (süslü parantez dışından)** alt satıra geçer.
- Yani; Kod çalışırken derleyici o süslü parantez `{ ... }` bloğunu görür, "Burası veriyi alınca çalıştırılacak bir paket" der, 
o paketi hafızaya atar ama **içine girmez (çalıştırmaz).** Hemen parantezin kapandığı yerden sonraki satıra (alt satıra) zıplar ve orayı çalıştırır. (Saniyeler sonra) Veri gelince `{}` bloğunun içine girilir.

Bunu sıralarsak:

1. Fonksiyon çağrılır.
2. `{}` bloğu **ATLANIR**. (Burası hafızaya atılır)
3. Fonksiyonun altındaki satırlar **HEMEN** çalışır.
4. (Saniyeler sonra) Veri gelince `{}` bloğunun içine girilir.

**Completion handler gerçek dünya  kod parçası:**

```swift
func butonTiklandi() {
    print("--- Butona basıldı ---")
    
    // Fonksiyonu çağırıyoruz
    NetworkManager.shared.fetchUserProfile(userId: 101) { result in
        // --- BURASI COMPLETION BLOĞU (GELECEKTE ÇALIŞACAK) ---
        **// Buraya ancak internetten cevap dönünce girilir.**
        switch result {
        case .success(let user):
            print("👤 Ekrana yazılıyor: \(user.fullName)")
        case .failure(let error):
            print("⚠️ Hata oluştu: \(error)")
        }
    }
    
    // --- KRİTİK NOKTA: Burası hemen çalışır ---
    print("--- Fonksiyonun altındaki kodlar çalışmaya devam ediyor ---")
    print("--- UI Donmadı, loading animasyonu dönüyor ---")
}
```

```swift
class NetworkManager {
    
    // --- COMPELTION HANDLER FONKSİYONU ---
    // @escaping NEDEN VAR? 
    // Çünkü -> 'completion' bloğu, fonksiyonun çalışması bittikten ÇOK SONRA çağrılacak.
    // Hafızada tutulması ve fonksiyon scope'undan "kaçmasına" izin verilmesi gerekiyor.
    
    func fetchUserProfile(userId: Int, completion: @escaping (Result<UserProfile, APIError>) -> Void) {
        
        // 1. URL Kontrolü
        guard let url = URL(string: "https://api.ornek.com/users/\(userId)") else {
            completion(.failure(.invalidURL))
            return
        }
        
        print("🚀 İstek başlatılıyor...") // Step A
        
        // 2. URLSession Task Oluşturma
        let task = URLSession.shared.dataTask(with: url) { data, response, error in
            
            // --- BURASI ASYNC OLARAK ÇALIŞIR (Sonradan gelir) ---
            
            // Hata kontrolü
            if let _ = error {
                DispatchQueue.main.async { completion(.failure(.serverError("Bağlantı hatası"))) }
                return
            }
            
            // Response kodu kontrolü (200 OK mi?)
            guard let httpResponse = response as? HTTPURLResponse, 
                  (200...299).contains(httpResponse.statusCode) else {
                DispatchQueue.main.async { completion(.failure(.serverError("Sunucu hatası"))) }
                return
            }
            
            // Data kontrolü
            guard let data = data else {
                DispatchQueue.main.async { completion(.failure(.noData)) }
                return
            }
            
            // JSON Decode İşlemi
            do {
                let user = try JSONDecoder().decode(UserProfile.self, from: data)
                
                // ✅ BAŞARILI! 
                // UI güncelleyeceği için mutlaka Main Thread'e dönüyoruz.
                DispatchQueue.main.async {
                    print("✅ Veri geldi ve işlendi!") // Step C
                    completion(.success(user))
                }
            } catch {
                DispatchQueue.main.async { completion(.failure(.decodingError)) }
            }
        }
        
        // 3. İsteği Tetikle
        task.resume()
        
        print("⌛️ İstek kuyruğa atıldı, fonksiyon bitiyor (Return).") // Step B
    }
}
```

### Kodun Çalışma Sırası

1. `-- Butona basıldı ---`
2. `🚀 İstek başlatılıyor...`
3. `⌛️ İstek kuyruğa atıldı, fonksiyon bitiyor (Return).`
4. `-- Fonksiyonun altındaki kodlar çalışmaya devam ediyor ---` (Buton tıklandı fonksiyonunun devamı)
5. `-- UI Donmadı, loading animasyonu dönüyor ---`
6. *(Burada 1-2 saniye boşluk/bekleme olur)*
7. `✅ Veri geldi ve işlendi!`

### Neden `@escaping` kullandık?

1. `fetchUserProfile` fonksiyonu çalışır, `task.resume()` der ve **BİTER (Return eder).**
2. Ama bizim `completion` bloğumuz henüz çalışmadı! O bloğun internetten cevap gelene kadar **hafızada canlı kalması** lazım.
3. İşte bu yüzden derleyiciye diyoruz ki: *"Hey Swift, bu closure fonksiyonun yaşam döngüsünden **KAÇACAK (Escape)** ve daha sonra (asenkron olarak) çağrılacak. Onu hafızadan silme!"*

Eğer `@escaping` yazmazsan, Xcode sana şu hatayı verir: `Escaping closure captures non-escaping parameter 'completion'`

---

### 2. Task & Await (Yeni usül)

- `Task` bloğunu yazdığında, Swift süslü parantez `{}` içine girip o işin bitmesini **BEKLEMEZ**. O bloğu ana akıştan **KOPARIR** (ayırır) ve hemen süslü parantez dışındaki alt satıra geçer.
- Yani; Kod çalışırken derleyici o `Task { ... }` bloğunu görür, "Tamam, bu arkadaşın acelesi yok, bu asenkron bir paket, bunu **yan şeride** alıyorum" der. O paketi çalışması için sisteme teslim eder ama bitmesini beklemez. Hemen parantezin kapandığı yerden sonraki satıra (alt satıra) zıplar ve orayı çalıştırır. `Task`'ın içi ise o sırada (milisaniyeler içinde) kendi başına çalışmaya başlar.

**Bunu sıralarsak:**

1. Fonksiyon çağrılır.
2. `Task {}` bloğu görülür. (Burası ana yoldan ayrılır/fırlatılır).
3. Fonksiyonun altındaki satırlar (Task'ın dışı) **HEMEN** çalışır.
4. (Eş zamanlı olarak) `Task {}` bloğunun **içi** çalışmaya başlar.
5. Task'ın içinde `await` görülünce o "yan şerit" donar, veri gelince kaldığı yerden devam eder.

**Task & await gerçek dünya  kod parçası:**

```swift

class NewsViewController: UIViewController {
    let viewModel = NewsViewModel()
    
    @IBAction func buttonTapped() {
        print("--- Butona basıldı ---")
        
        viewModel.haberleriYenile()
        
        print("--- Fonksiyonun altındaki kodlar çalışmaya devam ediyor ---")
        print("--- UI Donmadı, loading animasyonu dönüyor ---")
    }
}
```

```swift
@MainActor
class NewsViewModel {
    
    var onNewsUpdated: ((String) -> Void)?
    
    func haberleriYenile() {
        print("🚀 [VM] İstek başlatılıyor...") 
        
        // --- TASK BAŞLATMA ANI ---
        // Derleyici burayı görür. "Bu asenkron, yan şeride at" der.
        // İçine GİRMEZ.
        Task {
            // --- YAN ŞERİT (Milisaniyeler sonra başlar) ---
            print("⚡️ [Task] Yan şerit çalışmaya başladı (Task içi)")
            
            // FREN ANI (Await)
            // Sadece burası donar.
            let news = await NetworkManager.shared.fetchNews()
            
            print("✅ [Task] Veri geldi ve işlendi!")
            self.onNewsUpdated?(news)
        }
        
        // --- ZIPLAMA ANI ---
        print("⌛️ [VM] Task kuyruğa atıldı, fonksiyon bitiyor (Return).")
    }
}
```

### Kodun Çalışma Sırası (Konsol Çıktısı)

Aşağıdaki sıraya dikkat et, **Zıplama** olayını net göreceksin:

```swift
-- Butona basıldı ---
🚀 [VM] İstek başlatılıyor...
⌛️ [VM] Task kuyruğa atıldı, fonksiyon bitiyor (Return).
--- Fonksiyonun altındaki kodlar çalışmaya devam ediyor ---
--- UI Donmadı, loading animasyonu dönüyor ---
(Burada milisaniyeler sonra Task uyanır)
⚡️ [Task] Yan şerit çalışmaya başladı (Task içi)
(Burada 1-2 saniye boşluk/bekleme olur - Await anı)
✅ [Task] Veri geldi ve işlendi!
```

### Neden `Task { ... }` kullandık?

**Completion Handler'da `@escaping` diyorduk, burada mantık ne?**

Fonksiyon (`haberleriYenile`) çalışır, `Task` satırına gelir ve **BİTER (Return eder).**

Ama bizim Task bloğumuzun içi henüz çalışmadı bile! O bloğun arka planda kendi yaşamına başlaması lazım.

İşte bu yüzden derleyiciye diyoruz ki:
**"Hey Swift, bu süslü parantez içindeki kod bloğunu (`{...}`) şimdiki akıştan KOPAR. Ben yoluma devam edeceğim, sen bu bloğu müsait olduğunda (hemen veya milisaniyeler sonra) ayrı bir iş olarak çalıştır."**

Eğer `Task` içine almasaydın:
Swift kodu satır satır okumaya çalışırdı ve `await` (bekleme) komutunu gördüğü an **bütün uygulamayı (UI dahil)** dondurmak zorunda kalırdı. `Task`, o bekleme işlemini "yan şeride" alarak ana yolun (UI'ın) akmasını sağlar.

işte bu kadar. Artık completion nedir, task neden vardır, hangisinde ne kullanacağım durumunu güzelce netleştirdik. Umarım net bir şekilde anlatabilmişimdir. Benim en çok hoşuma gideni ise: Task çünkü callback hell çilesinden kurtarıyor.
Bu yazı şimdilik bu kadar, iyi günler 🤘👋🏻
