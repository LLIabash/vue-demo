<template>
  <div :class="['weather-app', theme]">
    <div class="content-container">
      <header class="app-header">
        <h1 class="app-title">Прогноз погоды</h1>
        <button @click="toggleTheme" class="theme-toggle"
          :aria-label="theme === 'light' ? 'Switch to dark theme' : 'Switch to light theme'">
          <SunIcon v-if="theme === 'dark'" class="theme-icon" />
          <MoonIcon v-else class="theme-icon" />
        </button>
      </header>

      <div class="select-container">
        <input v-model="searchQuery" @input="searchCities" @change="handleCityChange" list="city-list"
          placeholder="Поиск или выбор города" class="city-search" />
        <datalist id="city-list">
          <option v-for="city in filteredCities" :key="city.id" :value="city.name">
            {{ city.name }}
          </option>
        </datalist>
      </div>

      <div class="weather-grid">
        <div class="weather-card current-weather">
          <h2 class="card-title">Текущая погода</h2>
          <div class="weather-info">
            <div>
              <p class="temperature">{{ weatherData?.current?.temp_c }}°C</p>
              <p class="condition">{{ weatherData?.current?.condition.text }}</p>
              <p class="humidity">Влажность: {{ weatherData?.current?.humidity }}%</p>
            </div>
          </div>
        </div>
        <div class="weather-card weather-visual-card">
          <WeatherVisual :weatherData="weatherData" />
        </div>
      </div>
      <div class="weather-card weekly-forecast">
        <WeeklyForecast :city="selectedCity" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import { SunIcon, MoonIcon } from 'lucide-vue-next'
import WeeklyForecast from './components/WeeklyForecast.vue'
import WeatherVisual from './components/WeatherVisual.vue'

const API_KEY = 'a47e843f73b544f6bf6194727242710'

const cities = [
  { name: 'Москва', id: 'Moscow' },
  { name: 'Санкт-Петербург', id: 'Saint Petersburg' },
  { name: 'Новосибирск', id: 'Novosibirsk' },
  { name: 'Екатеринбург', id: 'Yekaterinburg' },
  { name: 'Нижний Новгород', id: 'Nizhny Novgorod' },
  { name: 'Казань', id: 'Kazan' },
  { name: 'Челябинск', id: 'Chelyabinsk' },
  { name: 'Самара', id: 'Samara' },
  { name: 'Омск', id: 'Omsk' },
  { name: 'Ростов-на-Дону', id: 'Rostov-on-Don' },
  { name: 'Уфа', id: 'Ufa' },
  { name: 'Красноярск', id: 'Krasnoyarsk' },
  { name: 'Воронеж', id: 'Voronezh' },
  { name: 'Пермь', id: 'Perm' },
  { name: 'Волгоград', id: 'Volgograd' },
  { name: 'Краснодар', id: 'Krasnodar' },
  { name: 'Сочи', id: 'Sochi' },
  { name: 'Минск', id: 'Minsk' },
  { name: 'Киев', id: 'Kyiv' },
  { name: 'Нур-Султан', id: 'Nur-Sultan' },
  { name: 'Алматы', id: 'Almaty' }
]

const weatherData = ref(null)
const selectedCity = ref(cities[0])
const searchQuery = ref('')
const theme = ref('light')

const filteredCities = computed(() => {
  if (!searchQuery.value) return cities
  return cities.filter(city =>
    city.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

const fetchWeatherData = async (cityName) => {
  try {
    const response = await fetch(
      `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=${cityName}&days=1`
    )
    if (!response.ok) throw new Error(response.statusText)
    weatherData.value = await response.json()
  } catch (error) {
    console.error('Error fetching weather data:', error.message)
  }
}

const handleCityChange = () => {
  const selectedCityObj = cities.find(city => city.name === searchQuery.value)
  if (selectedCityObj) {
    selectedCity.value = selectedCityObj
    fetchWeatherData(selectedCityObj.id)
  }
}

const searchCities = () => {
  // Функция вызывается при вводе в поле поиска
  // Фильтрация городов происходит автоматически благодаря computed свойству
}

const toggleTheme = () => {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
  document.body.className = theme.value
}

onMounted(() => {
  fetchWeatherData(selectedCity.value.id)
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches
  theme.value = prefersDark ? 'dark' : 'light'
})

watch(theme, (newTheme) => {
  document.body.className = newTheme
})
</script>

<style scoped>
.weather-app {
  min-height: 100vh;
  width: 100%;
  transition: all 0.3s ease;
}

.weather-app.light {
  background: linear-gradient(135deg, #e0f2fe, #bae6fd);
  color: #1e293b;
}

.weather-app.dark {
  background: linear-gradient(135deg, #1e293b, #0f172a);
  color: #f1f5f9;
}

.content-container {
  max-width: 1200px;
  width: 100%;
  margin: 0 auto;
  padding: 2rem;
  flex-grow: 1;
  display: flex;
  flex-direction: column;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.app-title {
  font-size: 2.5rem;
  font-weight: bold;
}

.theme-toggle {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 50%;
  transition: background-color 0.3s ease;
}

.theme-toggle:hover {
  background-color: rgba(0, 0, 0, 0.1);
}

.dark .theme-toggle:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.select-container {
  position: relative;
  margin-bottom: 1.5rem;
}

.city-search {
  width: 100%;
  max-width: 300px;
  border-radius: 0.5rem;
  padding: 0.75rem 1rem;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.light .city-search {
  background-color: white;
  border: 1px solid #e2e8f0;
  color: #1e293b;
}

.dark .city-search {
  background-color: #334155;
  border: 1px solid #475569;
  color: #f1f5f9;
}


.weather-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

.weather-card {
  border-radius: 1rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
  transition: all 0.3s ease;
}

.light .weather-card {
  background-color: rgba(255, 255, 255, 0.8);
}

.dark .weather-card {
  background-color: rgba(30, 41, 59, 0.8);
}

.card-title {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 1rem;
}

.weather-info {
  display: flex;
  flex-direction: column;
}

.temperature {
  font-size: 3rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.condition {
  font-size: 1.25rem;
  margin-bottom: 0.5rem;
}

.humidity {
  opacity: 0.7;
}

.weather-visual-card {
  display: flex;
  justify-content: center;
  align-items: center;
}

.theme-icon {
  width: 1.5rem;
  height: 1.5rem;
}

.dark .theme-icon {
  color: white;
}

@media (max-width: 768px) {
  .weather-grid {
    grid-template-columns: 1fr;
  }
}
</style>
