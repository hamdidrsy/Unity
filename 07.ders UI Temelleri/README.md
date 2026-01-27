# 7. Ders - UI Temelleri (User Interface)

## Giriş

Kullanıcı arayüzü (UI), oyuncunun oyunla etkileşim kurduğu görsel elemanlardır. Sağlık barları, skor göstergeleri, menüler, butonlar - bunların hepsi UI sisteminin parçasıdır.

Bu derste:
- Canvas sistemi ve render modları
- Temel UI elementleri (Text, Image, Button, Panel)
- RectTransform ve Anchor sistemi
- Layout grupları
- Event System ve etkileşimler
- TextMeshPro kullanımı
- Pratik örnekler (menü, HUD, popup)

konularını işleyeceğiz.

---

## Canvas Sistemi

Canvas, tüm UI elementlerinin yerleştirildiği ana konteynerdir. UI elemanları Canvas olmadan çalışmaz.

### Canvas Oluşturma

```
Hierarchy > Sağ Tık > UI > Canvas
```

Canvas oluşturduğunuzda otomatik olarak **EventSystem** objesi de eklenir.

### Canvas Bileşenleri

```
┌─────────────────────────────────────────────────────────────┐
│ Canvas GameObject                                           │
├─────────────────────────────────────────────────────────────┤
│ ├── Canvas (Component)         → Render modu ayarları       │
│ ├── Canvas Scaler              → Ölçeklendirme ayarları     │
│ ├── Graphic Raycaster          → UI tıklama algılama        │
│ └── RectTransform              → Pozisyon ve boyut          │
└─────────────────────────────────────────────────────────────┘
```

### Render Modları

#### 1. Screen Space - Overlay (Varsayılan)

```csharp
// UI her zaman ekranın üstünde, kameradan bağımsız
// En yaygın kullanım: HUD, menüler, popuplar
```

Özellikleri:
- Her zaman en üstte render edilir
- Kamera pozisyonundan etkilenmez
- Performans açısından en verimli
- Çoğu oyun için ideal seçim

#### 2. Screen Space - Camera

```csharp
// UI belirli bir kameraya bağlı
// Kamera ile birlikte hareket eder
// Post-processing efektleri UI'ı etkiler
```

Ayarlar:
- **Render Camera**: UI'ı render edecek kamera
- **Plane Distance**: Kameradan uzaklık
- **Sorting Layer**: Render sırası

Kullanım alanları:
- UI'a shader efektleri uygulamak
- Kamera efektlerinin UI'ı etkilemesini istediğinizde

#### 3. World Space

```csharp
// UI 3D dünyada bir obje gibi davranır
// Oyun dünyasının parçası olur
```

Kullanım alanları:
- NPC üzerindeki isim etiketleri
- 3D menüler (VR/AR)
- Oyun içi ekranlar, tabelalar
- Düşman sağlık barları (head-up)

```csharp
// World Space Canvas ayarları
Canvas canvas = GetComponent<Canvas>();
canvas.renderMode = RenderMode.WorldSpace;
canvas.worldCamera = Camera.main;
```

---

## Canvas Scaler

Farklı ekran boyutlarında UI'ın nasıl ölçekleneceğini belirler.

### UI Scale Mode Seçenekleri

#### 1. Constant Pixel Size

```
UI elemanları her ekranda aynı piksel boyutunda kalır
- Küçük ekranda: UI büyük görünür
- Büyük ekranda: UI küçük görünür
```

Kullanım: Piksel-perfect 2D oyunlar

#### 2. Scale With Screen Size (Önerilen)

```
UI ekran boyutuyla orantılı olarak ölçeklenir
```

Ayarlar:
- **Reference Resolution**: Tasarım çözünürlüğü (örn: 1920x1080)
- **Screen Match Mode**: Genişlik mi, yükseklik mi baz alınsın
  - Match Width Or Height: Slider ile ayarla
  - Expand: Referansı aşmasına izin ver
  - Shrink: Referansı aşmamasını sağla

```
Match Width Or Height için slider değerleri:
0 = Sadece genişliğe göre (yatay oyunlar)
1 = Sadece yüksekliğe göre (dikey oyunlar)
0.5 = Her ikisinin ortası (çoğu durum için ideal)
```

#### 3. Constant Physical Size

```
Fiziksel boyut (cm/inch) sabit kalır
Cihazın DPI değerine göre ölçeklenir
```

Kullanım: Dokunmatik cihazlar, gerçek boyut önemli olduğunda

---

## RectTransform

UI elemanlarının pozisyon, boyut ve hizalama ayarlarını kontrol eder. Normal Transform'un UI versiyonudur.

### Temel Özellikler

```csharp
RectTransform rectTransform = GetComponent<RectTransform>();

// Pozisyon
rectTransform.anchoredPosition = new Vector2(100, 50);

// Boyut
rectTransform.sizeDelta = new Vector2(200, 100);

// Pivot (dönüş merkezi)
rectTransform.pivot = new Vector2(0.5f, 0.5f); // Merkez

// Anchor pozisyonları
rectTransform.anchorMin = new Vector2(0, 0);    // Sol alt
rectTransform.anchorMax = new Vector2(1, 1);    // Sağ üst
```

### Anchor Sistemi

Anchor, UI elemanının parent içinde nasıl konumlandırılacağını belirler.

