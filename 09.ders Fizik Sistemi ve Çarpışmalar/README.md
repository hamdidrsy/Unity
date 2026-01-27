# 9. Ders - Fizik Sistemi ve Çarpışmalar

## Giriş

Unity'nin fizik motoru, nesnelerin gerçekçi şekilde hareket etmesini, çarpışmasını ve etkileşmesini sağlar. Yerçekimi, sürtünme, zıplama, patlama kuvvetleri - bunların hepsi fizik sistemiyle yönetilir.

Bu derste:
- Rigidbody ve fizik temelleri
- Collider türleri ve kullanımları
- Collision vs Trigger farkları
- Physics Material (sürtünme, zıplama)
- Raycast ile ışın atma
- Layer ve LayerMask sistemi
- Joint türleri (Hinge, Spring, Fixed)
- Fizik optimizasyonu
- Pratik örnekler

konularını işleyeceğiz.

---

## Fizik Sistemi Temelleri

Unity iki farklı fizik motoru kullanır:

```
┌─────────────────────────────────────────────────────────────┐
│                    UNITY FİZİK MOTORLARI                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  3D Fizik (PhysX)          2D Fizik (Box2D)                │
│  ─────────────────         ────────────────                 │
│  Rigidbody                 Rigidbody2D                      │
│  BoxCollider               BoxCollider2D                    │
│  SphereCollider            CircleCollider2D                 │
│  CapsuleCollider           CapsuleCollider2D                │
│  MeshCollider              PolygonCollider2D                │
│  Physics.Raycast           Physics2D.Raycast                │
│                                                             │
│  NOT: 3D ve 2D fizik birbirleriyle etkileşmez!            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Fizik Ayarları

```
Edit > Project Settings > Physics
```

```
┌─────────────────────────────────────────────────────────────┐
│                   PHYSICS SETTINGS                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Gravity             → Yerçekimi (varsayılan: 0, -9.81, 0)  │
│ Default Material    → Varsayılan fizik materyali           │
│ Bounce Threshold    → Zıplama eşiği                        │
│ Default Max Depenetration Velocity → Çakışma çözüm hızı   │
│                                                             │
│ Sleep Threshold     → Uyku eşiği (performans için)         │
│ Default Contact Offset → Temas mesafesi                    │
│                                                             │
│ Layer Collision Matrix → Hangi layer'lar çarpışır         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Rigidbody

Rigidbody, bir GameObject'e fiziksel özellikler kazandırır. Fizik motorunun nesneyi kontrol etmesini sağlar.

### Rigidbody Ekleme

```
GameObject seç > Inspector > Add Component > Rigidbody
```

### Rigidbody Özellikleri

```
┌─────────────────────────────────────────────────────────────┐
│                   RIGIDBODY ÖZELLİKLERİ                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Mass              → Kütle (kg). Hareket ve çarpışma etkisi │
│ Drag              → Hava direnci (hareket yavaşlatma)      │
│ Angular Drag      → Dönme direnci                          │
│                                                             │
│ Use Gravity       → Yerçekimi etkisi                       │
│ Is Kinematic      → Fizik yerine script kontrolü           │
│                                                             │
│ Interpolate       → Görsel yumuşatma                       │
│   - None          → Yumuşatma yok                          │
│   - Interpolate   → Önceki frame'e göre                    │
│   - Extrapolate   → Sonraki frame'i tahmin et              │
│                                                             │
│ Collision Detection → Çarpışma algılama modu               │
│   - Discrete      → Normal (varsayılan)                    │
│   - Continuous    → Sürekli (hızlı objeler)                │
│   - Continuous Dynamic → En hassas                         │
│   - Continuous Speculative → Kinematic için                │
│                                                             │
│ Constraints       → Hareket/dönme kısıtlamaları           │
│   - Freeze Position X/Y/Z                                  │
│   - Freeze Rotation X/Y/Z                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Temel Rigidbody Kullanımı

```csharp
using UnityEngine;

public class RigidbodyBasics : MonoBehaviour
{
    private Rigidbody rb;

    void Start()
    {
        rb = GetComponent<Rigidbody>();

        // Temel ayarlar
        rb.mass = 1f;
        rb.drag = 0.5f;
        rb.angularDrag = 0.05f;
        rb.useGravity = true;
        rb.isKinematic = false;
    }

    void FixedUpdate()
    {
        // Fizik işlemleri FixedUpdate'te yapılmalı!
    }
}
```

### Kuvvet Uygulama

```csharp
using UnityEngine;

public class ForceExamples : MonoBehaviour
{
    private Rigidbody rb;

    [SerializeField] private float forcePower = 10f;
    [SerializeField] private float jumpForce = 5f;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }

    void FixedUpdate()
    {
        // Sürekli kuvvet (motor, itme)
        if (Input.GetKey(KeyCode.W))
        {
            rb.AddForce(transform.forward * forcePower);
        }
    }

    // ===== KUVVET TÜRLERİ =====

    public void ApplyForce()
    {
        // Varsayılan: Force (sürekli kuvvet, kütle etkili)
        rb.AddForce(Vector3.forward * 10f, ForceMode.Force);
    }

    public void ApplyAcceleration()
    {
        // Acceleration: Kütle etkisiz ivme
        rb.AddForce(Vector3.forward * 10f, ForceMode.Acceleration);
    }

    public void ApplyImpulse()
    {
        // Impulse: Anlık kuvvet (zıplama, patlama)
        rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
    }

    public void ApplyVelocityChange()
    {
        // VelocityChange: Anlık hız değişimi, kütle etkisiz
        rb.AddForce(Vector3.forward * 5f, ForceMode.VelocityChange);
    }

    // ===== DÖNME KUVVETİ =====

    public void ApplyTorque()
    {
        // Dönme kuvveti
        rb.AddTorque(Vector3.up * 10f);
    }

    // ===== PATLAMA KUVVETİ =====

    public void ApplyExplosion(Vector3 explosionPos)
    {
        // Patlama etkisi
        rb.AddExplosionForce(500f, explosionPos, 10f, 1f);
        // (kuvvet, merkez, yarıçap, yukarı kaldırma)
    }

    // ===== POZİSYONDA KUVVET =====

    public void ApplyForceAtPosition(Vector3 position)
    {
        // Belirli noktada kuvvet (gerçekçi dönme)
        rb.AddForceAtPosition(Vector3.forward * 10f, position);
    }
}
```

### ForceMode Karşılaştırması

```
┌─────────────────────────────────────────────────────────────┐
│                    FORCEMODE TÜRLERİ                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ForceMode.Force (varsayılan)                               │
│   → Sürekli kuvvet, kütle etkiler                          │
│   → Kullanım: Motor, rüzgar, itme                          │
│   → Birim: Newton (kg*m/s²)                                │
│                                                             │
│ ForceMode.Acceleration                                      │
│   → Sürekli ivme, kütle ETKİLEMEZ                          │
│   → Kullanım: Tüm nesnelere eşit etki                      │
│   → Birim: m/s²                                            │
│                                                             │
│ ForceMode.Impulse                                           │
│   → Anlık kuvvet, kütle etkiler                            │
│   → Kullanım: Zıplama, vuruş, patlama                      │
│   → Birim: Newton-saniye (kg*m/s)                          │
│                                                             │
│ ForceMode.VelocityChange                                    │
│   → Anlık hız değişimi, kütle ETKİLEMEZ                    │
│   → Kullanım: Teleport benzeri hareket                     │
│   → Birim: m/s                                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Hız Kontrolü

