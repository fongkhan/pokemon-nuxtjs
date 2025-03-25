<script setup lang="ts">
import { ref, computed } from 'vue';

// Get the Pokemon ID from the route parameter
const route = useRoute();
const pokemonId = computed(() => route.params.id as string);

// Add a timestamp to force refresh when navigating to the same Pokemon
const timestamp = ref(Date.now());

// Using English as the default language
const language = 'en';

// Fetch Pokemon data
const { data: pokemon, error, pending, refresh } = useLazyFetch(() => 
  `https://pokeapi.co/api/v2/pokemon/${pokemonId.value.toLowerCase()}`,
  {
    immediate: true,
    watch: [pokemonId],
    key: `pokemon-${pokemonId.value}-${timestamp.value}` // Add unique key to prevent caching
  }
);

// Fetch Pokemon species data for additional information
const { data: species } = useLazyFetch(() => 
  pokemon.value?.species?.url,
  {
    immediate: false,
    watch: [pokemon],
    key: `species-${pokemonId.value}-${timestamp.value}` // Add unique key to prevent caching
  }
);

// Watch for Pokemon data and fetch species data when available
watch(pokemon, (newValue) => {
  if (newValue?.species?.url) {
    species.refresh();
  }
}, { immediate: true });

// Format Pokemon ID with leading zeros
const formattedId = computed(() => {
  if (!pokemon.value?.id) return '';
  return `#${String(pokemon.value.id).padStart(3, '0')}`;
});

// Get Pokemon types
const types = computed(() => {
  if (!pokemon.value?.types) return [];
  return pokemon.value.types.map(type => type.type.name);
});

// Get Pokemon abilities
const abilities = computed(() => {
  if (!pokemon.value?.abilities) return [];
  return pokemon.value.abilities.map(ability => ({
    name: ability.ability.name,
    isHidden: ability.is_hidden
  }));
});

// Get Pokemon stats
const stats = computed(() => {
  if (!pokemon.value?.stats) return [];
  return pokemon.value.stats.map(stat => ({
    name: stat.stat.name,
    value: stat.base_stat
  }));
});

// Get Pokemon description from species data in English
const description = computed(() => {
  if (!species.value?.flavor_text_entries) return '';
  
  // Get the English description
  const englishEntry = species.value.flavor_text_entries.find(
    entry => entry.language.name === 'en'
  );
  
  return englishEntry?.flavor_text.replace(/\f/g, ' ') || '';
});

// Get Pokemon name in English
const pokemonName = computed(() => {
  if (!species.value?.names) return pokemon.value?.name || '';
  
  // Get the English name
  const englishName = species.value.names.find(
    name => name.language.name === 'en'
  );
  
  // If not found, use the default name from the Pokemon data
  return englishName?.name || pokemon.value?.name || '';
});

// Calculate stat percentage for visual representation
const getStatPercentage = (value) => {
  // Max base stat is typically around 255
  const maxStat = 255;
  return Math.min(100, (value / maxStat) * 100);
};

// Format stat name for display
const formatStatName = (name) => {
  const statNames = {
    'hp': 'HP',
    'attack': 'Attack',
    'defense': 'Defense',
    'special-attack': 'Sp. Atk',
    'special-defense': 'Sp. Def',
    'speed': 'Speed'
  };
  return statNames[name] || name;
};

// Get color based on Pokemon type
const getTypeColor = (type) => {
  const typeColors = {
    normal: '#A8A878',
    fire: '#F08030',
    water: '#6890F0',
    electric: '#F8D030',
    grass: '#78C850',
    ice: '#98D8D8',
    fighting: '#C03028',
    poison: '#A040A0',
    ground: '#E0C068',
    flying: '#A890F0',
    psychic: '#F85888',
    bug: '#A8B820',
    rock: '#B8A038',
    ghost: '#705898',
    dragon: '#7038F8',
    dark: '#705848',
    steel: '#B8B8D0',
    fairy: '#EE99AC'
  };
  return typeColors[type] || '#777';
};

// Go back to search page
const router = useRouter();
const goBack = () => {
  router.push('/');
};

// Update timestamp when component is mounted to force refresh
onMounted(() => {
  timestamp.value = Date.now();
});
</script>

