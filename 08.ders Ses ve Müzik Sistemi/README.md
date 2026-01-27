# 8. Ders - Ses ve Müzik Sistemi

## Giriş

Ses, oyun deneyiminin vazgeçilmez bir parçasıdır. Silah sesleri, ayak sesleri, ortam müzikleri, UI geri bildirimleri - bunların hepsi oyunun atmosferini ve geri bildirimini oluşturur.

Bu derste:
- AudioSource ve AudioClip temelleri
- 2D ve 3D ses farkları
- AudioListener ve ses algılama
- AudioMixer ile ses yönetimi
- Ses efektleri ve müzik ayrımı
- Sound Manager tasarımı
- Object Pooling ile ses optimizasyonu
- Pratik örnekler (ayak sesleri, silah sesleri, ortam sesleri)

konularını işleyeceğiz.

---

## Temel Kavramlar

Unity'nin ses sistemi üç ana bileşenden oluşur:

```
┌─────────────────────────────────────────────────────────────┐
│                    UNITY SES SİSTEMİ                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  AudioClip        → Ses dosyası (.mp3, .wav, .ogg)         │
│  AudioSource      → Ses çalan component                     │
│  AudioListener    → Sesi dinleyen (genelde kamerada)       │
│                                                             │
│  AudioClip → AudioSource → AudioListener → Hoparlör        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## AudioClip

AudioClip, ses dosyasının Unity içindeki temsilidir.

### Desteklenen Formatlar

| Format | Özellik | Kullanım |
|--------|---------|----------|
| .wav | Sıkıştırılmamış, yüksek kalite | Kısa ses efektleri |
| .mp3 | Sıkıştırılmış, küçük boyut | Müzik, uzun sesler |
| .ogg | Sıkıştırılmış, iyi kalite | Her ikisi için ideal |
| .aiff | Sıkıştırılmamış | Profesyonel ses |

### Import Ayarları

```
Project'te ses dosyasını seç → Inspector
```

```
┌─────────────────────────────────────────────────────────────┐
│                 AUDIOCLIP IMPORT SETTINGS                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Force To Mono      → Stereo'yu mono'ya çevir               │
│                     (3D sesler için önerilir)              │
│                                                             │
│ Load In Background → Arka planda yükle                     │
│                     (uzun müzikler için)                   │
│                                                             │
│ Preload Audio Data → Başlangıçta belleğe yükle            │
│                     (sık kullanılan sesler için)           │
│                                                             │
│ Load Type:                                                  │
│   - Decompress On Load → Başta aç (RAM kullanır)           │
│   - Compressed In Memory → Sıkışık tut (CPU kullanır)      │
│   - Streaming → Diskten oku (büyük dosyalar)               │
│                                                             │
│ Compression Format:                                         │
│   - PCM → Sıkıştırılmamış (en iyi kalite)                  │
│   - ADPCM → Hafif sıkıştırma (ses efektleri)               │
│   - Vorbis → Yüksek sıkıştırma (müzik)                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Önerilen Ayarlar

**Kısa Ses Efektleri (silah, zıplama vb.):**
```
Load Type: Decompress On Load
Compression: ADPCM
Preload Audio Data: ✓
```

**Müzik ve Uzun Sesler:**
```
Load Type: Streaming (veya Compressed In Memory)
Compression: Vorbis (Quality: %70)
Preload Audio Data: ✗
Load In Background: ✓
```

**3D Sesler:**
```
Force To Mono: ✓ (stereo gereksiz, mono yeterli)
```

---

## AudioSource

AudioSource, ses çalmak için kullanılan ana component'tir.

### AudioSource Ekleme

```
GameObject > Sağ Tık > Audio > Audio Source
veya
Inspector > Add Component > Audio Source
```

### Temel Özellikler

```csharp
using UnityEngine;

public class AudioSourceBasics : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    [SerializeField] private AudioClip clip;

    void Start()
    {
        // AudioSource referansı alma
        audioSource = GetComponent<AudioSource>();

        // Clip atama
        audioSource.clip = clip;

        // Temel ayarlar
        audioSource.volume = 0.8f;      // 0-1 arası ses seviyesi
        audioSource.pitch = 1f;         // Hız/ton (1 = normal)
        audioSource.loop = false;       // Döngü
        audioSource.playOnAwake = true; // Başlangıçta çal

        // Ses çalma
        audioSource.Play();
    }

    void Update()
    {
        // Ses çalıyor mu kontrol
        if (audioSource.isPlaying)
        {
            Debug.Log($"Çalıyor: {audioSource.time} / {audioSource.clip.length}");
        }
    }
}
```

### Ses Çalma Metodları

```csharp
public class PlayMethods : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    [SerializeField] private AudioClip shootSound;
    [SerializeField] private AudioClip reloadSound;
    [SerializeField] private AudioClip[] footstepSounds;

    // 1. Play() - Atanmış clip'i çalar
    public void PlayAssignedClip()
    {
        audioSource.Play();
    }

    // 2. PlayOneShot() - Farklı clip çalar, üst üste binebilir
    public void Shoot()
    {
        // Mevcut sesi kesmeden yeni ses çalar
        audioSource.PlayOneShot(shootSound);
    }

    // 3. PlayOneShot with volume
    public void ShootWithVolume()
    {
        audioSource.PlayOneShot(shootSound, 0.5f);
    }

    // 4. Rastgele ses çalma
    public void PlayFootstep()
    {
        AudioClip randomClip = footstepSounds[Random.Range(0, footstepSounds.Length)];
        audioSource.PlayOneShot(randomClip);
    }

    // 5. Gecikmeli çalma
    public void PlayDelayed()
    {
        audioSource.clip = reloadSound;
        audioSource.PlayDelayed(0.5f); // 0.5 saniye sonra çal
    }

    // 6. Zamanlanmış çalma (DSP time)
    public void PlayScheduled()
    {
        double startTime = AudioSettings.dspTime + 1.0; // 1 saniye sonra
        audioSource.PlayScheduled(startTime);
    }

    // 7. Durdurma
    public void StopSound()
    {
        audioSource.Stop();
    }

    // 8. Duraklatma
    public void PauseSound()
    {
        audioSource.Pause();
    }

    // 9. Devam ettirme
    public void UnPauseSound()
    {
        audioSource.UnPause();
    }
}
```

