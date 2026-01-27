# 12. Ders - Sahne Yönetimi ve Geçişler

## Giriş

Oyunlar genellikle birden fazla sahneden oluşur: ana menü, oyun seviyeleri, ayarlar ekranı, game over sahnesi. Bu sahneler arasında geçiş yapmak ve verileri taşımak oyun geliştirmenin önemli bir parçasıdır.

Bu derste:
- Scene yapısı ve Build Settings
- SceneManager ile sahne yükleme
- Senkron ve asenkron yükleme
- Additive scene loading
- Loading screen oluşturma
- Sahneler arası veri taşıma
- DontDestroyOnLoad kullanımı
- Sahne geçiş efektleri
- Scene workflow best practices

konularını işleyeceğiz.

---

## Scene Temelleri

### Scene Nedir?

```
┌─────────────────────────────────────────────────────────────┐
│                    SCENE YAPISI                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Scene = Oyunun bir bölümü/ekranı                           │
│                                                             │
│ İçerir:                                                     │
│   - GameObjects ve hierarchy                               │
│   - Lighting ayarları                                      │
│   - Navigation data                                        │
│   - Render settings                                        │
│                                                             │
│ Tipik scene'ler:                                           │
│   - MainMenu                                               │
│   - Level1, Level2, Level3...                             │
│   - GameOver                                               │
│   - Settings                                               │
│   - Loading                                                │
│   - Credits                                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Scene Oluşturma

```
File > New Scene
veya
Ctrl + N
```

### Scene Kaydetme

```
File > Save Scene
veya
Ctrl + S

