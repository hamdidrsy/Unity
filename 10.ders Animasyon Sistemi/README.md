# 10. Ders - Animasyon Sistemi

## Giriş

Animasyon, oyunlara hayat veren temel unsurdur. Karakter yürüyüşleri, kapı açılmaları, UI geçişleri - hepsi animasyon sistemiyle oluşturulur.

Bu derste:
- Animation Clip oluşturma
- Animator Controller ve State Machine
- Animator parametreleri ve geçişler
- Blend Tree kullanımı
- Animation Events
- Root Motion
- Avatar ve Humanoid animasyonlar
- IK (Inverse Kinematics)
- Scripting ile animasyon kontrolü

konularını işleyeceğiz.

---

## Animasyon Temelleri

Unity'de iki animasyon sistemi vardır:

```
┌─────────────────────────────────────────────────────────────┐
│                 ANİMASYON SİSTEMLERİ                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Legacy Animation (Eski)                                     │
│   - Animation component                                     │
│   - Basit kullanım                                         │
│   - Sınırlı özellikler                                     │
│   - Artık önerilmiyor                                      │
│                                                             │
│ Mecanim (Yeni - Önerilen)                                  │
│   - Animator component                                      │
│   - State Machine                                          │
│   - Blend Trees                                            │
│   - Humanoid retargeting                                   │
│   - IK desteği                                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Temel Kavramlar

```
┌─────────────────────────────────────────────────────────────┐
│                 TEMEL KAVRAMLAR                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Animation Clip    → Tek bir animasyon (yürüme, zıplama)    │
│ Animator Controller → State machine, geçişler              │
│ Animator          → GameObject'e eklenen component         │
│ Avatar            → Humanoid kemik yapısı eşlemesi         │
│ State             → Animasyon durumu                       │
│ Transition        → Durumlar arası geçiş                   │
│ Parameter         → Geçişleri kontrol eden değişken        │
│ Layer             → Animasyon katmanları                   │
│ Blend Tree        → Animasyonları karıştırma               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Animation Clip Oluşturma

### Unity'de Animasyon Oluşturma

```
1. GameObject seç
2. Window > Animation > Animation (Ctrl+6)
3. Create butonuna tıkla
4. Animasyon dosyasını kaydet (.anim)
```

### Animation Window

```
┌─────────────────────────────────────────────────────────────┐
│                 ANIMATION WINDOW                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ┌─────────────────────────────────────────────────────────┐│
│ │ [►] [●]  0:00  |  0:30  |  1:00  |  1:30  |  2:00      ││
│ ├─────────────────────────────────────────────────────────┤│
│ │ ▼ Transform                                             ││
│ │   ├─ Position    ◆────────◆────────◆                   ││
│ │   ├─ Rotation    ◆────────────────◆                    ││
│ │   └─ Scale       ◆                                      ││
│ │ ▼ Sprite Renderer                                       ││
│ │   └─ Color       ◆────────◆                            ││
│ └─────────────────────────────────────────────────────────┘│
│                                                             │
│ ◆ = Keyframe                                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Keyframe Ekleme

```csharp
// Manuel keyframe ekleme
// 1. Timeline'da istediğin zamana git
// 2. Property'nin yanındaki ◆ butonuna tıkla
// veya
// 3. Inspector'da değeri değiştir, otomatik kayıt açıksa eklenir

// Record Mode (●)
// - Kırmızı kayıt butonuna tıkla
// - Inspector'da değişiklik yap
// - Otomatik keyframe eklenir
```

### Curves ve Dopesheet

```
┌─────────────────────────────────────────────────────────────┐
│                 GÖRÜNÜM MODLARI                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Dopesheet (Varsayılan)                                     │
│   - Keyframe'leri nokta olarak gösterir                   │
│   - Zamanlama düzenlemesi için ideal                       │
│                                                             │
│ Curves                                                      │
│   - Değer değişimini grafik olarak gösterir               │
│   - Easing ve interpolasyon ayarı için                    │
│   - Bezier curve düzenleme                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Animasyon Özellikleri

```csharp
// Inspector'da Animation Clip seçiliyken:

Loop Time        → Döngü olsun mu
Loop Pose        → Döngüde poz uyumu
Cycle Offset     → Döngü başlangıç ofseti

// Root Motion ayarları
Root Transform Rotation    → Dönüş root motion
Root Transform Position Y  → Dikey hareket
Root Transform Position XZ → Yatay hareket
```

---

## Animator Controller

State Machine mantığıyla animasyonları yönetir.

### Animator Controller Oluşturma

```
Project > Sağ Tık > Create > Animator Controller
```

### Animator Window

```
Window > Animation > Animator
```