### Play vs PlayOneShot Farkı

```
┌─────────────────────────────────────────────────────────────┐
│                  PLAY vs PLAYONESHOT                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Play():                                                     │
│   - audioSource.clip gerektirir                            │
│   - Mevcut sesi keser                                      │
│   - Loop çalışır                                           │
│   - Pause/Stop ile kontrol edilebilir                      │
│   - isPlaying ile takip edilebilir                         │
│                                                             │
│ PlayOneShot():                                              │
│   - Parametre olarak clip alır                             │
│   - Üst üste çalabilir (fire-and-forget)                   │
│   - Loop çalışmaz                                          │
│   - Durdurulamaz                                           │
│   - Ses efektleri için ideal                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## AudioListener

AudioListener, sahnedeki "kulak"tır. Ses buradan duyulur.

### Önemli Kurallar

```
┌─────────────────────────────────────────────────────────────┐
│                    AUDIOLISTENER KURALLARI                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 1. Sahnede sadece 1 tane olmalı                            │
│ 2. Genellikle Main Camera'da bulunur                       │
│ 3. 3D sesler için pozisyonu önemli                         │
│ 4. VR'da head transform'da olmalı                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### AudioListener Ayarları

```csharp
using UnityEngine;

public class AudioListenerControl : MonoBehaviour
{
    void Start()
    {
        // Global ses seviyesi
        AudioListener.volume = 1f; // 0-1 arası

        // Tüm sesleri duraklat
        AudioListener.pause = false;
    }

    // Oyun duraklatıldığında
    public void OnGamePause()
    {
        AudioListener.pause = true;
    }

    public void OnGameResume()
    {
        AudioListener.pause = false;
    }

    // Master volume
    public void SetMasterVolume(float volume)
    {
        AudioListener.volume = Mathf.Clamp01(volume);
    }
}
```

---

## 2D ve 3D Ses

### 2D Ses (Spatial Blend = 0)

```
- Pozisyondan bağımsız
- Her yerde aynı duyulur
- UI sesleri, müzik için
```

### 3D Ses (Spatial Blend = 1)

```
- Pozisyona bağlı
- Mesafe ve yön etkili
- Düşman sesleri, ortam sesleri için
```

### Spatial Blend Ayarı

```csharp
public class SpatialBlendExample : MonoBehaviour
{
    [SerializeField] private AudioSource musicSource;    // 2D
    [SerializeField] private AudioSource sfxSource;      // 3D

    void Start()
    {
        // 2D ses (müzik, UI)
        musicSource.spatialBlend = 0f;

        // 3D ses (ortam, düşmanlar)
        sfxSource.spatialBlend = 1f;

        // Yarı 3D (bazı pozisyon etkisi)
        // audioSource.spatialBlend = 0.5f;
    }
}
```

### 3D Ses Ayarları

```
┌─────────────────────────────────────────────────────────────┐
│                    3D SOUND SETTINGS                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Doppler Level    → Doppler etkisi (araç geçerken vın)     │
│                   0 = kapalı, 1 = normal                   │
│                                                             │
│ Spread           → Ses yayılma açısı (0-360)              │
│                   0 = nokta, 360 = her yön                 │
│                                                             │
│ Min Distance     → Tam ses duyulan mesafe                  │
│                   Bu mesafeden yakın = max volume          │
│                                                             │
│ Max Distance     → Sesin duyulmaz olduğu mesafe           │
│                   Bu mesafeden uzak = sıfır volume         │
│                                                             │
│ Volume Rolloff:                                             │
│   - Logarithmic → Gerçekçi (varsayılan)                   │
│   - Linear → Düz azalma                                    │
│   - Custom → Curve ile özel                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3D Ses Örneği

```csharp
using UnityEngine;

public class Sound3DExample : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;

    void Start()
    {
        // 3D ses ayarları
        audioSource.spatialBlend = 1f;          // Tam 3D
        audioSource.dopplerLevel = 0.5f;        // Hafif doppler
        audioSource.spread = 0f;                // Nokta kaynak
        audioSource.minDistance = 1f;           // 1m içinde tam ses
        audioSource.maxDistance = 50f;          // 50m'de duyulmaz
        audioSource.rolloffMode = AudioRolloffMode.Logarithmic;
    }
}
```

### Mesafe Bazlı Ses Kontrolü (Custom)

```csharp
using UnityEngine;

