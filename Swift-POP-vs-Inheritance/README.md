# Swift Derinlemesine POP ve Inheritance

# Protocol Nedir?

Bir sınıf, struct veya enum’un hangi görevleri/davranışları yerine getirmek zorunda olduğunu tanımlayan bir **"standartlar listesi"** veya **"iş sözleşmesi"**dir diyebiliriz.

- protocol fonksiyonun adını, parametresini, dönüş tipini belirtir.
- protocol fonksiyonun içini (body’yi) doldurmaz ve işin *nasıl* yapılacağına karışmaz.

Yani→ Protocol, "Bu işin **NASIL** yapılacağını değil, **NE** yapılacağını" söyler.


<br>
<br>



### Neden Kullanmalıyız?

1. **Abstraction (Soyutlama):** 
    - Detayları gizleriz. Karşımızdaki objenin tam olarak kim olduğuyla (Class adı vs.) değil, hangi yeteneğe sahip olduğuyla ilgileniriz.
2. **Decoupling (Bağları Koparma):** 
    - Sınıflar birbirine sıkı sıkıya (*tightly coupled*) bağlanmaz.
    - A sınıfı B sınıfını tanımaz, sadece B'nin elindeki "sözleşmeyi" (protocol) tanır.
3. **Testability (Test Edilebilirlik):**
    - En kritik avantajlardan biridir. Gerçek bir Network servisi yerine, aynı protokolü uygulayan sahte (*Mock*) bir servis yazarak unit testleri internetsiz ortamda çalıştırabiliriz.
4. **Composition over Inheritance:** 
    - Dikey hiyerarşi yerine yatay **YETENEK (yapabilme)** paylaşımı sağlar.

---

Protocol için bilmemiz gereken **5 Kritik Teknik Kavram** daha var. Bunlar;

### **1. Associated Types (Generic Protocols)**

Class'larda class Box<T> diyerek Generic yapabiliriz ama Protocol'lerde <T> kullanamayız. Bunun yerine associatedtype kullanırız.

**Örnek:** Bir "Depo" protokolü yazacağız ama ne depolayacağımızı (Kitap mı, Ayakkabı mı?) uygulayan sınıf seçsin.

```swift
protocol Storage {
    associatedtype Item // "Burada bir tip olacak ama ne olduğuna CONFORM eden karar versin"
    func store(item: Item)
    func retrieve(index: Int) -> Item
}

struct BookStorage: Storage {
    typealias Item = String // Item artık String oldu
    
    func store(item: String) { ... }
    func retrieve(index: Int) -> String { ... }
}
```

> Mülakat Sorusu: "Protocollerde neden <T> kullanamıyoruz da associatedtype kullanıyoruz?"
Cevap: Çünkü Protocol bir tip değildir, bir şablondur. Derleme zamanında netleşmesi gerekir.
> 

---

### 2. some vs any (Swift 5.7+ Modern Dönem)

"Opaque Types" vs "Existential Types".

