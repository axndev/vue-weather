<script setup>
import { onMounted, ref, watchEffect } from 'vue';
import axios, { all } from 'axios';
import { useDark, useToggle } from "@vueuse/core";
import { useGeolocation } from '@vueuse/core'


const searchInput = ref('');
const apiKey = '5caf657bd0f3550114586fe98b37e5e2';
const weatherImage = ref('')
const weatherData = ref({});
const forecastData = ref({});
const dailyForcast = ref([]);
const selectUnit = ref('C')
const hourlyForecast = ref([]);
const showTodaysWeather = ref(false)

const { coords, locatedAt, error, resume, pause } = useGeolocation();

const lat = ref();
const long = ref();

watchEffect(() => {
    if (coords.value) {
        lat.value = coords.value.latitude;
        long.value = coords.value.longitude;
        console.log('Latitude:', lat.value);
        console.log('Longitude:', long.value);
    }
});

function toggleClass() {
    const searchBox = document.getElementById("searchBox");
    searchBox?.classList.toggle('active-search');
}

function getWeather() {
    const todayWeather = async () => {
        try {
            const { data } = await axios.get(`https://api.openweathermap.org/data/2.5/weather?appid=${apiKey}&q=${searchInput.value}`);
            weatherData.value = data;
            weatherImage.value = `https://openweathermap.org/img/wn/${data.weather?.[0]?.icon}@2x.png`;
            console.log(weatherData.value)
        } catch (error) {
            console.log('Error fetching weather:', error);
        }
    };

    const getForeCast = async () => {
        try {
            const { data } = await axios.get(`https://api.openweathermap.org/data/2.5/forecast?appid=${apiKey}&q=${searchInput.value}`);
            forecastData.value = data;
            const allForecasts = forecastData.value.list;
            hourlyForecast.value = allForecasts.slice(0, 4);
            dailyForcast.value = allForecasts.filter(list => list.dt_txt.endsWith("12:00:00"));

        } catch (error) {
            console.log('Error fetching ForeCast:', error);
        }
    };
    todayWeather(); getForeCast();
}

const getLocationData = async () => {

    const todayWeather = async () => {
        try {
            const { data } = await axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${lat.value}&lon=${long.value}&appid=${apiKey}`);
            weatherData.value = data;
            weatherImage.value = `https://openweathermap.org/img/wn/${data.weather?.[0]?.icon}@2x.png`;
            searchInput.value = data.name
        } catch (error) {
            console.log('Error fetching weather:', error);
        }
    };

    const getForeCast = async () => {
        try {
            const { data } = await axios.get(`https://api.openweathermap.org/data/2.5/forecast?lat=${lat.value}&lon=${long.value}&appid=${apiKey}`);
            forecastData.value = data;
            const allForecasts = forecastData.value.list;
            hourlyForecast.value = allForecasts.slice(0, 4);
            console.log(hourlyForecast.value)
            dailyForcast.value = allForecasts.filter(list => list.dt_txt.endsWith("12:00:00"));

        } catch (error) {
            console.log('Error fetching ForeCast:', error);
        }
    };
    todayWeather(); getForeCast();
}

function getLocalDate(date) {
    const dateObject = new Date(date);
    return dateObject.toLocaleString('en-US', { weekday: 'short' });
}

function getLocalTime(datetimeStr) {
    const timePart = datetimeStr.split(" ")[1];
    const [hh, mm] = timePart.split(":").map(Number);

    const period = hh >= 12 ? "PM" : "AM";
    const hour12 = hh % 12 || 12;

    const paddedMinutes = mm < 10 ? "0" + mm : mm;

    return `${hour12}:${paddedMinutes} ${period}`;
}

const isDark = useDark({ selector: 'html' });
const toggleDark = () => isDark.value = !isDark.value




function toggleUnit() {
    const toggle = document.getElementById('toggleUnit');
    toggle.classList.toggle('kelvin');
    const iskelvin = toggle.classList.contains('kelvin');

    selectUnit.value = iskelvin ? 'K' : 'C';
}

function toggleTodays(){
    showTodaysWeather.value = !showTodaysWeather.value
}
</script>

