<template>
  <div class="weather-app">
    <div class="content-container">
      <h1 class="app-title">Прогноз погоды</h1>
      <div class="select-container">
        <input v-model="searchQuery" @input="searchCities" placeholder="Поиск города" class="city-search" />
        <select v-model="selectedCity" @change="handleCityChange" class="city-select">
          <option v-for="city in filteredCities" :key="city.id" :value="city">
            {{ city.name }}
          </option>
        </select>
        <div class="select-arrow">
          <svg class="arrow-icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20">
            <path d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z" />
          </svg>
        </div>
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
import { ref, onMounted, computed } from 'vue'
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
];

const weatherData = ref(null)
const selectedCity = ref(cities[0])
const searchQuery = ref('')

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
  fetchWeatherData(selectedCity.value.id)
}

const searchCities = () => {
  // Функция вызывается при вводе в поле поиска
  // Фильтрация городов происходит автоматически благодаря computed свойству
}

onMounted(() => {
  fetchWeatherData(selectedCity.value.id)
})
</script>

<style scoped>
.weather-app {
  min-height: 100vh;
  width: 100vw;
  background: linear-gradient(to bottom right, #e0f2fe, #bae6fd);
  color: #1e293b;
  display: flex;
  flex-direction: column;
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

.app-title {
  font-size: 2.5rem;
  font-weight: bold;
  margin-bottom: 1.5rem;
  text-align: center;
}

.select-container {
  position: relative;
  margin-bottom: 1.5rem;
}

.city-search {
  width: 100%;
  max-width: 300px;
  background-color: white;
  border: 1px solid #e2e8f0;
  border-radius: 0.5rem;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  margin-bottom: 0.5rem;
}

.city-select {
  width: 100%;
  max-width: 300px;
  background-color: white;
  border: 1px solid #e2e8f0;
  border-radius: 0.5rem;
  padding: 0.5rem 2.5rem 0.5rem 1rem;
  appearance: none;
  font-size: 1rem;
}

.select-arrow {
  position: absolute;
  right: 1rem;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
}

.arrow-icon {
  width: 1rem;
  height: 1rem;
  fill: currentColor;
}

.weather-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

.weather-card {
  background-color: white;
  border-radius: 0.5rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
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
  color: #64748b;
}

.weather-visual-card {
  display: flex;
  justify-content: center;
  align-items: center;
}

@media (max-width: 768px) {
  .weather-grid {
    grid-template-columns: 1fr;
  }
}

@media (prefers-color-scheme: dark) {
  .weather-app {
    background: linear-gradient(to bottom right, #1e293b, #0f172a);
    color: #f1f5f9;
  }

  .weather-card {
    background-color: #1e293b;
  }

  .city-select,
  .city-search {
    background-color: #334155;
    color: #f1f5f9;
    border-color: #475569;
  }

  .humidity {
    color: #94a3b8;
  }
}
</style>
