<script setup lang="ts">
import { ref, watch, computed } from 'vue';

// Redirect to the Pokemon details page
const router = useRouter();
const redirectToPokemonDetails = () => {
  // Add timestamp to force refresh when searching for the same Pokemon multiple times
  const timestamp = Date.now();
  router.push(`/pokemon/${nameSearch.value.trim()}?_=${timestamp}`);
};

const nameSearch = ref('');

// Watch for Enter key press to trigger search
const handleKeyDown = (event) => {
  if (event.key === 'Enter' && nameSearch.value.trim()) {
    redirectToPokemonDetails();
  }
};
</script>

<template>
  <div class="container">
    <div class="search-container">
      <h1 class="title">Pokémon Finder</h1>
      <p class="subtitle">Search for a Pokémon by name or ID</p>
      
      <div class="search-box">
        <input 
          v-model="nameSearch" 
          @keydown="handleKeyDown"
          placeholder="Enter Pokémon name or ID"
          class="search-input"
        >
        <button 
          @click="redirectToPokemonDetails"
          class="search-button"
          :disabled="!nameSearch.trim()"
        >
          Search
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-color: #f5f5f5;
  padding: 20px;
}

.search-container {
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  padding: 40px;
  text-align: center;
  max-width: 500px;
  width: 100%;
}

.title {
  color: #e53935;
  font-size: 2.5rem;
  margin-bottom: 10px;
}

.subtitle {
  color: #757575;
  margin-bottom: 20px;
}

.search-box {
  display: flex;
  margin-bottom: 20px;
}

.search-input {
  flex: 1;
  padding: 12px 15px;
  border: 2px solid #e0e0e0;
  border-radius: 5px 0 0 5px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.search-input:focus {
  outline: none;
  border-color: #e53935;
}

.search-button {
  background-color: #e53935;
  color: white;
  border: none;
  border-radius: 0 5px 5px 0;
  padding: 0 20px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.search-button:hover {
  background-color: #c62828;
}

.search-button:disabled {
  background-color: #e0e0e0;
  cursor: not-allowed;
}
</style>