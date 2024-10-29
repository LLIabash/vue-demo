<template>
    <div :class="{ 'dark': isDarkMode }" class="min-h-screen bg-white dark:bg-gray-900 transition-colors duration-300">
        <header class="sticky top-0 z-50 bg-white dark:bg-gray-800 shadow-sm transition-colors duration-300">
            <div class="container mx-auto px-4">
                <nav class="flex items-center justify-between h-16">
                    <router-link to="/"
                        class="text-2xl font-semibold text-gray-900 dark:text-white transition-colors duration-300">
                        Stylish
                    </router-link>
                    <div class="flex items-center space-x-4">
                        <button @click="toggleSettings"
                            class="text-gray-700 dark:text-gray-300 hover:text-gray-900 dark:hover:text-white transition-colors duration-300 rounded-full p-2">
                            <Settings class="w-6 h-6" />
                        </button>
                        <button @click="toggleCart"
                            class="text-gray-700 dark:text-gray-300 hover:text-gray-900 dark:hover:text-white transition-colors duration-300 relative rounded-full p-2">
                            <ShoppingBag class="w-6 h-6" :class="{ 'animate-bounce': cartItems.length > 0 }" />
                            <span v-if="cartItems.length > 0"
                                class="absolute -top-1 -right-1 bg-red-500 text-white text-xs rounded-full w-4 h-4 flex items-center justify-center">
                                {{ cartItems.length }}
                            </span>
                        </button>
                        <router-link to="/auth"
                            class="text-gray-700 dark:text-gray-300 hover:text-gray-900 dark:hover:text-white transition-colors duration-300 rounded-full p-2">
                            <User class="w-6 h-6" />
                        </router-link>
                    </div>
                </nav>
            </div>
        </header>

        <main class="flex-grow bg-white dark:bg-gray-900">
            <section class="py-6">
                <div class="container mx-auto px-4">
                    <div class="flex justify-center space-x-4 overflow-x-auto pb-4">
                        <button v-for="category in categories" :key="category" @click="selectCategory(category)"
                            class="px-4 py-2 text-sm font-medium rounded-full transition-colors duration-300"
                            :class="selectedCategory === category ? 'bg-gray-900 dark:bg-white text-white dark:text-gray-900' : 'bg-white dark:bg-gray-800 text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700'">
                            {{ t(category.toLowerCase()) }}
                        </button>
                    </div>
                </div>
            </section>

            <section class="py-12">
                <div class="container mx-auto px-4">
                    <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-6">
                        <div v-for="product in paginatedProducts" :key="product.id"
                            class="bg-white dark:bg-gray-800 rounded-lg overflow-hidden shadow-sm hover:shadow-md transition-colors duration-300"
                            @click="openProductModal(product)">
                            <img :src="product.image" :alt="t(product.name)" class="w-full h-48 object-cover" />
                            <div class="p-4">
                                <h3 class="text-sm font-medium text-gray-900 dark:text-white mb-1">{{ t(product.name) }}
                                </h3>
                                <p class="text-sm text-gray-500 dark:text-gray-400">{{ product.price }}</p>
                            </div>
                        </div>
                    </div>

                    <div class="mt-8 flex justify-center">
                        <nav class="inline-flex rounded-full shadow overflow-hidden">
                            <button v-for="pageNumber in totalPages" :key="pageNumber" @click="currentPage = pageNumber"
                                class="px-4 py-2 text-sm font-medium transition-colors duration-300 rounded-full"
                                :class="currentPage === pageNumber ? 'bg-gray-900 dark:bg-white text-white dark:text-gray-900' : 'bg-white dark:bg-gray-800 text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700'">
                                {{ pageNumber }}
                            </button>
                        </nav>
                    </div>
                </div>
            </section>
        </main>

        <footer class="bg-gray-100 dark:bg-gray-800 py-8 transition-colors duration-300">
            <div class="container mx-auto px-4 text-center text-sm text-gray-600 dark:text-gray-400">
                <p>{{ t('copyright', { year: new Date().getFullYear() }) }}</p>
            </div>
        </footer>

        <!-- Product Modal -->
        <div v-if="selectedProduct" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div class="bg-white dark:bg-gray-800 p-8 rounded-lg max-w-md w-full relative">
                <button @click="closeProductModal"
                    class="absolute top-4 right-4 text-gray-600 dark:text-gray-400 hover:text-gray-800 dark:hover:text-gray-200 transition-colors duration-300">
                    <X class="w-6 h-6" />
                </button>
                <h2 class="text-2xl font-bold mb-4 text-gray-900 dark:text-white">{{ t(selectedProduct.name) }}</h2>
                <img :src="selectedProduct.image" :alt="t(selectedProduct.name)"
                    class="w-full h-64 object-cover mb-4 rounded" />
                <p class="text-gray-600 dark:text-gray-400 mb-4">{{ selectedProduct.price }}</p>
                <div class="mb-4">
                    <h3 class="text-lg font-semibold mb-2 text-gray-900 dark:text-white">{{ t('characteristics') }}</h3>
                    <ul class="list-disc pl-5 text-gray-600 dark:text-gray-400">
                        <li v-for="(value, key) in selectedProduct.characteristics" :key="key">
                            {{ t(key) }}: {{ t(value) }}
                        </li>
                    </ul>
                </div>
                <div class="mb-4">
                    <h3 class="text-lg font-semibold mb-2 text-gray-900 dark:text-white">{{ t('description') }}</h3>
                    <p class="text-gray-600 dark:text-gray-400">{{ t(selectedProduct.description) }}</p>
                </div>
                <button @click="addToCart(selectedProduct)"
                    class="w-full bg-gray-900 dark:bg-white text-white dark:text-gray-900 px-4 py-2 rounded-full hover:bg-gray-800 dark:hover:bg-gray-100 transition-colors duration-300 mt-4">
                    {{ t('addToCart') }}
                </button>
            </div>
        </div>

        <!-- Cart Modal -->
        <div v-if="isCartOpen" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div class="bg-white dark:bg-gray-800 p-8 rounded-lg max-w-md w-full relative">
                <button @click="toggleCart"
                    class="absolute top-4 right-4 text-gray-600 dark:text-gray-400 hover:text-gray-800 dark:hover:text-gray-200 transition-colors duration-300">
                    <X class="w-6 h-6" />
                </button>
                <h2 class="text-2xl font-bold mb-4 text-gray-900 dark:text-white">{{ t('yourCart') }}</h2>
                <div v-if="cartItems.length === 0" class="text-gray-600 dark:text-gray-400">{{ t('emptyCart') }}</div>
                <div v-else>
                    <div v-for="item in cartItems" :key="item.id" class="flex justify-between items-center mb-4">
                        <div>
                            <h3 class="font-medium text-gray-900 dark:text-white">{{ t(item.name) }}</h3>
                            <p class="text-gray-600 dark:text-gray-400">{{ item.price }}</p>
                        </div>
                        <button @click="removeFromCart(item)"
                            class="text-red-500 hover:text-red-700 transition-colors duration-300 rounded-full p-2">
                            <Trash2 class="w-5 h-5" />
                        </button>
                    </div>
                    <button @click="checkout"
                        class="w-full bg-gray-900 dark:bg-white text-white dark:text-gray-900 px-4 py-2 rounded-full hover:bg-gray-800 dark:hover:bg-gray-100 transition-colors duration-300 mt-4">
                        {{ t('checkout') }}
                    </button>
                </div>
            </div>
        </div>

        <!-- Settings Modal -->
        <div v-if="isSettingsOpen" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div class="bg-white dark:bg-gray-800 p-8 rounded-lg max-w-md w-full relative">
                <button @click="toggleSettings"
                    class="absolute top-4 right-4 text-gray-600 dark:text-gray-400 hover:text-gray-800 dark:hover:text-gray-200 transition-colors duration-300">
                    <X class="w-6 h-6" />
                </button>
                <h2 class="text-2xl font-bold mb-4 text-gray-900 dark:text-white">{{ t('settings') }}</h2>
                <div class="mb-4">
                    <h3 class="text-lg font-medium mb-2 text-gray-900 dark:text-white">{{ t('theme') }}</h3>
                    <div class="flex space-x-4">
                        <button @click="setTheme('light')" class="px-4 py-2 rounded-full"
                            :class="isDarkMode ? 'bg-gray-200 text-gray-900' : 'bg-gray-900 text-white'">
                            {{ t('light') }}
                        </button>
                        <button @click="setTheme('dark')" class="px-4 py-2 rounded-full"
                            :class="isDarkMode ? 'bg-gray-900 text-white' : 'bg-gray-200 text-gray-900'">
                            {{ t('dark') }}
                        </button>
                    </div>
                </div>
                <div class="mb-4">
                    <h3 class="text-lg font-medium mb-2 text-gray-900 dark:text-white">{{ t('language') }}</h3>
                    <div class="flex space-x-4">
                        <button @click="setLanguage('en')" class="px-4 py-2 rounded-full"
                            :class="currentLanguage === 'en' ? 'bg-gray-900 text-white' : 'bg-gray-200 text-gray-900'">
                            English
                        </button>
                        <button @click="setLanguage('ru')" class="px-4 py-2 rounded-full"
                            :class="currentLanguage === 'ru' ? 'bg-gray-900 text-white' : 'bg-gray-200 text-gray-900'">
                            Русский
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { Search, ShoppingBag, Settings, User, X, Trash2 } from 'lucide-vue-next'

