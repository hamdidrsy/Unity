# 11. Ders - Parçacık Sistemleri (Particle System)

## Giriş

Parçacık sistemleri, oyunlara görsel zenginlik katan efektler oluşturmak için kullanılır. Ateş, duman, patlama, yağmur, büyü efektleri, kan, kıvılcım - bunların hepsi parçacık sistemleriyle yapılır.

Bu derste:
- Particle System temelleri
- Emission ve Shape modülleri
- Lifetime, Speed ve Size ayarları
- Color over Lifetime
- Texture Sheet Animation
- Sub Emitters
- Collision ve Trigger
- Trail ve Lights
- Script ile kontrol
- Pratik efekt örnekleri

konularını işleyeceğiz.

---

## Particle System Temelleri

### Particle System Oluşturma

```
GameObject > Effects > Particle System
veya
Hierarchy > Sağ Tık > Effects > Particle System
```

### Particle System Yapısı

```
┌─────────────────────────────────────────────────────────────┐
│               PARTICLE SYSTEM YAPISI                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Particle System                                             │
│   ├── Main Module         → Temel ayarlar                  │
│   ├── Emission            → Parçacık üretimi               │
│   ├── Shape               → Yayılma şekli                  │
│   ├── Velocity over Lifetime                               │
│   ├── Limit Velocity over Lifetime                         │
│   ├── Inherit Velocity                                     │
│   ├── Force over Lifetime                                  │
│   ├── Color over Lifetime → Renk değişimi                  │
│   ├── Color by Speed                                       │
│   ├── Size over Lifetime  → Boyut değişimi                 │
│   ├── Size by Speed                                        │
│   ├── Rotation over Lifetime                               │
│   ├── Rotation by Speed                                    │
│   ├── External Forces                                      │
│   ├── Noise               → Rastgele hareket               │
│   ├── Collision           → Çarpışma                       │
│   ├── Triggers            → Tetikleyiciler                 │
│   ├── Sub Emitters        → Alt parçacık sistemleri        │
│   ├── Texture Sheet Animation                              │
│   ├── Lights              → Işık efekti                    │
│   ├── Trails              → İz efekti                      │
│   ├── Custom Data                                          │
│   └── Renderer            → Render ayarları                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Main Module (Ana Modül)

Parçacık sisteminin temel özelliklerini belirler.

```
┌─────────────────────────────────────────────────────────────┐
│                    MAIN MODULE                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Duration          → Sistem süresi (saniye)                 │
│ Looping           → Döngü                                  │
│ Prewarm           → Baştan dolu başla                     │
│ Start Delay       → Başlangıç gecikmesi                   │
│                                                             │
│ Start Lifetime    → Parçacık ömrü                         │
│ Start Speed       → Başlangıç hızı                        │
│ Start Size        → Başlangıç boyutu                      │
│ Start Rotation    → Başlangıç rotasyonu                   │
│ Start Color       → Başlangıç rengi                       │
│                                                             │
│ Gravity Modifier  → Yerçekimi etkisi                      │
│ Simulation Space  → World / Local / Custom                 │
│ Simulation Speed  → Simülasyon hızı                       │
│ Delta Time        → Scaled / Unscaled                      │
│                                                             │
│ Play On Awake     → Başlangıçta otomatik oynat            │
│ Max Particles     → Maksimum parçacık sayısı              │
│                                                             │
│ Scaling Mode      → Ölçeklendirme modu                    │
│ Emitter Velocity  → Hareket eden emitter için             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Değer Türleri

```
┌─────────────────────────────────────────────────────────────┐
│                   DEĞER TÜRLERİ                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Constant          → Sabit değer                            │
│                    Örn: Size = 1                           │
│                                                             │
│ Curve             → Zaman bazlı değişim                    │
│                    Grafik ile ayarlama                     │
│                                                             │
│ Random Between Two Constants                                │
│                    → İki değer arası rastgele              │
│                    Örn: Size = 0.5 ile 1.5 arası          │
│                                                             │
│ Random Between Two Curves                                   │
│                    → İki curve arası rastgele              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Emission Module

Parçacık üretim hızını kontrol eder.

```
┌─────────────────────────────────────────────────────────────┐
│                  EMISSION MODULE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Rate over Time    → Saniyede üretilen parçacık sayısı     │
│                    10 = saniyede 10 parçacık               │
│                                                             │
│ Rate over Distance → Mesafe başına parçacık               │
│                     Hareket eden objeler için              │
│                     (ayak izi, lastik izi)                 │
│                                                             │
│ Bursts            → Anlık patlama üretimi                  │
│   Time            → Ne zaman                               │
│   Count           → Kaç tane                               │
│   Cycles          → Kaç kez tekrar                         │
│   Interval        → Tekrar aralığı                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Burst Örneği

