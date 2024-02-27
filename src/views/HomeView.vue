<script setup>
import { ref } from 'vue'
import axios from 'axios'

const searchQuery = ref('')
const queryTimeout = ref(null)
const mapboxSearchResult = ref(null)
const mapboxApiKey = import.meta.env.VITE_MAPBOX_API_KEY
const getSearchResults = () => {
  clearTimeout(queryTimeout.value)
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== '') {
      const result = await axios.get(
        `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxApiKey}&types=place`
      )

      console.log(result, 'hhh')

      return (mapboxSearchResult.value = result.data.features)
    }
    mapboxSearchResult.value = null
  }, 300)
}
</script>

<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
        v-model="searchQuery"
        @input="getSearchResults"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResult"
      >
        <li
          v-for="searchResult in mapboxSearchResult"
          :key="searchResult.id"
          class="py-2 cursor-pointer"
        >
          {{ searchResult.place_name }}
        </li>
      </ul>
    </div>
  </main>
</template>
