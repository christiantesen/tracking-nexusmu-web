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
const classImages = ref<{ [key: string]: string }>({});
const familyImages = ref<{ [key: string]: string }>({});
const classImage = ref<string>(''); // Imagen de clase
const familyImage = ref<string>(''); // Imagen de familia

const fetchData = async () => {
  isLoading.value = true;
  try {
    const response = await axios.get(
      'https://tracking-nexusmu.onrender.com/scrape/'
    );

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

const openModal = async (character: Character) => {
  selectedCharacter.value = character;
  classImage.value = await getClassImage(character['Información del Personaje'].Clase);
  familyImage.value = await getFamilyImage(character['Información Gens'].Familia);
};

const closeModal = () => {
  selectedCharacter.value = null;
  classImage.value = '';
  familyImage.value = '';
};

const getFamilyImage = async (family: string) => {
  if (family === 'Vanert') {
    return (await import('@/assets/vanert.png')).default; // Carga dinámica
  } else if (family === 'Duprian') {
    return (await import('@/assets/duprian.png')).default; // Carga dinámica
  }
  return '';
};

const getClassImage = async (className: string) => {
  // Clases de 1ª, 2ª, 3ª y 4ª
  if (
    className === 'Dark Knight' ||
    className === 'Blade Knight' ||
    className === 'Blade Master' ||
    className === 'Dragon Knight' ||
    className === 'Ignition Knight'
  ) {
    return (await import('@/assets/dark_knight.jpg')).default; // Carga dinámica
  } else if (
    className === 'Dark Wizard' ||
    className === 'Soul Master' ||
    className === 'Grand Master' ||
    className === 'Soul Wizard' ||
    className === 'Darkness Wizard'
  ) {
    return (await import('@/assets/dark_wizard.jpg')).default; // Carga dinámica
  } else if (
    className === 'Fairy Elf' ||
    className === 'Muse Elf' ||
    className === 'High Elf' ||
    className === 'Noble Elf' ||
    className === 'Royal Elf'
  ) {
    return (await import('@/assets/fairy_elf.jpg')).default; // Carga dinámica
  } else if (
    className === 'Magic Gladiator' ||
    className === 'Duel Master' ||
    className === 'Magic Knight' ||
    className === 'Duple Knight'
  ) {
    return (await import('@/assets/magic_knight.jpg')).default; // Carga dinámica
  } else if (
    className === 'Dark Lord' ||
    className === 'Lord Emperor' ||
    className === 'Dark Emperor' ||
    className === 'Empire Lord' ||
    className === 'Force Emperor'
  ) {
    return (await import('@/assets/dark_lord.jpg')).default; // Carga dinámica
  } else if (
    className === 'Summoner' ||
    className === 'Bloody Summoner' ||
    className === 'Dimension Summoner' ||
    className === 'Endless Summoner'
  ) {
    return (await import('@/assets/summoner.jpg')).default; // Carga dinámica
  } else if (
    className === 'Rage Fighter' ||
    className === 'Fist Master' ||
    className === 'Fist Blazer' ||
    className === 'Bloody Fighter'
  ) {
    return (await import('@/assets/rage_fighter.jpg')).default; // Carga dinámica
  } else if (
    className === 'Grow Lancer' ||
    className === 'Mirage Lancer' ||
    className === 'Shining Lancer' ||
    className === 'Arcane Lancer'
  ) {
    return (await import('@/assets/arcane_lancer.jpg')).default; // Carga dinámica
  } else if (
    className === 'Rune Mage' ||
    className === 'Rune Spell Master' ||
    className === 'Grand Rune Master' ||
    className === 'Majestic Rune Wizard' ||
    className === 'Infinity Rune Wizard'
  ) {
    return (await import('@/assets/infinity_rune_wizard.jpg')).default; // Carga dinámica
  } else if (
    className === 'Slayer' ||
    className === 'Royal Slayer' ||
    className === 'Master Slayer' ||
    className === 'Slaughterer' ||
    className === 'Rogue Slayer'
  ) {
    return (await import('@/assets/slayer.jpg')).default; // Carga dinámica
  } else if (
    className === 'Gun Crusher' ||
    className === 'Gun Breaker' ||
    className === 'Master Gun Breaker' ||
    className === 'Heist Gun Crusher' ||
    className === 'Magnus Gun Crusher'
  ) {
    return (await import('@/assets/gun_crusher.jpg')).default; // Carga dinámica
  } else if (
    className === 'White Wizard' ||
    className === 'Light Master' ||
    className === 'Shine Wizard' ||
    className === 'Shine Master' ||
    className === 'Glory Wizard'
  ) {
    return (await import('@/assets/glory_wizard.jpg')).default; // Carga dinámica
  } else if (
    className === 'Mage' ||
    className === 'Wo Mage' ||
    className === 'Arch Mage' ||
    className === 'Mystic Mage' ||
    className === 'Battle Mage'
  ) {
    return (await import('@/assets/battle_mage.jpg')).default; // Carga dinámica
  } else if (
    className === 'Illusion Knight' ||
    className === 'Mirage Knight' ||
    className === 'Illusion Master' ||
    className === 'Mystic Knight' ||
    className === 'Phantom Pain Knight'
  ) {
    return (await import('@/assets/phantom_pain_knight.jpg')).default; // Carga dinámica
  }

  // Valor por defecto si no se encuentra una clase
  return (await import('@/assets/default_class_image.png')).default; // Imagen por defecto
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

const preloadImages = async () => {
  for (const character of characters.value) {
    const className = character['Información del Personaje'].Clase;
    const familyName = character['Información Gens'].Familia;

    if (!classImages.value[className]) {
      classImages.value[className] = await getClassImage(className);
    }
    
    if (!familyImages.value[familyName]) {
      familyImages.value[familyName] = await getFamilyImage(familyName);
    }
  }
};

onMounted(() => {
  fetchData().then(preloadImages);
});
</script>

<template>
  <div class="container">
    <h1>Tracking Table</h1>
    <div v-if="errorMessage" class="error-message">
      {{ errorMessage }}
    </div>
    <div v-if="characters.length > 0" class="filters">
      <input v-model="searchQuery" placeholder="Search..." class="search-input" />
      <select v-model="selectedClass" class="filter-select">
        <option value="">All Classes</option>
        <option v-for="className in uniqueClasses" :key="className" :value="className">
          {{ className }}
        </option>
      </select>
      <select v-model="selectedFamily" class="filter-select">
        <option value="">All Gens</option>
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
          <th>Character</th>
          <th>Class</th>
          <th>Location</th>
          <th>Gens</th>
          <th>Guild</th>
          <th>Options</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(character) in filteredCharacters" :key="character.id">
          <td>{{ character['Información del Personaje'].Personaje }}</td>
          <td>
            <img 
          :src="classImages[character['Información del Personaje'].Clase]" 
          alt="Class Image" 
          v-if="classImages[character['Información del Personaje'].Clase]" 
        />
          </td>
          <td>{{ character['Información del Personaje'].Ubicación }}</td>
          <td>
            <img 
          :src="familyImages[character['Información Gens'].Familia]" 
          alt="Family Image" 
          v-if="familyImages[character['Información Gens'].Familia]" 
        />
          </td>
          <td>{{ character['Información del Guild'].Guild }}</td>
          <td>
            <button @click="openModal(character)">Details</button>
          </td>
        </tr>
      </tbody>
    </table>
    <div v-else-if="characters.length > 0" class="no-results">
      No characters match the current filters.
    </div>
    <div v-else-if="isLoading" class="loading-message">
      Loading characters data...
    </div>

    <div v-if="selectedCharacter" class="modal" @click.self="closeModal">
      <div class="modal-content">
        <div class="character-info">
          <img :src="familyImage" alt="Family Image" class="family-image"/>
          <h1>{{ selectedCharacter['Información del Personaje'].Personaje }}</h1>
          <img :src="classImage" alt="Class Image" class="class-image" />
      </div>
        <table class="custom-table">
          <tbody>
            <tr>
              <td><strong>Guild:</strong></td>
              <td>
                {{ selectedCharacter['Información del Guild'].Guild }}
              </td>
            </tr>
            <tr>
              <td><strong>Location:</strong></td>
              <td>{{ selectedCharacter['Información del Personaje'].Ubicación }}</td>
            </tr>
            <tr>
              <td><strong>Log Login:</strong></td>
              <td>{{ selectedCharacter['Información del Personaje']['Último Ingreso'] }}</td>
            </tr>
            <tr>
              <td><strong>Level:</strong></td>
              <td>
                {{
                  (() => {
                    const valor = selectedCharacter['Información del Personaje']['NivelNivel M.'].replace(/[.,\s]/g, '');
                    const parte1 = parseInt(valor.slice(0, 3), 10); // Primeros 3 dígitos
                    const parte2 = parseInt(valor.slice(3), 10); // Últimos dígitos
                    return parte1 + parte2;
                })()
                }}
              </td>
            </tr>
            <tr>
              <td><strong>Stats Pack:</strong></td>
              <td>
                {{
                  (parseInt(
                    selectedCharacter['Información del Personaje']['Fuerza (Bonus)'].replace(/[.,]/g, '')
                  ) + parseInt(
                    selectedCharacter['Información del Personaje']['Agilidad (Bonus)'].replace(/[.,]/g, '')
                  ) + parseInt(
                    selectedCharacter['Información del Personaje']['Vitalidad (Bonus)'].replace(/[.,]/g, '')
                  ) + parseInt(
                    selectedCharacter['Información del Personaje']['Energía (Bonus)'].replace(/[.,]/g, '')
                  )) > 6000
                    ? 'Sí'
                    : 'No'
                }}
              </td>
            </tr>
            <tr>
              <td><strong>Stats GR Full:</strong></td>
              <td>
                {{
                  (parseInt(
                    selectedCharacter['Información del Personaje']['Fuerza (Bonus)'].replace(/[.,]/g, '')
                  ) + parseInt(
                    selectedCharacter['Información del Personaje']['Agilidad (Bonus)'].replace(/[.,]/g, '')
                  ) + parseInt(
                    selectedCharacter['Información del Personaje']['Vitalidad (Bonus)'].replace(/[.,]/g, '')
                  ) + parseInt(
                    selectedCharacter['Información del Personaje']['Energía (Bonus)'].replace(/[.,]/g, '')
                  )) >= 6000
                    ? 'Sí'
                    : 'No'
                }}
              </td>
            </tr>
          </tbody>
        </table>
        <button @click="closeModal">Close</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  user-select: none; /* Para navegadores modernos */
    -moz-user-select: none; /* Para Firefox */
    -webkit-user-select: none; /* Para Safari */
    -ms-user-select: none; /* Para Internet Explorer/Edge */
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
  max-height: 80%;
  /* Limitar la altura del modal */
  overflow-y: auto;
  /* Habilitar el desplazamiento vertical */
}

h2,
h3 {
  margin: 0 0 10px;
  color: #333;
}

.character-info {
  display: flex; /* Usar flexbox para alinear elementos en fila */
  align-items: center; /* Alinea verticalmente los elementos al centro */
  justify-content: center; /* Centra horizontalmente los elementos */
}

.family-image,
.class-image {
  width: 50px; /* Ajusta el ancho según sea necesario */
  height: auto; /* Mantiene la proporción de aspecto */
  margin: 0 10px; /* Espaciado entre las imágenes y el nombre */
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