```
┌─────────────────────────────────────────────────────────────┐
│                    ANCHOR PRESETS                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌───┐ ┌───┐ ┌───┐      ┌───┐ ┌───┐ ┌───┐                │
│   │TL │ │TC │ │TR │      │←→│ │←→│ │←→│  (stretch)        │
│   └───┘ └───┘ └───┘      └───┘ └───┘ └───┘                │
│                                                             │
│   ┌───┐ ┌───┐ ┌───┐      ┌───┐ ┌───┐ ┌───┐                │
│   │ML │ │MC │ │MR │      │↕ │ │↕ │ │↕ │  (stretch)        │
│   └───┘ └───┘ └───┘      └───┘ └───┘ └───┘                │
│                                                             │
│   ┌───┐ ┌───┐ ┌───┐      ┌─────────────────┐               │
│   │BL │ │BC │ │BR │      │   FULL STRETCH  │               │
│   └───┘ └───┘ └───┘      └─────────────────┘               │
│                                                             │
│   T=Top, M=Middle, B=Bottom                                │
│   L=Left, C=Center, R=Right                                │
└─────────────────────────────────────────────────────────────┘
```

### Yaygın Anchor Kullanımları

```csharp
// Sol üst köşeye sabitle (skor göstergesi)
rectTransform.anchorMin = new Vector2(0, 1);
rectTransform.anchorMax = new Vector2(0, 1);
rectTransform.pivot = new Vector2(0, 1);

// Sağ alt köşeye sabitle (mini harita)
rectTransform.anchorMin = new Vector2(1, 0);
rectTransform.anchorMax = new Vector2(1, 0);
rectTransform.pivot = new Vector2(1, 0);

// Alt orta (sağlık barı)
rectTransform.anchorMin = new Vector2(0.5f, 0);
rectTransform.anchorMax = new Vector2(0.5f, 0);
rectTransform.pivot = new Vector2(0.5f, 0);

// Tam ekran stretch (arka plan)
rectTransform.anchorMin = Vector2.zero;
rectTransform.anchorMax = Vector2.one;
```

### Pivot Noktası

Pivot, elemanın dönüş ve ölçekleme merkezini belirler.

```
(0,1)───────(0.5,1)───────(1,1)
  │                         │
  │                         │
(0,0.5)────(0.5,0.5)────(1,0.5)
  │                         │
  │                         │
(0,0)───────(0.5,0)───────(1,0)
```

---

## Temel UI Elementleri

### Text (Legacy)

```
Hierarchy > Sağ Tık > UI > Text
```

```csharp
using UnityEngine;
using UnityEngine.UI;

public class TextExample : MonoBehaviour
{
    [SerializeField] private Text scoreText;

    private int score = 0;

    void Start()
    {
        UpdateScore();
    }

    public void AddScore(int amount)
    {
        score += amount;
        UpdateScore();
    }

    void UpdateScore()
    {
        scoreText.text = "Skor: " + score;
        // veya
        scoreText.text = $"Skor: {score}";
    }
}
```

Text özellikleri:
- **Font**: Yazı tipi
- **Font Size**: Yazı boyutu
- **Line Spacing**: Satır aralığı
- **Rich Text**: HTML benzeri etiketler (`<b>`, `<i>`, `<color>`)
- **Alignment**: Hizalama (sol, orta, sağ / üst, orta, alt)
- **Horizontal/Vertical Overflow**: Taşma davranışı

### Rich Text Örnekleri

```csharp
// Kalın
text.text = "<b>Kalın Yazı</b>";

// İtalik
text.text = "<i>İtalik Yazı</i>";

// Renkli
text.text = "<color=red>Kırmızı</color> ve <color=#00FF00>Yeşil</color>";

// Boyut
text.text = "<size=24>Büyük</size> ve <size=12>Küçük</size>";

// Kombinasyon
text.text = "<b><color=yellow>UYARI!</color></b>";
```

---

### TextMeshPro (Önerilen)

TextMeshPro, Unity'nin gelişmiş metin render sistemidir. Daha kaliteli ve daha fazla özellik sunar.

#### Kurulum

```
Window > Package Manager > Unity Registry > TextMeshPro > Install
```

İlk kullanımda "Import TMP Essentials" diyalog kutusunu onaylayın.

#### Kullanım

```
Hierarchy > Sağ Tık > UI > Text - TextMeshPro
```

```csharp
using UnityEngine;
using TMPro;  // TextMeshPro namespace

public class TMPExample : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI scoreText;
    // Not: UI için TextMeshProUGUI, 3D için TextMeshPro

    private int score = 0;

    public void AddScore(int amount)
    {
        score += amount;
        scoreText.text = $"Skor: {score}";
    }
}
```

#### TextMeshPro Avantajları

| Özellik | Legacy Text | TextMeshPro |
|---------|-------------|-------------|
| Render Kalitesi | Bulanık olabilir | Her zaman keskin |
| Outline | Yok | Var |
| Glow | Yok | Var |
| Gradient | Yok | Var |
| Font Asset | TTF direkt | Font Asset gerekli |
| Performans | Orta | Daha iyi |

#### TextMeshPro Rich Text

```csharp
// Sprite ekleme
text.text = "Kalp: <sprite=0>";

// Gradient
text.text = "<gradient=\"Yellow to Red\">Gradient Yazı</gradient>";

// Link
text.text = "<link=\"https://unity.com\">Tıkla</link>";

// Monospace
text.text = "<mspace=0.5em>Sabit Genişlik</mspace>";
```