```
┌─────────────────────────────────────────────────────────────┐
│                 ANIMATOR WINDOW                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   [Entry] ──────► [Idle] ◄──────► [Walk]                   │
│                     │                │                      │
│                     ▼                ▼                      │
│                  [Jump]          [Run]                      │
│                     │                                       │
│                     ▼                                       │
│                  [Fall]                                     │
│                     │                                       │
│                     ▼                                       │
│                [Any State] ─────► [Death]                  │
│                                                             │
│   [Exit]                                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Özel State'ler

```
┌─────────────────────────────────────────────────────────────┐
│                  ÖZEL STATE'LER                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Entry       → Başlangıç noktası (değiştirilemez)          │
│ Exit        → Çıkış noktası (sub-state machine için)       │
│ Any State   → Herhangi bir state'den geçiş               │
│               (ölüm, hasar alma gibi)                      │
│                                                             │
│ Default State (Turuncu) → İlk oynatılacak animasyon       │
│   - Sağ tık > Set as Layer Default State                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### State Ekleme

```
1. Animator Window'da sağ tık
2. Create State > Empty veya From New Blend Tree
3. State'i seç, Inspector'da Motion alanına Animation Clip ata
```

---

## Transitions (Geçişler)

State'ler arası geçişleri tanımlar.

### Transition Oluşturma

```
1. Kaynak state'e sağ tık
2. Make Transition
3. Hedef state'e tıkla
```

### Transition Ayarları

```
┌─────────────────────────────────────────────────────────────┐
│               TRANSITION AYARLARI                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Has Exit Time     → Animasyon bitince mi geçsin           │
│                    ✓ = Animasyon sonunda geçer            │
│                    ✗ = Condition sağlanınca hemen geçer   │
│                                                             │
│ Exit Time         → Geçiş zamanı (0-1, normalize)         │
│                    0.8 = %80'inde geçiş başlar            │
│                                                             │
│ Fixed Duration    → Süre saniye mi, normalize mi          │
│                                                             │
│ Transition Duration → Geçiş süresi (blending)             │
│                    0 = Anında geçiş                        │
│                    0.25 = Yumuşak geçiş                    │
│                                                             │
│ Transition Offset → Hedef animasyonun başlangıç noktası  │
│                                                             │
│ Interruption Source → Geçiş kesilirse ne olur             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Transition Graph

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Kaynak Animasyon          Hedef Animasyon                 │
│  ████████████░░░░          ░░░░████████████                │
│             ↑  ↑            ↑  ↑                           │
│          Exit  Transition   Offset                         │
│          Time  Duration                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Parameters (Parametreler)

Animasyon geçişlerini kontrol eden değişkenler.

### Parametre Türleri

```
┌─────────────────────────────────────────────────────────────┐
│                 PARAMETRE TÜRLERİ                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Float    → Ondalıklı sayı (hız, yön)                      │
│            Blend Tree'lerde kullanılır                     │
│                                                             │
│ Int      → Tam sayı (silah tipi, combo sayısı)            │
│                                                             │
│ Bool     → Doğru/Yanlış (yerde mi, koşuyor mu)            │
│            Sürekli durum kontrolü                          │
│                                                             │
│ Trigger  → Tek seferlik tetik (zıpla, ateş et)            │
│            Otomatik resetlenir                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Parametre Ekleme

```
Animator Window > Parameters sekmesi > + butonu
```

### Condition (Koşul) Ekleme

```
1. Transition okunu seç
2. Inspector'da Conditions bölümüne git
3. + butonuyla koşul ekle
4. Parametre ve değeri seç
```

```
┌─────────────────────────────────────────────────────────────┐
│                   CONDITION ÖRNEKLERİ                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Speed > 0.1         → Float koşulu                        │
│ IsGrounded = true   → Bool koşulu                         │
│ WeaponType = 2      → Int koşulu                          │
│ Jump                → Trigger (değer gerekmez)            │
│                                                             │
│ Birden fazla condition = AND mantığı                       │
│ (Hepsi sağlanmalı)                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Script ile Animator Kontrolü

### Temel Kullanım

```csharp
using UnityEngine;

public class AnimatorBasics : MonoBehaviour
{
    private Animator animator;

    // Parametre hash'leri (performans için)
    private static readonly int SpeedHash = Animator.StringToHash("Speed");
    private static readonly int IsGroundedHash = Animator.StringToHash("IsGrounded");
    private static readonly int JumpHash = Animator.StringToHash("Jump");
    private static readonly int AttackHash = Animator.StringToHash("Attack");

    void Start()
    {
        animator = GetComponent<Animator>();
    }

    void Update()
    {
        // Float parametre
        float speed = Input.GetAxis("Vertical");
        animator.SetFloat(SpeedHash, Mathf.Abs(speed));

        // Bool parametre
        animator.SetBool(IsGroundedHash, IsGrounded());

        // Trigger parametre
        if (Input.GetButtonDown("Jump"))
        {
            animator.SetTrigger(JumpHash);
        }

        if (Input.GetButtonDown("Fire1"))
        {
            animator.SetTrigger(AttackHash);
        }
    }

