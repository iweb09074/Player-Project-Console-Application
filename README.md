
## 🔧 1. **Build Settings (File > Build Settings > Android)**

### ✅ **Build Settings → Texture Compression**

Unity bu kısımda Android için hangi **Texture Compression formatının** kullanılacağını belirlemeni ister.

#### **Texture Compression:**

Aşağıdaki seçenekler arasından seçilir:

| Seçenek                              | Açıklama                                                                         |
| ------------------------------------ | -------------------------------------------------------------------------------- |
| **ETC (default)**                    | En geniş cihaz uyumluluğu. Ancak sıkıştırma kalitesi ve verimlilik düşüktür.     |
| **ETC2 (GLES 3.0 ve üstü cihazlar)** | Daha kaliteli, daha modern; Android 4.3+ cihazlarda desteklenir.                 |
| **ASTC**                             | En yüksek kalite ve en iyi sıkıştırma; ancak sadece yeni cihazlarda desteklenir. |
| **DXT (Tegra cihazlar)**             | Eski NVIDIA Tegra cihazlar için. Yaygın değil.                                   |
| **PVRTC**                            | Sadece iOS cihazlar için uygundur, Android’de kullanılmaz.                       |

#### **Önerilen:**

* Eğer oyun **geniş Android cihaz yelpazesini** hedefliyorsa:
  🔹 **ETC2** (tavsiye edilen)
* Eğer oyun **yüksek kalite grafik** gerektiriyorsa ve **sadece yeni cihazlar** hedefleniyorsa:
  🔹 **ASTC**

> ❗ Unutma: Texture Compression tipi bazı cihazlarda desteklenmezse oyun açılmayabilir veya bozuk görseller olur.

### ✅ Platform:

* **Platform**: Android
* **Texture Compression**: `ETC2` (genel uyumluluk için)

  > Not: Eğer oyununuz sadece belirli cihazlar içinse (örneğin Mali GPU), `ASTC` ya da `DXT` gibi daha verimli seçenekler değerlendirilebilir.

### ✅ Build System:

* **Build System**: Gradle (recommended)
* **Export Project**: ❌ (sadece özel işlemler gerekiyorsa işaretleyin)

### ✅ Development Build (sadece test aşamasında):

* **Development Build**: ✔️ (Debug için)
* **Script Debugging**: ✔️
* **Autoconnect Profiler**: ✔️ (performans analizi için)

### ✅ Minify:

* `Release` modunda:

  * **Minify**: Proguard ile minify işlemi yapılabilir (Proguard ayarlarına dikkat edin)

### ✅ **Compression Method:**

Unity 2018’de bu ayar:
**Build Settings > Compression Method** altında görünür.

#### **Compression Method Seçenekleri:**

| Seçenek     | Açıklama                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------ |
| **Default** | Unity’nin platforma göre uygun gördüğü sıkıştırma yöntemi. Genelde LZ4.                    |
| **LZ4**     | Hızlı yükleme süresi, debug build’lerde iyidir.                                            |
| **LZ4HC**   | Daha yüksek sıkıştırma oranı, ama daha uzun build süresi. Release için daha uygundur.      |
| **None**    | Hiçbir sıkıştırma yapmaz. Büyük build dosyaları üretir. Sadece özel durumlarda kullanılır. |

#### **Önerilen:**

* Geliştirme (Debug/Test) için:
  🔹 **LZ4**
* Yayınlama (Release/Store) için:
  🔹 **LZ4HC**


## 🔍 Özetle:

| Ayar                    | Önerilen                                                      |
| ----------------------- | ------------------------------------------------------------- |
| **Texture Compression** | `ETC2` (genel uyumluluk) veya `ASTC` (yüksek kalite cihazlar) |
| **Compression Method**  | `LZ4HC` (Release build)                                       |

---

## ⚙️ 2. **Project Settings > Player (Android Sekmesi)**

### ✅ Other Settings:

#### ✔️ Identification:

* **Package Name**: `com.companyname.gamename`
* **Version**: `1.0.0`
* **Bundle Version Code**: `1` (Google Play için arttırılır)

#### ✔️ Configuration:

* **Scripting Backend**: `IL2CPP` (daha iyi performans için)
* **API Level**:

  * **Minimum API Level**: `API 19: Android 4.4 (KitKat)` *(isteğe göre 21 veya 23 olabilir)*
  * **Target API Level**: `API 28: Android 9.0 (Pie)` ✅
* **Target Architectures**:
  ✔️ ARMv7
  ✔️ ARM64 *(tavsiye edilir, Google Play şart koşar)*
  ❌ x86 (genelde gerekli değil)

#### ✔️ Internet & Permissions:

* **Internet Access**: `Require`
* **Write Permission**: `External (SDCard)` (içerik kaydı yapıyorsa)
* **Auto Graphics API**: ❌ (manuel ayar önerilir)

#### ✔️ Graphics API:

* ✔️ OpenGLES3 (ilk sırada)
* ✔️ OpenGLES2 (eğer geniş cihaz desteği isteniyorsa)

#### ✔️ Optimization:

* **Multithreaded Rendering**: ✔️ (performans için)
* **GPU Skinning**: ✔️
* **Static Batching**: ✔️
* **Dynamic Batching**: ✔️ (düşük poligonlu objelerde etkili)
* **IL2CPP Code Stripping Level**: `Medium` veya `High` (gereksiz kodları atmak için)

---

## 🎛️ 3. **Project Settings > Quality (Android platformu seçili)**

### ✅ Seviye Ayarı:

* **Default Quality Level (Android)**: `Medium` veya `Low`
  (oyun grafiklerine göre optimize edilir)

### ✅ Önemli Kalite Ayarları:

| Ayar                 | Öneri                                                             |
| -------------------- | ----------------------------------------------------------------- |
| Pixel Light Count    | `1` veya `0`                                                      |
| Texture Quality      | `Half Res` veya `Quarter Res`                                     |
| Anti Aliasing        | `2x` veya `Disabled`                                              |
| Shadows              | `Hard Shadows Only` veya `Disabled`                               |
| Shadow Distance      | `15-30`                                                           |
| Blend Weights        | `2 Bones` (mobil için yeterli)                                    |
| VSync Count          | `Don't Sync` (FPS sınırlaması gerekirse yapay olarak sınır koyun) |
| Anisotropic Textures | `Per Texture`                                                     |

---

## 📦 Ekstra: Player Settings > Publishing Settings

* **Keystore** yapılandırması (release build için zorunlu)
* **Minify with Proguard**: Proguard dosyası düzenlenmeli (gereksiz kodları silmek için)
