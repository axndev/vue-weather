<script setup>
import { ref, reactive, computed } from 'vue'
import QuoteGenerator from './components/QuoteGenerator.vue';
import WeatherApp from './components/WeatherApp.vue';
import OpenWeather from './components/OpenWeather.vue';


// Reactive -- Use Reactive Objects which values changes

const initialState = reactive({
  val: {
    count: 0
  },
  users: ['Ahsan', 'Huxan']
})

const changesUsers = () => {
  initialState.users[0] = 'Sami';
  initialState.users[1] = 'Ahsan';
}

function addUser(newUser) {
  initialState.users.push(newUser);
}

const myIamge = {
  src: 'https://i.ytimg.com/vi/EtG0tv2a9Uw/hqdefault.jpg?sqp=-oaymwEbCKgBEF5IVfKriqkDDggBFQAAiEIYAXABwAEG&rs=AOn4CLCB6jr2J9lQuw1LaqqwouPH7bYnxA',
  alt: 'This is demo alt',
  width: 400,
  height: 400
}
let counter = ref(0);
let squaredCounter = computed(() => counter.value ** 2);

const increaseCounter = () => {
  counter.value++
}

</script>

<template>
  <div class="p-5">
    <!-- <QuoteGenerator /> -->
     <!-- <WeatherApp /> -->
     <OpenWeather />
  </div>
  <div class="p-5 hidden">
    <span>Count: {{ initialState.val.count }}</span>
    <br>
    <span>Users: {{ initialState.users }}</span>
    <br>
    <button @click="initialState.val.count++">Increament</button>
    <br>
    <button @click="changesUsers">Change User</button>
    <br>
    <button @click="addUser('Noman')">Add User</button>
    <br>
    <img :=myIamge>
    <div>
      <span>
        Counter : {{ counter }}
      </span>
      <br>
      <span>
        Squared Counter : {{ squaredCounter }}
      </span>
      <br>
      <button @click="increaseCounter">Increase Counter</button>
    </div>
  </div>
</template>