const categories = ['All', 'New', 'Tops', 'Bottoms', 'Dresses', 'Accessories', 'Shoes']
const selectedCategory = ref('All')
const currentPage = ref(1)
const itemsPerPage = 12
const selectedProduct = ref(null)
const isCartOpen = ref(false)
const isSettingsOpen = ref(false)
const cartItems = ref([])
const isDarkMode = ref(false)
const currentLanguage = ref('en')

const products = ref([
    {
        id: 1,
        name: 'classicWhiteTee',
        price: '$29.99',
        category: 'Tops',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'cotton100',
            fit: 'regular',
            care: 'machineWashCold'
        },
        description: 'classicWhiteTeeDescription'
    },
    {
        id: 2,
        name: 'slimFitJeans',
        price: '$59.99',
        category: 'Bottoms',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'cotton98Elastane2',
            fit: 'slim',
            rise: 'midRise'
        },
        description: 'slimFitJeansDescription'
    },
    {
        id: 3,
        name: 'floralSummerDress',
        price: '$49.99',
        category: 'Dresses',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'viscose100',
            fit: 'aLine',
            length: 'knee'
        },
        description: 'floralSummerDressDescription'
    },
    {
        id: 4,
        name: 'leatherBelt',
        price: '$34.99',
        category: 'Accessories',
        image: '/placeholder.svg?height=200&width=200',

        characteristics: {
            material: 'leather100',
            width: '3cm',
            buckle: 'metallic'
        },
        description: 'leatherBeltDescription'
    },
    {
        id: 5,
        name: 'runningShoes',
        price: '$89.99',
        category: 'Shoes',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'syntheticMesh',
            sole: 'rubber',
            type: 'athletic'
        },
        description: 'runningShoesDescription'
    },
    {
        id: 6,
        name: 'denimJacket',
        price: '$69.99',
        category: 'Tops',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'denim100',
            fit: 'regular',
            wash: 'medium'
        },
        description: 'denimJacketDescription'
    },
    {
        id: 7,
        name: 'pleatedSkirt',
        price: '$39.99',
        category: 'Bottoms',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'polyester100',
            fit: 'aLine',
            length: 'midi'
        },
        description: 'pleatedSkirtDescription'
    },
    {
        id: 8,
        name: 'silkScarf',
        price: '$29.99',
        category: 'Accessories',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'silk100',
            size: '90x90cm',
            pattern: 'floral'
        },
        description: 'silkScarfDescription'
    },
    {
        id: 9,
        name: 'casualSneakers',
        price: '$54.99',
        category: 'Shoes',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'canvas',
            sole: 'rubber',
            style: 'laceUp'
        },
        description: 'casualSneakersDescription'
    },
    {
        id: 10,
        name: 'floralBlouse',
        price: '$39.99',
        category: 'Tops',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'polyester100',
            fit: 'loose',
            pattern: 'floral'
        },
        description: 'floralBlouseDescription'
    },
    {
        id: 11,
        name: 'leatherHandbag',
        price: '$129.99',
        category: 'Accessories',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'leather100',
            size: 'medium',
            style: 'tote'
        },
        description: 'leatherHandbagDescription'
    },
    {
        id: 12,
        name: 'formalTrousers',
        price: '$69.99',
        category: 'Bottoms',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'wool',
            fit: 'straight',
            rise: 'high'
        },
        description: 'formalTrousersDescription'
    },
    {
        id: 13,
        name: 'stripedPoloShirt',
        price: '$34.99',
        category: 'Tops',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'cotton100',
            fit: 'regular',
            pattern: 'striped'
        },
        description: 'stripedPoloShirtDescription'
    },
    {
        id: 14,
        name: 'maxiDress',
        price: '$89.99',
        category: 'Dresses',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'cotton100',
            fit: 'loose',
            length: 'maxi'
        },
        description: 'maxiDressDescription'
    },
    {
        id: 15,
        name: 'leatherWallet',
        price: '$49.99',
        category: 'Accessories',
        image: '/placeholder.svg?height=200&width=200',
        characteristics: {
            material: 'leather100',
            size: 'compact',
            type: 'bifold'
        },
        description: 'leatherWalletDescription'
    }
])

