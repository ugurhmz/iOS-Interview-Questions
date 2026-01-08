# Architectural Design Patterns & Principles 

<img width="1280" height="880" alt="Architectural Design Patterns   Principles" src="https://github.com/user-attachments/assets/7c854c45-fe0e-4441-84e0-b5e927dc219b" />



<br>
<br>


# Neden Singleton Yerine Dependency Injection Tercih Ederiz?

**1. Test Edilebilirlik (Testability) - En Kritik Nedenimiz**
Bizim için bir kodun kalitesi, test edilebilirliği ile ölçülür.

- **Singleton:** Eğer **NetworkManager.shared** kullanırsak, Unit Test yazarken bu yapıyı **sahtesiyle (Mock) değiştiremeyiz.** Test çalışınca gerçekten internete gitmeye kalkar. Bu da testlerin yavaşlamasına, sunucu hatalarında patlamasına neden olur.
- **DI:** **init(network: NetworkManagerProtocol)** dediğimiz an kontrolü ele alırız. Test ortamında içeriye sahte  bir network nesnesi verip, "İnternet yokmuş gibi davran" diyebiliriz. **DI, bize bu simülasyon gücünü verir.**

**2. Bağımlılıkların Görünürlüğü (Explicit Dependencies)**
Biz bir sınıfın (örneğin **HomeViewModel**) çalışmak için neye ihtiyacı olduğunu **ilk bakışta görmek isteriz.** Bunu şöyle izah edebiliriz;

- **Singletonda:** Sınıfın içine girip satır satır kod okumadan, içeride **AuthService.shared** veya **Database.shared** kullanıldığını anlayamayız. Bu **"Gizli Bağımlılık" (Hidden Dependency)** yaratır.
- **DI (Dependency Injection)da:** **Init** bloğuna bakan herkes, 'Bu sınıfın çalışması için **bağımlılıklarına (Dependencies)** ihtiyacı var der. Yani Kod kendini belgeler."

**3. Global State Tehlikesi (Race Conditions & Side Effects)**
Singleton, uygulama yaşadığı sürece bellekte kalan tek bir örnektir (Instance’tır). Şöyle izah edelim;

- **Singleton:** A ekranında Singleton içindeki bir değişkeni değiştirdiğimizde, bu durum B ekranını da etkiler. Yanlışlıkla önceki oturumdan kalan verilerin yeni ekrana bulaşması gibi **"Yan Etkiler" (Side Effects)** oluşur.
- **DI (Dependency Injection)da:** Her ekran açıldığında (Builder ile) taptaze, sıfır kilometre nesneler oluştururuz. Önceki ekranın kiri pası yeni ekrana bulaşmaz.

**4. Değişime Karşı Esneklik (Loose Coupling - Gevşek Bağlılık)**
Biz yazılımı bugünü kurtarmak için değil, yarın değişebileceğini öngörerek tasarlıyoruz. Şöyleki;

- **Singleton:** Bizi somut bir sınıfa (Concrete Class) kelepçeler. Eğer 500 farklı dosyada **NetworkManager.shared** kullanırsak, yarın "Alamofire'ı kaldırıp Apple'ın kendi URLSession'ına geçelim" dediğimizde, o 500 dosyayı tek tek değiştirmek zorunda kalırız. .OoPs işte bu, projemizin bakımını kabusa çevirir.
- **DI (Dependency Injection):** Bizi somut sınıflara değil, soyut **Protokollere** (Interface) bağlar. Yarın tüm altyapıyı değiştirmek istersek, sadece **Builder** (kurucu) katmanındaki tek bir satırı değiştiririz. Kodun geri kalanı (ViewModel, Repository vb.) altyapının değiştiğini ruhu bile duymaz.

<br>
<br>

# Hangi Durumlar için Singleton tercih etmeliyiz ?

Biz sektörde Singleton desenini tamamen çöpe atmayız. Ancak kullanım alanını çok keskin çizgilerle daraltırız. 

Prensibimiz şudur: **"Business Logic (İş Mantığı) asla Singleton olmamalıdır, ancak Infrastructure (Altyapı) araçları Singleton olabilir."**

### 1.  Logger & Analytics

