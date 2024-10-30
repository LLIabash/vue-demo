<template>
    <div class="weather-visual">
        <div class="sky-arc">
            <div :class="['celestial-body', { 'rainy': isRainy, 'cloudy': isCloudy }]" :style="celestialBodyStyle">
                <div v-if="isDaytime" class="sun"></div>
                <div v-else class="moon"></div>
            </div>
        </div>
        <div class="weather-conditions">
            <div v-if="isCloudy || isOvercast || isRainy || isSnowy" class="clouds">
                <div v-for="i in 5" :key="i" class="cloud"></div>
            </div>
            <div v-if="isSnowy" class="snow">
                <div v-for="i in 100" :key="i" class="snowflake"
                    :style="`--left-position: ${Math.random() * 110}; --animation-delay: ${Math.random() * 3};`"></div>
            </div>
            <div v-if="isMisty" class="mist"></div>
            <div v-if="isRainy" class="rain">
                <div v-for="i in 100" :key="i" class="raindrop"
                    :style="`--left-position: ${Math.random() * 110}; --animation-delay: ${Math.random() * 3};`">
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const props = defineProps({
    weatherData: Object,
})

// Обновляем now каждую минуту
const now = ref(new Date().getTime() / 1000);
onMounted(() => {
    setInterval(() => {
        now.value = new Date().getTime() / 1000;
    }, 60000);
});

const isDaytime = computed(() => {
    const sunrise = props.weatherData?.forecast?.forecastday[0]?.astro?.sunrise
    const sunset = props.weatherData?.forecast?.forecastday[0]?.astro?.sunset
    if (!sunrise || !sunset) return true
    const sunriseTime = new Date(props.weatherData.location.localtime.split(' ')[0] + ' ' + sunrise).getTime() / 1000
    const sunsetTime = new Date(props.weatherData.location.localtime.split(' ')[0] + ' ' + sunset).getTime() / 1000
    return now.value > sunriseTime && now.value < sunsetTime
})

const isRainy = computed(() => {
    const conditionText = props.weatherData?.current?.condition.text.toLowerCase();
    return conditionText.includes('rain') || conditionText.includes('drizzle');
});

const isSnowy = computed(() => {
    const conditionText = props.weatherData?.current?.condition.text.toLowerCase();
    return conditionText.includes('snow') || conditionText.includes('sleet');
});
const isCloudy = computed(() => props.weatherData?.current?.condition.text.toLowerCase().includes('cloud'))
const isOvercast = computed(() => props.weatherData?.current?.condition.text.toLowerCase().includes('overcast'))
const isMisty = computed(() => props.weatherData?.current?.condition.text.toLowerCase().includes('mist'))


const celestialBodyStyle = computed(() => {
    const sunrise = props.weatherData?.forecast?.forecastday[0]?.astro?.sunrise
    const sunset = props.weatherData?.forecast?.forecastday[0]?.astro?.sunset
    if (!sunrise || !sunset) return { left: '50%', bottom: '0' }

    const sunriseTime = new Date(props.weatherData.location.localtime.split(' ')[0] + ' ' + sunrise).getTime() / 1000
    const sunsetTime = new Date(props.weatherData.location.localtime.split(' ')[0] + ' ' + sunset).getTime() / 1000
    const dayDuration = sunsetTime - sunriseTime
    const timeSinceSunrise = now.value - sunriseTime

    let position = (timeSinceSunrise / dayDuration) * 100
    position = Math.max(0, Math.min(position, 100))

    const angle = (position / 100) * Math.PI
    const x = 50 + Math.sin(angle) * 37.5
    const y = Math.cos(angle) * 37.5

    return `{ left: ${x}%, bottom: ${y}% }`
})
</script>