```csharp
using UnityEngine;

public class VelocityControl : MonoBehaviour
{
    private Rigidbody rb;

    [SerializeField] private float maxSpeed = 10f;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }

    void FixedUpdate()
    {
        // Mevcut hız
        Vector3 currentVelocity = rb.velocity;
        float speed = currentVelocity.magnitude;

        Debug.Log($"Hız: {speed:F2} m/s");

        // Maksimum hız sınırı
        if (speed > maxSpeed)
        {
            rb.velocity = currentVelocity.normalized * maxSpeed;
        }
    }

    // Direkt hız atama
    public void SetVelocity(Vector3 newVelocity)
    {
        rb.velocity = newVelocity;
    }

    // Anlık durdurma
    public void Stop()
    {
        rb.velocity = Vector3.zero;
        rb.angularVelocity = Vector3.zero;
    }

    // Yatay hızı koru, dikey sıfırla (zıplama için)
    public void Jump(float jumpForce)
    {
        Vector3 vel = rb.velocity;
        vel.y = 0f; // Mevcut dikey hızı sıfırla
        rb.velocity = vel;
        rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
    }
}
```

### Kinematic vs Dynamic

```csharp
using UnityEngine;

public class KinematicExample : MonoBehaviour
{
    private Rigidbody rb;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }

    // Kinematic: Script kontrolünde, fizikten etkilenmez
    // Ama diğer rigidbody'leri etkileyebilir
    public void SetKinematic(bool isKinematic)
    {
        rb.isKinematic = isKinematic;
    }

    // Kinematic objeyi hareket ettirme (doğru yol)
    public void MoveKinematic(Vector3 targetPosition)
    {
        // MovePosition fizik sistemini kullanır
        rb.MovePosition(targetPosition);
    }

    public void RotateKinematic(Quaternion targetRotation)
    {
        rb.MoveRotation(targetRotation);
    }

    // Platform örneği
    void FixedUpdate()
    {
        if (rb.isKinematic)
        {
            // Kinematic platform hareketi
            float x = Mathf.Sin(Time.time) * 3f;
            rb.MovePosition(new Vector3(x, transform.position.y, transform.position.z));
        }
    }
}
```

---

## Collider (Çarpıştırıcı)

Collider, nesnenin fiziksel şeklini tanımlar. Rigidbody olmadan da çalışır (statik objeler için).

### Collider Türleri

```
┌─────────────────────────────────────────────────────────────┐
│                    COLLIDER TÜRLERİ                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ BoxCollider         → Kutu şekli                           │
│   [====]              Duvarlar, kutular, kapılar           │
│                                                             │
│ SphereCollider      → Küre şekli                           │
│   (  O  )             Toplar, mermiler, trigger alanları   │
│                                                             │
│ CapsuleCollider     → Kapsül şekli                         │
│   (===)               Karakterler, insanlar                 │
│                                                             │
│ MeshCollider        → Mesh şekli                           │
│   /\/\/\              Karmaşık objeler (performans dikkat)  │
│                                                             │
│ TerrainCollider     → Arazi için                           │
│   ~~~                 Terrain objelerine otomatik eklenir   │
│                                                             │
│ WheelCollider       → Araç tekerlekleri                    │
│   (O)                 Süspansiyon, sürtünme içerir          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Collider Ekleme ve Ayarlama

```csharp
using UnityEngine;

public class ColliderSetup : MonoBehaviour
{
    void Start()
    {
        // Box Collider
        BoxCollider box = gameObject.AddComponent<BoxCollider>();
        box.center = Vector3.zero;
        box.size = new Vector3(1f, 2f, 1f);

        // Sphere Collider
        SphereCollider sphere = gameObject.AddComponent<SphereCollider>();
        sphere.center = Vector3.zero;
        sphere.radius = 0.5f;

        // Capsule Collider
        CapsuleCollider capsule = gameObject.AddComponent<CapsuleCollider>();
        capsule.center = new Vector3(0, 1f, 0);
        capsule.radius = 0.5f;
        capsule.height = 2f;
        capsule.direction = 1; // 0=X, 1=Y, 2=Z
    }
}
```

### Mesh Collider

```csharp
using UnityEngine;

public class MeshColliderSetup : MonoBehaviour
{
    [SerializeField] private Mesh customMesh;

    void Start()
    {
        MeshCollider meshCol = gameObject.AddComponent<MeshCollider>();

        // Mesh atama
        meshCol.sharedMesh = customMesh;
        // veya
        meshCol.sharedMesh = GetComponent<MeshFilter>().sharedMesh;

        // Convex: Rigidbody ile kullanım için gerekli
        // Sadece 255 üçgene kadar destekler
        meshCol.convex = true;

        // Convex olmayan: Sadece statik objeler için
        // meshCol.convex = false;
    }
}
```

### Compound Collider (Bileşik)

Karmaşık şekiller için birden fazla collider kullanılır:

```csharp
// Hierarchy yapısı:
// Player (Rigidbody)
//   ├── Body (CapsuleCollider)
//   ├── Head (SphereCollider)
//   └── Weapon (BoxCollider)