```
Patlama efekti için:
- Rate over Time: 0
- Bursts:
  - Time: 0, Count: 50, Cycles: 1
```

---

## Shape Module

Parçacıkların yayıldığı şekli belirler.

```
┌─────────────────────────────────────────────────────────────┐
│                   SHAPE MODULE                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Sphere            → Küre (her yöne)                        │
│ Hemisphere        → Yarım küre                             │
│ Cone              → Koni (ateş, fıskiye)                   │
│ Box               → Kutu                                   │
│ Circle            → Daire (2D)                             │
│ Edge              → Çizgi                                  │
│ Donut             → Halka                                  │
│ Mesh              → Özel mesh                              │
│ Mesh Renderer     → Mesh yüzeyinden                       │
│ Skinned Mesh      → Animasyonlu mesh                      │
│ Sprite            → Sprite şeklinden                       │
│ Rectangle         → Dikdörtgen                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Cone (Koni) Ayarları

```
┌─────────────────────────────────────────────────────────────┐
│                    CONE SHAPE                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│        \         /                                         │
│         \   ↑   /   Angle: Açı (0-90)                     │
│          \  │  /    Radius: Taban yarıçapı                │
│           \ │ /     Length: Uzunluk                       │
│            \│/                                             │
│             ●       Emit from:                            │
│                       - Base: Tabandan                     │
│                       - Volume: Hacimden                   │
│                       - Shell: Yüzeyden                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Velocity over Lifetime

Parçacıkların hızını zaman içinde değiştirir.

```csharp
// Linear (Doğrusal)
// X, Y, Z yönünde sabit hız ekler

// Orbital
// Merkez etrafında döndürme
// Girdap efekti için

// Radial
// Merkeze doğru/uzağa hareket
// Patlama veya içe çekme
```

---

## Color over Lifetime

Parçacık rengini ömrü boyunca değiştirir.

```
┌─────────────────────────────────────────────────────────────┐
│              COLOR OVER LIFETIME                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Gradient Editor:                                            │
│                                                             │
│ Alpha ████████████░░░░░░░░                                 │
│       ↑                  ↓                                 │
│      255                 0                                 │
│   (doğuşta)        (ölümde)                               │
│                                                             │
│ Color ●────●────●────●                                     │
│     Sarı  Turuncu Kırmızı Siyah                           │
│   (ateş renk geçişi)                                       │
│                                                             │
│ Kullanım:                                                   │
│ - Ateş: Sarı → Turuncu → Kırmızı → Siyah (duman)         │
│ - Duman: Beyaz → Gri (solarak)                            │
│ - Büyü: Parlak → Şeffaf                                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Size over Lifetime

Parçacık boyutunu zaman içinde değiştirir.

```
┌─────────────────────────────────────────────────────────────┐
│               SIZE OVER LIFETIME                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Curve kullanımı:                                           │
│                                                             │
│    1 ┤      ╭──╮                                          │
│      │     ╱    ╲                                         │
│      │    ╱      ╲                                        │
│    0 ┼───╱────────╲────                                   │
│      0%   50%     100%                                     │
│      ↑              ↑                                      │
│   Doğuş          Ölüm                                      │
│                                                             │
│ Yaygın eğriler:                                            │
│ - Büyüyüp küçül: Patlama                                  │
│ - Küçülme: Duman dağılma                                  │
│ - Büyüme: Genişleyen dalga                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Noise Module

Parçacıklara rastgele hareket ekler.

