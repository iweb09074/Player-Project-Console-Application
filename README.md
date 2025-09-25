
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

FPS sınırlama gibi sistemsel görevleri `GameManager` gibi bir ana kontrol yapısına entegre etmek mantıklıdır. Çünkü:

* **`QualitySettings.vSyncCount = 0`** satırı kritiktir. VSync açıksa, `targetFrameRate` işe yaramaz. Özellikle Android cihazlarda FPS sınırı çalışmıyor gibi görünmesinin en büyük sebebi budur.
* Unity 2018 kullanıyorsan ve oyunun Android platformunda çalışıyorsa, FPS’yi sabit bir değere (örneğin 30) sınırlamak için `Application.targetFrameRate` özelliğini kullanabilirsin.
* Bu da bizi genellikle **GameManager** gibi bir merkezi yönetime (veya çekirdek singleton yapılara) götürür.
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

### ✅ **2. Modüler FrameRateLimiter + GameManager'dan çağrı**

Ayrı sistem olarak yazılır, GameManager veya başka yerden tetiklenir. Büyük/karmaşık sistemlerde iyi çalışır.

* Bu ayar genellikle oyun başlatılırken uygulanır ve tüm oyun süresince aktif kalır, ancak bazı durumlarda (örneğin sahne geçişleri veya `QualitySettings` değişiklikleri) tekrar ayarlaman gerekebilir.
* `FrameRateLimiter` sınıfı **statik (static)** yapılmalı.
* Bu durumda, **`FrameRateLimiter` sınıfını `static` olarak tutmak en doğru tercih** olur.
* Sadece sahne geçişi yapıyorsan, `FrameRateLimiter`'ı **static** ve tamamen **stateless** (yani içinde veri tutmayan) hale getir. 

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

### ✅ **3. FPS Sabitleme Scripti (Singleton Yapısıyla)**


- **Kullanım**: Statik sınıflar üzerinden doğrudan script metodu script adına ön ek olarak çağırılır.
- Diğer scriptler doğrudan `FrameRateLimiter.ApplyLimit(30);` şeklinde çağırmalı.
- Böylece sahneye GameObject eklemene gerek kalmaz.

    ```csharp
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine.SceneManagement;
    using UnityEngine;
    using UnityEngine.UI;

    public class LevelComplete : MonoBehaviour
    {
        public FrameRateLimiter frameRateLimiter;

        private void OnTriggerEnter(Collider other)
        {
            frameRateLimiter.ApplyLimit(30);
        }
        
    }
    ```
    ```cs
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

---

### ✅ **4. Harici Script ile Tercih Edilen Konumdan çağrı**

- **Kullanım**: Değişken kullanılmaz ise (sahnede mevcut olduğu için) FindObjectOfType<> metodu kullanılabilir.
- `FindObjectOfType<FrameRateLimiter>()` ile erişip, sahne geçişini tetikliyor.
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
            FindObjectOfType<FrameRateLimiter>().ApplyLimit(30);
        }
        
    }
    ```
    ```cs
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

---