    // Smooth değer değişimi
    void SmoothSpeed()
    {
        float targetSpeed = Input.GetAxis("Vertical");
        float currentSpeed = animator.GetFloat(SpeedHash);
        float smoothSpeed = Mathf.Lerp(currentSpeed, targetSpeed, Time.deltaTime * 10f);
        animator.SetFloat(SpeedHash, smoothSpeed);

        // veya SetFloat'ın dampTime parametresi
        animator.SetFloat(SpeedHash, targetSpeed, 0.1f, Time.deltaTime);
    }

    bool IsGrounded()
    {
        return Physics.Raycast(transform.position, Vector3.down, 0.1f);
    }
}
```

### Parametre Okuma

```csharp
public class ReadAnimatorParams : MonoBehaviour
{
    private Animator animator;

    void Start()
    {
        animator = GetComponent<Animator>();
    }

    void Update()
    {
        // Parametre değerlerini oku
        float speed = animator.GetFloat("Speed");
        bool isGrounded = animator.GetBool("IsGrounded");
        int weaponType = animator.GetInteger("WeaponType");

        // Mevcut state bilgisi
        AnimatorStateInfo stateInfo = animator.GetCurrentAnimatorStateInfo(0);

        // State adı kontrolü
        if (stateInfo.IsName("Idle"))
        {
            Debug.Log("Idle animasyonunda");
        }

        // Tag kontrolü
        if (stateInfo.IsTag("Attack"))
        {
            Debug.Log("Saldırı animasyonunda");
        }

        // Animasyon ilerleme (0-1)
        float progress = stateInfo.normalizedTime % 1f;

        // Animasyon uzunluğu
        float length = stateInfo.length;
    }
}
```

### Animasyon Kontrolü

```csharp
public class AnimatorControl : MonoBehaviour
{
    private Animator animator;

    void Start()
    {
        animator = GetComponent<Animator>();
    }

    // Animasyonu doğrudan oynat
    public void PlayAnimation(string stateName)
    {
        animator.Play(stateName);
    }

    // Belirli bir noktadan başlat
    public void PlayFromPoint(string stateName, float normalizedTime)
    {
        animator.Play(stateName, 0, normalizedTime);
        // 0 = layer index
        // normalizedTime = 0-1 arası başlangıç noktası
    }

    // Crossfade ile geçiş
    public void CrossfadeTo(string stateName)
    {
        animator.CrossFade(stateName, 0.25f);
        // 0.25f = geçiş süresi
    }

    // Hız kontrolü
    public void SetAnimationSpeed(float speed)
    {
        animator.speed = speed;
    }

    // Animasyonu duraklat
    public void PauseAnimation()
    {
        animator.speed = 0f;
    }

    // Devam ettir
    public void ResumeAnimation()
    {
        animator.speed = 1f;
    }

    // Animator'ı devre dışı bırak
    public void DisableAnimator()
    {
        animator.enabled = false;
    }

    // Layer ağırlığı
    public void SetLayerWeight(int layerIndex, float weight)
    {
        animator.SetLayerWeight(layerIndex, weight);
    }
}
```

---

## Blend Tree

Birden fazla animasyonu parametre değerine göre karıştırır.

### Blend Tree Türleri

```
┌─────────────────────────────────────────────────────────────┐
│                  BLEND TREE TÜRLERİ                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 1D Blend Tree                                              │
│    Tek parametre (hız)                                     │
│    [Idle]──[Walk]──[Run]                                   │
│      0      0.5      1                                     │
│                                                             │
│ 2D Simple Directional                                       │
│    İki parametre (x, y yön)                                │
│    Her yönde tek animasyon                                 │
│                                                             │
│ 2D Freeform Directional                                     │
│    İki parametre, aynı yönde birden fazla                  │
│    Walk Forward, Run Forward aynı yönde                    │
│                                                             │
│ 2D Freeform Cartesian                                       │
│    İki bağımsız parametre                                  │
│    Örn: Speed ve Turn açısı                                │
│                                                             │
│ Direct                                                      │
│    Her animasyon için ayrı ağırlık parametresi            │
│    Yüz ifadeleri için ideal                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1D Blend Tree Oluşturma

```
1. Animator'da sağ tık > Create State > From New Blend Tree
2. Blend Tree'ye çift tıkla (içine gir)
3. Inspector'da Blend Type: 1D seç
4. Parameter seç (örn: Speed)
5. Motion listesine animasyonları ekle
6. Threshold değerlerini ayarla
```

