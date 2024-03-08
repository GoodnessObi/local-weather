<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>

  <p v-if="savedCities.length === 0">
    No locations added. To start tracking a location, search in the field above.
  </p>
</template>

<script setup>
import axios from 'axios'
import { ref } from 'vue'
import CityCard from '@/components/CityCard.vue'
import { useRouter } from 'vue-router'

const savedCities = ref([])
const apiKey = import.meta.env.VITE_OPENWEATHER_API_KEY
const router = useRouter()

const getCities = async () => {
  if (localStorage.getItem('savedCities')) {
    savedCities.value = JSON.parse(localStorage.getItem('savedCities'))

    const requests = []

    console.log(savedCities.value, 'pppppp')

    savedCities.value.forEach((city) => {
      console.log(city, 'vvvvv')
      requests.push(
        axios.get(
          `https://api.openweathermap.org/data/3.0/onecall?lat=${city.coords.lat}&lon=${city.coords.lon}&exclude=minutely,daily,hourly&appid=${apiKey}&units=metric`
        )
      )
    })

    const weatherData = await Promise.all(requests)

    await new Promise((res) => setTimeout(res, 1000))

    weatherData.forEach((value, index) => {
      savedCities.value[index].weather = value.data.current
    })
  }
}

await getCities()

const goToCityView = (city) => {
  router.push({
    name: 'cityView',
    params: { state: city.state, city: city.city },
    query: { lat: city.coords.lat, lon: city.coords.lon, id: city.id }
  })
}
</script>
