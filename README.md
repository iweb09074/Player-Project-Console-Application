
## ⚙️ 1. **Player Settings (File > Player Settings)**

### ✅ **Resolution and Presentation (Android)**

Aşağıda tüm ayarları açıklamaları ve **mobil oyunlar için önerilen en iyi değerlerle** birlikte listeliyorum:

### 1. Start in fullscreen mode**

* **Açıklama**: Oyunun tam ekran mı başlayacağını belirler.
* **Önerilen**: ✅ **İşaretli olmalı** (Tam ekran mobilde standarttır.)


### 2. Render outside safe area** *(Unity 2018.2+)*

* **Açıklama**: Özellikle "notch" (çentikli) cihazlar için. Bu ayar, çentikli bölgelere içerik çizilip çizilmeyeceğini belirler.
* **Önerilen**: Kapalı

  * **Oyun UI’si ekranın kenarına kadar gitmemeliyse**: ❌ Devre dışı
  * **Full ekran kullanımı istiyorsan**: ✅ Aktif edilebilir ama UI dikkatle tasarlanmalı.


### 3. Resolution Scaling Mode**

* **Seçenekler**:

  * **Disabled**: Ölçekleme yapılmaz.
  * **Fixed DPI**: Belirli bir DPI değeri hedeflenir.
  * **Fixed Resolution**: Sabit çözünürlük kullanılır (tavsiye edilmez).
* **Önerilen**: ✅ **Fixed DPI**


### 4. Target DPI** *(sadece "Fixed DPI" seçiliyse etkinleşir)*

* **Açıklama**: Oyun ekranının render edileceği hedef DPI değeri.
* **Önerilen**: `240` veya `320`

  > Düşük DPI = daha az işlem gücü = daha iyi performans
  > Yüksek DPI = daha net görüntü = daha fazla kaynak kullanımı


### 5. Orientation**

* **Default Orientation**: Portrait / Landscape (ihtiyaca göre)

  * `Portrait`: ✅
  * `Portrait Upside Down`: ❌ (çoğu cihaz desteklemez veya rahatsız edici olabilir)
  * `Landscape Left / Right`: Oyunun yönüne göre seçilmeli


### 6. Use 32-bit Display Buffer**

* **Açıklama**: Daha geniş renk aralığı sağlar ama performans etkilenebilir.
* **Önerilen**:

  * ✅ Aktif: Renk kalitesi önemliyse
  * ❌ Pasif: Düşük cihazlar hedefleniyorsa


### 7. Use Protected Graphics Memory**

* **Açıklama**: Ekran görüntüsü alınmasını engeller (DRM korumalı içerikler için).
* **Önerilen**: ❌ Devre dışı

  > Sadece özel durumlar için kullanılır (örneğin güvenlik gerekçesiyle).


### 8. Allow HDR Display**

* \*\*Unity 2018’de Android için genelde **pasif** veya **görünmez** olabilir.
* **Önerilen**: Varsayılan bırakın (mobilde nadiren kullanılır).



---

### ✅ **Other Settings:**

### Identification:

* **Package Name**: `com.companyname.gamename`
* **Version**: `1.0.0`
* **Bundle Version Code**: `1` (Google Play için arttırılır)

### Configuration:

* **Scripting Backend**: `IL2CPP` (daha iyi performans için)
* **API Level**:

  * **Minimum API Level**: `API 19: Android 4.4 (KitKat)` *(isteğe göre 21 veya 23 olabilir)*
  * **Target API Level**: `API 28: Android 9.0 (Pie)` ✅
* **Target Architectures**:
  ✔️ ARMv7
  ✔️ ARM64 *(tavsiye edilir, Google Play şart koşar)*
  ❌ x86 (genelde gerekli değil)

### Internet & Permissions:

* **Internet Access**: `Require`
* **Write Permission**: `External (SDCard)` (içerik kaydı yapıyorsa)
* **Auto Graphics API**: ❌ (manuel ayar önerilir)

### Graphics API:

* ✔️ OpenGLES3 (Vulkan değil, **OpenGLES3** seçili olsun (daha uyumlu))
* ✔️ OpenGLES2 (eğer geniş cihaz desteği isteniyorsa)

### Optimization:

* **Multithreaded Rendering**: ✔️ (performans için)
* **GPU Skinning**: ✔️
* **Static Batching**: ✔️
* **Dynamic Batching**: ✔️ (düşük poligonlu objelerde etkili)
* **IL2CPP Code Stripping Level**: `Medium` veya `High` (gereksiz kodları atmak için)

---

### ✅ **Publishing Settings**

* **Keystore** yapılandırması (release build için zorunlu)
* **Minify with Proguard**: Proguard dosyası düzenlenmeli (gereksiz kodları silmek için)

---

