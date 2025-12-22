# 5. Ders - C# Scripting Temelleri

## Script Nedir?

Script, GameObject'lere davranış kazandıran C# kod dosyalarıdır. Unity'de tüm oyun mantığı, karakter kontrolü ve etkileşimler script'ler aracılığıyla yapılır.

### İlk Script Oluşturma
```
Project Panel > Sağ Tık > Create > C# Script
```

### Script Yapısı
```csharp
using UnityEngine;

public class MyFirstScript : MonoBehaviour
{
    // Değişkenler buraya

    void Start()
    {
        // Oyun başladığında bir kez çalışır
    }

    void Update()
    {
        // Her frame'de çalışır
    }
}
```

### Önemli Kurallar
1. **Script adı = Class adı** olmalı (büyük/küçük harf dahil)
2. Script'i kullanmak için bir GameObject'e ekleyin
3. `MonoBehaviour`'dan türetilmiş olmalı

---

## Değişkenler ve Veri Tipleri

### Temel Veri Tipleri

```csharp
// Tam sayılar
int score = 100;              // -2.1 milyar ile +2.1 milyar arası
long bigNumber = 9999999999;  // Çok büyük sayılar için

// Ondalıklı sayılar
float speed = 5.5f;           // 'f' son eki zorunlu
double preciseValue = 3.14159265359;

// Metin
string playerName = "Oyuncu1";
char letter = 'A';            // Tek karakter

// Mantıksal
bool isAlive = true;
bool isGameOver = false;
```

### Unity'ye Özel Tipler

```csharp
// 2D ve 3D vektörler
Vector2 position2D = new Vector2(1.5f, 2.0f);
Vector3 position3D = new Vector3(1.0f, 2.0f, 3.0f);

// Renk
Color red = Color.red;
Color custom = new Color(0.5f, 0.8f, 0.2f, 1.0f); // RGBA

// Quaternion (rotasyon)
Quaternion rotation = Quaternion.identity;

// GameObject ve Component referansları
GameObject player;
Transform playerTransform;
Rigidbody rb;
```

### Değişken Erişim Belirleyicileri

```csharp
public int health = 100;      // Her yerden erişilebilir, Inspector'da görünür
private float speed = 5f;     // Sadece bu script içinden erişilebilir
protected int armor = 50;     // Bu script ve türetilen sınıflardan erişilebilir

[SerializeField]
private int secretValue = 42; // Private ama Inspector'da görünür (önerilen yöntem)

[HideInInspector]
public int hiddenPublic = 10; // Public ama Inspector'da gizli
```

### SerializeField Neden Önemli?
```csharp
// YANLIŞ - Encapsulation bozulur
public float speed = 5f;

// DOĞRU - Encapsulation korunur, Inspector'da düzenlenebilir
[SerializeField]
private float speed = 5f;
```

---

## MonoBehaviour Yaşam Döngüsü

Unity'de script'ler belirli bir sırayla çalışan özel fonksiyonlara sahiptir.

### Yaşam Döngüsü Sırası

```
┌─────────────────────────────────────────────────────────┐
│                    BAŞLANGIÇ                            │
├─────────────────────────────────────────────────────────┤
│ Awake()        → İlk çağrılan, nesne aktif olmasa bile  │
│ OnEnable()     → Nesne aktif olduğunda                  │
│ Start()        → İlk frame'den önce, bir kez           │
├─────────────────────────────────────────────────────────┤
│                    GÜNCELLEME DÖNGÜSÜ                   │
├─────────────────────────────────────────────────────────┤
│ FixedUpdate()  → Sabit aralıklarla (fizik için)        │
│ Update()       → Her frame'de bir kez                   │
│ LateUpdate()   → Update'den sonra (kamera takibi için) │
├─────────────────────────────────────────────────────────┤
│                    SONLANMA                             │
├─────────────────────────────────────────────────────────┤
│ OnDisable()    → Nesne deaktif olduğunda               │
│ OnDestroy()    → Nesne yok edildiğinde                 │
└─────────────────────────────────────────────────────────┘
```

