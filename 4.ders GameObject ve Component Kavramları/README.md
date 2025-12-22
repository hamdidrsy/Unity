# 4. Ders - GameObject ve Component Kavramları

## GameObject Nedir?

Unity'de sahnedeki her şey bir **GameObject**'tir. Karakterler, ağaçlar, ışıklar, kameralar, ses kaynakları - hepsi GameObject'tir.

### GameObject'in Temel Özellikleri
```
GameObject
├── Name (İsim)
├── Tag (Etiket)
├── Layer (Katman)
├── Static (Statik mi?)
└── Active (Aktif mi?)
```

### Boş GameObject
- Tek başına hiçbir şey yapmaz
- Sadece bir "konteyner" görevi görür
- Component'ler eklenerek işlevsel hale gelir
- Her GameObject'te zorunlu olarak **Transform** bileşeni bulunur

### GameObject Oluşturma Yolları
```
Hierarchy > Sağ Tık > Create Empty           → Boş GameObject
Hierarchy > Sağ Tık > 3D Object > Cube       → Hazır 3D nesne
GameObject menüsü > Create Empty             → Menüden oluşturma
Ctrl + Shift + N                             → Kısayol ile oluşturma
```

---

## Component (Bileşen) Sistemi

Unity, **Component-Based Architecture** (Bileşen Tabanlı Mimari) kullanır. Bu, oyun nesnelerine davranış ve özellik kazandırmanın temel yoludur.

### Component Nedir?
- GameObject'lere eklenen modüler parçalardır
- Her component belirli bir işlev sağlar
- Birden fazla component birleştirilerek karmaşık davranışlar oluşturulur

### Temel Component Türleri

| Component | Açıklama |
|-----------|----------|
| **Transform** | Pozisyon, rotasyon, ölçek (zorunlu) |
| **Mesh Filter** | 3D model verisi |
| **Mesh Renderer** | Modeli görünür kılar |
| **Collider** | Çarpışma algılama |
| **Rigidbody** | Fizik simülasyonu |
| **Audio Source** | Ses çalma |
| **Light** | Işık kaynağı |
| **Camera** | Görüntü yakalama |
| **Script** | Özel C# kodları |

### Component Ekleme
```
1. GameObject'i seçin
2. Inspector'da "Add Component" butonuna tıklayın
3. Arama yapın veya kategoriden seçin
4. Component eklenir
```

### Component Kaldırma
```
1. Component başlığında sağ tık
2. "Remove Component" seçin
```

---

## Transform Bileşeni

Her GameObject'in sahip olduğu zorunlu bileşendir. Nesnenin uzaydaki konumunu tanımlar.

### Transform Özellikleri

```csharp
// Position (Pozisyon) - Dünya koordinatları
transform.position = new Vector3(x, y, z);

// Local Position - Ebeveyne göre koordinatlar
transform.localPosition = new Vector3(x, y, z);

// Rotation (Döndürme) - Euler açıları
transform.rotation = Quaternion.Euler(x, y, z);

// Local Rotation
transform.localRotation = Quaternion.Euler(x, y, z);

// Scale (Ölçek)
transform.localScale = new Vector3(x, y, z);
```

### Koordinat Sistemi
```
        Y (Yukarı)
        |
        |
        |_______ X (Sağ)
       /
      /
     Z (İleri)
```

- **X ekseni**: Sağ (+) / Sol (-)
- **Y ekseni**: Yukarı (+) / Aşağı (-)
- **Z ekseni**: İleri (+) / Geri (-)

### World vs Local Space
- **World Space**: Dünyanın merkezi (0,0,0) referans alınır
- **Local Space**: Ebeveyn nesne referans alınır

```
Araba (World Position: 10, 0, 5)
└── Tekerlek (Local Position: 1, 0, 1)
    → World Position: 11, 0, 6
```

---

## Rigidbody - Fizik Bileşeni

Rigidbody, GameObject'e fizik motoru desteği ekler. Yerçekimi, kuvvetler ve çarpışmalar simüle edilir.

### Rigidbody Ekleme
```
Add Component > Physics > Rigidbody
```

### Rigidbody Özellikleri

| Özellik | Açıklama | Varsayılan |
|---------|----------|------------|
| **Mass** | Kütle (kg) | 1 |
| **Drag** | Hava direnci | 0 |
| **Angular Drag** | Dönme direnci | 0.05 |
| **Use Gravity** | Yerçekimi etkisi | true |
| **Is Kinematic** | Fizik motorunu devre dışı bırak | false |

### Is Kinematic Kullanımı
```
Is Kinematic = true
→ Fizik motoru etkilemez
→ Script ile kontrol edilir
→ Hareketli platformlar için ideal

Is Kinematic = false
→ Fizik motoru kontrol eder
→ Yerçekimi ve kuvvetler etkiler
→ Gerçekçi fizik davranışı
```