Bir uygulamanın yüzlerce sınıfı olabilir. Her bir sınıfa **Logger** veya **AnalyticsManager** servisini Dependency Injection ile enjekte etmek, constructor'ları gereksiz yere şişirir (Boilerplate Code).

- Uygulamanın herhangi bir yerinden sadece **"Log atmak"** veya **"Event fırlatmak"** istiyorsunuz. Bu işlemin geri dönüş değeri (Return Value) yok ve uygulamanın akışını değiştirmiyor (Side Effect free).
- **Kullanım:** **Logger.shared.log("Error")** veya **Analytics.shared.track("Button_Clicked")**.
- **Neden Kabul Edilir?** Çünkü bu sınıflar "Fire and Forget" (Ateşle ve Unut) mantığıyla çalışır. Test sırasında mocklanmasalar bile uygulamanın Business Logic testini bozmazlar.

### 2. Apple Ekosistemi ve Sistem Servisleri

Apple, iOS SDK'sının doğası gereği bazı yapıları Singleton olarak tasarlamıştır. Biz de bu yapıları sarmalarken  veya doğrudan kullanırken Singleton yapısına uyarız.

- Örnek vermek gerekirse: **:**
    - **UIApplication.shared** (Uygulama yaşam döngüsü)
    - **NotificationCenter.default** (Observer pattern merkezi)
    - **FileManager.default** (Dosya sistemi erişimi)
    - **UserDefaults.standard** (Basit veri saklama)

Peki Neden: İşletim sistemi, bu kaynakların process başına tek bir instance olacağını garanti eder. Bizim buna karşı çıkmamızın teknik bir anlamı yoktur.

### 3. Read-Only Global Configuration (Salt Okunur Yapılandırma)

Uygulama ayağa kalkarken belirlenen ve çalışma zamanında (Runtime da) asla değişmeyecek olan **sabit veriler için Singleton kullanılabilir.**

- Örnek vermek gerekirse: **BaseURL**, **APIKey**, **AppVersion**, **Environment (Dev/Prod)** gibi bilgilerin tutulduğu bir yapı.
- **Kullanım:** **AppConfig.shared.baseURL**
- **Neden Kabul Edilir?** Çünkü bu veri **Immutable** (Değiştirilemez) durumdadır. "Global State" tehlikesi yaratmaz çünkü state değişmez. A sınıfı da okusa, B sınıfı da okusa aynı veriyi görür. Race Condition  riski yoktur.

### 4. Expensive Resource Management (Pahalı Kaynak Yönetimi - Caching)

Bellek yönetimi açısından oluşturulması maliyetli olan ve uygulamanın yaşamı boyunca ortak bir havuz olarak kullanılması gereken yapılar.

- Örnek vermek gerekirse: Image Cache veya veritabanı bağlantı havuzu.
- **Kullanım:** **ImageCache.shared.getImage(url)**
- **Neden Kabul Edilir?** Burada amaç performanstır. Her ekran için ayrı bir Cache mekanizması kurmak, belleği verimsiz kullanmaya neden olur. Merkezi bir Singleton, bellekteki resimlerin tek bir yerden yönetilmesini ve gerektiğinde topluca temizlenmesini sağlar.
- **(*) Thread Safety  Güvenliği Uyarısı:**
Singleton kullandığımız nadir durumlarda (örneğin **ImageCache** veya **Logger**), bu tekil örneğe farklı thread'lerden aynı anda erişileceğini biliriz. Bu yüzden, Singleton tasarlarken **Swift Actor** yapısını veya **Serial Queue** kullanarak **Thread Safety** (İş parçacığı güvenliği) sağlamak bizim mühendislik sorumluluğumuzdur. Aksi halde Crash kaçınılmazdır.

### ✖️ Ne Zaman KESİNLİKLE Singleton Kullanmayız?

Mülakatlar kırmızı çizgi burasıdır. Şu durumlarda Singleton kullanmak **Anti-Pattern** kabul edilir:

1. **State Management (Durum Yönetimi): ``UserSession.shared, Cart.shared** gibi içinde veri tutan ve bu verinin sürekli değiştiği yapılar Singleton olamaz. Bu, veri tutarlılığını bozar ve Race Condition yaratır.