---

### Image

```
Hierarchy > Sağ Tık > UI > Image
```

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ImageExample : MonoBehaviour
{
    [SerializeField] private Image iconImage;
    [SerializeField] private Sprite[] icons;

    public void ChangeIcon(int index)
    {
        iconImage.sprite = icons[index];
    }

    public void SetTransparency(float alpha)
    {
        Color color = iconImage.color;
        color.a = alpha;
        iconImage.color = color;
    }

    public void FillAmount(float amount)
    {
        // Image Type = Filled olmalı
        iconImage.fillAmount = amount; // 0-1 arası
    }
}
```

#### Image Types

```
┌─────────────────────────────────────────────────────────────┐
│                    IMAGE TYPES                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Simple: Sprite olduğu gibi gösterilir                      │
│                                                             │
│ Sliced: 9-slice sprite için (kenarlar korunur)             │
│         Panel, buton arka planları için ideal              │
│                                                             │
│ Tiled: Sprite tekrarlanır                                  │
│        Desen arka planları için                            │
│                                                             │
│ Filled: Dolum animasyonları için                           │
│         Cooldown, sağlık barı, loading                     │
│         Fill Method: Horizontal, Vertical, Radial          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Filled Image Örneği (Cooldown)

```csharp
public class CooldownUI : MonoBehaviour
{
    [SerializeField] private Image cooldownImage;
    [SerializeField] private float cooldownTime = 3f;

    private float currentCooldown;
    private bool isOnCooldown;

    public void StartCooldown()
    {
        currentCooldown = cooldownTime;
        isOnCooldown = true;
    }

    void Update()
    {
        if (isOnCooldown)
        {
            currentCooldown -= Time.deltaTime;
            cooldownImage.fillAmount = currentCooldown / cooldownTime;

            if (currentCooldown <= 0)
            {
                isOnCooldown = false;
                cooldownImage.fillAmount = 0;
            }
        }
    }
}
```

---

### Raw Image

Texture2D göstermek için kullanılır. Render Texture, video veya dinamik görseller için idealdir.

```csharp
using UnityEngine;
using UnityEngine.UI;

public class RawImageExample : MonoBehaviour
{
    [SerializeField] private RawImage renderDisplay;
    [SerializeField] private RenderTexture renderTexture;

    void Start()
    {
        // Render Texture göster (mini harita, güvenlik kamerası vb.)
        renderDisplay.texture = renderTexture;
    }

    // UV Rect ile texture kaydırma (animasyon)
    public void ScrollTexture(float speed)
    {
        Rect uvRect = renderDisplay.uvRect;
        uvRect.x += speed * Time.deltaTime;
        renderDisplay.uvRect = uvRect;
    }
}
```

---

### Button

```
Hierarchy > Sağ Tık > UI > Button
```

#### Inspector'dan Event Bağlama

1. Button component'inde "On Click ()" bölümünü bul
2. "+" butonuna tıkla
3. Script içeren GameObject'i sürükle
4. Fonksiyonu seç

#### Script ile Event Bağlama

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ButtonExample : MonoBehaviour
{
    [SerializeField] private Button playButton;
    [SerializeField] private Button quitButton;

    void Start()
    {
        // Lambda ile
        playButton.onClick.AddListener(() => {
            Debug.Log("Play'e tıklandı!");
            StartGame();
        });

        // Metod referansı ile
        quitButton.onClick.AddListener(QuitGame);
    }

    void OnDestroy()
    {
        // Listener'ları temizle
        playButton.onClick.RemoveAllListeners();
        quitButton.onClick.RemoveAllListeners();
    }

    void StartGame()
    {
        Debug.Log("Oyun başlıyor...");
    }

    void QuitGame()
    {
        Debug.Log("Oyundan çıkılıyor...");
        Application.Quit();
    }
}
```

#### Button Transition Modları

```
┌─────────────────────────────────────────────────────────────┐
│                 BUTTON TRANSITIONS                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ None: Görsel değişiklik yok                                │
│                                                             │
│ Color Tint: Renk değişimi                                  │
│   - Normal: Varsayılan renk                                │
│   - Highlighted: Üzerine gelince                           │
│   - Pressed: Basılınca                                     │
│   - Selected: Seçilince (gamepad/klavye)                   │
│   - Disabled: Devre dışı                                   │
│                                                             │
│ Sprite Swap: Farklı sprite göster                          │
│   Her durum için ayrı sprite atanır                        │
│                                                             │
│ Animation: Animator ile geçiş                              │
│   En esnek ama en karmaşık                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Toggle (Onay Kutusu)