// Tüm child collider'lar parent'ın Rigidbody'sini kullanır
```

---

## Collision vs Trigger

### Collision (Çarpışma)

Fiziksel çarpışma - nesneler birbirini iter.

```csharp
using UnityEngine;

public class CollisionExample : MonoBehaviour
{
    // Çarpışma BAŞLADIĞINDA (ilk temas)
    void OnCollisionEnter(Collision collision)
    {
        Debug.Log($"Çarpıştı: {collision.gameObject.name}");

        // Çarpışma bilgileri
        ContactPoint contact = collision.contacts[0];
        Vector3 point = contact.point;       // Temas noktası
        Vector3 normal = contact.normal;     // Temas normali

        // Çarpışma kuvveti
        float impactForce = collision.relativeVelocity.magnitude;
        Debug.Log($"Çarpışma kuvveti: {impactForce}");

        // Tag kontrolü
        if (collision.gameObject.CompareTag("Enemy"))
        {
            TakeDamage(10);
        }

        // Layer kontrolü
        if (collision.gameObject.layer == LayerMask.NameToLayer("Ground"))
        {
            isGrounded = true;
        }
    }

    // Çarpışma DEVAM EDİYOR (her physics frame)
    void OnCollisionStay(Collision collision)
    {
        // Sürekli temas halinde
    }

    // Çarpışma BİTTİĞİNDE (ayrılma)
    void OnCollisionExit(Collision collision)
    {
        Debug.Log($"Ayrıldı: {collision.gameObject.name}");

        if (collision.gameObject.layer == LayerMask.NameToLayer("Ground"))
        {
            isGrounded = false;
        }
    }

    private bool isGrounded;
    void TakeDamage(int amount) { }
}
```

### Trigger (Tetikleyici)

Fiziksel olmayan algılama - nesneler birbirinden geçer.

```csharp
using UnityEngine;

public class TriggerExample : MonoBehaviour
{
    // Trigger'a GİRDİĞİNDE
    void OnTriggerEnter(Collider other)
    {
        Debug.Log($"Trigger'a girdi: {other.gameObject.name}");

        if (other.CompareTag("Player"))
        {
            // Coin toplama
            CollectCoin();
        }

        if (other.CompareTag("Enemy"))
        {
            // Düşman algılama alanı
            DetectEnemy(other.gameObject);
        }
    }

    // Trigger İÇİNDE (her physics frame)
    void OnTriggerStay(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            // Hasar alanı
            DamageOverTime(other.gameObject);
        }
    }

    // Trigger'dan ÇIKTIĞINDA
    void OnTriggerExit(Collider other)
    {
        Debug.Log($"Trigger'dan çıktı: {other.gameObject.name}");
    }

    void CollectCoin() { }
    void DetectEnemy(GameObject enemy) { }
    void DamageOverTime(GameObject target) { }
}
```

### Collision vs Trigger Karşılaştırması

```
┌─────────────────────────────────────────────────────────────┐
│              COLLISION vs TRIGGER                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                 COLLISION              TRIGGER              │
│ ─────────────────────────────────────────────────────────  │
│ Is Trigger       ✗ (kapalı)           ✓ (açık)            │
│ Fizik etkisi     Var (iter)           Yok (geçer)          │
│ Callback         OnCollision___       OnTrigger___         │
│ Parametre        Collision            Collider             │
│                                                             │
│ Kullanım:                                                   │
│ - Collision: Duvarlar, zemin, fiziksel objeler            │
│ - Trigger: Coin toplama, kapı algılama, hasar alanı       │
│                                                             │
│ Rigidbody Gereksinimi:                                      │
│ - En az bir objede Rigidbody olmalı                        │
│ - İkisi de static ise callback çağrılmaz                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Çarpışma Matrisi

```
┌─────────────────────────────────────────────────────────────┐
│                  ÇARPIŞMA MATRİSİ                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    Static    Rigidbody   Kinematic         │
│                    Collider  Collider    Rigidbody         │
│ ─────────────────────────────────────────────────────────  │
│ Static Collider      ✗          ✓           ✓              │
│ Rigidbody Collider   ✓          ✓           ✓              │
│ Kinematic Rigidbody  ✓          ✓           ✗              │
│                                                             │
│ ✓ = Çarpışma/Trigger algılanır                             │
│ ✗ = Algılanmaz                                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Physics Material

Yüzey özelliklerini (sürtünme, zıplama) tanımlar.

### Physics Material Oluşturma

```
Project > Sağ Tık > Create > Physic Material
```

### Physics Material Özellikleri

```
┌─────────────────────────────────────────────────────────────┐
│                 PHYSIC MATERIAL                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Dynamic Friction  → Hareket halinde sürtünme (0-1)         │
│                    0 = buz, 1 = kauçuk                     │
│                                                             │
│ Static Friction   → Durağan sürtünme (0-1)                 │
│                    Harekete başlama direnci                │
│                                                             │
│ Bounciness        → Zıplama (0-1)                          │
│                    0 = zıplamaz, 1 = tam zıplama           │
│                                                             │
│ Friction Combine  → Sürtünme hesaplama modu                │
│   - Average       → Ortalama                               │
│   - Minimum       → Minimum değer                          │
│   - Maximum       → Maksimum değer                         │
│   - Multiply      → Çarpım                                 │
│                                                             │
│ Bounce Combine    → Zıplama hesaplama modu                 │
│                    (aynı seçenekler)                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Örnek Materyaller

```csharp
// Buz (kaygan)
// Dynamic Friction: 0.05
// Static Friction: 0.05
// Bounciness: 0

// Kauçuk (yapışkan, zıplayan)
// Dynamic Friction: 0.8
// Static Friction: 0.9
// Bounciness: 0.8

// Metal
// Dynamic Friction: 0.4
// Static Friction: 0.4
// Bounciness: 0.1

// Bouncy Ball (süper zıplayan)
// Dynamic Friction: 0.5
// Static Friction: 0.5
// Bounciness: 1.0
// Bounce Combine: Maximum
```

### Script ile Physics Material

