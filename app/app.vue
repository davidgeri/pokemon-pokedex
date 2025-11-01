<template>
  <div class="app-container">
    <NuxtRouteAnnouncer />
    <div class="header">
      <div class="header-left">
        <div class="title-section">
          <img
            src="https://www.pngkey.com/png/full/144-1446994_pokeball-clipart-transparent-background-pokeball-png.png"
            alt="Pokeball"
            class="pokeball-icon"
          />
          <h1>Pokédex</h1>
        </div>
      </div>

      <div class="header-center">
        <div class="logo-container">
          <img
            src="https://1000logos.net/wp-content/uploads/2017/05/Pokemon-Logo.png"
            alt="Pokemon Logo"
            class="pokemon-logo"
          />
        </div>
      </div>

      <div class="header-right">
        <!-- Search Bar -->
        <div class="search-container">
          <input
            v-model="searchTerm"
            type="text"
            placeholder="Search Pokémon..."
            class="search-input"
            @input="handleSearch"
          />
          <div class="search-icon">🔍</div>
        </div>
      </div>
    </div>

    <!-- Loading State -->
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>Loading Pokémon...</p>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="error">
      <p>Error loading Pokémon: {{ error }}</p>
      <button @click="fetchPokemon" class="retry-btn">Retry</button>
    </div>

    <!-- Pokemon Grid -->
    <div v-else class="pokemon-container">
      <div class="card-grid">
        <div
          v-for="pokemon in currentPagePokemon"
          :key="pokemon.id"
          class="pokemon-card"
          :style="{ borderColor: pokemon.color }"
          @click="openModal(pokemon)"
        >
          <div class="card-content">
            <div class="pokemon-image">
              <img
                :src="pokemon.sprite"
                :alt="pokemon.name"
                @error="handleImageError"
              />
            </div>
            <h3>{{ capitalize(pokemon.name) }}</h3>
            <p class="pokemon-number">
              #{{ String(pokemon.id).padStart(3, "0") }}
            </p>
            <div class="pokemon-types">
              <span
                v-for="type in pokemon.types"
                :key="type"
                class="pokemon-type"
                :style="{ backgroundColor: getTypeColor(type) }"
              >
                {{ capitalize(type) }}
              </span>
            </div>
            <div class="pokemon-stats">
              <div class="stat-item">
                <span class="stat-label">Height:</span>
                <span class="stat-value"
                  >{{ (pokemon.height / 10).toFixed(1) }} m</span
                >
              </div>
              <div class="stat-item">
                <span class="stat-label">Weight:</span>
                <span class="stat-value">{{
                  formatWeight(pokemon.weight)
                }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- No Results Message -->
      <div v-if="searchTerm && filteredPokemonCount === 0" class="no-results">
        <div class="no-results-content">
          <div class="no-results-icon">🔍</div>
          <h3>No Pokémon Found</h3>
          <p>Try searching for a different name</p>
        </div>
      </div>

      <!-- Pagination -->
      <div v-if="filteredPokemonCount > 0" class="pagination">
        <button
          @click="goToPage(currentPage - 1)"
          :disabled="currentPage === 1"
          class="pagination-btn"
        >
          ← Previous
        </button>

        <div class="page-numbers">
          <button
            v-for="page in visiblePages"
            :key="page"
            @click="goToPage(page)"
            :disabled="
              page === currentPage &&
              ![1, 2, totalPages - 1, totalPages].includes(currentPage)
            "
            :class="['page-btn', { active: page === currentPage }]"
          >
            {{ page }}
          </button>
        </div>

        <button
          @click="goToPage(currentPage + 1)"
          :disabled="currentPage === totalPages"
          class="pagination-btn"
        >
          Next →
        </button>
      </div>

      <div v-if="filteredPokemonCount > 0" class="pagination-info">
        Showing {{ (currentPage - 1) * itemsPerPage + 1 }} -
        {{ Math.min(currentPage * itemsPerPage, filteredPokemonCount) }} of
        {{ filteredPokemonCount }} Pokémon
        <span v-if="searchTerm" class="search-results">
          (filtered from {{ allPokemon.length }} total)
        </span>
      </div>
    </div>

    <!-- Pokemon Modal -->
    <div v-if="showModal" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <button class="modal-close" @click="closeModal">×</button>

        <div
          v-if="selectedPokemon"
          class="modal-pokemon-card"
          :style="{ borderColor: selectedPokemon.color }"
        >
          <div class="card-content">
            <div class="pokemon-image">
              <img
                :src="selectedPokemon.sprite"
                :alt="selectedPokemon.name"
                @error="handleImageError"
              />
            </div>
            <h3>{{ capitalize(selectedPokemon.name) }}</h3>
            <p class="pokemon-number">
              #{{ String(selectedPokemon.id).padStart(3, "0") }}
            </p>

            <div class="pokemon-types">
              <span
                v-for="type in selectedPokemon.types"
                :key="type"
                class="pokemon-type"
                :style="{ backgroundColor: getTypeColor(type) }"
              >
                {{ capitalize(type) }}
              </span>
            </div>

            <div class="pokemon-stats">
              <div class="stat-item">
                <span class="stat-label">Height:</span>
                <span class="stat-value"
                  >{{ (selectedPokemon.height / 10).toFixed(1) }} m</span
                >
              </div>
              <div class="stat-item">
                <span class="stat-label">Weight:</span>
                <span class="stat-value">{{
                  formatWeight(selectedPokemon.weight)
                }}</span>
              </div>
            </div>

            <!-- Abilities Section -->
            <div class="abilities-section">
              <h4>Abilities</h4>
              <div class="abilities-list">
                <span
                  v-for="ability in selectedPokemon.abilities"
                  :key="ability"
                  class="ability-badge"
                >
                  {{ capitalize(ability) }}
                </span>
              </div>
            </div>

            <!-- Moves Section -->
            <div class="moves-section">
              <h4>Moves ({{ selectedPokemon.moves.length }})</h4>
              <div class="moves-list">
                <span
                  v-for="move in selectedPokemon.moves"
                  :key="move"
                  class="move-badge"
                >
                  {{ capitalize(move) }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
// Reactive data
const allPokemon = ref([]);
const loading = ref(true);
const error = ref(null);
const currentPage = ref(1);
const itemsPerPage = 10;
const showModal = ref(false);
const selectedPokemon = ref(null);
const searchTerm = ref("");

// Type colors mapping
const typeColors = {
  normal: "#A8A878",
  fighting: "#C03028",
  flying: "#A890F0",
  poison: "#A040A0",
  ground: "#E0C068",
  rock: "#B8A038",
  bug: "#A8B820",
  ghost: "#705898",
  steel: "#B8B8D0",
  fire: "#F08030",
  water: "#6890F0",
  grass: "#78C850",
  electric: "#F8D030",
  psychic: "#F85888",
  ice: "#98D8D8",
  dragon: "#7038F8",
  dark: "#705848",
  fairy: "#EE99AC",
};

// Computed properties
const totalPages = computed(() => {
  const filteredPokemon = searchTerm.value
    ? allPokemon.value.filter((pokemon) =>
        pokemon.name.toLowerCase().includes(searchTerm.value.toLowerCase())
      )
    : allPokemon.value;
  return Math.ceil(filteredPokemon.length / itemsPerPage);
});

const currentPagePokemon = computed(() => {
  // First filter by search term
  const filteredPokemon = searchTerm.value
    ? allPokemon.value.filter((pokemon) =>
        pokemon.name.toLowerCase().includes(searchTerm.value.toLowerCase())
      )
    : allPokemon.value;

  // Then paginate
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredPokemon.slice(start, end);
});

const visiblePages = computed(() => {
  const pages = [];
  const total = totalPages.value;
  const current = currentPage.value;

  // Logic for showing page numbers (max 5 pages)
  if (current <= 3) {
    // First 3 pages: show 1-5
    for (let i = 1; i <= Math.min(5, total); i++) {
      pages.push(i);
    }
  } else if (current >= total - 2) {
    // Last 3 pages: show last 5 pages
    for (let i = Math.max(1, total - 4); i <= total; i++) {
      pages.push(i);
    }
  } else {
    // Middle pages: show current-2 to current+2
    for (let i = current - 2; i <= current + 2; i++) {
      pages.push(i);
    }
  }

  return pages;
});

const filteredPokemonCount = computed(() => {
  return searchTerm.value
    ? allPokemon.value.filter((pokemon) =>
        pokemon.name.toLowerCase().includes(searchTerm.value.toLowerCase())
      ).length
    : allPokemon.value.length;
});

// Methods
const capitalize = (str) => {
  return str.charAt(0).toUpperCase() + str.slice(1);
};

const getTypeColor = (type) => {
  return typeColors[type.toLowerCase()] || "#68A090";
};

const formatWeight = (weight) => {
  const kg = weight / 10;
  return kg % 1 === 0 ? `${kg} kg` : `${kg.toFixed(1)} kg`;
};

const handleImageError = (event) => {
  event.target.style.display = "none";
};

const goToPage = (page) => {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page;
    // Scroll to top when changing pages
    // window.scrollTo({ top: 0, behavior: "smooth" });
  }
};

const handleSearch = () => {
  // Reset to first page when searching
  currentPage.value = 1;
};

const fetchPokemonDetails = async (url) => {
  try {
    const response = await fetch(url);
    const data = await response.json();

    return {
      id: data.id,
      name: data.name,
      weight: data.weight,
      height: data.height,
      abilities: data.abilities.map((ability) => ability.ability.name),
      moves: data.moves.slice(0, 10).map((move) => move.move.name), // Limit to first 10 moves
      types: data.types.map((type) => type.type.name),
      sprite:
        data.sprites.other["official-artwork"].front_default ||
        data.sprites.front_default ||
        "https://via.placeholder.com/150x150?text=No+Image",
      color: getTypeColor(data.types[0].type.name),
    };
  } catch (err) {
    console.error("Error fetching Pokemon details:", err);
    return null;
  }
};

const fetchPokemon = async () => {
  try {
    loading.value = true;
    error.value = null;

    // Fetch the list of first 151 Pokemon
    const response = await fetch("https://pokeapi.co/api/v2/pokemon?limit=151");
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();

    // Fetch detailed information for each Pokemon
    const pokemonPromises = data.results.map((pokemon, index) =>
      fetchPokemonDetails(pokemon.url)
    );

    const pokemonDetails = await Promise.all(pokemonPromises);

    // Filter out any failed requests and sort by ID
    allPokemon.value = pokemonDetails
      .filter((pokemon) => pokemon !== null)
      .sort((a, b) => a.id - b.id);
  } catch (err) {
    console.error("Error fetching Pokemon:", err);
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

// Modal methods
const openModal = (pokemon) => {
  selectedPokemon.value = pokemon;
  showModal.value = true;
};

const closeModal = () => {
  showModal.value = false;
  selectedPokemon.value = null;
};

// Fetch Pokemon on component mount
onMounted(() => {
  fetchPokemon();
});
</script>

<style>
/* Global reset */
*,
*::before,
*::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* Remove default body margin */
:root,
body {
  margin: 0;
  padding: 0;
  overflow-x: hidden;
}
</style>

<style scoped>
.app-container {
  min-height: 100vh;
  width: 100vw;
  margin: 0;
  padding: 0;
  background: radial-gradient(
      ellipse at top,
      rgba(26, 26, 62, 0.8) 0%,
      transparent 50%
    ),
    radial-gradient(
      ellipse at bottom,
      rgba(45, 27, 105, 0.6) 0%,
      transparent 50%
    ),
    radial-gradient(
      ellipse at left,
      rgba(138, 43, 226, 0.3) 0%,
      transparent 70%
    ),
    radial-gradient(
      ellipse at right,
      rgba(30, 144, 255, 0.2) 0%,
      transparent 70%
    ),
    linear-gradient(135deg, #0f0f23 0%, #1a1a3e 50%, #2d1b69 100%);
  background-attachment: fixed;
  font-family: "Inter", "Segoe UI", system-ui, -apple-system, sans-serif;
  display: flex;
  flex-direction: column;
  padding: 20px 0;
  position: relative;
}

.app-container::before {
  content: "";
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-image: radial-gradient(
      circle at 20% 80%,
      rgba(120, 119, 198, 0.1) 0%,
      transparent 50%
    ),
    radial-gradient(
      circle at 80% 20%,
      rgba(255, 119, 198, 0.08) 0%,
      transparent 50%
    ),
    radial-gradient(
      circle at 40% 40%,
      rgba(120, 219, 255, 0.06) 0%,
      transparent 50%
    );
  pointer-events: none;
  z-index: -1;
}

.header {
  width: 100%;
  max-width: 1400px;
  margin: 0 auto 20px auto;
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 20px;
}

.header-left {
  display: flex;
  align-items: center;
}

.header-center {
  display: flex;
  justify-content: center;
  flex: 1;
}

.header-right {
  display: flex;
  align-items: center;
}

.header h1 {
  color: white;
  font-size: 2.5rem;
  font-weight: bold;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  margin: 0;
}

.logo-container {
  margin-bottom: 0;
}

.pokemon-logo {
  height: 100px;
  width: auto;
  filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.3));
  transition: transform 0.3s ease;
}

.title-section {
  display: flex;
  align-items: center;
  gap: 12px;
}

.pokeball-icon {
  height: 50px;
  width: 50px;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.3));
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%,
  20%,
  50%,
  80%,
  100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-5px);
  }
  60% {
    transform: translateY(-3px);
  }
}