2. **Networking (Servis Katmanı):** **APIService.shared** demek, test edilebirliği öldürür!!!!. 
Örneğin: Bir ViewModel içinde **NetworkManager.shared.fetch()** çağrısı varsa, o test gerçekten internete çıkmaya çalışır; bu da testlerimizi dış etkenlere (internet hızı, sunucu hatası) bağımlı kılar ve **Mock  veri** ile senaryo üretmemizi imkansız hale getirir. Ayrıca kodumuzu somut sınıfa kelepçeler (**Tight Coupling yapar**); yarın altyapıyı (örn: Alamofire -> URLSession) değiştirmek istediğimizde yüzlerce dosyayı tek tek düzeltmek zorunda kalırız. En kötüsü de, **init** bloğunda görünmediği için **"Gizli Bağımlılık" (Hidden Dependency)** yaratır; sınıfın çalışmak için neye ihtiyacı olduğunu kodun içine girmeden anlayamayız.

3. **Data Passing (Veri Taşıma):** A ekranından B ekranına veri aktarmak için veriyi bir Singleton'a atayıp diğer taraftan okumak amatörce bir yöntemdir.

**Özet Olarak:**
→ Eğer yazdığımız sınıf, uygulamanın **"Nasıl Davrandığını"** (Logic) değiştiriyorsa **DI** kullanmamız lazım. 

→ Eğer sadece **"Ne Yaptığını Kaydediyorsa"** veya **"Sistem Kaynaklarına Erişiyorsa"** (Infrastructure) **Singleton** kullanabiliriz.

<br>
<br>

# Neden Coordinator Pattern  Kullanırız?

Projeye başlarken karşılaştığımız ilk sorun şuydu: **View Controller’lar (HomeVC, DetailVC) çok fazla sorumluluk alıyordu.**
Standart Apple yaklaşımında **HomeVC** içinde **self.navigationController?.pushViewController(DetailVC())** yazdığımız an, HomeVC’yi DetailVC’ye göbekten bağlamış oluyorduk (Tight Coupling).

Şöyleki; HomeVC dediğimiz sınıfın tek derdi, elindeki listeyi/veriyi ekrana basmak olmalı. "Listeden birine tıklanınca Detail mı açılacak, yoksa Login ekranı mı gelecek?" sorusunun cevabı HomeVC'nin sorumluluğunda olmamalı. Eğer bunu VC'nin içine gömersek, yarın öbür gün bu HomeVC'yi başka bir projede veya akışta tekrar kullanamayız.

**İşte Bu yüzden:**
Navigasyon  mantığını, View Controller’ların içinden söküp alıyoruz ve **Coordinator** adını verdiğimiz özel bir yönetici sınıfa devrediyoruz.

### Bunu yaparak neyi sağladık?

1. **Tam Bağımsızlık (Decoupling):**
HomeVC artık DetailVC’nin varlığından bile haberdar değil. Sadece "Kullanıcı şu satıra tıkladı" diye söyler (Delegate ile). O an onu duyan Coordinator, "Tamam, şimdi DetailVC'ye geçişi sağlıyorum" der. **Böylece A ekranını, B ekranına bağımlı kılmadan özgürleştirdik.**

2. **Esneklik ve Modülerlik:**
Yarın Product Manager gelip "Artık tıklayınca Detay açılmasın, Ödeme Ekranı açılsın" dediğinde, HomeVC'nin koduna dokunmak zorunda kalmayacağız. Sadece Coordinator içindeki yönlendirmeyi değiştirmemiz yetecek. Yani asıl a**macımız, değişime direnç göstermeyen, esnek bir yapı kurmaktı.**

3. **Test Edilebilirliği Kolaylaştırmak:**
View Controller’ları test ederken "Navigation Controller var mı, push etti mi?" dertleriyle uğraşmak istemedik. Navigasyon mantığını Coordinator içine izole ettiğimiz için, akış testlerini orada, UI testlerini burada ayrı ayrı yapabilir hale geldik.

**Özet Olarak:**
Biz Coordinator kullanarak; View Controller’ları birer **"Standart bir view"**  haline getirdik, Navigation işini ise Coordinator’a yükledik. Böylece projemiz büyüse bile spagetti koda dönüşmeyen, yönetilebilir bir trafik akışı kurmuş olduk.

<br>
<br>