<style scoped>
.weather-visual {
    position: relative;
    width: 100%;
    height: 300px;
    overflow: hidden;
    border-radius: 0.5rem;
    background: linear-gradient(to bottom, #87CEEB, #E0F6FF);
}

.sky-arc {
    position: absolute;
    width: 150%;
    height: 150%;
    left: -25%;
    bottom: -100%;
    border-radius: 50%;
    border: 2px solid rgba(255, 255, 255, 0.3);
    border-bottom: none;
}

.celestial-body {
    position: absolute;
    transform: translate(-50%, 50%);
    width: 2.5rem;
    height: 2.5rem;
    border-radius: 50%;
    transition: left 0.5s ease, bottom 0.5s ease;
    z-index: 10;
}

.sun {
    background: radial-gradient(circle, #FFD700 60%, #FFA500);
    box-shadow: 0 0 20px #FFD700;
}

.moon {
    background: radial-gradient(circle, #F0F0F0 60%, #C0C0C0);
    box-shadow: 0 0 20px #F0F0F0;
}

.weather-conditions {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
}

.clouds {
    position: absolute;
    width: 100%;
    height: 100%;
}

.cloud {
    position: absolute;
    background: #FFFFFF;
    border-radius: 50px;
    opacity: 0.8;
    animation: float 10s infinite ease-in-out;
}

.cloud::before,
.cloud::after {
    content: '';
    position: absolute;
    background: #FFFFFF;
    border-radius: 50%;
}

.cloud:nth-child(1) {
    width: 100px;
    height: 30px;
    top: 20%;
    left: 10%;
    animation-delay: 0s;
}

.cloud:nth-child(2) {
    width: 120px;
    height: 40px;
    top: 40%;
    left: 30%;
    animation-delay: -3s;
}

.cloud:nth-child(3) {
    width: 90px;
    height: 35px;
    top: 60%;
    left: 50%;
    animation-delay: -6s;
}

.cloud:nth-child(4) {
    width: 80px;
    height: 25px;
    top: 30%;
    left: 70%;
    animation-delay: -4s;
}

.cloud:nth-child(5) {
    width: 110px;
    height: 35px;
    top: 70%;
    left: 20%;
    animation-delay: -7s;
}

.cloud:nth-child(1)::before {
    width: 40px;
    height: 40px;
    top: -20px;
    left: 10px;
}

.cloud:nth-child(1)::after {
    width: 60px;
    height: 60px;
    top: -30px;
    right: 15px;
}

.cloud:nth-child(2)::before {
    width: 50px;
    height: 50px;
    top: -25px;
    left: 15px;
}

.cloud:nth-child(2)::after {
    width: 70px;
    height: 70px;
    top: -35px;
    right: 20px;
}

.cloud:nth-child(3)::before {
    width: 45px;
    height: 45px;
    top: -22px;
    left: 12px;
}

.cloud:nth-child(3)::after {
    width: 65px;
    height: 65px;
    top: -32px;
    right: 18px;
}

.cloud:nth-child(4)::before {
    width: 35px;
    height: 35px;
    top: -18px;
    left: 10px;
}

.cloud:nth-child(4)::after {
    width: 55px;
    height: 55px;
    top: -28px;
    right: 15px;
}

.cloud:nth-child(5)::before {
    width: 45px;
    height: 45px;
    top: -22px;
    left: 12px;
}

.cloud:nth-child(5)::after {
    width: 65px;
    height: 65px;
    top: -32px;
    right: 18px;
}

@keyframes float {

    0%,
    100% {
        transform: translateX(0);
    }

    50% {
        transform: translateX(20px);
    }
}

.rain {
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: 5;
}

.raindrop {
    position: absolute;
    width: 2px;
    height: 10px;
    background: linear-gradient(to bottom, rgba(135, 206, 235, 0.8), rgba(135, 206, 235, 0.4));
    border-radius: 20% 20% 50% 50%;
    opacity: 0;
    /* Начальная прозрачность */
    animation: fall 1.2s linear infinite;
    left: calc(var(--left-position) * 1%);
    animation-delay: calc(var(--animation-delay) * 0.5s);
}

@keyframes fall {
    0% {
        transform: translateY(-20px);
        opacity: 0;
        /* Начальная прозрачность */
    }

    20% {
        opacity: 0.6;
        /* Появляется наполовину через 20% пути */
    }

    100% {
        transform: translateY(300px);
        opacity: 0;
        /* Пропадает после достижения конца */
    }
}

.raindrop:nth-child(2n) {
    animation-delay: -0.5s;
}

.raindrop:nth-child(3n) {
    animation-delay: -0.7s;
}

.raindrop:nth-child(4n) {
    animation-delay: -0.3s;
}

.raindrop:nth-child(5n) {
    animation-delay: -0.9s;
}

.celestial-body.rainy::before,
.celestial-body.cloudy::before {
    content: '';
    position: absolute;
    top: -25%;
    left: -25%;
    right: -25%;
    bottom: -25%;
    border-radius: 50%;
    z-index: 20;
}

.celestial-body.rainy::before {
    background: radial-gradient(circle, transparent 30%, rgba(135, 206, 235, 0.3) 70%);
}

.celestial-body.cloudy::before {
    background: radial-gradient(circle, transparent 40%, rgba(255, 255, 255, 0.4) 70%);
}

@media (prefers-color-scheme: dark) {
    .weather-visual {
        background: linear-gradient(to bottom, #1a365d, #2d3748);
    }

    .cloud {
        background: #E2E8F0;
    }

    .cloud::before,
    .cloud::after {
        background: #E2E8F0;
    }

    .raindrop {
        background: #60A5FA;
    }

    .celestial-body.cloudy::before {
        background: radial-gradient(circle, transparent 40%, rgba(255, 255, 255, 0.2) 70%);
    }
}

.snow {
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: 5;
}

.snowflake {
    position: absolute;
    width: 5px;
    height: 5px;
    background: #FFFFFF;
    border-radius: 50%;
    opacity: 0;
    /* Начальная прозрачность */
    animation: snowfall 3s linear infinite;
    left: calc(var(--left-position) * 1%);
    animation-delay: calc(var(--animation-delay) * 2s);
}

@keyframes snowfall {
    0% {
        transform: translateY(-10px);
        opacity: 0;
        /* Начальная прозрачность */
    }

    20% {
        opacity: 0.5;
        /* Появляется наполовину через 20% пути */
    }

    100% {
        transform: translateY(300px);
        opacity: 0.8;
        /* Окончательная прозрачность */
    }
}

.mist {
    position: absolute;
    width: 100%;
    height: 100%;
    background: rgba(200, 200, 200, 0.5);
    z-index: 6;
    backdrop-filter: blur(3px);
}
</style>