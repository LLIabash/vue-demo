<template>
    <div class="forecast-card">
        <h2 class="forecast-title">Прогноз на неделю</h2>
        <div class="forecast-grid">
            <div v-for="(day, index) in forecast" :key="index" class="forecast-day">
                <p class="day-name">{{ formatDay(day.date) }}</p>
                <img :src="day.day.condition.icon" :alt="day.day.condition.text" class="weather-icon" />
                <p class="day-temp">{{ Math.round(day.day.avgtemp_c) }}°C</p>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'

const props = defineProps({
    city: Object,
})

const forecast = ref([])

const API_KEY = 'a47e843f73b544f6bf6194727242710'

const fetchWeeklyForecast = async (cityName) => {
    try {
        const response = await fetch(
            `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=${cityName}&days=7`
        )
        const data = await response.json()
        forecast.value = data.forecast.forecastday
    } catch (error) {
        console.error('Error fetching forecast data:', error.message)
    }
}

const formatDay = (date) => {
    return new Date(date).toLocaleDateString('ru-RU', { weekday: 'short' })
}

watch(
    () => props.city,
    () => fetchWeeklyForecast(props.city.id),
    { immediate: true }
)
</script>

<style scoped>
.forecast-card {
    overflow-x: auto;
}

.forecast-title {
    font-size: 1.5rem;
    font-weight: 600;
    margin-bottom: 1rem;
}

.forecast-grid {
    display: flex;
    gap: 1rem;
}

.forecast-day {
    display: flex;
    flex-direction: column;
    align-items: center;
    min-width: 4rem;
}

.day-name {
    font-size: 0.875rem;
    font-weight: 500;
    margin-bottom: 0.5rem;
}

.weather-icon {
    width: 2.5rem;
    height: 2.5rem;
    margin-bottom: 0.5rem;
}

.day-temp {
    font-size: 1rem;
    font-weight: 600;
}

@media (prefers-color-scheme: dark) {
    .forecast-card {
        color: #f1f5f9;
    }
}
</style>