- **any (Existential Type):** Bir kutudur (Box). İçine o protokole uyan **herhangi** bir şey koyabilirsin. Tipi çalışma zamanında (Runtime’da) belli olur. Esnektir ama maliyetlidir.
- **some (Opaque Type):** Derleyiciye "Buradan tek bir tip dönecek ama dışarıya söylemiyorum" deriz. Tipi derleme zamanında (Compile time) bellidir. Performansı çok yüksektir. (SwiftUI'daki var body: some View buradan gelir

```swift
// any: "Bana uyan HERHANGİ BİR TÜR dönebilirim, kutuya koyarım." (Daha yavaş, dinamik)
func createList() -> [any View] {
    return [Text("Hello"), Image("icon"), Button("Ok") {}]
}

// some: "Bana uyan TEK BİR TÜR döneceğim, ama adını söylemem." (Hızlı)
func createView() -> some View {
    return Text("Hello")
}
```
<br>

### 3. Method Dispatch (Static vs Dynamic) - Çok Kritik!

**Bir fonksiyon protocol extension'da tanımlanırsa nasıl çalışır?**

**İki durum vardır: "Sözleşmede Var mı, Yok mu?"**

Bir **Protokol** tanımladığımızda, aslında sisteme bir "Kimlik/Pasaport" tanımlıyoruz.
Derleyici  kod çalışırken şuna bakar: **"Bu fonksiyon, asıl sözleşmenin (Protocol'ün) içinde imzalanmış mı?"**

1. **Eğer İmza VARSA (Dynamic Dispatch):**
    - Derleyici der ki: "Bu fonksiyon sözleşmede var. O zaman bu görevi yapan **ASIL kişiyi (Class'ı)** bulup onun yöntemini çalıştırmalıyım."
    - Buna **Dynamic Dispatch** denir. (Doğrusu budur).
2. **Eğer İmza YOKSA ama Extension'da VARSA (Static Dispatch):**
    - Derleyici der ki: "Sözleşmede böyle bir madde yok. Bu sadece sonradan eklenmiş bir özellik (Extension). O zaman kimin yaptığına bakmama gerek yok, direkt **varsayılan (Extension'daki)** kodu çalıştırıp geçerim."
    - Buna **Static Dispatch** denir. (Tehlikeli olan budur).

```swift
protocol Logger {
    func log()     // 1. İMZA VAR: Sözleşmeye yazılmış.
    
    // DİKKAT: 'warn()' burada YOK! Sözleşmeye yazılmamış.
}

extension Logger {
    
    func log() { 
	    print("Protocol Log")  // log() için varsayılan bir kod var.
    } 

  
    func warn() { 
	    print("Protocol Warn") // 2. İMZA YOK: Sadece burada (Extension'da) var.
    } 
}

class MyLogger: Logger {
    func log() { 
	    print("Class Log") // Kendi özel loglamamızı yazdık.
    }   
    
    func warn() { print("Class Warn") }      // Kendi özel uyarımızı yazdık (AMACIMIZ BUNUN ÇALIŞMASI)
}

// MARK: -

let logger: Logger = MyLogger()

// Senaryo 1: log() Çağrısı
logger.log()
/*
    1. Derleyici bakıyor: "Logger protokolünde 'log' imzası var mı?" -> EVET.
    2. "O zaman gidip gerçekte kim olduğuna bakayım (MyLogger) ve onun kodunu çalıştırayım."
    3. Çıktı: "Class Log" ✔︎ (İstediğimiz gibi)
 */
 
 
 // Senaryo 2: warn() Çağrısı
logger.warn()
/*
    1. Derleyici bakıyor: "Logger protokolünde 'warn' imzası var mı?" -> HAYIR.
    2. "Sözleşmede yok. O zaman MyLogger'ın içine bakmama gerek yok.
    3. Direkt olarak Extension'daki kodu yapıştırıp geçeyim."
    4. Çıktı: "Protocol Warn" ✖️ (MyLogger içindeki warn çalışmadı!)
*/
```

<br>


**Neden "Sinsi Bug" Diyoruz?**

Biz **MyLogger** sınıfının içine **func warn()** yazdık ve kodumuz hatasız şekilde derlendi. Sınıfımızın içinde **warn()** fonksiyonu var mı? Var.
Ama program çalıştığında bizim yazdığımız kod **ASLA** çalışmıyor. Sanki kodumuz yok sayılıyor.  :(

Bu hatayı bulmak saatlerimizi alabilir çünkü kodda "Hata" yok, sadece **yanlış davranış** var.

**Nasıl Düzeltiriz?**

Çok basit. warn() fonksiyonunu protokolün içine  eklersek, derleyici artık "Ha, bu sözleşmede (protocol’de) varmış, gidip Class'takini çalıştırayım" der.

```swift
protocol Logger {
    func log()
    func warn() // <-- Bunu buraya eklersek sorun çözülür :)
}
```

| **Fonksiyon Nerede Tanımlı?** | **Dispatch Türü** | **Çalışma Mantığı** | **Override Çalışır mı?** |
| --- | --- | --- | --- |
| **Protocol içinde imzası VAR** | **Dynamic Dispatch** (Table Dispatch) | Çalışma anında (Runtime) asıl sınıfın (MyLogger) kodu bulunur. | **EVET ✔︎** |
| **Protocol içinde imzası YOK (Sadece Ext.)** | **Static Dispatch** (Direct Dispatch) | Derleme anında (Compile time) direkt Extension kodu yapıştırılır. | **HAYIR ✖️** |

ÇOK Önemli not:  Neden **let logger: Logger = MyLogger()** böyle yaptık?

Yani  sol taraf protocol   oldu, eşitliğin sağ tarafı ise Class oldu bu ne demek diyorsanız gelin, dahada detayına inelim( **Polymorphism (Çok Biçimlilik)** ) .

```swift
let logger: Logger = MyLogger()
```

**1. Sol Taraf (Görünen Yüz / Maske)**

```swift
let logger: Logger
```

- **Ne diyoruz:** "Benim değişkenimin tipi Logger protokolüdür."
- **Derleyici (Compiler) ne görür:** Derleyici bu satırdan sonra logger değişkenine baktığında **MyLogger'ı görmez**, sadece **Logger protokolünü** görür. MyLogger içinde 100 tane farklı fonksiyon olsa bile, eğer bunlar Logger protokolünde tanımlı değilse, onlara erişemezsin.

<br>


**2. Sağ Taraf (Gerçek Kişi / Öz)**

```swift
= MyLogger()
```

- **Ne diyoruz:** "Hafızada (RAM) gerçekten bir MyLogger objesi yarat."
- **Gerçekte ne var:** Hafızada kanlı canlı bir MyLogger class'ı duruyor.

---

"Ben şu an MyLogger kullanıyorum ama kodun geri kalanı MyLogger sınıfına bağımlı olmasın. Onlar sadece bir Logger ile konuştuğunu bilsin."

1. **Erişim Kısıtlaması (Sol Taraf):** "Ben logger değişkenini kullanırken, bana **sadece** Logger protokolünde tanımlı olan fonksiyonları göster. Eğer MyLogger sınıfının içinde kendine özel başka fonksiyonlar varsa, onları bana **gösterme ve kullandırma** (gizle)."

2. **Çalıştırma Emri (Sağ Taraf):** "Ama program çalıştığında, arka planda **gerçekten** MyLogger sınıfının kodlarını çalıştır."

**Özetle:**
"Hafızada **MyLogger** yarat, ama kod yazarken ona sadece **Logger** kurallarıyla erişmeme izin ver."

<br>


**a- Yapma Amacımız Ne? (Neden kendimizi kısıtlıyoruz?)**

Bunu yapmamızın tek ve devasa bir sebebi var: **Bağımlılığı Yok Etmek (Decoupling).**

Eğer kodumuzu MyLogger sınıfına (Class) göre yazarsak, o sınıfa **nikah kıymış oluruz :D.** Yarın öbür gün o sınıfı değiştirmek istersek, kodu yazdığın her yeri tek tek bulup değiştirmemiz gerekir. Ama kodumuzu Logger protokolüne göre yazarsan:

- **Kodumuz MyLogger sınıfını tanımaz.**
- Sadece "Log atabilen herhangi bir şey" ile çalıştığını bilir.
- Yarın MyLogger silip yerine CloudLogger koyduğumuzda, kodunun geri kalanı (sol taraf değişmediği için) bozulmadan çalışmaya devam eder.

Amaç: Kodun "Kiminle" çalıştığını bilmesin, sadece "Ne iş yapabildiğiyle" ilgilensin. Böylece parçaları (Class'ları) istediğin gibi söküp takabilirsin.

---

<br>


**b-  Bu Polymorphism (Çok Biçimlilik) mi?**

**Evet, tam olarak budur.** Polymorphism'in kelime anlamı "Çok Biçimlilik"tir. Buradaki logger değişkeni **Tek Bir İsimdir** ama **Çok Farklı Biçimlere** girebilir.

- Bugün: let logger: Logger = MyLogger() (Biçimi: MyLogger)
- Yarın: let logger: Logger = DatabaseLogger() (Biçimi: DatabaseLogger)
- Testte: let logger: Logger = FakeLogger() (Biçimi: FakeLogger)

Tam özetleyen Cümlemiz → Tek bir değişkenin (`logger`), arkasında farklı farklı Class'lar gibi davranabilmesine Polymorphism denir.

c-  "Logger kurallarıyla erişmeme izin ver" ne demek?

Bu, **"Derleyici (Compiler) bana sadece Protokolde yazanları göstersin"** demektir. Bunu kod üzerinde görelim, çok net anlaşılacak:

```swift

protocol Logger {
    func log() // Sadece bu var!
}

class MyLogger: Logger {
    // Protokoldeki zorunlu fonksiyon
    func log() {
        print("Loglandı")
    }
    
    // SINIFIN KENDİNE ÖZEL FONKSİYONU (Protokolde yok!)
    func databaseSil() {
        print("Veritabanı silindi!")
    }
}

let logger: Logger = MyLogger()

// İZİN VAR: Çünkü 'log' fonksiyonu Logger kurallarında (protokolde) yazıyor.
logger.log() ✔︎

/*
 İZİN YOK (HATA): Çünkü 'databaseSil' fonksiyonu Logger kurallarında YOK.
 İçerideki gerçek obje (MyLogger) bunu yapabiliyor olsa bile,
 sen değişkene "Bu sadece bir Logger'dır" dediğin için erişemezsin.
 */
logger.databaseSil() // Derleyici Hatası! ✖️
```

**“Logger kurallarıyla erişmek"** demek:

"Arka plandaki Class ne kadar yetenekli olursa olsun (veritabanı silebilir, kahve yapabilir), ben değişkene protokol etiketi yapıştırdığım için sadece ve sadece protokolde listelenen fonksiyonları çağırabilirim."

Çıkarım:

Yani sol tarafa Protocol tipini veririz, sağ tarafa ise bu protokolü uygulayan (conform eden) yapıyı koyarız. Böylece o yapının her şeyini kullanamayacağımı bilirim. Kim nasıl conform ettiyse arka planda onun kendi yöntemi çalışsın ve yapı esnek olsun.

<br>
<br>


# Protocol Property Requirements (get & set Nedir?)

Protocoller değişkenin kendisini (hafızadaki yerini) tutmazlar. Sadece **erişim kuralını** (Rule) belirlerler.

### 1. { get } (Sadece Okunabilir Olsun Yeter)

Protocol der ki: *"Bana bu veriyi ver de nasıl verirsen ver."*

- Bunu uygulayan (conform eden) taraf:
    - let (sabit) olabilir.
    - var (değişken) olabilir.
    - Computed Property (Hesaplanan değer) olabilir.

```swift
protocol UserProtocol {
    var name: String { get } // "Bana sadece ismini söylemen yeterli"
}

// Seçenek 1: 'let' ile uydum (Sabit)
struct UserA: UserProtocol {
    let name = "Ahmet" 
}

// Seçenek 2: 'var' ile uydum (Değişken)
struct UserB: UserProtocol {
    var name = "Mehmet"
}

// Seçenek 3: Computed Property ile uydum (Hesaplama)
struct UserC: UserProtocol {
    var name: String {
        return "Misafir Kullanıcı"
    }
}
```
<br>

### 2. { get set } (Hem Okunabilir Hem Yazılabilir Zorunluluğu)

Protocol der ki: *"Ben bu veriyi hem okuyacağım hem de değiştireceğim. Ona göre bir değişken ver."*

- Bunu uygulayan taraf:
    - **SADECE var olabilir.**
    - let OLAMAZ (Çünkü let değiştirilemez, set edilemez).
    - Getter ve Setter'ı olan Computed Property olabilir.

```swift
protocol EditableUser {
    var nickname: String { get set } // "Bunu değiştirebilmeliyim"
}

struct Gamer: EditableUser {
    var nickname = "ProPlayer" // 'var' olmak ZORUNDA. 'let' yaparsak hata verir.
}
```


<br>


### ❓ "State Tutmaz" Ne Demekti O Zaman?

→ Inheritance'da (Class) var energy = 100 dediğimizde, Class o **100** sayısını hafızada tutar.
→ Protocol'de var energy: Int { get set } dediğimizde, Protocol hafızada yer ayırmaz. Sadece bir **talep formu** oluşturur.


<br>


### ❓ Protocol İçinde let Tanımlayabilir miyiz?

**Kısa Cevap:** **HAYIR, tanımlayamayız.** ✖️ 

- Protocol içinde **her zaman var** kullanmak zorundayız.

**Neden?**
Çünkü Protocol, bir değişkenin "sabit" (let) veya "değişken" (var) olup olmadığıyla ilgilenmez. Bu, verinin hafızada nasıl tutulduğuyla ilgili bir **detaydır**. Protocol detaylara karışmaz.

Protocol sadece **Erişim Hakkı (Access Level)** ile ilgilenir:

- "Ben bunu okuyabilir miyim?" ({ get })
- "Ben bunu değiştirebilir miyim?" ({ set })

Bu yüzden kural şudur: **Protocol içinde daima var yazarız.**
Ama onu uygulayan (conform eden) struct/class, duruma göre let veya var yapabilir.

<br>


### ❓ Peki { get } ve { set } Neden Kullanıyoruz? (Amaç Ne?)

Bunu kullanmamızın sebebi yine o meşhur **Sol Taraf (Protocol Tipi)** ile ilgilidir.

Sen bir değişkene UserProtocol etiketi yapıştırdığında, derleyici o değişkenin içindeki gerçek objeyi (Struct/Class) göremez. Sadece Protocol'de yazan kuralları görür.

Eğer Protocol'de { get set } demezsen, o değeri değiştiremezsin.

<br>

### Senaryo ile İspatlayalım:

Diyelim ki bir **Email Değiştirme** fonksiyonu yazıyorsun.

**Senaryo A: Protocol Sadece { get } Demiş (Sadece Oku)**

```swift
protocol UserProtocol {
    var email: String { get } // Sadece okuyabilirsin dedik!
}

struct User: UserProtocol {
    var email: String // Gerçekte 'var' (değiştirilebilir)
}

func changeEmail(user: inout UserProtocol) {
    //  HATA! Derleyici der ki: "UserProtocol bana sadece okuma izni ({ get }) verdi."
    // "Gerçek objenin (User) içinde 'var' olması umrumda değil. Ben protokole bakarım, değiştirmene izin vermem."
    user.email = "yeni@mail.com"  ✖️
}
```

<br>

**Senaryo B: Protocol { get set } (Oku ve Yaz)**

```swift
protocol UserProtocol {
    var email: String { get set } // Okuyabilir ve DEĞİŞTİREBİLİRSİN dedik.
}

struct User: UserProtocol {
    var email: String
}

func changeEmail(user: inout UserProtocol) {
    //  ÇALIŞIR, Çünkü Protocol sözleşmesinde "değiştirilebilir" maddesi var. ✔︎
    user.email = "yeni@mail.com"
}
```


<br>
<br>


### Özetle Neden Kullanıyoruz?

1. **Güvenlik:** Bazı verilerin dışarıdan değiştirilmesini istemeyiz. Protocol'de sadece { get } diyerek, o veriyi **Read-Only (Sadece Okunabilir)** hale getiririz. (Encapsulation).
2. **Derleyiciye Talimat:** Kodun geri kalanına *"Bak bu değişkeni değiştirmene izin veriyorum"* veya *"Sadece okumana izin veriyorum"* demek için kullanırız.

# İleri Protocol Özellikleri

### 1. Marker Protocols (İşaretçi Protokoller) - Swift 5.5+

Biz bu yapıyı özellikle **Concurrency (Eşzamanlılık)** ve **Swift 6** geçişlerinde çok sık kullanırız. İçinde hiçbir fonksiyon veya değişken olmayan, yani **içi boş** olan protokollerdir.

**Neden Yaparız?**
Runtime’da (çalışma anında) bir etkisi yoktur. Biz bunu tamamen **Derleyiciye (Compiler)** bir "Sertifika" veya "Rozet" göstermek için kullanırız.

En kritik örneğimiz **Sendable** protokolüdür. Bir struct'a Sendable dediğimizde derleyiciye şu garantiyi veririz: *"Bak, bu obje Thread-Safe'tir. Yani arka plandaki thread'den ana thread'e veri taşırken veri bozulmaz, çökme olmaz. Gönül rahatlığıyla taşıyabilirsin."*

```swift
// Sendable İçinde hiçbir kural yok, Sadece bir "Rozet/İşaretçi".
struct User: Sendable {
    let name: String
}
```

<br>

### 2. Protocol Composition (& Operatörü)

Bazen bir objenin **aynı anda iki protokole** uymasını isteriz. Bunun için "ikisini birleştiren yeni bir protokol" yazıp kodumuzu kirletmeyiz.

**Nasıl Yaparız? &** operatörünü kullanarak protokolleri o anlık birleştiririz. Böylece gereksiz dosya kalabalığından kurtuluruz.

Örneğin; bir fonksiyonumuz var ve parametre olarak gelen objenin hem **Named**  olmasını hem de **Ageable** olmasını istiyoruz.

```swift
// "Bize öyle bir tip ver ki; HEM Named HEM Ageable olsun." diyoruz.
func saveUser(user: Named & Ageable) {
    print(user.name) // Named'den geldi, kullanabiliriz.
    print(user.age)  // Ageable'dan geldi, kullanabiliriz.
}
```
<br>

### 3. Conditional Conformance (Koşullu Uyum)

Generic yapılar kurarken (örneğin Array'ler veya kendi yazdığımız Wrapper'lar), o yapının yeteneklerini **içindeki elemana göre** belirleriz.

**Örnek:**[Int] dizilerini birbirine == ile karşılaştırabiliriz ama [User] dizilerini karşılaştıramayabiliriz. Neden? Çünkü Array aptaldır, içindekine bakar.

Biz kodlarımızda **where** anahtar kelimesini kullanarak şöyle deriz:
*"Eğer bu kutunun (Wrapper) içindeki **T** tipi karşılaştırılabilir (Equatable) ise, kutunun kendisi de karşılaştırılabilir olsun. Yoksa olmasın."*

```swift
struct Kutu<T> {
    let esya: T
}

// SADECE içindeki 'esya' Equatable ise, 'Kutu' da Equatable olsun.
extension Kutu: Equatable where T: Equatable {
    static func == (lhs: Kutu<T>, rhs: Kutu<T>) -> Bool {
        return lhs.esya == rhs.esya
    }
}
```
<br>

### 4. Retroactive Modeling (Sonradan Modelleme)

Bu, bizim **3. Parti Kütüphanelerle** veya **Apple'ın kendi sınıflarıyla (String, Int vs.)** çalışırken hayat kurtarıcımızdır.

**Nedir?**
Kodu bize ait olmayan, değiştiremeyeceğimiz sınıflara **sonradan** kendi protokolümüzü uygulatırız.

**Nasıl Yaparız?**
Kendi yazdığımız bir JSONExportable protokolümüz olsun. Apple'ın String sınıfının kaynak kodunu açıp içine yazamayız. Ama **Extension** açarak onu sanki bizim sistemimizin bir parçasıymış gibi davranmaya zorlarız. Böylece tüm sistemimiz tek bir dili konuşur.

```swift
protocol JSONExportable {
    func toJSON() -> String
}

// Apple'ın String sınıfına sonradan yetenek ekliyoruz.
extension String: JSONExportable {
    func toJSON() -> String {
        return "{ \"text\": \"\(self)\" }"
    }
}

// Artık String'leri de kendi protokolümüz gibi kullanabiliriz.
let metin = "Merhaba"
print(metin.toJSON())
```


<br>
<br>



# Inheritance (OOP) vs. Protocol (POP)

Temel fark **"Dikey"** ve **"Yatay"** yapılanmadır.

<br>

### 1. Dikey Hiyerarşi Problemi (Inheritance / Kalıtım)

OOP, **"X, Y'nin bir türüdür" (is-a)** ilişkisine dayanır. Yani "O da öyledir" der.

**Örnek olarak:** Bir oyun geliştiriyoruz.

- En tepede Character (Karakter) sınıfı var.
- Altında Attacker (Kılıç atar) ve Healer (Can verir) sınıfları türettik.

```swift
          [Character]
          /         \
    [Attacker]    [Healer]
   (kılıç atar)   (can verir)
       /              \
   [Knight]        [Wizard]
```

Proje yöneticisi geldi ve dedi ki: *"Yeni bir sınıf ekleyeceğiz: **Paladin**. Hem kılıçla saldıracak hem de kendini iyileştirebilecek."*

- **Seçenek A:** Attacker'dan türetsek? -> İyileştirme kodunu kopyala-yapıştır yapman lazım.
- **Seçenek B:** Healer'dan türetsek? -> Saldırı kodunu kopyala-yapıştır yapman lazım.
- **Seçenek C (En Kötüsü):** Her şeyi en tepeye Character sınıfına taşırsak? -> Basit bir asker bile gereksiz yere "iyileştirme" yeteneği taşır.

Bu **"Dikey"** bir problemdir. Ağaç dallanıp budaklandıkça, en alttaki bir sınıf, en tepedeki tüm özellikleri (gereksiz olsa bile) sırtında taşır. Buna **Tight Coupling (Sıkı Bağlılık)** denir.


<br>

### 2. Yatay Genişleme Çözümü (Protocol Oriented / Composition)

Protokoller hiyerarşik değil, **yetenek bazlıdır**. İlişki "is-a" (o da öyledir) değil, **"has-a / can-do" (yapabilir)** ilişkisidir. Kimin kimden türediği önemsizdir, **"NE YAPABİLDİĞİ"** önemlidir.

Swift

```swift
import UIKit

protocol Attackable { // Bu tip  Saldırı yapabilme yeteneğine sahiptir. = YETENEK
    func swingSword() // Kılıçla saldırma aksiyonunu gerçekleştirebilir
}

protocol Healable { // Bu tip iyileştirme yapabilme yeteneğine sahiptir. = YETENEK
    func castHeal() // İyileştirme  aksiyonunu gerçekleştirir.
}

/*
 Yani protocol ile yetenek katıyoruz. YAPABİLME Yeteneği.
*/

struct Knight: Attackable {
    func swingSword() {
        print("Kılıçla saldırı yapıldı")
    }
}

struct Wizard: Healable {
    func castHeal() {
        print("Büyü ile iyileştirme yapıldı")
    }
}

struct Paladin: Attackable, Healable {
    func swingSword() { print("Saldırı yapabildi") }
    func castHeal() { print("Kendini iyileştirdi") }
}

```

<br>

**Özetle:**

- **OOP (Dikey):** Bir binanın katları gibidir. 5. kata çıkmak için alt katların temeline muhtacız. Temel değişirse bina yıkılır.
- **Protocol (Yatay):** Lego parçaları gibidir. Bir Lego adamının eline kılıç (*Attackable*), sırtına şifa çantası (*Healable*) takabiliriz. Birbirlerinden bağımsızdırlar.

---

<br>
<br>

# Inheritance Hakkında Kritik Gerçekler

**1. Sadece Class'lar İçindir**

- **class Dog: Animal**    → (Inheritance var - Strong Coupling) ✔︎
- **struct Dog: Animal**  →  (HATA! Value Type’larda inheritance yoktur.) ✖️

**Not:** Struct ve Enum'lar sadece Protocol **conform** edebilir (uyabilir). Inheritance YAPAMAZ, çünkü Value Type’tırlar.

<br>

**2. Inheritance → Kod + State (Durum) Mirasıdır** 

Inheritance yaptığımızda sadece fonksiyonları (davranışları) değil, o sınıfın sahip olduğu değişkenleri (state) de mecburen sırtımıza yükleriz.

```swift
class Animal {
    var energy: Int = 100 // İşte bu STATE (Hafızadaki Veri)
    func eat() { energy += 10 }
}

class Dog: Animal {
    func bark() {}
}
```

**Sorun:** Dog sınıfı Animal'dan türediği an, energy değişkenini de hafızasında tutmak **zorundadır**. Belki senin oyununda köpeğin enerjiye ihtiyacı yok? Kurtulamazsın.

- **Protocol Farkı:** Protocoller **State (veri) tutmaz**, sadece kuralları belirler. Gereksiz yük taşımazsın.

<br>

**3. Override → Kırılganlık Kaynağı (Fragility)** 

Inheritance'ın en sinsi problemi **"Fragile Base Class" (Kırılgan Temel Sınıf)** sorunudur.

```swift
class Animal {
    func makeSound() {
        prepareThroat() // Parent'ın önemli hazırlığı
        print("Sound")
    }
}

class Dog: Animal {
    override func makeSound() {
        print("Bark") // DİKKAT: super.makeSound() çağırmayı unuttuk!
    }
}
```

**Sorun:** Dog sınıfı override ederek Parent'ın (Animal) kurduğu mantığı bilmeden bozdu (prepareThroat çalışmadı). Parent değişirse Child beklenmedik şekilde patlayabilir.

- **Protocol Farkı:** Protocollerde override riski yoktur. Her sınıf kendi mantığını bağımsız kurar.

<br>

### 4. "Is-a" İlişkisi ve Liskov Prensibi (LSP)

Mülakatlarda en sık sorulan tuzak soru: **"Kare (Square) bir Dikdörtgen (Rectangle) midir?"**

- **Matematiksel/Teorik:** Evet, kesinlikle. (Kare, kenarları eşit bir dikdörtgendir).
- **Yazılım Mimarisinde:** **HAYIR, değildir.**

**Neden?** 
Çünkü Inheritance sadece verileri miras almak değil, **davranışları** da miras almaktır.

1. **Dikdörtgenin Davranışı:** "Genişliğini değiştirdiğimde, yüksekliği **sabit kalır**."
2. **Karenin Davranışı:** "Genişliğini değiştirdiğimde, kare olarak kalabilmek için yüksekliği de **değişmek zorundadır**."

Eğer **class Square: Rectangle** yaparsak şu çelişki doğar:
Parent sınıf (Dikdörtgen) der ki: *"Genişliği değiştirirsen yüksekliğe dokunmam"*.
Child sınıf (Kare) der ki: *"Mecburen dokunurum"*.

Sonuç olarak →

- Child sınıf, Parent sınıfın davranış kuralını ihlal eder (yani bozar). Bu da Liskov Substitution Principle'a aykırıdır.
- Bir yerde Parent yerine Child kullandığında sistemin mantığı çöker.

---

<br>
<br>

### iOS Örneği (Anti-Pattern): BaseViewController

Aynı mantık hatasını iOS projelerinde sıkça görürüz:

**a- Yanlış Tasarım (Inheritance / BaseVC)**

```swift
//  YANLIŞ TASARIM: Kalıtım (Inheritance)

// 1. Her şeyi bilen "Baba" sınıf
class BaseViewController: UIViewController {
    
    // Yükleme gösterme kodu burada :D
    func showLoading() {
        print("Loading açıldı (BaseVC'den)")
    }
    
    // Alert gösterme kodu da burada :D:D
    func showAlert(message: String) {
        print("Alert: \(message)")
    }
}

// HomeVC mecburen BaseVC'den türüyor. Belki Alert özelliğine ihtiyacı yok ama onu da miras almak ZORUNDA.
class HomeViewController: BaseViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        showLoading() // BaseVC'den geldi
        showAlert(message: "Hata") // İstemesem de geldi :((
    }
}
```

**Sorun:** HomeVC, BaseVC'ye göbekten bağlı. Yarın BaseVC içine gereksiz 50 tane fonksiyon eklense, HomeVC hepsini sırtında taşır.


<br>

**b. Doğru Tasarım (Protocol & Extension)**

Buradaki mantığımız şudur: "Özellikleri küçük parçalara ayıralım (Lego gibi). Kimin neye ihtiyacı varsa onu taksın."

```swift
// DOĞRU TASARIM: Protokol (Composition)
protocol Loadable {
    func showLoading()
}

// Bu Extension sayesinde kodu her class içinde tekrar tekrar yazmaktan kurtuluyoruz.
extension Loadable where Self: UIViewController {
    func showLoading() {
        print("Loading açıldı (Protocol Extension'dan)") // Gerçek loading kodu buraya yazılır
    }
}

protocol Alertable {
    func showAlert(message: String)
}

extension Alertable where Self: UIViewController {
    func showAlert(message: String) {
        print("✔︎ Alert: \(message)")
    }
}

// HomeVC artık özgür. Sadece UIViewController. İhtiyacı olduğu için hem "Loadable" hem "Alertable" legolarını taktı.
class HomeViewController: UIViewController, Loadable, Alertable {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Extension sayesinde bu fonksiyonlar otomatik geldi!
        showLoading() 
        showAlert(message: "İşlem Tamam")
    }
}

// ÖRNEK: Başka bir VC sadece Alert istiyorsa:
class SettingsViewController: UIViewController, Alertable {
    // Sadece Alertable taktık. showLoading() buraya bulaşmadı. TERTEMİZ!
}
```

<br>

# Bizim İçin Altın Kural

Bir sınıfı diğerinden türetmeden önce kendimize şu soruyu sormalıyız:

- "Child sınıf, Parent sınıfın sahip olduğu TÜM özellikleri ve davranışları mantıklı bir şekilde destekliyor mu? Yoksa bazılarını bozuyor/değiştiriyor mu?"

Eğer en ufak bir şüphe varsa (kare örneğindeki gibi): **Inheritance YANLIŞTIR. Protocol kullanılmalıdır.**

| **Özellik** | **✖️ BaseViewController (Inheritance)** | **✔︎  Protocol + Extension (Composition)** |
| --- | --- | --- |
| **Bağımlılık** | Tüm alt sınıflar BaseVC'ye mahkumdur. | Sınıflar özgürdür, istediğini seçer. |
| **Esneklik** | İstemediğin özelliği çıkaramazsın. | İstediğin özelliği takıp çıkarabilirsin (Lego). |
| **State (Hafıza)** | Parent'ın tüm değişkenlerini (gereksiz olsa da) miras alırsın. | State taşımazsın, sadece yetenek alırsın. |
| **Kodun Yeri** | Her şey tek bir devasa sınıfta birikir. | Her özellik kendi küçük protokolünde yaşar. |

<br>

### Mülakat İçin Özet Cümlemiz

"OOP'deki inheritance (kalıtım) yapısında, bir sınıfın özelliklerini değiştirmek istediğimizde tüm alt sınıflar etkilenir. Ebeveyn sınıfın hem 'Hafızasını' (State) hem de 'Hatalarını' miras alırız. Ancak Protokoller ile dikey bir hiyerarşiye sıkışmadan, nesnelere yatay olarak özellik (capability) ekleyebiliriz. Bu da kodumuzu daha esnek (decoupled), test edilebilir ve 'Side Effect'lerden (yan etkilerden) arınmış yapar.”

---