```
Hierarchy > Sağ Tık > UI > Toggle
```

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ToggleExample : MonoBehaviour
{
    [SerializeField] private Toggle musicToggle;
    [SerializeField] private Toggle sfxToggle;

    void Start()
    {
        // Başlangıç değerlerini ayarla
        musicToggle.isOn = PlayerPrefs.GetInt("Music", 1) == 1;
        sfxToggle.isOn = PlayerPrefs.GetInt("SFX", 1) == 1;

        // Event listener ekle
        musicToggle.onValueChanged.AddListener(OnMusicToggle);
        sfxToggle.onValueChanged.AddListener(OnSFXToggle);
    }

    void OnMusicToggle(bool isOn)
    {
        PlayerPrefs.SetInt("Music", isOn ? 1 : 0);
        // AudioManager.Instance.SetMusicEnabled(isOn);
        Debug.Log($"Müzik: {(isOn ? "Açık" : "Kapalı")}");
    }

    void OnSFXToggle(bool isOn)
    {
        PlayerPrefs.SetInt("SFX", isOn ? 1 : 0);
        Debug.Log($"Ses Efektleri: {(isOn ? "Açık" : "Kapalı")}");
    }
}
```

### Toggle Group

Radyo butonları gibi davranış için:

```csharp
// Hierarchy:
// ToggleGroup (boş GameObject + ToggleGroup component)
//   ├── Toggle1 (Group = ToggleGroup)
//   ├── Toggle2 (Group = ToggleGroup)
//   └── Toggle3 (Group = ToggleGroup)

using UnityEngine;
using UnityEngine.UI;

public class DifficultySelector : MonoBehaviour
{
    [SerializeField] private ToggleGroup difficultyGroup;
    [SerializeField] private Toggle easyToggle;
    [SerializeField] private Toggle normalToggle;
    [SerializeField] private Toggle hardToggle;

    void Start()
    {
        easyToggle.onValueChanged.AddListener((isOn) => {
            if (isOn) SetDifficulty("Easy");
        });
        normalToggle.onValueChanged.AddListener((isOn) => {
            if (isOn) SetDifficulty("Normal");
        });
        hardToggle.onValueChanged.AddListener((isOn) => {
            if (isOn) SetDifficulty("Hard");
        });
    }

    void SetDifficulty(string difficulty)
    {
        Debug.Log($"Zorluk: {difficulty}");
    }
}
```

---

### Slider

```
Hierarchy > Sağ Tık > UI > Slider
```

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class SliderExample : MonoBehaviour
{
    [SerializeField] private Slider volumeSlider;
    [SerializeField] private TextMeshProUGUI volumeText;

    void Start()
    {
        // Slider ayarları
        volumeSlider.minValue = 0f;
        volumeSlider.maxValue = 1f;
        volumeSlider.value = PlayerPrefs.GetFloat("Volume", 0.8f);

        // Listener ekle
        volumeSlider.onValueChanged.AddListener(OnVolumeChanged);

        // İlk değeri göster
        UpdateVolumeText(volumeSlider.value);
    }

    void OnVolumeChanged(float value)
    {
        UpdateVolumeText(value);
        PlayerPrefs.SetFloat("Volume", value);
        AudioListener.volume = value;
    }

    void UpdateVolumeText(float value)
    {
        int percentage = Mathf.RoundToInt(value * 100);
        volumeText.text = $"Ses: {percentage}%";
    }
}
```

### Slider ile Sağlık Barı

```csharp
public class HealthBar : MonoBehaviour
{
    [SerializeField] private Slider healthSlider;
    [SerializeField] private Image fillImage;
    [SerializeField] private Gradient colorGradient;

    private float maxHealth = 100f;
    private float currentHealth;

    void Start()
    {
        currentHealth = maxHealth;
        UpdateHealthBar();
    }

    public void TakeDamage(float damage)
    {
        currentHealth = Mathf.Max(0, currentHealth - damage);
        UpdateHealthBar();
    }

    public void Heal(float amount)
    {
        currentHealth = Mathf.Min(maxHealth, currentHealth + amount);
        UpdateHealthBar();
    }

    void UpdateHealthBar()
    {
        float healthPercent = currentHealth / maxHealth;
        healthSlider.value = healthPercent;

        // Gradient ile renk değişimi (yeşil → sarı → kırmızı)
        fillImage.color = colorGradient.Evaluate(healthPercent);
    }
}
```

---

### Input Field

```
Hierarchy > Sağ Tık > UI > Input Field - TextMeshPro
```

```csharp
using UnityEngine;
using TMPro;

public class InputFieldExample : MonoBehaviour
{
    [SerializeField] private TMP_InputField nameInput;
    [SerializeField] private TMP_InputField passwordInput;
    [SerializeField] private TextMeshProUGUI feedbackText;

    void Start()
    {
        // Placeholder ayarla
        nameInput.placeholder.GetComponent<TextMeshProUGUI>().text = "Kullanıcı adı...";

        // Content Type ayarla
        passwordInput.contentType = TMP_InputField.ContentType.Password;

        // Event listener'lar
        nameInput.onEndEdit.AddListener(OnNameEntered);
        nameInput.onValueChanged.AddListener(OnNameChanged);
    }

    void OnNameEntered(string name)
    {
        Debug.Log($"İsim girildi: {name}");
    }

    void OnNameChanged(string name)
    {
        // Gerçek zamanlı karakter sayısı
        feedbackText.text = $"{name.Length}/20 karakter";
    }

    public void OnLoginButtonClick()
    {
        string name = nameInput.text;
        string password = passwordInput.text;

        if (string.IsNullOrEmpty(name) || string.IsNullOrEmpty(password))
        {
            feedbackText.text = "Lütfen tüm alanları doldurun!";
            feedbackText.color = Color.red;
            return;
        }

        Debug.Log($"Giriş deneniyor: {name}");
    }
}
```

#### Input Field Content Types