```csharp
using UnityEngine;

public class PhysicsMaterialExample : MonoBehaviour
{
    [SerializeField] private PhysicMaterial iceMaterial;
    [SerializeField] private PhysicMaterial normalMaterial;

    private Collider col;

    void Start()
    {
        col = GetComponent<Collider>();
    }

    public void SetIcy(bool isIcy)
    {
        col.material = isIcy ? iceMaterial : normalMaterial;
    }

    // Dinamik materyal oluşturma
    public void CreateBouncyMaterial()
    {
        PhysicMaterial bouncy = new PhysicMaterial("Bouncy");
        bouncy.bounciness = 0.9f;
        bouncy.dynamicFriction = 0.3f;
        bouncy.staticFriction = 0.3f;
        bouncy.bounceCombine = PhysicMaterialCombine.Maximum;

        col.material = bouncy;
    }
}
```

---

## Raycast

Görünmez bir ışın atarak çarpışma kontrolü yapar.

### Temel Raycast

```csharp
using UnityEngine;

public class RaycastBasics : MonoBehaviour
{
    [SerializeField] private float rayDistance = 100f;

    void Update()
    {
        // En basit raycast
        if (Physics.Raycast(transform.position, transform.forward, rayDistance))
        {
            Debug.Log("Bir şeye çarptı!");
        }

        // Çarpılan obje bilgisi ile
        RaycastHit hit;
        if (Physics.Raycast(transform.position, transform.forward, out hit, rayDistance))
        {
            Debug.Log($"Çarpılan: {hit.collider.gameObject.name}");
            Debug.Log($"Mesafe: {hit.distance}");
            Debug.Log($"Nokta: {hit.point}");
            Debug.Log($"Normal: {hit.normal}");
        }
    }

    // Debug çizimi
    void OnDrawGizmos()
    {
        Gizmos.color = Color.red;
        Gizmos.DrawRay(transform.position, transform.forward * rayDistance);
    }
}
```

### RaycastHit Bilgileri

```
┌─────────────────────────────────────────────────────────────┐
│                    RAYCASTHIT                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ hit.collider       → Çarpılan collider                     │
│ hit.transform      → Çarpılan transform                    │
│ hit.rigidbody      → Çarpılan rigidbody (varsa)           │
│ hit.point          → Çarpışma noktası (world space)       │
│ hit.normal         → Çarpışma yüzey normali               │
│ hit.distance       → Işın başlangıcından mesafe           │
│ hit.textureCoord   → UV koordinatı (MeshCollider)         │
│ hit.triangleIndex  → Üçgen indeksi (MeshCollider)         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Layer Mask ile Raycast

```csharp
using UnityEngine;

public class RaycastWithLayers : MonoBehaviour
{
    [SerializeField] private LayerMask targetLayers;
    [SerializeField] private float rayDistance = 100f;

    void Update()
    {
        RaycastHit hit;

        // Sadece belirli layer'lara çarp
        if (Physics.Raycast(transform.position, transform.forward, out hit, rayDistance, targetLayers))
        {
            Debug.Log($"Hedef: {hit.collider.name}");
        }

        // Kod ile LayerMask
        int groundLayer = LayerMask.GetMask("Ground");
        int enemyLayer = LayerMask.GetMask("Enemy");
        int combinedMask = groundLayer | enemyLayer; // İkisini de dahil et

        // Belirli layer'ı hariç tut
        int everythingExceptPlayer = ~LayerMask.GetMask("Player");
    }
}
```

### Farklı Raycast Türleri

```csharp
using UnityEngine;

public class RaycastTypes : MonoBehaviour
{
    void Update()
    {
        Vector3 origin = transform.position;
        Vector3 direction = transform.forward;
        float distance = 100f;
        RaycastHit hit;

        // 1. Normal Raycast (çizgi)
        Physics.Raycast(origin, direction, out hit, distance);

        // 2. Ray struct ile
        Ray ray = new Ray(origin, direction);
        Physics.Raycast(ray, out hit, distance);

        // 3. Kameradan mouse pozisyonuna
        Ray mouseRay = Camera.main.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(mouseRay, out hit, distance))
        {
            Debug.Log($"Mouse hedefi: {hit.collider.name}");
        }

        // 4. SphereCast (kalın ışın)
        float radius = 0.5f;
        if (Physics.SphereCast(origin, radius, direction, out hit, distance))
        {
            // Küre şeklinde tarama
        }

        // 5. BoxCast (kutu ışın)
        Vector3 halfExtents = new Vector3(0.5f, 0.5f, 0.5f);
        if (Physics.BoxCast(origin, halfExtents, direction, out hit))
        {
            // Kutu şeklinde tarama
        }

        // 6. CapsuleCast (kapsül ışın)
        Vector3 point1 = origin + Vector3.up * 0.5f;
        Vector3 point2 = origin - Vector3.up * 0.5f;
        float capsuleRadius = 0.3f;
        if (Physics.CapsuleCast(point1, point2, capsuleRadius, direction, out hit))
        {
            // Karakter hareketi için ideal
        }
    }
}
```

### RaycastAll (Çoklu Sonuç)

```csharp
using UnityEngine;

public class RaycastAllExample : MonoBehaviour
{
    void Update()
    {
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);

        // Tüm çarpışmaları al
        RaycastHit[] hits = Physics.RaycastAll(ray, 100f);

        foreach (RaycastHit hit in hits)
        {
            Debug.Log($"Çarpılan: {hit.collider.name} - Mesafe: {hit.distance}");
        }

        // Non-alloc versiyon (GC yok)
        RaycastHit[] results = new RaycastHit[10];
        int hitCount = Physics.RaycastNonAlloc(ray, results, 100f);

        for (int i = 0; i < hitCount; i++)
        {
            Debug.Log(results[i].collider.name);
        }
    }
}
```

### Pratik: Silah Sistemi

```csharp
using UnityEngine;

public class WeaponRaycast : MonoBehaviour
{
    [SerializeField] private Transform firePoint;
    [SerializeField] private float damage = 25f;
    [SerializeField] private float range = 100f;
    [SerializeField] private LayerMask hitLayers;
    [SerializeField] private ParticleSystem muzzleFlash;
    [SerializeField] private GameObject impactEffect;

