<script setup>
import axios from 'axios';
import { ref, onMounted } from 'vue';


const quote = ref([{}]);
const fetchData = async () => {
    await axios.get('https://api.api-ninjas.com/v1/quotes', {
        headers: { 'X-Api-Key': '9g6EaO+nKHkfClFYndHtXQ==Khw2m1oKpoOC5CMB' }
    }).then(response => {
        quote.value = response.data[0]
    }).catch(error => {
        console.error(error);
    });
}
onMounted(fetchData)
</script>
<template>
    <div class="h-[90vh] flex justify-center items-center">
        <div class="bg-white flex flex-col text-center justify-center items-center max-w-[700px] shadow-xl ring-blue-100 ring-1 p-5 rounded-md">
            <blockquote class="mb-5">
                <p class="font-u text-lg my-2 "> <strong>“</strong> {{ quote.quote }} <strong>”</strong></p>
                <cite class="font-semibold">{{ quote.author }}</cite>
            </blockquote>
            <button @click="fetchData">Generate Quotte</button>
        </div>
    </div>
</template>