# Neden SOLID Prensiplerine Bu Kadar Takıntılıyız?

- **Single Responsibility (Tek Sorumluluk - SRP):**
Üzerinde çalıştığımız şey ne ise onun **tekbir sorumluluğu olmalıdır**.
    - Örnek: **NetworkManager** sadece veri çeker, veriyi parse etmez veya UI göstermez. **HomeViewModel** veriyi yönetir, veriyi formatlamaz (Onu Strategy yapar).
    - Eğer bir sınıf 5 farklı iş yapıyorsa, o sınıfta bir bug çıktığında 5 farklı yeri bozma riskimiz vardır. Bu risk alınmaması bizim yararımıza.
- **Open/Closed (Gelişime Açık, Değişime Kapalı - OCP):**
Bir sınıf yeni özellikler eklemeye müsait olmalı, ama var olan kodu değiştirmemeliyiz.
    - Örnek: Projemizdeki **Strategy Pattern**. Yarın "Ölü Karakter" için yeni bir tasarım istendiğinde, **DeadCharacterStrategy** diye yeni bir dosya açarız (Gelişim). Var olan **HomeVC** koduna gidip **if-else** eklemeyiz (Değişime Kapalı).
    - *Çünkü:* Var olan ve test edilmiş çalışan koda dokunmak, yeni bug yaratmanın en garantili yoludur. Biz çalışan koda dokunmadan özellik eklemek isteriz. (Çalışıyorsa Dokunma 😄 = OCP)

---

- **Interface Segregation Principle (Arayüz Ayrımı - ISP ⇒ Odaklanmış Protokoller: Şişman Protokollerden Kaçın)**
    
    Biz devasa, her işi yapan "Tanrı Protokoller" (Fat Protocols) yerine, işe odaklı minik protokolleri severiz.
    
    - **Yanlış Olan:** **UserActionProtocol** diye bir şey yapıp içine **login()**, **logout()**, **fetchFeed()**, **deleteAccount()** hepsini koymak.
        - Eğer bir **FeedViewModel** sadece akışı çekecekse, neden **login** fonksiyonunu da implement etmek zorunda kalsın? Bu gereksiz bir yük (Code Pollution).
    - **Bizim Yaptığımız (Doğru Olan):**
        - **AuthServiceProtocol** (Sadece Login/Logout)
        - **FeedServiceProtocol** (Sadece Feed çekme)
    - Bir sınıfın ihtiyacı olmayan özellikleri ona zorla vermemek için. Swift'te buna **"Protocol Oriented Programming"** yeteneği diyoruz. İhtiyacımız kadarını alıyoruz.

- **Dependency Inversion Principle ( Bağımlılıkların Tersine Çevrilmesi - DIP = Soyutlama Takıntısı: Detaylara Değil, Kontratlara Bağlan )**
    
    Aslında **Dependency Injection** yapmamızın teorik dayanağı budur.
    
    - **Kuralı şöyle:** "Yüksek seviyeli modüller (ViewModel), düşük seviyeli modüllere (Alamofire/URLSession) doğrudan bağlı olmamalıdır. İkisi de soyutlamalara (Protocol) bağlı olmalıdır."
    - **Bizdeki Karşılığı:**
        - **HomeViewModel**, **CharacterAPIService** sınıfını tanımaz.
        - **HomeViewModel**, sadece **CharacterRepositoryProtocol**'ü (Protocol) tanır.
    - Çünkü; Detaylar (Alamofire mı kullanıyoruz, URLSession mı?) değişebilir. Ama kurallar (Veri gelmeli) değişmez. Biz binayı detayların üzerine değil, kuralların üzerine inşa ederiz. Böylece alt tarafta kütüphaneyi değiştirsek bile üst katlar (UI/ViewModel) yıkılmaz.

---

**Özetle:**

1. **SRP (Tek Sorumluluk):** Her sınıfın tek bir işi olsun, dağılmayalım.
2. **OCP (Açık/Kapalı):** Çalışan kodu bozma, yeni özellik için yeni dosya aç (Extension/Strategy).
3. **LSP (Yerine Geçme):** Test için Mock servisi taktığımda sistem anlamasın, patlamasın.
4. **ISP (Arayüz Ayrımı):** Protokolleri (Interface) küçük tut, kimseye kullanmadığı fonksiyonu zorla kullandırtma.
5. **DIP (Tersine Çevirme):** Sınıflara değil, Protokollere güven.