Kayıt yeri: Assets klasörü içinde
Uzantı: .unity
```

---

## Build Settings

Oyuna dahil edilecek sahneleri belirler.

### Build Settings Açma

```
File > Build Settings
veya
Ctrl + Shift + B
```

### Sahne Ekleme

```
┌─────────────────────────────────────────────────────────────┐
│                  BUILD SETTINGS                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Scenes In Build:                                           │
│ ┌─────────────────────────────────────────────────────────┐│
│ │ ☑ Scenes/MainMenu           0                          ││
│ │ ☑ Scenes/Level1             1                          ││
│ │ ☑ Scenes/Level2             2                          ││
│ │ ☑ Scenes/GameOver           3                          ││
│ └─────────────────────────────────────────────────────────┘│
│                                                             │
│ Ekleme yöntemleri:                                         │
│ 1. Sahneyi buraya sürükle                                 │
│ 2. "Add Open Scenes" butonu                               │
│                                                             │
│ Build Index: Sıra numarası (0'dan başlar)                 │
│ İlk sahne (index 0) oyun başlangıcında yüklenir           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## SceneManager

Sahneleri yönetmek için ana sınıf.

### Namespace

```csharp
using UnityEngine.SceneManagement;
```

### Temel Sahne Yükleme

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneLoader : MonoBehaviour
{
    // İsim ile yükleme
    public void LoadByName(string sceneName)
    {
        SceneManager.LoadScene(sceneName);
    }

    // Index ile yükleme
    public void LoadByIndex(int sceneIndex)
    {
        SceneManager.LoadScene(sceneIndex);
    }

    // Mevcut sahneyi yeniden yükle
    public void ReloadCurrentScene()
    {
        Scene currentScene = SceneManager.GetActiveScene();
        SceneManager.LoadScene(currentScene.name);
        // veya
        SceneManager.LoadScene(currentScene.buildIndex);
    }

    // Sonraki seviye
    public void LoadNextLevel()
    {
        int currentIndex = SceneManager.GetActiveScene().buildIndex;
        int nextIndex = currentIndex + 1;

        // Son seviye kontrolü
        if (nextIndex < SceneManager.sceneCountInBuildSettings)
        {
            SceneManager.LoadScene(nextIndex);
        }
        else
        {
            // Ana menüye dön veya krediler göster
            SceneManager.LoadScene(0);
        }
    }
}
```

### Sahne Bilgilerini Alma

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneInfo : MonoBehaviour
{
    void Start()
    {
        // Aktif sahne
        Scene activeScene = SceneManager.GetActiveScene();

        Debug.Log($"Sahne Adı: {activeScene.name}");
        Debug.Log($"Build Index: {activeScene.buildIndex}");
        Debug.Log($"Path: {activeScene.path}");
        Debug.Log($"Is Loaded: {activeScene.isLoaded}");
        Debug.Log($"Root Count: {activeScene.rootCount}");

        // Yüklü sahne sayısı
        int loadedSceneCount = SceneManager.sceneCount;

        // Build'deki toplam sahne sayısı
        int totalSceneCount = SceneManager.sceneCountInBuildSettings;

        // Tüm yüklü sahneleri listele
        for (int i = 0; i < SceneManager.sceneCount; i++)
        {
            Scene scene = SceneManager.GetSceneAt(i);
            Debug.Log($"Yüklü sahne: {scene.name}");
        }
    }
}
```

---

## LoadSceneMode

Sahne yükleme modları.

```
┌─────────────────────────────────────────────────────────────┐
│                  LOADSCENEMODE                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Single (Varsayılan)                                        │
│   - Mevcut sahneyi kapatır                                │
│   - Yeni sahneyi yükler                                   │
│   - DontDestroyOnLoad hariç tüm objeler yok edilir        │
│                                                             │
│ Additive                                                    │
│   - Mevcut sahne açık kalır                               │
│   - Yeni sahne ek olarak yüklenir                         │
│   - Birden fazla sahne aynı anda aktif                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

```csharp
// Single (varsayılan)
SceneManager.LoadScene("Level1", LoadSceneMode.Single);

// Additive
SceneManager.LoadScene("UI_Overlay", LoadSceneMode.Additive);
```

---

## Asenkron Sahne Yükleme

Oyunun donmaması için arka planda yükleme.

### Temel Async Loading

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;
using System.Collections;

public class AsyncSceneLoader : MonoBehaviour
{
    public void LoadSceneAsync(string sceneName)
    {
        StartCoroutine(LoadSceneAsyncCoroutine(sceneName));
    }

    IEnumerator LoadSceneAsyncCoroutine(string sceneName)
    {
        // Async operasyonu başlat
        AsyncOperation asyncOperation = SceneManager.LoadSceneAsync(sceneName);

        // Yükleme tamamlanana kadar bekle
        while (!asyncOperation.isDone)
        {
            // İlerleme: 0 - 0.9 arası (0.9'da bekler)
            float progress = Mathf.Clamp01(asyncOperation.progress / 0.9f);
            Debug.Log($"Yükleme: {progress * 100}%");

            yield return null;
        }
    }
}
```

### Manuel Aktivasyon

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;
using System.Collections;

public class ManualActivation : MonoBehaviour
{
    IEnumerator LoadWithManualActivation(string sceneName)
    {
        AsyncOperation asyncOperation = SceneManager.LoadSceneAsync(sceneName);

        // Otomatik geçişi engelle
        asyncOperation.allowSceneActivation = false;

        // Yükleme (%90'a kadar)
        while (asyncOperation.progress < 0.9f)
        {
            float progress = asyncOperation.progress / 0.9f;
            Debug.Log($"Yükleme: {progress * 100}%");
            yield return null;
        }

        Debug.Log("Yükleme tamamlandı, geçiş için hazır");

        // Kullanıcı input'u bekle
        yield return new WaitUntil(() => Input.anyKeyDown);

        // Sahneyi aktifleştir
        asyncOperation.allowSceneActivation = true;
    }
}
```

---

## Loading Screen

Profesyonel yükleme ekranı oluşturma.

### Loading Screen Script

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.UI;
using TMPro;
using System.Collections;

public class LoadingScreen : MonoBehaviour
{
    [Header("UI Elements")]
    [SerializeField] private GameObject loadingPanel;
    [SerializeField] private Slider progressBar;
    [SerializeField] private TextMeshProUGUI progressText;
    [SerializeField] private TextMeshProUGUI tipText;
    [SerializeField] private Image fadeImage;

    [Header("Settings")]
    [SerializeField] private float minimumLoadTime = 1f;
    [SerializeField] private float fadeTime = 0.5f;

    [Header("Tips")]
    [SerializeField] private string[] loadingTips;

    private static LoadingScreen instance;
    public static LoadingScreen Instance => instance;

    void Awake()
    {
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    public void LoadScene(string sceneName)
    {
        StartCoroutine(LoadSceneRoutine(sceneName));
    }

    public void LoadScene(int sceneIndex)
    {
        StartCoroutine(LoadSceneRoutine(sceneIndex));
    }

    IEnumerator LoadSceneRoutine(string sceneName)
    {
        yield return StartCoroutine(LoadSceneRoutineInternal(
            SceneManager.LoadSceneAsync(sceneName)
        ));
    }

    IEnumerator LoadSceneRoutine(int sceneIndex)
    {
        yield return StartCoroutine(LoadSceneRoutineInternal(
            SceneManager.LoadSceneAsync(sceneIndex)
        ));
    }

    IEnumerator LoadSceneRoutineInternal(AsyncOperation asyncOperation)
    {
        // Loading panelini göster
        loadingPanel.SetActive(true);

        // Rastgele ipucu göster
        if (loadingTips.Length > 0 && tipText != null)
        {
            tipText.text = loadingTips[Random.Range(0, loadingTips.Length)];
        }

        // Fade in
        yield return StartCoroutine(Fade(0f, 1f));

        // Otomatik aktivasyonu kapat
        asyncOperation.allowSceneActivation = false;

        float elapsedTime = 0f;
        float fakeProgress = 0f;

        // Yükleme döngüsü
        while (!asyncOperation.isDone)
        {
            elapsedTime += Time.deltaTime;

            // Gerçek ilerleme (0-0.9)
            float realProgress = Mathf.Clamp01(asyncOperation.progress / 0.9f);

            // Minimum yükleme süresi için fake progress
            float timeProgress = elapsedTime / minimumLoadTime;

            // İkisinin minimumunu al (smooth görünüm için)
            fakeProgress = Mathf.MoveTowards(fakeProgress,
                Mathf.Min(realProgress, timeProgress), Time.deltaTime);

            // UI güncelle
            UpdateProgress(fakeProgress);

            // Yükleme tamamlandı ve minimum süre geçti
            if (asyncOperation.progress >= 0.9f && elapsedTime >= minimumLoadTime)
            {
                // Progress'i %100'e tamamla
                yield return StartCoroutine(CompleteProgress(fakeProgress));

                // Fade out
                yield return StartCoroutine(Fade(1f, 0f));

                // Sahneyi aktifleştir
                asyncOperation.allowSceneActivation = true;
            }

            yield return null;
        }

        // Loading panelini gizle
        loadingPanel.SetActive(false);
    }

    void UpdateProgress(float progress)
    {
        if (progressBar != null)
            progressBar.value = progress;

        if (progressText != null)
            progressText.text = $"{Mathf.RoundToInt(progress * 100)}%";
    }

    IEnumerator CompleteProgress(float currentProgress)
    {
        while (currentProgress < 1f)
        {
            currentProgress = Mathf.MoveTowards(currentProgress, 1f, Time.deltaTime * 2f);
            UpdateProgress(currentProgress);
            yield return null;
        }
    }

    IEnumerator Fade(float from, float to)
    {
        if (fadeImage == null) yield break;

        float elapsed = 0f;
        Color color = fadeImage.color;

        while (elapsed < fadeTime)
        {
            elapsed += Time.deltaTime;
            color.a = Mathf.Lerp(from, to, elapsed / fadeTime);
            fadeImage.color = color;
            yield return null;
        }

        color.a = to;
        fadeImage.color = color;
    }
}
```

### Kullanım

```csharp
// Herhangi bir yerden
LoadingScreen.Instance.LoadScene("Level1");
LoadingScreen.Instance.LoadScene(2);
```

---

## Additive Scene Loading

Birden fazla sahneyi aynı anda yükleme.

### Kullanım Alanları

```
┌─────────────────────────────────────────────────────────────┐
│             ADDITIVE LOADING KULLANIM ALANLARI             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 1. UI Overlay                                              │
│    - Ana oyun sahnesi + UI sahnesi                        │
│                                                             │
│ 2. Streaming Levels                                        │
│    - Açık dünya oyunları                                  │
│    - Bölgeler ayrı sahnelerde                             │
│                                                             │
│ 3. Modüler Tasarım                                         │
│    - Environment sahnesi                                   │
│    - Gameplay sahnesi                                      │
│    - Audio sahnesi                                         │
│                                                             │
│ 4. Multiplayer                                             │
│    - Her oyuncu için ayrı sahne                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Additive Loading Örneği

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;
using System.Collections;

public class AdditiveSceneManager : MonoBehaviour
{
    [SerializeField] private string gameplayScene = "Gameplay";
    [SerializeField] private string uiScene = "UI_Overlay";
    [SerializeField] private string audioScene = "Audio";

    void Start()
    {
        StartCoroutine(LoadGameScenes());
    }

    IEnumerator LoadGameScenes()
    {
        // Ana sahne zaten yüklü (Gameplay)

        // UI sahnesini additive yükle
        AsyncOperation uiLoad = SceneManager.LoadSceneAsync(uiScene, LoadSceneMode.Additive);
        yield return uiLoad;

        // Audio sahnesini additive yükle
        AsyncOperation audioLoad = SceneManager.LoadSceneAsync(audioScene, LoadSceneMode.Additive);
        yield return audioLoad;

        Debug.Log("Tüm sahneler yüklendi");
    }

    // Sahne kaldırma
    public void UnloadScene(string sceneName)
    {
        SceneManager.UnloadSceneAsync(sceneName);
    }

    // Aktif sahneyi değiştirme
    public void SetActiveScene(string sceneName)
    {
        Scene scene = SceneManager.GetSceneByName(sceneName);
        if (scene.isLoaded)
        {
            SceneManager.SetActiveScene(scene);
        }
    }
}
```

### Streaming World Örneği

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;
using System.Collections.Generic;

public class WorldStreaming : MonoBehaviour
{
    [System.Serializable]
    public class WorldChunk
    {
        public string sceneName;
        public Vector3 centerPosition;
        public float loadRadius = 100f;
        public float unloadRadius = 150f;
        [HideInInspector] public bool isLoaded = false;
        [HideInInspector] public bool isLoading = false;
    }

    [SerializeField] private Transform player;
    [SerializeField] private List<WorldChunk> chunks;

    void Update()
    {
        Vector3 playerPos = player.position;

        foreach (WorldChunk chunk in chunks)
        {
            float distance = Vector3.Distance(playerPos, chunk.centerPosition);

            // Yakınsa yükle
            if (distance < chunk.loadRadius && !chunk.isLoaded && !chunk.isLoading)
            {
                LoadChunk(chunk);
            }
            // Uzaksa kaldır
            else if (distance > chunk.unloadRadius && chunk.isLoaded)
            {
                UnloadChunk(chunk);
            }
        }
    }

    void LoadChunk(WorldChunk chunk)
    {
        chunk.isLoading = true;

        AsyncOperation asyncOp = SceneManager.LoadSceneAsync(chunk.sceneName, LoadSceneMode.Additive);
        asyncOp.completed += (op) =>
        {
            chunk.isLoaded = true;
            chunk.isLoading = false;
            Debug.Log($"Chunk yüklendi: {chunk.sceneName}");
        };
    }

    void UnloadChunk(WorldChunk chunk)
    {
        AsyncOperation asyncOp = SceneManager.UnloadSceneAsync(chunk.sceneName);
        asyncOp.completed += (op) =>
        {
            chunk.isLoaded = false;
            Debug.Log($"Chunk kaldırıldı: {chunk.sceneName}");
        };
    }
}
```

---

## DontDestroyOnLoad

Sahne geçişlerinde objeleri koruma.

### Temel Kullanım

```csharp
using UnityEngine;

public class PersistentObject : MonoBehaviour
{
    void Awake()
    {
        DontDestroyOnLoad(gameObject);
    }
}
```

### Singleton Pattern ile

```csharp
using UnityEngine;

public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    [Header("Game State")]
    public int currentLevel = 1;
    public int playerScore = 0;
    public int playerLives = 3;

    void Awake()
    {
        // Singleton kontrolü
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    public void ResetGame()
    {
        currentLevel = 1;
        playerScore = 0;
        playerLives = 3;
    }

    public void AddScore(int amount)
    {
        playerScore += amount;
    }

    public void LoseLife()
    {
        playerLives--;
        if (playerLives <= 0)
        {
            GameOver();
        }
    }

    void GameOver()
    {
        UnityEngine.SceneManagement.SceneManager.LoadScene("GameOver");
    }
}
```

### DontDestroyOnLoad Objeleri Yönetme

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;
using System.Collections.Generic;

public class PersistentObjectManager : MonoBehaviour
{
    public static PersistentObjectManager Instance { get; private set; }

    private List<GameObject> persistentObjects = new List<GameObject>();

    void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    public void RegisterPersistent(GameObject obj)
    {
        if (!persistentObjects.Contains(obj))
        {
            DontDestroyOnLoad(obj);
            persistentObjects.Add(obj);
        }
    }

    public void UnregisterPersistent(GameObject obj)
    {
        persistentObjects.Remove(obj);
        // Obje bir sonraki sahne değişiminde yok edilecek
    }

    public void ClearAllPersistent()
    {
        foreach (GameObject obj in persistentObjects)
        {
            if (obj != null && obj != gameObject)
            {
                Destroy(obj);
            }
        }
        persistentObjects.Clear();
        persistentObjects.Add(gameObject); // Manager'ı koru
    }
}
```

---

## Sahneler Arası Veri Taşıma

### Yöntem 1: Static Değişkenler

```csharp
public static class GameData
{
    public static int SelectedLevel;
    public static string PlayerName;
    public static int HighScore;
    public static GameDifficulty Difficulty;
}

public enum GameDifficulty { Easy, Normal, Hard }

// Kullanım:
// GameData.SelectedLevel = 3;
// SceneManager.LoadScene("Level" + GameData.SelectedLevel);
```

### Yöntem 2: PlayerPrefs

```csharp
using UnityEngine;

public class DataPersistence : MonoBehaviour
{
    // Kaydetme
    public void SaveData()
    {
        PlayerPrefs.SetInt("HighScore", 1000);
        PlayerPrefs.SetString("PlayerName", "Player1");
        PlayerPrefs.SetFloat("MusicVolume", 0.8f);
        PlayerPrefs.Save();
    }

    // Yükleme
    public void LoadData()
    {
        int highScore = PlayerPrefs.GetInt("HighScore", 0);
        string playerName = PlayerPrefs.GetString("PlayerName", "Guest");
        float musicVolume = PlayerPrefs.GetFloat("MusicVolume", 1f);
    }

    // Silme
    public void ClearData()
    {
        PlayerPrefs.DeleteAll();
    }
}
```

### Yöntem 3: Singleton Manager

```csharp
using UnityEngine;
using System;

public class DataManager : MonoBehaviour
{
    public static DataManager Instance { get; private set; }

    // Oyun verileri
    public PlayerData CurrentPlayer { get; private set; }
    public GameSettings Settings { get; private set; }

    void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
            Initialize();
        }
        else
        {
            Destroy(gameObject);
        }
    }

    void Initialize()
    {
        CurrentPlayer = new PlayerData();
        Settings = new GameSettings();
        LoadFromDisk();
    }

    public void SaveToDisk()
    {
        string playerJson = JsonUtility.ToJson(CurrentPlayer);
        string settingsJson = JsonUtility.ToJson(Settings);

        PlayerPrefs.SetString("PlayerData", playerJson);
        PlayerPrefs.SetString("GameSettings", settingsJson);
        PlayerPrefs.Save();
    }

    public void LoadFromDisk()
    {
        if (PlayerPrefs.HasKey("PlayerData"))
        {
            string playerJson = PlayerPrefs.GetString("PlayerData");
            CurrentPlayer = JsonUtility.FromJson<PlayerData>(playerJson);
        }

        if (PlayerPrefs.HasKey("GameSettings"))
        {
            string settingsJson = PlayerPrefs.GetString("GameSettings");
            Settings = JsonUtility.FromJson<GameSettings>(settingsJson);
        }
    }
}

[Serializable]
public class PlayerData
{
    public string playerName = "Player";
    public int level = 1;
    public int score = 0;
    public int coins = 0;
    public int[] unlockedLevels = { 1 };
}

[Serializable]
public class GameSettings
{
    public float masterVolume = 1f;
    public float musicVolume = 1f;
    public float sfxVolume = 1f;
    public bool fullscreen = true;
    public int qualityLevel = 2;
}
```

### Yöntem 4: ScriptableObject

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "GameData", menuName = "Game/Game Data")]
public class GameDataSO : ScriptableObject
{
    public int currentLevel;
    public int score;
    public int lives;

