<script setup>
import { onMounted, ref } from 'vue';
import axios from 'axios';

const searchInput = ref('Kamalia');
const searches = ref([]);
const forecast = ref({});
const formattedMonth = ref('');
const formattedTime = ref('');
const formattedWeekday = ref('');
const formattedDay = ref('');

function formatDate(inputValue) {
    const dateObj = new Date(inputValue.replace(" ", "T"));
    return {
        month: new Intl.DateTimeFormat('en-US', { month: 'long' }).format(dateObj),
        time: new Intl.DateTimeFormat('en-US', {
            hour: 'numeric',
            hour12: true
        }).format(dateObj),
        weekday: new Intl.DateTimeFormat('en-US', {
            weekday: 'short'
        }).format(dateObj),
        day: dateObj.getDate()
    };
}

const updateFormattedDate = (dateStr) => {
    const result = formatDate(dateStr);
    formattedMonth.value = result.month;
    formattedTime.value = result.time;
    formattedWeekday.value = result.weekday;
    formattedDay.value = result.day;
};

const getSearchResponse = async () => {
    if (searchInput.value !== '') {
        try {
            const { data } = await axios.get(`http://api.weatherapi.com/v1/search.json?key=ff24fad548e24d4ea68152442251507&q=${searchInput.value}`);
            searches.value = data;
        } catch (error) {
            console.log(error);
        }
    } else {
        searches.value = [];
    }
};

const getForecastResponse = async (city) => {
    try {
        const { data } = await axios.get(`http://api.weatherapi.com/v1/forecast.json?key=ff24fad548e24d4ea68152442251507&q=${city}`);
        forecast.value = data;
        updateFormattedDate(data.location.localtime);
        searchInput.value = '';
        searches.value = [];
    } catch (error) {
        console.log(error);
    }
};

onMounted(async () => {
    try {
        const { data } = await axios.get(`http://api.weatherapi.com/v1/forecast.json?key=ff24fad548e24d4ea68152442251507&q=${searchInput.value}`);
        forecast.value = data;
        updateFormattedDate(data.location.localtime);
    } catch (error) {
        console.log(error);
    }
});

function toggleClass() {
    const searchBox = document.getElementById("searchBox");
    searchBox?.classList.toggle('active-search');
}
</script>

<template>
    <main>
        <div class="max-w-100 m-auto relative">
            <div class="flex m-auto  flex-col justify-center items-center p-3 my-5 border-2 border-blue-300 bg-blue-300 shadow rounded-full "
                id="searchBox">
                <div class="w-full flex items-center gap-3">
                    <i class="fa-solid fa-magnifying-glass p-1 cursor-pointer text-blue-500"></i>
                    <input type="text" v-model="searchInput" placeholder="Enter your location" class="outline-0 w-full"
                        @input="getSearchResponse" @focus="toggleClass" @blur="toggleClass">
                </div>

            </div>
            <div class=" m-auto absolute bg-white w-full rounded-xl overflow-hidden  ">
                <ul class="w-full">
                    <li v-for="city in searches" :key="city.id" @click="() => getForecastResponse(city.name)"
                        class="p-2  hover:bg-blue-100 border-b hover:border-blue-300 border-blue-200 cursor-pointer">
                        {{ city.name }}
                    </li>
                </ul>
            </div>

            <div class="text-center font-lg my-5  text-blue-800">
                {{ formattedWeekday + ", " + formattedDay + " " + formattedMonth }}
            </div>

            <div v-if="forecast && forecast.location"
                class=" flex flex-col gap-5 text-white m-auto bg-linear-to-b from-blue-500 to-blue-400 rounded-3xl shadow">
                <header class="flex justify-between items-center p-5">
                    <div class="flex items-center gap-2 text-xl">
                        <i class="fa-solid fa-location-dot"></i>
                        <span>{{ forecast.location.name }}</span>
                    </div>
                    <span class="text-lg">{{ formattedTime }}</span>
                </header>

                <div class="p-4">
                    <div class="grid grid-cols-2">
                        <div class="flex items-center flex-col">
                            <img :src="forecast.current.condition.icon" alt="" class="w-20">
                            <span class="text-xl font-light    ">{{ forecast.current.condition.text }}</span>
                        </div>
                        <div class="flex flex-col">
                            <span class="md:text-5xl text-4xl font-bold">{{ Math.round(forecast.current.temp_c)
                                }}°C</span>
                            <span class="text-xl font-light">RealFeel {{ forecast.current.feelslike_c }}°</span>
                        </div>
                    </div>

                    <div class="my-5">
                        <div class="flex flex-col mt-5 gap-1">
                            <div class="grid grid-cols-4 p-3 border-t border-blue-400">
                                <div class="flex justify-between flex-col  items-center text-center">
                                    <span class="infoHead">Max UV Index</span>
                                    <span class="infoDesc">{{ Math.round(forecast.current.uv) }}</span>
                                </div>
                                <div class="flex justify-between flex-col  items-center text-center">
                                    <span class="infoHead">Wind</span>
                                    <span class="infoDesc">W {{ Math.round(forecast.current.wind_kph) }} km/h</span>
                                </div>
                                <div class="flex justify-between flex-col  items-center text-center">
                                    <span class="infoHead">Wind Gusts</span>
                                    <span class="infoDesc">{{ Math.round(forecast.current.gust_kph) }} km/h</span>
                                </div>
                                <div class="flex justify-between flex-col  items-center text-center">
                                    <span class="infoHead">Humidity</span>
                                    <span class="infoDesc">{{ Math.round(forecast.current.humidity) }}%</span>
                                </div>
                            </div>
                            <div class="grid grid-cols-4 p-3 border-t border-blue-400">
                                <div class="flex justify-between flex-col  items-center text-center">
                                    <span class="infoHead">Dew Point</span>
                                    <span class="infoDesc">{{ Math.round(forecast.current.dewpoint_c) }}° C</span>
                                </div>
                                <div class="flex justify-between flex-col  items-center text-center">
                                    <span class="infoHead">Pressure</span>
                                    <span class="infoDesc">{{ Math.round(forecast.current.pressure_mb) }} ↑ mb</span>
                                </div>
                                <div class="flex justify-between flex-col  items-center text-center">
                                    <span class="infoHead">Cloud Cover</span>
                                    <span class="infoDesc">{{ Math.round(forecast.current.cloud) }}%</span>
                                </div>
                                <div class="flex justify-between flex-col  items-center text-center">
                                    <span class="infoHead">Visibility</span>
                                    <span class="infoDesc">{{ Math.round(forecast.current.vis_km) }} km</span>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="text-center">More Details</div>

                </div>
            </div>
            <div
                class="mt-5 text-white m-auto bg-linear-to-b from-blue-400 to-blue-300 rounded-3xl shadow p-5 overflow-x-auto scroll-hidden">
                <div v-if="forecast?.forecast?.forecastday" v-for="castDay in forecast.forecast.forecastday"
                    :key="castDay.date" class="gap-3 flex">
                    <div v-for="hour in castDay.hour" :key="hour.time_epoch"
                        class=" flex-none w-[55px] flex-col items-center">
                        <span class="text-sm">{{ formatDate(hour.time).time }}</span>
                        <img :src="hour.condition.icon" alt="" class="w-10 h-10" />
                        <span class="text-sm font-medium">{{ Math.round(hour.temp_c) }}°C</span>
                    </div>
                </div>
            </div>
        </div>
    </main>
</template>
