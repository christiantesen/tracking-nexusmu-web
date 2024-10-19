<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';

interface Character {
  id: string;
  'Información del Personaje': {
    Personaje: string;
    Clase: string;
    País: string;  // Agregado
    'Último Ingreso': string;  // Agregado
    'NivelNivel M.': string;  // Agregado
    Resets: string;  // Agregado
    'Grand Resets': string;
    Fuerza: string;  // Agregado
    Agilidad: string;  // Agregado
    Vitalidad: string;  // Agregado
    Energía: string;  // Agregado
    Comando: string;  // Agregado
    Ubicación: string;
    'Nivel PK': string;  // Agregado
    'Fuerza (Bonus)': string;
    'Agilidad (Bonus)': string;  // Agregado
    'Vitalidad (Bonus)': string;  // Agregado
    'Energía (Bonus)': string;  // Agregado
  };
  'Información Gens': {
    Familia: string;
    Rango: string;  // Agregado
    Puntos: string;  // Agregado
  };
  'Información del Guild': {
    Guild: string;
    Master: string;  // Agregado
    Miembros: string;  // Agregado
    Puntos: string;  // Agregado
    Aliados: string;  // Agregado
    Enemigos: string;  // Agregado
    Posición: string;  // Agregado
  };
}

const characters = ref<Character[]>([]);
const selectedCharacter = ref<Character | null>(null);
const errorMessage = ref<string | null>(null);
const searchQuery = ref('');
const selectedClass = ref('');
const selectedFamily = ref('');
const selectedGuild = ref('');
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

const uniqueGuilds = computed(() => {
  const guilds = new Set(
    characters.value.map((char) => char['Información del Guild'].Guild)
  );
  return ['', ...Array.from(guilds)];
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
    const matchesGuild =
      selectedGuild.value === '' ||
      char['Información del Guild'].Guild === selectedGuild.value;

    return matchesSearch && matchesClass && matchesFamily && matchesGuild;
  });
});

onMounted(fetchData);
</script>