    public void Fire()
    {
        muzzleFlash.Play();

        RaycastHit hit;
        if (Physics.Raycast(firePoint.position, firePoint.forward, out hit, range, hitLayers))
        {
            Debug.Log($"Vurulan: {hit.collider.name}");

            // Hasar ver
            IDamageable target = hit.collider.GetComponent<IDamageable>();
            if (target != null)
            {
                target.TakeDamage(damage);
            }

            // Etki efekti
            if (impactEffect != null)
            {
                Instantiate(impactEffect, hit.point, Quaternion.LookRotation(hit.normal));
            }

            // Rigidbody varsa it
            Rigidbody rb = hit.collider.GetComponent<Rigidbody>();
            if (rb != null)
            {
                rb.AddForceAtPosition(firePoint.forward * 100f, hit.point);
            }
        }
    }
}

public interface IDamageable
{
    void TakeDamage(float amount);
}
```

### Pratik: Zemin Kontrolü

```csharp
using UnityEngine;

public class GroundCheck : MonoBehaviour
{
    [SerializeField] private float groundCheckDistance = 0.2f;
    [SerializeField] private LayerMask groundLayers;
    [SerializeField] private Transform groundCheckPoint;

    public bool IsGrounded { get; private set; }
    public Vector3 GroundNormal { get; private set; }

    void FixedUpdate()
    {
        CheckGround();
    }

    void CheckGround()
    {
        RaycastHit hit;
        Vector3 origin = groundCheckPoint != null ? groundCheckPoint.position : transform.position;

        // SphereCast daha güvenilir
        if (Physics.SphereCast(origin, 0.1f, Vector3.down, out hit, groundCheckDistance, groundLayers))
        {
            IsGrounded = true;
            GroundNormal = hit.normal;
        }
        else
        {
            IsGrounded = false;
            GroundNormal = Vector3.up;
        }
    }

    void OnDrawGizmosSelected()
    {
        Vector3 origin = groundCheckPoint != null ? groundCheckPoint.position : transform.position;

        Gizmos.color = IsGrounded ? Color.green : Color.red;
        Gizmos.DrawWireSphere(origin + Vector3.down * groundCheckDistance, 0.1f);
    }
}
```

---

## OverlapSphere / OverlapBox

Belirli bir alanda collider araması yapar.

```csharp
using UnityEngine;

public class OverlapExamples : MonoBehaviour
{
    [SerializeField] private float explosionRadius = 5f;
    [SerializeField] private float explosionForce = 500f;
    [SerializeField] private LayerMask affectedLayers;

    public void Explode()
    {
        // Yarıçap içindeki tüm collider'ları bul
        Collider[] colliders = Physics.OverlapSphere(transform.position, explosionRadius, affectedLayers);

        foreach (Collider col in colliders)
        {
            Rigidbody rb = col.GetComponent<Rigidbody>();
            if (rb != null)
            {
                rb.AddExplosionForce(explosionForce, transform.position, explosionRadius);
            }

            // Hasar ver
            IDamageable target = col.GetComponent<IDamageable>();
            if (target != null)
            {
                float distance = Vector3.Distance(transform.position, col.transform.position);
                float damagePercent = 1f - (distance / explosionRadius);
                target.TakeDamage(100f * damagePercent);
            }
        }
    }

    // Kutu şeklinde arama
    public Collider[] FindInBox(Vector3 center, Vector3 halfExtents)
    {
        return Physics.OverlapBox(center, halfExtents, Quaternion.identity, affectedLayers);
    }

    // Kapsül şeklinde arama
    public Collider[] FindInCapsule(Vector3 point1, Vector3 point2, float radius)
    {
        return Physics.OverlapCapsule(point1, point2, radius, affectedLayers);
    }

    // Non-alloc versiyonları (GC yok)
    private Collider[] results = new Collider[20];

    public int FindNearbyNonAlloc()
    {
        return Physics.OverlapSphereNonAlloc(transform.position, explosionRadius, results, affectedLayers);
    }

    void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.red;
        Gizmos.DrawWireSphere(transform.position, explosionRadius);
    }
}
```

---

## Layer ve LayerMask

### Layer Oluşturma

```
Edit > Project Settings > Tags and Layers
```

```
┌─────────────────────────────────────────────────────────────┐
│                    LAYERS (0-31)                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 0  - Default           (sistem)                            │
│ 1  - TransparentFX     (sistem)                            │
│ 2  - Ignore Raycast    (sistem)                            │
│ 3  - (boş)                                                 │
│ 4  - Water             (sistem)                            │
│ 5  - UI                (sistem)                            │
│ 6  - (boş)                                                 │
│ 7  - (boş)                                                 │
│ 8  - Ground            (özel)                              │
│ 9  - Player            (özel)                              │
│ 10 - Enemy             (özel)                              │
│ 11 - Projectile        (özel)                              │
│ ...                                                        │
│ 31 - (son)                                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Layer Collision Matrix

```
Edit > Project Settings > Physics > Layer Collision Matrix
```

Hangi layer'ların birbiriyle çarpışacağını belirler.

### Script ile Layer Kullanımı

```csharp
using UnityEngine;

public class LayerExample : MonoBehaviour
{
    void Start()
    {
        // Layer atama
        gameObject.layer = LayerMask.NameToLayer("Enemy");

        // Layer kontrolü
        if (gameObject.layer == LayerMask.NameToLayer("Player"))
        {
            Debug.Log("Bu bir player!");
        }
    }
}
```

### LayerMask Kullanımı

```csharp
using UnityEngine;

public class LayerMaskExample : MonoBehaviour
{
    [SerializeField] private LayerMask groundLayers;    // Inspector'dan ayarla
    [SerializeField] private LayerMask enemyLayers;

    void Example()
    {
        // Kod ile LayerMask oluşturma
        int groundMask = LayerMask.GetMask("Ground");
        int enemyMask = LayerMask.GetMask("Enemy");
        int playerMask = LayerMask.GetMask("Player");

        // Birden fazla layer
        int combinedMask = LayerMask.GetMask("Ground", "Enemy", "Wall");

        // Bit işlemleri ile
        int mask = (1 << 8) | (1 << 9); // Layer 8 ve 9

        // Hariç tutma (everything except)
        int everythingExceptPlayer = ~(1 << LayerMask.NameToLayer("Player"));

        // Raycast'te kullanım
        RaycastHit hit;
        if (Physics.Raycast(transform.position, transform.forward, out hit, 100f, groundMask))
        {
            // Sadece ground layer'a çarpar
        }
    }

    // Layer kontrolü
    bool IsInLayerMask(GameObject obj, LayerMask mask)
    {
        return (mask.value & (1 << obj.layer)) != 0;
    }
}
```