### Detaylı Açıklamalar

```csharp
void Awake()
{
    // - Sahnedeki tüm nesneler için Start'tan ÖNCE çağrılır
    // - Referansları ayarlamak için ideal
    // - Nesne deaktif olsa bile çalışır
    rb = GetComponent<Rigidbody>();
}

void Start()
{
    // - Awake'den sonra, ilk Update'den önce
    // - Sadece nesne aktifse çalışır
    // - Diğer nesnelere bağımlı başlatmalar için
    Debug.Log("Oyun başladı!");
}

void Update()
{
    // - Her frame'de çalışır
    // - Frame rate'e bağlı (değişken)
    // - Input kontrolü, hareket, oyun mantığı
    // - Time.deltaTime kullanımı önemli!
}

void FixedUpdate()
{
    // - Sabit aralıklarla çalışır (varsayılan: 0.02 saniye)
    // - Fizik işlemleri için (Rigidbody)
    // - Time.fixedDeltaTime kullanılır
}

void LateUpdate()
{
    // - Tüm Update'ler bittikten sonra
    // - Kamera takibi için ideal
    // - Animasyon sonrası düzeltmeler
}
```

### Update vs FixedUpdate

```csharp
// YANLIŞ - Fizik işlemi Update'de
void Update()
{
    rb.AddForce(Vector3.forward * 10f); // Tutarsız davranış!
}

// DOĞRU - Fizik işlemi FixedUpdate'de
void FixedUpdate()
{
    rb.AddForce(Vector3.forward * 10f); // Tutarlı davranış
}

// DOĞRU - Hareket Update'de ama deltaTime ile
void Update()
{
    transform.Translate(Vector3.forward * speed * Time.deltaTime);
}
```

---

## Koşul İfadeleri

### if-else Yapısı

```csharp
int health = 75;

if (health > 80)
{
    Debug.Log("Sağlık iyi!");
}
else if (health > 40)
{
    Debug.Log("Sağlık orta.");
}
else if (health > 0)
{
    Debug.Log("Sağlık kritik!");
}
else
{
    Debug.Log("Öldün!");
}
```

### Karşılaştırma Operatörleri

```csharp
==    // Eşit mi?
!=    // Eşit değil mi?
>     // Büyük mü?
<     // Küçük mü?
>=    // Büyük veya eşit mi?
<=    // Küçük veya eşit mi?
```

### Mantıksal Operatörler

```csharp
&&    // VE (AND) - Her iki koşul da doğru olmalı
||    // VEYA (OR) - En az biri doğru olmalı
!     // DEĞİL (NOT) - Koşulu tersine çevir

// Örnek
if (health > 0 && hasWeapon)
{
    Debug.Log("Savaşabilirsin!");
}

if (isInWater || isOnFire)
{
    Debug.Log("Tehlikedesin!");
}

if (!isGameOver)
{
    Debug.Log("Oyun devam ediyor.");
}
```

### switch-case Yapısı

```csharp
int weaponType = 2;

switch (weaponType)
{
    case 0:
        Debug.Log("Kılıç seçildi");
        break;
    case 1:
        Debug.Log("Yay seçildi");
        break;
    case 2:
        Debug.Log("Asa seçildi");
        break;
    default:
        Debug.Log("Bilinmeyen silah");
        break;
}
```

### Ternary Operatör (Kısa if-else)

```csharp
// Uzun yol
string status;
if (health > 50)
    status = "Sağlıklı";
else
    status = "Yaralı";

// Kısa yol (Ternary)
string status = health > 50 ? "Sağlıklı" : "Yaralı";
```

---

## Döngüler

### for Döngüsü

```csharp
// 0'dan 9'a kadar sayıları yazdır
for (int i = 0; i < 10; i++)
{
    Debug.Log("Sayı: " + i);
}

// 10'dan geriye say
for (int i = 10; i > 0; i--)
{
    Debug.Log("Geri sayım: " + i);
}
```