const translations = {
    en: {
        brand: 'Stylish',
        all: 'All',
        new: 'New',
        tops: 'Tops',
        bottoms: 'Bottoms',
        dresses: 'Dresses',
        accessories: 'Accessories',
        shoes: 'Shoes',
        addToCart: 'Add to Cart',
        close: 'Close',
        yourCart: 'Your Cart',
        emptyCart: 'Your cart is empty',
        remove: 'Remove',
        settings: 'Settings',
        theme: 'Theme',
        light: 'Light',
        dark: 'Dark',
        language: 'Language',
        copyright: 'Copyright © {year} Stylish. All rights reserved.',
        characteristics: 'Characteristics',
        description: 'Description',
        material: 'Material',
        fit: 'Fit',
        care: 'Care',
        rise: 'Rise',
        length: 'Length',
        width: 'Width',
        buckle: 'Buckle',
        sole: 'Sole',
        type: 'Type',
        wash: 'Wash',
        size: 'Size',
        pattern: 'Pattern',
        style: 'Style',
        checkout: 'Checkout',
        classicWhiteTee: 'Classic White Tee',
        slimFitJeans: 'Slim Fit Jeans',
        floralSummerDress: 'Floral Summer Dress',
        leatherBelt: 'Leather Belt',
        runningShoes: 'Running Shoes',
        denimJacket: 'Denim Jacket',
        pleatedSkirt: 'Pleated Skirt',
        silkScarf: 'Silk Scarf',
        casualSneakers: 'Casual Sneakers',
        floralBlouse: 'Floral Blouse',
        leatherHandbag: 'Leather Handbag',
        formalTrousers: 'Formal Trousers',
        stripedPoloShirt: 'Striped Polo Shirt',
        maxiDress: 'Maxi Dress',
        leatherWallet: 'Leather Wallet',
        cotton100: '100% Cotton',
        cotton98Elastane2: '98% Cotton, 2% Elastane',
        viscose100: '100% Viscose',
        leather100: '100% Leather',
        syntheticMesh: 'Synthetic Mesh',
        denim100: '100% Denim',
        polyester100: '100% Polyester',
        silk100: '100% Silk',
        canvas: 'Canvas',
        wool: 'Wool',
        regular: 'Regular',
        slim: 'Slim',
        aLine: 'A-Line',
        loose: 'Loose',
        straight: 'Straight',
        machineWashCold: 'Machine wash cold',
        midRise: 'Mid-rise',
        high: 'High',
        knee: 'Knee-length',
        midi: 'Midi',
        maxi: 'Maxi',
        metallic: 'Metallic',
        rubber: 'Rubber',
        athletic: 'Athletic',
        medium: 'Medium',
        floral: 'Floral',
        striped: 'Striped',
        laceUp: 'Lace-up',
        tote: 'Tote',
        compact: 'Compact',
        bifold: 'Bifold',
        classicWhiteTeeDescription: 'A timeless classic, this white tee is perfect for any casual occasion. Made from soft, breathable cotton for all-day comfort.',
        slimFitJeansDescription: 'These slim fit jeans offer a modern, tailored look with a comfortable stretch. Perfect for both casual and semi-formal occasions.',
        floralSummerDressDescription: 'A light and breezy summer dress with a beautiful floral print. Perfect for warm days and outdoor events.',
        leatherBeltDescription: 'A versatile leather belt that adds a touch of sophistication to any outfit. Durable and stylish.',
        runningShoesDescription: 'High-performance running shoes designed for comfort and support. Ideal for jogging, training, or casual wear.',
        denimJacketDescription: 'A classic denim jacket that never goes out of style. Versatile and perfect for layering in any season.',
        pleatedSkirtDescription: 'An elegant pleated skirt that moves beautifully. Dress it up or down for various occasions.',
        silkScarfDescription: 'A luxurious silk scarf with a stunning floral pattern. Adds a touch of elegance to any outfit.',
        casualSneakersDescription: 'Comfortable and stylish sneakers perfect for everyday wear. Pairs well with jeans, shorts, or casual dresses.',
        floralBlouseDescription: 'A lightweight and feminine blouse with a delicate floral print. Great for both work and weekend outings.',
        leatherHandbagDescription: 'A spacious and elegant leather handbag. Features multiple compartments for organized storage of your essentials.',
        formalTrousersDescription: 'Tailored trousers suitable for office wear or formal events. Crafted from high-quality wool for comfort and durability.',
        stripedPoloShirtDescription: 'A classic polo shirt with a modern striped pattern. Perfect for a smart-casual look on or off the golf course.',
        maxiDressDescription: 'A flowing maxi dress made from soft cotton. Ideal for summer parties, beach outings, or casual dinners.',
        leatherWalletDescription: 'A sleek and compact leather wallet with multiple card slots and a bill compartment. Combines style with functionality.'
    },
    ru: {
        all: 'Все',
        new: 'Новинки',
        tops: 'Верх',
        bottoms: 'Низ',
        dresses: 'Платья',
        accessories: 'Аксессуары',
        shoes: 'Обувь',
        addToCart: 'В корзину',
        close: 'Закрыть',
        yourCart: 'Ваша корзина',
        emptyCart: 'Ваша корзина пуста',
        remove: 'Удалить',
        settings: 'Настройки',
        theme: 'Тема',
        light: 'Светлая',
        dark: 'Темная',
        language: 'Язык',
        copyright: 'Авторское право © {year} Stylish. Все права защищены.',
        characteristics: 'Характеристики',
        description: 'Описание',
        material: 'Материал',
        fit: 'Посадка',
        care: 'Уход',
        rise: 'Высота посадки',
        length: 'Длина',
        width: 'Ширина',
        buckle: 'Пряжка',
        sole: 'Подошва',
        type: 'Тип',
        wash: 'Стирка',
        size: 'Размер',
        pattern: 'Узор',
        style: 'Стиль',
        checkout: 'Оформить заказ',
        classicWhiteTee: 'Классическая белая футболка',
        slimFitJeans: 'Джинсы зауженного кроя',
        floralSummerDress: 'Летнее платье с цветочным принтом',
        leatherBelt: 'Кожаный ремень',
        runningShoes: 'Беговые кроссовки',
        denimJacket: 'Джинсовая куртка',
        pleatedSkirt: 'Плиссированная юбка',
        silkScarf: 'Шелковый шарф',
        casualSneakers: 'Повседневные кроссовки',
        floralBlouse: 'Блузка с цветочным принтом',
        leatherHandbag: 'Кожаная сумка',
        formalTrousers: 'Классические брюки',
        stripedPoloShirt: 'Полосатая рубашка-поло',
        maxiDress: 'Платье макси',
        leatherWallet: 'Кожаный кошелек',
        cotton100: '100% хлопок',
        cotton98Elastane2: '98% хлопок, 2% эластан',
        viscose100: '100% вискоза',
        leather100: '100% кожа',
        syntheticMesh: 'Синтетическая сетка',
        denim100: '100% деним',
        polyester100: '100% полиэстер',
        silk100: '100% шелк',
        canvas: 'Канвас',
        wool: 'Шерсть',
        regular: 'Стандартная',
        slim: 'Зауженная',
        aLine: 'А-силуэт',
        loose: 'Свободная',
        straight: 'Прямая',
        machineWashCold: 'Машинная стирка в холодной воде',
        midRise: 'Средняя посадка',
        high: 'Высокая',
        knee: 'До колена',
        midi: 'Миди',
        maxi: 'Макси',
        metallic: 'Металлическая',
        rubber: 'Резиновая',
        athletic: 'Спортивный',
        medium: 'Средний',
        floral: 'Цветочный',
        striped: 'Полосатый',
        laceUp: 'На шнуровке',
        tote: 'Тоут',
        compact: 'Компактный',
        bifold: 'Двойного сложения',
        classicWhiteTeeDescription: 'Классическая белая футболка - идеальный выбор для любого повседневного образа. Изготовлена из мягкого, дышащего хлопка для комфорта в течение всего дня.',
        slimFitJeansDescription: 'Эти джинсы зауженного кроя предлагают современный, приталенный вид с комфортным растяжением. Идеально подходят как для повседневных, так и для полуформальных случаев.',
        floralSummerDressDescription: 'Легкое и воздушное летнее платье с красивым цветочным принтом. Идеально подходит для теплых дней и мероприятий на открытом воздухе.',
        leatherBeltDescription: 'Универсальный кожаный ремень, который добавляет нотку изысканности любому наряду. Прочный и стильный.',
        runningShoesDescription: 'Высокоэффективные беговые кроссовки, разработанные для комфорта и поддержки. Идеально подходят для бега трусцой, тренировок или повседневной носки.',
        denimJacketDescription: 'Классическая джинсовая куртка, которая никогда не выходит из моды. Универсальная и идеальная для многослойных образов в любое время года.',
        pleatedSkirtDescription: 'Элегантная плиссированная юбка, которая красиво движется. Подходит как для повседневных, так и для нарядных образов.',
        silkScarfDescription: 'Роскошный шелковый шарф с потрясающим цветочным узором. Добавляет нотку элегантности любому наряду.',
        casualSneakersDescription: 'Удобные и стильные кроссовки, идеально подходящие для повседневной носки. Хорошо сочетаются с джинсами, шортами или повседневными платьями.',
        floralBlouseDescription: 'Легкая и женственная блузка с нежным цветочным принтом. Отлично подходит как для работы, так и для выходных.',
        leatherHandbagDescription: 'Просторная и элегантная кожаная сумка. Имеет несколько отделений для организованного хранения ваших вещей.',
        formalTrousersDescription: 'Классические брюки, подходящие для офисной одежды или официальных мероприятий. Изготовлены из высококачественной шерсти для комфорта и долговечности.',
        stripedPoloShirtDescription: 'Классическая рубашка-поло с современным полосатым узором. Идеально подходит для smart-casual образа на поле для гольфа и за его пределами.',
        maxiDressDescription: 'Струящееся платье макси из мягкого хлопка. Идеально подходит для летних вечеринок, пляжного отдыха или непринужденных ужинов.',
        leatherWalletDescription: 'Элегантный и компактный кожаный кошелек с несколькими отделениями для карт и отделением для купюр. Сочетает в себе стиль и функциональность.'
    },
}