    public void Reset()
    {
        currentLevel = 1;
        score = 0;
        lives = 3;
    }
}

// Kullanım
public class GameController : MonoBehaviour
{
    [SerializeField] private GameDataSO gameData;

    void Start()
    {
        Debug.Log($"Level: {gameData.currentLevel}");
    }

    public void NextLevel()
    {
        gameData.currentLevel++;
        SceneManager.LoadScene("Level" + gameData.currentLevel);
    }
}
```

---

## Sahne Geçiş Efektleri

### Fade Transition

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;
using System.Collections;

public class SceneTransition : MonoBehaviour
{
    public static SceneTransition Instance { get; private set; }

    [SerializeField] private Image fadeImage;
    [SerializeField] private float fadeTime = 1f;
    [SerializeField] private AnimationCurve fadeCurve = AnimationCurve.EaseInOut(0, 0, 1, 1);

    void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    public void TransitionTo(string sceneName)
    {
        StartCoroutine(TransitionRoutine(sceneName));
    }

    IEnumerator TransitionRoutine(string sceneName)
    {
        // Fade out (siyaha)
        yield return StartCoroutine(Fade(0f, 1f));

        // Sahneyi yükle
        SceneManager.LoadScene(sceneName);

        // Bir frame bekle
        yield return null;

        // Fade in (şeffafa)
        yield return StartCoroutine(Fade(1f, 0f));
    }

    IEnumerator Fade(float from, float to)
    {
        float elapsed = 0f;
        Color color = fadeImage.color;

        while (elapsed < fadeTime)
        {
            elapsed += Time.deltaTime;
            float t = fadeCurve.Evaluate(elapsed / fadeTime);
            color.a = Mathf.Lerp(from, to, t);
            fadeImage.color = color;
            yield return null;
        }

        color.a = to;
        fadeImage.color = color;
    }
}
```