```
┌─────────────────────────────────────────────────────────────┐
│                   1D BLEND TREE                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Parameter: Speed                                            │
│                                                             │
│ Motion          Threshold                                   │
│ ─────────────────────────                                  │
│ Idle            0.0                                        │
│ Walk            0.5                                        │
│ Run             1.0                                        │
│                                                             │
│      ◆─────────◆─────────◆                                │
│    Idle      Walk       Run                                │
│     0        0.5         1                                 │
│              ↑                                              │
│         Speed = 0.5 ise                                    │
│         %100 Walk animasyonu                               │
│                                                             │
│         Speed = 0.75 ise                                   │
│         %50 Walk + %50 Run                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2D Blend Tree

```csharp
// 2D Blend Tree için iki parametre gerekli
// Örnek: Horizontal ve Vertical

void Update()
{
    float h = Input.GetAxis("Horizontal");
    float v = Input.GetAxis("Vertical");

    animator.SetFloat("Horizontal", h);
    animator.SetFloat("Vertical", v);
}
```

```
┌─────────────────────────────────────────────────────────────┐
│                  2D BLEND TREE                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│              Forward (0,1)                                  │
│                   ▲                                        │
│                   │                                        │
│   Left ◄─────────●─────────► Right                        │
│  (-1,0)          │           (1,0)                         │
│                   │                                        │
│                   ▼                                        │
│             Backward (0,-1)                                │
│                                                             │
│   ● = Blend noktası (Horizontal, Vertical)                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Blend Tree Script Örneği

```csharp
using UnityEngine;

public class BlendTreeController : MonoBehaviour
{
    private Animator animator;
    private CharacterController controller;

    [SerializeField] private float walkSpeed = 2f;
    [SerializeField] private float runSpeed = 6f;
    [SerializeField] private float acceleration = 10f;

    private float currentSpeed;
    private Vector2 smoothInput;

    void Start()
    {
        animator = GetComponent<Animator>();
        controller = GetComponent<CharacterController>();
    }

    void Update()
    {
        // Input al
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");
        Vector2 input = new Vector2(h, v);

        // Input'u smooth yap
        smoothInput = Vector2.Lerp(smoothInput, input, Time.deltaTime * acceleration);

        // Koşma kontrolü
        bool isRunning = Input.GetKey(KeyCode.LeftShift);
        float targetSpeed = isRunning ? runSpeed : walkSpeed;

        // Hareket hızı
        float inputMagnitude = smoothInput.magnitude;
        currentSpeed = Mathf.Lerp(currentSpeed, inputMagnitude * targetSpeed, Time.deltaTime * acceleration);

        // Animator parametreleri (1D için)
        animator.SetFloat("Speed", currentSpeed / runSpeed);

        // 2D Blend Tree için
        animator.SetFloat("Horizontal", smoothInput.x);
        animator.SetFloat("Vertical", smoothInput.y);

        // Hareket
        Vector3 move = new Vector3(smoothInput.x, 0, smoothInput.y);
        move = transform.TransformDirection(move);
        controller.Move(move * currentSpeed * Time.deltaTime);
    }
}
```

---

## Animation Events

Animasyon sırasında fonksiyon çağırır.

### Event Ekleme

```
1. Animation Window'da animasyonu aç
2. Timeline'da istediğin frame'e git
3. Sağ tık > Add Animation Event
4. Inspector'da Function seç
```

### Event Handler Script

```csharp
using UnityEngine;

public class AnimationEventHandler : MonoBehaviour
{
    [SerializeField] private AudioClip footstepSound;
    [SerializeField] private AudioClip attackSound;
    [SerializeField] private GameObject hitEffectPrefab;
    [SerializeField] private Transform weaponTip;

    private AudioSource audioSource;

    void Start()
    {
        audioSource = GetComponent<AudioSource>();
    }

    // Ayak sesi (Animation Event tarafından çağrılır)
    public void Footstep()
    {
        audioSource.PlayOneShot(footstepSound);
    }

    // Parametreli event
    public void FootstepWithVolume(float volume)
    {
        audioSource.PlayOneShot(footstepSound, volume);
    }

    // Saldırı başlangıcı
    public void AttackStart()
    {
        Debug.Log("Saldırı başladı");
        // Hasar collider'ı aktif et
    }

    // Saldırı vuruş anı
    public void AttackHit()
    {
        audioSource.PlayOneShot(attackSound);

        // Hasar ver
        Collider[] hits = Physics.OverlapSphere(weaponTip.position, 1f);
        foreach (Collider hit in hits)
        {
            IDamageable target = hit.GetComponent<IDamageable>();
            target?.TakeDamage(25f);
        }
    }

    // Saldırı bitişi
    public void AttackEnd()
    {
        Debug.Log("Saldırı bitti");
        // Hasar collider'ı deaktif et
    }

    // Efekt spawn
    public void SpawnEffect(Object effectPrefab)
    {
        GameObject prefab = effectPrefab as GameObject;
        if (prefab != null)
        {
            Instantiate(prefab, transform.position, Quaternion.identity);
        }
    }

    // String parametre
    public void PlaySound(string soundName)
    {
        AudioClip clip = Resources.Load<AudioClip>($"Sounds/{soundName}");
        if (clip != null)
        {
            audioSource.PlayOneShot(clip);
        }
    }
}
```

