<template>
  <div 
    class="weather-app" 
    :class="[
      'min-vh-100 d-flex align-items-center justify-content-center p-4',
      isDarkMode ? 'dark-theme' : 'light-theme'
    ]"
  >
    <div 
      class="weather-card position-relative overflow-hidden"
      :class="[
        'card shadow-lg border-0 rounded-4 p-4',
        isDarkMode ? 'bg-dark-glass text-light' : 'bg-light-glass text-dark'
      ]"
    >
      <div class="glass-overlay"></div>
      
      <!-- Header -->
      <div class="d-flex justify-content-between align-items-center mb-4 position-relative z-index-2">
        <div class="d-flex align-items-center gap-3">
          <div class="app-icon">
            <i class="bi bi-cloud-sun-fill"></i>
          </div>
          <h1 class="app-title mb-0">Ob-havo</h1>
        </div>
        
        <div class="d-flex gap-2">
          <button
            @click="fetchWeatherData"
            :disabled="isLoading"
            class="control-btn btn rounded-3 border-0"
            :title="'Yangilash'"
          >
            <i class="bi bi-arrow-clockwise"></i>
          </button>
          
          <button
            @click="toggleTheme"
            class="control-btn btn rounded-3 border-0"
            :title="isDarkMode ? 'Yorug\' rejim' : 'Qorong\' rejim'"
          >
            <i :class="isDarkMode ? 'bi bi-sun-fill' : 'bi bi-moon-stars-fill'"></i>
          </button>
        </div>
      </div>

      <!-- Error State -->
      <div v-if="error" class="text-center py-5 position-relative z-index-2">
        <div class="error-icon mb-3">
          <i class="bi bi-exclamation-triangle-fill"></i>
        </div>
        <p class="error-message mb-4">{{ error }}</p>
        <button @click="fetchWeatherData" class="btn btn-primary rounded-3 px-4">
          <i class="bi bi-arrow-clockwise me-2"></i>
          Qayta urinish
        </button>
      </div>

      <!-- Loading State -->
      <div v-else-if="isLoading" class="text-center py-5 position-relative z-index-2">
        <div class="loading-icon mb-3">
          <i class="bi bi-cloud-sun-fill"></i>
        </div>
        <p class="mb-0">Ma'lumotlar yuklanmoqda...</p>
      </div>

      <!-- Weather Data -->
      <template v-else-if="weatherData">
        <div class="position-relative z-index-2">
          <!-- Main Weather Display -->
          <div class="text-center mb-4">
            <div class="d-flex align-items-center justify-content-center gap-2 mb-3">
              <i class="bi bi-geo-alt-fill location-icon"></i>
              <span class="location-text">
                {{ weatherData.name }}, {{ weatherData.sys?.country }}
              </span>
            </div>

            <div class="d-flex align-items-center justify-content-center gap-4 mb-3 flex-wrap">
              <div class="weather-icon-main">
                <i :class="getWeatherIcon(weatherData.weather?.[0]?.icon)"></i>
              </div>
              
              <div class="temperature-main">
                <span class="temperature-value">{{ Math.round(weatherData.main?.temp || 0) }}</span>
                <span class="temperature-unit">°C</span>
              </div>
            </div>

            <div class="weather-description text-capitalize mb-2">
              {{ weatherData.weather?.[0]?.description }}
            </div>
            
            <div class="feels-like">
              His qilish: {{ Math.round(weatherData.main?.feels_like || 0) }}°C
            </div>
          </div>

          <!-- Weather Details Grid -->
          <div class="row g-3 mb-4">
            <div class="col-6">
              <div class="detail-card h-100">
                <i class="bi bi-droplet-half detail-icon text-info"></i>
                <div class="detail-label">Namlik</div>
                <div class="detail-value">{{ weatherData.main?.humidity }}%</div>
              </div>
            </div>
            
            <div class="col-6">
              <div class="detail-card h-100">
                <i class="bi bi-wind detail-icon text-success"></i>
                <div class="detail-label">Shamol</div>
                <div class="detail-value">{{ weatherData.wind?.speed }} m/s</div>
              </div>
            </div>
          </div>

          <!-- Sun Times -->
          <div v-if="weatherData.sys" class="sun-times-card">
            <div class="d-flex justify-content-between align-items-center">
              <div class="sun-time">
                <i class="bi bi-sunrise-fill text-warning me-2"></i>
                <div>
                  <small class="d-block opacity-75">Quyosh chiqishi</small>
                  <strong>{{ formatTime(weatherData.sys.sunrise) }}</strong>
                </div>
              </div>
              
              <div class="sun-time">
                <i class="bi bi-sunset-fill text-danger me-2"></i>
                <div>
                  <small class="d-block opacity-75">Quyosh botishi</small>
                  <strong>{{ formatTime(weatherData.sys.sunset) }}</strong>
                </div>
              </div>
            </div>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'