```
Standard         → Normal metin
Autocorrected    → Otomatik düzeltme
Integer Number   → Sadece tam sayı
Decimal Number   → Ondalıklı sayı
Alphanumeric     → Harf ve rakam
Name             → İsim formatı
Email Address    → E-posta formatı
Password         → Gizli karakterler
Pin              → PIN kodu
Custom           → Özel filtre
```

---

### Dropdown

```
Hierarchy > Sağ Tık > UI > Dropdown - TextMeshPro
```

```csharp
using UnityEngine;
using TMPro;
using System.Collections.Generic;

public class DropdownExample : MonoBehaviour
{
    [SerializeField] private TMP_Dropdown resolutionDropdown;
    [SerializeField] private TMP_Dropdown qualityDropdown;

    private Resolution[] resolutions;

    void Start()
    {
        SetupResolutionDropdown();
        SetupQualityDropdown();
    }

    void SetupResolutionDropdown()
    {
        resolutions = Screen.resolutions;
        resolutionDropdown.ClearOptions();

        List<string> options = new List<string>();
        int currentResolutionIndex = 0;

        for (int i = 0; i < resolutions.Length; i++)
        {
            string option = $"{resolutions[i].width} x {resolutions[i].height}";
            options.Add(option);

            if (resolutions[i].width == Screen.currentResolution.width &&
                resolutions[i].height == Screen.currentResolution.height)
            {
                currentResolutionIndex = i;
            }
        }

        resolutionDropdown.AddOptions(options);
        resolutionDropdown.value = currentResolutionIndex;
        resolutionDropdown.RefreshShownValue();

        resolutionDropdown.onValueChanged.AddListener(SetResolution);
    }

    void SetupQualityDropdown()
    {
        qualityDropdown.ClearOptions();
        qualityDropdown.AddOptions(new List<string>(QualitySettings.names));
        qualityDropdown.value = QualitySettings.GetQualityLevel();
        qualityDropdown.RefreshShownValue();

        qualityDropdown.onValueChanged.AddListener(SetQuality);
    }

    void SetResolution(int index)
    {
        Resolution res = resolutions[index];
        Screen.SetResolution(res.width, res.height, Screen.fullScreen);
    }

    void SetQuality(int index)
    {
        QualitySettings.SetQualityLevel(index);
    }
}
```

---

### Scrollbar ve Scroll View

```
Hierarchy > Sağ Tık > UI > Scroll View
```

Scroll View yapısı:
```
ScrollView
├── Viewport (Mask component)
│   └── Content (Gerçek içerik)
├── Scrollbar Horizontal
└── Scrollbar Vertical
```

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ScrollViewExample : MonoBehaviour
{
    [SerializeField] private ScrollRect scrollRect;
    [SerializeField] private RectTransform content;
    [SerializeField] private GameObject itemPrefab;

    public void PopulateList(string[] items)
    {
        // Mevcut içeriği temizle
        foreach (Transform child in content)
        {
            Destroy(child.gameObject);
        }

        // Yeni itemları ekle
        foreach (string item in items)
        {
            GameObject newItem = Instantiate(itemPrefab, content);
            newItem.GetComponentInChildren<TextMeshProUGUI>().text = item;
        }
    }

    public void ScrollToTop()
    {
        scrollRect.normalizedPosition = new Vector2(0, 1);
    }

    public void ScrollToBottom()
    {
        scrollRect.normalizedPosition = new Vector2(0, 0);
    }
}
```

---

## Layout Grupları

Otomatik düzenleme için layout component'leri kullanılır.

### Horizontal Layout Group

Çocuk elemanları yatay sıralar.

```
┌──────────────────────────────────────────┐
│ [Item1] [Item2] [Item3] [Item4]          │
└──────────────────────────────────────────┘
```

### Vertical Layout Group

Çocuk elemanları dikey sıralar.

```
┌─────────────┐
│ [Item1]     │
│ [Item2]     │
│ [Item3]     │
│ [Item4]     │
└─────────────┘
```

### Grid Layout Group

Çocuk elemanları ızgara şeklinde sıralar.

```
┌──────────────────────┐
│ [1] [2] [3]         │
│ [4] [5] [6]         │
│ [7] [8] [9]         │
└──────────────────────┘
```

### Layout Ayarları

```csharp
// Horizontal/Vertical Layout Group
Padding             // Kenar boşlukları (left, right, top, bottom)
Spacing             // Elemanlar arası boşluk
Child Alignment     // Hizalama
Child Controls Size // Eleman boyutunu kontrol et
Child Force Expand  // Boşluğu paylaştır

// Grid Layout Group
Cell Size          // Hücre boyutu
Spacing            // Yatay ve dikey boşluk
Start Corner       // Başlangıç köşesi
Start Axis         // Başlangıç ekseni
Constraint         // Sınırlama (satır/sütun sayısı)
```

### Content Size Fitter

İçeriğe göre boyut ayarlar.

```csharp
// Text içeriğine göre boyut
// Text GameObject'ine Content Size Fitter ekle:
Horizontal Fit: Preferred Size
Vertical Fit: Preferred Size
```

### Layout Element

Elemanın layout içindeki davranışını özelleştirir.

```csharp
// Minimum ve tercih edilen boyutlar
Min Width, Min Height
Preferred Width, Preferred Height
Flexible Width, Flexible Height
```

---

## Event System

UI etkileşimlerini yöneten sistemdir.

### Event System Bileşenleri

```
EventSystem GameObject
├── EventSystem (Component)      → Ana yönetici
└── Standalone Input Module      → Input işleme
    (veya Input System UI Input Module)
