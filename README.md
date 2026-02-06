# İST-DEĞER | İstanbul Değerleme Portal v3.0 Pro

## 🎯 Proje Hakkında

İstanbul Büyükşehir Belediyesi Emlak ve Değerleme Müdürlüğü için geliştirilmiş, **tamamen açık kaynak API'ler** kullanarak çalışan profesyonel, gerçek zamanlı dashboard sistemi.

### ✨ Özellikler

- 🗺️ **Interaktif Harita Sistemi** - Leaflet.js ile güçlendirilmiş
- 📊 **Canlı Veri Akışı** - Gerçek zamanlı ekonomik göstergeler
- 🏢 **İBB Açık Veri Entegrasyonu** - İSPARK, Metro, İETT verileri
- 🌍 **Deprem Takibi** - Kandilli Rasathanesi verileri
- 💱 **TCMB Döviz Kurları** - EVDS API entegrasyonu
- 🌤️ **Hava Durumu** - OpenWeatherMap API
- 📰 **Haber Akışı** - Güncel emlak haberleri
- 📈 **İlçe Bazlı Analiz** - Chart.js ile görselleştirme
- 🕌 **Namaz Vakitleri** - İstanbul için güncel saatler

---

## 🚀 Kurulum

### 1. Dosyaları İndirin

```bash
git clone https://github.com/yourusername/istanbul-degerleme-dashboard.git
cd istanbul-degerleme-dashboard
```

### 2. Web Sunucusu ile Çalıştırın

**Python ile:**
```bash
python -m http.server 8000
```

**PHP ile:**
```bash
php -S localhost:8000
```

**Node.js ile:**
```bash
npx http-server
```

### 3. Tarayıcıda Açın

```
http://localhost:8000/istanbul-degerleme-dashboard.html
```

---

## 🔑 API Entegrasyonları

### 1️⃣ İBB Açık Veri Portalı

**Kayıt:** https://data.ibb.gov.tr

**Kullanılan Servisler:**
- İSPARK Otopark Bilgileri
- Metro İstasyon Verileri
- İETT Otobüs Lokasyonları
- Şehir Haritası API

**Örnek Kullanım:**
```javascript
// İSPARK Otoparkları
const isparkAPI = 'https://data.ibb.gov.tr/api/3/action/datastore_search?resource_id=f4f56e58-5210-4f17-b852-effe356a890c';

fetch(isparkAPI)
  .then(response => response.json())
  .then(data => {
    console.log('İSPARK verileri:', data);
  });
```

**Doküman:** https://data.ibb.gov.tr/dataset?res_format=API

---

### 2️⃣ TCMB EVDS (Elektronik Veri Dağıtım Sistemi)

**Kayıt:** https://evds2.tcmb.gov.tr

**API Anahtarı Alma:**
1. https://evds2.tcmb.gov.tr adresine gidin
2. "Giriş Yap" → "Kayıt Ol"
3. Profil → "API ANAHTARI" butonuna tıklayın

**Örnek Kullanım:**
```javascript
const API_KEY = 'YOUR_EVDS_API_KEY';
const series = 'TP.DK.USD.S.YTL'; // USD/TRY
const startDate = '01-01-2025';
const endDate = '31-12-2025';

const url = `https://evds2.tcmb.gov.tr/service/evds/series=${series}&startDate=${startDate}&endDate=${endDate}&type=json&key=${API_KEY}`;

fetch(url, {
  headers: {
    'key': API_KEY
  }
})
  .then(response => response.json())
  .then(data => {
    console.log('Döviz kurları:', data);
  });
```

**Mevcut Seriler:**
- `TP.DK.USD.S.YTL` - USD/TRY Satış
- `TP.DK.EUR.S.YTL` - EUR/TRY Satış
- `TP.DK.GBP.S.YTL` - GBP/TRY Satış
- `TP.AB.A01` - Gram Altın
- `TP.FE.OKTG01` - TÜFE (Enflasyon)

**Doküman:** https://evds2.tcmb.gov.tr/help/videos/EVDS_Web_Servis_Kullanim_Kilavuzu.pdf

---

### 3️⃣ Kandilli Rasathanesi (Deprem Verileri)

**API:** http://www.koeri.boun.edu.tr/scripts/lst0.asp

**Örnek Kullanım:**
```javascript
// AFAD Deprem API (Alternatif)
const depremAPI = 'https://api.orhanaydogdu.com.tr/deprem/kandilli/live';