```
┌─────────────────────────────────────────────────────────────┐
│                    NOISE MODULE                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Strength       → Noise gücü                                │
│ Frequency      → Değişim sıklığı                          │
│ Scroll Speed   → Noise pattern kayma hızı                 │
│ Damping        → Sönümleme                                 │
│ Octaves        → Detay seviyesi                           │
│                                                             │
│ Separate Axes  → X, Y, Z ayrı ayarlanabilir               │
│                                                             │
│ Position Amount → Pozisyon etkisi                         │
│ Rotation Amount → Rotasyon etkisi                         │
│ Size Amount     → Boyut etkisi                            │
│                                                             │
│ Kullanım:                                                   │
│ - Duman: Yumuşak dalgalanma                               │
│ - Ateş: Titreşim                                          │
│ - Sihir: Kaotik hareket                                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Texture Sheet Animation

Sprite sheet animasyonu.

```
┌─────────────────────────────────────────────────────────────┐
│            TEXTURE SHEET ANIMATION                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Mode:                                                       │
│   Grid    → Satır ve sütun sayısı belirt                  │
│   Sprites → Ayrı sprite'lar kullan                        │
│                                                             │
│ Grid ayarları:                                              │
│   Tiles X/Y   → Sütun/Satır sayısı                        │
│                                                             │
│   ┌──┬──┬──┬──┐                                           │
│   │1 │2 │3 │4 │   4x2 Grid                                │
│   ├──┼──┼──┼──┤                                           │
│   │5 │6 │7 │8 │                                           │
│   └──┴──┴──┴──┘                                           │
│                                                             │
│ Animation:                                                  │
│   Frame over Time → Zamana göre frame                     │
│   Start Frame     → Başlangıç frame                       │
│   Cycles          → Döngü sayısı                          │
│                                                             │
│ Kullanım:                                                   │
│ - Patlama sprite animasyonu                               │
│ - Duman animasyonu                                         │
│ - Yangın efekti                                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Collision Module

Parçacıkların yüzeylerle çarpışması.

```
┌─────────────────────────────────────────────────────────────┐
│                 COLLISION MODULE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Type:                                                       │
│   Planes    → Belirli düzlemlerle (performanslı)          │
│   World     → Tüm collider'larla (yavaş)                  │
│                                                             │
│ Mode:                                                       │
│   3D        → 3D fizik                                     │
│   2D        → 2D fizik                                     │
│                                                             │
│ Dampen       → Hız kaybı (0-1)                            │
│ Bounce       → Zıplama (0-1)                              │
│ Lifetime Loss → Çarpışmada ömür kaybı                     │
│                                                             │
│ Collides With → Çarpışacağı layer'lar                     │
│ Max Kill Speed → Bu hızın altında yok et                  │
│                                                             │
│ Send Collision Messages → Script'e mesaj gönder           │
│   OnParticleCollision() çağrılır                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Collision Script

```csharp
using UnityEngine;
using System.Collections.Generic;

public class ParticleCollisionHandler : MonoBehaviour
{
    private ParticleSystem ps;
    private List<ParticleCollisionEvent> collisionEvents;

    [SerializeField] private GameObject impactEffect;

    void Start()
    {
        ps = GetComponent<ParticleSystem>();
        collisionEvents = new List<ParticleCollisionEvent>();
    }

    void OnParticleCollision(GameObject other)
    {
        // Çarpışma eventlerini al
        int numCollisionEvents = ps.GetCollisionEvents(other, collisionEvents);

        for (int i = 0; i < numCollisionEvents; i++)
        {
            Vector3 pos = collisionEvents[i].intersection;
            Vector3 normal = collisionEvents[i].normal;

            // Çarpışma noktasında efekt
            if (impactEffect != null)
            {
                Instantiate(impactEffect, pos, Quaternion.LookRotation(normal));
            }

            // Çarpılan objeye hasar ver
            IDamageable target = other.GetComponent<IDamageable>();
            target?.TakeDamage(10f);
        }
    }
}
```

---

## Triggers Module

Belirli collider'larla etkileşim.

```
┌─────────────────────────────────────────────────────────────┐
│                  TRIGGERS MODULE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Inside     → Collider içindeyken                          │
│ Outside    → Collider dışındayken                         │
│ Enter      → Collider'a girince                           │
│ Exit       → Collider'dan çıkınca                         │
│                                                             │
│ Her durum için aksiyon:                                     │
│   Callback → Script'e bildir                              │
│   Kill     → Parçacığı yok et                             │
│   Ignore   → Yoksay                                        │
│                                                             │
│ Collider List → Trigger olarak kullanılacak collider'lar  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Trigger Script

