<template>
  <div class="card-container">
    <div class="card-box">
      <!-- Card content -->
      <div class="card-number-row">
        <p>
          Pokemon Name: <span>{{ pokemon.name }}</span>
        </p>
      </div>
      <div class="card-details">
        <div class="card-holder-col">
          <button @click="fetchSinglePokemonURL">
            Check out the <span>{{ pokemon.name }}</span>
          </button>
          <span class="card-holder-name">ALI ABDI</span>
        </div>
        <div class="card-date-col">
          <span class="card-date-title">VALID THRU</span>
          <span class="card-date-sub">06/26</span>
        </div>
      </div>
    </div>
    <DitalCard
      v-if="openDetailCard"
      :pokemonData="fetchedPokemonData"
      @close="handleClose"
    />
  </div>
</template>

<script>
import { ref } from "vue";
import DitalCard from "./DitalCard.vue";

export default {
  props: {
    pokemon: {
      type: Object,
      required: true,
    },
  },
  components: {
    DitalCard,
  },
  setup(props) {
    const fetchedPokemonData = ref(null);
    const openDetailCard = ref(false);

    const fetchSinglePokemonURL = async () => {
      try {
        const response = await fetch(props.pokemon.url);
        const data = await response.json();
        fetchedPokemonData.value = data;
        openDetailCard.value = true;
      } catch (error) {
        console.error("Failed to fetch Pokémon:", error);
      }
    };

    function handleClose() {
      openDetailCard.value = false;
    }

    return {
      fetchedPokemonData,
      openDetailCard,
      fetchSinglePokemonURL,
      handleClose,
    };
  },
};
</script>
<style>
.card-container,
.card-box,
.card-head,
.card-number-row,
.card-details,
.card-holder-col,
.card-date-col {
  display: flex;
}
.card-container {
  align-items: center;
  justify-content: center;
}
.card-box {
  width: 300px;
  min-height: 160px;
  background: #fff;
  box-shadow: 0px 0px 15px -2px rgba(0, 0, 0, 0.1);
  border-radius: 18px;
  margin: 16px;
  padding: 1.5em;
  flex-direction: column;
  justify-content: space-around;
  gap: 1.2em;
}
.card-head {
  justify-content: space-between;
  align-items: center;
}
.card-chip svg {
  width: 32px;
  height: 32px;
}
.card-chip svg path {
  fill: #636363;
}
.card-logo svg {
  width: 48px;
  height: 48px;
}
.card-number-row {
  justify-content: center;
  word-spacing: 1em;
  font-size: 1.3em;
  font-weight: 600;
}
.card-box:hover .card-number-row {
  font-size: 1.32em;
}
.card-details {
  justify-content: space-between;
  text-transform: uppercase;
}
.card-holder-col {
  flex-direction: column;
  gap: 2px;
}
.card-holder-title,
.card-date-title {
  color: #bdbdbd;
  font-size: 0.7em;
}
.card-holder-name {
  font-size: 1.1em;
  font-weight: 600;
}
.card-date-col {
  flex-direction: column;
  align-items: flex-end;
  gap: 2px;
}
</style>