<br>
<br>

# Modular Architecture ile Neden Projeyi Parçalara Bölüyoruz? (Monolitik Yapı vs Modüler Mimari)

Küçük projelerde tüm kodları tek bir "App Target" içinde tutmak (Monolitik) kolaydır. Ancak ekipte 20+ iOS geliştirici olduğunda ve proje Trendyol/Getir boyutuna geldiğinde Monolitik yapı bizi boğar.

- **Biz Neden Modüler Mimarisi Konuşuyoruz?**
Projeyi **Home**, **Profile**, **Payment**, **Core**, **UIKits** gibi ufak paketlere (Swift Package Manager - SPM veya Framework) böleriz.
- **Derleme Süresi (Build Time):** Tek bir satır değiştirince tüm projeyi derlemek yerine, sadece ilgili modülü derleriz. 15 dakikalık build süresini 1 dakikaya düşürmek mühendislik başarısıdır.
- **Sorumluluk Ayrımı:** Ödeme ekibi **Payment** modülünde çalışırken, Anasayfa ekibinin kodunu bozamaz. Fiziksel sınırlar çizeriz.
- **Circular Dependency (Döngüsel Bağımlılık) Engelleyici:** Modüler yapı, spagetti kod yazmayı fiziksel olarak imkansız kılar. (Payment modülü Home'u bilemez, Home Payment'ı bilemez, ikisi de Core'u bilir).

---

**Peki Bunu Teknik Olarak Nasıl Sağlıyoruz?**
Biz bu mimariyi Xcode’da klasör açarak değil, **Workspace** ve **Local Swift Packages (SPM)** kullanarak fiziksel olarak ayrıştırırız. Yapıyı 3 katmanlı bir hiyerarşiye oturturuz:

1. **Core & UIKits Layer (Temel):** Hiyerarşinin en altıdır. **NetworkManager**, **Logger**, **Extensions** ve **DesignSystem** buradadır. Bu katman kimseyi tanımaz, herkes onu kullanır.
2. **Feature Layer (Özellikler):** İşin yapıldığı yerdir (**Home**, **Payment**, **Profile**). Her biri ayrı birer pakettir. **Altın Kural:** Feature modülleri **Core'u** bilir ama **birbirlerini asla tanımazlar.** (Örn: **Home** modülü içinde **import** ``**Payment** yazarsan kod derlenmez). İzolasyonun garantisi budur.
3. **App Layer (Birleştirici):** En tepedeki ana projedir (**Main App**). Tüm modülleri (**Home**, **Payment**, **Core**) import eder, **Coordinator** aracılığıyla bunları birbirine bağlar ve uygulamayı ayağa kaldırır.

```swift
📦 RickAndMortyHub (Workspace - Beyaz Dosya İkonu)
│
├── 📱 RickAndMortyApp (Main App - Mavi Proje İkonu)
│   ├── App
│   │   ├── AppDelegate.swift
│   │   └── SceneDelegate.swift
│   │   └── AppCoordinator.swift
│   │
│   └── Resources (Assets, Info.plist)
│
└── 📦 Packages (Sanal Klasör - Local Swift Packages)
    │
    ├── 📦 Core (Bağımsız Paket)
    │   ├── Sources
    │   │   ├── NetworkManager.swift
    │   │   └── Extensions.swift
    │   └── Package.swift  <-- (Kimseye bağımlı değil)
    │
    ├── 📦 UIKits (Bağımsız Paket)
    │   ├── Sources
    │   │   ├── Components
    │   │   └── Colors.swift
    │   └── Package.swift  <-- (Dependencies: SnapKit)
    │
    ├── 📦 Home (Feature Paket)
    │   ├── Sources
    │   │   ├── HomeVC.swift
    │   │   ├── HomeViewModel.swift
    │   │   └── HomeBuilder.swift
    │   └── Package.swift  <-- (Dependencies: Core, UIKits)
    │
    └── 📦 Detail (Feature Paket)
        ├── Sources
        │   └── DetailVC.swift
        └── Package.swift  <-- (Dependencies: Core, UIKits)
```