// Constants
const API_KEY = 'c7644c52f5deebcb8a641825fda94843'
const API_BASE_URL = 'https://api.openweathermap.org/data/2.5/weather'

// Reactive state
const weatherData = ref(null)
const isLoading = ref(true)
const error = ref(null)
const isDarkMode = ref(false)

// Computed properties
const currentTheme = computed(() => isDarkMode.value ? 'dark' : 'light')

// Methods
const fetchWeatherData = async () => {
  try {
    isLoading.value = true
    error.value = null

    const position = await getCurrentPosition()
    const { latitude, longitude } = position.coords

    const response = await axios.get(API_BASE_URL, {
      params: {
        lat: latitude,
        lon: longitude,
        appid: API_KEY,
        units: 'metric',
      },
      timeout: 10000
    })

    weatherData.value = response.data
    console.log('Weather data loaded:', response.data)
  } catch (err) {
    console.error('Weather fetch error:', err)
    error.value = getErrorMessage(err)
  } finally {
    isLoading.value = false
  }
}

const getCurrentPosition = () => {
  return new Promise((resolve, reject) => {
    if (!navigator.geolocation) {
      reject(new Error('Geolocation not supported'))
      return
    }

    navigator.geolocation.getCurrentPosition(
      resolve,
      reject,
      {
        enableHighAccuracy: true,
        timeout: 10000,
        maximumAge: 300000
      }
    )
  })
}

const getErrorMessage = (error) => {
  if (error.code === 1) {
    return 'Joylashuvga ruxsat berilmagan. Brauzer sozlamalarida joylashuvni yoqing.'
  } else if (error.code === 2) {
    return 'Joylashuv ma\'lumotlari mavjud emas.'
  } else if (error.code === 3) {
    return 'Joylashuvni aniqlash vaqti tugadi.'
  } else if (error.response?.status) {
    return `Server xatosi: ${error.response.status}`
  } else if (error.code === 'ECONNABORTED') {
    return 'So\'rov vaqti tugadi. Internet aloqasini tekshiring.'
  } else {
    return 'Ob-havo ma\'lumotlarini yuklashda xatolik yuz berdi.'
  }
}

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
  // Save theme preference (if you want to persist it)
  if (typeof localStorage !== 'undefined') {
    localStorage.setItem('weatherAppTheme', isDarkMode.value ? 'dark' : 'light')
  }
}

const formatTime = (timestamp) => {
  return new Date(timestamp * 1000).toLocaleTimeString('uz-UZ', {
    hour: '2-digit',
    minute: '2-digit'
  })
}

const getWeatherIcon = (iconCode) => {
  const iconMap = {
    '01d': 'bi bi-sun-fill',
    '01n': 'bi bi-moon-stars-fill',
    '02d': 'bi bi-cloud-sun-fill',
    '02n': 'bi bi-cloud-moon-fill',
    '03d': 'bi bi-cloud-fill',
    '03n': 'bi bi-cloud-fill',
    '04d': 'bi bi-clouds-fill',
    '04n': 'bi bi-clouds-fill',
    '09d': 'bi bi-cloud-drizzle-fill',
    '09n': 'bi bi-cloud-drizzle-fill',
    '10d': 'bi bi-cloud-rain-fill',
    '10n': 'bi bi-cloud-rain-fill',
    '11d': 'bi bi-cloud-lightning-fill',
    '11n': 'bi bi-cloud-lightning-fill',
    '13d': 'bi bi-cloud-snow-fill',
    '13n': 'bi bi-cloud-snow-fill',
    '50d': 'bi bi-cloud-haze-fill',
    '50n': 'bi bi-cloud-haze-fill'
  }
  return iconMap[iconCode] || 'bi bi-cloud-fill'
}