### Rigidbody Constraints (Kısıtlamalar)
Belirli eksenlerde hareketi veya dönmeyi kilitler:
```
Freeze Position: X, Y, Z    → Pozisyon kilitleme
Freeze Rotation: X, Y, Z    → Rotasyon kilitleme
```

### Script ile Rigidbody Kontrolü
```csharp
Rigidbody rb = GetComponent<Rigidbody>();

// Kuvvet uygulama
rb.AddForce(Vector3.forward * 10f);

// Anlık hız verme
rb.velocity = new Vector3(0, 5, 0);

// Tork (döndürme kuvveti)
rb.AddTorque(Vector3.up * 5f);
```

---

## Collider - Çarpışma Bileşeni

Collider, nesnelerin birbirleriyle çarpışmasını algılar. Fiziksel etkileşim için gereklidir.

### Collider Türleri

#### Temel Collider'lar (Primitive)
```
Box Collider      → Kutu şeklinde (en hızlı)
Sphere Collider   → Küre şeklinde (en hızlı)
Capsule Collider  → Kapsül şeklinde (karakterler için ideal)
```

#### Karmaşık Collider'lar
```
Mesh Collider     → 3D modelin şekline uyar (yavaş)
Terrain Collider  → Arazi için özel collider
Wheel Collider    → Araç tekerlekleri için
```

### Collider Özellikleri

| Özellik | Açıklama |
|---------|----------|
| **Is Trigger** | Fiziksel çarpışma yerine tetikleme |
| **Material** | Fizik materyali (sürtünme, sıçrama) |
| **Center** | Collider merkezi |
| **Size/Radius** | Boyut ayarları |

### Trigger vs Collider

```
Is Trigger = false (Normal Collider)
→ Fiziksel çarpışma olur
→ Nesneler birbirini iter
→ OnCollisionEnter/Stay/Exit

Is Trigger = true (Trigger)
→ Fiziksel çarpışma olmaz
→ Nesneler birbirinden geçer
→ OnTriggerEnter/Stay/Exit
→ Örnek: Coin toplama, checkpoint
```

### Collision Olayları (Script)
```csharp
// Normal çarpışma
void OnCollisionEnter(Collision collision)
{
    Debug.Log("Çarpışma başladı: " + collision.gameObject.name);
}

void OnCollisionStay(Collision collision)
{
    Debug.Log("Çarpışma devam ediyor");
}

void OnCollisionExit(Collision collision)
{
    Debug.Log("Çarpışma bitti");
}
```

### Trigger Olayları (Script)
```csharp
// Trigger geçişi
void OnTriggerEnter(Collider other)
{
    Debug.Log("Trigger'a girildi: " + other.gameObject.name);
}

void OnTriggerStay(Collider other)
{
    Debug.Log("Trigger içinde");
}

void OnTriggerExit(Collider other)
{
    Debug.Log("Trigger'dan çıkıldı");
}
```

---

## Physic Material (Fizik Materyali)

Yüzeylerin fiziksel özelliklerini tanımlar.

### Oluşturma
```
Project > Sağ Tık > Create > Physic Material
```

### Özellikler
| Özellik | Açıklama | Değer Aralığı |
|---------|----------|---------------|
| **Dynamic Friction** | Hareket halinde sürtünme | 0-1 |
| **Static Friction** | Durağan sürtünme | 0-1 |
| **Bounciness** | Sıçrama miktarı | 0-1 |
| **Friction Combine** | Sürtünme hesaplama modu | Average/Min/Max/Multiply |
| **Bounce Combine** | Sıçrama hesaplama modu | Average/Min/Max/Multiply |

### Örnek Materyaller
```
Buz:        Friction = 0.05, Bounciness = 0
Lastik:     Friction = 0.8,  Bounciness = 0.6
Zıplayan:   Friction = 0.3,  Bounciness = 1.0
```

---

## Pratik Uygulama: Zıplayan Top

Öğrendiklerimizi uygulayarak fiziksel bir top oluşturalım.

### Adım 1: Sahne Hazırlığı
1. Yeni sahne oluşturun veya mevcut sahneyi kullanın
2. Zemin için Plane ekleyin: Position (0, 0, 0)

### Adım 2: Top Oluşturma
1. **Hierarchy > 3D Object > Sphere**
2. İsim: "Ball"
3. Position: (0, 5, 0) → Zeminden yukarıda

### Adım 3: Rigidbody Ekleme
1. Ball seçili iken: **Add Component > Physics > Rigidbody**
2. Ayarlar:
   - Mass: 1
   - Use Gravity: ✓ (işaretli)

### Adım 4: Zıplayan Materyal
1. **Project > Create > Physic Material**
2. İsim: "BouncyMaterial"
3. Ayarlar:
   - Bounciness: 0.8
   - Bounce Combine: Maximum