```

### Pointer Events

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class PointerEventsExample : MonoBehaviour,
    IPointerEnterHandler,
    IPointerExitHandler,
    IPointerClickHandler,
    IPointerDownHandler,
    IPointerUpHandler,
    IBeginDragHandler,
    IDragHandler,
    IEndDragHandler
{
    public void OnPointerEnter(PointerEventData eventData)
    {
        Debug.Log("Mouse üzerine geldi");
    }

    public void OnPointerExit(PointerEventData eventData)
    {
        Debug.Log("Mouse ayrıldı");
    }

    public void OnPointerClick(PointerEventData eventData)
    {
        Debug.Log("Tıklandı");
    }

    public void OnPointerDown(PointerEventData eventData)
    {
        Debug.Log("Basıldı");
    }

    public void OnPointerUp(PointerEventData eventData)
    {
        Debug.Log("Bırakıldı");
    }

    public void OnBeginDrag(PointerEventData eventData)
    {
        Debug.Log("Sürükleme başladı");
    }

    public void OnDrag(PointerEventData eventData)
    {
        // Sürüklerken pozisyonu güncelle
        transform.position = eventData.position;
    }

    public void OnEndDrag(PointerEventData eventData)
    {
        Debug.Log("Sürükleme bitti");
    }
}
```

### Drag & Drop Örneği

```csharp
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;

public class DraggableItem : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    private RectTransform rectTransform;
    private CanvasGroup canvasGroup;
    private Transform originalParent;
    private Vector2 originalPosition;

    void Awake()
    {
        rectTransform = GetComponent<RectTransform>();
        canvasGroup = GetComponent<CanvasGroup>();
    }

    public void OnBeginDrag(PointerEventData eventData)
    {
        originalParent = transform.parent;
        originalPosition = rectTransform.anchoredPosition;

        // En üste taşı
        transform.SetParent(transform.root);

        // Raycast'i devre dışı bırak (altındakilere ulaşabilmek için)
        canvasGroup.blocksRaycasts = false;
        canvasGroup.alpha = 0.7f;
    }

    public void OnDrag(PointerEventData eventData)
    {
        rectTransform.anchoredPosition += eventData.delta;
    }

    public void OnEndDrag(PointerEventData eventData)
    {
        canvasGroup.blocksRaycasts = true;
        canvasGroup.alpha = 1f;

        // Eğer geçerli bir slot'a bırakılmadıysa eski yerine dön
        if (transform.parent == transform.root)
        {
            transform.SetParent(originalParent);
            rectTransform.anchoredPosition = originalPosition;
        }
    }
}
```

---

## Pratik Uygulamalar

### 1. Ana Menü

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class MainMenu : MonoBehaviour
{
    [SerializeField] private GameObject mainPanel;
    [SerializeField] private GameObject settingsPanel;
    [SerializeField] private GameObject creditsPanel;

    void Start()
    {
        ShowMainPanel();
    }

    public void PlayGame()
    {
        SceneManager.LoadScene("GameScene");
    }

    public void ShowSettings()
    {
        mainPanel.SetActive(false);
        settingsPanel.SetActive(true);
    }

    public void ShowCredits()
    {
        mainPanel.SetActive(false);
        creditsPanel.SetActive(true);
    }

    public void ShowMainPanel()
    {
        mainPanel.SetActive(true);
        settingsPanel.SetActive(false);
        creditsPanel.SetActive(false);
    }

    public void QuitGame()
    {
        #if UNITY_EDITOR
            UnityEditor.EditorApplication.isPlaying = false;
        #else
            Application.Quit();
        #endif
    }
}
```

### 2. Pause Menüsü

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class PauseMenu : MonoBehaviour
{
    [SerializeField] private GameObject pausePanel;

    private bool isPaused = false;

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Escape))
        {
            if (isPaused)
                Resume();
            else
                Pause();
        }
    }

    public void Pause()
    {
        pausePanel.SetActive(true);
        Time.timeScale = 0f;
        isPaused = true;

        // Mouse'u göster
        Cursor.lockState = CursorLockMode.None;
        Cursor.visible = true;
    }

    public void Resume()
    {
        pausePanel.SetActive(false);
        Time.timeScale = 1f;
        isPaused = false;

        // Mouse'u kilitle (FPS oyunu için)
        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;
    }

    public void RestartLevel()
    {
        Time.timeScale = 1f;
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
    }

    public void QuitToMainMenu()
    {
        Time.timeScale = 1f;
        SceneManager.LoadScene("MainMenu");
    }
}
```