---

## Joint (Eklem)

İki rigidbody'yi birbirine bağlar.

### Joint Türleri

```
┌─────────────────────────────────────────────────────────────┐
│                    JOINT TÜRLERİ                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Fixed Joint        → Sabit bağlantı (kaynak)               │
│   ══════             İki objeyi birleştirir                │
│                                                             │
│ Hinge Joint        → Menteşe (kapı, tekerlek)              │
│   ─○─                Tek eksen etrafında dönüş             │
│                                                             │
│ Spring Joint       → Yay bağlantısı                        │
│   ~/~/~             Esnek bağlantı                         │
│                                                             │
│ Character Joint    → Karakter eklemi (ragdoll)             │
│   ─●─                Sınırlı dönüş açıları                 │
│                                                             │
│ Configurable Joint → Tam özelleştirilebilir                │
│   ─?─                Her yönde ayrı ayar                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Fixed Joint

```csharp
using UnityEngine;

public class FixedJointExample : MonoBehaviour
{
    [SerializeField] private Rigidbody objectToAttach;

    void Start()
    {
        // Fixed Joint oluştur
        FixedJoint joint = gameObject.AddComponent<FixedJoint>();
        joint.connectedBody = objectToAttach;

        // Kırılma gücü (opsiyonel)
        joint.breakForce = 1000f;
        joint.breakTorque = 1000f;
    }

    // Joint kırıldığında
    void OnJointBreak(float breakForce)
    {
        Debug.Log($"Joint kırıldı! Kuvvet: {breakForce}");
    }
}
```

### Hinge Joint (Kapı)

```csharp
using UnityEngine;

public class DoorHinge : MonoBehaviour
{
    void Start()
    {
        HingeJoint hinge = gameObject.AddComponent<HingeJoint>();

        // Menteşe ekseni
        hinge.axis = Vector3.up; // Y ekseni etrafında dön

        // Anchor noktası (menteşe konumu)
        hinge.anchor = new Vector3(-0.5f, 0, 0); // Sol kenar

        // Sınırlar
        hinge.useLimits = true;
        JointLimits limits = hinge.limits;
        limits.min = 0f;
        limits.max = 90f;
        hinge.limits = limits;

        // Yay (kapanma kuvveti)
        hinge.useSpring = true;
        JointSpring spring = hinge.spring;
        spring.spring = 50f;
        spring.damper = 5f;
        spring.targetPosition = 0f; // Kapalı pozisyon
        hinge.spring = spring;
    }
}
```

### Spring Joint

```csharp
using UnityEngine;

public class SpringJointExample : MonoBehaviour
{
    [SerializeField] private Rigidbody connectedBody;

    void Start()
    {
        SpringJoint spring = gameObject.AddComponent<SpringJoint>();
        spring.connectedBody = connectedBody;

        // Yay ayarları
        spring.spring = 100f;       // Yay gücü
        spring.damper = 5f;         // Sönümleme
        spring.minDistance = 0f;    // Minimum mesafe
        spring.maxDistance = 2f;    // Maksimum mesafe

        // Anchor noktaları
        spring.anchor = Vector3.zero;
        spring.connectedAnchor = Vector3.zero;
    }
}
```

### Rope/Chain (Zincir) Örneği

```csharp
using UnityEngine;

public class RopeGenerator : MonoBehaviour
{
    [SerializeField] private GameObject linkPrefab;
    [SerializeField] private int linkCount = 10;
    [SerializeField] private float linkDistance = 0.5f;

    void Start()
    {
        CreateRope();
    }

    void CreateRope()
    {
        Rigidbody previousLink = GetComponent<Rigidbody>();

        for (int i = 0; i < linkCount; i++)
        {
            Vector3 position = transform.position - Vector3.up * linkDistance * (i + 1);
            GameObject link = Instantiate(linkPrefab, position, Quaternion.identity);

            // Configurable Joint ile bağla
            ConfigurableJoint joint = link.AddComponent<ConfigurableJoint>();
            joint.connectedBody = previousLink;

            // Hareket kısıtlamaları
            joint.xMotion = ConfigurableJointMotion.Locked;
            joint.yMotion = ConfigurableJointMotion.Locked;
            joint.zMotion = ConfigurableJointMotion.Locked;

            // Dönme serbestliği
            joint.angularXMotion = ConfigurableJointMotion.Free;
            joint.angularYMotion = ConfigurableJointMotion.Free;
            joint.angularZMotion = ConfigurableJointMotion.Free;

            previousLink = link.GetComponent<Rigidbody>();
        }
    }
}
```

---

## Pratik Örnekler

### 1. Basit Karakter Kontrolcüsü

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
public class PhysicsCharacterController : MonoBehaviour
{
    [Header("Movement")]
    [SerializeField] private float moveSpeed = 5f;
    [SerializeField] private float jumpForce = 7f;

    [Header("Ground Check")]
    [SerializeField] private Transform groundCheck;
    [SerializeField] private float groundCheckRadius = 0.2f;
    [SerializeField] private LayerMask groundLayers;

    private Rigidbody rb;
    private bool isGrounded;
    private Vector3 moveInput;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
        rb.freezeRotation = true; // Devrilmeyi önle
    }

    void Update()
    {
        // Input al
        float horizontal = Input.GetAxisRaw("Horizontal");
        float vertical = Input.GetAxisRaw("Vertical");
        moveInput = new Vector3(horizontal, 0, vertical).normalized;

        // Zemin kontrolü
        isGrounded = Physics.CheckSphere(groundCheck.position, groundCheckRadius, groundLayers);

        // Zıplama
        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
        }
    }

    void FixedUpdate()
    {
        // Hareket
        Vector3 move = transform.TransformDirection(moveInput) * moveSpeed;
        rb.velocity = new Vector3(move.x, rb.velocity.y, move.z);
    }

    void OnDrawGizmosSelected()
    {
        if (groundCheck == null) return;

        Gizmos.color = isGrounded ? Color.green : Color.red;
        Gizmos.DrawWireSphere(groundCheck.position, groundCheckRadius);
    }
}
```

