<template>
    <div :class="['weather-visual', { 'day': isDaytime, 'night': !isDaytime }]">
        <div v-if="isDaytime" class="sun moving-sun"></div>
        <div v-else class="moon moving-moon"></div>
        <div class="weather-conditions">
            <div v-if="isCloudy || isOvercast || isRainy || isSnowy" class="clouds">
                <div v-for="i in 5" :key="i" class="cloud"></div>
            </div>
            <div v-if="isSnowy" class="snow">
                <div v-for="i in 100" :key="i" class="snowflake"
                    :style="`--left-position: ${Math.random() * 100}; --animation-delay: ${Math.random() * 3};`"></div>
            </div>
            <div v-if="isMisty" class="mist"></div>
            <div v-if="isRainy" class="rain">
                <div v-for="i in 100" :key="i" class="raindrop"
                    :style="`--left-position: ${Math.random() * 110}; --animation-delay: ${Math.random() * 3};`"></div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const props = defineProps({
    weatherData: Object,
    theme: String
})

const now = ref(new Date().getTime() / 1000);
onMounted(() => {
    setInterval(() => {
        now.value = new Date().getTime() / 1000;
    }, 60000);
});

const isDaytime = computed(() => {
    const currentHour = new Date(now.value * 1000).getUTCHours();
    return currentHour >= 6 && currentHour < 18;
});

// Остальные вычисляемые свойства
const isRainy = computed(() => {
    const conditionText = props.weatherData?.current?.condition.text.toLowerCase();
    return conditionText.includes('rain') || conditionText.includes('drizzle');
});

const isSnowy = computed(() => {
    const conditionText = props.weatherData?.current?.condition.text.toLowerCase();
    return conditionText.includes('snow') || conditionText.includes('sleet');
});

const isCloudy = computed(() => props.weatherData?.current?.condition.text.toLowerCase().includes('cloud'));
const isOvercast = computed(() => props.weatherData?.current?.condition.text.toLowerCase().includes('overcast'));
const isMisty = computed(() => props.weatherData?.current?.condition.text.toLowerCase().includes('mist'));

</script>

<style scoped>
.weather-visual {
    position: relative;
    width: 100%;
    height: 300px;
    overflow: hidden;
    border-radius: 0.5rem;
    transition: background 0.3s ease;
}

.weather-visual.day {
    background: linear-gradient(to bottom, #87CEEB, #E0F6FF);
    /* Голубое небо */
}

.weather-visual.night {
    background: linear-gradient(to bottom, #1a365d, #2d3748);
    /* Темное небо */
}

.sun,
.moon {
    position: absolute;
    top: 170px;
    left: 50%;
    transform: translate(-50%, -100%);
    width: 50px;
    height: 50px;
    border-radius: 50%;
}

/* Стили для солнца и луны */
.sun {
    background: radial-gradient(circle at center, #FFD700, #FFA500);
    box-shadow: 0 0 20px rgba(255, 165, 0, 0.8);
    animation: move-sun 20s linear infinite;
}

.moon {
    background: radial-gradient(circle at center, #C0C0C0, #808080);
    box-shadow: 0 0 20px rgba(128, 128, 128, 0.6);
    animation: move-moon 20s linear infinite;
}

@keyframes move-sun {
    0% {
        transform: translate(-50%, -100%) translateY(0);
    }

    50% {
        transform: translate(-50%, -100%) translateY(-100px);
        /* Максимальная высота */
    }

    100% {
        transform: translate(-50%, -100%) translateY(0);
    }
}

@keyframes move-moon {
    0% {
        transform: translate(-50%, -100%) translateY(0);
    }

    50% {
        transform: translate(-50%, -100%) translateY(-100px);
        /* Максимальная высота */
    }

    100% {
        transform: translate(-50%, -100%) translateY(0);
    }
}

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
    animation: fall 1.2s linear infinite;
    left: calc(var(--left-position) * 1%);
    animation-delay: calc(var(--animation-delay) * 0.5s);
}

@keyframes fall {
    0% {
        transform: translateY(-20px);
        opacity: 0;
    }

    20% {
        opacity: 0.6;
    }

    100% {
        transform: translateY(300px);
        opacity: 0;
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
}

.snow {
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: 10;
}

.snowflake {
    position: absolute;
    background: white;
    border-radius: 50%;
    opacity: 0;
    width: 5px;
    height: 5px;
    animation: fall-snow 3s linear infinite;
    left: calc(var(--left-position) * 1%);
    animation-delay: calc(var(--animation-delay) * 1.5s);
}

@keyframes fall-snow {
    0% {
        transform: translateY(-10px);
        opacity: 0;
    }

    15% {
        opacity: 0.8;
    }

    100% {
        transform: translateY(300px);
        opacity: 0;
    }
}

.mist {
    position: absolute;
    width: 130%;
    height: 100%;
    background: radial-gradient(circle at 50% 50%, rgba(200, 200, 200, 0.35), transparent 70%);
    border-radius: 50%;
    z-index: 6;
    backdrop-filter: blur(7px);
    animation: mistFlow 8s ease-in-out infinite;
    opacity: 0.5;
}

.mist::before,
.mist::after {
    content: '';
    position: absolute;
    width: 170%;
    height: 170%;
    background: radial-gradient(circle at 50% 50%, rgba(200, 200, 200, 0.3), transparent 80%);
    border-radius: 50%;
    opacity: 0.4;
    backdrop-filter: blur(10px);
    animation: mistFlow 12s ease-in-out infinite;
}

.mist::before {
    top: -15%;
    left: -10%;
    animation-direction: alternate;
}

.mist::after {
    top: 15%;
    left: 10%;
    animation-direction: alternate-reverse;
}

@keyframes mistFlow {
    0% {
        transform: translateY(0);
        opacity: 0.4;
    }

    50% {
        transform: translateY(-10px);
        /* Поднимаем туман вверх */
        opacity: 0.6;
    }

    100% {
        transform: translateY(0);
        /* Возвращаем туман обратно */
        opacity: 0.4;
    }
}
</style>
