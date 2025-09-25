
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

### ✅ **1. GameManager İçine Entegre Etmek (Tercih Edilen Yöntem)**

FPS sınırı dahil tüm sistem ayarları tek merkezde yönetmek Küçük/orta projeler için idealdir.

* **`QualitySettings.vSyncCount = 0`** satırı kritiktir. VSync açıksa, `targetFrameRate` işe yaramaz. Özellikle Android cihazlarda FPS sınırı çalışmıyor gibi görünmesinin en büyük sebebi budur.
* Unity 2018 kullanıyorsan ve oyunun Android platformunda çalışıyorsa, FPS’yi sabit bir değere (örneğin 30) sınırlamak için `Application.targetFrameRate` özelliğini kullanabilirsin.
* Bu da bizi genellikle **GameManager** gibi bir merkezi yönetime (veya çekirdek singleton yapılara) götürür.

FPS sınırlama gibi sistemsel görevleri `GameManager` gibi bir ana kontrol yapısına entegre etmek mantıklıdır. Çünkü:

* Oyun boyunca aktif olur.
* Diğer sistemler de buraya bağlanabilir (örneğin ayarlardan FPS limiti değiştirilebilir).
* Yönetimi merkezi olur.

#### 🎮 Örnek `GameManager` Yapısı:

```csharp
using UnityEngine;

public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    public int targetFPS = 30;

    void Awake()
    {
        if (Instance != null)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
        DontDestroyOnLoad(gameObject);
        ApplyGlobalSettings();
    }

    void ApplyGlobalSettings()
    {
        QualitySettings.vSyncCount = 0;
        Application.targetFrameRate = targetFPS;
    }

    void OnApplicationFocus(bool hasFocus)
    {
        if (hasFocus)
        {
            ApplyGlobalSettings();
        }
    }

    void OnApplicationPause(bool pauseStatus)
    {
        if (!pauseStatus)
        {
            ApplyGlobalSettings();
        }
    }
}
```

#### 🔧 Kullanım:

* GameManager objesini yalnızca ilk sahneye yerleştir.
* Diğer sahnelerde yok, ama sahne geçişlerinde `DontDestroyOnLoad` sayesinde kalıcı.

---

### ✅ **2. FPS Sabitleme Scripti (Singleton Yapısıyla)**

Ayrı sistem olarak yazılır, GameManager veya başka yerden tetiklenir. Büyük/karmaşık sistemlerde iyi çalışır.

* Bu ayar genellikle oyun başlatılırken uygulanır ve tüm oyun süresince aktif kalır, ancak bazı durumlarda (örneğin sahne geçişleri veya `QualitySettings` değişiklikleri) tekrar ayarlaman gerekebilir.

- **Kullanım**: Singleton ile sahnede destroy öncesinde kullanılır.
- `SplashController` sahnede olmasa bile, **ilk erişimde otomatik olarak kendi GameObject'ini oluşturur**.
- Böylece `FindObjectOfType<SplashController>()` çağrısı işe yarar hale gelir.

## 📌 Uygulama Adımları

1. Unity Editor’da yeni bir `GameObject` oluştur (örn. `FrameRateManager`)
2. Yukarıdaki `FrameRateLimiter` scriptini oluştur ve bu objeye ata.
3. Script içindeki `targetFPS` değerini 30 yap.
4. Script’in sahnede her zaman yüklü olduğundan emin ol (örneğin ilk sahnede olsun ve `DontDestroyOnLoad` sayesinde sahne geçişlerinde silinmesin).
5. `Application.targetFrameRate = 30` doğrudan sahneye yazılsa da işe yarar ama sahne değişimlerinde sıfırlanabilir. Bu nedenle `DontDestroyOnLoad` yapısı kullanmak en sağlam yöntemdir.

Ve GameManager içinde:

```csharp
FrameRateLimiter.ApplyLimit(30);
```
```csharp
using UnityEngine;

public class FrameRateLimiter : MonoBehaviour
{
    public int targetFPS = 30;

    private static FrameRateLimiter instance;

    void Awake()
    {
        // Singleton yapısı: sadece bir tane olsun
        if (instance != null)
        {
            Destroy(gameObject);
            return;
        }

        instance = this;
        DontDestroyOnLoad(gameObject); // Sahne geçişlerinde silinmesin

        ApplySettings();
    }

    void ApplySettings()
    {
        Application.targetFrameRate = targetFPS;
        QualitySettings.vSyncCount = 0; // VSync kapatılmalı, yoksa FPS'yi sınırlandırmaz
    }

    void OnApplicationFocus(bool hasFocus)
    {
        // Uygulama odağını kaybedip geri gelince ayar kaybolmasın diye tekrar uygula
        if (hasFocus)
        {
            ApplySettings();
        }
    }

    void OnApplicationPause(bool pauseStatus)
    {
        // Aynı şekilde uygulama duraklatılıp devam ederse de uygula
        if (!pauseStatus)
        {
            ApplySettings();
        }
    }
}
```

