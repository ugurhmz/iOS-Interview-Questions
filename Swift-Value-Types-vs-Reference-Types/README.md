# 📘 Swift Veri Mimarisi: Value Types vs Reference Types

Swift dilinde her veri tipi bu iki kategoriden birine girer.  
Bu ayrım, verinin bellekte (RAM) nerede saklandığını ve kopyalanma davranışını belirler.

<br>

## 📑 İçindekiler

1. [Temel Ayrım ve Bellek Alanları](#1-temel-ayrım-ve-bellek-alanları)  
   - [Value Types (Değer Tipleri) → Stack (Yığın)](#a-value-types-değer-tipleri--stack-yığın)  
   - [Reference Types (Referans Tipleri) → Heap (Küme)](#b-reference-types-referans-tipleri--heap-küme)
2. [Reference Types (Class) Analizi: "Shared State"](#2-reference-types-class-analizi-shared-state)  
   - [Identity (Kimlik) Kavramı – `===` Operatörü](#21-identity-kimlik-kavramı---operatörü)  
   - [Mutable Shared State Problemi](#22-mutable-shared-state-problemi)  
   - [Ne Zaman Class Kullanmalıyız?](#23-ne-zaman-class-kullanmalıyız)
3. [Class vs Struct Farkı](#3-class-vs-struct-farkı)  
   - [Temel Kavramsal Ayrım](#-temel-kavramsal-ayrım)  
   - [Bellek ve Yaşam Döngüsü Perspektifi](#-bellek-ve-yaşam-döngüsü-perspektifi)  
   - [Veri Güvenliği ve Yan Etki (Side Effect)](#-veri-güvenliği-ve-yan-etki-side-effect)  
   - [Concurrency ve Thread Safety](#-concurrency-ve-thread-safety)  
   - [Mimari Perspektif (iOS Tarafı)](#-mimari-perspektif-ios-tarafı)
4. [📊 Class vs Struct – Detaylı Karşılaştırma Tablosu](#-class-vs-struct--detaylı-karşılaştırma-tablosu)
5. [📌 Mimari Özet (Altın Kural)](#-mimari-özet-altın-kural)
6. [🎯 Mülakat Tek Cümlelik Özet](#-mülakat-tek-cümlelik-özet)
7. [🎯 iOS Mülakat Soruları – Class vs Struct](#-ios-mülakat-soruları--class-vs-struct)
8. [📌 Kısa Özet](#-kısa-özet)

<br>

## 1. Temel Ayrım ve Bellek Alanları

Hafızayı iki ana bölge olarak yönetiriz: **Stack** ve **Heap**.



<br>

### A. Value Types (Değer Tipleri) → Stack (Yığın)

Verinin **kendisinin doğrudan saklandığı** yapılardır.

### Kimler Buradadır?
- `Struct`
- `Enum`
- `Int`, `Double`, `Bool`
- `Tuple`
- `String`
- `Array`
- `Dictionary`

### Hafıza Davranışı
- Stack bölgesinde tutulur.
- Stack çok hızlıdır (**LIFO – Last In First Out**).
- İşletim sistemi tarafından otomatik yönetilir.
- Değişken scope dışına çıktığı anda bellekten **anında silinir**.
- **ARC maliyeti yoktur.**

### Davranış
- **Kopyalama (Copy)** mantığı ile çalışır.

<br>

### B. Reference Types (Referans Tipleri) → Heap (Küme)

Verinin **kendisi değil**, verinin adresi (**pointer**) saklanır.

### Kimler Buradadır?
- `Class`
- `Closure`
- `Actor`

### Hafıza Davranışı
- Heap bölgesinde tutulur.
- Heap büyük ve karmaşık bir bellek havuzudur.
- Veriye erişim, Stack’e göre **daha maliyetlidir**.

### Yönetim
- **ARC (Automatic Reference Counting)** tarafından yönetilir.

### Davranış
- **Paylaşım (Shared Instance)** mantığı ile çalışır.

<br>
<br>

## 2. Reference Types (Class) Analizi: "Shared State"

Bir **Class** örneğini başka bir değişkene atadığınızda, Swift **yeni bir nesne yaratmaz**.  
Sadece o nesnenin **hafıza adresini (reference)** kopyalar.  
Bu nedenle iki değişken de **aynı nesneyi** işaret eder.



**Teknik Terim:** *Pointer Copy* (İşaretçi Kopyalama)

#### Swift Örneği

```swift
class Bilgisayar {
    var model: String
    var ram: Int
    
    init(model: String, ram: Int) {
        self.model = model
        self.ram = ram
    }
}

// 1. Heap'te bir nesne oluştu, 'mac1' buna referans veriyor.
var mac1 = Bilgisayar(model: "MacBook Pro", ram: 16)

// 2. REFERANS kopyalandı. 'mac2' de AYNI nesneye bakıyor.
var mac2 = mac1 

// 3. mac2 üzerinden RAM'i artırdık.
mac2.ram = 32

// SONUÇ: mac1 etkilendi mi?
print("Mac1 Ram: \(mac1.ram)")
// ÇIKTI: 32
// (Çünkü onlar aslında aynı bilgisayar.)
```

##### ⚠️ Risk
Çoklu thread (Multi-threading) ortamlarında, Reference Type’lar Race Condition hatalarına açıktır. Bunun nedeni:
- Aynı nesnenin birden fazla yerden, aynı anda, kontrolsüz şekilde değiştirilebilmesidir.
Bu yüzden paylaşılan mutable state içeren class’lar, concurrency açısından dikkatli tasarlanmalıdır.

<br>
<br>

## 2.1 Identity (Kimlik) Kavramı  `===` Operatörü

Reference Type’ların en önemli özelliği **identity (kimlik)** kavramıdır.  
İki değişkenin **aynı veriye mi**, yoksa sadece **eşit değerlere mi** sahip olduğunu ayırt edebiliriz.

Swift’te:
- `==` → **Değer eşitliği**
- `===` → **Aynı nesne mi? (Aynı hafıza adresi)**

```swift
class User {
    let id: Int
    init(id: Int) { self.id = id }
}

let user1 = User(id: 1)
let user2 = user1
let user3 = User(id: 1)

print(user1 === user2) // true  → aynı nesne
print(user1 === user3) // false → farklı nesneler
```

📌 Önemli:
- `===` operatörü sadece Class (Reference Type)’lar için vardır.
- Value Type’larda identity kavramı yoktur.

<br>
<br>

## 2.2 Mutable Shared State Problemi

Reference Type’lar **paylaşılan mutable state** oluşturur. Bu şu anlama gelir:

- Bir yerden değiştirilen veri  
- Başka bir yeri **yan etkili (side-effect)** şekilde etkiler  

```swift
func upgradeRAM(_ bilgisayar: Bilgisayar) {
    bilgisayar.ram += 16
}

upgradeRAM(mac1)
print(mac2.ram) // 48
```

<br>

##### 📉 Problem:
- Fonksiyon sadece `mac1`’i değiştiriyor gibi görünür ama aslında tüm referans sahiplerini etkiler.
- Bu durum: Debug sürecinin zorlaşmasına, beklenmeyen UI bug’larına, concurrency (eşzamanlılık) problemlerine yol açar.

<br>
<br>

## 2.3 Ne Zaman `class` Kullanmalıyız?

Reference Type’lar **bilinçli kullanıldığında** son derece güçlüdür.  
Ancak yanlış yerde kullanıldıklarında **yan etki (side-effect)**, **memory leak** ve
**concurrency** problemlerine yol açabilirler.

Bu yüzden `class` tercih etmeden önce kendine şu soruları sormalısın:

---

#### ✅ `class` Kullan (Reference Type)

Aşağıdaki durumlardan **en az biri** geçerliyse `class` doğru tercihtir:

- Nesnenin **bir kimliği (identity)** varsa  
  → Aynı nesnenin uygulamanın farklı yerlerinden **tek bir varlık olarak** görülmesi gerekiyorsa  

- Nesne **paylaşılıyorsa** → Birden fazla bileşen aynı instance üzerinde çalışıyorsa  

- Nesnenin **yaşam döngüsü** önemliyse  
  → Oluşturulması, tutulması ve yok edilmesi kontrol edilmeliyse  

- ARC ilişkileri gerekiyorsa  
  → `weak`, `unowned`, `delegate` gibi referans yönetimleri söz konusuysa  

📌 Tipik `class` örnekleri:
- `UIViewController`
- `Coordinator`
- `Manager`
- `Service`
- `Network / Cache / DataSource` katmanları

---

#### ❌ `class` Kullanma (Value Type Daha Uygun)

Aşağıdaki durumlarda `class` kullanmak **kötü bir tercihtir**:

- Nesne **sadece veri taşıyorsa** → Davranışı yoksa veya çok sınırlıysa  

- Nesnenin **immutable (değişmez)** olması gerekiyorsa  

- **Thread-safe** olması kritikse  
  → Çoklu thread ortamında güvenli çalışması gerekiyorsa  

- Yan etkisiz (side-effect free) davranış isteniyorsa  

📌 Bu durumlarda `struct` tercih edilmelidir.

<br>

## 2.4 Ne Zaman `struct` Kullanmalıyız?

Struct’lar **değer tipleri (Value Types)** oldukları için, çoğu durumda `class` yerine daha güvenli ve öngörülebilir bir kullanım sunarlar.  

Aşağıdaki durumlarda `struct` tercih edilmelidir:

---

#### ✅ `struct` Kullan (Value Type)

- Nesne **sadece veri taşıyorsa** → Davranışı yok veya çok sınırlıysa  

- Nesnenin **immutable (değişmez)** olması gerekiyorsa  

- **Thread-safe** olması kritikse  
  → Çoklu thread ortamında güvenli çalışması gerekiyorsa  

- Yan etkisiz (side-effect free) davranış isteniyorsa  

📌 Tipik `struct` örnekleri:
- Model katmanındaki veri objeleri
- Data Transfer Object (DTO)
- SwiftUI State veya ViewModel’lerde küçük immutable yapı taşları
- Immutable koleksiyonlar (`Array`, `Dictionary` gibi kopyalanabilir yapılar)

<br>

### 📌 Swift Ekosisteminde Kabul Görmüş Ayrım

Swift’te mimari olarak yaygın ve sağlıklı yaklaşım şudur:

- **Model katmanı** → `struct`  
- **Controller / Coordinator / Manager** → `class`

Bu ayrım sayesinde:

- Veriler **güvenli ve öngörülebilir** olur  
- Yan etkiler minimize edilir  
- Debug ve test süreçleri kolaylaşır  
- Concurrency problemleri büyük ölçüde azalır  

<br>

#### 🎯 Kuralı Netleştirelim

> **Davranışı olan, kimliği olan ve yönetilen nesneler → `class`** > **Sadece veri olan, kopyalanabilir ve güvenli nesneler → `struct`**

<br>
<br>

## 3 Class vs Struct Farkı

Swift’te `class` ve `struct` arasındaki fark yalnızca sözdizimsel değildir.  
Bu iki yapı, **bellek yönetimi**, **veri güvenliği**, **mimari kararlar** ve **eşzamanlılık (concurrency)** açısından kökten farklı davranır.

Bu farkı doğru anlamak, iOS mimarisinde doğru veri yapısını seçmenin temelidir.

---

### 🔹 Temel Kavramsal Ayrım

- **Struct** → *Değerin kendisini temsil eder* - **Class** → *Bir nesnenin kimliğini (identity) temsil eder*

Struct’ta önemli olan **ne olduğu**,  
Class’ta önemli olan **hangi nesne olduğu**dur.

---

### 🧠 Bellek ve Yaşam Döngüsü Perspektifi

**Struct (Value Type)** - Değer bazlıdır  
- Sahibi yoktur  
- Kapsam (scope) bittiğinde otomatik olarak yok olur  
- ARC ile izlenmez  
- Bellek yaşam döngüsü basit ve deterministiktir  

**Class (Reference Type)** - Kimlik sahibidir (identity)  
- Birden fazla yerden paylaşılabilir  
- Bellekten ne zaman silineceği referans sayısına bağlıdır  
- ARC tarafından yönetilir  
- Yanlış kullanıldığında memory leak riski taşır  

---

### 🔄 Veri Güvenliği ve Yan Etki (Side Effect)

**Struct**
- Her kullanım kendi kopyasıyla çalışır  
- Yan etki üretmez  
- Bir yerdeki değişiklik başka bir yeri etkilemez  
- Predictable (öngörülebilir) davranır  

**Class**
- Paylaşılan mutable state oluşturabilir  
- Bir yerdeki değişiklik başka yerleri etkileyebilir  
- Yan etki üretme riski yüksektir  
- Debug ve test süreçleri daha zordur  

---

### ⚙️ Concurrency ve Thread Safety

**Struct**
- Varsayılan olarak thread-safe davranışa daha yakındır  
- Aynı veriye eşzamanlı erişim riski düşüktür  
- Swift Concurrency (Sendable, Actor) ekosistemiyle doğal uyumludur  

**Class**
- Race condition riskine açıktır  
- Eşzamanlı erişim mutlaka kontrol edilmelidir  
- Genellikle `actor`, `lock` veya serial queue gerektirir  

---

### 🧩 Mimari Perspektif (iOS Tarafı)

**Struct tercih edilir:**
- Model katmanında  
- Data transfer objelerinde (DTO)  
- Immutable veri yapılarında  
- State yönetiminde (SwiftUI, Redux benzeri mimariler)

**Class tercih edilir:**
- ViewController  
- Coordinator  
- Manager / Service  
- Network, Cache, Session gibi yaşam döngüsü önemli nesnelerde  

📌 Swift ekosisteminde bu yüzden yaygın ayrım şudur:

- **Model → Struct** - **Controller / Manager → Class**

<br>
<br>

## 📊 Class vs Struct – Detaylı Karşılaştırma Tablosu

Aşağıdaki tablo, **Class** ve **Struct** arasındaki farkları yalnızca yüzeysel değil;  
**bellek**, **mimari**, **concurrency** ve **mülakat perspektifiyle** net şekilde karşılaştırır.

| Karşılaştırma Kriteri | `struct` (Value Type) | `class` (Reference Type) |
|:----------------------|:-----------------------|:--------------------------|
| **Temel Anlam** | Değerin kendisini temsil eder | Nesnenin kimliğini (identity) temsil eder |
| **Bellekte Tutulma** | Stack (çoğunlukla) | Heap |
| **Sahiplik (Ownership)** | Yok | Var |
| **Paylaşım** | Paylaşılmaz (kopyalanır) | Paylaşılır (aynı instance) |
| **Kopyalama Davranışı** | Copy Semantics | Reference (Pointer) Semantics |
| **Yan Etki (Side Effect)** | ❌ Üretmez | ⚠️ Üretebilir |
| **Debug Kolaylığı** | ✅ Çok kolay | ⚠️ Zor olabilir |
| **Thread Safety** | ✅ Doğal olarak daha güvenli | ❌ Race condition riski |
| **Concurrency Uyumu** | ✅ Swift Concurrency ile doğal | ⚠️ Ek kontrol gerekir |
| **ARC ile Yönetim** | ❌ Hayır | ✅ Evet |
| **`weak / unowned` Kullanımı** | ❌ Yok | ✅ Var |
| **Memory Leak Riski** | ❌ Yok | ⚠️ Yanlış kullanımda var |
| **Performans** | ✅ Hızlı ve deterministik | ⚠️ Daha maliyetli |
| **Immutable Kullanım** | ✅ Çok uygun | ⚠️ Zor |
| **Kimlik (Identity)** | ❌ Yok | ✅ Var |
| **Yaşam Döngüsü Kontrolü** | ❌ Yok | ✅ Var |
| **Kalıtım (Inheritance)** | ❌ Yok | ✅ Var |
| **Protokol Uyumu** | ✅ Var | ✅ Var |
| **SwiftUI Uygunluğu** | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **UIKit Uygunluğu** | ⭐⭐ | ⭐⭐⭐⭐⭐ |

<br>
<br>

### 📌 Mimari Özet (Altın Kural)

- **Struct kullan:** - Veri taşıyorsa  
  - Immutable olması gerekiyorsa  
  - Yan etki istenmiyorsa  
  - Concurrency önemliyse  

- **Class kullan:** - Nesnenin kimliği varsa  
  - Paylaşılan state gerekiyorsa  
  - Yaşam döngüsü kontrol edilecekse  
  - Delegate, weak, unowned ilişkileri gerekiyorsa  

<br>
<br>

### 🎯 Mülakat Tek Cümlelik Özet

> **“Struct veri içindir, Class davranış ve kimlik içindir.”**

Bu tabloyu zihninde netleştiren bir iOS Developer:
- Memory management’ı
- Concurrency risklerini
- Mimari kararları  
çok daha bilinçli verir.

<br>
<br>

### 🎯 iOS Mülakat Soruları – Class vs Struct

Aşağıdaki sorular bu konudan **doğrudan** gelir ve mülakatlarda sıkça sorulur.

<br>

#### ❓ Soru 1: “Class yerine neden Struct kullanmayı tercih edersin?”

**Cevap:** Struct’lar değer bazlıdır, yan etki üretmez ve thread-safe davranışa daha yakındır.  
Bu sayede debug, test ve concurrency yönetimi daha güvenlidir.

<br>

#### ❓ Soru 2: “Her yerde Struct kullansak olmaz mı?”

**Cevap:** Hayır. Kimliği olan, paylaşılan ve yaşam döngüsü önemli nesneler için Class gerekir.  
Örneğin ViewController, Service veya Manager gibi yapılar Struct ile temsil edilemez.

<br>

#### ❓ Soru 3: “Class kullanmanın en büyük riski nedir?”

**Cevap:** Paylaşılan mutable state nedeniyle side-effect üretmesi ve yanlış referans ilişkileri sonucu memory leak oluşmasıdır.

<br>

#### ❓ Soru 4: “SwiftUI neden Struct tabanlıdır?”

**Cevap:** Çünkü SwiftUI state bazlı çalışır.  
Struct’lar copy semantics sunduğu için UI yeniden çizimleri deterministic ve güvenlidir.

<br>

#### ❓ Soru 5: “Bir yapının Struct mı Class mı olması gerektiğine nasıl karar verirsin?”

**Cevap:** Şu soruları sorarım:
- Kimliği var mı?
- Paylaşılıyor mu?
- Yaşam döngüsü önemli mi?
- Yan etki üretmesi kabul edilebilir mi?

Eğer cevaplar *hayır* ise → Struct  
*evet* ise → Class

<br>
<br>

### 📌 Kısa Özet

- **Struct** → Güvenli, basit, yan etkisiz  
- **Class** → Güçlü ama dikkatli kullanılmalı  
- **Yanlış seçim** → Debug kabusu  
- **Doğru seçim** → Temiz mimari
