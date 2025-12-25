# 🧠 Swift Memory Management  
## ARC, Retain Cycle ve Memory Leak Rehberi

Swift’te bellek yönetimi, özellikle **iOS uygulamalarında** performans, stabilite ve sürdürülebilir mimari için kritik bir konudur.  
Bu doküman; **ARC mantığını**, **retain cycle problemini** ve **memory leak** kavramını sade, okunabilir ve örnek odaklı şekilde tek bir README altında toplar.


## 📌 İçindekiler
- [ARC (Automatic Reference Counting) Nedir?](#1-temel-kavram-arc-automatic-reference-counting-)
- [Retain Cycle ve Memory Leak Problemi](#2-retain-cycle-referans-döngüsü-ve-memory-leak--sorunu)
- [Çözüm: weak ve unowned](#3-çözüm-weak-ve-unowned)
  - [weak Referans](#31-weak-referans)
  - [unowned Referans](#32-unowned-referans)
  - [Özet Karşılaştırma Tablosu](#33-özet-karşılaştırma-tablosu)
  - [Closure İçinde Retain Cycle](#34-closure-i̇çinde-retain-cycle-sıkça-unutulan)
- [Delegate Pattern ve weak & AnyObject İlişkisi](#35--delegate-pattern-ve-weak--anyobject-i̇lişkisi)
- [Hafıza Sızıntısı Testi](#-test-edelim-hafıza-sızıntısı-var-mı)
- [weak Kullanılmazsa Ne Olur?](#-eğer-weak-yapmasaydık-ne-olurdu)
- [Kritik Mülakat Soruları ve Cevapları](#-kritik-mülakat-soruları-ve-cevapları)

<br>


## 1. Temel Kavram: ARC (Automatic Reference Counting) 🧠

Swift, belleği yönetmek için **ARC (Automatic Reference Counting)** mekanizmasını kullanır.  
ARC, objelerin bellekte ne kadar süre yaşayacağını **referans sayısına** bakarak otomatik olarak belirler.

### 🔹 ARC Nasıl Çalışır?

- 🆕 Bir obje oluşturulduğunda (`init`) bellekte yer ayrılır  
- ⬆️ Obje bir değişken tarafından tutulduğunda referans sayısı artar  
- ➕ Aynı obje başka bir değişken tarafından tutulursa sayı tekrar artar  
- ⬇️ Değişken `nil` olduğunda referans sayısı azalır  
- 🗑️ **Referans sayısı 0 olduğunda**, ARC objeyi bellekten siler (`deinit` çağrılır)

### 🔍 Basit Bir Örnek

```swift
class Insan {
    let isim: String
    
    init(isim: String) {
        self.isim = isim
        print("\(isim) oluşturuldu (init).")
    }
    
    deinit {
        print("\(isim) hafızadan silindi (deinit).")
    }
}

// 1️⃣ Referans 1
var ahmet: Insan? = Insan(isim: "Ahmet")

// 2️⃣ Referans 2 (aynı obje)
var mehmet = ahmet

// ❓ Obje silinir mi?
ahmet = nil
// Hayır. Çünkü 'mehmet' hâlâ referans tutuyor.

// ✂️ Son referans da bırakılıyor
mehmet = nil
// ✅ Referans sayısı 0 → deinit çalışır
```
📌 Önemli Nokta:
ARC tamamen referans sayısına bakar. Tek bir güçlü referans bile kaldığı sürece obje bellekten silinmez.

<br>


## 2. Retain Cycle (Referans Döngüsü) ve Memory Leak 🚫 Sorunu
ARC otomatik çalışır ancak yanlış referans ilişkileri kurulduğunda bellek problemleri ortaya çıkar.

### ⚠️ Retain Cycle Nedir?

İki objenin birbirini **strong (güçlü)** referansla tutması durumudur. Bu döngü şu şekilde gerçekleşir:

* 🤝 **Obje A** → Obje B’yi tutar.
* 🤝 **Obje B** → Obje A’yı tutar.
* ❌ Hiçbiri referansı bırakamaz.
* 🧠 ARC referans sayısını **0**’a düşüremez.



📉 **Sonuç:** Memory Leak (Hafıza Sızıntısı)



**Senaryo:** Bir `Musteri` 👤 var ve bu müşterinin bir `KrediKarti` 💳 var. Kredi kartının da bir sahibi (`Musteri`) var.

```swift
class Musteri {
    let isim: String
    var kart: KrediKarti? // 🔗 Varsayılan olarak STRONG (Güçlü) referans
    
    init(isim: String) { self.isim = isim }
    deinit { print("\(isim) silindi.") }
}

class KrediKarti {
    let numara: String
    var sahip: Musteri? // 🔗 Varsayılan olarak STRONG (Güçlü) referans
    
    init(numara: String) { self.numara = numara }
    deinit { print("Kart \(numara) silindi.") }
}

var ali: Musteri? = Musteri(isim: "Ali")
var alisKarti: KrediKarti? = KrediKarti(numara: "1234")

// 🔄 Döngü Başlıyor:
ali?.kart = alisKarti // Ali kartı tutuyor (Strong)
alisKarti?.sahip = ali // Kart Ali'yi tutuyor (Strong)

// ✂️ Şimdi ikisini de nil yapalım:
ali = nil
alisKarti = nil

// ❌ SONUÇ: HİÇBİR ŞEY YAZDIRILMADI!
// 'deinit' metodları çalışmadı. Çünkü Ali kartı bırakmadı, Kart Ali'yi bırakmadı.
// Bu bir MEMORY LEAK (Hafıza Sızıntısı) durumudur.
```
<br> 

## 3. Çözüm: `weak` ve `unowned`

Retain cycle problemini çözmek için referanslardan **en az birinin güçlü (strong) olmaması** gerekir.  
Yani şu mesajı vermeliyiz:

> “Bu objeyi kullanıyorum ama onun bellekte hayatta kalmasından ben sorumlu değilim.”

Swift’te bunun için iki anahtar kelime vardır:

- `weak`
- `unowned`

---

### 3.1 `weak` Referans

`weak` referanslar, ARC’nin referans sayısını **artırmaz**.

#### Özellikleri

- Referans sayısını artırmaz  
- Objeyi hayatta tutmaz  
- Obje silinirse, `weak` referans **otomatik olarak `nil` olur**  
- Bu yüzden `weak` **her zaman Optional (`?`) olmak zorundadır**

#### Ne Zaman Kullanılır?

Genellikle:
- **Child (bağımlı)** nesne  
- **Parent (sahip)** nesneyi  
`weak` olarak tutar.

Önceki örnekte **KrediKarti**, **Musteri**’yi zayıf referansla tutmalıdır.

#### Çözüm Uygulaması

```swift
class KrediKarti {
    let numara: String

    // ÇÖZÜM: weak referans
    weak var sahip: Musteri?

    init(numara: String) {
        self.numara = numara
    }

    deinit {
        print("Kart \(numara) silindi.")
    }
}
// ... Önceki kodun aynısını çalıştırırsak ...
// ali = nil dediğimiz anda, KrediKarti Ali'yi tutmadığı (weak olduğu) için
// Ali hafızadan silinir. Ali silinince Kartı tutan kimse kalmaz, Kart da silinir.
// Çıktı:
// Ali silindi.
// Kart 1234 silindi.

```
<br> 

### 3.2 `unowned` Referans

`unowned` referanslar, `weak` gibi ARC’nin referans sayısını **artırmaz**;  
ancak **daha riskli** bir kullanım sunar.

#### Özellikleri

- Referans sayısını artırmaz  
- **Optional OLAMAZ**  
- Şu iddiayı yaparsınız:

> “Ben hayattaysam, bu obje de kesinlikle hayattadır.”

- Eğer `unowned` referans verilen obje silinirse ve erişilmeye çalışılırsa:  
  **👉 Uygulama CRASH olur**

---

#### Ne Zaman Kullanılır?

İki obje:
- Birbirine **çok sıkı bağlıysa**  
- Biri olmadan diğeri **anlamsızsa**

---

#### Örnek Senaryo: Öğrenci – Kimlik Kartı

Kimlik kartı öğrencisiz olamaz.  
Kimlik kartı varsa, öğrencinin de bellekte olduğu varsayılır.

```swift
class Ogrenci {
    let isim: String
    var kimlik: KimlikKarti?

    init(isim: String) {
        self.isim = isim
    }

    deinit {
        print("Öğrenci silindi")
    }
}

class KimlikKarti {
    let id: Int

    // Kimlik kartının sahibi nil olamaz
    // Kimlik kartı varsa, öğrenci de bellektedir varsayımı yapılır
    unowned let sahip: Ogrenci

    init(id: Int, sahip: Ogrenci) {
        self.id = id
        self.sahip = sahip
    }

    deinit {
        print("Kimlik silindi")
    }
}
```
<br>

### 3.3 Özet Karşılaştırma Tablosu

Konuyu netleştirmek için bu tabloyu zihninde canlandırabilirsin:

| Özellik                | `strong`                | `weak`                         | `unowned`                          |
|------------------------|-------------------------|---------------------------------|------------------------------------|
| Referans sayısını artırır mı | ✅ Evet                  | ❌ Hayır                        | ❌ Hayır                           |
| Optional olabilir mi   | ❌ Hayır                 | ✅ Evet (`?`)                   | ❌ Hayır                           |
| Obje silinirse ne olur | Silinmez                | Otomatik `nil` olur            | ❌ **CRASH**                       |
| Güvenlik seviyesi      | Yüksek                  | En güvenli                     | ⚠️ Riskli                          |
| Kullanım amacı         | Normal sahiplik         | Hayatta olup olmayacağı belirsiz ilişkiler | Hayat döngüsü kesin bağlı ilişkiler |


<br>

### 3.4 Closure İçinde Retain Cycle (Sıkça Unutulan!)

Sadece sınıflar arasında değil, **Closure** kullanırken de döngü oluşur.  
Çünkü Closure'lar da referans tipidir (**reference type**).

Eğer bir class içinde bir closure tanımlar ve o closure içinde `self` (sınıfın kendisi) kullanırsan, döngü oluşur:
Class -> Closure tutar -> Closure da self’i (Class’ı) tutar

---

#### Sorunlu Kod

```swift
class Islem {
    var tamamlamaBlogu: (() -> Void)?
    var isim = "İşlem A"
    
    func baslat() {
        tamamlamaBlogu = {
            // self.isim diyerek 'Islem' objesini güçlü tutuyoruz!
            print("\(self.isim) bitti.")
        }
    }
    
    deinit {
        print("Islem silindi")
    }
}
```

<br>

#### Çözüm: [weak self]
```swift
func baslat() {
    // [weak self] ekledik. self artık içeride optional oldu.
    tamamlamaBlogu = { [weak self] in
        // self optional olduğu için ? ile açıyoruz veya guard let yapıyoruz.
        print("\(self?.isim ?? "Bilinmiyor") bitti.")
    }
}

```

<br>

### Sonuç

Bir iOS Developer olarak şu kuralı benimsemelisin:

> **“Bir objenin sahibi kim?”**  
> Eğer hiyerarşik bir yapı varsa (örneğin: `TableView → Cell`),  
> **üstteki alttakini `strong`, alttaki üsttekini `weak`** tutmalıdır  
> (Delegate Pattern).

**Delegate Pattern (Delegasyon Deseni)**, az önce öğrendiğimiz `weak` anahtar kelimesinin
gerçek hayatta **en çok kullanıldığı** yerdir.

Bir iOS uygulamasında `UITableView`, `UICollectionView`, `UITextField` gibi bileşenlerin
tamamı bu mantıkla çalışır.

📌 Eğer bu ilişkiyi net anlarsan, **UIKit’in yaklaşık %50’sini çözmüşsün** demektir.

<br>

<br>

## 3.5  Delegate Pattern ve `weak` & `AnyObject` İlişkisi

**Delegate Pattern**, iOS geliştirmede retain cycle problemini çözmek için kullanılan
en temel ve en yaygın tasarım desenlerinden biridir.

Bu desenin temel amacı şudur:

> Bir obje, yaptığı işi başka bir objeye **bildirmek ister**  
> ama o objenin **sahibi olmak istemez**.

---

#### Delegate Pattern Neden Retain Cycle Üretir?

Varsayılan durumda Swift’te referanslar **strong**’dur.

Eğer:
- Parent obje (örneğin `ViewController`)
- Child obje (örneğin `CustomView` veya `Cell`)

birbirini **strong** referansla tutarsa, retain cycle oluşur.

Bu yüzden delegate ilişkilerinde **delegate her zaman `weak` tutulur**.

<br>


#### Doğru Delegate Tanımı

📌 **Çok Önemli Bir Detay: `AnyObject` Kullanımı**

Bir protocol’ü `weak` olarak tutabilmek için **mutlaka `AnyObject` ile sınırlandırmamız gerekir**.  
Bunun nedeni, Swift’te **sadece class’ların (reference type)** `weak` olabilmesidir.

- `class` → `weak` olabilir  
- `struct` → ❌ `weak` olamaz  
- `enum` → ❌ `weak` olamaz  

Eğer bir protocol:
- `struct` veya `enum` tarafından uygulanabilirse  
- onu `weak` olarak tanımlamak **derleme hatası** verir

Bu yüzden delegate protocol’leri **her zaman** `AnyObject` ile yazılır.

#### 1️⃣ Önce Protocol Tanımlanır (`AnyObject` ile)

<br>

```swift
// Stajyerin yapıp haber vereceği işlerin listesi
protocol StajyerDelegate: AnyObject {
    // ': AnyObject' sayesinde bu protokolü uygulayan referans 'weak' olabilir
    func kahveIsmarladim(tur: String)
}
```

<br>

#### 2️⃣ Delegate Değişkeni `weak` Olarak Tanımlanır

Protocol tanımlandıktan sonra, delegate’in tutulacağı yerde  
**mutlaka `weak` anahtar kelimesi kullanılmalıdır**.

Bunun sebebi:
- Delegate genellikle **üst nesneyi** işaret eder
- Güçlü (`strong`) tutulursa **retain cycle** oluşur
- `weak` kullanımı bu döngüyü kırar

```swift
class Stajyer {
    // KİLİT NOKTA: Burası 'weak' olmalı!
    // Eğer 'weak' olmazsa, Patron stajyeri tutarken, Stajyer de patronu tutar -> LEAK!
    weak var delegate: StajyerDelegate? 
    
    func kahveAlmayaGit() {
        print("Stajyer: Kahve almaya gidiyorum...")
        delegate?.kahveIsmarladim(tur: "Filtre Kahve")   // İş bitti, patrona (delegate) haber verelim
    }
}
```

<br>

- 📌 delegate değişkeninin Optional (?) olmasının sebebi, weak referansların obje silindiğinde otomatik olarak nil olmasıdır.

<br>

#### 3️⃣ Patron (İşi Veren / Parent)

**Patron**, stajyeri **sahiplenir (strong)** ve delegate protocol’ündeki işleri karşılar.  
Bu yapı, Delegate Pattern’in **doğru ve klasik** kullanımını temsil eder.

- Patron → Stajyeri **strong** tutar  
- Stajyer → Patronu **weak delegate** olarak tutar  
- Böylece retain cycle oluşmaz ✅

```swift
class Patron: StajyerDelegate {
    var stajyer: Stajyer? // Patron stajyeri STRONG tutuyor (normal davranış)
    
    init() {
        stajyer = Stajyer()
        stajyer?.delegate = self // Stajyerin rapor vereceği kişi BENİM diyoruz
    }
    
    func stajyereIsVer() {
        print("Patron: Bana kahve al.")
        stajyer?.kahveAlmayaGit()
    }

    // Stajyer işi bitirip haber verdiğinde burası çalışır
    func kahveIsmarladim(tur: String) {
        print("Patron: Ooo harika, \(tur) gelmiş. Teşekkürler!")
    }
    
    deinit {
        print("Patron ofisten ayrıldı (Silindi).")
    }
}
```
- 📌 Bu ilişkide sahiplik tek yönlüdür: Patron stajyeri yönetir, stajyer patronu sadece haber verir.

<br>

### 🧪 Test Edelim: Hafıza Sızıntısı Var mı?

Şimdi bu sistemi çalıştıralım ve `weak` kullandığımız için objelerin **doğru şekilde bellekten silindiğini** görelim.

```swift
// Bir scope (alan) oluşturalım ki bitince değişkenler silinsin
do {
    let mehmetBey = Patron()
    mehmetBey.stajyereIsVer()
}
 
// Bu süslü parantez } bittiğinde 'mehmetBey' değişkeni yok olacak.
// mehmetBey silinince, Stajyer de (Patron onu tuttuğu için) serbest kalmalı.
```

<br>

### ❌ Eğer `weak` Yapmasaydık Ne Olurdu?

Eğer `Stajyer` sınıfında  
`weak var delegate` yerine sadece `var delegate` yazsaydık:

1. `mehmetBey` (**Patron**) → `stajyer`i **strong** tutar  
2. `stajyer` → `delegate` (yani `mehmetBey`i) **strong** tutar  
3. `do {}` bloğu bittiğinde sistem `mehmetBey`i silmek ister  
4. Ancak `Stajyer` der ki: *“Ben onu tutuyorum, silemezsin.”*  
5. Sistem bu kez `Stajyer`i silmek ister  
6. Patron der ki: *“Ben onu tutuyorum, silemezsin.”*  
7. **Sonuç:** `deinit` metodu **ASLA** çalışmaz  
8. Patron ve Stajyer **sonsuz süre RAM’de kalır** → **Memory Leak**

---

#### 🎯 Özet: Mülakat Tiyosu

Bir mülakatta sana  **“Delegate Pattern kullanırken nelere dikkat edersin?”**  diye sorulursa, şu cevabı net şekilde ver:

> **“Delegate tanımlarken mutlaka `weak` kullanırım.  
> Bunu yapabilmek için de protocol’ümü `AnyObject`’ten türetirim.  Böylece Parent–Child arasında retain cycle oluşmasını engellerim.”**

<br>

### 🎯 Kritik Mülakat Soruları ve Cevapları

Bu konuyla ilgili mülakatlarda karşına çıkabilecek **en teknik 4 soru** ve **ideal cevapları**:

---

### ❓ Soru 1: *“ARC ile Garbage Collection (Çöp Toplayıcı) arasındaki fark nedir?”*

- **Cevap:**  
  Garbage Collection (Java, C# gibi dillerde) **runtime** sırasında periyodik olarak çalışır ve
  kullanılmayan nesneleri tarayıp temizler. Bu tarama işlemi CPU üzerinde **anlık performans yükleri**
  oluşturabilir.  

  **ARC** ise **derleme zamanında (compile time)** çalışır. Swift derleyicisi kodu analiz eder ve
  `retain / release` çağrılarını otomatik olarak üretip koda ekler.
  Runtime’da ekstra bir tarama yapılmaz; bu nedenle **daha performanslı, deterministik ve öngörülebilir**dir.

---

### ❓ Soru 2: *“Ne zaman `unowned`, ne zaman `weak` kullanırsın?”*

- **Cevap:**  
  Eğer referans verdiğim nesnenin, benim nesnemden **önce silinme ihtimali varsa**
  (yaşam döngüleri bağımsızsa) **`weak`** kullanırım ve `nil` kontrolü yaparım.  

  Ancak referans verdiğim nesnenin, benim nesnem hayatta olduğu sürece
  **kesinlikle var olacağını biliyorsam**
  (örneğin `init` içinde atanıyorsa ve asla `nil` olmayacaksa) **`unowned`** kullanırım.  

  `unowned`, optional olmadığı için `nil` kontrolü maliyetini ortadan kaldırır
  ancak yanlış kullanılırsa **CRASH** riski vardır.

---

### ❓ Soru 3: *“Delegate Pattern’da delegate değişkeni neden `weak` tanımlanır?”*

- **Cevap:**  
  **Retain Cycle’ı önlemek için.**  
  Tipik olarak bir `ViewController` (Parent), bir `TableView` veya `Cell`’i (Child) **strong** tutar.
  Eğer child da delegate aracılığıyla parent’ı **strong** tutarsa,
  iki obje birbirini bırakamaz ve **memory leak** oluşur.  

  `weak var delegate` kullanarak bu referans döngüsünü kırarız.

---

### ❓ Soru 4: *“Memory Leak olup olmadığını nasıl tespit edersin?”*

- **Cevap:**  
  Xcode’daki **Instruments → Leaks** profilini kullanırım.
  Ayrıca **Memory Graph Debugger** ile nesneler arasındaki referans ilişkilerini
  görsel olarak incelerim; retain cycle’lar genellikle ünlem işaretiyle işaretlenir.  

  Geliştirme sırasında kritik sınıfların `deinit` metodlarına `print` koyarak
  objelerin gerçekten serbest bırakılıp bırakılmadığını da kontrol ederim.

<br>