/* Search Styles */
.search-container {
  position: relative;
  width: 250px;
  display: flex;
  align-items: center;
}

.search-input {
  width: 100%;
  padding: 10px 40px 10px 14px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  color: white;
  font-size: 0.9rem;
  outline: none;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.search-input::placeholder {
  color: rgba(255, 255, 255, 0.7);
}

.search-input:focus {
  border-color: #4c51bf;
  background: rgba(255, 255, 255, 0.15);
  box-shadow: 0 4px 16px rgba(76, 81, 191, 0.3);
  transform: scale(1.02);
}

.search-icon {
  position: absolute;
  right: 12px;
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
  pointer-events: none;
}

/* No Results Styles */
.no-results {
  text-align: center;
  padding: 60px 20px;
  margin: 40px 0;
}

.no-results-content {
  max-width: 400px;
  margin: 0 auto;
}

.no-results-icon {
  font-size: 4rem;
  margin-bottom: 20px;
  opacity: 0.6;
}

.no-results h3 {
  color: white;
  font-size: 1.8rem;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.no-results p {
  color: rgba(255, 255, 255, 0.8);
  font-size: 1.1rem;
  margin: 0;
}

.subtitle {
  color: rgba(255, 255, 255, 0.9);
  font-size: 1.2rem;
  margin: 10px 0 0 0;
}

.loading,
.error {
  text-align: center;
  color: white;
  margin: 50px 0;
}

.spinner {
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top: 4px solid white;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
  margin: 0 auto 20px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.retry-btn {
  background: rgba(255, 255, 255, 0.9);
  color: #1a1a3e;
  border: none;
  padding: 12px 24px;
  border-radius: 30px;
  cursor: pointer;
  font-weight: bold;
  margin-top: 15px;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.retry-btn:hover {
  background: #4c51bf;
  color: white;
  transform: scale(1.05) translateY(-2px);
  box-shadow: 0 6px 20px rgba(76, 81, 191, 0.4);
}

.pokemon-container {
  width: 100%;
  max-width: 1400px;
  margin: 0 auto;
  padding: 20px 0;
}

.card-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  grid-template-rows: repeat(2, 1fr);
  gap: 30px;
  width: 100%;
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 20px;
}

.pokemon-card {
  background: linear-gradient(
    145deg,
    rgba(255, 255, 255, 0.1) 0%,
    rgba(255, 255, 255, 0.05) 50%,
    rgba(255, 255, 255, 0.02) 100%
  );
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 24px;
  padding: 24px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3), 0 2px 8px rgba(255, 255, 255, 0.1),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: pointer;
  position: relative;
  overflow: hidden;
  min-height: 320px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  backdrop-filter: blur(20px);
}

.pokemon-card::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    45deg,
    transparent 30%,
    rgba(255, 255, 255, 0.03) 50%,
    transparent 70%
  );
  opacity: 0;
  transition: opacity 0.3s ease;
  pointer-events: none;
}

