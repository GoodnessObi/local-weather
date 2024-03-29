<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'
import CityList from '@/components/CityList.vue'
import CityCardSkeleton from '@/components/CityCardSkeleton.vue'

const router = useRouter()

const searchQuery = ref('')
const queryTimeout = ref(null)
const mapboxSearchResult = ref(null)
const searchError = ref(null)
const mapboxApiKey = import.meta.env.VITE_MAPBOX_API_KEY
const getSearchResults = () => {
  clearTimeout(queryTimeout.value)
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== '') {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxApiKey}&types=place`
        )

        mapboxSearchResult.value = result.data.features
      } catch (e) {
        searchError.value = true
      }
      return
    }
    mapboxSearchResult.value = null
  }, 300)
}

const previewCity = (result) => {
  console.log(result)
  const [city, state] = result.place_name.split(',')

  router.push({
    name: 'cityView',
    params: { state: state.replaceAll(' ', ''), city: city.replaceAll(' ', '') },
    query: {
      lat: result.geometry.coordinates[1],
      lon: result.geometry.coordinates[0],
      preview: true
    }
  })
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
        <p v-if="searchError">Sorry, something went wrong, please try again.</p>
        <p v-if="!searchError && mapboxSearchResult.length === 0">
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResult"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>

    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>