### Wipe Transition

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Collections;

public class WipeTransition : MonoBehaviour
{
    [SerializeField] private RectTransform wipePanel;
    [SerializeField] private float wipeTime = 0.5f;

    public IEnumerator WipeIn()
    {
        wipePanel.gameObject.SetActive(true);

        float screenWidth = Screen.width;
        wipePanel.anchoredPosition = new Vector2(-screenWidth, 0);

        float elapsed = 0f;
        while (elapsed < wipeTime)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / wipeTime;
            float x = Mathf.Lerp(-screenWidth, 0, t);
            wipePanel.anchoredPosition = new Vector2(x, 0);
            yield return null;
        }

        wipePanel.anchoredPosition = Vector2.zero;
    }

    public IEnumerator WipeOut()
    {
        float screenWidth = Screen.width;
        wipePanel.anchoredPosition = Vector2.zero;

        float elapsed = 0f;
        while (elapsed < wipeTime)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / wipeTime;
            float x = Mathf.Lerp(0, screenWidth, t);
            wipePanel.anchoredPosition = new Vector2(x, 0);
            yield return null;
        }

        wipePanel.gameObject.SetActive(false);
    }
}
```

### Circle Transition (Shader ile)

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Collections;

public class CircleTransition : MonoBehaviour
{
    [SerializeField] private Image transitionImage;
    [SerializeField] private Material circleMaterial;
    [SerializeField] private float transitionTime = 1f;

    void Start()
    {
        transitionImage.material = circleMaterial;
    }

    public IEnumerator CircleIn()
    {
        transitionImage.gameObject.SetActive(true);
        yield return StartCoroutine(AnimateCircle(0f, 1.5f));
    }

    public IEnumerator CircleOut()
    {
        yield return StartCoroutine(AnimateCircle(1.5f, 0f));
        transitionImage.gameObject.SetActive(false);
    }

    IEnumerator AnimateCircle(float from, float to)
    {
        float elapsed = 0f;

        while (elapsed < transitionTime)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / transitionTime;
            float radius = Mathf.Lerp(from, to, t);
            circleMaterial.SetFloat("_Radius", radius);
            yield return null;
        }

        circleMaterial.SetFloat("_Radius", to);
    }
}
```