### while Döngüsü

```csharp
int count = 0;

while (count < 5)
{
    Debug.Log("Count: " + count);
    count++;
}
```

### do-while Döngüsü

```csharp
int value = 0;

do
{
    Debug.Log("Value: " + value);
    value++;
} while (value < 3);

// En az bir kez çalışır!
```

### foreach Döngüsü

```csharp
string[] enemies = { "Goblin", "Orc", "Troll" };

foreach (string enemy in enemies)
{
    Debug.Log("Düşman: " + enemy);
}
```

### Döngü Kontrolleri

```csharp
// break - Döngüyü tamamen sonlandır
for (int i = 0; i < 100; i++)
{
    if (i == 5)
        break;  // Döngüden çık
    Debug.Log(i);
}
// Çıktı: 0, 1, 2, 3, 4

// continue - Bu iterasyonu atla, sonrakine geç
for (int i = 0; i < 5; i++)
{
    if (i == 2)
        continue;  // 2'yi atla
    Debug.Log(i);
}
// Çıktı: 0, 1, 3, 4
```

---

## Diziler ve Listeler

### Diziler (Arrays)

```csharp
// Dizi tanımlama
int[] scores = new int[5];                    // 5 elemanlı boş dizi
string[] names = { "Ali", "Veli", "Ayşe" };   // Başlangıç değerli
float[] values = new float[] { 1.5f, 2.5f };  // Alternatif sözdizimi

// Elemana erişim (0'dan başlar!)
Debug.Log(names[0]);  // "Ali"
Debug.Log(names[2]);  // "Ayşe"

// Eleman değiştirme
names[1] = "Mehmet";

// Dizi uzunluğu
int length = names.Length;  // 3

// Dizi üzerinde döngü
for (int i = 0; i < names.Length; i++)
{
    Debug.Log(names[i]);
}
```

### Listeler (List)

```csharp
using System.Collections.Generic;  // Gerekli!

// Liste tanımlama
List<string> inventory = new List<string>();

// Eleman ekleme
inventory.Add("Kılıç");
inventory.Add("Kalkan");
inventory.Add("İksir");

// Eleman çıkarma
inventory.Remove("Kalkan");

// Belirli indeksten çıkarma
inventory.RemoveAt(0);

// Eleman sayısı
int count = inventory.Count;

// Eleman var mı kontrolü
if (inventory.Contains("İksir"))
{
    Debug.Log("İksir var!");
}

// Listeyi temizleme
inventory.Clear();
```

### Dizi vs Liste

| Özellik | Dizi | Liste |
|---------|------|-------|
| Boyut | Sabit | Dinamik |
| Performans | Daha hızlı | Biraz yavaş |
| Esneklik | Düşük | Yüksek |
| Kullanım | Boyut biliniyorsa | Boyut değişkenise |

---

## Fonksiyonlar (Metodlar)

### Temel Fonksiyon Yapısı

```csharp
// Değer döndürmeyen fonksiyon
void SayHello()
{
    Debug.Log("Merhaba!");
}

// Değer döndüren fonksiyon
int AddNumbers(int a, int b)
{
    return a + b;
}

// Parametreli fonksiyon
void TakeDamage(int damage)
{
    health -= damage;
    Debug.Log("Hasar alındı: " + damage);
}

// Çoklu parametre
void SpawnEnemy(string enemyType, Vector3 position, int health)
{
    Debug.Log($"{enemyType} oluşturuldu, HP: {health}");
}
```

### Fonksiyon Çağırma

```csharp
void Start()
{
    SayHello();                        // "Merhaba!"

    int result = AddNumbers(5, 3);     // result = 8

    TakeDamage(25);                    // Hasar alındı: 25

    SpawnEnemy("Goblin", Vector3.zero, 100);
}
```

### Varsayılan Parametre Değerleri