Böylece:

* `FrameRateLimiter` bir sistem sınıfı olur.
* Ayar GameManager'dan yapılır.
* Testi ve kullanımı kolaylaşır.

---


### ✅ **3. Modüler FrameRateLimiter + GameManager'dan çağrı**



- **Kullanım**: Statik sınıflar üzerinden doğrudan script metodu script adına ön ek olarak çağırılır.
- `SplashController` sınıfı **statik (static)** yapılmalı.
- Diğer scriptler doğrudan `SplashController.Play(index);` şeklinde çağırmalı.
- Bu durumda, **`SplashController` sınıfını `static` olarak tutmak en doğru tercih** olur.
- Sadece sahne geçişi yapıyorsan, `SplashController`'ı **static** ve tamamen **stateless** (yani içinde veri tutmayan) hale getir. 
- Böylece sahneye GameObject eklemene gerek kalmaz.
  ```cs
    using UnityEngine;
    using UnityEngine.SceneManagement;

    public class LevelComplete : MonoBehaviour
    {
        private void OnTriggerEnter(Collider other)
        {
            if (other.CompareTag("Player"))
            {
                int index = SceneManager.GetActiveScene().buildIndex + 1;

                SplashController.Play(index);
            }
        }
    }
  ```
  ```cs
    using UnityEngine;
    using UnityEngine.SceneManagement;

    public static class SplashController
    {
        public static void Play(string sceneName)
        {
            SceneManager.LoadScene(sceneName);
        }

        public static void Play(int buildIndex)
        {
            if (buildIndex < SceneManager.sceneCountInBuildSettings)
            {
                SceneManager.LoadScene(buildIndex);
            }
        }
    }
  ```












---

### ✅ **4. Harici Script ile Tercih Edilen Konumdan çağrı**

### 1. **Sahnede Empty Object Zorunludur**
- **Kullanım**: Değişken kullanılır
- `SplashController` sahne geçişleri için **yalnızca bir araçtır.**
- `LevelComplete` tetiklendiğinde direkt sahne geçişi yapılır.
- Sahne Build Settings'de sıralı olarak eklenmiş mi kontrol et.
- `SplashController` sahnede bir GameObject’e atanmış olmalı.
- **Yazım Örneği**:
  ```cs    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine.SceneManagement;
    using UnityEngine;
    using UnityEngine.UI;

    public class LevelComplete : MonoBehaviour
    {
        public SplashController splashController;

        private void OnTriggerEnter(Collider other)
        {
            int index = SceneManager.GetActiveScene().buildIndex + 1;

            splashController.Play(index);
        }
        
    }
  ```
  ```cs
    using UnityEngine;
    using UnityEngine.SceneManagement;

    public class SplashController : MonoBehaviour
    {
        public void Play(string sceneName)
        {
            SceneManager.LoadScene(sceneName);
        }

        public void Play(int buildIndex)
        {
            if (buildIndex < SceneManager.sceneCountInBuildSettings)
            {
                SceneManager.LoadScene(buildIndex);
            }
        }
    }
  ```

---

### 2. **Çağırı Yapmak İçin Entegre Etmek**
- **Kullanım**: Değişken kullanılmaz ise (sahnede mevcut olduğu için) FindObjectOfType<> metodu kullanılabilir.
- `FindObjectOfType<SplashController>()` ile erişip, sahne geçişini tetikliyor.
- Bu durumda normalde `FindObjectOfType<T>()`, **yalnızca sahnede bulunan aktif GameObject'e atanmış MonoBehaviour türevlerini** bulabilir. 
  ```cs    
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine.SceneManagement;
    using UnityEngine;
    using UnityEngine.UI;

    public class LevelComplete : MonoBehaviour
    {

        private void OnTriggerEnter(Collider other)
        {
            int index = SceneManager.GetActiveScene().buildIndex + 1;

            FindObjectOfType<SplashController>().Play(index);
        }
        
    }
  ```
  ```cs
    using UnityEngine;
    using UnityEngine.SceneManagement;

    public class SplashController : MonoBehaviour
    {
        public void Play(string sceneName)
        {
            SceneManager.LoadScene(sceneName);
        }

        public void Play(int buildIndex)
        {
            if (buildIndex < SceneManager.sceneCountInBuildSettings)
            {
                SceneManager.LoadScene(buildIndex);
            }
        }
    }
  ```
  








---




