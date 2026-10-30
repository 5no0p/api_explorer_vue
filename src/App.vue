<script setup>
import { ref, onMounted } from 'vue';

// Application to fetch Pokemon data and display it in cards

// Define variables for Pokemon data, loading status and error
const pokemons = ref([])
const isLoading = ref(false)
const error = ref(null)


const fetchPokemonData = async () => {
  isLoading.value = true
  
  try{

    const data = await fetch('https://pokeapi.co/api/v2/pokemon?limit=15')
    if(!data.ok){
      throw new Error(`HTTP error! status: ${data.status}`)
    }  
    const listData = await data.json()
    
    const detailPromises = listData.results.map(async (item) => {
      const detailResponse = await fetch(item.url)
      if(!detailResponse.ok){
        throw new Error(`Faild to load details for ${item.name}`)
      }
      return detailResponse.json()
    })
    
    const detailsPokemons = await Promise.all(detailPromises)
    pokemons.value = detailsPokemons
  }catch (err){
    error.value = err.message
  }finally{
    isLoading.value = false
  }
  }

// Helper to get the official artwork URL from the sprites object
const getSprite = (pokemon) => {
  return pokemon.sprites?.other?.['official-artwork']?.front_default ?? ''
}
 // Helper to get the abillites from the abillites object
 const getAbiilites = (pokemon) => {
  return pokemon.abilities.map((a) => a.ability.name)
 }

// Helper to get moves as extra skills
const getMoves = (pokemon) => {
  return pokemon.moves.slice(0, 3).map((m)=> m.move.name)
}

// Hepler to get the total power
const getTotalPower = (pokemon) => {
  return pokemon.stats.reduce((total, stat) => total + stat.base_stat, 0)
}

onMounted(fetchPokemonData)
</script>

<template>
  <div class="container min-h-screen bg-gray-100 p-8 font-sans">
    <div class="max-w-7xl mx-auto">
      <h1 class="text-3xl font-bold text-center text-gray-800 mb-8">
        PokéAPI Card Demo
      </h1>
      <!-- Loading status -->
      <div v-if="isLoading" class="text-center py-10">
      <p class="text-xl text-gray-600">Loading Pokémon...</p>
      </div>

       <!-- Error State with Retry Button -->
      <div v-else-if="error" class="text-center py-10 bg-red-50 border border-red-200 rounded-lg p-6 max-w-xl mx-auto">
        <p class="text-lg text-red-700 mb-4">Failed to load data: {{ error }}</p>
        <button
          @click="fetchPokemonData"
          class="px-4 py-2 bg-red-600 text-white font-semibold rounded-lg hover:bg-red-700"
        >
          Try Again
        </button>
      </div>

      <!-- Success State: Card Grid -->
      <div v-else class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6"> 
        <div 
        v-for="pokemon in pokemons" 
        :key="pokemon.id"
        class="bg-white rounded-2xl shadow-md hover:shadow-xl transition-shadow duration-300 overflow-hidden flex flex-col"
        >
          <!-- Image -->
          <div class="bg-linear-to-br from-indigo-50 to-blue-100 p-4 flex items-center justify-center">
            <img 
              :src="getSprite(pokemon)"
              :alt="pokemon.name"
              class="w-32 h-32 object-contain drop-shadow-lg"
              loading="lazy"            
            />
          </div>

          <!-- Content -->
          <div class="p-4 flex flex-col grow">
            <!-- Name -->
            <h2 class="text-xl font-bold text-gray-800 capitalize mb-2">
              {{ pokemon.name }}
            </h2>

            <!-- Skills (Abillites) -->
             <div class="mb-4">
              <h3 class="text-xs font-semibold text-gray-500 uppercase tracking-wider mb-1">Skills</h3>
              <div class="flex flex-wrap gap-1">
                <span 
                  v-for="abillity in getAbiilites(pokemon)" 
                  :key="abillity" 
                  class="inline-block bg-indigo-100 text-indigo-700 text-xs font-medium px-2 py-0.5 rounded-full capitalize"
                >
                  {{ abillity }}
                </span>
              </div>
            </div>
            <!-- Moves (extra skills) -->
             <div>
              <h3>Moves</h3>
              <div>
                <span
                  v-for="move in getMoves(pokemon)"
                  :key="move"
                  class="inline-block bg-gray-100 text-gray-600 text-xs font-medium px-2 py-0.5 rounded-full capitalize"
                >
                  {{ move }}
                </span>
              </div>
             </div>

             <!-- Power (Total Base Stats) -->
              <div class="mt-auto pt-3 border-t border-gray-100 flex justify-between items-center">
                <span class="text-sm text-gray-500">Power</span>
                <span class="text-lg text-bold text-gray-800">{{ getTotalPower(pokemon) }}</span>
              </div>

          </div>
          </div>
      </div>
      </div>
  </div>
</template>

<style scoped>

</style>