```csharp
using UnityEngine;
using System.Collections.Generic;

public class ParticleTriggerHandler : MonoBehaviour
{
    private ParticleSystem ps;
    private List<ParticleSystem.Particle> enterParticles;

    void Start()
    {
        ps = GetComponent<ParticleSystem>();
        enterParticles = new List<ParticleSystem.Particle>();
    }

    void OnParticleTrigger()
    {
        // Trigger'a giren parçacıkları al
        int numEnter = ps.GetTriggerParticles(ParticleSystemTriggerEventType.Enter, enterParticles);

        for (int i = 0; i < numEnter; i++)
        {
            ParticleSystem.Particle p = enterParticles[i];

            // Parçacık rengini değiştir
            p.startColor = Color.red;

            enterParticles[i] = p;
        }

        // Değişiklikleri uygula
        ps.SetTriggerParticles(ParticleSystemTriggerEventType.Enter, enterParticles);
    }
}
```

---

## Sub Emitters

Parçacıklardan yeni parçacık sistemleri doğurur.

```
┌─────────────────────────────────────────────────────────────┐
│                  SUB EMITTERS                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Trigger Types:                                              │
│   Birth       → Parçacık doğduğunda                       │
│   Collision   → Çarpışmada                                │
│   Death       → Öldüğünde                                 │
│   Trigger     → Trigger'a girdiğinde                      │
│   Manual      → Script ile                                 │
│                                                             │
│ Örnek: Havai fişek                                         │
│   Ana parçacık (roket) → Death'te patlama                 │
│   Patlama → Death'te kıvılcım                             │
│                                                             │
│ Inherit:                                                    │
│   Color    → Ana parçacığın rengini al                    │
│   Size     → Ana parçacığın boyutunu al                   │
│   Rotation → Ana parçacığın rotasyonunu al                │
│   Lifetime → Ana parçacığın ömrünü al                     │
│   Duration → Ana parçacığın süresini al                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Trails Module

Parçacıkların arkasında iz bırakması.

```
┌─────────────────────────────────────────────────────────────┐
│                   TRAILS MODULE                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Mode:                                                       │
│   Particles  → Her parçacığın izi                         │
│   Ribbon     → Parçacıkları birleştiren şerit             │
│                                                             │
│ Ratio            → İz bırakan parçacık oranı              │
│ Lifetime         → İz ömrü                                 │
│ Minimum Vertex Distance → Min vertex mesafesi             │
│                                                             │
│ World Space      → Dünya koordinatında iz                 │
│ Die With Particles → Parçacık ölünce iz de ölsün         │
│                                                             │
│ Width over Trail  → İz genişliği değişimi                 │
│ Color over Trail  → İz rengi değişimi                     │
│ Color over Lifetime → Zaman bazlı renk                    │
│                                                             │
│ Texture Mode:                                               │
│   Stretch     → Texture'ı uzat                            │
│   Tile        → Texture'ı tekrarla                        │
│   Distribute  → Vertex başına                             │
│   Repeat      → Mesafe bazlı tekrar                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Lights Module

Parçacıkların etrafında ışık.

