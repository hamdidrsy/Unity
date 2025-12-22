# 3. Ders - Unity Arayüzü Detaylı İnceleme ve İlk Sahne

## Unity Arayüzü Genel Bakış

Unity'yi ilk açtığınızda karşınıza çıkan arayüz başlangıçta karmaşık görünebilir, ancak her panelin belirli bir görevi vardır. Bu derste tüm panelleri detaylı inceleyeceğiz.

---

## Scene View (Sahne Görünümü)

Scene View, oyun dünyanızı oluşturduğunuz ve düzenlediğiniz ana çalışma alanıdır.

### Navigasyon Kontrolleri
| Kontrol | Eylem |
|---------|-------|
| **Sağ Tık + WASD** | Sahnede uçarak gezinme (Flythrough) |
| **Alt + Sol Tık** | Seçili nesne etrafında döndürme (Orbit) |
| **Alt + Sağ Tık** | Zoom in/out |
| **Orta Tık Sürükleme** | Sahneyi kaydırma (Pan) |
| **Mouse Tekerleği** | Hızlı zoom |
| **F tuşu** | Seçili nesneye odaklanma (Frame) |

### Sahne Araçları (Scene Tools)
Sol üst köşedeki araç çubuğu:

```
[Q] Hand Tool      → Sahneyi kaydırma
[W] Move Tool      → Nesneleri taşıma (X, Y, Z eksenleri)
[E] Rotate Tool    → Nesneleri döndürme
[R] Scale Tool     → Nesneleri boyutlandırma
[T] Rect Tool      → 2D için dikdörtgen düzenleme
[Y] Transform Tool → Tüm dönüşümler tek araçta
```

### Gizmo Ayarları
Sağ üst köşedeki ayarlar:
- **Pivot/Center**: Dönüşüm noktası seçimi
- **Local/Global**: Koordinat sistemi seçimi
- **Scene Lighting**: Sahne ışığını aç/kapa
- **2D/3D**: Görünüm modu değiştirme

---

## Game View (Oyun Görünümü)

Game View, oyuncunun göreceği son görüntüyü simüle eder.

### Kontrol Çubuğu Özellikleri
- **Display**: Birden fazla kamera varsa hangisinden bakılacağı
- **Aspect Ratio**: Ekran oranı simülasyonu (16:9, 4:3, Free vb.)
- **Scale**: Önizleme büyütme/küçültme
- **Maximize on Play**: Oyun modunda tam ekran
- **Mute Audio**: Sesleri susturma
- **Stats**: Performans istatistikleri

### Stats Paneli Metrikleri
```
FPS (Frames Per Second)  → Saniyedeki kare sayısı
Batches                  → Draw call sayısı
Tris (Triangles)         → Ekrandaki üçgen sayısı
Verts (Vertices)         → Ekrandaki köşe noktası sayısı
Screen                   → Çözünürlük bilgisi
SetPass calls            → Shader değişim sayısı
```

---

## Hierarchy Paneli

Sahnedeki tüm GameObject'lerin ağaç yapısında listelendiği panel.

### Temel İşlemler
- **Sağ Tık > Create Empty**: Boş GameObject oluşturma
- **Sağ Tık > 3D Object**: Küp, Küre, Kapsül vb. ekleme
- **Sağ Tık > 2D Object**: Sprite, Tilemap ekleme
- **Sağ Tık > Light**: Işık kaynakları ekleme
- **Sağ Tık > UI**: Arayüz elemanları ekleme

### Parent-Child (Ebeveyn-Çocuk) İlişkisi
```
ParentObject (Ebeveyn)
├── ChildObject1 (Çocuk)
│   └── GrandChild (Torun)
└── ChildObject2 (Çocuk)
```

- Çocuk nesneler, ebeveynin Transform'unu miras alır
- Ebeveyni taşırsanız, çocuklar da birlikte hareket eder
- Sürükle-bırak ile hiyerarşi düzenlenebilir

### Arama ve Filtreleme
- Üst kısımdaki arama çubuğunu kullanın
- İsme, türe veya etikete göre filtreleme yapılabilir

---

## Inspector Paneli

Seçili GameObject'in tüm özelliklerini ve bileşenlerini gösteren panel.

### Temel Bölümler
1. **GameObject Başlığı**
   - Aktif/Pasif checkbox
   - İsim alanı
   - Static ayarları
   - Tag ve Layer seçimi

