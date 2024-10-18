<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';

interface Character {
  id: string;
  'Información del Personaje': {
    Personaje: string;
    Clase: string;
    'Grand Resets': string;
    Ubicación: string;
    'Fuerza (Bonus)': string;
  };
  'Información Gens': {
    Familia: string;
  };
  'Información del Guild': {
    Guild: string;
  };
}

const characters = ref<Character[]>([]);
const selectedCharacter = ref<Character | null>(null);
const errorMessage = ref<string | null>(null);
const searchQuery = ref('');
const selectedClass = ref('');
const selectedFamily = ref('');
const isLoading = ref(false);

const fetchData = async () => {
  isLoading.value = true;
  try {
    const response = await axios.get(
      'https://tracking-nexusmu.onrender.com/scrape/'
    );

    // Agrega este console.log para verificar la respuesta
    console.log('Response from API:', response);

    const data = response.data;
    characters.value = Object.entries(data).map(
      ([id, info]: [string, any]) => ({
        id,
        'Información del Personaje': info['Información del Personaje'],
        'Información Gens': info['Información Gens'],
        'Información del Guild': info['Información del Guild'],
      })
    );
    errorMessage.value = null;
  } catch (error) {
    console.error('Error fetching data:', error);
    errorMessage.value =
      'Failed to fetch character data. Please try again later.';
    characters.value = [];
  } finally {
    isLoading.value = false;
  }
};

const openModal = (character: Character) => {
  selectedCharacter.value = character;
};

const closeModal = () => {
  selectedCharacter.value = null;
};

const uniqueClasses = computed(() => {
  const classes = new Set(
    characters.value.map((char) => char['Información del Personaje'].Clase)
  );
  return ['', ...Array.from(classes)];
});

const uniqueFamilies = computed(() => {
  const families = new Set(
    characters.value.map((char) => char['Información Gens'].Familia)
  );
  return ['', ...Array.from(families)];
});

const filteredCharacters = computed(() => {
  return characters.value.filter((char) => {
    const matchesSearch =
      Object.values(char['Información del Personaje']).some((value) =>
        value.toString().toLowerCase().includes(searchQuery.value.toLowerCase())
      ) ||
      char['Información Gens'].Familia.toLowerCase().includes(
        searchQuery.value.toLowerCase()
      ) ||
      char['Información del Guild'].Guild.toLowerCase().includes(
        searchQuery.value.toLowerCase()
      );

    const matchesClass =
      selectedClass.value === '' ||
      char['Información del Personaje'].Clase === selectedClass.value;
    const matchesFamily =
      selectedFamily.value === '' ||
      char['Información Gens'].Familia === selectedFamily.value;

    return matchesSearch && matchesClass && matchesFamily;
  });
});

onMounted(fetchData);
</script>

<template>
  <div class="container">
    <h1>Character Table</h1>
    <div v-if="errorMessage" class="error-message">
      {{ errorMessage }}
    </div>
    <div v-if="characters.length > 0" class="filters">
      <input
        v-model="searchQuery"
        placeholder="Search..."
        class="search-input"
      />
      <select v-model="selectedClass" class="filter-select">
        <option value="">All Classes</option>
        <option
          v-for="className in uniqueClasses"
          :key="className"
          :value="className"
        >
          {{ className }}
        </option>
      </select>
      <select v-model="selectedFamily" class="filter-select">
        <option value="">All Families</option>
        <option v-for="family in uniqueFamilies" :key="family" :value="family">
          {{ family }}
        </option>
      </select>
    </div>
    <table v-if="filteredCharacters.length > 0">
      <thead>
        <tr>
          <th>Personaje</th>
          <th>Clase</th>
          <th>Grand Resets</th>
          <th>Ubicación</th>
          <th>Pack de Beneficios</th>
          <th>Familia</th>
          <th>Guild</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="character in filteredCharacters" :key="character.id">
          <td>{{ character['Información del Personaje'].Personaje }}</td>
          <td>{{ character['Información del Personaje'].Clase }}</td>
          <td>{{ character['Información del Personaje']['Grand Resets'] }}</td>
          <td>{{ character['Información del Personaje'].Ubicación }}</td>
          <td>
            {{
              parseInt(
                character['Información del Personaje']['Fuerza (Bonus)']
              ) > 1500
                ? 'SI'
                : 'NO'
            }}
          </td>
          <td>{{ character['Información Gens'].Familia }}</td>
          <td>{{ character['Información del Guild'].Guild }}</td>
          <td>
            <button @click="openModal(character)">VER</button>
          </td>
        </tr>
      </tbody>
    </table>
    <div v-else-if="characters.length > 0" class="no-results">
      No characters match the current filters.
    </div>
    <div v-else-if="isLoading" class="loading-message">
      Loading character data...
    </div>

    <div v-if="selectedCharacter" class="modal">
      <div class="modal-content">
        <h2>Detalles del Personaje</h2>
        <pre>{{ JSON.stringify(selectedCharacter, null, 2) }}</pre>
        <button @click="closeModal">Cerrar</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.filters {
  margin-bottom: 20px;
}

.search-input,
.filter-select {
  margin-right: 10px;
  padding: 5px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f2f2f2;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 5px;
  max-width: 80%;
  max-height: 80%;
  overflow: auto;
}

.error-message {
  color: red;
  margin-bottom: 20px;
}

.no-results,
.loading-message {
  text-align: center;
  margin-top: 20px;
}
</style>