```
┌─────────────────────────────────────────────────────────────┐
│                   LIGHTS MODULE                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Light         → Kullanılacak Light prefab                 │
│ Ratio         → Işık taşıyan parçacık oranı              │
│ Random Distribution → Rastgele dağılım                    │
│                                                             │
│ Use Particle Color → Parçacık rengini kullan             │
│                                                             │
│ Size Affects Range → Boyut ışık menzilini etkiler        │
│ Alpha Affects Intensity → Alpha parlaklığı etkiler       │
│                                                             │
│ Range Multiplier    → Menzil çarpanı                      │
│ Intensity Multiplier → Parlaklık çarpanı                  │
│                                                             │
│ Max Lights    → Maksimum ışık sayısı (performans)        │
│                                                             │
│ Kullanım:                                                   │
│ - Ateş parçacıkları ortamı aydınlatsın                   │
│ - Büyü efektleri parlasın                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Renderer Module

Parçacıkların nasıl çizileceğini belirler.

```
┌─────────────────────────────────────────────────────────────┐
│                  RENDERER MODULE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Render Mode:                                                │
│   Billboard   → Kameraya dönük (varsayılan)              │
│   Stretched Billboard → Hıza göre uzayan                  │
│   Horizontal Billboard → Yatay                            │
│   Vertical Billboard → Dikey                              │
│   Mesh        → 3D mesh kullan                            │
│   None        → Görünmez (trigger için)                   │
│                                                             │
│ Material      → Parçacık materyali                        │
│ Trail Material → İz materyali                             │
│                                                             │
│ Sort Mode:                                                  │
│   None        → Sıralama yok                              │
│   By Distance → Mesafeye göre                             │
│   Oldest in Front → Eskiler önde                         │
│   Youngest in Front → Yeniler önde                       │
│                                                             │
│ Min/Max Particle Size → Ekran boyutu sınırı              │
│                                                             │
│ Render Alignment:                                           │
│   View        → Kameraya dönük                            │
│   World       → Dünya eksenlerine                         │
│   Local       → Yerel eksenlere                           │
│   Facing      → Hareket yönüne                            │
│   Velocity    → Hız vektörüne                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Script ile Kontrol

### Temel Kontrol

```csharp
using UnityEngine;

public class ParticleController : MonoBehaviour
{
    private ParticleSystem ps;
    private ParticleSystem.MainModule mainModule;
    private ParticleSystem.EmissionModule emissionModule;

    void Start()
    {
        ps = GetComponent<ParticleSystem>();
        mainModule = ps.main;
        emissionModule = ps.emission;
    }

    // Oynat / Durdur
    public void PlayEffect()
    {
        ps.Play();
    }

    public void StopEffect()
    {
        ps.Stop();
    }

    public void PauseEffect()
    {
        ps.Pause();
    }

    // Temizle
    public void ClearParticles()
    {
        ps.Clear();
    }

    // Çalışıyor mu?
    public bool IsPlaying()
    {
        return ps.isPlaying;
    }

    // Parçacık sayısı
    public int GetParticleCount()
    {
        return ps.particleCount;
    }
}
```

### Modül Ayarlarını Değiştirme

```csharp
using UnityEngine;

public class ParticleModuleControl : MonoBehaviour
{
    private ParticleSystem ps;

    void Start()
    {
        ps = GetComponent<ParticleSystem>();
    }

    // Main Module
    public void SetDuration(float duration)
    {
        var main = ps.main;
        main.duration = duration;
    }

    public void SetStartColor(Color color)
    {
        var main = ps.main;
        main.startColor = color;
    }

    public void SetStartSize(float minSize, float maxSize)
    {
        var main = ps.main;
        main.startSize = new ParticleSystem.MinMaxCurve(minSize, maxSize);
    }

    public void SetGravity(float gravity)
    {
        var main = ps.main;
        main.gravityModifier = gravity;
    }

    public void SetSimulationSpeed(float speed)
    {
        var main = ps.main;
        main.simulationSpeed = speed;
    }

    // Emission Module
    public void SetEmissionRate(float rate)
    {
        var emission = ps.emission;
        emission.rateOverTime = rate;
    }

    public void EnableEmission(bool enabled)
    {
        var emission = ps.emission;
        emission.enabled = enabled;
    }

    // Shape Module
    public void SetShapeRadius(float radius)
    {
        var shape = ps.shape;
        shape.radius = radius;
    }

    public void SetShapeAngle(float angle)
    {
        var shape = ps.shape;
        shape.angle = angle;
    }

    // Burst ekle
    public void AddBurst(float time, int count)
    {
        var emission = ps.emission;
        emission.SetBurst(0, new ParticleSystem.Burst(time, count));
    }
}
```

### Parçacıklara Doğrudan Erişim