### Event Parametre Türleri

```
┌─────────────────────────────────────────────────────────────┐
│                EVENT PARAMETRELERİ                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Desteklenen türler:                                        │
│   - Float                                                  │
│   - Int                                                    │
│   - String                                                 │
│   - Object (UnityEngine.Object)                           │
│                                                             │
│ Örnek fonksiyon imzaları:                                  │
│   void MyFunction()                                        │
│   void MyFunction(float value)                             │
│   void MyFunction(int value)                               │
│   void MyFunction(string text)                             │
│   void MyFunction(Object obj)                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Animator Layers

Birden fazla animasyonu aynı anda oynatmak için.

### Layer Kullanım Alanları

```
┌─────────────────────────────────────────────────────────────┐
│                   LAYER KULLANIMI                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Base Layer      → Tam vücut animasyonları                  │
│                   (Idle, Walk, Run, Jump)                  │
│                                                             │
│ Upper Body Layer → Üst vücut override                      │
│                   (Silah tutma, el sallama)                │
│                   Avatar Mask ile sadece üst vücut         │
│                                                             │
│ Face Layer      → Yüz ifadeleri                            │
│                   Additive blending                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Layer Oluşturma

```
1. Animator Window > Layers sekmesi
2. + butonuna tıkla
3. Layer ayarlarını yap
```

### Layer Ayarları

```
┌─────────────────────────────────────────────────────────────┐
│                   LAYER AYARLARI                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Weight         → Katman ağırlığı (0-1)                     │
│                                                             │
│ Blending Mode:                                              │
│   Override     → Alt katmanı değiştirir                    │
│   Additive     → Alt katmana eklenir                       │
│                                                             │
│ Sync           → Başka layer ile senkronize               │
│                                                             │
│ Avatar Mask    → Hangi kemikler etkilensin                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Avatar Mask Oluşturma

```
Project > Sağ Tık > Create > Avatar Mask
```

```csharp
// Script ile layer weight kontrolü
public class LayerController : MonoBehaviour
{
    private Animator animator;
    private int upperBodyLayerIndex;

    void Start()
    {
        animator = GetComponent<Animator>();
        upperBodyLayerIndex = animator.GetLayerIndex("UpperBody");
    }

    public void SetUpperBodyWeight(float weight)
    {
        animator.SetLayerWeight(upperBodyLayerIndex, weight);
    }

    // Silah tutarken üst vücut layer'ı aktif
    public void EquipWeapon()
    {
        StartCoroutine(FadeLayerWeight(upperBodyLayerIndex, 1f, 0.3f));
    }

    public void UnequipWeapon()
    {
        StartCoroutine(FadeLayerWeight(upperBodyLayerIndex, 0f, 0.3f));
    }

    System.Collections.IEnumerator FadeLayerWeight(int layer, float target, float duration)
    {
        float start = animator.GetLayerWeight(layer);
        float elapsed = 0f;

        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            float weight = Mathf.Lerp(start, target, elapsed / duration);
            animator.SetLayerWeight(layer, weight);
            yield return null;
        }

        animator.SetLayerWeight(layer, target);
    }
}
```

---

## Root Motion

Animasyondaki hareket verisini GameObject'e uygular.

### Root Motion Ayarları

```csharp
// Animator'da Apply Root Motion:
// ✓ = Animasyondaki hareket uygulanır
// ✗ = Hareket script ile yapılır

// Rigidbody ile kullanım için:
// Animator.applyRootMotion = true
// Rigidbody.isKinematic = false
```

### OnAnimatorMove

```csharp
using UnityEngine;

public class RootMotionController : MonoBehaviour
{
    private Animator animator;
    private CharacterController controller;

    [SerializeField] private bool useRootMotion = true;

    void Start()
    {
        animator = GetComponent<Animator>();
        controller = GetComponent<CharacterController>();
    }

    // Root Motion'ı özelleştir
    void OnAnimatorMove()
    {
        if (useRootMotion)
        {
            // Animasyonun delta pozisyonunu uygula
            Vector3 deltaPosition = animator.deltaPosition;

            // Yerçekimi ekle
            deltaPosition.y -= 9.81f * Time.deltaTime;

            controller.Move(deltaPosition);

            // Dönüşü uygula
            transform.rotation *= animator.deltaRotation;
        }
    }