### 2. Obje Toplama (Pickup)

```csharp
using UnityEngine;

public class Pickup : MonoBehaviour
{
    public enum PickupType { Health, Coin, Ammo, PowerUp }

    [SerializeField] private PickupType type;
    [SerializeField] private int value = 1;
    [SerializeField] private AudioClip pickupSound;
    [SerializeField] private GameObject pickupEffect;

    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            Collect(other.gameObject);
        }
    }

    void Collect(GameObject player)
    {
        // Türe göre işlem
        switch (type)
        {
            case PickupType.Health:
                player.GetComponent<PlayerHealth>()?.Heal(value);
                break;
            case PickupType.Coin:
                GameManager.Instance?.AddScore(value);
                break;
            case PickupType.Ammo:
                player.GetComponent<PlayerWeapon>()?.AddAmmo(value);
                break;
            case PickupType.PowerUp:
                player.GetComponent<PlayerPowerUp>()?.Activate(value);
                break;
        }

        // Efektler
        if (pickupSound != null)
        {
            AudioSource.PlayClipAtPoint(pickupSound, transform.position);
        }

        if (pickupEffect != null)
        {
            Instantiate(pickupEffect, transform.position, Quaternion.identity);
        }

        Destroy(gameObject);
    }
}
```

### 3. Hasar Alanı (Damage Zone)

```csharp
using UnityEngine;
using System.Collections.Generic;

public class DamageZone : MonoBehaviour
{
    [SerializeField] private float damagePerSecond = 10f;
    [SerializeField] private float damageInterval = 0.5f;

    private List<IDamageable> targetsInZone = new List<IDamageable>();
    private float damageTimer;

    void Update()
    {
        damageTimer += Time.deltaTime;

        if (damageTimer >= damageInterval)
        {
            DealDamage();
            damageTimer = 0f;
        }
    }

    void DealDamage()
    {
        // Null olanları temizle
        targetsInZone.RemoveAll(t => t == null);

        foreach (IDamageable target in targetsInZone)
        {
            target.TakeDamage(damagePerSecond * damageInterval);
        }
    }

    void OnTriggerEnter(Collider other)
    {
        IDamageable target = other.GetComponent<IDamageable>();
        if (target != null && !targetsInZone.Contains(target))
        {
            targetsInZone.Add(target);
        }
    }

    void OnTriggerExit(Collider other)
    {
        IDamageable target = other.GetComponent<IDamageable>();
        if (target != null)
        {
            targetsInZone.Remove(target);
        }
    }
}
```

### 4. İtme Mekaniği (Push)

```csharp
using UnityEngine;

public class Pushable : MonoBehaviour
{
    private Rigidbody rb;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }

    void OnControllerColliderHit(ControllerColliderHit hit)
    {
        // CharacterController ile çarpışma
    }
}

// Player tarafında
public class PlayerPush : MonoBehaviour
{
    [SerializeField] private float pushForce = 5f;

    void OnControllerColliderHit(ControllerColliderHit hit)
    {
        Rigidbody rb = hit.collider.attachedRigidbody;

        if (rb == null || rb.isKinematic) return;

        // Yukarıdan itmeyi engelle
        if (hit.moveDirection.y < -0.3f) return;

        // İtme kuvveti uygula
        Vector3 pushDir = new Vector3(hit.moveDirection.x, 0, hit.moveDirection.z);
        rb.velocity = pushDir * pushForce;
    }
}
```

### 5. Platform Sistemi

```csharp
using UnityEngine;

public class MovingPlatform : MonoBehaviour
{
    [SerializeField] private Vector3[] waypoints;
    [SerializeField] private float speed = 2f;
    [SerializeField] private float waitTime = 1f;

    private Rigidbody rb;
    private int currentWaypoint = 0;
    private float waitTimer;
    private bool waiting;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
        rb.isKinematic = true;

        if (waypoints.Length > 0)
        {
            transform.position = waypoints[0];
        }
    }

    void FixedUpdate()
    {
        if (waypoints.Length < 2) return;

        if (waiting)
        {
            waitTimer -= Time.fixedDeltaTime;
            if (waitTimer <= 0)
            {
                waiting = false;
            }
            return;
        }

        Vector3 target = waypoints[currentWaypoint];
        Vector3 direction = (target - transform.position).normalized;
        float distance = Vector3.Distance(transform.position, target);

        if (distance > 0.05f)
        {
            rb.MovePosition(transform.position + direction * speed * Time.fixedDeltaTime);
        }
        else
        {
            currentWaypoint = (currentWaypoint + 1) % waypoints.Length;
            waiting = true;
            waitTimer = waitTime;
        }
    }

    void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.CompareTag("Player"))
        {
            collision.transform.SetParent(transform);
        }
    }

    void OnCollisionExit(Collision collision)
    {
        if (collision.gameObject.CompareTag("Player"))
        {
            collision.transform.SetParent(null);
        }
    }

    void OnDrawGizmosSelected()
    {
        if (waypoints == null || waypoints.Length == 0) return;

        Gizmos.color = Color.yellow;
        for (int i = 0; i < waypoints.Length; i++)
        {
            Gizmos.DrawSphere(waypoints[i], 0.3f);
            if (i < waypoints.Length - 1)
            {
                Gizmos.DrawLine(waypoints[i], waypoints[i + 1]);
            }
            else
            {
                Gizmos.DrawLine(waypoints[i], waypoints[0]);
            }
        }
    }
}
```

### 6. Patlama Efekti