```csharp
using UnityEngine;

public class ParticleDirectAccess : MonoBehaviour
{
    private ParticleSystem ps;
    private ParticleSystem.Particle[] particles;

    void Start()
    {
        ps = GetComponent<ParticleSystem>();
        particles = new ParticleSystem.Particle[ps.main.maxParticles];
    }

    void Update()
    {
        // Parçacıkları al
        int count = ps.GetParticles(particles);

        // Her parçacığı işle
        for (int i = 0; i < count; i++)
        {
            // Pozisyona göre renk değiştir
            if (particles[i].position.y > 5f)
            {
                particles[i].startColor = Color.red;
            }

            // Belirli koşulda yok et
            if (particles[i].position.y > 10f)
            {
                particles[i].remainingLifetime = 0f;
            }
        }

        // Değişiklikleri uygula
        ps.SetParticles(particles, count);
    }

    // Tek parçacık emit et
    public void EmitSingle(Vector3 position, Vector3 velocity)
    {
        var emitParams = new ParticleSystem.EmitParams();
        emitParams.position = position;
        emitParams.velocity = velocity;
        emitParams.startSize = 1f;
        emitParams.startLifetime = 2f;
        emitParams.startColor = Color.white;

        ps.Emit(emitParams, 1);
    }

    // Belirli sayıda emit
    public void EmitMultiple(int count)
    {
        ps.Emit(count);
    }
}
```

---

## Pratik Efekt Örnekleri

### 1. Ateş Efekti

```
Main Module:
  Duration: 1
  Looping: ✓
  Start Lifetime: 1-2
  Start Speed: 2-3
  Start Size: 0.5-1
  Start Color: Gradient (Sarı → Turuncu)
  Gravity: -0.2 (yukarı)

Emission:
  Rate: 50

Shape:
  Shape: Cone
  Angle: 15
  Radius: 0.3

Color over Lifetime:
  Gradient: Sarı → Turuncu → Kırmızı → Siyah (Alpha: 0)

Size over Lifetime:
  Curve: Büyüyerek küçül

Noise:
  Strength: 0.5
  Frequency: 2

Renderer:
  Material: Additive particle material
```

### 2. Duman Efekti

```
Main Module:
  Duration: 1
  Looping: ✓
  Start Lifetime: 3-5
  Start Speed: 1-2
  Start Size: 1-2
  Start Color: Gri (alpha 0.5)
  Gravity: -0.1

Emission:
  Rate: 20

Shape:
  Shape: Cone
  Angle: 30
  Radius: 0.5

Color over Lifetime:
  Alpha: 0.5 → 0

Size over Lifetime:
  Curve: Büyüyen (1 → 3)

Noise:
  Strength: 1
  Frequency: 0.5

Rotation over Lifetime:
  Angular Velocity: 30
```

### 3. Patlama Efekti

```csharp
using UnityEngine;

public class ExplosionEffect : MonoBehaviour
{
    [SerializeField] private ParticleSystem explosionCore;
    [SerializeField] private ParticleSystem explosionSparks;
    [SerializeField] private ParticleSystem explosionSmoke;
    [SerializeField] private ParticleSystem explosionDebris;

    public void Explode()
    {
        // Hepsini aynı anda oynat
        explosionCore.Play();
        explosionSparks.Play();
        explosionSmoke.Play();
        explosionDebris.Play();

        // Belirli süre sonra yok et
        float maxDuration = Mathf.Max(
            explosionCore.main.duration,
            explosionSparks.main.duration,
            explosionSmoke.main.duration,
            explosionDebris.main.duration
        );

        Destroy(gameObject, maxDuration + 1f);
    }
}
```

**Patlama Core ayarları:**
```
Emission:
  Bursts: Time 0, Count 30

Shape:
  Shape: Sphere

Color over Lifetime:
  Sarı → Turuncu → Siyah (alpha 0)

Size over Lifetime:
  Hızlı büyüyüp küçül
```

### 4. Yağmur Efekti

```
Main Module:
  Duration: 1
  Looping: ✓
  Start Lifetime: 1
  Start Speed: 20
  Start Size: 0.05-0.1
  Start Color: Açık mavi
  Gravity: 0 (zaten hızlı)
  Simulation Space: World

Emission:
  Rate: 500

Shape:
  Shape: Box
  Scale: (20, 0, 20)
  Position: Kameranın üstünde

Renderer:
  Render Mode: Stretched Billboard
  Length Scale: 0.1
  Speed Scale: 0.05
```

