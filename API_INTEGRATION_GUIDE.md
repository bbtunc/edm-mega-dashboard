# API Entegrasyon Kılavuzu

Bu doküman, dashboard'da kullanılan tüm API'lerin gerçek entegrasyonu için adım adım rehberdir.

## 📋 İçindekiler

1. [İBB Açık Veri API](#1-ibb-acik-veri-api)
2. [TCMB EVDS API](#2-tcmb-evds-api)
3. [Deprem API](#3-deprem-api)
4. [OpenWeatherMap API](#4-openweathermap-api)
5. [CollectAPI](#5-collectapi)
6. [Namaz Vakitleri API](#6-namaz-vakitleri-api)
7. [Haber API](#7-haber-api)

---

## 1. İBB Açık Veri API

### 🔗 Kayıt ve Kurulum

1. **Web sitesi:** https://data.ibb.gov.tr
2. **Giriş Yap** → **Kayıt Ol**
3. İBB Hesabınızla giriş yapın

### 📦 Kullanılabilir Veri Setleri

#### İSPARK Otoparkları

```javascript
async function fetchIsparkData() {
  const url = 'https://data.ibb.gov.tr/api/3/action/datastore_search';
  const params = {
    resource_id: 'f4f56e58-5210-4f17-b852-effe356a890c',
    limit: 1000
  };
  
  const queryString = new URLSearchParams(params).toString();
  
  try {
    const response = await fetch(`${url}?${queryString}`);
    const data = await response.json();
    
    if (data.success) {
      const parkingLots = data.result.records;
      
      parkingLots.forEach(park => {
        console.log({
          name: park.PARK_ADI,
          district: park.ILCE_ADI,
          capacity: park.PARK_KAPASITE,
          lat: park.ENLEM,
          lon: park.BOYLAM
        });
      });
      
      return parkingLots;
    }
  } catch (error) {
    console.error('İSPARK veri hatası:', error);
  }
}

// Kullanım
fetchIsparkData();
```

#### Metro İstasyonları

```javascript
async function fetchMetroStations() {
  // İBB Metro API
  const url = 'https://data.ibb.gov.tr/api/3/action/datastore_search';
  const params = {
    resource_id: 'METRO_RESOURCE_ID', // İBB'den alınacak
    limit: 500
  };
  
  const queryString = new URLSearchParams(params).toString();
  
  try {
    const response = await fetch(`${url}?${queryString}`);
    const data = await response.json();
    return data.result.records;
  } catch (error) {
    console.error('Metro veri hatası:', error);
  }
}
```

#### İETT Otobüs Konumları (GeoJSON)

```javascript
async function fetchBusLocations() {
  // Hakanatak'ın hazırladığı API
  const url = 'https://mekansal.herokuapp.com/api/filo';
  
  try {
    const response = await fetch(url);
    const geoJsonData = await response.json();
    
    // Leaflet'e ekle
    L.geoJSON(geoJsonData, {
      pointToLayer: function (feature, latlng) {
        return L.marker(latlng, {
          icon: L.divIcon({
            html: '<i class="fas fa-bus"></i>',
            className: 'bus-icon'
          })
        });
      }
    }).addTo(map);
    
  } catch (error) {
    console.error('Otobüs veri hatası:', error);
  }
}
```

---

## 2. TCMB EVDS API

### 🔑 API Anahtarı Alma

1. **Kayıt:** https://evds2.tcmb.gov.tr
2. Giriş Yap → Profil
3. **API ANAHTARI** butonuna tıkla
4. Anahtarı kopyala

### 💱 Döviz Kurları

```javascript
class TCMBService {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://evds2.tcmb.gov.tr/service/evds';
  }
  
  async getExchangeRates(startDate, endDate) {
    const series = [
      'TP.DK.USD.S.YTL', // USD Satış
      'TP.DK.EUR.S.YTL', // EUR Satış
      'TP.DK.GBP.S.YTL'  // GBP Satış
    ].join('-');
    
    const url = `${this.baseUrl}/series=${series}&startDate=${startDate}&endDate=${endDate}&type=json`;
    
    try {
      const response = await fetch(url, {
        headers: {
          'key': this.apiKey
        }
      });
      
      const data = await response.json();
      
      // Son değerleri al
      const latestData = data.items[data.items.length - 1];
      
      return {
        USD: parseFloat(latestData.TP_DK_USD_S_YTL),
        EUR: parseFloat(latestData.TP_DK_EUR_S_YTL),
        GBP: parseFloat(latestData.TP_DK_GBP_S_YTL),
        date: latestData.Tarih
      };
      
    } catch (error) {
      console.error('TCMB API hatası:', error);
      throw error;
    }
  }
  
  async getGoldPrices() {
    const today = new Date();
    const startDate = this.formatDate(new Date(today.setMonth(today.getMonth() - 1)));
    const endDate = this.formatDate(new Date());
    
    const url = `${this.baseUrl}/series=TP.AB.A01&startDate=${startDate}&endDate=${endDate}&type=json`;
    
    try {
      const response = await fetch(url, {
        headers: {
          'key': this.apiKey
        }
      });
      
      const data = await response.json();
      const latest = data.items[data.items.length - 1];
      
      return {
        price: parseFloat(latest.TP_AB_A01),
        date: latest.Tarih
      };
      
    } catch (error) {
      console.error('Altın fiyat hatası:', error);
      throw error;
    }
  }
  
  formatDate(date) {
    const day = String(date.getDate()).padStart(2, '0');
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const year = date.getFullYear();
    return `${day}-${month}-${year}`;
  }
}

// Kullanım
const tcmb = new TCMBService('YOUR_API_KEY');

// Bugünün döviz kurları
const today = new Date();
const rates = await tcmb.getExchangeRates(
  tcmb.formatDate(today),
  tcmb.formatDate(today)
);

console.log('USD/TRY:', rates.USD);
console.log('EUR/TRY:', rates.EUR);

// Altın fiyatları
const gold = await tcmb.getGoldPrices();
console.log('Gram Altın:', gold.price);
```

### 📊 TÜFE (Enflasyon)

```javascript
async function getInflationData() {
  const tcmb = new TCMBService('YOUR_API_KEY');
  const series = 'TP.FE.OKTG01'; // TÜFE
  
  const today = new Date();
  const startDate = tcmb.formatDate(new Date(today.setFullYear(today.getFullYear() - 1)));
  const endDate = tcmb.formatDate(new Date());
  
  const url = `https://evds2.tcmb.gov.tr/service/evds/series=${series}&startDate=${startDate}&endDate=${endDate}&type=json`;
  
  const response = await fetch(url, {
    headers: { 'key': 'YOUR_API_KEY' }
  });
  
  const data = await response.json();
  return data.items;
}
```

---

## 3. Deprem API

### 🌍 AFAD API (Resmi)

```javascript
// AFAD için API key gerekli (https://deprem.afad.gov.tr/apidocs/)
async function fetchEarthquakesAFAD(apiKey) {
  const url = 'https://deprem.afad.gov.tr/apiv2/event/filter';
  
  const params = {
    start: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString(), // Son 24 saat
    end: new Date().toISOString(),
    minmag: 2.0 // Minimum büyüklük
  };
  
  try {
    const response = await fetch(url + '?' + new URLSearchParams(params), {
      headers: {
        'Authorization': `Bearer ${apiKey}`
      }
    });
    
    const data = await response.json();
    return data.data;
    
  } catch (error) {
    console.error('AFAD API hatası:', error);
  }
}
```

### 🔄 Alternatif - Orhanaydogdu API (Ücretsiz)

```javascript
async function fetchEarthquakesKandilli() {
  const url = 'https://api.orhanaydogdu.com.tr/deprem/kandilli/live';
  
  try {
    const response = await fetch(url);
    const data = await response.json();
    
    if (data.status) {
      const earthquakes = data.result.map(eq => ({
        location: eq.title,
        magnitude: parseFloat(eq.mag),
        depth: parseFloat(eq.depth),
        time: eq.date,
        lat: parseFloat(eq.geojson.coordinates[1]),
        lon: parseFloat(eq.geojson.coordinates[0])
      }));
      
      return earthquakes;
    }
    
  } catch (error) {
    console.error('Deprem API hatası:', error);
  }
}

// Kullanım ve haritaya ekleme
async function displayEarthquakes() {
  const earthquakes = await fetchEarthquakesKandilli();
  
  earthquakes.forEach(eq => {
    const color = eq.magnitude >= 4 ? '#ef4444' : 
                  eq.magnitude >= 3 ? '#f59e0b' : '#eab308';
    
    L.circle([eq.lat, eq.lon], {
      color: color,
      fillColor: color,
      fillOpacity: 0.3,
      radius: eq.magnitude * 5000
    })
    .bindPopup(`
      <strong>${eq.magnitude} ML</strong><br>
      ${eq.location}<br>
      Derinlik: ${eq.depth} km<br>
      ${new Date(eq.time).toLocaleString('tr-TR')}
    `)
    .addTo(map);
  });
}
```

---

## 4. OpenWeatherMap API

### 🌤️ Kayıt ve Kurulum

1. **Kayıt:** https://openweathermap.org/api
2. Free Plan: 60 calls/minute
3. API Key: Account → API keys

### 📡 Hava Durumu

```javascript
class WeatherService {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://api.openweathermap.org/data/2.5';
  }
  
  async getCurrentWeather(city = 'Istanbul') {
    const url = `${this.baseUrl}/weather?q=${city}&appid=${this.apiKey}&units=metric&lang=tr`;
    
    try {
      const response = await fetch(url);
      const data = await response.json();
      
      return {
        temp: Math.round(data.main.temp),
        feelsLike: Math.round(data.main.feels_like),
        humidity: data.main.humidity,
        pressure: data.main.pressure,
        windSpeed: (data.wind.speed * 3.6).toFixed(1), // m/s → km/h
        windDirection: data.wind.deg,
        description: data.weather[0].description,
        icon: data.weather[0].icon,
        clouds: data.main.clouds
      };
      
    } catch (error) {
      console.error('Hava durumu hatası:', error);
      throw error;
    }
  }
  
  async getForecast(city = 'Istanbul', days = 5) {
    const url = `${this.baseUrl}/forecast?q=${city}&appid=${this.apiKey}&units=metric&lang=tr`;
    
    try {
      const response = await fetch(url);
      const data = await response.json();
      
      // 3 saatlik periyotlar halinde tahmin
      return data.list.slice(0, days * 8).map(item => ({
        datetime: new Date(item.dt * 1000),
        temp: Math.round(item.main.temp),
        description: item.weather[0].description,
        icon: item.weather[0].icon
      }));
      
    } catch (error) {
      console.error('Tahmin hatası:', error);
      throw error;
    }
  }
  
  getWeatherIcon(iconCode) {
    // OpenWeatherMap icon'ları Font Awesome'a çevir
    const iconMap = {
      '01d': 'fa-sun',
      '01n': 'fa-moon',
      '02d': 'fa-cloud-sun',
      '02n': 'fa-cloud-moon',
      '03d': 'fa-cloud',
      '03n': 'fa-cloud',
      '04d': 'fa-cloud',
      '04n': 'fa-cloud',
      '09d': 'fa-cloud-showers-heavy',
      '09n': 'fa-cloud-showers-heavy',
      '10d': 'fa-cloud-sun-rain',
      '10n': 'fa-cloud-moon-rain',
      '11d': 'fa-bolt',
      '11n': 'fa-bolt',
      '13d': 'fa-snowflake',
      '13n': 'fa-snowflake',
      '50d': 'fa-smog',
      '50n': 'fa-smog'
    };
    
    return iconMap[iconCode] || 'fa-cloud';
  }
}

// Kullanım
const weather = new WeatherService('YOUR_API_KEY');

const current = await weather.getCurrentWeather('Istanbul');
console.log(`Sıcaklık: ${current.temp}°C`);
console.log(`Hissedilen: ${current.feelsLike}°C`);
console.log(`Durum: ${current.description}`);

// Dashboard'a ekle
document.getElementById('temp').textContent = `${current.temp}°C`;
document.getElementById('weather-desc').textContent = current.description;
document.getElementById('humidity').innerHTML = 
  `<i class="fas fa-tint mr-1"></i>Nem: ${current.humidity}%`;
document.getElementById('weather-icon').className = 
  `fas ${weather.getWeatherIcon(current.icon)} text-5xl opacity-75`;
```

---

## 5. CollectAPI

### 🔑 Kayıt

1. **Web sitesi:** https://collectapi.com
2. Kayıt ol ve API key al
3. Dashboard'dan istediğin servisleri aktifleştir

### 💊 Nöbetçi Eczaneler

```javascript
async function fetchPharmacies(city = 'Istanbul', district = 'Kadikoy') {
  const url = 'https://api.collectapi.com/health/dutyPharmacy';
  const API_KEY = 'YOUR_COLLECTAPI_KEY';
  
  const params = new URLSearchParams({
    il: city,
    ilce: district
  });
  
  try {
    const response = await fetch(`${url}?${params}`, {
      method: 'GET',
      headers: {
        'authorization': `apikey ${API_KEY}`,
        'content-type': 'application/json'
      }
    });
    
    const data = await response.json();
    
    if (data.success) {
      return data.result.map(pharmacy => ({
        name: pharmacy.name,
        address: pharmacy.address,
        phone: pharmacy.phone,
        lat: pharmacy.loc ? parseFloat(pharmacy.loc.split(',')[0]) : null,
        lon: pharmacy.loc ? parseFloat(pharmacy.loc.split(',')[1]) : null
      }));
    }
    
  } catch (error) {
    console.error('Eczane API hatası:', error);
  }
}

// Haritaya ekle
async function displayPharmacies() {
  const pharmacies = await fetchPharmacies('Istanbul', 'Kadikoy');
  
  pharmacies.forEach(pharmacy => {
    if (pharmacy.lat && pharmacy.lon) {
      L.marker([pharmacy.lat, pharmacy.lon], {
        icon: L.divIcon({
          html: '<i class="fas fa-pills"></i>',
          className: 'pharmacy-marker'
        })
      })
      .bindPopup(`
        <strong>${pharmacy.name}</strong><br>
        ${pharmacy.address}<br>
        Tel: ${pharmacy.phone}
      `)
      .addTo(map);
    }
  });
}
```

### 📰 Haber API

```javascript
async function fetchNews(country = 'tr', tag = 'general') {
  const url = 'https://api.collectapi.com/news/getNews';
  const API_KEY = 'YOUR_COLLECTAPI_KEY';
  
  const params = new URLSearchParams({
    country: country,
    tag: tag
  });
  
  try {
    const response = await fetch(`${url}?${params}`, {
      headers: {
        'authorization': `apikey ${API_KEY}`,
        'content-type': 'application/json'
      }
    });
    
    const data = await response.json();
    
    if (data.success) {
      return data.result.slice(0, 10).map(news => ({
        title: news.name,
        description: news.description,
        url: news.url,
        image: news.image,
        source: news.source,
        date: new Date(news.key)
      }));
    }
    
  } catch (error) {
    console.error('Haber API hatası:', error);
  }
}
```

---

## 6. Namaz Vakitleri API

### 🕌 CollectAPI ile

```javascript
async function fetchPrayerTimes(city = 'istanbul') {
  const url = 'https://api.collectapi.com/pray/all';
  const API_KEY = 'YOUR_COLLECTAPI_KEY';
  
  try {
    const response = await fetch(`${url}?data.city=${city}`, {
      headers: {
        'authorization': `apikey ${API_KEY}`,
        'content-type': 'application/json'
      }
    });
    
    const data = await response.json();
    
    if (data.success) {
      const today = data.result[0];
      return {
        fajr: today.fajr,
        sunrise: today.tulu,
        dhuhr: today.dhuhr,
        asr: today.asr,
        maghrib: today.maghrib,
        isha: today.isha,
        date: today.date
      };
    }
    
  } catch (error) {
    console.error('Namaz vakti hatası:', error);
  }
}

// Sıradaki vakti bul
function getNextPrayer(times) {
  const now = new Date();
  const currentTime = now.getHours() * 60 + now.getMinutes();
  
  const prayers = [
    { name: 'İmsak', time: times.fajr },
    { name: 'Güneş', time: times.sunrise },
    { name: 'Öğle', time: times.dhuhr },
    { name: 'İkindi', time: times.asr },
    { name: 'Akşam', time: times.maghrib },
    { name: 'Yatsı', time: times.isha }
  ];
  
  for (let prayer of prayers) {
    const [hours, minutes] = prayer.time.split(':').map(Number);
    const prayerTime = hours * 60 + minutes;
    
    if (prayerTime > currentTime) {
      return prayer;
    }
  }
  
  // Yarının imsak
  return { name: 'İmsak', time: times.fajr };
}

// Kullanım
const times = await fetchPrayerTimes('istanbul');
const next = getNextPrayer(times);

document.getElementById('prayer-time').innerHTML = `
  <i class="fas fa-mosque mr-2"></i>
  Sıradaki Vakit: <span class="font-semibold">${next.name} (${next.time})</span>
`;
```

---

## 7. Haber API

### 📰 NewsAPI (Alternatif)

```javascript
// NewsAPI - https://newsapi.org
async function fetchNewsAPI(query = 'emlak istanbul') {
  const API_KEY = 'YOUR_NEWSAPI_KEY';
  const url = 'https://newsapi.org/v2/everything';
  
  const params = new URLSearchParams({
    q: query,
    language: 'tr',
    sortBy: 'publishedAt',
    pageSize: 10,
    apiKey: API_KEY
  });
  
  try {
    const response = await fetch(`${url}?${params}`);
    const data = await response.json();
    
    return data.articles.map(article => ({
      title: article.title,
      description: article.description,
      url: article.url,
      image: article.urlToImage,
      source: article.source.name,
      date: new Date(article.publishedAt)
    }));
    
  } catch (error) {
    console.error('NewsAPI hatası:', error);
  }
}
```

---

## 🔒 Güvenlik Notları

### API Anahtarlarını Gizleme

**1. Environment Variables (Backend):**

```javascript
// .env dosyası
EVDS_API_KEY=your_evds_key
OPENWEATHER_API_KEY=your_weather_key
COLLECTAPI_KEY=your_collect_key

// Node.js'de kullanım
require('dotenv').config();
const evdsKey = process.env.EVDS_API_KEY;
```

**2. Backend Proxy (Önerilen):**

```javascript
// server.js (Express)
const express = require('express');
const axios = require('axios');
require('dotenv').config();

const app = express();

app.get('/api/weather', async (req, res) => {
  const { city } = req.query;
  const API_KEY = process.env.OPENWEATHER_API_KEY;
  
  try {
    const response = await axios.get(
      `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric`
    );
    res.json(response.data);
  } catch (error) {
    res.status(500).json({ error: 'API hatası' });
  }
});

app.listen(3000);
```

**Frontend'den çağır:**
```javascript
const weather = await fetch('/api/weather?city=Istanbul');
const data = await weather.json();
```

---

## 🎯 Performans İpuçları

### 1. Caching

```javascript
class APICache {
  constructor(ttl = 60000) {
    this.cache = new Map();
    this.ttl = ttl;
  }
  
  set(key, value) {
    this.cache.set(key, {
      value,
      timestamp: Date.now()
    });
  }
  
  get(key) {
    const item = this.cache.get(key);
    
    if (!item) return null;
    
    if (Date.now() - item.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value;
  }
}

// Kullanım
const cache = new APICache(300000); // 5 dakika

async function getWeatherCached(city) {
  const cacheKey = `weather_${city}`;
  const cached = cache.get(cacheKey);
  
  if (cached) {
    return cached;
  }
  
  const data = await fetchWeather(city);
  cache.set(cacheKey, data);
  return data;
}
```

### 2. Rate Limiting

```javascript
class RateLimiter {
  constructor(maxRequests, timeWindow) {
    this.maxRequests = maxRequests;
    this.timeWindow = timeWindow;
    this.requests = [];
  }
  
  async throttle(fn) {
    const now = Date.now();
    this.requests = this.requests.filter(
      time => now - time < this.timeWindow
    );
    
    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = Math.min(...this.requests);
      const waitTime = this.timeWindow - (now - oldestRequest);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
    
    this.requests.push(Date.now());
    return fn();
  }
}

// 60 istek / dakika sınırı
const limiter = new RateLimiter(60, 60000);

async function fetchWithLimit() {
  return limiter.throttle(() => fetch('API_URL'));
}
```

---

**✅ Bu kılavuz ile tüm API'leri başarıyla entegre edebilirsiniz!**