public class DistanceBasedAudio : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    [SerializeField] private Transform listener; // veya Camera.main.transform

    [SerializeField] private float minDistance = 5f;
    [SerializeField] private float maxDistance = 30f;

    void Update()
    {
        if (listener == null) return;

        float distance = Vector3.Distance(transform.position, listener.position);

        // Mesafeye göre ses seviyesi hesapla
        float volume = 1f - Mathf.InverseLerp(minDistance, maxDistance, distance);
        audioSource.volume = Mathf.Clamp01(volume);

        // Çok uzaktaysa çalmayı durdur (performans)
        if (distance > maxDistance && audioSource.isPlaying)
        {
            audioSource.Pause();
        }
        else if (distance <= maxDistance && !audioSource.isPlaying)
        {
            audioSource.UnPause();
        }
    }
}
```

---

## PlayClipAtPoint

Hızlı bir şekilde 3D ses çalmak için statik metod.

```csharp
using UnityEngine;

public class QuickSound : MonoBehaviour
{
    [SerializeField] private AudioClip explosionSound;

    public void Explode()
    {
        // Pozisyonda ses çal (otomatik temizlenir)
        AudioSource.PlayClipAtPoint(explosionSound, transform.position);

        // Volume ile
        AudioSource.PlayClipAtPoint(explosionSound, transform.position, 0.8f);
    }
}
```

**Not:** PlayClipAtPoint geçici bir GameObject oluşturur ve ses bitince siler. Sık kullanımda performans sorunu yaratabilir - bunun yerine Object Pooling tercih edilmeli.

---

## AudioMixer

AudioMixer, sesleri gruplamak ve efekt uygulamak için kullanılır.

### AudioMixer Oluşturma

```
Project > Sağ Tık > Create > Audio Mixer
```

### Mixer Yapısı

```
┌─────────────────────────────────────────────────────────────┐
│                      AUDIO MIXER                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Master                                                     │
│    ├── Music          → Müzik kanalı                       │
│    ├── SFX            → Ses efektleri                      │
│    │     ├── Weapons  → Silah sesleri                      │
│    │     └── Ambient  → Ortam sesleri                      │
│    └── UI             → Arayüz sesleri                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Grup Oluşturma

```
1. Audio Mixer penceresini aç (Window > Audio > Audio Mixer)
2. Groups'ta + butonuna tıkla
3. Yeni grup adı ver (Music, SFX, UI vb.)
4. Alt gruplar için parent'a sağ tık > Add child group
```

### AudioSource'u Mixer'a Bağlama

```
1. AudioSource component'inde Output alanını bul
2. İstediğin Mixer Group'u seç
```

### Script ile Mixer Kontrolü

```csharp
using UnityEngine;
using UnityEngine.Audio;

public class AudioMixerController : MonoBehaviour
{
    [SerializeField] private AudioMixer audioMixer;

    // Exposed Parameters gerekli:
    // Mixer'da parametre üstüne sağ tık > Expose to script
    // Parameters'ta isim ver (örn: "MusicVolume")

    public void SetMusicVolume(float volume)
    {
        // Volume 0-1 arasında, Mixer -80 ile 0 dB arası
        float dB = Mathf.Log10(Mathf.Max(volume, 0.0001f)) * 20f;
        audioMixer.SetFloat("MusicVolume", dB);
    }

    public void SetSFXVolume(float volume)
    {
        float dB = Mathf.Log10(Mathf.Max(volume, 0.0001f)) * 20f;
        audioMixer.SetFloat("SFXVolume", dB);
    }

    public void SetMasterVolume(float volume)
    {
        float dB = Mathf.Log10(Mathf.Max(volume, 0.0001f)) * 20f;
        audioMixer.SetFloat("MasterVolume", dB);
    }

    // Mute
    public void MuteMusic()
    {
        audioMixer.SetFloat("MusicVolume", -80f);
    }

    // Değer okuma
    public float GetMusicVolume()
    {
        audioMixer.GetFloat("MusicVolume", out float dB);
        return Mathf.Pow(10f, dB / 20f);
    }
}
```

### Volume Slider ile Kullanım

```csharp
using UnityEngine;
using UnityEngine.Audio;
using UnityEngine.UI;

public class VolumeSettings : MonoBehaviour
{
    [SerializeField] private AudioMixer audioMixer;
    [SerializeField] private Slider masterSlider;
    [SerializeField] private Slider musicSlider;
    [SerializeField] private Slider sfxSlider;

    void Start()
    {
        // Kaydedilmiş değerleri yükle
        masterSlider.value = PlayerPrefs.GetFloat("MasterVolume", 1f);
        musicSlider.value = PlayerPrefs.GetFloat("MusicVolume", 1f);
        sfxSlider.value = PlayerPrefs.GetFloat("SFXVolume", 1f);

        // Listener ekle
        masterSlider.onValueChanged.AddListener(SetMasterVolume);
        musicSlider.onValueChanged.AddListener(SetMusicVolume);
        sfxSlider.onValueChanged.AddListener(SetSFXVolume);

        // Başlangıç değerlerini uygula
        SetMasterVolume(masterSlider.value);
        SetMusicVolume(musicSlider.value);
        SetSFXVolume(sfxSlider.value);
    }

    void SetMasterVolume(float value)
    {
        SetMixerVolume("MasterVolume", value);
        PlayerPrefs.SetFloat("MasterVolume", value);
    }

    void SetMusicVolume(float value)
    {
        SetMixerVolume("MusicVolume", value);
        PlayerPrefs.SetFloat("MusicVolume", value);
    }

    void SetSFXVolume(float value)
    {
        SetMixerVolume("SFXVolume", value);
        PlayerPrefs.SetFloat("SFXVolume", value);
    }

    void SetMixerVolume(string parameterName, float normalizedValue)
    {
        // 0-1 değerini dB'ye çevir
        float dB = normalizedValue > 0.0001f
            ? Mathf.Log10(normalizedValue) * 20f
            : -80f;
        audioMixer.SetFloat(parameterName, dB);
    }
}
```