### 5. Büyü Efekti (Magic Circle)

```csharp
using UnityEngine;

public class MagicCircleEffect : MonoBehaviour
{
    [SerializeField] private ParticleSystem outerRing;
    [SerializeField] private ParticleSystem innerGlow;
    [SerializeField] private ParticleSystem sparkles;
    [SerializeField] private ParticleSystem runes;

    [SerializeField] private float chargeTime = 2f;
    private float currentCharge = 0f;
    private bool isCharging = false;

    public void StartCharging()
    {
        isCharging = true;
        outerRing.Play();
        innerGlow.Play();
        sparkles.Play();
    }

    public void StopCharging()
    {
        isCharging = false;
        outerRing.Stop();
        innerGlow.Stop();
        sparkles.Stop();
        currentCharge = 0f;
    }

    void Update()
    {
        if (isCharging)
        {
            currentCharge += Time.deltaTime;

            // Şarj ilerledikçe efekti güçlendir
            float chargePercent = currentCharge / chargeTime;

            var mainOuter = outerRing.main;
            mainOuter.startSize = Mathf.Lerp(1f, 3f, chargePercent);

            var emissionSparkles = sparkles.emission;
            emissionSparkles.rateOverTime = Mathf.Lerp(10f, 100f, chargePercent);

            if (currentCharge >= chargeTime)
            {
                CastSpell();
            }
        }
    }

    void CastSpell()
    {
        Debug.Log("Büyü yapıldı!");
        runes.Play();
        StopCharging();
    }
}
```

### 6. Kan/Hasar Efekti

```
Main Module:
  Duration: 0.5
  Looping: ✗
  Start Lifetime: 0.5-1
  Start Speed: 5-10
  Start Size: 0.1-0.3
  Start Color: Kırmızı
  Gravity: 2

Emission:
  Bursts: Time 0, Count 20-30

Shape:
  Shape: Cone
  Angle: 45

Color over Lifetime:
  Kırmızı → Koyu kırmızı (alpha 0)

Collision:
  Type: World
  Bounce: 0.2
  Lifetime Loss: 0.5
```

### 7. İz Bırakan Mermi

```csharp
using UnityEngine;

public class BulletTrail : MonoBehaviour
{
    [SerializeField] private ParticleSystem trailEffect;

    private ParticleSystem.EmissionModule emission;

    void Start()
    {
        emission = trailEffect.emission;
    }

    void Update()
    {
        // Hareket ediyorsa iz bırak
        if (GetComponent<Rigidbody>().velocity.magnitude > 0.1f)
        {
            if (!trailEffect.isPlaying)
                trailEffect.Play();
        }
        else
        {
            trailEffect.Stop();
        }
    }
}
```

**Trail ayarları:**
```
Emission:
  Rate over Distance: 10

Trails:
  Mode: Particles
  Lifetime: 0.5
  Width over Trail: 1 → 0
  Color over Trail: Beyaz → Şeffaf
```

---

## Object Pooling ile Particle System