fetch(depremAPI)
  .then(response => response.json())
  .then(data => {
    console.log('Son depremler:', data.result);
  });
```

**Not:** Resmi AFAD API için https://deprem.afad.gov.tr/apidocs/ adresinden kayıt olunması gerekir.

---

### 4️⃣ OpenWeatherMap API

**Kayıt:** https://openweathermap.org/api

**Free Tier:** 60 çağrı/dakika, 1,000,000 çağrı/ay

**Örnek Kullanım:**
```javascript
const API_KEY = 'YOUR_OPENWEATHER_API_KEY';
const city = 'Istanbul';

const weatherAPI = `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric&lang=tr`;

fetch(weatherAPI)
  .then(response => response.json())
  .then(data => {
    console.log('Hava durumu:', data);
    console.log('Sıcaklık:', data.main.temp);
    console.log('Nem:', data.main.humidity);
  });
```

**Doküman:** https://openweathermap.org/current

---

### 5️⃣ CollectAPI (Eczane, Haber, vs.)

**Kayıt:** https://collectapi.com

**Free Tier:** API'ye göre değişir

**Kullanılabilir Servisler:**
- Nöbetçi Eczaneler
- Namaz Vakitleri
- Döviz Kurları
- Hava Durumu
- Haber Akışı

**Örnek Kullanım:**
```javascript
const API_KEY = 'YOUR_COLLECTAPI_KEY';

// Nöbetçi Eczaneler
fetch('https://api.collectapi.com/health/dutyPharmacy?ilce=Kadikoy&il=Istanbul', {
  headers: {
    'authorization': `apikey ${API_KEY}`,
    'content-type': 'application/json'
  }
})
  .then(response => response.json())
  .then(data => {
    console.log('Nöbetçi eczaneler:', data.result);
  });

// Namaz Vakitleri
fetch('https://api.collectapi.com/pray/all?data.city=istanbul', {
  headers: {
    'authorization': `apikey ${API_KEY}`,
    'content-type': 'application/json'
  }
})
  .then(response => response.json())
  .then(data => {
    console.log('Namaz vakitleri:', data.result);
  });
```

**Doküman:** https://collectapi.com/tr/api

---

### 6️⃣ GitHub - Turkish APIs (Topluluk)

**Repo:** https://github.com/3rt4nm4n/turkish-apis

Bu repoda 100+ Türk API'si listelenmiştir:
- Akbank Developer Portal
- BTCTurk API
- Garanti Bankası API
- İETT API
- ve daha fazlası...

---

## 📂 Proje Yapısı

```
istanbul-degerleme-dashboard/
│
├── istanbul-degerleme-dashboard.html    # Ana dashboard dosyası
├── README.md                             # Bu dosya
└── assets/                               # (Opsiyonel) Görseller, ikonlar
```

---

## 🛠️ Teknolojiler

- **HTML5** - Yapı
- **CSS3** - Stil (TailwindCSS CDN)
- **JavaScript (Vanilla)** - İşlevsellik
- **Leaflet.js** - Harita
- **Chart.js** - Grafikler
- **Font Awesome** - İkonlar

---

## 📝 Özelleştirme

### API Anahtarlarını Ekleme

Dosyada şu satırları bulun ve kendi API anahtarlarınızı ekleyin:

```javascript
// TCMB EVDS
const EVDS_API_KEY = 'YOUR_EVDS_API_KEY';

// OpenWeatherMap
const WEATHER_API_KEY = 'YOUR_OPENWEATHER_API_KEY';