<template>
    <main>
        <div class="max-w-240 m-auto relative">
            <div class="flex justify-between mb-10 md:my-10">
                <h2 :class="['text-(--primary) text-3xl  md:text-4xl font-bold', isDark ? '' : '']">Vue Weather</h2>
                <div class="flex justify-center items-center gap-5">
                    <label class="relative inline-flex items-center cursor-pointer">
                        <input type="checkbox" class="sr-only peer" @change="toggleDark" :checked="isDark" />
                        <div
                            class="w-17 h-8 bg-(--secondary)/30 dark:bg-gray-600 rounded-full peer peer-checked:bg-[#   051A33] transition-colors duration-2500 px-2 flex items-center justify-between relative">
                            <!-- Sun Icon -->
                            <i class="fas fa-sun text-yellow-400 text-base transition-all duration-2500 ease-in-out z-0"
                                :class="isDark ? '-rotate-90 opacity-0 scale-0' : 'rotate-0 opacity-100 scale-100 z-99 !text-(--secondary)'"></i>

                            <!-- Moon Icon -->
                            <i class="fas fa-moon text-white text-base transition-all duration-2500 ease-in-out z-0 mr-1"
                                :class="isDark ? 'rotate-0 opacity-100 scale-100  z-99 !text-(--primary)' : 'rotate-90 opacity-0 scale-0'"></i>

                            <!-- Toggle Knob (ensure it’s above icons) -->
                            <div class="w-6 h-6 bg-white dark:bg-gray-900 rounded-full shadow-md absolute top-1 transition-all duration-300 z-10"
                                :class="isDark ? 'left-9.5' : 'left-1'"></div>
                        </div>
                    </label>
                    <a href="http://www.github.com/kaleemullahahsan"><i
                            class="fa-brands fa-github text-3xl text-(--primary)"></i></a>
                </div>
            </div>
            <div :class="['flex m-auto  flex-col justify-center items-center p-3 px-4 my-5 border border-(--secondary)/30 bg-white shadow rounded-full', isDark ? '!bg-[#04162D]' : '']"
                id="searchBox">
                <div class="w-full flex items-center gap-3">
                    <i class="fa-solid fa-magnifying-glass p-1 cursor-pointer text-(--primary)/80 text-xl"
                        @click="getWeather(searchInput)"></i>
                    <input type="text" v-model="searchInput" placeholder="Search for location"
                        class="outline-0 w-full font-medium placeholder:text-(--primary)/50 text-lg text-gray-500"
                        @keyup.enter="getWeather(searchInput)" @focus="toggleClass" @blur="toggleClass">
                    <i class="fa-solid fa-location-crosshairs  p-1 cursor-pointer text-(--primary)/80 text-2xl"
                        @click="getLocationData"></i>
                </div>
            </div>

            <div v-if="weatherData && weatherData.main"
                :class="['flex flex-col gap-5 bg-white rounded-xl shadow p-5', isDark ? '!bg-[#051A33]' : '']">
                <header class="flex justify-between items-center">
                    <span :class="['text-lg text-gray-500 font-medium', isDark ? '!text-(--primary)' : '']">Current
                        Weather</span>
                    <div id="toggleUnit" @click="toggleUnit"
                        class="relative w-17 h-8 bg-(--secondary)/20 rounded-full cursor-pointer flex items-center justify-between px-2 text-sm font-semibold select-none transition-all duration-300 group">
                        <span class="z-10 text-white group-[.kelvin]:text-gray-700 pl-1">°C</span>
                        <span class="z-10 text-gray-700 group-[.kelvin]:text-white pr-2">K</span>
                        <div
                            class="absolute w-6 h-6 bg-(--secondary) rounded-full top-1 left-5 transition-all duration-300 group-[.kelvin]:left-12 -translate-x-1/2">
                        </div>
                    </div>
                </header>
                <div class="p-4 pb-0">
                    <div class="grid md:grid-cols-2 gap-10">
                        <div class="flex items-start flex-col md:gap-5">
                            <span class="text-xl text-(--primary) font-bold capitalize">{{ weatherData.name }}, {{
                                weatherData.sys.country }}</span>
                            <div class="flex justify-center items-center">
                                <img :src="weatherImage" alt="" class="w-30">
                                <span class="text-8xl text-(--primary) font-thin">{{
                                    Math.round(selectUnit == 'C' ? weatherData.main.temp - 273.15 :
                                        weatherData.main.temp)
                                }}{{ selectUnit == 'K' ? 'K' : '°C' }}</span>
                            </div>
                            <span class="text-2xl font-bold capitalize text-(--primary)/70">{{
                                weatherData.weather?.[0]?.description }}</span>
                        </div>
                        <div class="flex flex-col gap-4">
                            <span class="text-(--primary)/80 font-semibold text-xl">Feel like {{
                                Math.round(selectUnit == 'C' ? weatherData.main.feels_like - 273.15 :
                                    weatherData.main.feels_like) }}{{ selectUnit == 'K' ? 'K' : '°C' }}</span>
                            <div class="flex justify-start items-center text-(--primary)/80 gap-10">
                                <div class="flex justify-center items-center gap-3">
                                    <svg width="16" height="21" viewBox="0 0 16 21" class="fill-(--primary)/60">
                                        <path d="M20,15H15V3a1,1,0,0,0-1-1H10A1,1,0,0,0,9,3V15H4l8,8,8-8Z"
                                            transform="translate(20 23) rotate(180)"></path>
                                    </svg>
                                    <span class="text-xl font-bold">{{ Math.round(selectUnit === 'C' ?
                                        weatherData.main.temp_max - 273.15 : weatherData.main.temp_max)
                                    }}{{ selectUnit == 'K' ? 'K' : '°C' }}</span>
                                </div>
                                <div class="flex justify-center items-center gap-3">
                                    <svg width="16" height="21" viewBox="0 0 16 21" class="fill-(--primary)/60">
                                        <path d="M20,15H15V3a1,1,0,0,0-1-1H10A1,1,0,0,0,9,3V15H4l8,8,8-8Z"
                                            transform="translate(-4 -2)"></path>
                                    </svg>
                                    <span class="text-xl font-bold">{{ Math.round(selectUnit === 'C' ?
                                        weatherData.main.temp_min - 273.15 : weatherData.main.temp_min)
                                    }}{{ selectUnit == 'K' ? 'K' : '°C' }}</span>
                                </div>
                            </div>
                            <div class="flex justify-start flex-col items-start text-(--primary)/80 gap-3 my-3">
                                <div class="flex justify-center items-center gap-5">
                                    <svg width="12" height="18" viewBox="0 0 12 18" class="fill-(--primary)/60">
                                        <path
                                            d="M12,1a.667.667,0,0,0-.536.272l-.02.026C9.87,3.324,6,9.682,6,13a6,6,0,1,0,12,0c0-3.314-3.861-9.66-5.439-11.693,0,0,0,0,0,0l-.025-.034A.667.667,0,0,0,12,1Zm2.333,13.333a1,1,0,1,1-1,1A1,1,0,0,1,14.333,14.333Z"
                                            transform="translate(-6 -1)"></path>
                                    </svg>
                                    <span>Humidity</span>
                                    <span class="text-lg font-semibold">{{
                                        Math.round(weatherData.main.humidity) }}%</span>
                                </div>
                                <div class="flex justify-center items-center gap-5">
                                    <svg width="17" height="16.346" viewBox="0 0 17 16.346" class="fill-(--primary)/60">
                                        <path
                                            d="M12.135,3A4.244,4.244,0,0,0,8.462,5.117a2.89,2.89,0,0,0-3.816,2.49A2.932,2.932,0,0,0,2,10.519a2.992,2.992,0,0,0,.033.327h4a1.984,1.984,0,0,1-.111-.654A1.961,1.961,0,0,1,7.885,8.231h.654A3.273,3.273,0,0,1,11.808,11.5a3.235,3.235,0,0,1-.67,1.962h4.92a2.934,2.934,0,0,0,.309-5.853c.01-.118.018-.238.018-.359A4.25,4.25,0,0,0,12.135,3ZM7.885,9.538a.654.654,0,1,0,0,1.308h.654a.654.654,0,1,1,0,1.308H2.654a.654.654,0,1,0,0,1.308H8.538a1.962,1.962,0,0,0,0-3.923ZM3.961,14.769a.654.654,0,1,0,.654.654A.654.654,0,0,0,3.961,14.769Zm2.615,0a.654.654,0,1,0,0,1.308h6.865a.981.981,0,1,1,0,1.962h-.981a.654.654,0,1,0,0,1.308h.981a2.288,2.288,0,0,0,0-4.577Z"
                                            transform="translate(-2 -3)"></path>
                                    </svg>
                                    <span>Wind</span>
                                    <span class="text-lg font-semibold">{{ Math.round(weatherData.wind.speed * 3.6)
                                    }}kph</span>
                                </div>
                                <div class="flex justify-center items-center gap-5">
                                    <svg width="16.889" height="16.92" viewBox="0 0 16.889 16.92"
                                        class="fill-(--primary)/60">
                                        <path
                                            d="M12.444,4A8.46,8.46,0,1,1,4,12.46,8.463,8.463,0,0,1,12.444,4ZM13.5,7.173l1.352,1.355-1.88,1.884a2.008,2.008,0,0,0-.528-.066,2.112,2.112,0,0,0-2.111,2.115,2.019,2.019,0,0,0,.066.529L8.948,14.476a2.652,2.652,0,0,0-2.837.595L7.6,16.558a.537.537,0,0,1,.759.76l1.484,1.487a2.663,2.663,0,0,0,.594-2.842l1.484-1.454a2.008,2.008,0,0,0,.528.066,2.112,2.112,0,0,0,2.111-2.115,2.019,2.019,0,0,0-.066-.529l1.88-1.884L17.722,11.4V7.173Z"
                                            transform="translate(-4 -4)"></path>
                                    </svg>
                                    <span>Pressure</span>
                                    <span class="text-lg font-semibold">{{ Math.round(weatherData.main.pressure)
                                    }}hPa</span>
                                </div>
                            </div>

                        </div>
                    </div>
                    <div class="flex justify-center">
                        <span
                        @click="toggleTodays"
                        class="text-center flex flex-col items-center mt-5 text-(--primary)/90 font-semibold cursor-pointer hover:text-(--primary)">
                        <span> {{showTodaysWeather == false ? 'Show Current Conditions' : 'Show Upcoming Weather'}}</span>
                        <svg fill="#000000" height="22px" width="22px" version="1.1" id="Layer_1"
                            xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
                            viewBox="0 0 512.04 512.04" xml:space="preserve" stroke="var(--primary)" stroke-width="19.45752">
                            <g id="SVGRepo_bgCarrier" stroke-width="0"></g>
                            <g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g>
                            <g id="SVGRepo_iconCarrier">
                                <g>
                                    <g>
                                        <path
                                            d="M508.933,146.807l-42.347-42.347c-4.267-4.053-10.88-4.053-15.147,0L256.027,300.193L60.507,104.46 c-4.267-4.053-10.88-4.053-15.147,0L3.12,146.807c-4.16,4.16-4.16,10.88,0,15.04L248.453,407.5c4.16,4.16,10.88,4.16,15.04,0 l245.333-245.653C513.093,157.687,513.093,150.967,508.933,146.807z M256.027,384.887L25.733,154.38l27.2-27.307l195.52,195.733 c4.267,4.053,10.88,4.053,15.147,0l195.52-195.733l27.2,27.307L256.027,384.887z">
                                        </path>
                                    </g>
                                </g>
                            </g>
                        </svg>
                    </span>
                    </div>
                </div>
            </div>
            <div v-if="showTodaysWeather && dailyForcast.length > 0"
                :class="['mt-5 bg-white/50 rounded-xl shadow p-5', isDark ? '!bg-[#04162D] opacity-70' : '']">
                <span :class="['text-lg text-gray-500 font-medium', isDark ? '!text-(--primary)' : '']">Current Conditions</span>
                <div class="md:grid grid-cols-4 flex md:overflow-hidden overflow-scroll mt-5">
                    <div v-for="day in hourlyForecast" class="flex flex-col justify-center items-center flex-[1_0_33%]">
                        <span class="text-(--primary)/80 font-bold text-lg">{{ getLocalTime(day.dt_txt) }}</span>

                        <img :src="'https://openweathermap.org/img/wn/' + day.weather?.[0]?.icon + '@2x.png'"
                            :alt="day.weather?.[0]?.main">
                        <span class="text-(--primary)/80 font-bold text-lg">{{ day.weather?.[0]?.main }}</span>
                        <div class="flex text-lg text-(--primary) mt-1">
                            <span>{{ Math.round(selectUnit === 'C' ? day.main.temp_max - 273.15 : day.main.temp_max)
                            }}°</span> /
                            <span>{{ Math.round(selectUnit === 'C' ? day.main.temp_min - 273.15 : day.main.temp_min)
                            }}°</span>
                        </div>
                    </div>
                </div>
            </div>
            <div v-else-if="dailyForcast.length > 0" :class="['mt-5 bg-white/50 rounded-xl shadow p-5', isDark ? '!bg-[#04162D] opacity-70' : '']">
                <span :class="['text-lg text-gray-500 font-medium', isDark ? '!text-(--primary)' : '']">Upcoming Weather</span>
                <div class="md:grid grid-cols-5 flex md:overflow-hidden overflow-scroll mt-5">
                    <div v-for="day in dailyForcast" class="flex flex-col justify-center items-center flex-[1_0_33%]">
                        <span class="text-(--primary)/80 font-bold text-lg">{{ getLocalDate(day.dt_txt) }}</span>

                        <img :src="'https://openweathermap.org/img/wn/' + day.weather?.[0]?.icon + '@2x.png'"
                            :alt="day.weather?.[0]?.main">
                        <span class="text-(--primary)/80 font-bold text-lg">{{ day.weather?.[0]?.main }}</span>
                        <div class="flex text-lg text-(--primary) mt-1">
                            <span>{{ Math.round(selectUnit === 'C' ? day.main.temp_max - 273.15 : day.main.temp_max)
                            }}°</span> /
                            <span>{{ Math.round(selectUnit === 'C' ? day.main.temp_min - 273.15 : day.main.temp_min)
                            }}°</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </main>
</template>