### 3. HUD (Heads-Up Display)

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class GameHUD : MonoBehaviour
{
    [Header("Sağlık")]
    [SerializeField] private Slider healthBar;
    [SerializeField] private TextMeshProUGUI healthText;
    [SerializeField] private Image healthFill;
    [SerializeField] private Gradient healthGradient;

    [Header("Skor")]
    [SerializeField] private TextMeshProUGUI scoreText;

    [Header("Mermi")]
    [SerializeField] private TextMeshProUGUI ammoText;

    [Header("Minimap")]
    [SerializeField] private RawImage minimapImage;

    private int currentScore = 0;

    public void UpdateHealth(float current, float max)
    {
        float percent = current / max;
        healthBar.value = percent;
        healthText.text = $"{Mathf.CeilToInt(current)}/{max}";
        healthFill.color = healthGradient.Evaluate(percent);
    }

    public void AddScore(int amount)
    {
        currentScore += amount;
        scoreText.text = $"Skor: {currentScore:N0}";
    }

    public void UpdateAmmo(int current, int max)
    {
        ammoText.text = $"{current} / {max}";

        // Düşük mermi uyarısı
        if (current <= 5)
        {
            ammoText.color = Color.red;
        }
        else
        {
            ammoText.color = Color.white;
        }
    }
}
```

### 4. Popup / Dialog Sistemi

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;
using System;

public class DialogPopup : MonoBehaviour
{
    [SerializeField] private GameObject popupPanel;
    [SerializeField] private TextMeshProUGUI titleText;
    [SerializeField] private TextMeshProUGUI messageText;
    [SerializeField] private Button confirmButton;
    [SerializeField] private Button cancelButton;
    [SerializeField] private TextMeshProUGUI confirmButtonText;
    [SerializeField] private TextMeshProUGUI cancelButtonText;

    private Action onConfirm;
    private Action onCancel;

    public void ShowDialog(string title, string message,
        string confirmText = "Evet", string cancelText = "Hayır",
        Action onConfirmAction = null, Action onCancelAction = null)
    {
        titleText.text = title;
        messageText.text = message;
        confirmButtonText.text = confirmText;
        cancelButtonText.text = cancelText;

        onConfirm = onConfirmAction;
        onCancel = onCancelAction;

        popupPanel.SetActive(true);
    }

    public void ShowAlert(string title, string message, string buttonText = "Tamam")
    {
        titleText.text = title;
        messageText.text = message;
        confirmButtonText.text = buttonText;

        cancelButton.gameObject.SetActive(false);
        popupPanel.SetActive(true);
    }

    public void OnConfirmClicked()
    {
        onConfirm?.Invoke();
        CloseDialog();
    }

    public void OnCancelClicked()
    {
        onCancel?.Invoke();
        CloseDialog();
    }

    void CloseDialog()
    {
        popupPanel.SetActive(false);
        cancelButton.gameObject.SetActive(true);
        onConfirm = null;
        onCancel = null;
    }
}

// Kullanım:
// dialogPopup.ShowDialog(
//     "Çıkış",
//     "Oyundan çıkmak istediğinize emin misiniz?",
//     "Evet", "Hayır",
//     () => Application.Quit(),
//     null
// );
```

### 5. Loading Screen

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;
using TMPro;
using System.Collections;

public class LoadingScreen : MonoBehaviour
{
    [SerializeField] private GameObject loadingPanel;
    [SerializeField] private Slider progressBar;
    [SerializeField] private TextMeshProUGUI progressText;
    [SerializeField] private TextMeshProUGUI tipText;

    [SerializeField] private string[] tips;

    public void LoadScene(string sceneName)
    {
        StartCoroutine(LoadSceneAsync(sceneName));
    }

    IEnumerator LoadSceneAsync(string sceneName)
    {
        loadingPanel.SetActive(true);

        // Rastgele ipucu göster
        if (tips.Length > 0)
        {
            tipText.text = tips[Random.Range(0, tips.Length)];
        }

        AsyncOperation operation = SceneManager.LoadSceneAsync(sceneName);
        operation.allowSceneActivation = false;

        while (!operation.isDone)
        {
            // İlerleme 0-0.9 arası (0.9'da bekler)
            float progress = Mathf.Clamp01(operation.progress / 0.9f);

            progressBar.value = progress;
            progressText.text = $"{Mathf.RoundToInt(progress * 100)}%";

            // Yükleme tamamlandığında
            if (operation.progress >= 0.9f)
            {
                progressText.text = "Devam etmek için herhangi bir tuşa basın";

                if (Input.anyKeyDown)
                {
                    operation.allowSceneActivation = true;
                }
            }

            yield return null;
        }
    }
}
```

---

## UI Animasyonları

### DOTween ile UI Animasyonu (Önerilen)

DOTween, Unity için popüler bir tween kütüphanesidir.

```
Window > Package Manager > + > Add package from git URL
https://github.com/Demigiant/dotween.git
```

```csharp
using UnityEngine;
using UnityEngine.UI;
using DG.Tweening;

public class UIAnimations : MonoBehaviour
{
    [SerializeField] private RectTransform panel;
    [SerializeField] private CanvasGroup canvasGroup;
    [SerializeField] private Button button;

    public void FadeIn(float duration = 0.5f)
    {
        canvasGroup.alpha = 0;
        canvasGroup.DOFade(1, duration);
    }

    public void FadeOut(float duration = 0.5f)
    {
        canvasGroup.DOFade(0, duration)
            .OnComplete(() => gameObject.SetActive(false));
    }

    public void SlideIn(float duration = 0.5f)
    {
        panel.anchoredPosition = new Vector2(-Screen.width, 0);
        panel.DOAnchorPos(Vector2.zero, duration).SetEase(Ease.OutBack);
    }

    public void ScalePopup(float duration = 0.3f)
    {
        panel.localScale = Vector3.zero;
        panel.DOScale(Vector3.one, duration).SetEase(Ease.OutBack);
    }

    public void ButtonPunch()
    {
        button.transform.DOPunchScale(Vector3.one * 0.1f, 0.2f);
    }
}
```

### Animator ile UI Animasyonu

1. UI elemanına Animator component ekle
2. Animator Controller oluştur
3. Animation clip'leri oluştur
4. Trigger veya bool parametreleri ile kontrol et

```csharp
using UnityEngine;