```csharp
using UnityEngine;
using System.Collections.Generic;

public class ParticlePool : MonoBehaviour
{
    public static ParticlePool Instance { get; private set; }

    [System.Serializable]
    public class PoolItem
    {
        public string name;
        public ParticleSystem prefab;
        public int poolSize = 10;
    }

    [SerializeField] private List<PoolItem> poolItems;

    private Dictionary<string, Queue<ParticleSystem>> pools;

    void Awake()
    {
        Instance = this;
        InitializePools();
    }

    void InitializePools()
    {
        pools = new Dictionary<string, Queue<ParticleSystem>>();

        foreach (PoolItem item in poolItems)
        {
            Queue<ParticleSystem> queue = new Queue<ParticleSystem>();

            for (int i = 0; i < item.poolSize; i++)
            {
                ParticleSystem ps = Instantiate(item.prefab, transform);
                ps.gameObject.SetActive(false);
                queue.Enqueue(ps);
            }

            pools[item.name] = queue;
        }
    }

    public ParticleSystem GetParticle(string name)
    {
        if (!pools.ContainsKey(name))
        {
            Debug.LogWarning($"Pool '{name}' bulunamadı!");
            return null;
        }

        Queue<ParticleSystem> queue = pools[name];

        if (queue.Count == 0)
        {
            // Pool boşsa yeni oluştur
            PoolItem item = poolItems.Find(x => x.name == name);
            if (item != null)
            {
                ParticleSystem newPs = Instantiate(item.prefab, transform);
                return newPs;
            }
            return null;
        }

        ParticleSystem ps = queue.Dequeue();
        ps.gameObject.SetActive(true);
        return ps;
    }

    public void ReturnParticle(string name, ParticleSystem ps)
    {
        ps.Stop();
        ps.Clear();
        ps.gameObject.SetActive(false);

        if (pools.ContainsKey(name))
        {
            pools[name].Enqueue(ps);
        }
    }

    // Kullanım kolaylığı için
    public void PlayAt(string name, Vector3 position, Quaternion rotation)
    {
        ParticleSystem ps = GetParticle(name);
        if (ps != null)
        {
            ps.transform.position = position;
            ps.transform.rotation = rotation;
            ps.Play();

            // Otomatik geri dönüş
            StartCoroutine(ReturnAfterPlay(name, ps));
        }
    }

    System.Collections.IEnumerator ReturnAfterPlay(string name, ParticleSystem ps)
    {
        yield return new WaitForSeconds(ps.main.duration + ps.main.startLifetime.constantMax);
        ReturnParticle(name, ps);
    }
}

// Kullanım:
// ParticlePool.Instance.PlayAt("Explosion", transform.position, Quaternion.identity);
```

---

## Performans İpuçları

```
┌─────────────────────────────────────────────────────────────┐
│              PERFORMANS İPUÇLARI                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 1. Max Particles sınırla                                   │
│    - Çok fazla parçacık performans düşürür               │
│    - Görsel kaliteyi koruyarak minimize et                │
│                                                             │
│ 2. Collision dikkatli kullan                               │
│    - World collision yavaş                                 │
│    - Mümkünse Planes kullan                               │
│    - Collision Quality: Low/Medium                         │
│                                                             │
│ 3. Texture boyutu                                          │
│    - Particle texture'ları küçük tut                      │
│    - Atlas kullan                                          │
│                                                             │
│ 4. Shader seçimi                                           │
│    - Particles/Standard Unlit hızlı                       │
│    - Soft particles maliyetli                             │
│                                                             │
│ 5. Object Pooling                                          │
│    - Instantiate/Destroy yerine pool kullan               │
│                                                             │
│ 6. LOD (Level of Detail)                                   │
│    - Uzaktaki efektler basitleştirilsin                   │
│    - Veya hiç gösterilmesin                               │
│                                                             │
│ 7. Culling                                                  │
│    - Kamera dışındaki efektler durdurul                   │
│                                                             │
│ 8. Emission rate                                            │
│    - Gerekenden fazla emit etme                           │
│    - Daha az parçacık, daha büyük boyut                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Özet ve Kontrol Listesi

Bu derste öğrendiklerimiz:
- [x] Particle System yapısı ve modüller
- [x] Main Module ayarları
- [x] Emission (Rate, Bursts)
- [x] Shape modülü ve şekil türleri
- [x] Color over Lifetime
- [x] Size over Lifetime
- [x] Noise modülü
- [x] Collision ve Triggers
- [x] Sub Emitters
- [x] Trails modülü
- [x] Lights modülü
- [x] Texture Sheet Animation
- [x] Script ile kontrol
- [x] Pratik efekt örnekleri
- [x] Object Pooling
- [x] Performans optimizasyonu

---

## Alıştırmalar

1. **Ateş Efekti**: Kamp ateşi yapın
2. **Duman**: Baca dumanı efekti
3. **Patlama**: Sub Emitter'lı patlama
4. **Yağmur**: Düşen damla efekti
5. **Büyü**: Şarj olan büyü çemberi
6. **İz**: Mermi/roket izi
7. **Çarpışma**: Zemine çarpan ve sıçrayan su

---

## Sonraki Ders

12. Ders'te **Sahne Yönetimi ve Geçişler** konusunu işleyeceğiz. Scene loading, additive scenes, loading screen ve sahne geçiş efektleri öğreneceğiz.