---

## Scene Events

Sahne olaylarını dinleme.

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneEventHandler : MonoBehaviour
{
    void OnEnable()
    {
        // Event'lere abone ol
        SceneManager.sceneLoaded += OnSceneLoaded;
        SceneManager.sceneUnloaded += OnSceneUnloaded;
        SceneManager.activeSceneChanged += OnActiveSceneChanged;
    }

    void OnDisable()
    {
        // Aboneliği iptal et
        SceneManager.sceneLoaded -= OnSceneLoaded;
        SceneManager.sceneUnloaded -= OnSceneUnloaded;
        SceneManager.activeSceneChanged -= OnActiveSceneChanged;
    }

    void OnSceneLoaded(Scene scene, LoadSceneMode mode)
    {
        Debug.Log($"Sahne yüklendi: {scene.name}, Mode: {mode}");

        // Sahneye göre işlem
        switch (scene.name)
        {
            case "MainMenu":
                InitializeMainMenu();
                break;
            case "Gameplay":
                InitializeGameplay();
                break;
        }
    }

    void OnSceneUnloaded(Scene scene)
    {
        Debug.Log($"Sahne kaldırıldı: {scene.name}");
    }

    void OnActiveSceneChanged(Scene oldScene, Scene newScene)
    {
        Debug.Log($"Aktif sahne değişti: {oldScene.name} -> {newScene.name}");
    }

    void InitializeMainMenu()
    {
        // Ana menü başlatma
        Cursor.lockState = CursorLockMode.None;
        Cursor.visible = true;
    }

    void InitializeGameplay()
    {
        // Oyun başlatma
        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;
    }
}
```

---

## Pratik Örnekler

### 1. Level Manager

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class LevelManager : MonoBehaviour
{
    public static LevelManager Instance { get; private set; }

    [SerializeField] private string mainMenuScene = "MainMenu";
    [SerializeField] private string[] levelScenes;
    [SerializeField] private string gameOverScene = "GameOver";

    private int currentLevelIndex = 0;

    void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    public void StartGame()
    {
        currentLevelIndex = 0;
        LoadLevel(currentLevelIndex);
    }

    public void LoadLevel(int levelIndex)
    {
        if (levelIndex >= 0 && levelIndex < levelScenes.Length)
        {
            currentLevelIndex = levelIndex;
            LoadingScreen.Instance.LoadScene(levelScenes[levelIndex]);
        }
    }

    public void NextLevel()
    {
        currentLevelIndex++;

        if (currentLevelIndex < levelScenes.Length)
        {
            LoadLevel(currentLevelIndex);
        }
        else
        {
            // Tüm seviyeler bitti - Victory
            LoadMainMenu();
        }
    }

    public void RestartLevel()
    {
        LoadLevel(currentLevelIndex);
    }

    public void LoadMainMenu()
    {
        Time.timeScale = 1f;
        LoadingScreen.Instance.LoadScene(mainMenuScene);
    }

    public void GameOver()
    {
        SceneManager.LoadScene(gameOverScene);
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

### 2. Main Menu Controller

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class MainMenuController : MonoBehaviour
{
    [Header("Panels")]
    [SerializeField] private GameObject mainPanel;
    [SerializeField] private GameObject settingsPanel;
    [SerializeField] private GameObject levelSelectPanel;

    [Header("Level Buttons")]
    [SerializeField] private Button[] levelButtons;

    void Start()
    {
        ShowMainPanel();
        UpdateLevelButtons();
    }

    void UpdateLevelButtons()
    {
        int[] unlockedLevels = DataManager.Instance.CurrentPlayer.unlockedLevels;

        for (int i = 0; i < levelButtons.Length; i++)
        {
            bool isUnlocked = System.Array.Exists(unlockedLevels, level => level == i + 1);
            levelButtons[i].interactable = isUnlocked;
        }
    }

    public void OnPlayClicked()
    {
        LevelManager.Instance.StartGame();
    }

    public void OnLevelSelectClicked()
    {
        ShowLevelSelectPanel();
    }

    public void OnLevelSelected(int levelIndex)
    {
        LevelManager.Instance.LoadLevel(levelIndex);
    }

    public void OnSettingsClicked()
    {
        ShowSettingsPanel();
    }

    public void OnBackClicked()
    {
        ShowMainPanel();
    }

    public void OnQuitClicked()
    {
        LevelManager.Instance.QuitGame();
    }

    void ShowMainPanel()
    {
        mainPanel.SetActive(true);
        settingsPanel.SetActive(false);
        levelSelectPanel.SetActive(false);
    }

    void ShowSettingsPanel()
    {
        mainPanel.SetActive(false);
        settingsPanel.SetActive(true);
        levelSelectPanel.SetActive(false);
    }

    void ShowLevelSelectPanel()
    {
        mainPanel.SetActive(false);
        settingsPanel.SetActive(false);
        levelSelectPanel.SetActive(true);
    }
}
```