public class UIAnimatorExample : MonoBehaviour
{
    [SerializeField] private Animator panelAnimator;

    public void ShowPanel()
    {
        panelAnimator.SetTrigger("Show");
    }

    public void HidePanel()
    {
        panelAnimator.SetTrigger("Hide");
    }
}
```

### Coroutine ile Basit Animasyon

```csharp
using UnityEngine;
using System.Collections;

public class SimpleUIAnimation : MonoBehaviour
{
    [SerializeField] private CanvasGroup canvasGroup;

    public void FadeIn(float duration)
    {
        StartCoroutine(FadeCoroutine(0, 1, duration));
    }

    public void FadeOut(float duration)
    {
        StartCoroutine(FadeCoroutine(1, 0, duration));
    }

    IEnumerator FadeCoroutine(float from, float to, float duration)
    {
        float elapsed = 0;

        while (elapsed < duration)
        {
            elapsed += Time.unscaledDeltaTime; // Pause sırasında da çalışsın
            canvasGroup.alpha = Mathf.Lerp(from, to, elapsed / duration);
            yield return null;
        }

        canvasGroup.alpha = to;
    }
}
```

---

## Performans İpuçları

### 1. Canvas Optimizasyonu

```csharp
// Birden fazla Canvas kullan
// - Statik elemanlar (arka plan) ayrı Canvas'ta
// - Dinamik elemanlar (skor, sağlık) ayrı Canvas'ta
// Canvas rebuild sadece değişen Canvas'ı etkiler
```

### 2. Layout Group Dikkatli Kullan

```csharp
// Layout grupları her frame hesaplama yapar
// Dinamik olmayan UI için:
// 1. Layout'u ayarla
// 2. Component'i devre dışı bırak veya sil
// 3. RectTransform değerleri korunur
```

### 3. Raycast Target

```csharp
// Tıklanamayan UI elemanlarında Raycast Target'ı kapat
// Image, Text component'lerinde checkbox'ı kaldır
// Performansı artırır
```

### 4. Object Pooling

```csharp
// Sık oluşturulan UI elemanları için pool kullan
// Örnek: Damage popup, inventory item
```

### 5. Canvas Group Kullanımı

```csharp
// Fade animasyonları için her elemana Alpha ayarlamak yerine
// Parent'a CanvasGroup ekle, tek alpha değişikliği yeterli
canvasGroup.alpha = 0.5f;
canvasGroup.interactable = false;    // Etkileşimi kapat
canvasGroup.blocksRaycasts = false;  // Tıklamayı engelleme
```

---

## Özet ve Kontrol Listesi

Bu derste öğrendiklerimiz:
- [x] Canvas sistemi ve render modları
- [x] Canvas Scaler ve ekran uyumu
- [x] RectTransform ve Anchor sistemi
- [x] Temel UI elementleri (Text, Image, Button, Toggle, Slider, Dropdown, Input Field)
- [x] TextMeshPro kullanımı
- [x] Layout grupları (Horizontal, Vertical, Grid)
- [x] Event System ve pointer events
- [x] Drag & Drop sistemi
- [x] Ana Menü, Pause Menü, HUD örnekleri
- [x] Dialog/Popup sistemi
- [x] Loading Screen
- [x] UI animasyonları
- [x] Performans optimizasyonları

---

## Alıştırmalar

1. **Basit Menü**: Play ve Quit butonlu ana menü yapın
2. **Sağlık Barı**: Slider ile sağlık barı yapın, renk değişsin
3. **Skor Sistemi**: Toplanan coinlerde skor artsın, TextMeshPro ile gösterin
4. **Settings Menüsü**: Ses ve grafik ayarları menüsü yapın
5. **Inventory UI**: Grid Layout ile 4x4 envanter yapın
6. **Popup Dialog**: Onay/İptal butonlu dialog sistemi yapın

---

## Faydalı İpuçları

### 1. UI Elemanlarını Bulmak

```csharp
// Sahnede
GameObject.Find("Canvas/Panel/Button");

// Transform ile
transform.Find("ChildName").GetComponent<Button>();

// Tag ile
GameObject.FindWithTag("UIButton");

// En iyisi: SerializeField kullan!
[SerializeField] private Button myButton;
```

### 2. UI Pozisyonu Dünya Koordinatına

```csharp
// Dünya pozisyonunu ekran pozisyonuna
Vector3 screenPos = Camera.main.WorldToScreenPoint(worldPosition);

// Ekran pozisyonunu Canvas pozisyonuna
RectTransformUtility.ScreenPointToLocalPointInRectangle(
    canvas.transform as RectTransform,
    screenPos,
    canvas.worldCamera,
    out Vector2 canvasPos
);
```

### 3. Safe Area (Çentikli Ekranlar)

```csharp
// iPhone çentik, Android kamera deliği için
Rect safeArea = Screen.safeArea;
// UI'ı bu alana göre konumlandır
```

---

## Sonraki Ders

8. Ders'te Ses ve Müzik Sistemi konusunu işleyeceğiz. AudioSource, AudioClip, AudioMixer ve 3D ses konularını öğreneceğiz. Müzik ve ses efektleri yönetimi, ses havuzları oluşturacağız.