.pokemon-card:hover {
  transform: translateY(-12px) scale(1.02);
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.4),
    0 8px 24px rgba(255, 255, 255, 0.15), inset 0 1px 0 rgba(255, 255, 255, 0.3);
  border-color: rgba(255, 255, 255, 0.4);
}

.pokemon-card:hover::before {
  opacity: 1;
}

.card-content {
  text-align: center;
  position: relative;
  z-index: 2;
  width: 100%;
}

.card-content h3 {
  font-size: 1.4rem;
  font-weight: 700;
  margin: 0 0 8px 0;
  color: #ffffff;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  letter-spacing: 0.5px;
}

.pokemon-number {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.9);
  margin: 4px 0 16px 0;
  font-weight: 500;
  opacity: 0.9;
  background: rgba(255, 255, 255, 0.1);
  padding: 4px 12px;
  border-radius: 12px;
  display: inline-block;
  backdrop-filter: blur(10px);
}

.pokemon-image {
  margin-bottom: 16px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.pokemon-image::before {
  content: "";
  position: absolute;
  width: 130px;
  height: 130px;
  border-radius: 50%;
  background: radial-gradient(
    circle,
    rgba(255, 255, 255, 0.15) 0%,
    rgba(255, 255, 255, 0.08) 40%,
    rgba(255, 255, 255, 0.03) 70%,
    transparent 100%
  );
  backdrop-filter: blur(15px);
  z-index: 1;
  box-shadow: 0 8px 32px rgba(255, 255, 255, 0.1),
    inset 0 2px 4px rgba(255, 255, 255, 0.2);
}

.pokemon-image img {
  width: 110px;
  height: 110px;
  object-fit: contain;
  filter: drop-shadow(0 4px 12px rgba(0, 0, 0, 0.3));
  position: relative;
  z-index: 2;
  transition: transform 0.3s ease;
}

.pokemon-card:hover .pokemon-image img {
  transform: scale(1.1);
}

.pokemon-types {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  justify-content: center;
  margin-top: 12px;
}

.pokemon-type {
  color: white;
  padding: 6px 16px;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 600;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.5);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
}

