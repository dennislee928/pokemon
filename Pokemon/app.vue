<template>
  <div id="app">
    <div v-for="pokemon in allPokemon" :key="`${pokemon.name}-${pokemon.url}`">
      <PokemonCard :pokemon="pokemon" @pokemon-details="updatePokemonDetails" />
    </div>
  </div>
</template>

<script>
import PokemonCard from './components/PokemonCard.vue'

export default {
  components: {
    PokemonCard
  },
  data() {
    return {
      allPokemon: [] // This will be filled with Pokémon data
    }
  },
  mounted() {
    this.fetchAllPokemon();
  },
  methods: {
    async fetchAllPokemon() {
      try {
        const response = await fetch("https://pokeapi.co/api/v2/pokemon?limit=10");
        const data = await response.json();
        this.allPokemon = data.results;
      } catch (error) {
        console.error('Failed to fetch Pokémon:', error);
      }
    },
    updatePokemonDetails(data) {
      // Update allPokemon with the details fetched
      this.allPokemon = data;
    }
  }
}
</script>
<style>
/* Ensure this is all valid CSS */
.pokemon-card {
  border: 1px solid #ccc;
  padding: 10px;
}
</style>