4. Bu materyali Ball'un Sphere Collider > Material alanına sürükleyin

### Adım 5: Test
1. Play tuşuna basın
2. Top düşecek ve zıplayacak
3. Bounciness değerini değiştirerek farklı davranışlar deneyin

---

## Pratik Uygulama: Domino Etkisi

### Adım 1: Domino Taşı Oluşturma
1. **3D Object > Cube**
2. İsim: "Domino"
3. Scale: (0.2, 1, 0.5) → İnce dikdörtgen
4. **Add Component > Rigidbody**

### Adım 2: Domino Dizisi
1. Domino'yu seçin
2. **Ctrl + D** ile çoğaltın (5-6 adet)
3. Her birini Z ekseninde 0.6 birim arayla dizin:
   ```
   Domino1: Position (0, 0.5, 0)
   Domino2: Position (0, 0.5, 0.6)
   Domino3: Position (0, 0.5, 1.2)
   ...
   ```

### Adım 3: Tetikleyici Top
1. **3D Object > Sphere**
2. Position: (0, 1, -2) → Dominoların gerisinde
3. Scale: (0.5, 0.5, 0.5)
4. **Add Component > Rigidbody**

### Adım 4: İlk Hareketi Ver
Top için bir script oluşturun:
```csharp
using UnityEngine;

public class PushBall : MonoBehaviour
{
    public float pushForce = 5f;

    void Start()
    {
        Rigidbody rb = GetComponent<Rigidbody>();
        rb.AddForce(Vector3.forward * pushForce, ForceMode.Impulse);
    }
}
```

---

## Tag ve Layer Sistemi

### Tag (Etiket)
GameObject'leri tanımlamak için kullanılır.

```csharp
// Tag kontrolü
if (other.gameObject.tag == "Player")
{
    // Oyuncu ile etkileşim
}

// Veya CompareTag (daha hızlı)
if (other.gameObject.CompareTag("Player"))
{
    // Oyuncu ile etkileşim
}
```

#### Varsayılan Tag'ler
- Untagged
- Respawn
- Finish
- EditorOnly
- MainCamera
- Player
- GameController

#### Yeni Tag Ekleme
```
Inspector > Tag > Add Tag > + butonu
```

### Layer (Katman)
Fizik çarpışmalarını ve rendering'i kontrol eder.

```
Edit > Project Settings > Physics
→ Layer Collision Matrix
→ Hangi layer'ların çarpışacağını belirler
```

#### Kullanım Alanları
- Kamera görünürlüğü (Culling Mask)
- Fizik çarpışma filtreleme
- Raycast filtreleme
- Lighting layer'ları

---

## GetComponent Kullanımı

Diğer component'lere erişmek için kullanılır.

### Temel Kullanım
```csharp
// Aynı GameObject'teki component
Rigidbody rb = GetComponent<Rigidbody>();

// Çocuk nesnelerdeki component
Collider childCollider = GetComponentInChildren<Collider>();

// Ebeveyn nesnelerdeki component
Transform parentTransform = GetComponentInParent<Transform>();

// Tüm component'ler (dizi)
Collider[] allColliders = GetComponents<Collider>();
```

### Performans İpucu
```csharp
// YANLIŞ - Her frame'de arama yapar
void Update()
{
    GetComponent<Rigidbody>().AddForce(Vector3.up);
}

// DOĞRU - Bir kez ara, referansı sakla
private Rigidbody rb;

void Start()
{
    rb = GetComponent<Rigidbody>();
}

void Update()
{
    rb.AddForce(Vector3.up);
}
```

---

## Özet ve Kontrol Listesi

Bu derste öğrendiklerimiz:
- [x] GameObject kavramı ve özellikleri
- [x] Component tabanlı mimari
- [x] Transform bileşeni ve koordinat sistemi
- [x] Rigidbody ile fizik simülasyonu
- [x] Collider türleri ve kullanımı
- [x] Trigger vs Collider farkı
- [x] Physic Material oluşturma
- [x] Tag ve Layer sistemi
- [x] GetComponent kullanımı
- [x] Pratik uygulamalar (Zıplayan top, Domino)

---

## Alıştırmalar

1. Bir rampa oluşturun ve topun yuvarlanmasını izleyin
2. Farklı kütlelerde toplar oluşturun ve çarpışma davranışlarını gözlemleyin
3. Trigger kullanarak "coin toplama" sistemi yapın
4. Freeze Rotation kullanarak topun dönmesini engelleyin
5. Farklı Physic Material'lar ile denemeler yapın

---

## Sonraki Ders

5. Ders'te C# Scripting Temellerini öğreneceğiz. Değişkenler, fonksiyonlar, MonoBehaviour yaşam döngüsü ve temel oyun mantığı kodlamayı işleyeceğiz.