.pokemon-type:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4),
    inset 0 1px 0 rgba(255, 255, 255, 0.3);
}

.pokemon-stats {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-top: 16px;
  width: 100%;
}

.stat-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(255, 255, 255, 0.08);
  padding: 8px 16px;
  border-radius: 16px;
  backdrop-filter: blur(15px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.stat-item:hover {
  background: rgba(255, 255, 255, 0.12);
  transform: translateX(2px);
}

.stat-label {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.8);
  font-weight: 500;
}

.stat-value {
  font-size: 0.85rem;
  color: white;
  font-weight: 600;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  margin: 40px 0 20px 0;
  flex-wrap: wrap;
}

.pagination-btn {
  background: rgba(255, 255, 255, 0.9);
  color: #1a1a3e;
  border: none;
  padding: 12px 24px;
  border-radius: 30px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.pagination-btn:hover:not(:disabled) {
  background: #4c51bf;
  color: white;
  transform: translateY(-3px);
  box-shadow: 0 6px 20px rgba(76, 81, 191, 0.4);
}

.pagination-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.page-numbers {
  display: flex;
  gap: 5px;
  flex-wrap: wrap;
}

.page-btn {
  background: rgba(255, 255, 255, 0.8);
  color: #1a1a3e;
  border: none;
  width: 45px;
  height: 45px;
  border-radius: 50%;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.page-btn:hover {
  background: white;
  /* transform: scale(1.15); */
  box-shadow: 0 4px 12px rgba(255, 255, 255, 0.3);
}

.page-btn.active {
  background: #4c51bf;
  color: white;
  box-shadow: 0 4px 15px rgba(76, 81, 191, 0.5);
}

.page-btn:disabled {
  cursor: not-allowed;
  opacity: 0.8;
}

.pagination-info {
  text-align: center;
  color: rgba(255, 255, 255, 0.9);
  font-size: 0.9rem;
  margin: 15px 0 20px 0;
}

.search-results {
  display: block;
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.6);
  margin-top: 5px;
}

/* Responsive design */
@media (max-width: 1200px) {
  .card-grid {
    grid-template-columns: repeat(4, 1fr);
    grid-template-rows: repeat(3, 1fr);
  }
}

@media (max-width: 900px) {
  .card-grid {
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(4, 1fr);
  }
}

@media (max-width: 600px) {
  .header {
    flex-direction: column;
    align-items: center;
    gap: 15px;
  }

  .header-center {
    order: -1; /* Logo comes first on mobile */
  }

  .header-left {
    order: 1; /* Title comes second */
  }

  .header-right {
    order: 2; /* Search comes last */
  }

  .pokemon-logo {
    height: 70px;
  }

  .pokemon-logo {
    height: 80px;
  }

  .card-grid {
    grid-template-columns: repeat(2, 1fr);
    grid-template-rows: repeat(5, 1fr);
    gap: 20px;
  }

  .header h1 {
    font-size: 2rem;
  }

  .title-section {
    gap: 8px;
  }

  .pokeball-icon {
    height: 35px;
    width: 35px;
  }

  .pokemon-card {
    min-height: 280px;
    padding: 20px;
    border-radius: 20px;
  }

  .pokemon-image img {
    width: 85px;
    height: 85px;
  }

  .pokemon-image::before {
    width: 100px;
    height: 100px;
  }

  .card-content h3 {
    font-size: 1.2rem;
  }

  .pokemon-number {
    font-size: 0.8rem;
    padding: 3px 10px;
  }

  .pokemon-type {
    font-size: 0.75rem;
    padding: 4px 12px;
  }

  .stat-item {
    padding: 6px 12px;
  }

  .stat-label {
    font-size: 0.75rem;
  }

  .stat-value {
    font-size: 0.8rem;
  }

  .search-container {
    width: 100%;
    max-width: none;
  }

  .search-input {
    font-size: 1rem;
    padding: 14px 45px 14px 18px;
  }

  .modal-close {
    width: 48px;
    height: 48px;
    font-size: 24px;
    top: 20px;
    right: 20px;
  }

  .pagination {
    flex-direction: column;
    gap: 20px;
    margin: 30px 0 15px 0;
    align-items: center;
    justify-content: center;
  }

  .pagination-btn {
    width: auto;
    max-width: 140px;
    padding: 12px 20px;
    font-size: 0.95rem;
    margin: 0 20px;
  }

  .page-numbers {
    justify-content: center;
    gap: 8px;
    order: 0; /* Page numbers in the middle */
  }

  .pagination-btn:first-of-type {
    order: -1; /* Previous button on top */
  }

  .pagination-btn:last-of-type {
    order: 1; /* Next button on bottom */
  }
}

@media (max-width: 400px) {
  .card-grid {
    grid-template-columns: 1fr;
    grid-template-rows: repeat(10, 1fr);
  }

  .pokemon-card {
    min-height: 260px;
    padding: 18px;
  }

  .pokemon-image img {
    width: 80px;
    height: 80px;
  }

  .pokemon-image::before {
    width: 90px;
    height: 90px;
  }

  .modal-close {
    width: 50px;
    height: 50px;
    font-size: 26px;
    top: 15px;
    right: 15px;
  }

  .pagination {
    gap: 8px;
    margin: 25px 0 10px 0;
  }

  .pagination-btn {
    padding: 10px 12px;
    font-size: 0.85rem;
  }

  .page-btn {
    width: 40px;
    height: 40px;
    font-size: 0.85rem;
  }
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  backdrop-filter: blur(5px);
  animation: modalFadeIn 0.3s ease-out;
}

.modal-content {
  position: relative;
  max-width: 500px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  background: rgba(15, 15, 35, 0.95);
  border-radius: 20px;
  padding: 5px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  animation: modalSlideIn 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.modal-close {
  position: absolute;
  top: 15px;
  right: 15px;
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: white;
  width: 35px;
  height: 35px;
  border-radius: 50%;
  font-size: 20px;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  z-index: 1001;
}

.modal-close:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: scale(1.1);
  border-color: rgba(255, 255, 255, 0.5);
}

.modal-pokemon-card {
  background: rgba(255, 255, 255, 0.1);
  border: 3px solid;
  border-radius: 20px;
  padding: 40px;
  margin: 0;
  min-height: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  backdrop-filter: blur(10px);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3), 0 0 20px rgba(255, 255, 255, 0.1);
  position: relative;
}