### Mixer Efektleri

```
┌─────────────────────────────────────────────────────────────┐
│                    MIXER EFFECTS                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Lowpass Filter   → Yüksek frekansları keser               │
│                   (su altı, duvar arkası efekti)           │
│                                                             │
│ Highpass Filter  → Düşük frekansları keser                │
│                   (telefon, radyo efekti)                  │
│                                                             │
│ Echo             → Yankı efekti                            │
│                   (mağara, büyük salon)                    │
│                                                             │
│ Distortion       → Bozulma efekti                          │
│                   (hasar aldığında)                        │
│                                                             │
│ Chorus           → Koro efekti                             │
│                   (rüya, halüsinasyon)                     │
│                                                             │
│ Reverb           → Reverb efekti                           │
│                   (mekan simülasyonu)                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Efekt Örneği: Su Altı

```csharp
using UnityEngine;
using UnityEngine.Audio;

public class UnderwaterEffect : MonoBehaviour
{
    [SerializeField] private AudioMixer audioMixer;
    [SerializeField] private float underwaterCutoff = 500f;
    [SerializeField] private float normalCutoff = 22000f;
    [SerializeField] private float transitionSpeed = 2f;

    private bool isUnderwater = false;
    private float targetCutoff;

    void Start()
    {
        targetCutoff = normalCutoff;
    }

    void Update()
    {
        // Smooth geçiş
        audioMixer.GetFloat("LowpassCutoff", out float currentCutoff);
        float newCutoff = Mathf.Lerp(currentCutoff, targetCutoff, Time.deltaTime * transitionSpeed);
        audioMixer.SetFloat("LowpassCutoff", newCutoff);
    }

    public void EnterWater()
    {
        isUnderwater = true;
        targetCutoff = underwaterCutoff;
    }

    public void ExitWater()
    {
        isUnderwater = false;
        targetCutoff = normalCutoff;
    }
}
```

### Mixer Snapshot

Snapshot, mixer'ın belirli bir anındaki tüm ayarlarını kaydeder.

```csharp
using UnityEngine;
using UnityEngine.Audio;

public class AudioSnapshotController : MonoBehaviour
{
    [SerializeField] private AudioMixerSnapshot normalSnapshot;
    [SerializeField] private AudioMixerSnapshot pausedSnapshot;
    [SerializeField] private AudioMixerSnapshot battleSnapshot;
    [SerializeField] private float transitionTime = 1f;

    public void OnGamePause()
    {
        pausedSnapshot.TransitionTo(transitionTime);
    }

    public void OnGameResume()
    {
        normalSnapshot.TransitionTo(transitionTime);
    }

    public void OnBattleStart()
    {
        battleSnapshot.TransitionTo(0.5f);
    }

    public void OnBattleEnd()
    {
        normalSnapshot.TransitionTo(2f);
    }

    // Birden fazla snapshot'ı karıştır
    public void BlendSnapshots()
    {
        AudioMixerSnapshot[] snapshots = { normalSnapshot, battleSnapshot };
        float[] weights = { 0.3f, 0.7f };
        audioMixer.TransitionToSnapshots(snapshots, weights, transitionTime);
    }
}
```

---

## Sound Manager (Singleton)

Merkezi ses yönetimi için bir manager sınıfı.

```csharp
using UnityEngine;
using UnityEngine.Audio;
using System.Collections.Generic;

public class SoundManager : MonoBehaviour
{
    public static SoundManager Instance { get; private set; }

    [Header("Audio Mixer")]
    [SerializeField] private AudioMixer audioMixer;

    [Header("Audio Sources")]
    [SerializeField] private AudioSource musicSource;
    [SerializeField] private AudioSource sfxSource;
    [SerializeField] private AudioSource uiSource;

    [Header("Music Tracks")]
    [SerializeField] private AudioClip menuMusic;
    [SerializeField] private AudioClip gameMusic;
    [SerializeField] private AudioClip bossMusic;

    [Header("Sound Effects")]
    [SerializeField] private AudioClip buttonClick;
    [SerializeField] private AudioClip buttonHover;
    [SerializeField] private AudioClip[] footsteps;
    [SerializeField] private AudioClip jump;
    [SerializeField] private AudioClip land;

    private Dictionary<string, AudioClip> sfxLibrary;