    // Root Motion'ı kapatıp manuel hareket
    void Update()
    {
        if (!useRootMotion)
        {
            // Kendi hareket kodun
        }
    }
}
```

### Root Motion vs Script Movement

```
┌─────────────────────────────────────────────────────────────┐
│            ROOT MOTION vs SCRIPT MOVEMENT                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Root Motion:                                                │
│   + Animasyon ile senkronize hareket                       │
│   + Gerçekçi ayak kaydırması önlenir                       │
│   - Daha az kontrol                                        │
│   - Animasyona bağımlı                                     │
│   Kullanım: Dövüş oyunları, cinematic                      │
│                                                             │
│ Script Movement:                                            │
│   + Tam kontrol                                            │
│   + Responsive (anında tepki)                              │
│   - Ayak kaydırması olabilir                               │
│   - Animasyon uyumu zor                                    │
│   Kullanım: Platformer, FPS                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Humanoid ve Avatar

İnsan karakterler için özel sistem.

### Humanoid Rig Ayarı

```
1. Model'i seç (FBX)
2. Inspector > Rig sekmesi
3. Animation Type: Humanoid
4. Avatar Definition: Create From This Model
5. Configure... ile kemik eşlemesini kontrol et
```

### Avatar Yapısı

```
┌─────────────────────────────────────────────────────────────┐
│                   HUMANOID AVATAR                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                        Head                                 │
│                          │                                  │
│                        Neck                                 │
│                          │                                  │
│    LeftArm ────── Spine ────── RightArm                    │
│        │            │            │                          │
│    LeftHand      Hips      RightHand                       │
│                    │                                        │
│            ┌───────┴───────┐                               │
│        LeftLeg          RightLeg                           │
│            │                │                               │
│        LeftFoot        RightFoot                           │
│                                                             │
│ Gerekli kemikler (yeşil): 15 adet                          │
│ Opsiyonel kemikler (gri): Parmaklar vb.                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Animation Retargeting

```csharp
// Humanoid avatarlar arasında animasyon paylaşımı
// Aynı animasyon farklı karakterlerde kullanılabilir

// Örnek:
// - "walk.anim" hem Player hem NPC'de çalışır
// - Kemik isimleri farklı olsa bile
```

---

## IK (Inverse Kinematics)

Uzuv hedefleme sistemi.

### IK Kullanım Alanları

```
- Ayakların zemine uyumu
- Ellerin kapı kolunu tutması
- Başın hedefe bakması
- Silahın hedefe nişan alması
```

### IK Ayarları

```
1. Animator Controller'da layer seç
2. Layer ayarlarında IK Pass'ı aktifleştir
```

### IK Script

```csharp
using UnityEngine;

public class IKController : MonoBehaviour
{
    private Animator animator;

    [Header("Look At")]
    [SerializeField] private Transform lookTarget;
    [SerializeField] private float lookWeight = 1f;

    [Header("Hand IK")]
    [SerializeField] private Transform rightHandTarget;
    [SerializeField] private Transform leftHandTarget;
    [SerializeField] private float handWeight = 1f;

    [Header("Foot IK")]
    [SerializeField] private bool enableFootIK = true;
    [SerializeField] private LayerMask groundLayers;
    [SerializeField] private float footOffset = 0.1f;

    void Start()
    {
        animator = GetComponent<Animator>();
    }

    // IK callback - Animator'da IK Pass aktif olmalı
    void OnAnimatorIK(int layerIndex)
    {
        if (animator == null) return;

        // ===== LOOK AT IK =====
        if (lookTarget != null)
        {
            animator.SetLookAtWeight(lookWeight);
            animator.SetLookAtPosition(lookTarget.position);
        }

        // ===== HAND IK =====
        if (rightHandTarget != null)
        {
            animator.SetIKPositionWeight(AvatarIKGoal.RightHand, handWeight);
            animator.SetIKRotationWeight(AvatarIKGoal.RightHand, handWeight);
            animator.SetIKPosition(AvatarIKGoal.RightHand, rightHandTarget.position);
            animator.SetIKRotation(AvatarIKGoal.RightHand, rightHandTarget.rotation);
        }

        if (leftHandTarget != null)
        {
            animator.SetIKPositionWeight(AvatarIKGoal.LeftHand, handWeight);
            animator.SetIKRotationWeight(AvatarIKGoal.LeftHand, handWeight);
            animator.SetIKPosition(AvatarIKGoal.LeftHand, leftHandTarget.position);
            animator.SetIKRotation(AvatarIKGoal.LeftHand, leftHandTarget.rotation);
        }

        // ===== FOOT IK =====
        if (enableFootIK)
        {
            UpdateFootIK(AvatarIKGoal.LeftFoot);
            UpdateFootIK(AvatarIKGoal.RightFoot);
        }
    }