// Lifecycle hooks
onMounted(async () => {
  // Load saved theme preference
  if (typeof localStorage !== 'undefined') {
    const savedTheme = localStorage.getItem('weatherAppTheme')
    if (savedTheme) {
      isDarkMode.value = savedTheme === 'dark'
    } else {
      // Detect system preference
      isDarkMode.value = window.matchMedia?.('(prefers-color-scheme: dark)')?.matches || false
    }
  }

  // Initial data load
  await fetchWeatherData()
})
</script>


<style scoped>
/* Theme Backgrounds */
.light-theme {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.dark-theme {
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
}

/* Glass Effects */
.weather-card {
  min-width: 380px;
  max-width: 420px;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
}

.bg-light-glass {
  background: rgba(255, 255, 255, 0.15);
}

.bg-dark-glass {
  background: rgba(0, 0, 0, 0.25);
}

.glass-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(45deg, rgba(255,255,255,0.1) 0%, transparent 100%);
  pointer-events: none;
  z-index: 1;
}

/* Header Styles */
.app-icon {
  width: 50px;
  height: 50px;
  background: linear-gradient(45deg, #4facfe 0%, #00f2fe 100%);
  border-radius: 15px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  color: white;
}

.app-title {
  font-size: 28px;
  font-weight: 700;
  background: linear-gradient(45deg, #4facfe, #00f2fe);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.control-btn {
  width: 45px;
  height: 45px;
  background: rgba(255, 255, 255, 0.15);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
}

.control-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Weather Display */
.location-icon {
  color: #4facfe;
  font-size: 16px;
}

.location-text {
  font-size: 16px;
  font-weight: 500;
}

.weather-icon-main {
  width: 80px;
  height: 80px;
  background: linear-gradient(45deg, #ff6b6b, #feca57);
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40px;
  color: white;
}

.temperature-main {
  display: flex;
  align-items: flex-start;
  line-height: 1;
}

.temperature-value {
  font-size: 64px;
  font-weight: 300;
}

.temperature-unit {
  font-size: 24px;
  opacity: 0.8;
  margin-top: 8px;
}

.weather-description {
  font-size: 18px;
  font-weight: 500;
  opacity: 0.9;
}

.feels-like {
  font-size: 14px;
  opacity: 0.8;
}

/* Detail Cards */
.detail-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 20px;
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
}

.detail-icon {
  font-size: 24px;
  margin-bottom: 8px;
}

.detail-label {
  font-size: 12px;
  opacity: 0.8;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 4px;
}

.detail-value {
  font-size: 18px;
  font-weight: 600;
}

/* Sun Times */
.sun-times-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 20px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
}

.sun-time {
  display: flex;
  align-items: center;
  font-size: 14px;
}

/* Error & Loading */
.loading-icon {
  width: 60px;
  height: 60px;
  background: linear-gradient(45deg, #4facfe 0%, #00f2fe 100%);
  border-radius: 15px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 28px;
  color: white;
}

.error-icon {
  font-size: 60px;
  color: #ff6b6b;
}

.error-message {
  font-size: 16px;
  opacity: 0.9;
}

.z-index-2 {
  z-index: 2;
}

/* Responsive */
@media (max-width: 768px) {
  .weather-card {
    min-width: 95vw;
    max-width: 95vw;
  }

  .app-title {
    font-size: 24px;
  }

  .app-icon {
    width: 45px;
    height: 45px;
    font-size: 20px;
  }

  .temperature-value {
    font-size: 48px;
  }

  .weather-icon-main {
    width: 70px;
    height: 70px;
    font-size: 35px;
  }
}

@media (max-width: 480px) {
  .weather-card {
    min-width: 98vw;
    max-width: 98vw;
    padding: 1.5rem !important;
  }

  .temperature-value {
    font-size: 40px;
  }

  .sun-time {
    flex-direction: column;
    text-align: center;
    gap: 0.5rem;
  }
}
</style>