<template>
  <div class="container">
    <h1>Tracking Table</h1>
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
      <select v-model="selectedGuild" class="filter-select">
        <option value="">All Guilds</option>
        <option v-for="guild in uniqueGuilds" :key="guild" :value="guild">
          {{ guild }}
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
              (parseInt(
                character['Información del Personaje']['Fuerza (Bonus)'].replace(/[.,]/g, '')
              ) + parseInt(
                character['Información del Personaje']['Agilidad (Bonus)'].replace(/[.,]/g, '')
              ) + parseInt(
                character['Información del Personaje']['Vitalidad (Bonus)'].replace(/[.,]/g, '')
              ) + parseInt(
                character['Información del Personaje']['Energía (Bonus)'].replace(/[.,]/g, '')
              )) > 6000
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

    <div v-if="selectedCharacter" class="modal" @click.self="closeModal">
      <div class="modal-content">
        <h2>Detalles del Personaje</h2>

        <h3>Información del Personaje</h3>
    <table>
      <tbody>
        <tr>
          <td><strong>Personaje:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje'].Personaje }}</td>
        </tr>
        <tr>
          <td><strong>Clase:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje'].Clase }}</td>
        </tr>
        <tr>
          <td><strong>País:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje'].País }}</td>
        </tr>
        <tr>
          <td><strong>Último Ingreso:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje']['Último Ingreso'] }}</td>
        </tr>
        <tr>
          <td><strong>Nivel Nivel M.:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje']['NivelNivel M.'] }}</td>
        </tr>
        <tr>
          <td><strong>Resets:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje'].Resets }}</td>
        </tr>
        <tr>
          <td><strong>Grand Resets:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje']['Grand Resets'] }}</td>
        </tr>
        <tr>
          <td><strong>Ubicación:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje'].Ubicación }}</td>
        </tr>
        <tr>
          <td><strong>Nivel PK:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje']['Nivel PK'] }}</td>
        </tr>
      </tbody>
    </table>

    <h3>Estadísticas</h3>
    <table>
      <tbody>
        <tr>
          <td><strong>Fuerza:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje'].Fuerza }}</td>
        </tr>
        <tr>
          <td><strong>Agilidad:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje'].Agilidad }}</td>
        </tr>
        <tr>
          <td><strong>Vitalidad:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje'].Vitalidad }}</td>
        </tr>
        <tr>
          <td><strong>Energía:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje'].Energía }}</td>
        </tr>
        <tr>
          <td><strong>Comando:</strong></td>
          <td>{{ selectedCharacter['Información del Personaje'].Comando }}</td>
        </tr>
      </tbody>
    </table>

    <h3>Bonificaciones</h3>
    <table>
      <tbody>
        <tr>
          <td><strong>Fuerza (Bonus):</strong></td>
          <td>{{ selectedCharacter['Información del Personaje']['Fuerza (Bonus)'] }}</td>
        </tr>
        <tr>
          <td><strong>Agilidad (Bonus):</strong></td>
          <td>{{ selectedCharacter['Información del Personaje']['Agilidad (Bonus)'] }}</td>
        </tr>
        <tr>
          <td><strong>Vitalidad (Bonus):</strong></td>
          <td>{{ selectedCharacter['Información del Personaje']['Vitalidad (Bonus)'] }}</td>
        </tr>
        <tr>
          <td><strong>Energía (Bonus):</strong></td>
          <td>{{ selectedCharacter['Información del Personaje']['Energía (Bonus)'] }}</td>
        </tr>
      </tbody>
    </table>

    <h3>Información Gens</h3>
    <table>
      <tbody>
        <tr>
          <td><strong>Familia:</strong></td>
          <td>{{ selectedCharacter['Información Gens'].Familia }}</td>
        </tr>
        <tr>
          <td><strong>Rango:</strong></td>
          <td>{{ selectedCharacter['Información Gens'].Rango }}</td>
        </tr>
        <tr>
          <td><strong>Puntos:</strong></td>
          <td>{{ selectedCharacter['Información Gens'].Puntos }}</td>
        </tr>
      </tbody>
    </table>

    <h3>Información del Guild</h3>
    <table>
      <tbody>
        <tr>
          <td><strong>Guild:</strong></td>
          <td>{{ selectedCharacter['Información del Guild'].Guild }}</td>
        </tr>
        <tr>
          <td><strong>Master:</strong></td>
          <td>{{ selectedCharacter['Información del Guild'].Master }}</td>
        </tr>
        <tr>
          <td><strong>Miembros:</strong></td>
          <td>{{ selectedCharacter['Información del Guild'].Miembros }}</td>
        </tr>
        <tr>
          <td><strong>Puntos:</strong></td>
          <td>{{ selectedCharacter['Información del Guild'].Puntos }}</td>
        </tr>
        <tr>
          <td><strong>Aliados:</strong></td>
          <td>{{ selectedCharacter['Información del Guild'].Aliados }}</td>
        </tr>
        <tr>
          <td><strong>Enemigos:</strong></td>
          <td>{{ selectedCharacter['Información del Guild'].Enemigos }}</td>
        </tr>
        <tr>
          <td><strong>Posición:</strong></td>
          <td>{{ selectedCharacter['Información del Guild'].Posición }}</td>
        </tr>
      </tbody>
    </table>
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
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  z-index: 1000;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  max-width: 600px;
  width: 90%;
  max-height: 80%; /* Limitar la altura del modal */
  overflow-y: auto; /* Habilitar el desplazamiento vertical */
}

h2, h3 {
  margin: 0 0 10px;
  color: #333;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

td {
  padding: 10px;
  border: 1px solid #ccc;
}

td:first-child {
  font-weight: bold;
  background-color: #f9f9f9;
}

.close-button {
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  padding: 10px 15px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.close-button:hover {
  background-color: #0056b3;
}

.character-info {
  margin-bottom: 20px;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 10px;
  margin-bottom: 20px;
}

.info-item {
  padding: 10px;
  background: #f9f9f9;
  border-radius: 5px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.close-button {
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  padding: 10px 15px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.close-button:hover {
  background-color: #0056b3;
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