    void Awake()
    {
        // Singleton pattern
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
            InitializeSFXLibrary();
        }
        else
        {
            Destroy(gameObject);
        }
    }

    void InitializeSFXLibrary()
    {
        sfxLibrary = new Dictionary<string, AudioClip>
        {
            { "buttonClick", buttonClick },
            { "buttonHover", buttonHover },
            { "jump", jump },
            { "land", land }
        };
    }

    // ===== MUSIC =====

    public void PlayMusic(string trackName, float fadeTime = 1f)
    {
        AudioClip clip = trackName switch
        {
            "menu" => menuMusic,
            "game" => gameMusic,
            "boss" => bossMusic,
            _ => null
        };

        if (clip != null)
        {
            StartCoroutine(CrossfadeMusic(clip, fadeTime));
        }
    }

    System.Collections.IEnumerator CrossfadeMusic(AudioClip newClip, float duration)
    {
        float startVolume = musicSource.volume;

        // Fade out
        while (musicSource.volume > 0)
        {
            musicSource.volume -= startVolume * Time.deltaTime / duration;
            yield return null;
        }

        musicSource.Stop();
        musicSource.clip = newClip;
        musicSource.Play();

        // Fade in
        while (musicSource.volume < startVolume)
        {
            musicSource.volume += startVolume * Time.deltaTime / duration;
            yield return null;
        }

        musicSource.volume = startVolume;
    }

    public void StopMusic(float fadeTime = 1f)
    {
        StartCoroutine(FadeOutMusic(fadeTime));
    }

    System.Collections.IEnumerator FadeOutMusic(float duration)
    {
        float startVolume = musicSource.volume;

        while (musicSource.volume > 0)
        {
            musicSource.volume -= startVolume * Time.deltaTime / duration;
            yield return null;
        }

        musicSource.Stop();
        musicSource.volume = startVolume;
    }

    // ===== SFX =====

    public void PlaySFX(string soundName)
    {
        if (sfxLibrary.TryGetValue(soundName, out AudioClip clip))
        {
            sfxSource.PlayOneShot(clip);
        }
        else
        {
            Debug.LogWarning($"Sound not found: {soundName}");
        }
    }

    public void PlaySFX(AudioClip clip)
    {
        if (clip != null)
        {
            sfxSource.PlayOneShot(clip);
        }
    }

    public void PlaySFX(AudioClip clip, float volume)
    {
        if (clip != null)
        {
            sfxSource.PlayOneShot(clip, volume);
        }
    }

    public void PlayRandomFootstep()
    {
        if (footsteps.Length > 0)
        {
            AudioClip clip = footsteps[Random.Range(0, footsteps.Length)];
            sfxSource.PlayOneShot(clip);
        }
    }

    // ===== UI SOUNDS =====

    public void PlayUIClick()
    {
        uiSource.PlayOneShot(buttonClick);
    }

    public void PlayUIHover()
    {
        uiSource.PlayOneShot(buttonHover, 0.5f);
    }

    // ===== 3D SFX =====

    public void PlaySFXAtPosition(AudioClip clip, Vector3 position, float volume = 1f)
    {
        // Basit yöntem (her seferinde yeni object)
        // AudioSource.PlayClipAtPoint(clip, position, volume);

        // Daha iyi: Object pooling kullan (aşağıda)
        AudioSource source = GetPooledAudioSource();
        source.transform.position = position;
        source.clip = clip;
        source.volume = volume;
        source.spatialBlend = 1f;
        source.Play();
    }

    // ===== VOLUME CONTROL =====

    public void SetMasterVolume(float volume)
    {
        float dB = VolumeToDecibels(volume);
        audioMixer.SetFloat("MasterVolume", dB);
        PlayerPrefs.SetFloat("MasterVolume", volume);
    }

    public void SetMusicVolume(float volume)
    {
        float dB = VolumeToDecibels(volume);
        audioMixer.SetFloat("MusicVolume", dB);
        PlayerPrefs.SetFloat("MusicVolume", volume);
    }

    public void SetSFXVolume(float volume)
    {
        float dB = VolumeToDecibels(volume);
        audioMixer.SetFloat("SFXVolume", dB);
        PlayerPrefs.SetFloat("SFXVolume", volume);
    }

    float VolumeToDecibels(float volume)
    {
        return volume > 0.0001f ? Mathf.Log10(volume) * 20f : -80f;
    }

    public void LoadVolumeSettings()
    {
        SetMasterVolume(PlayerPrefs.GetFloat("MasterVolume", 1f));
        SetMusicVolume(PlayerPrefs.GetFloat("MusicVolume", 1f));
        SetSFXVolume(PlayerPrefs.GetFloat("SFXVolume", 1f));
    }

    // Object pooling için (basit versiyon)
    [SerializeField] private int poolSize = 10;
    private List<AudioSource> audioPool;

    void CreateAudioPool()
    {
        audioPool = new List<AudioSource>();
        for (int i = 0; i < poolSize; i++)
        {
            GameObject obj = new GameObject($"PooledAudioSource_{i}");
            obj.transform.SetParent(transform);
            AudioSource source = obj.AddComponent<AudioSource>();
            source.playOnAwake = false;
            audioPool.Add(source);
        }
    }

    AudioSource GetPooledAudioSource()
    {
        foreach (AudioSource source in audioPool)
        {
            if (!source.isPlaying)
            {
                return source;
            }
        }
        // Hepsi doluysa yeni oluştur
        GameObject obj = new GameObject($"PooledAudioSource_{audioPool.Count}");
        obj.transform.SetParent(transform);
        AudioSource newSource = obj.AddComponent<AudioSource>();
        newSource.playOnAwake = false;
        audioPool.Add(newSource);
        return newSource;
    }
}
```

### Kullanım Örnekleri

```csharp
// Herhangi bir scriptten:

// Müzik
SoundManager.Instance.PlayMusic("game");
SoundManager.Instance.StopMusic();

// Ses efekti
SoundManager.Instance.PlaySFX("jump");
SoundManager.Instance.PlayRandomFootstep();

// UI
SoundManager.Instance.PlayUIClick();

// 3D ses
SoundManager.Instance.PlaySFXAtPosition(explosionClip, transform.position);

// Volume
SoundManager.Instance.SetMasterVolume(0.8f);
```

---

## Pratik Örnekler

### 1. Ayak Sesleri

```csharp
using UnityEngine;