```csharp
void Attack(int damage = 10, bool isCritical = false)
{
    if (isCritical)
        damage *= 2;
    Debug.Log("Hasar: " + damage);
}

// Kullanım
Attack();              // damage=10, isCritical=false
Attack(25);            // damage=25, isCritical=false
Attack(25, true);      // damage=25, isCritical=true
```

### ref ve out Parametreleri

```csharp
// ref - Değeri değiştir ve geri döndür
void DoubleValue(ref int value)
{
    value *= 2;
}

int number = 5;
DoubleValue(ref number);
Debug.Log(number);  // 10

// out - Birden fazla değer döndür
void GetMinMax(int[] array, out int min, out int max)
{
    min = array[0];
    max = array[0];
    foreach (int num in array)
    {
        if (num < min) min = num;
        if (num > max) max = num;
    }
}

int[] numbers = { 5, 2, 8, 1, 9 };
GetMinMax(numbers, out int minimum, out int maximum);
Debug.Log($"Min: {minimum}, Max: {maximum}");  // Min: 1, Max: 9
```

---

## String (Metin) İşlemleri

### String Birleştirme

```csharp
string firstName = "Ahmet";
string lastName = "Yılmaz";

// Yöntem 1: + operatörü
string fullName = firstName + " " + lastName;

// Yöntem 2: String interpolation (önerilen)
string fullName = $"{firstName} {lastName}";

// Yöntem 3: String.Format
string fullName = string.Format("{0} {1}", firstName, lastName);
```

### String Metodları

```csharp
string text = "  Merhaba Dünya  ";

text.Length;                    // 17 (karakter sayısı)
text.ToLower();                 // "  merhaba dünya  "
text.ToUpper();                 // "  MERHABA DÜNYA  "
text.Trim();                    // "Merhaba Dünya" (boşlukları sil)
text.Contains("Dünya");         // true
text.StartsWith("Mer");         // false (boşluk var)
text.Replace("Dünya", "Unity"); // "  Merhaba Unity  "
text.Split(' ');                // ["", "", "Merhaba", "Dünya", "", ""]
text.Substring(2, 7);           // "Merhaba"
```

---

## Pratik Uygulama: Basit Oyuncu Kontrolü

### PlayerController.cs

```csharp
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [Header("Hareket Ayarları")]
    [SerializeField] private float moveSpeed = 5f;
    [SerializeField] private float rotateSpeed = 100f;

    [Header("Sağlık Sistemi")]
    [SerializeField] private int maxHealth = 100;
    private int currentHealth;

    [Header("Referanslar")]
    private Rigidbody rb;

    void Awake()
    {
        rb = GetComponent<Rigidbody>();
    }

    void Start()
    {
        currentHealth = maxHealth;
        Debug.Log("Oyuncu hazır! Sağlık: " + currentHealth);
    }

    void Update()
    {
        // Input kontrolü Update'de
        HandleInput();
    }

    void FixedUpdate()
    {
        // Fizik hareketi FixedUpdate'de
        Move();
    }

    void HandleInput()
    {
        // Escape tuşu ile çıkış
        if (Input.GetKeyDown(KeyCode.Escape))
        {
            Debug.Log("Oyun duraklatıldı");
        }

        // Space tuşu ile zıplama
        if (Input.GetKeyDown(KeyCode.Space))
        {
            Jump();
        }
    }

    void Move()
    {
        float horizontal = Input.GetAxis("Horizontal"); // A/D veya Sol/Sağ ok
        float vertical = Input.GetAxis("Vertical");     // W/S veya Yukarı/Aşağı ok

        Vector3 movement = new Vector3(horizontal, 0, vertical);
        movement = movement.normalized * moveSpeed * Time.fixedDeltaTime;

        rb.MovePosition(transform.position + movement);
    }

    void Jump()
    {
        rb.AddForce(Vector3.up * 5f, ForceMode.Impulse);
        Debug.Log("Zıpladı!");
    }

    public void TakeDamage(int damage)
    {
        currentHealth -= damage;
        Debug.Log($"Hasar alındı! Kalan sağlık: {currentHealth}");

        if (currentHealth <= 0)
        {
            Die();
        }
    }

    void Die()
    {
        Debug.Log("Oyuncu öldü!");
        // Oyun sonu işlemleri
    }
}
```