<template>
  <div class="pokemon-details-container">
    <!-- Back button -->
    <button @click="goBack" class="back-button">
      &larr; Back to Search
    </button>
    
    <!-- Loading state -->
    <div v-if="pending" class="loading-container">
      <div class="loading-spinner"></div>
      <p>Loading Pokémon data...</p>
    </div>
    
    <!-- Error state -->
    <div v-else-if="error" class="error-container">
      <h2>Pokémon Not Found</h2>
      <p>Sorry, we couldn't find a Pokémon with that name or ID.</p>
      <button @click="goBack" class="primary-button">Try Another Search</button>
    </div>
    
    <!-- Pokemon details -->
    <div v-else-if="pokemon" class="pokemon-card">
      <div class="pokemon-header">
        <h1>{{ pokemonName.charAt(0).toUpperCase() + pokemonName.slice(1) }}</h1>
        <span class="pokemon-id">{{ formattedId }}</span>
      </div>
      
      <div class="pokemon-main-info">
        <!-- Pokemon image -->
        <div class="pokemon-image-container">
          <img 
            :src="pokemon.sprites?.other['official-artwork']?.front_default || pokemon.sprites?.front_default" 
            :alt="pokemonName"
            class="pokemon-image"
          >
          
          <!-- Pokemon types -->
          <div class="pokemon-types">
            <span 
              v-for="type in types" 
              :key="type"
              class="type-badge"
              :style="{ backgroundColor: getTypeColor(type) }"
            >
              {{ type.charAt(0).toUpperCase() + type.slice(1) }}
            </span>
          </div>
        </div>
        
        <!-- Pokemon info -->
        <div class="pokemon-info">
          <!-- Basic info -->
          <div class="info-section">
            <h2>Basic Info</h2>
            <div class="info-grid">
              <div class="info-item">
                <span class="info-label">Height</span>
                <span class="info-value">{{ (pokemon.height / 10).toFixed(1) }} m</span>
              </div>
              <div class="info-item">
                <span class="info-label">Weight</span>
                <span class="info-value">{{ (pokemon.weight / 10).toFixed(1) }} kg</span>
              </div>
              <div class="info-item">
                <span class="info-label">Base Experience</span>
                <span class="info-value">{{ pokemon.base_experience || 'N/A' }}</span>
              </div>
            </div>
          </div>
          
          <!-- Description -->
          <div class="info-section" v-if="description">
            <h2>Description</h2>
            <p class="pokemon-description">{{ description }}</p>
          </div>
          
          <!-- Abilities -->
          <div class="info-section">
            <h2>Abilities</h2>
            <ul class="abilities-list">
              <li v-for="ability in abilities" :key="ability.name">
                {{ ability.name.split('-').map(word => word.charAt(0).toUpperCase() + word.slice(1)).join(' ') }}
                <span v-if="ability.isHidden" class="hidden-ability">(Hidden)</span>
              </li>
            </ul>
          </div>
        </div>
      </div>
      
      <!-- Stats -->
      <div class="stats-section">
        <h2>Base Stats</h2>
        <div class="stat-bars">
          <div v-for="stat in stats" :key="stat.name" class="stat-item">
            <div class="stat-label">{{ formatStatName(stat.name) }}</div>
            <div class="stat-value">{{ stat.value }}</div>
            <div class="stat-bar-container">
              <div 
                class="stat-bar" 
                :style="{ width: `${getStatPercentage(stat.value)}%`, backgroundColor: getTypeColor(types[0]) }"
              ></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.pokemon-details-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
}

.back-button {
  background-color: transparent;
  border: none;
  color: #e53935;
  font-size: 16px;
  cursor: pointer;
  padding: 10px 0;
  margin-bottom: 20px;
  display: inline-block;
}

.back-button:hover {
  text-decoration: underline;
}

.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 400px;
}

.loading-spinner {
  width: 50px;
  height: 50px;
  border: 5px solid #f3f3f3;
  border-top: 5px solid #e53935;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 20px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error-container {
  text-align: center;
  padding: 40px;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.primary-button {
  background-color: #e53935;
  color: white;
  border: none;
  border-radius: 5px;
  padding: 12px 24px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
  margin-top: 20px;
}

.primary-button:hover {
  background-color: #c62828;
}

.pokemon-card {
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.pokemon-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid #eee;
}

.pokemon-header h1 {
  margin: 0;
  font-size: 2.5rem;
  color: #333;
}

.pokemon-id {
  font-size: 1.5rem;
  color: #777;
  font-weight: bold;
}

.pokemon-main-info {
  display: flex;
  flex-wrap: wrap;
  padding: 20px;
}

.pokemon-image-container {
  flex: 1;
  min-width: 300px;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.pokemon-image {
  width: 100%;
  max-width: 300px;
  height: auto;
}

.pokemon-types {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.type-badge {
  padding: 8px 16px;
  border-radius: 20px;
  color: white;
  font-weight: bold;
  text-transform: capitalize;
}

.pokemon-info {
  flex: 2;
  min-width: 300px;
  padding: 20px;
}

.info-section {
  margin-bottom: 30px;
}

.info-section h2 {
  color: #333;
  border-bottom: 2px solid #eee;
  padding-bottom: 10px;
  margin-top: 0;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 15px;
}

.info-item {
  display: flex;
  flex-direction: column;
}

.info-label {
  font-size: 14px;
  color: #777;
  margin-bottom: 5px;
}

.info-value {
  font-size: 18px;
  font-weight: bold;
  color: #333;
}

.pokemon-description {
  line-height: 1.6;
  color: #555;
}

.abilities-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.abilities-list li {
  padding: 8px 0;
  border-bottom: 1px solid #eee;
  text-transform: capitalize;
}

.abilities-list li:last-child {
  border-bottom: none;
}

.hidden-ability {
  font-size: 12px;
  color: #e53935;
  margin-left: 8px;
}

.stats-section {
  padding: 20px;
  border-top: 1px solid #eee;
}

.stat-bars {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.stat-item {
  display: grid;
  grid-template-columns: 100px 50px 1fr;
  align-items: center;
  gap: 10px;
}

.stat-label {
  font-weight: bold;
  color: #555;
}

.stat-value {
  font-weight: bold;
  color: #333;
  text-align: right;
}

.stat-bar-container {
  height: 10px;
  background-color: #eee;
  border-radius: 5px;
  overflow: hidden;
}

.stat-bar {
  height: 100%;
  border-radius: 5px;
}

@media (max-width: 768px) {
  .pokemon-main-info {
    flex-direction: column;
  }
  
  .stat-item {
    grid-template-columns: 80px 40px 1fr;
  }
}
</style>