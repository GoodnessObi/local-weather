<template>
  <header class="sticky top-0 shadow-lg">
    <nav class="container flex flex-col sm:flex-row items-center gap-4 py-6 text-white">
      <RouterLink :to="{ name: 'home' }">
        <div class="flex item-center gap-3">
          <i class="fa-solid fa-sun text-2xl"></i>
          <p class="text-2xl">The Local Weather</p>
        </div>
      </RouterLink>

      <div class="flex gap-3 flex-1 justify-end">
        <i
          class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          @click="toggleModal"
        ></i>

        <i
          class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          @click="addCity"
          v-if="route.query.preview"
        ></i>
      </div>

      <BaseModal :modalActive="modalActive" @close-modal="toggleModal">
        <h1 class="text-black">Hello from modal</h1>
      </BaseModal>
    </nav>
  </header>
</template>

<script setup>
import { RouterLink, useRoute, useRouter } from 'vue-router'
import BaseModal from './BaseModal.vue'
import { ref } from 'vue'
import { uid } from 'uid'

const modalActive = ref(null)
const savedCities = ref([])
const route = useRoute()
const router = useRouter()

const toggleModal = () => {
  modalActive.value = !modalActive.value
}

const addCity = () => {
  if (localStorage.getItem('savedCities')) {
    savedCities.value = JSON.parse(localStorage.getItem('savedCities'))
  }

  const locationObject = {
    id: uid(),
    state: route.params.state,
    city: route.params.city,
    coords: {
      lat: route.query.lat,
      lon: route.query.lon
    }
  }

  savedCities.value.push(locationObject)

  localStorage.setItem('savedCities', JSON.stringify(savedCities.value))

  let query = { ...route.query }

  delete query.preview

  router.replace({ query })
}
</script>