### 3. Pause Menu

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

        // Mouse'u gizle (FPS oyunu için)
        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;
    }

    public void Restart()
    {
        Time.timeScale = 1f;
        LevelManager.Instance.RestartLevel();
    }

    public void MainMenu()
    {
        Time.timeScale = 1f;
        LevelManager.Instance.LoadMainMenu();
    }
}
```

---

## Best Practices

```
┌─────────────────────────────────────────────────────────────┐
│                   BEST PRACTICES                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 1. Sahne Organizasyonu                                     │
│    - Sahneleri klasörlere ayır (Scenes/Levels, Scenes/UI)  │
│    - Tutarlı isimlendirme kullan                           │
│                                                             │
│ 2. Loading Screen                                          │
│    - Her zaman async loading kullan                        │
│    - Minimum yükleme süresi koy (UX için)                 │
│    - Progress bar göster                                   │
│                                                             │
│ 3. DontDestroyOnLoad                                       │
│    - Sadece gerekli objeler için kullan                   │
│    - Singleton pattern uygula                             │
│    - Memory leak'e dikkat                                  │
│                                                             │
│ 4. Veri Taşıma                                             │
│    - Static değişkenler basit veriler için                │
│    - Singleton Manager karmaşık veriler için              │
│    - Önemli veriler için PlayerPrefs backup               │
│                                                             │
│ 5. Transition Efektleri                                    │
│    - Smooth geçişler kullan                               │
│    - Kullanıcıyı bilgilendir (progress)                   │
│                                                             │
│ 6. Scene Events                                            │
│    - sceneLoaded event'ini dinle                          │
│    - Sahne bazlı initialization yap                       │
│                                                             │
│ 7. Additive Loading                                        │
│    - UI'ı ayrı sahnede tut                                │
│    - Modüler tasarım kullan                               │
│    - Unload'ı unutma                                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Özet ve Kontrol Listesi

