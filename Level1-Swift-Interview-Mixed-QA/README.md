# 📘 Swift & iOS Geliştirme Notları

Bu döküman, Swift programlama dili ve iOS uygulama geliştirme süreçlerindeki temel kavramları, mülakatlarda sıkça sorulan soruları ve modern çözüm yaklaşımlarını içermektedir.

---

## 📑 İçindekiler

1.  [Variables: let vs var](#let-vs-var)
2.  [Optionals Nedir?](#optionals-nedir)
3.  [Unwrapping Yöntemleri](#unwrapping-yöntemleri)
4.  [Optional Chaining](#optional-chaining-nedir)
5.  [Value Type vs Reference Type](#value-type-vs-reference-type)
6.  [Class ve Struct Farkları](#class-ve-struct-farkı)
7.  [Array / Dictionary / Set](#array--dictionary--set-temel-kullanımı-ve-farkları)
8.  [Equatable ve Hashable](#equatable-ve-hashable-nedir)
9.  [Swift Access Control](#swift-access-control)
10. [View Lifecycle (Yaşam Döngüsü)](#view-lifecycle-yaşam-döngüsü)
11. [Extension Nedir?](#extension-nedir)
12. [Frame ve Bounds Farkı](#frame-ve-bounds-farkı)
13. [Layout Lifecycle (viewDidLayoutSubviews)](#viewwilllayoutsubviews-vs-viewdidlayoutsubviews)
14. [ARC (Hafıza Yönetimi)](#arc-automatic-reference-counting)
15. [Closure & Escaping & Completion Handler](#escaping-closure--completion-handler)
16. [Alamofire ile Gerçek Senaryo](#alamofire-ile-gerçek-senaryo-veri-çekme-get-request)
17. [Enum & Raw Values](#enum-enumerations)



<br><br>

# let vs var

### Tanımlar
* **let**: Immutable (değişmez) değerler tanımlar. Bir kez değer atandığında (initialize edildiğinde) hafızadaki o değer değiştirilemez.
* **var**: Mutable (değişebilir) değerler tanımlar. İstediğin zaman yeni bir değer atayabilirsin.

### Özet
> "Swift'te let immutability (değişmezlik) sağlar. Ancak class gibi referans tiplerinde let sadece referans adresini tutar, nesnenin iç durumu (state) değişebilir; struct gibi değer tiplerinde ise nesnenin tamamını kilitler."

<br><br>

# Optionals Nedir?

Optional, Swift’te bir değişkenin değer içerebileceğini ya da `nil` olabileceğini ifade eden bir tiptir.
* **Amaç:** `nil` kaynaklı hataları (crash) daha kod yazarken önlemektir.
* **Neden var?** → Runtime crash olmasını engeller.

### Optional Durumunda Swift'in "Kıyak Geçmesi"

#### 1. `var` Kullanımı
`var name: String?` -> Swift: "Değer verilmezse `nil` kabul ederim, sorun yok."

```swift
class HomeVC: UIViewController {
    var name: String?

    override func viewDidLoad() {
        super.viewDidLoad()
        print(name)
    }
}

// Çıktı -> nil
// crash olmaz, sorun çıkarmaz.
```
<br>

#### 2. `let` Kullanımı
`let name: String?` -> Swift: "Bu bir sabit. Değerini açıkça sen vermelisin, ben kafama göre nil atamam." 

```swift
class HomeVC: UIViewController {
    let name: String?

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .blue
        print(name)
    }
}
// Hata verir compile error -> Class 'HomeVC' has no initializers
```
<br><br>

# Unwrapping Yöntemleri

Bir Optional değişkenimiz olsun: `var gelenVeri: String?`  
Bu veriyi kullanmanın 4 temel yolu vardır:

### 1. if let
* **Mantık:** "Bak bakalım `gelenVeri` dolu mu? Eğer doluysa, veriyi al yeni bir isme ata ve süslü parantez içinde kullan. `nil` ise hiçbir şey yapma (veya else bloğuna git)."
* **Sonuç:** Uygulama asla çökmez (Safe).

**Doluysa:**
```swift
var myData: String? = "Ugur hm"

// 'myData' dolu olduğu için if bloğuna girer
if let safeData = myData {
    print(safeData) // Burada artık 'safeData' normal String'dir, ? kalkmıştır.
}
```

**Nil ise:**
```swift
var name: String? // Swift otomatik nil atar.

if let myName = name {
     print(myName)
}
// if let içine hiç girmez. Hata vermez, uygulama çalışmaya devam eder.
```

---

### 2. ?? (Nil Coalescing)
* **Mantık:** "`gelenVeri` dolu mu? Doluysa onu kullan. Boşsa (`nil`), sağ taraftaki **varsayılan** değeri kullan."
* **Sonuç:** Elinde her zaman kesin bir veri olur.

```swift
// Veri yoksa otomatik olarak "Bilinmiyor" yazar.
let sonuc = gelenVeri ?? "Bilinmiyor"
```

---

### 3. guard let
Genelde fonksiyonların başında kullanılır. Hatalı durumu en başta eleyip (return), doğru durumu aşağıda kullanmaya odaklanır.

**Veri nil ise → `guard let` içine (else bloğuna) girer:**
```swift
class HomeVC: UIViewController {
    var name: String? // Swift otomatik nil atar

    override func viewDidLoad() {
        super.viewDidLoad()
        
        guard let myName = name else {
            print("Hello World") // Kod buraya düşer, çünkü name başlangıçta nil
            return // Fonksiyondan çıkar
        }
        
        print(myName) // Burası çalışmaz
    }
}
// Ekran Çıktısı -> Hello World
```

**Veri dolu ise → `guard let` else bloğuna girmez, aşağı devam eder:**
```swift
class HomeVC: UIViewController {
    var name: String? = "Ugur Hamzaoglu"

    override func viewDidLoad() {
        super.viewDidLoad()
        
        // name'i optionalliktan çıkarıp myName'e atar.
        // myName artık bu scope'un geri kalanında kullanılabilir.
        guard let myName = name else {
            print("Hata") 
            return
        }
        
        print(myName) 
    }
}
// Ekran Çıktısı -> Ugur Hamzaoglu
```

---

### 4. Force Unwrap (!)
* Optional bir değeri kontrol etmeden **zorla** açar.
* Bu değerin kesinlikle `nil` olmadığını garanti etmiş oluruz (programcı garantisi).
* ⚠️ **Risk:** Eğer değer `nil` ise, uygulama **run-time**'da crash olur (çöker).
* **Öneri:** Kullanımından mümkün olduğunca kaçınılmalıdır.

<br><br>

# Optional Chaining Nedir?

Zincirleme olarak iç içe geçmiş optional değerlere güvenli bir şekilde erişmeyi sağlar.

**Örnek:** `user?.profile?.email`

### Mantık Akışı
1. `user` var mı?
2. **Varsa** → `profile` var mı?
3. **Varsa** → `email` değerine ulaş.

### 📌 Sonuç
Eğer zincirdeki **herhangi bir halka** `nil` ise:
* ❌ **Crash olmaz.**
* ✅ Tüm işlemin sonucu `nil` olur.

### Örnek Kod

```swift
class User {
    var profile: Profile?
}

class Profile {
    var email: String?
}

class HomeVC: UIViewController {
    var user: User? // Başlangıç değeri atanmadığı için nil

    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Optional Chaining Kullanımı
        // user nil olduğu için profile bakmaz, direkt nil döner.
        let email = user?.profile?.email 
        
        if let safeEmail = email {
            print(safeEmail)
        } else {
            print("Email bulunamadi") // Buraya gelir, ekrana bunu yazdırır.
        }
    }
}
```
<br><br>

# Value Type vs Reference Type

**Temel fark:** Verinin hafızada nasıl taşındığıdır.

* **Value Type** → Verinin *kendisini* kopyalar.
* **Reference Type** → Verinin *adresini* paylaşır.

---

### 1. 🔹 Swift’te Value Type (Değer Tipi) Olan Yapılar
* Struct
* Enum
* Tuple
* Temel Tipler (Int, String, Double, Float, Bool)
* Koleksiyonlar (Array, Dictionary, Set)
* Optional, Result, Date

> **Davranış:** Bir değişkeni başka bir değişkene atadığında, sistem o verinin **birebir kopyasını** oluşturur. İkisi birbirinden tamamen bağımsızdır.

### 2. 🔹 Swift’te Reference Type (Referans Tipi) Olan Yapılar
* Class
* Closure
* Function
* Actor
* NSObject, UIView / UIViewController
* URLSession, NSManagedObject
* DispatchQueue, Timer

> **Davranış:** Bir değişkeni başkasına atadığında, verinin kendisi kopyalanmaz. Sadece o verinin **hafızadaki adresi (pointer)** kopyalanır. İkisi de aynı kaynağa bakar.

---

### 3. Hafıza Yönetimi (Stack vs Heap) - Teknik Derinlik

#### 🟦 Stack (Yığın Bellek)
Burası işlemcinin (CPU) doğrudan ve en hızlı eriştiği, geçici çalışma alanıdır.

* **Barındırdığı:** Value Types (Struct, Enum, Int, Bool...).
* **Özellikler:**
  * Çok hızlıdır.
  * **LIFO** (Last In First Out) mantığıyla çalışır.
  * Fonksiyon bitince otomatik temizlenir.
  * Thread-safe'tir.

#### 🟥 Heap (Öbek Bellek)
Burası uygulamanın uzun süreli ve büyük verilerinin saklandığı, dinamik alandır.

* **Barındırdığı:** Reference type’lar (Class Instance'ları).
* **Özellikler:**
  * Stack’e göre daha yavaştır.
  * Dinamik bellek alanıdır.
  * **ARC** (Automatic Reference Counting) ile yönetilir.
  * **Retain cycle** riski vardır.

---

### ⚠️ NOT: Mülakat Tuzağı

**Soru:** "Struct’lar her zaman stack’te mi tutulur?"

**Cevap:** Hayır. Özellikle **Copy-on-Write** kullanan yapılar istisnadır:
* Array
* Dictionary
* String
* Set

Bu yapılarda durum şöyledir:
* **Yapının kendisi:** Stack
* **İç veri (buffer):** Heap

<br><br>

# Class ve Struct Farkı

| Özellik | Struct (Yapı) | Class (Sınıf) |
| :--- | :--- | :--- |
| **Tip Türü** | **Value Type** (Değer Tipi) | **Reference Type** (Referans Tipi) |
| **Hafıza Bölgesi** | **Stack** (Genellikle). Çok hızlıdır. | **Heap**. Maliyetlidir, yer arama ve temizleme gerekir. |
| **Veri Aktarımı** | **Kopyalanır.** `var a = b` dersen b kopyalanıp a olur. | **Paylaşılır.** `var a = b` dersen ikisi de aynı adrese bakar. |
| **Miras (Inheritance)** | **Yoktur.** Başkasından türetilemez. | **Vardır.** Başka bir Class'tan özellik alabilir. |
| **Initializer (Init)** | **Otomatik gelir** (Memberwise Init). Yazmana gerek yoktur. | **Zorunludur.** Başlangıç değerlerini elle atamalısın. |
| **Değişkenlik** | Fonksiyon içinde özellik değiştirmek için `mutating` yazmak şarttır. | `mutating` gerekmez. Doğrudan değişebilir. |
| **Hafıza Yönetimi** | İşletim sistemi otomatik halleder (Scope bitince silinir). | **ARC** (Automatic Reference Counting) yönetir. |
| **Kimlik (Identity)** | **Yoktur.** `===` operatörü kullanılamaz. | **Vardır.** `===` ile "aynı nesne mi?" diye bakılır. |
| **Deinit** | Yoktur. | Vardır (`deinit` ile bellekten silindiğini yakalarsın). |

---

### ⚠️ NOT: Miras Alma (Inheritance)
Swift’te Miras Alma (Inheritance) özelliği **SADECE CLASS'lara** (Referans Tiplerine) özgüdür.

* ❌ Bir Class, bir Struct'ı miras alamaz.
* ❌ Bir Struct, bir Class'ı miras alamaz.
* ❌ Bir Struct, başka bir Struct'ı da miras alamaz.

### ❓ Peki Ortak Özellik Kazandırmak İçin Ne Yaparız?

Mülakatçı: *"Tamam miras alamıyor ama ben hem struct hem class kullanıp ortak özellikler kazandırmak istiyorum, ne yapacağım?"* derse, cevabın **Protocol** olmalı.

* Swift, **"Protocol Oriented Programming"** (Protokol Odaklı Programlama) dilidir.
* Miras almak yerine, bir "sözleşme" (**protocol**) imzalatırız.

<br><br>

<br><br>

# Array / Dictionary / Set Temel Kullanımı ve Farkları

### 1. Array (Dizi) - "Sıralı Liste"
Verileri belirli bir sırada (index) tutar.
* **Özellik:** Sıralıdır (Ordered). Eklediğin sırayla durur. Duplicate (tekrar eden) eleman serbesttir.
* **Erişim:** Index biliniyorsa direkt gider. Ama arama yapmak için baştan sona bakar.

### 2. Dictionary - "Key-Value"
Verileri Key-Value (Anahtar-Değer) eşleşmesi olarak tutar.
* **Özellik:** Sırasızdır (Unordered).
* **Kural:** Key (Anahtar) benzersiz (unique) olmak zorundadır. Value (Değer) aynı olabilir.
* **Çalışma:** Hash table kullanır. Doğrudan doğru bucket’a (kutucuğa) gider.

### 3. Set (Küme) - "Benzersiz Torba"
* **Özellik:** Sırasızdır (Unordered). Karışık bir torba gibidir.
* **Kural:** **Tekrarı asla kabul etmez.** Her eleman sadece 1 kere bulunabilir.
* **Erişim:** Index yoktur. "İçinde var mı?" (`contains`) diye sorarsınız. Arama, ekleme ve silme işlemleri hash değerine bakarak doğrudan yapılır.

---

## ⚠️ Mülakat Tuzakları ve Performans

**Not:** Dictionary ve Set sıralı değildir. Hızlı olmalarının sebebi **Hashing**'dir. Bu yüzden `Hashable` protokolü zorunludur.

### Soru 1: Arama Performansı
**Senaryo:** "Elimde 1 milyon eleman var. Bir elemanın listede olup olmadığını kontrol edeceğim (`contains`). Array mi kullanayım Set mi?"

* **Cevap:** Kesinlikle **Set** (veya Dictionary).
* **Neden?**
    * **Array:** Tek tek en baştan sona kadar bakar (Linear Search - `O(n)`). Çok yavaştır.
    * **Set:** Verinin "Hash" değerine bakar ve nokta atışı bulur (Constant Time - `O(1)`). Eleman sayısı artsa da hız değişmez.

### Soru 2: Array Kapasitesi (Senior Detayı)
**Senaryo:** "Bir Array'e sürekli `append` ile eleman ekliyoruz. Bu işlem her zaman çok hızlı (`O(1)`) mıdır?"

* ❌ **Yanlış Cevap:** "Evet, sona eklemek her zaman hızlıdır."
* ✅ **Doğru Cevap (Senior):** "Genellikle hızlıdır (`O(1)`). **ANCAK**, Array'in kapasitesi dolduğunda, Swift arka planda daha büyük yeni bir Array oluşturur ve eski elemanların hepsini oraya kopyalar. O anlık işlem `O(n)` olur ve maliyetlidir."
* **Çözüm:** Eğer elinde kaç eleman olacağı yaklaşık belliyse (örneğin 10.000 eleman), Swift'e baştan yer ayırtırsın (`reserveCapacity`). Böylece Array dolup taşmaz, kopyalama yapmaz.

### Soru 3: `filter` vs `first(where:)`
**Senaryo:** "1 milyon elemanlı bir listem var. İçindeki **ilk** 'negatif' sayıyı bulmak istiyorum. `filter` mı kullanayım, `first(where:)` mi?"

* **Tuzak:** İkisi de aynı sonucu verir ama çalışma mantıkları farklıdır.
* ❌ **Yanlış Cevap:** `numbers.filter { $0 < 0 }.first` yazarım.
* ✅ **Doğru Cevap:** Kesinlikle `first(where:)` kullanırım.
* **Neden?**
    * **filter:** Listenin sonuna kadar gider (`O(n)`). Hepsini tarar, negatif olanlardan hafızada *yeni bir Array* oluşturur. Sonra onun ilkini alır. (Çok işlem, ekstra hafıza).
    * **first(where:):** Baştan başlar, ilk bulduğu anda **DURUR** ve döner (Early Exit). Listede 1 milyon eleman olsa bile, negatif sayı 2. sıradaysa sadece 2 işlem yapar.

### Soru 4: Dictionary Key Kısıtlaması
**Senaryo:** "Kendi oluşturduğum bir `Ogrenci` class'ını Dictionary'de 'Key' (Anahtar) olarak kullanabilir miyim?"
```swift
var sinif: [Ogrenci: Int] = [:]
```

* **Cevap:** "Hayır, doğrudan kullanamam. O class'ın **Hashable** protokolüne uyması gerekir."
* **Neden?** Set ve Dictionary'nin `O(1)` hızında çalışabilmesi (nokta atışı bulması) için, o anahtarın benzersiz bir **Hash ID (Parmak izi)** üretmesi gerekir. Swift bu parmak izine bakarak verinin hangi rafta olduğunu bulur. Eğer `Hashable` yapmazsan, Swift o veriyi nereye koyacağını bilemez ve derleyici hata verir.

<br><br>

<br><br>

# Equatable ve Hashable Nedir?

### 1. Equatable (`==`)
İki nesnenin birbirine eşit olup olmadığını (`==` operatörü ile) kontrol etmeni sağlayan protokoldür.

* **Ne İşe Yarar:** Swift, senin oluşturduğun özel bir `User` struct'ının eşitliğini nasıl kontrol edeceğini bilmez. (İsmi mi aynı olmalı? ID'si mi?). `Equatable` ile ona bu kuralı öğretirsin.
* **Kullanım Alanı:** `if user1 == user2`, `array.contains(user1)`, `array.firstIndex(of: user1)` gibi işlemlerde zorunludur.

### 2. Hashable (`#`)
Bir nesneyi benzersiz bir Tam Sayıya (**Integer Hash Value**) dönüştüren protokoldür.

* **Ne İşe Yarar:** Nesnenin hafızadaki "adresini" hesaplamak için kullanılır. Bu sayede arama yaparken tek tek gezmek yerine, doğrudan o adrese gidilir.
* **Kullanım Alanı:** `Set` içine eleman eklerken veya `Dictionary`'de **Key** olarak kullanırken ZORUNLUDUR.

---

### Struct ve Class Farkı

#### Struct’ta Durum (Value Type)
* Çoğu struct **otomatik olarak** `Equatable` ve `Hashable` olur.
* Yani çoğu zaman elle implement etmeye gerek yoktur.

#### Class’ta Durum (Reference Type)
* **Equatable:** Eğer içerik bazlı karşılaştırma istiyorsan `Equatable` implement etmelisin.
* **Hashable:** Kullanmak istiyorsan `hash(into:)` ve `==` fonksiyonlarını elle yazmalısın.

```swift
class User: Hashable {
    var id: Int
    init(id: Int) { self.id = id }
    
    // Equatable Gerekliliği
    static func == (lhs: User, rhs: User) -> Bool {
        return lhs.id == rhs.id
    }

    // Hashable Gerekliliği
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }
}
```

---

### ⚠️ NOT: `==` ve `===` Farkı

* **`==` (Eşitlik):** "İçindeki veriler aynı mı?" (Örn: İkisinin de adı "Ali" mi?)
* **`===` (Kimlik):** "Bunlar fiziksel olarak aynı nesne mi?" (Örn: Hafıza adresleri `0x123` ve `0x123` mü?)

#### Kritik Kural
`===` operatörü **SADECE CLASS** (Referans tipleri) için kullanılır. Struct'larda (Değer tipleri) kullanılamaz, çünkü Struct'ların hafızada sabit bir kimliği yoktur, sürekli kopyalanırlar.

<br><br>


<br><br>

# Swift Access Control

Swift'te erişim belirleyiciler, kodunuzun diğer dosyalar veya modüller tarafından ne kadar görünebilir olduğunu yönetir.

### Access Control Hiyerarşisi (En Dar -> En Geniş)

#### 1. `private` (En Gizli / Kilitli) 🔒
* **Kapsam:** Sadece tanımlandığı süslü parantezler `{ }` içinde (ve aynı dosyadaki extension'larında) geçerlidir.
* **Özellik:** Dışarıdan kimse göremez.
* **Kullanım:** Sadece o sınıfın iç işleyişini ilgilendiren yardımcı fonksiyonlar veya değişkenler için kullanılır.

#### 2. `fileprivate` (Dosya Özel) 📁
* **Kapsam:** Sadece bulunduğu `.swift` dosyasının içinde geçerlidir.
* **Özellik:** Aynı dosya içinde farklı class veya struct'lar varsa birbirini görebilir.
* **Kullanım:** Bir dosyada birden fazla class/extension yazıp, birbirlerinin özel verilerine erişmesini istediğinizde kullanılır.

#### 3. `internal` (Modül İçi - Varsayılan) 🏠
* **Kapsam:** Hiçbir şey yazmazsanız **otomatik olarak** budur.
* **Özellik:** Sadece kendi uygulaman (modülün) içindeki her yerden erişilebilir.
* **Sınır:** Uygulamanın dışından (başka bir framework'ten) görülemez.

#### 4. `public` (Halka Açık) 🌍
* **Kapsam:** Başka modüllerden (frameworklerden) erişilebilir.
* **Kritik Kısıtlama:** Başka modülde miras alınamaz (subclass) veya override edilemez. Sadece "kullanılabilir".

#### 5. `open` (Tamamen Açık) 🔓
* **Kapsam:** En geniş erişim seviyesidir. **Sadece Class'lar için geçerlidir.**
* **Özellik:** Başka modüllerden erişilebilir, miras alınabilir ve override edilebilir.
* **Kullanım:** UIKit içindeki `UIViewController` veya `UITableView` gibi, sizin geliştirip üzerine kod yazdığınız kütüphane elemanları `open`dır.

---

### ⚠️ Mülakat Tuzağı: `public` vs `open` Farkı

**Soru:** "Bir kütüphane yazdım, `public class` ile `open class` arasındaki fark nedir?"

**Cevap:**
* **public:** Sınıfı başka modülde sadece kullanabilirim (instance oluşturabilirim) ama ondan türetip (inheritance) özelliklerini değiştiremem (override).
* **open:** Buna izin verir; yani o sınıfı miras alıp genişletebilirim.

---

### ℹ️ Not: Varsayılan (Default) Davranış

Hiçbir şey yazmadan standart bir tanımlama yaparsan, Swift bunu otomatik olarak **internal** kabul eder.

* **Ne Anlama Geliyor?** Bu kodu, yazdığın uygulamanın (App Target / Modül) her yerinde kullanabilirsin. Ancak bu kodu bir kütüphane (Framework) haline getirip başka bir projeye eklersen, oradan görünmez.

**Mülakat Notu:**
Eğer mülakatçı *"Neden default olarak public değil de internal?"* derse:
> "Çünkü Swift güvenliği ön planda tutar. Yazdığım kodun varsayılan olarak dış dünyaya (API'lara) kapalı olmasını, sadece kendi projem içinde rahatça çalışmasını ister."

<br><br>


<br><br>

# View Lifecycle (Yaşam Döngüsü)

iOS'te bir ekranın (ViewController) oluşturulmasından yok edilmesine kadar geçen süreçtir.

### 1. viewDidLoad
* **Ne Zaman Çalışır:** Ekran hafızaya yüklendiğinde **sadece 1 kez** çalışır.
* **Ne Yapılır:**
    * Ekranın ilk kurulumu (Initial Setup).
    * Değişkenlere ilk değer atama.
    * Network isteği atma (Genellikle).
    * `delegate` ve `dataSource` atamaları (TableView vs).
* **Mülakat Notu:** Ekranı kapatıp geri gelmediğin sürece (Navigation Stack'te kaldığı sürece) bir daha çalışmaz.

### 2. viewWillAppear (Sahneye Çıkmadan Hemen Önce)
* **Ne Zaman Çalışır:** Ekran kullanıcıya görünmeden hemen önce çalışır. Her sayfaya giriş-çıkışta **tekrar tetiklenir**.
* **Ne Yapılır:**
    * Ekrandaki veriyi güncelleme (Örn: Profil sayfasına her girdiğinde Coin miktarını yenilemek).
    * Navigation Bar veya Tab Bar gizleme/gösterme işlemleri.
    * Klavye olaylarını dinlemeye başlama (`NotificationCenter addObserver`).

### 3. viewDidAppear (Sahneye Çıktı)
* **Ne Zaman Çalışır:** Ekran tamamen ekrana geldikten hemen sonra çalışır.
* **Ne Yapılır:**
    * Animasyonları başlatmak.
    * Analytics event göndermek ("Kullanıcı bu sayfayı gördü" verisi).
    * GPS/Konum servisini başlatmak.

### 4. viewWillDisappear (Sahneden İniyor)
* **Ne Zaman Çalışır:** Kullanıcı ekrandan ayrılmaya başladığı an (Back tuşuna bastığı an) çalışır.
* **Ne Yapılır:**
    * Animasyonları durdurmak.
    * Klavye'yi kapatmak (`view.endEditing(true)`).
    * Yapılan değişiklikleri kaydetmek (Autosave).

### 5. viewDidDisappear (Sahne Bitti)
* **Ne Zaman Çalışır:** Ekran tamamen kaybolduktan sonra çalışır.
* **Ne Yapılır:**
    * NotificationCenter dinlemeyi durdurmak (`removeObserver`).
    * Ağır işlemleri (Timer, Sensor) tamamen durdurmak.

---

## ⚠️ Mülakat Tuzağı: viewDidLoad vs viewWillAppear

**Soru:** "Kullanıcı profil sayfasına gitti, adını değiştirdi ve geri tuşuna basıp Ana Sayfa'ya döndü. Ana Sayfa'daki 'Hoşgeldin İsim' yazısını güncellemek için kodu `viewDidLoad`'a mı yazarsın `viewWillAppear`'a mı?"

**Cevap:** Kesinlikle **viewWillAppear** içine yazarım.

**Neden?**
* **viewDidLoad:** Ana Sayfa hafızada hala durduğu için (Stack'ten silinmediği için), geri dönüldüğünde tekrar çalışmaz. İsim eski kalır.
* **viewWillAppear:** Ekran her göründüğünde çalıştığı için, geri dönüldüğünde güncel ismi yakalar ve ekrana basar.

---

### 📊 Karşılaştırma Tablosu

| Metod | Çalışma Sıklığı | Temel Görev |
| :--- | :--- | :--- |
| **viewDidLoad** | **1 Kez** (O ekran RAM'de canlı kaldığı sürece). | UI Kurulumu, API İsteği |
| **viewWillAppear** | **Her Girişte** | Veri güncelleme, UI tazelemeleri |
| **viewDidAppear** | **Her Girişte** | Animasyon başlatma, Analytics |
| **viewWillDisappear** | **Her Çıkışta** | Klavye kapatma, Kayıt |
| **viewDidDisappear** | **Her Çıkışta** | Temizlik, Dinlemeyi durdurma |

### 🔄 Senaryo Analizi: viewDidLoad Çalışır mı?

| Senaryo | viewDidLoad Çalışır mı? | Neden? |
| :--- | :---: | :--- |
| **Uygulama İlk Açılışı** | ✅ EVET | Ekran ilk kez hafızaya giriyor. |
| **Başka Ekrana Gidip "GERİ" Dönmek** | ❌ HAYIR | Ekran zaten hafızada (arkada) bekliyordu. |
| **Tab Bar'da Başka Taba Gidip Dönmek** | ❌ HAYIR | Tab'lar genelde hafızada tutulur, silinmez. |
| **Ekranı Kapatıp (Back) Sonra Tekrar Girmek** | ✅ EVET | Ekran hafızadan silinmişti, (Back tuşu ekranı öldürür) yeniden yaratıldı. |

---

### 🔹 Kısa Özet (Sıralı Akış)

1. `loadView()` → View oluşturulur (Custom view yoksa elleme).
2. `viewDidLoad()` → View yüklendi, initial setup.
3. `viewWillAppear()` → Ekran görünmeden hemen önce.
4. `viewDidAppear()` → Ekran göründükten sonra.
5. `viewWillDisappear()` → Ekran kaybolmadan önce.
6. `viewDidDisappear()` → Ekran kaybolduktan sonra.
7. `deinit` → Controller hafızadan silinir.

<br><br>


<br><br>

# Extension Nedir?

**Extension (Genişletme):** Class, Struct, Enum veya Protocol gibi mevcut türlere, orijinal kaynak koduna erişimin olmasa bile yeni özellikler (fonksiyon, hesaplanan değişken) ekleme yeteneğidir.

### ⚠️ Kritik Mülakat Tuzağı: Ne Yapamaz?

**Soru:** "Extension ile `stored property` (hafızada yer kaplayan normal değişken) ekleyebilir miyim?"

* ❌ **Cevap:** **HAYIR!**
* **Neden:** Extension, o yapının hafızadaki boyutunu (memory footprint) değiştiremez. Sadece var olan veriyi kullanarak hesaplama yapabilir (computed property) veya fonksiyon ekleyebilir.

```swift
// ❌ YANLIŞ (Stored Property eklenemez)
extension Int {
    var name: String = "Ali" 
}

// ✅ DOĞRU (Computed Property eklenebilir)
extension Int {
    var isEven: Bool { return self % 2 == 0 }
}
```


<br><br>


# Frame ve Bounds Farkı

| Özellik | Frame | Bounds |
| :--- | :--- | :--- |
| **Referans Noktası** | **Parent (Superview)** | **Self (Kendisi)** |
| **Origin (x, y)** | Superview'un sol üst köşesine uzaklığı. | Genellikle `(0,0)`. (ScrollView kaydırılınca değişir). |
| **Size (En, Boy)** | Görünümü kaplayan dikdörtgen kutu. | Görünümün gerçek boyutları. |
| **Rotasyon Etkisi** | Dönünce boyutları değişir/büyür (kapsayıcı kutu büyür). | Dönse bile boyutları aynı kalır. |
| **Ne Zaman Kullanılır?** | Yerleştirme (Positioning), Animasyon. | İç Çizim (Drawing), Scroll işlemleri. |

---

# viewWillLayoutSubviews vs viewDidLayoutSubviews

### 1. viewWillLayoutSubviews
**"Hesaplama Başlamak Üzere"**

Bu metod çağrıldığında, View Controller'ın ana view'ının sınırları (bounds) değişmiştir (örneğin telefon yatay moda geçmiştir), ancak alt görünümler (subviews) henüz yeni yerlerine ve boyutlarına oturtulmamıştır.

* **Durum:** Frame'ler henüz güncel değildir (eski değerlerde olabilir).
* **Ne Zaman Çalışır?** Auto Layout hesaplaması başlamadan hemen önce.
* **Burada Ne Yapılır?** Genellikle çok aktif kullanılmaz.
    * Layout hesaplaması başlamadan önce eklenmesi veya kaldırılması gereken görünümler varsa burada müdahale edilebilir.
    * Constraint'lerde son dakika değişikliği (update) yapılacaksa burası kullanılabilir.

### 2. viewDidLayoutSubviews (Yıldızlı Madde ⭐)
**"Hesaplama Bitti, Her Şey Yerli Yerinde"**

Bu metod çağrıldığında, Auto Layout motoru işini bitirmiştir. Tüm butonlar, textler, resimler ekrandaki nihai yerlerine (frame) ve boyutlarına kavuşmuştur.

* **Durum:** Frame ve Bounds değerleri artık kesindir ve doğrudur.
* **Ne Zaman Çalışır?** Auto Layout hesaplaması bittikten hemen sonra.
* **Burada Ne Yapılır? (Çok Önemli)**
    * **Layer İşlemleri (Gradient, Shadow, Border):** `viewDidLoad` içinde bir view'a gradient verirsen veya `cornerRadius` ile tam yuvarlak yapmak istersen bazen hatalı görünür. Çünkü `viewDidLoad` anında ekranın genişliği henüz tam hesaplanmamış olabilir. Burası, frame'e bağlı UI makyajı için en doğru yerdir.
    * **Yuvarlak Buton Yapma:** `button.layer.cornerRadius = button.frame.height / 2` kodunu buraya yazarsan, buton boyutu ne olursa olsun her zaman tam yuvarlak olur.
    * **ScrollView ContentSize:** Manuel ScrollView yapıyorsan `contentSize` burada güncellenir.

#### ⚠️ BÜYÜK TEHLİKE: Sonsuz Döngü (Infinite Loop)
`viewDidLayoutSubviews` içinde, ekranın yerleşimini tekrar tetikleyecek bir kod yazarsan (örneğin `view.layoutIfNeeded()` çağırırsan veya bir constraint'i değiştirirsen), sistem tekrar layout hesaplar, tekrar bu metoda girer, tekrar hesaplar... **Uygulama donar (crash).**

> **Özet:** "Frame'e (boyutlara) bağlı görsel işlemler (layer, gradient, radius) için **viewDidLayoutSubviews** kullanırım çünkü viewDidLoad aşamasında frame değerleri henüz kesinleşmemiş olabilir."

<br><br>

<br><br>

# ARC (Automatic Reference Counting)

Swift’in RAM (Hafıza) yönetim sistemidir.

* **Çalışma Mantığı:** Basittir. Bir obje oluşturulduğunda ve bir değişkene atandığında, ARC o objenin **"Referans Sayısını" 1 artırır**. O değişkenle işin bittiğinde veya değişken `nil` olduğunda sayıyı **1 azaltır**.
* **Kural:** Referans sayısı **0** olduğu an, obje hafızadan (RAM) anında silinir (**Deallocation**).
* **Amaç:** Kullanılmayan objeleri silerek RAM'i boşaltmak.

---

### Referans Türleri (Strong, Weak, Unowned)
ARC'nin sayıyı artırıp artırmayacağını belirleyen anahtar kelimelerdir.

#### 1. Strong (Güçlü) Referans
* **Durum:** Varsayılandır. (`var user = User()` dediğinde strong olur).
* **Etki:** Referans sayısını **+1 artırır**.
* **Anlamı:** "Ben bu objeyi tutuyorum, ben bırakana kadar silinmesin."
* **Risk:** Güvenlidir ancak **Retain Cycle** yaratma riski vardır.

#### 2. Weak (Zayıf) Referans
* **Etki:** Referans sayısını **artırmaz (+0)**.
* **Davranış:** Obje silinirse (referans sayısı 0 olursa), bu değişken otomatik olarak `nil` olur.
* **Kural:** Bu yüzden weak değişkenler her zaman **Optional (`var?`)** olmak zorundadır.
* **Kullanım:** Retain Cycle'ı kırmak için en çok kullanılan yöntemdir.

#### 3. Unowned (Sahipsiz) Referans
* **Etki:** Referans sayısını **artırmaz (+0)**.
* **Fark:** `weak` gibidir AMA **Optional değildir.**
* **Garanti:** "Ben bu objeye eriştiğimde, obje hafızada kesinlikle var olacak."
* **Risk:** Eğer obje silinmişse ve sen `unowned` referansa erişmeye çalışırsan uygulama çöker (**crash**).

---

### Retain Cycle (Referans Döngüsü) Nedir?

İki objenin birbirini **Strong (Güçlü)** olarak tutması durumudur.

1. A objesi B'yi tutuyor (+1).
2. B objesi A'yı tutuyor (+1).
3. Sen bu iki objeyi kullanmayı bıraksan bile, birbirlerini tuttukları için referans sayıları asla 0'a düşmez.
* **Sonuç:** ARC bu objeleri asla silemez -> **Memory Leak**.

#### Çözümleri Nedir?
Zincirin bir halkasını zayıflatman gerekir. İki taraf da birbirini Strong tutamaz.

**1. Delegate Pattern'de:**
Delegate değişkeni her zaman `weak` tanımlanmalıdır.
```swift
weak var delegate: SomeDelegate?
```

**2. Closure Kullanımında:**
Closure içinde `self` kullanılacaksa, `[weak self]` yapısı kullanılmalıdır.

```swift
// ❌ Yanlış (Retain Cycle yapar):
profilGetir { sonuc in
    self.ismiGuncelle() // self strong tutuluyor
}

// ✅ Doğru:
profilGetir { [weak self] sonuc in
    self?.ismiGuncelle() // self weak tutuluyor, referans artmaz
}
```

### Memory Leak (Hafıza Sızıntısı) Nedir?
Retain Cycle sonucu oluşan durumdur. Uygulama içinde artık erişilemeyen, kullanılmayan ama RAM'de yer kaplamaya devam eden veri yığınıdır.

<br><br>


# Escaping Closure & Completion Handler

### 1. Escaping Closure (`@escaping`) Nedir?
Bir closure (blok), bir fonksiyona parametre olarak geçildiğinde; eğer bu closure, fonksiyonun gövdesi (scope) çalışmasını bitirdikten sonra çağrılıyorsa, bu closure **"escaping" (kaçan)** olarak adlandırılır.

* **Varsayılan Durum:** Swift'te tüm closure'lar varsayılan olarak `non-escaping`'dir (fonksiyon bitmeden çalışmak zorundadırlar).
* **Derleyici Bildirimi:** Bu durumu belirtmek için `@escaping` anahtar kelimesi kullanılır.
* **Bellek Yönetimi:** Bu closure'lar fonksiyon bittikten sonra da çalışacakları için bellekte (**heap**) tutulmaya devam ederler.

### 2. Completion Handler Nedir?
Bir dil özelliği değil, bir **dizayn desenidir (design pattern)**. Bir işlemin (genellikle uzun süren bir işin) bittiğini, sonucun başarılı mı yoksa hatalı mı olduğunu çağıran tarafa haber vermek için kullanılan yöntemdir.

### 3. Asenkronluk Var mıdır?
**Evet, genellikle vardır.** * Her `escaping closure` asenkron olmak zorunda değildir (örneğin bir değişken dışarıya atanıp sonra çalıştırılabilir).
* Ancak **Completion Handler**'ların neredeyse tamamı asenkron işlemler (network, veritabanı, animasyon) için kullanılır. İşlem arka planda devam ederken ana fonksiyon sonlanır; iş bittiğinde ise closure "kaçar" ve tetiklenir.

---

### 4. Normal Fonksiyondan Farkı Nedir?

| Özellik | Normal Fonksiyon (return) | Completion Handler (Escaping) |
| :--- | :--- | :--- |
| **Zamanlama** | Hemen değer döner (Synchronous). | İşlem bittiğinde (belirsiz süre sonra) döner. |
| **Bloklama** | İşlem bitene kadar thread bekler. | Thread serbest kalır, iş bitince tetiklenir. |
| **Kontrol** | Tek bir değer dönebilir. | Birden fazla kez çağrılabilir veya hiç çağrılmayabilir. |
| **Kapsam** | Fonksiyon bitince değişkenler ölür. | Değişkenler "capture" edilir ve yaşatılır. |

---

### 5. Hangi Durumlar İçin Kullanırız? (Use Cases)
* **Network İstekleri:** Sunucudan veri beklerken UI'ı dondurmamak için.
* **Animasyonlar:** Bir animasyon bittiğinde başka bir işlemi başlatmak için.
* **Dosya İşlemleri:** Büyük bir dosyayı okuma/yazma işlemi tamamlandığında haber almak için.

---

### 6. Modern Alternatif: Swift Concurrency (Async/Await)
Swift 5.5 ile gelen `async/await` yapısı artık sektör standardı haline gelmektedir.

**Neden Async/Await?**
* **Callback Hell:** İç içe geçen closure'ların (Pyramid of Doom) önüne geçer.
* **Hata Yönetimi:** `try-catch` blokları ile daha temiz kod sağlar.
* **Okunabilirlik:** Asenkron kodu, sanki senkronmuş gibi yukarıdan aşağıya okumayı sağlar.
* **Güvenlik:** Closure'da "unutup çağırmama" riski varken, `async` fonksiyonlar mutlaka bir değer veya hata dönmek zorundadır.

---

### 💻 Teknik Örnek (Mülakat Tipi)

**Klasik Yaklaşım (Completion Handler):**
```swift
func fetchData(url: String, completion: @escaping (Result<Data, Error>) -> Void) {
    URLSession.shared.dataTask(with: URL(string: url)!) { data, response, error in
        if let error = error {
            completion(.failure(error)) // Fonksiyon bittikten sonra çağrılıyor
            return
        }
        if let data = data {
            completion(.success(data)) // Kaçış (Escape) gerçekleşiyor
        }
    }.resume()
}
```

**Modern Yaklaşım (Async/Await):**
```swift
func fetchData(url: String) async throws -> Data {
    let (data, _) = try await URLSession.shared.data(from: URL(string: url)!)
    return data
}
```

> **Özet:** > * **@escaping:** Fonksiyonun kapsamından sağ kurtulan kod bloğudur.
> * **Completion Handler:** "İşim bitti, al bu da sonuç" deme yöntemidir.
> * **Async/Await:** Bu işlemlerin en modern ve güvenli halidir.

<br>

#### Mülakat İçin Özet Cümle:
"Closure'lar isimsiz kod bloklarıdır. Eğer bu bloklar fonksiyonun ömründen daha uzun süre yaşayacaksa (örneğin bir network isteği bittiğinde çalışacaksa), onları @escaping olarak işaretleriz. Bu yapıya genellikle Completion Handler deriz; ancak güncel projelerde bu yapının yerini daha temiz ve güvenli olan Async/Await almaktadır."

<br>


# Alamofire ile Gerçek Senaryo: Veri Çekme (GET Request)

Profesyonel projelerde asenkron veri çekme işlemlerinin nasıl yönetildiğine dair somut bir örnek.

```swift
import Alamofire

struct MovieService {
    
    // 1. Fonksiyon Tanımı: 'completion' parametresi @escaping'dir.
    // İnternet hızı belirsizdir; fonksiyon hemen biter ama veri sonra (asenkron) gelir.
    func fetchMovies(completion: @escaping (Result<[String], AFError>) -> Void) {
        
        let url = "[https://api.ornek.com/movies](https://api.ornek.com/movies)"
        
        // 2. Alamofire İsteği: AF.request arka planda (background) çalışır.
        AF.request(url).responseDecodable(of: [String].self) { response in
            
            // 4. Burası bir closure'dır ve fonksiyon bittikten sonra çalışır (Escaping).
            // Alamofire sonuçları otomatik olarak ANA THREAD (Main) üzerinden döner.
            
            switch response.result {
            case .success(let movies):
                completion(.success(movies)) // Başarı durumu
            case .failure(let error):
                completion(.failure(error)) // Hata durumu
            }
        }
        
        // 3. Fonksiyonun gövdesi burada biter ve Stack'ten silinir.
        print("LOG: fetchMovies fonksiyonu return etti.")
    }
}
```

---

### 🧠 Adım Adım Akış Analizi

1.  **Tetikleme:** `fetchMovies` çağrıldığında, içine bir kod bloğu (closure) teslim edilir.
2.  **Request:** Alamofire isteği başlatır. Bu sırada uygulama donmaz çünkü işlem arka planda yürütülür.
3.  **Fonksiyonun Sonu:** Fonksiyon son süslü parantezine ulaşır ve biter. Normalde her şeyin silinmesi gerekirken, `completion` bloğu `@escaping` olduğu için bellekte (**Heap**) tutulmaya devam eder.
4.  **Yanıt:** Sunucudan cevap geldiğinde Alamofire veriyi modeller.
5.  **Çalıştırma:** Alamofire, elinde tuttuğu o "paketlenmiş kod bloğunu" (`completion`) bulur, verileri içine koyar ve tetikler.

---

### 🚀 Modern Alternatif: Alamofire + Async/Await

Yeni nesil projelerde kodun okunabilirliğini artırmak için **closure** yerine bu yöntem tercih edilir:

```swift
// ❌ ESKİ: Completion Handler ile
func getMovies(completion: @escaping (Result<[String], AFError>) -> Void) { ... }

// ✅ YENİ: Async/Await ile
func getMovies() async throws -> [String] {
    let task = AF.request("[https://api.com/movies](https://api.com/movies)").serializingDecodable([String].self)
    
    // Burada cevap gelene kadar bekler ama main thread'i dondurmaz.
    let movies = try await task.value 
    
    return movies
}
```

> **Özet Not:** Alamofire kullanırken de mantık aynıdır: *"Sen git veriyi çek, gelince benim sana verdiğim bu closure paketini aç ve çalıştır."* Alamofire, bu süreci otomatik **thread yönetimi** ile daha güvenli hale getirir.

<br><br>


# Enum (Enumerations)

### 1. Raw Values (Ham Değerler)
Bir enum'ın her bir case'ine (durumuna), derleme zamanında (compile-time) önceden belirlenmiş, **sabit** bir değer atanmasıdır.

* **Özellikleri:** * Tüm case'ler aynı veri tipinde (`String`, `Int`, `Character` vb.) olmalıdır.
    * Kod çalışırken bu değerler değişemez, sabittir.
    * Her bir ham değer (raw value) benzersiz olmalıdır.

```swift
enum HTTPError: Int {
    case ok = 200
    case unauthorized = 401
    case notFound = 404
    case internalServerError = 500
}

// Kullanımı:
let status = HTTPError.notFound
print(status.rawValue) // Çıktı: 404
```

---

### 🛡️ Neden Kullanılır?

#### 1. Magic Numbers'dan Kurtulmak (Okunaklılık)
Yazılımda kodun içine rastgele serpiştirilmiş sayılara (404, 200 gibi) **"Magic Numbers"** denir ve bu kötü bir pratiktir.
* **Kötü Senaryo:** Kodun 10 farklı yerinde `if statusCode == 404` yazarsan, bir gün bu kodu güncellemek işkenceye döner.
* **İyi Senaryo (Enum):** `if error == .notFound` yazarsın. Kodun ne yaptığı bir bakışta anlaşılır. "404 neydi?" diye düşünmezsin.

#### 2. Type Safety (Tip Güvenliği)
Backend'den her zaman beklediğin sayılar gelmeyebilir. Enum kullanmak seni olmayan durumlara karşı korur.

Diyelim ki backend sana yanlışlıkla `999` diye bir kod gönderdi:
* **Enum Olmazsa:** Kodun 999 sayısını işlemeye çalışabilir ve beklenmedik hatalar verebilir.
* **Enum Olursa:** `HTTPError(rawValue: 999)` dediğinde sonuç `nil` döner. Swift sana *"Böyle bir durum tanımlı değil"* der ve sen de bunu `if let` veya `guard let` ile güvenli bir şekilde yönetirsin.

---

### 💻 Uygulamalı Karşılaştırma

**❌ Enum KULLANMADAN (Zor ve Hataya Açık):**
```swift
let incomingCode = 404

if incomingCode == 404 {
    print("Sayfa bulunamadı")
} else if incomingCode == 500 {
    print("Sunucu hatası")
}
// Sayıların ne anlama geldiğini ezbere bilmek zorundasın.
```

**✅ Enum KULLANARAK (Güvenli ve Okunaklı):**
```swift
enum HTTPError: Int {
    case notFound = 404
    case serverError = 500
    case unauthorized = 401
}

// Backend'den gelen veriyi tipe dönüştür
if let error = HTTPError(rawValue: 404) {
    // Artık sayılarla değil, anlamlı isimlerle konuşuyoruz:
    switch error {
    case .notFound:
        print("Sayfa bulunamadı")
    case .serverError:
        print("Sunucu hatası")
    case .unauthorized:
        print("Yetkisiz giriş")
    }
}
```

> **Özet:** Raw Value (Ham Değer), dış dünyadaki "ilkel" veriyi (sayı, string), Swift dünyasındaki "anlamlı" etiketlere dönüştüren bir güvenlik köprüsüdür.

<br><br>