---

## Pratik Uygulama: Coin Toplama

### Coin.cs

```csharp
using UnityEngine;

public class Coin : MonoBehaviour
{
    [SerializeField] private int coinValue = 10;
    [SerializeField] private float rotateSpeed = 50f;

    void Update()
    {
        // Coin'i döndür
        transform.Rotate(Vector3.up * rotateSpeed * Time.deltaTime);
    }

    void OnTriggerEnter(Collider other)
    {
        // Oyuncu ile temas ettiyse
        if (other.CompareTag("Player"))
        {
            // Skor ekle
            GameManager.Instance.AddScore(coinValue);

            // Coin'i yok et
            Destroy(gameObject);
        }
    }
}
```

### GameManager.cs (Basit Singleton)

```csharp
using UnityEngine;

public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    private int score = 0;

    void Awake()
    {
        // Singleton pattern
        if (Instance == null)
        {
            Instance = this;
        }
        else
        {
            Destroy(gameObject);
        }
    }

    public void AddScore(int amount)
    {
        score += amount;
        Debug.Log("Skor: " + score);
    }

    public int GetScore()
    {
        return score;
    }
}
```

---

## Debug ve Hata Ayıklama

### Debug Metodları

```csharp
// Temel log
Debug.Log("Normal mesaj");

// Uyarı
Debug.LogWarning("Dikkat! Bu bir uyarı.");

// Hata
Debug.LogError("HATA! Bir şeyler yanlış gitti.");

// Koşullu log
Debug.Assert(health > 0, "Sağlık sıfırın altında olamaz!");

// Renkli log
Debug.Log("<color=red>Kırmızı mesaj</color>");
Debug.Log("<color=green>Yeşil mesaj</color>");
Debug.Log("<color=yellow>Sarı mesaj</color>");
```

### Sahne İçi Debug Çizimleri

```csharp
void OnDrawGizmos()
{
    // Her zaman çizilir (Editor'da)
    Gizmos.color = Color.yellow;
    Gizmos.DrawWireSphere(transform.position, 2f);
}

void OnDrawGizmosSelected()
{
    // Sadece nesne seçiliyken çizilir
    Gizmos.color = Color.red;
    Gizmos.DrawLine(transform.position, transform.position + Vector3.forward * 5f);
}

void Update()
{
    // Oyun çalışırken çizgi çiz
    Debug.DrawRay(transform.position, transform.forward * 10f, Color.green);
}
```

---

## Özet ve Kontrol Listesi

Bu derste öğrendiklerimiz:
- [x] Script oluşturma ve temel yapı
- [x] Değişkenler ve veri tipleri
- [x] Erişim belirleyicileri (public, private, SerializeField)
- [x] MonoBehaviour yaşam döngüsü
- [x] Update vs FixedUpdate farkı
- [x] Koşul ifadeleri (if-else, switch)
- [x] Döngüler (for, while, foreach)
- [x] Diziler ve Listeler
- [x] Fonksiyonlar (metodlar)
- [x] String işlemleri
- [x] Debug ve hata ayıklama

---

## Alıştırmalar

1. Sağlık çubuğu mantığı yazın (0-100 arası, hasar alma, iyileşme)
2. Sayaç script'i yazın (her saniye 1 artan)
3. Rastgele düşman spawn eden sistem yapın
4. Envanter sistemi için List kullanan bir script yazın
5. Oyuncunun hızını Input ile değiştiren (Shift = koşma) sistem yapın

---

## Sonraki Ders

6. Ders'te Input Sistemi ve Karakter Kontrolü konularını işleyeceğiz. Yeni Input System, karakter hareketi, kamera kontrolü ve temel oynanış mekanikleri üzerinde çalışacağız.