public class FootstepSounds : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    [SerializeField] private AudioClip[] footstepClips;
    [SerializeField] private float footstepInterval = 0.4f;
    [SerializeField] private float pitchVariation = 0.1f;

    private CharacterController controller;
    private float footstepTimer;
    private int lastPlayedIndex = -1;

    void Start()
    {
        controller = GetComponent<CharacterController>();
    }

    void Update()
    {
        // Hareket ediyorsa ve yerdeyse
        if (controller.isGrounded && controller.velocity.magnitude > 0.1f)
        {
            footstepTimer -= Time.deltaTime;

            if (footstepTimer <= 0)
            {
                PlayFootstep();
                footstepTimer = footstepInterval;
            }
        }
    }

    void PlayFootstep()
    {
        if (footstepClips.Length == 0) return;

        // Aynı sesin arka arkaya çalmasını engelle
        int randomIndex;
        do
        {
            randomIndex = Random.Range(0, footstepClips.Length);
        } while (randomIndex == lastPlayedIndex && footstepClips.Length > 1);

        lastPlayedIndex = randomIndex;

        // Pitch varyasyonu
        audioSource.pitch = 1f + Random.Range(-pitchVariation, pitchVariation);
        audioSource.PlayOneShot(footstepClips[randomIndex]);
    }
}
```

### 2. Zemin Tipine Göre Ses

```csharp
using UnityEngine;

[System.Serializable]
public class SurfaceSound
{
    public string surfaceTag;
    public AudioClip[] footsteps;
}

public class SurfaceFootsteps : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    [SerializeField] private SurfaceSound[] surfaceSounds;
    [SerializeField] private SurfaceSound defaultSurface;

    private SurfaceSound currentSurface;

    void Start()
    {
        currentSurface = defaultSurface;
    }

    void OnControllerColliderHit(ControllerColliderHit hit)
    {
        UpdateSurface(hit.gameObject.tag);
    }

    void UpdateSurface(string tag)
    {
        foreach (SurfaceSound surface in surfaceSounds)
        {
            if (surface.surfaceTag == tag)
            {
                currentSurface = surface;
                return;
            }
        }
        currentSurface = defaultSurface;
    }

    public void PlayFootstep()
    {
        if (currentSurface.footsteps.Length == 0) return;

        AudioClip clip = currentSurface.footsteps[Random.Range(0, currentSurface.footsteps.Length)];
        audioSource.PlayOneShot(clip);
    }
}
```

### 3. Silah Ses Sistemi

```csharp
using UnityEngine;

public class WeaponAudio : MonoBehaviour
{
    [Header("Audio Source")]
    [SerializeField] private AudioSource audioSource;

    [Header("Sound Clips")]
    [SerializeField] private AudioClip fireSound;
    [SerializeField] private AudioClip reloadStartSound;
    [SerializeField] private AudioClip reloadEndSound;
    [SerializeField] private AudioClip emptyClickSound;
    [SerializeField] private AudioClip equipSound;

    [Header("Settings")]
    [SerializeField] private float firePitchMin = 0.95f;
    [SerializeField] private float firePitchMax = 1.05f;

    public void PlayFire()
    {
        audioSource.pitch = Random.Range(firePitchMin, firePitchMax);
        audioSource.PlayOneShot(fireSound);
    }

    public void PlayReloadStart()
    {
        audioSource.pitch = 1f;
        audioSource.PlayOneShot(reloadStartSound);
    }

    public void PlayReloadEnd()
    {
        audioSource.pitch = 1f;
        audioSource.PlayOneShot(reloadEndSound);
    }

    public void PlayEmptyClick()
    {
        audioSource.pitch = 1f;
        audioSource.PlayOneShot(emptyClickSound);
    }

    public void PlayEquip()
    {
        audioSource.pitch = 1f;
        audioSource.PlayOneShot(equipSound);
    }
}
```

### 4. Ortam Sesleri (Ambient)

```csharp
using UnityEngine;

public class AmbientSoundZone : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    [SerializeField] private AudioClip ambientClip;
    [SerializeField] private float maxVolume = 0.5f;
    [SerializeField] private float fadeSpeed = 2f;

    private float targetVolume = 0f;
    private bool playerInZone = false;

    void Start()
    {
        audioSource.clip = ambientClip;
        audioSource.loop = true;
        audioSource.volume = 0f;
        audioSource.Play();
    }

    void Update()
    {
        // Smooth volume geçişi
        audioSource.volume = Mathf.MoveTowards(
            audioSource.volume,
            targetVolume,
            fadeSpeed * Time.deltaTime
        );
    }

    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            playerInZone = true;
            targetVolume = maxVolume;
        }
    }

    void OnTriggerExit(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            playerInZone = false;
            targetVolume = 0f;
        }
    }
}
```

### 5. Rastgele Ortam Sesleri

```csharp
using UnityEngine;

public class RandomAmbientSounds : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    [SerializeField] private AudioClip[] ambientClips;

    [SerializeField] private float minInterval = 5f;
    [SerializeField] private float maxInterval = 15f;
    [SerializeField] private float volumeMin = 0.3f;
    [SerializeField] private float volumeMax = 0.7f;

    private float nextPlayTime;

    void Start()
    {
        ScheduleNextSound();
    }

    void Update()
    {
        if (Time.time >= nextPlayTime)
        {
            PlayRandomSound();
            ScheduleNextSound();
        }
    }

    void PlayRandomSound()
    {
        if (ambientClips.Length == 0) return;

        AudioClip clip = ambientClips[Random.Range(0, ambientClips.Length)];
        float volume = Random.Range(volumeMin, volumeMax);
        audioSource.PlayOneShot(clip, volume);
    }

    void ScheduleNextSound()
    {
        nextPlayTime = Time.time + Random.Range(minInterval, maxInterval);
    }
}
```

### 6. Müzik Playlist

```csharp
using UnityEngine;
using System.Collections;