.modal-pokemon-card::before {
  content: "";
  position: absolute;
  top: -3px;
  left: -3px;
  right: -3px;
  bottom: -3px;
  border-radius: 23px;
  background: linear-gradient(
    45deg,
    transparent,
    rgba(255, 255, 255, 0.1),
    transparent
  );
  z-index: -1;
  opacity: 0.5;
}

/* Modal text colors for readability */
.modal-pokemon-card h3 {
  color: white !important;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
}

.modal-pokemon-card .pokemon-number {
  color: rgba(255, 255, 255, 0.9) !important;
}

.modal-pokemon-card .stat-label {
  color: rgba(255, 255, 255, 0.8) !important;
}

.modal-pokemon-card .stat-value {
  color: white !important;
}

.modal-pokemon-card .abilities-section h4,
.modal-pokemon-card .moves-section h4 {
  color: white !important;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
}

.abilities-section,
.moves-section {
  width: 100%;
  margin-top: 25px;
  text-align: center;
}

.abilities-section h4,
.moves-section h4 {
  color: #1a1a1a;
  font-size: 1.2rem;
  font-weight: bold;
  margin-bottom: 15px;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.abilities-list,
.moves-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
  max-height: 200px;
  overflow-y: auto;
  padding: 15px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  backdrop-filter: blur(5px);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.ability-badge,
.move-badge {
  background: rgba(76, 81, 191, 0.8);
  color: white;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 600;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
  white-space: nowrap;
}

.move-badge {
  background: rgba(45, 27, 105, 0.8);
}

.abilities-list::-webkit-scrollbar,
.moves-list::-webkit-scrollbar {
  width: 8px;
}

.abilities-list::-webkit-scrollbar-track,
.moves-list::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.abilities-list::-webkit-scrollbar-thumb,
.moves-list::-webkit-scrollbar-thumb {
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.6) 0%,
    rgba(255, 255, 255, 0.4) 100%
  );
  border-radius: 4px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  transition: background 0.3s ease;
}

