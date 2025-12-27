# 📘 Swift Veri Mimarisi: Value Types vs Reference Types

Swift dilinde her veri tipi bu iki kategoriden birine girer. Bu ayrım, verinin bellekte (RAM) nerede saklandığını ve kopyalanma davranışını belirler.

<br><br>

## 📑 İçindekiler

1. [Temel Ayrım ve Bellek Alanları](#1-temel-ayrım-ve-bellek-alanları)
2. [Reference Types (Class) Analizi: "Shared State"](#2-reference-types-class-analizi-shared-state)
3. [Identity (Kimlik) Kavramı – === Operatörü](#21-identity-kimlik-kavramı--operatörü)
4. [Mutable Shared State Problemi](#22-mutable-shared-state-problemi)
5. [Ne Zaman Class Kullanmalıyız?](#23-ne-zaman-class-kullanmalıyız)
6. [Class vs Struct Farkı](#3-class-vs-struct-farkı)
7. [📊 Karşılaştırma Tablosu](#-class-vs-struct--detaylı-karşılaştırma-tablosu)
8. [🎯 Mülakat Soruları ve Özet](#-ios-mülakat-soruları--class-vs-struct)

<br><br>

## 1. Temel Ayrım ve Bellek Alanları

Hafızayı iki ana bölge olarak yönetiriz: **Stack** ve **Heap**.



### A. Value Types (Değer Tipleri) → Stack (Yığın)
Verinin **kendisinin doğrudan saklandığı** yapılardır.
* **Kimler Buradadır:** `Struct`, `Enum`, `Int`, `String`, `Array`, `Dictionary`, `Tuple`.
* **Hafıza Davranışı:** Stack çok hızlıdır (LIFO). İşletim sistemi tarafından otomatik yönetilir. ARC maliyeti yoktur.
* **Davranış:** **Kopyalama (Copy)** mantığı ile çalışır.

### B. Reference Types (Referans Tipleri) → Heap (Küme)
Verinin kendisi değil, verinin adresi (**pointer**) saklanır.
* **Kimler Buradadır:** `Class`, `Closure`, `Actor`.
* **Hafıza Davranışı:** Heap büyük ve karmaşık bir havuzdur. Erişim Stack’e göre daha maliyetlidir.
* **Yönetim:** **ARC (Automatic Reference Counting)** tarafından yönetilir.
* **Davranış:** **Paylaşım (Shared Instance)** mantığı ile çalışır.

<br><br>

## 2. Reference Types (Class) Analizi: "Shared State"

Bir `class` örneğini başka bir değişkene atadığınızda, Swift yeni bir nesne yaratmaz. Sadece o nesnenin **hafıza adresini (reference)** kopyalar. Bu nedenle iki değişken de **aynı nesneyi** işaret eder.

**Teknik Terim:** *Pointer Copy* (İşaretçi Kopyalama)

```swift
class Bilgisayar {
    var ram: Int
    init(ram: Int) { self.ram = ram }
}

var mac1 = Bilgisayar(ram: 16)
var mac2 = mac1 // REFERANS kopyalandı.

mac2.ram = 32
print(mac1.ram) // ÇIKTI: 32 (mac1 de etkilendi!)
```

⚠️ **Risk:** Çoklu thread ortamlarında Reference Type’lar **Race Condition** hatalarına açıktır.

<br><br>

## 2.1 Identity (Kimlik) Kavramı – `===` Operatörü

Reference Type’ların en önemli özelliği **identity (kimlik)** kavramıdır. 
* `==` → **Değer eşitliği** (İçindekiler aynı mı?)
* `===` → **Aynı nesne mi?** (Hafızada aynı kutuya mı bakıyorlar?)

📌 **Not:** `===` operatörü sadece Class'lar için geçerlidir. Value Type’larda identity yoktur.

<br><br>

## 2.2 Mutable Shared State Problemi

Reference Type’lar **paylaşılan mutable state** oluşturur. Bir yerden değiştirilen veri, başka bir yeri **yan etkili (side-effect)** şekilde etkiler. Bu durum, debug sürecini zorlaştırır ve beklenmeyen UI bug’larına yol açar.

<br><br>

## 2.3 Ne Zaman `class` Kullanmalıyız?

#### ✅ `class` Kullan (Reference Type):
* Nesnenin bir **kimliği (identity)** varsa.
* Birden fazla bileşen aynı instance üzerinde çalışıyorsa (**paylaşılıyorsa**).
* `weak`, `unowned`, `delegate` gibi referans yönetimi gerekiyorsa.
* **Örnek:** `UIViewController`, `Coordinator`, `Manager`, `Service`.

#### ❌ `class` Kullanma (Struct Daha Uygun):
* Nesne sadece **veri taşıyorsa**.
* Nesnenin **immutable (değişmez)** olması gerekiyorsa.
* **Thread-safe** olması kritikse.
* **Örnek:** Model katmanı, DTO'lar.

<br><br>

## 3. Class vs Struct Farkı

* **Struct** → *Değerin kendisini temsil eder.* Önemli olan **ne olduğu**dur.
* **Class** → *Nesnenin kimliğini temsil eder.* Önemli olan **hangi nesne olduğu**dur.

### 🧩 Mimari Perspektif
Swift ekosisteminde kabul görmüş en sağlıklı yaklaşım şudur:
* **Model Katmanı** → `struct`
* **Controller / Manager / Coordinator** → `class`

<br><br>

## 📊 Class vs Struct – Detaylı Karşılaştırma Tablosu

| Özellik | `struct` (Value Type) | `class` (Reference Type) |
| :--- | :--- | :--- |
| **Bellek Bölgesi** | Stack | Heap |
| **Kopyalama** | Değer kopyalanır | Referans (Adres) kopyalanır |
| **Miras (Inheritance)** | Yok | Var |
| **ARC** | Yok | Var |
| **Yan Etki** | Yok (Güvenli) | Var (Dikkat gerektirir) |
| **Thread Safety** | Doğal olarak güvenli | Kontrol edilmeli |
| **Performans** | Çok Hızlı | Maliyetli |

<br><br>

### 🎯 iOS Mülakat Soruları – Class vs Struct

**❓ Soru:** “Class yerine neden Struct kullanmayı tercih edersin?”  
**Cevap:** Yan etki üretmezler, thread-safe davranışa yakındırlar ve kopyalama mantığı sayesinde verinin beklenmedik şekilde değişmesini engellerler.

**❓ Soru:** “SwiftUI neden Struct tabanlıdır?”  
**Cevap:** SwiftUI state bazlı çalışır. Struct’ların copy semantics'i, UI yeniden çizimlerinin (re-render) tutarlı ve performanslı olmasını sağlar.

**❓ Soru:** “Mülakat Tek Cümlelik Özet nedir?”  
**Cevap:** **“Struct veri içindir, Class davranış ve kimlik içindir.”**

<br><br>

### 📌 Kısa Özet
* **Struct** → Güvenli, basit, yan etkisiz.
* **Class** → Güçlü ama karmaşık (ARC yönetimi gerektirir).
* **Doğru seçim** → Temiz mimari ve hatasız uygulama.

<br><br>