    void UpdateFootIK(AvatarIKGoal foot)
    {
        // Ayağın mevcut pozisyonunu al
        Vector3 footPos = animator.GetIKPosition(foot);

        // Zemine raycast at
        RaycastHit hit;
        if (Physics.Raycast(footPos + Vector3.up, Vector3.down, out hit, 1.5f, groundLayers))
        {
            // Ayağı zemine yerleştir
            Vector3 targetPos = hit.point + Vector3.up * footOffset;
            animator.SetIKPositionWeight(foot, 1f);
            animator.SetIKPosition(foot, targetPos);

            // Ayağı zemine paralel döndür
            Quaternion targetRot = Quaternion.LookRotation(transform.forward, hit.normal);
            animator.SetIKRotationWeight(foot, 1f);
            animator.SetIKRotation(foot, targetRot);
        }
    }

    // Look at ağırlığını smooth değiştir
    public void SetLookTarget(Transform target, float weight)
    {
        lookTarget = target;
        StartCoroutine(SmoothLookWeight(weight));
    }

    System.Collections.IEnumerator SmoothLookWeight(float targetWeight)
    {
        while (Mathf.Abs(lookWeight - targetWeight) > 0.01f)
        {
            lookWeight = Mathf.Lerp(lookWeight, targetWeight, Time.deltaTime * 5f);
            yield return null;
        }
        lookWeight = targetWeight;
    }
}
```

### IK Hint

```csharp
// Dirsek ve diz yönü için hint kullanımı
void OnAnimatorIK(int layerIndex)
{
    // Sağ dirsek hint'i (dirsek nereye bakacak)
    animator.SetIKHintPositionWeight(AvatarIKHint.RightElbow, 1f);
    animator.SetIKHintPosition(AvatarIKHint.RightElbow, rightElbowHint.position);

    // Sol diz hint'i
    animator.SetIKHintPositionWeight(AvatarIKHint.LeftKnee, 1f);
    animator.SetIKHintPosition(AvatarIKHint.LeftKnee, leftKneeHint.position);
}
```

---

## Sub-State Machine

Karmaşık state'leri gruplamak için.

### Kullanım

```
1. Animator'da sağ tık
2. Create Sub-State Machine
3. Çift tıkla içine gir
4. State'leri ekle
5. (Up) Base Layer ile üst seviyeye dön
```

```
┌─────────────────────────────────────────────────────────────┐
│                  SUB-STATE MACHINE                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Base Layer:                                                 │
│   [Locomotion] ◄───► [Combat] ◄───► [Death]               │
│                                                             │
│ Locomotion (Sub-State):                                     │
│   [Idle] ◄─► [Walk] ◄─► [Run] ◄─► [Sprint]               │
│                                                             │
│ Combat (Sub-State):                                         │
│   [Ready] ─► [Attack1] ─► [Attack2] ─► [Attack3]          │
│      ▲                                   │                  │
│      └───────────────────────────────────┘                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Pratik Örnekler

### 1. Karakter Hareket Animasyonu

```csharp
using UnityEngine;

public class CharacterAnimator : MonoBehaviour
{
    private Animator animator;
    private CharacterController controller;

    private static readonly int SpeedHash = Animator.StringToHash("Speed");
    private static readonly int IsGroundedHash = Animator.StringToHash("IsGrounded");
    private static readonly int JumpHash = Animator.StringToHash("Jump");
    private static readonly int FallHash = Animator.StringToHash("IsFalling");

    [SerializeField] private float moveSpeed = 5f;
    [SerializeField] private float jumpForce = 8f;

    private Vector3 velocity;
    private bool isGrounded;

    void Start()
    {
        animator = GetComponent<Animator>();
        controller = GetComponent<CharacterController>();
    }

    void Update()
    {
        // Zemin kontrolü
        isGrounded = controller.isGrounded;
        animator.SetBool(IsGroundedHash, isGrounded);

        if (isGrounded && velocity.y < 0)
        {
            velocity.y = -2f;
        }

        // Hareket
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");
        Vector3 move = transform.right * h + transform.forward * v;

        controller.Move(move * moveSpeed * Time.deltaTime);

        // Hareket hızı animasyonu
        float speed = new Vector3(controller.velocity.x, 0, controller.velocity.z).magnitude;
        animator.SetFloat(SpeedHash, speed, 0.1f, Time.deltaTime);

        // Zıplama
        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            velocity.y = jumpForce;
            animator.SetTrigger(JumpHash);
        }

        // Yerçekimi
        velocity.y -= 20f * Time.deltaTime;
        controller.Move(velocity * Time.deltaTime);

        // Düşme kontrolü
        animator.SetBool(FallHash, !isGrounded && velocity.y < 0);
    }
}
```