2. **Transform Bileşeni** (Her GameObject'te bulunur)
   - Position (Pozisyon): X, Y, Z koordinatları
   - Rotation (Döndürme): X, Y, Z dereceleri
   - Scale (Ölçek): X, Y, Z boyutları

3. **Diğer Bileşenler**
   - Mesh Renderer, Collider, Rigidbody vb.
   - Script'ler burada görünür
   - Add Component butonu ile yeni bileşen eklenir

### Bileşen Ekleme
```
Add Component > Physics > Rigidbody    (Fizik motoru)
Add Component > Physics > Box Collider (Çarpışma algılama)
Add Component > Audio > Audio Source   (Ses kaynağı)
Add Component > New Script             (Yeni C# script)
```

---

## Project Paneli

Projedeki tüm dosyaların (asset) yönetildiği panel.

### Klasör Organizasyonu (Önerilen)
```
Assets/
├── _Scenes/          → Oyun sahneleri
├── Scripts/          → C# kodları
├── Prefabs/          → Prefab dosyaları
├── Materials/        → Materyaller
├── Textures/         → Texture/Sprite dosyaları
├── Models/           → 3D modeller
├── Audio/            → Ses dosyaları
│   ├── Music/        → Müzikler
│   └── SFX/          → Ses efektleri
├── Animations/       → Animasyonlar
├── Fonts/            → Font dosyaları
└── Plugins/          → Üçüncü parti eklentiler
```

### İpuçları
- Alt çizgi (_) ile başlayan klasörler üstte sıralanır
- Arama çubuğu ile hızlı dosya bulma
- İki sütunlu veya tek sütunlu görünüm seçilebilir
- Favorilere ekleme özelliği (sağ tık > Add to Favorites)

---

## Console Paneli

Hata mesajları, uyarılar ve debug çıktılarının görüntülendiği panel.

### Mesaj Türleri
| İkon | Tür | Açıklama |
|------|-----|----------|
| Beyaz | Log | Bilgi mesajları (Debug.Log) |
| Sarı | Warning | Uyarılar (Debug.LogWarning) |
| Kırmızı | Error | Hatalar (Debug.LogError) |

### Kontroller
- **Clear**: Konsolu temizle
- **Collapse**: Aynı mesajları grupla
- **Clear on Play**: Oyun başladığında temizle
- **Error Pause**: Hata olunca oyunu duraklat

### Debug Kodları
```csharp
Debug.Log("Normal mesaj");
Debug.LogWarning("Uyarı mesajı");
Debug.LogError("Hata mesajı");
```

---

## İlk Sahnemizi Oluşturalım

Şimdi öğrendiklerimizi uygulayarak basit bir sahne oluşturalım.

### Adım 1: Yeni Proje Oluşturma
1. Unity Hub'ı açın
2. **New Project** butonuna tıklayın
3. Template olarak **3D (Built-in Render Pipeline)** seçin
4. Proje adını girin (örn: "IlkSahnem")
5. **Create Project** butonuna tıklayın

### Adım 2: Sahneye Zemin Ekleme
1. Hierarchy'de sağ tık > **3D Object > Plane**
2. Inspector'da Transform ayarları:
   - Position: (0, 0, 0)
   - Scale: (2, 1, 2) → Daha geniş bir zemin

### Adım 3: Küp Ekleme
1. Hierarchy'de sağ tık > **3D Object > Cube**
2. Transform ayarları:
   - Position: (0, 0.5, 0) → Zeminin üstünde
   - Scale: (1, 1, 1)

### Adım 4: Küre Ekleme
1. Hierarchy'de sağ tık > **3D Object > Sphere**
2. Transform ayarları:
   - Position: (2, 0.5, 0) → Küpün yanında
   - Scale: (1, 1, 1)

### Adım 5: Işık Ayarları
Sahnede varsayılan olarak **Directional Light** bulunur:
1. Hierarchy'den Directional Light'ı seçin
2. Inspector'da Light bileşenini inceleyin:
   - Color: Işık rengi
   - Intensity: Işık şiddeti
   - Shadow Type: Gölge türü (Soft/Hard/No Shadows)

### Adım 6: Kamera Ayarları
1. Hierarchy'den **Main Camera**'yı seçin
2. Transform ayarları:
   - Position: (0, 3, -5) → Sahneyi yukarıdan görecek şekilde
   - Rotation: (30, 0, 0) → Aşağıya baksın
3. Game View'da önizlemeyi kontrol edin

### Adım 7: Sahneyi Kaydetme
1. **Ctrl + S** tuşlarına basın
2. Kayıt konumu olarak `Assets/_Scenes/` klasörünü seçin
3. Sahne adı: "IlkSahne"
4. **Save** butonuna tıklayın

### Adım 8: Test Etme
1. **Play** butonuna basın (üst ortada)
2. Game View'da sahnenizi görün
3. Tekrar **Play** butonuna basarak durdurun

---

## Materyal Ekleme (Bonus)

Nesnelerimize renk verelim:

### Materyal Oluşturma
1. Project panelinde sağ tık > **Create > Material**
2. İsim: "RedMaterial"
3. Inspector'da:
   - Albedo rengi: Kırmızı seçin
   - Smoothness: 0.5

### Materyal Uygulama
1. Oluşturduğunuz materyali Hierarchy'deki Cube'un üzerine sürükleyin
2. Veya Inspector'da Mesh Renderer > Materials > Element 0'a atayın

### Farklı Renklerde Materyaller
```
BlueMaterial   → Küre için mavi
GreenMaterial  → Zemin için yeşil
```

---

## Özet ve Kontrol Listesi

Bu derste öğrendiklerimiz:
- [x] Scene View navigasyonu ve araçları
- [x] Game View ve performans metrikleri
- [x] Hierarchy'de nesne yönetimi
- [x] Inspector ile bileşen düzenleme
- [x] Project panelinde dosya organizasyonu
- [x] Console paneli kullanımı
- [x] İlk sahne oluşturma
- [x] Temel nesneler ekleme (Plane, Cube, Sphere)
- [x] Işık ve kamera ayarları
- [x] Sahne kaydetme
- [x] Materyal oluşturma ve uygulama

---

## Alıştırmalar

1. Sahneye bir silindir (Cylinder) ekleyin
2. Küpü döndürün (Y ekseninde 45 derece)
3. Sarı renkte bir materyal oluşturun ve silindire uygulayın
4. Kamerayı farklı açılardan test edin
5. Directional Light'ın rengini hafif turuncu yapın (gün batımı etkisi)

---

## Sonraki Ders

4. Ders'te GameObject ve Component kavramlarını derinlemesine inceleyeceğiz. Rigidbody, Collider gibi fizik bileşenlerini öğreneceğiz ve nesnelere hareket kazandıracağız.