```csharp
using UnityEngine;

public class Explosion : MonoBehaviour
{
    [SerializeField] private float explosionRadius = 5f;
    [SerializeField] private float explosionForce = 700f;
    [SerializeField] private float upwardsModifier = 1f;
    [SerializeField] private float damage = 100f;
    [SerializeField] private LayerMask affectedLayers;
    [SerializeField] private ParticleSystem explosionEffect;
    [SerializeField] private AudioClip explosionSound;

    public void Explode()
    {
        // Efektler
        if (explosionEffect != null)
        {
            Instantiate(explosionEffect, transform.position, Quaternion.identity);
        }

        if (explosionSound != null)
        {
            AudioSource.PlayClipAtPoint(explosionSound, transform.position);
        }

        // Etki alanındaki objeleri bul
        Collider[] colliders = Physics.OverlapSphere(transform.position, explosionRadius, affectedLayers);

        foreach (Collider col in colliders)
        {
            // Kuvvet uygula
            Rigidbody rb = col.GetComponent<Rigidbody>();
            if (rb != null)
            {
                rb.AddExplosionForce(explosionForce, transform.position, explosionRadius, upwardsModifier);
            }

            // Hasar ver (mesafeye göre azalan)
            IDamageable target = col.GetComponent<IDamageable>();
            if (target != null)
            {
                float distance = Vector3.Distance(transform.position, col.transform.position);
                float damagePercent = 1f - (distance / explosionRadius);
                target.TakeDamage(damage * Mathf.Max(0, damagePercent));
            }
        }

        Destroy(gameObject);
    }

    void OnDrawGizmosSelected()
    {
        Gizmos.color = new Color(1, 0, 0, 0.3f);
        Gizmos.DrawSphere(transform.position, explosionRadius);
    }
}
```

---

## Performans İpuçları

### 1. Collider Seçimi

```
┌─────────────────────────────────────────────────────────────┐
│              COLLIDER PERFORMANSI                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ En Hızlı → En Yavaş:                                       │
│                                                             │
│ 1. Sphere Collider    (en hızlı)                           │
│ 2. Capsule Collider                                        │
│ 3. Box Collider                                            │
│ 4. Mesh Collider (Convex)                                  │
│ 5. Mesh Collider (Non-Convex)  (en yavaş)                 │
│                                                             │
│ İpuçları:                                                   │
│ - Basit şekiller tercih edin                               │
│ - Compound collider kullanın                               │
│ - MeshCollider sadece gerektiğinde                         │
│ - Convex MeshCollider daha hızlı                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2. Layer Collision Matrix

```csharp
// Gereksiz çarpışmaları devre dışı bırakın
// Edit > Project Settings > Physics > Layer Collision Matrix

// Örnek:
// - Düşman mermileri düşmanlarla çarpışmasın
// - Oyuncu mermileri oyuncuyla çarpışmasın
// - UI layer hiçbir şeyle çarpışmasın
```

### 3. Rigidbody Optimizasyonu

```csharp
// Sleep: Hareket etmeyen rigidbody'ler uyur
rb.sleepThreshold = 0.005f; // Daha erken uyusun

// Interpolation: Sadece kamera takip ediyorsa
rb.interpolation = RigidbodyInterpolation.Interpolate;

// Collision Detection: Sadece hızlı objeler için Continuous
rb.collisionDetectionMode = CollisionDetectionMode.Discrete; // Normal objeler
```

### 4. Raycast Optimizasyonu

```csharp
// NonAlloc versiyonları kullan (GC yok)
private RaycastHit[] results = new RaycastHit[10];

void Update()
{
    int hitCount = Physics.RaycastNonAlloc(ray, results, 100f, layerMask);

    for (int i = 0; i < hitCount; i++)
    {
        // İşlem
    }
}

// Layer mask kullan (daha az kontrol)
Physics.Raycast(ray, out hit, 100f, targetLayerMask);

// Gereksiz raycast'lerden kaçın
// Her frame yerine belirli aralıklarla
```

### 5. Trigger vs Collision

```csharp
// Trigger daha performanslı (fizik hesabı yok)
// Sadece algılama gerekiyorsa trigger kullan

// Collision: Fiziksel etkileşim gerektiğinde
// Trigger: Sadece algılama (coin, checkpoint, damage zone)
```

### 6. Fixed Timestep

```csharp
// Edit > Project Settings > Time
// Fixed Timestep: 0.02 (50 FPS fizik, varsayılan)
// Daha yüksek = daha az hassas ama daha performanslı
// Daha düşük = daha hassas ama daha yavaş
```

---

## Özet ve Kontrol Listesi

Bu derste öğrendiklerimiz:
- [x] Rigidbody temelleri ve özellikleri
- [x] Kuvvet uygulama (Force, Impulse, VelocityChange)
- [x] Kinematic vs Dynamic Rigidbody
- [x] Collider türleri (Box, Sphere, Capsule, Mesh)
- [x] Collision vs Trigger farkları
- [x] OnCollision ve OnTrigger callback'leri
- [x] Physics Material (sürtünme, zıplama)
- [x] Raycast ve RaycastHit kullanımı
- [x] SphereCast, BoxCast, CapsuleCast
- [x] OverlapSphere, OverlapBox
- [x] Layer ve LayerMask sistemi
- [x] Joint türleri (Fixed, Hinge, Spring)
- [x] Pratik örnekler (karakter, platform, patlama)
- [x] Performans optimizasyonu

---

## Alıştırmalar

1. **Zıplayan Top**: Zıplama özelliği olan Physics Material oluşturun
2. **Kapı Sistemi**: Hinge Joint ile açılır kapı yapın
3. **Mermi Sistemi**: Raycast ile vuruş algılayan silah yapın
4. **Patlama**: OverlapSphere ile patlama efekti yapın
5. **Platformer**: Hareket eden platform sistemi yapın
6. **Pickup Sistemi**: Trigger ile coin toplama yapın
7. **Zemin Tipi**: Farklı yüzeylerde farklı sürtünme

---

## Faydalı Kaynaklar

- [Unity Physics Manual](https://docs.unity3d.com/Manual/PhysicsSection.html)
- [Rigidbody Documentation](https://docs.unity3d.com/Manual/class-Rigidbody.html)
- [Collider Documentation](https://docs.unity3d.com/Manual/CollidersOverview.html)
- [Physics Best Practices](https://docs.unity3d.com/Manual/PhysicsBestPractices.html)

---

## Sonraki Ders

10. Ders'te **Animasyon Sistemi** konusunu işleyeceğiz. Animator, Animation Clip, Blend Tree, State Machine ve IK (Inverse Kinematics) konularını öğreneceğiz.