// CollectAPI
const COLLECT_API_KEY = 'YOUR_COLLECTAPI_KEY';
```

### Renk Teması Değiştirme

TailwindCSS sınıflarını düzenleyerek kolayca özelleştirebilirsiniz:

```html
<!-- Mavi tema yerine yeşil -->
<div class="bg-blue-500">  →  <div class="bg-green-500">
```

### Harita Merkezi Değiştirme

```javascript
// İstanbul yerine başka şehir
map = L.map('main-map').setView([41.0082, 28.9784], 11);
                                  ↓       ↓
                              [LAT,    LON]
```

---

## 🔄 Veri Yenileme Sıklıkları

Kod içinde ayarlanmış yenileme süreleri:

- **Döviz Kurları:** Her 1 dakika
- **Deprem Verileri:** Her 5 dakika
- **Hava Durumu:** Her 10 dakika
- **Haberler:** Her 15 dakika
- **Saat:** Her saniye

---

## 🌐 Canlı Demo

**Not:** GitHub Pages ile yayınlamak için:

1. Repository oluşturun
2. Settings → Pages
3. Source: `main` branch
4. Dosya adını `index.html` olarak değiştirin

---

## 🤝 Katkıda Bulunma

1. Fork edin
2. Feature branch oluşturun (`git checkout -b feature/AmazingFeature`)
3. Commit edin (`git commit -m 'Add some AmazingFeature'`)
4. Push edin (`git push origin feature/AmazingFeature`)
5. Pull Request açın

---

## 📜 Lisans

Bu proje MIT lisansı altındadır. Detaylar için `LICENSE` dosyasına bakın.

---

## 🙏 Teşekkürler

Bu proje aşağıdaki açık kaynak API'leri kullanmaktadır:

- [İBB Açık Veri Portalı](https://data.ibb.gov.tr)
- [TCMB EVDS](https://evds2.tcmb.gov.tr)
- [Kandilli Rasathanesi](http://www.koeri.boun.edu.tr)
- [OpenWeatherMap](https://openweathermap.org)
- [CollectAPI](https://collectapi.com)
- [Leaflet.js](https://leafletjs.com)
- [Chart.js](https://www.chartjs.org)

---

## 📧 İletişim

Proje Yöneticisi: [Your Name]

- Email: your.email@example.com
- GitHub: [@yourusername](https://github.com/yourusername)

---

## 🐛 Bilinen Sorunlar

- [ ] CORS problemi için proxy sunucu gerekebilir
- [ ] Bazı API'ler rate limiting uygular
- [ ] İBB SOAP servisleri gece 00:15'te kapatılır

---

## 🔮 Gelecek Özellikler

- [ ] PDF rapor oluşturma
- [ ] Excel export
- [ ] Kullanıcı giriş sistemi
- [ ] Özel alan hesaplama araçları
- [ ] Mobil uygulama versiyonu
- [ ] Tahmine dayalı fiyat analizi (ML)

---

## 💡 İpuçları

### CORS Hatası Alıyorsanız

Bazı API'ler CORS engellemesi yapabilir. Çözüm:

**1. Backend Proxy Kullanın:**
```javascript
const proxyUrl = 'https://cors-anywhere.herokuapp.com/';
const apiUrl = 'API_URL';

fetch(proxyUrl + apiUrl)
  .then(response => response.json())
  .then(data => console.log(data));
```

**2. Kendi Proxy'nizi Oluşturun:**
```javascript
// Node.js Express örneği
const express = require('express');
const cors = require('cors');
const axios = require('axios');

const app = express();
app.use(cors());

app.get('/api/proxy', async (req, res) => {
  const { url } = req.query;
  const response = await axios.get(url);
  res.json(response.data);
});

app.listen(3000);
```

### API Rate Limit Yönetimi

```javascript
// Rate limiting için basit cache
const cache = {};
const CACHE_TIME = 60000; // 1 dakika

async function fetchWithCache(url) {
  const now = Date.now();
  
  if (cache[url] && (now - cache[url].time) < CACHE_TIME) {
    return cache[url].data;
  }
  
  const response = await fetch(url);
  const data = await response.json();
  
  cache[url] = { data, time: now };
  return data;
}
```

---

**🎉 Dashboard'u kullanırken keyif alın!**

**⭐ Beğendiyseniz yıldız vermeyi unutmayın!**