### 2. Saldırı Combo Sistemi

```csharp
using UnityEngine;

public class ComboSystem : MonoBehaviour
{
    private Animator animator;

    private static readonly int AttackHash = Animator.StringToHash("Attack");
    private static readonly int ComboHash = Animator.StringToHash("ComboCount");

    private int comboCount = 0;
    private float comboTimer = 0f;
    private float comboWindow = 1f;
    private bool canCombo = false;

    void Start()
    {
        animator = GetComponent<Animator>();
    }

    void Update()
    {
        // Combo timer
        if (comboTimer > 0)
        {
            comboTimer -= Time.deltaTime;
            if (comboTimer <= 0)
            {
                ResetCombo();
            }
        }

        // Saldırı input
        if (Input.GetButtonDown("Fire1"))
        {
            Attack();
        }
    }

    void Attack()
    {
        if (canCombo || comboCount == 0)
        {
            comboCount++;
            animator.SetInteger(ComboHash, comboCount);
            animator.SetTrigger(AttackHash);
            comboTimer = comboWindow;
            canCombo = false;
        }
    }

    // Animation Event tarafından çağrılır
    public void EnableCombo()
    {
        canCombo = true;
    }

    public void DisableCombo()
    {
        canCombo = false;
    }

    public void ResetCombo()
    {
        comboCount = 0;
        animator.SetInteger(ComboHash, 0);
        canCombo = false;
    }

    // Animation Event - Combo sonu
    public void ComboFinished()
    {
        ResetCombo();
    }
}
```

### 3. Kapı Animasyonu

```csharp
using UnityEngine;

public class AnimatedDoor : MonoBehaviour
{
    private Animator animator;
    private bool isOpen = false;

    private static readonly int OpenHash = Animator.StringToHash("Open");

    void Start()
    {
        animator = GetComponent<Animator>();
    }

    public void ToggleDoor()
    {
        isOpen = !isOpen;
        animator.SetBool(OpenHash, isOpen);
    }

    public void OpenDoor()
    {
        if (!isOpen)
        {
            isOpen = true;
            animator.SetBool(OpenHash, true);
        }
    }

    public void CloseDoor()
    {
        if (isOpen)
        {
            isOpen = false;
            animator.SetBool(OpenHash, false);
        }
    }

    // Trigger ile otomatik açma
    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            OpenDoor();
        }
    }

    void OnTriggerExit(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            CloseDoor();
        }
    }
}
```

---

## Performans İpuçları

```
┌─────────────────────────────────────────────────────────────┐
│              PERFORMANS İPUÇLARI                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 1. Culling Mode ayarla                                     │
│    - Cull Update Transforms: Görünmezken bone güncelleme   │
│    - Cull Completely: Görünmezken tamamen durdur           │
│                                                             │
│ 2. Optimize Game Objects                                    │
│    - Model import'ta aktifleştir                           │
│    - Gereksiz transform hierarchy'si kaldırır             │
│                                                             │
│ 3. Parameter hash kullan                                    │
│    - Animator.StringToHash("Speed") bir kere çağır        │
│    - Her frame string kullanma                             │
│                                                             │
│ 4. Layer sayısını minimize et                              │
│    - Gereksiz layer performans düşürür                    │
│                                                             │
│ 5. Transition süreleri                                     │
│    - Çok kısa transition = sert geçiş                     │
│    - Çok uzun transition = CPU yükü                       │
│                                                             │
│ 6. Animation Compression                                    │
│    - Import ayarlarında sıkıştırma kullan                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Özet ve Kontrol Listesi

Bu derste öğrendiklerimiz:
- [x] Animation Clip oluşturma ve düzenleme
- [x] Animator Controller ve State Machine
- [x] Transitions ve conditions
- [x] Parameters (Float, Int, Bool, Trigger)
- [x] Blend Trees (1D ve 2D)
- [x] Animation Events
- [x] Animator Layers ve Avatar Mask
- [x] Root Motion
- [x] Humanoid Avatar ve retargeting
- [x] IK (Inverse Kinematics)
- [x] Sub-State Machines
- [x] Script ile animasyon kontrolü

---

## Alıştırmalar

1. **Idle-Walk-Run**: 1D Blend Tree ile hareket animasyonu
2. **Zıplama**: Trigger ile zıplama animasyonu
3. **Kapı**: Bool ile açılıp kapanan kapı
4. **Combo**: Saldırı combo sistemi
5. **Foot IK**: Eğimli zeminde ayak uyumu
6. **Look At**: Karakterin fareye bakması

---

## Sonraki Ders

11. Ders'te **Parçacık Sistemleri (Particle System)** konusunu işleyeceğiz. Ateş, duman, patlama ve büyü efektleri oluşturmayı öğreneceğiz.