.abilities-list::-webkit-scrollbar-thumb:hover,
.moves-list::-webkit-scrollbar-thumb:hover {
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.8) 0%,
    rgba(255, 255, 255, 0.6) 100%
  );
  box-shadow: 0 0 8px rgba(255, 255, 255, 0.3);
}

.modal-content::-webkit-scrollbar {
  width: 8px;
}

.modal-content::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
}

.modal-content::-webkit-scrollbar-thumb {
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.4) 0%,
    rgba(255, 255, 255, 0.2) 100%
  );
  border-radius: 4px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.modal-content::-webkit-scrollbar-thumb:hover {
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.6) 0%,
    rgba(255, 255, 255, 0.4) 100%
  );
}

/* Modal responsive */
@media (max-width: 600px) {
  .modal-content {
    width: 95%;
    padding: 20px;
    max-height: 95vh;
  }

  .modal-pokemon-card {
    padding: 25px;
  }

  .abilities-list,
  .moves-list {
    max-height: 150px;
    gap: 6px;
  }

  .ability-badge,
  .move-badge {
    font-size: 0.8rem;
    padding: 5px 10px;
  }
}

/* Modal Animations */
@keyframes modalFadeIn {
  from {
    opacity: 0;
    backdrop-filter: blur(0px);
  }
  to {
    opacity: 1;
    backdrop-filter: blur(5px);
  }
}

@keyframes modalSlideIn {
  from {
    opacity: 0;
    transform: scale(0.7) translateY(50px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

@keyframes modalFadeOut {
  from {
    opacity: 1;
    backdrop-filter: blur(5px);
  }
  to {
    opacity: 0;
    backdrop-filter: blur(0px);
  }
}

@keyframes modalSlideOut {
  from {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
  to {
    opacity: 0;
    transform: scale(0.7) translateY(50px);
  }
}
</style>