Bu derste öğrendiklerimiz:
- [x] Scene yapısı ve Build Settings
- [x] SceneManager ile sahne yükleme
- [x] LoadSceneMode (Single, Additive)
- [x] Asenkron sahne yükleme
- [x] Loading Screen oluşturma
- [x] Additive scene loading
- [x] DontDestroyOnLoad kullanımı
- [x] Sahneler arası veri taşıma yöntemleri
- [x] Sahne geçiş efektleri (Fade, Wipe)
- [x] Scene events (sceneLoaded, sceneUnloaded)
- [x] Level Manager ve Pause Menu
- [x] Best practices

---

## Alıştırmalar

1. **Basit Geçiş**: İki sahne arası fade geçişi
2. **Loading Screen**: Progress bar'lı yükleme ekranı
3. **Level Select**: Seviye seçim menüsü
4. **Pause Menu**: ESC ile açılan pause menüsü
5. **Data Persistence**: Skor ve ayarları kaydetme
6. **Additive UI**: Ayrı UI sahnesi sistemi

---

## Faydalı Kaynaklar

- [Unity SceneManager Documentation](https://docs.unity3d.com/ScriptReference/SceneManagement.SceneManager.html)
- [Scene Loading Best Practices](https://docs.unity3d.com/Manual/scene-loading.html)

---

## Sonraki Adımlar

Bu 12 ders ile Unity'nin temel konularını öğrendiniz. Bundan sonra şu konulara bakabilirsiniz:

- **Save/Load Sistemi** (JSON, Binary serialization)
- **Networking** (Multiplayer oyunlar)
- **Shader Programlama**
- **AI ve NavMesh**
- **Timeline ve Cinemachine**
- **Post Processing**
- **Optimization ve Profiling**

Başarılar!
