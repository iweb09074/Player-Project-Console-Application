
## 🎛️ 2. **Project Settings > Quality (Android platformu seçili)**

### ✅ **Default Quality**

### Seviye Ayarı:

* **Default Quality Level (Android)**: Sadece `Low` ve `Medium` kalite seviyelerini tutun.


### ✅ **Rendering**

### Pixel Light Count

* **Pixel Light Count**: `1` veya `0` 

### Texture Quality

* **Texture Quality**: `Half Res` veya `Quarter Res` 

### Anti-Aliasing

* **Anti-Aliasing**: Kapalı veya 2x


### ✅ **Shadows**

### Shadows

* **Shadows**: `Hard Shadows Only` veya `Disabled`

### Shadow Distance

* **Shadow Distance**: `15-30`


### ✅ **Other**

Aşağıda tüm ayarları açıklamaları ve **mobil oyunlar için önerilen en iyi değerlerle** birlikte listeliyorum:

### Blend Weights

* **Blend Weights**: `2 Bones` (mobil için yeterli)  

### V Sync Count

* **VSync Count**: `Don't Sync` (FPS sınırlaması gerekirse yapay olarak sınır koyun)

### Anisotropic Textures

* **Anisotropic Textures**: `Per Texture`

### LOD Bias

* **LOD Bias**: 1

---

Unity 2018 kullanıcısı olarak kendi geliştirdiğim android mobil oyunda fpsyi sınırlandırıp 30a ayarlamak için yazılacak doğru bir kod acaba nasıl ve ne şekilde uygulamanın açılışından kapandığı ana kadar aktif bir uzantı görevi görür ?

------------------------------------------------------------------

Unity 2018 kullanıyorsan ve oyunun Android platformunda çalışıyorsa, FPS’yi sabit bir değere (örneğin 30) sınırlamak için `Application.targetFrameRate` özelliğini kullanabilirsin.

Bu ayar genellikle oyun başlatılırken uygulanır ve tüm oyun süresince aktif kalır, ancak bazı durumlarda (örneğin sahne geçişleri veya `QualitySettings` değişiklikleri) tekrar ayarlaman gerekebilir.

Aşağıda FPS’yi 30’a sınırlamak için kullanabileceğin sağlam ve kalıcı bir çözüm sunuyorum.


## ❗ Önemli Notlar

* **`QualitySettings.vSyncCount = 0`** satırı kritiktir. VSync açıksa, `targetFrameRate` işe yaramaz. Özellikle Android cihazlarda FPS sınırı çalışmıyor gibi görünmesinin en büyük sebebi budur.
* Eğer cihaz çok zayıfsa 30 FPS’yi bile tutturamayabilir, bu doğal.







-------------------------------------------------------------------------------------


Senin çözümüne göre "FrameRateLimiter" scripti eğer bir sahne içerisinde oluşturulduysa 2. bir sahnede bu kullanılabileceği anlaşıldı.

Fakat benim sorum Her sahneye etki etmesi zorunlu olan bir singleton yapısı için çözümü nedir?

Yada eğer bir çözüm oluyorsa GameManager gibi bir durum yönetim uzantısı içerisinde "FrameRateLimiter" scriptinin erişlebilir olması mıdır?


-------------------------------------------------------------------------------------


Evet, çok iyi bir noktaya değindin. Bu tür global etki göstermesi gereken sistemlerde (örneğin FPS sınırı, ses kontrolü, oyun durumu, reklam sistemi, vs.), **tek bir sahneye bağlı olmayan**, **oyunun her yerinde çalışan** bir yapıya ihtiyaç duyulur.

Bu da bizi genellikle **GameManager** gibi bir merkezi yönetime (veya çekirdek singleton yapılara) götürür.

---

## 🎯 Cevap: **FPS sınırlayıcı gibi sistemleri en iyi yönetme yolu**, aşağıdakilerden biridir:

---



---


## 📌 Sonuç ve Tercih

| Yöntem                                               | Açıklama                                                             | Kullanım Durumu                          |
| ---------------------------------------------------- | -------------------------------------------------------------------- | ---------------------------------------- |
| **GameManager içinde uygulama**                      | FPS sınırı dahil tüm sistem ayarları tek merkezde                    | ✅ Küçük/orta projeler için ideal         |
| **Modüler FrameRateLimiter + GameManager'dan çağrı** | Ayrı sistem olarak yazılır, GameManager veya başka yerden tetiklenir | ✅ Büyük/karmaşık sistemlerde iyi çalışır |
| **Her sahneye ayrı FPS scripti**                     | Kod tekrarına ve hataya açık                                         | ❌ Önerilmez                              |

---

İstersen sahne geçişlerinde GameManager objesinin gerçekten taşınıp taşınmadığını test eden kodlar da ekleyebilirim. Ya da bu sistemi Unity paketine dökebilirim.

Nasıl ilerleyelim? Projeni daha iyi anlamama yardımcı olacak bir yapı varsa da paylaşabilirsin.
