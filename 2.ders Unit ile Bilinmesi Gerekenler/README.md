# 2. Ders - Unity ile Bilinmesi Gerekenler

## Unity Editor Hakkında

### Temel Paneller
- **Scene View**: Sahnenizi görsel olarak düzenlediğiniz alan
- **Game View**: Oyununuzun oyuncuya nasıl görüneceğini gösteren önizleme
- **Hierarchy**: Sahnedeki tüm GameObject'lerin listesi
- **Inspector**: Seçili nesnenin özelliklerini düzenleme paneli
- **Project**: Projedeki tüm dosyaların (asset) bulunduğu alan
- **Console**: Hata mesajları ve debug çıktıları

### Kısayol Tuşları
- **W**: Taşıma (Move) aracı
- **E**: Döndürme (Rotate) aracı
- **R**: Ölçekleme (Scale) aracı
- **F**: Seçili nesneye odaklan
- **Ctrl + S**: Sahneyi kaydet
- **Ctrl + P**: Oyunu başlat/durdur

---

## Asset Nedir?

Asset, Unity projenizde kullanılan her türlü dosyadır.

### Asset Türleri
| Tür | Açıklama | Uzantılar |
|-----|----------|-----------|
| **3D Model** | Karakterler, objeler | .fbx, .obj, .blend |
| **Texture** | Görsel dokular | .png, .jpg, .psd |
| **Audio** | Ses efektleri, müzik | .mp3, .wav, .ogg |
| **Script** | Kod dosyaları | .cs |
| **Prefab** | Yeniden kullanılabilir objeler | .prefab |
| **Scene** | Oyun sahneleri | .unity |
| **Material** | Yüzey görünümleri | .mat |
| **Animation** | Animasyon dosyaları | .anim |

### Asset Store
- Unity'nin resmi market yeri
- Ücretsiz ve ücretli asset'ler bulunur
- 3D modeller, ses efektleri, script paketleri, shader'lar mevcut
- **Window > Asset Store** menüsünden erişilir

### Asset Import Etme
1. Dosyaları Project penceresine sürükleyin
2. Veya **Assets > Import New Asset** menüsünü kullanın
3. Unity otomatik olarak uygun formata dönüştürür

---

## Microsoft Visual Studio Hakkında

### Neden Visual Studio?
- Unity'nin resmi desteklediği IDE
- IntelliSense ile kod tamamlama
- Güçlü hata ayıklama (debugging) özellikleri
- Unity API dokümantasyonuna kolay erişim

### Kurulum
1. Unity Hub'dan Editor kurarken **Visual Studio** modülünü seçin
2. Veya [Visual Studio](https://visualstudio.microsoft.com/) sitesinden indirin
3. Kurulum sırasında **"Game development with Unity"** workload'ını seçin

### Unity ile Bağlama
1. **Edit > Preferences > External Tools**
2. External Script Editor olarak Visual Studio'yu seçin
3. Artık script'lere çift tıklayınca VS açılır

### VS Code Alternatifi
- Daha hafif bir seçenek
- **C# Dev Kit** eklentisini kurun
- **Unity** eklentisini kurun

---

## Kritik Bilgiler ve İpuçları

### Proje Yapısı
```
Assets/           → Tüm oyun dosyaları burada
├── Scripts/      → C# kodları
├── Prefabs/      → Hazır objeler
├── Scenes/       → Oyun sahneleri
├── Materials/    → Materyaller
├── Textures/     → Dokular
└── Audio/        → Ses dosyaları

Library/          → Unity'nin cache'i (Git'e eklemeyin!)
Packages/         → Paket bağımlılıkları
ProjectSettings/  → Proje ayarları
```

### Önemli Kavramlar
- **GameObject**: Sahnedeki her şey bir GameObject'tir
- **Component**: GameObject'lere özellik katan parçalar
- **Transform**: Her GameObject'in pozisyon, rotasyon ve ölçek bilgisi
- **Prefab**: Yeniden kullanılabilir GameObject şablonu
- **Script**: C# ile yazılan davranış kodları

### Performans İpuçları
- Gereksiz yere Update() fonksiyonunu kullanmayın
- Büyük texture'ları sıkıştırın
- Polygon sayısını makul tutun (mobil için özellikle)
- Object pooling kullanın (sürekli Instantiate/Destroy yerine)

### Sık Yapılan Hatalar
1. **Sahneyi kaydetmemek** - Ctrl+S alışkanlığı edinin
2. **Library klasörünü Git'e eklemek** - .gitignore kullanın
3. **Public değişkenleri Inspector'da görmemek** - Serialize edilmiş olmalı
4. **Script adı ile class adının uyuşmaması** - Aynı olmalı!

---

## Sonraki Ders
3. Ders'te Unity Arayüzünü detaylı inceleyeceğiz ve ilk sahnemizi oluşturacağız.
