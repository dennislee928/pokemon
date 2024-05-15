<template>
  <div v-if="isOpen">
    <div class="detail-card">
      <h2>Pokemon Details</h2>
      <div>
        <p><strong>Name:</strong> {{ pokemonData.name }}</p>
        <p><strong>Height:</strong> {{ pokemonData.height }}</p>
        <p><strong>Weight:</strong> {{ pokemonData.weight }}</p>
        <p><strong>Abilities:</strong></p>
        <ul>
          <li
            v-for="ability in pokemonData.abilities"
            :key="ability.ability.name"
          >
            {{ ability.ability.name }}
          </li>
        </ul>
        <button @click="closeDetailCard">Close Card</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";

export default {
  props: {
    pokemonData: {
      type: Object,
      required: true,
    },
  },
  emits: ["close"],
  setup(props, { emit }) {
    const isOpen = ref(true);

    function closeDetailCard() {
      isOpen.value = false;
      emit("close"); // Emit 'close' event to notify parent component
    }

    return {
      isOpen,
      closeDetailCard,
    };
  },
};
</script>

<style scoped>
.detail-card {
  border: 1px solid #ccc;
  padding: 10px;
  margin-top: 20px;
}
</style>