const t = (key, params = {}) => {
    const translation = translations[currentLanguage.value][key] || key
    return Object.entries(params).reduce(
        (acc, [key, value]) => acc.replace(`{${key}}`, value),
        translation
    )
}

const filteredProducts = computed(() => {
    if (selectedCategory.value === 'All') {
        return products.value
    }
    return products.value.filter(product => product.category === selectedCategory.value)
})

const totalPages = computed(() => Math.ceil(filteredProducts.value.length / itemsPerPage))

const paginatedProducts = computed(() => {
    const startIndex = (currentPage.value - 1) * itemsPerPage
    const endIndex = startIndex + itemsPerPage
    return filteredProducts.value.slice(startIndex, endIndex)
})

const selectCategory = (category) => {
    selectedCategory.value = category
    currentPage.value = 1
}

const openProductModal = (product) => {
    selectedProduct.value = product
}

const closeProductModal = () => {
    selectedProduct.value = null
}

const toggleCart = () => {
    isCartOpen.value = !isCartOpen.value
}

const toggleSettings = () => {
    isSettingsOpen.value = !isSettingsOpen.value
}

const addToCart = (product) => {
    cartItems.value.push(product)
    closeProductModal()
}

const removeFromCart = (product) => {
    const index = cartItems.value.findIndex(item => item.id === product.id)
    if (index !== -1) {
        cartItems.value.splice(index, 1)
    }
}

const setTheme = (theme) => {
    isDarkMode.value = theme === 'dark'
}

const setLanguage = (lang) => {
    currentLanguage.value = lang
}

const checkout = () => {
    // Здесь будет логика оформления заказа
    console.log('Checkout initiated')
    // Можно добавить дополнительную логику, например, перенаправление на страницу оформления заказа
}
</script>

<style scoped>
/* Add any additional component-specific styles here */
</style>