public class MusicPlaylist : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    [SerializeField] private AudioClip[] tracks;
    [SerializeField] private bool shuffle = true;
    [SerializeField] private float crossfadeTime = 2f;

    private int currentTrackIndex = 0;
    private int[] shuffledIndices;

    void Start()
    {
        if (tracks.Length == 0) return;

        if (shuffle)
        {
            ShufflePlaylist();
        }

        PlayCurrentTrack();
    }

    void Update()
    {
        // Şarkı bittiyse sonrakine geç
        if (!audioSource.isPlaying && audioSource.clip != null)
        {
            NextTrack();
        }
    }

    void ShufflePlaylist()
    {
        shuffledIndices = new int[tracks.Length];
        for (int i = 0; i < tracks.Length; i++)
        {
            shuffledIndices[i] = i;
        }

        // Fisher-Yates shuffle
        for (int i = shuffledIndices.Length - 1; i > 0; i--)
        {
            int j = Random.Range(0, i + 1);
            int temp = shuffledIndices[i];
            shuffledIndices[i] = shuffledIndices[j];
            shuffledIndices[j] = temp;
        }
    }

    int GetTrackIndex(int playlistIndex)
    {
        if (shuffle)
        {
            return shuffledIndices[playlistIndex % tracks.Length];
        }
        return playlistIndex % tracks.Length;
    }

    void PlayCurrentTrack()
    {
        int trackIndex = GetTrackIndex(currentTrackIndex);
        audioSource.clip = tracks[trackIndex];
        audioSource.Play();
    }

    public void NextTrack()
    {
        currentTrackIndex++;
        if (currentTrackIndex >= tracks.Length && shuffle)
        {
            ShufflePlaylist();
            currentTrackIndex = 0;
        }

        StartCoroutine(CrossfadeToNextTrack());
    }

    public void PreviousTrack()
    {
        currentTrackIndex--;
        if (currentTrackIndex < 0)
        {
            currentTrackIndex = tracks.Length - 1;
        }

        StartCoroutine(CrossfadeToNextTrack());
    }

    IEnumerator CrossfadeToNextTrack()
    {
        float startVolume = audioSource.volume;

        // Fade out
        while (audioSource.volume > 0)
        {
            audioSource.volume -= startVolume * Time.deltaTime / crossfadeTime;
            yield return null;
        }

        PlayCurrentTrack();

        // Fade in
        while (audioSource.volume < startVolume)
        {
            audioSource.volume += startVolume * Time.deltaTime / crossfadeTime;
            yield return null;
        }
    }
}
```

### 7. UI Ses Geri Bildirimi

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class UIButtonSound : MonoBehaviour, IPointerEnterHandler, IPointerClickHandler
{
    [SerializeField] private AudioClip hoverSound;
    [SerializeField] private AudioClip clickSound;

    private AudioSource audioSource;

    void Start()
    {
        // UI için 2D AudioSource
        audioSource = gameObject.AddComponent<AudioSource>();
        audioSource.playOnAwake = false;
        audioSource.spatialBlend = 0f; // 2D
    }

    public void OnPointerEnter(PointerEventData eventData)
    {
        if (hoverSound != null)
        {
            audioSource.PlayOneShot(hoverSound, 0.5f);
        }
    }

    public void OnPointerClick(PointerEventData eventData)
    {
        if (clickSound != null)
        {
            audioSource.PlayOneShot(clickSound);
        }
    }
}
```

---

## Object Pooling ile Ses Optimizasyonu

```csharp
using UnityEngine;
using System.Collections.Generic;

public class AudioPool : MonoBehaviour
{
    public static AudioPool Instance { get; private set; }

    [SerializeField] private int initialPoolSize = 20;
    [SerializeField] private AudioMixerGroup sfxMixerGroup;

    private Queue<AudioSource> availableSources;
    private List<AudioSource> activeSources;

    void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
            InitializePool();
        }
        else
        {
            Destroy(gameObject);
        }
    }

    void InitializePool()
    {
        availableSources = new Queue<AudioSource>();
        activeSources = new List<AudioSource>();

        for (int i = 0; i < initialPoolSize; i++)
        {
            CreateNewAudioSource();
        }
    }

    AudioSource CreateNewAudioSource()
    {
        GameObject obj = new GameObject("PooledAudio");
        obj.transform.SetParent(transform);

        AudioSource source = obj.AddComponent<AudioSource>();
        source.playOnAwake = false;
        source.outputAudioMixerGroup = sfxMixerGroup;

        availableSources.Enqueue(source);
        return source;
    }

    public AudioSource GetSource()
    {
        // Müsait source yoksa yeni oluştur
        if (availableSources.Count == 0)
        {
            CreateNewAudioSource();
        }

        AudioSource source = availableSources.Dequeue();
        activeSources.Add(source);
        source.gameObject.SetActive(true);
        return source;
    }

    public void ReturnSource(AudioSource source)
    {
        source.Stop();
        source.clip = null;
        source.gameObject.SetActive(false);
        activeSources.Remove(source);
        availableSources.Enqueue(source);
    }

    void Update()
    {
        // Çalmayı bitirenleri geri al
        for (int i = activeSources.Count - 1; i >= 0; i--)
        {
            if (!activeSources[i].isPlaying)
            {
                ReturnSource(activeSources[i]);
            }
        }
    }

    // Kullanım metodları
    public void PlayAt(AudioClip clip, Vector3 position, float volume = 1f)
    {
        AudioSource source = GetSource();
        source.transform.position = position;
        source.spatialBlend = 1f; // 3D
        source.volume = volume;
        source.clip = clip;
        source.Play();
    }

    public void Play2D(AudioClip clip, float volume = 1f)
    {
        AudioSource source = GetSource();
        source.spatialBlend = 0f; // 2D
        source.volume = volume;
        source.clip = clip;
        source.Play();
    }

    public void PlayWithPitch(AudioClip clip, Vector3 position, float pitch, float volume = 1f)
    {
        AudioSource source = GetSource();
        source.transform.position = position;
        source.spatialBlend = 1f;
        source.pitch = pitch;
        source.volume = volume;
        source.clip = clip;
        source.Play();
    }
}
```

### Kullanımı

```csharp
// 3D ses
AudioPool.Instance.PlayAt(explosionClip, transform.position);

// 2D ses
AudioPool.Instance.Play2D(uiClickClip);

// Pitch ile
AudioPool.Instance.PlayWithPitch(hitClip, transform.position, Random.Range(0.9f, 1.1f));
```

---

## Performans İpuçları

### 1. Ses Dosyası Optimizasyonu

```
┌─────────────────────────────────────────────────────────────┐
│              SES DOSYASI OPTİMİZASYONU                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Kısa sesler (< 1 sn):                                      │
│   → Decompress On Load                                     │
│   → ADPCM sıkıştırma                                       │
│   → Preload: Evet                                          │
│                                                             │
│ Orta sesler (1-5 sn):                                      │
│   → Compressed In Memory                                    │
│   → Vorbis sıkıştırma                                      │
│   → Preload: Duruma göre                                   │
│                                                             │
│ Uzun sesler/Müzik (> 5 sn):                                │
│   → Streaming                                               │
│   → Vorbis sıkıştırma                                      │
│   → Preload: Hayır                                         │
│   → Load In Background: Evet                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2. AudioSource Limitleri

```csharp
// Aynı anda çalan maksimum ses sayısı
// Edit > Project Settings > Audio > Max Virtual Voices

// Gerçek voice sayısı
// Max Real Voices = 32 (varsayılan)
// Aşılırsa düşük öncelikli sesler susturulur
```

### 3. Priority Ayarı

```csharp
// 0 = En yüksek öncelik
// 256 = En düşük öncelik
// Önemli sesler (dialog, UI): 0-50
// Normal efektler: 128
// Ortam sesleri: 200+

audioSource.priority = 0; // Önemli ses
audioSource.priority = 128; // Normal
audioSource.priority = 255; // Düşük öncelik
```

### 4. Mesafe Kontrolü

```csharp
// Uzaktaki sesleri çalma
public class DistanceCulling : MonoBehaviour
{
    [SerializeField] private AudioSource audioSource;
    [SerializeField] private float cullDistance = 100f;

    private Transform listener;

    void Start()
    {
        listener = Camera.main.transform;
    }

    public void PlaySound(AudioClip clip)
    {
        float distance = Vector3.Distance(transform.position, listener.position);

        if (distance < cullDistance)
        {
            audioSource.PlayOneShot(clip);
        }
    }
}
```

### 5. Birden Fazla AudioSource

```csharp
// Farklı kategoriler için ayrı AudioSource
// - Tek bir source birden fazla PlayOneShot kaldırır
// - Ama farklı volume/pitch için ayrı source gerekli

[SerializeField] private AudioSource footstepSource;  // Ayak sesleri
[SerializeField] private AudioSource weaponSource;    // Silah sesleri
[SerializeField] private AudioSource voiceSource;     // Diyalog
```

---

## Özet ve Kontrol Listesi

Bu derste öğrendiklerimiz:
- [x] AudioClip import ayarları
- [x] AudioSource temel kullanımı
- [x] Play vs PlayOneShot farkı
- [x] AudioListener ve konumu
- [x] 2D ve 3D ses farkları
- [x] Spatial Blend ve 3D ses ayarları
- [x] AudioMixer oluşturma ve kullanma
- [x] Volume kontrolü (slider ile)
- [x] Mixer efektleri ve snapshot'lar
- [x] Sound Manager singleton
- [x] Object Pooling ile optimizasyon
- [x] Ayak sesleri sistemi
- [x] Silah sesleri
- [x] Ortam sesleri (ambient)
- [x] Müzik playlist sistemi
- [x] Performans ipuçları

---

## Alıştırmalar

1. **Basit Ses Sistemi**: Bir butona tıklayınca ses çalan sistem yapın
2. **Volume Kontrolü**: Slider ile ses seviyesi ayarlayan ayarlar menüsü
3. **Ayak Sesleri**: Karakterin yürürken ayak sesi çıkarması
4. **Ortam Müziği**: Sahneye girince müzik başlasın, crossfade ile değişsin
5. **3D Ses**: Bir objeye yaklaştıkça sesi artan sistem
6. **AudioMixer**: Music, SFX, UI kanalları olan mixer sistemi
7. **Ambient Zone**: Belirli bölgelerde farklı ortam sesleri

---

## Faydalı Kaynaklar

- [Unity Audio Manual](https://docs.unity3d.com/Manual/Audio.html)
- [AudioMixer Documentation](https://docs.unity3d.com/Manual/AudioMixer.html)
- [Audio Optimization](https://docs.unity3d.com/Manual/AudioPerformanceOptimizations.html)

---

## Sonraki Ders

9. Ders'te **Fizik Sistemi ve Çarpışmalar** konusunu işleyeceğiz. Rigidbody, Collider, Trigger, Physics Material ve raycast konularını öğreneceğiz.
