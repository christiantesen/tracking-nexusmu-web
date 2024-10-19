<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue';
import ThemeToggle from './components/ThemeToggle.vue';


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
const connectionStatus = ref<'connecting' | 'connected' | 'error' | 'fallback'>('connecting');
let socket: WebSocket | null = null;
let reconnectTimeout: any = null; // Change to 'any' for compatibility
let fallbackTimeout: any = null; // Change to 'any' for compatibility
let reconnectAttempts = 0;
const FALLBACK_DURATION = 30000; // Duration before fallback to mock data in ms

const mockCharacters: Character[] = [
  // ... (keep the existing mock data)
];

const fetchData = () => {
  isLoading.value = true;
  // Crear una nueva conexión WebSocket
  connectionStatus.value = 'connecting';
  socket = new WebSocket('wss://tracking-nexusmu.onrender.com/ws/scrape/');
  //const socket = new WebSocket('wss://tracking-nexusmu.onrender.com/ws/scrape');

  // Evento que se ejecuta cuando se abre la conexión
  socket.onopen = () => {
    connectionStatus.value = 'connected';
    reconnectAttempts = 0;
    errorMessage.value = null;
    if (fallbackTimeout) {
      clearTimeout(fallbackTimeout); // Clear fallback timeout on success
      fallbackTimeout = null;
    }
  };

  // Evento que se ejecuta cuando se recibe un mensaje desde el WebSocket
  socket!.onmessage = (event) => {
    try {
      const data = JSON.parse(event.data);

      // Actualizar los personajes con la información recibida
      characters.value = Object.entries(data).map(
        ([id, info]: [string, any]) => ({
          id,
          'Información del Personaje': info['Información del Personaje'],
          'Información Gens': info['Información Gens'],
          'Información del Guild': info['Información del Guild'],
        })
      );

      // Llamar a preloadImages cuando los datos de los personajes estén listos
      preloadImages();
    } catch (e) {
      console.error('Error parsing WebSocket message:', e);
    }
  };

  // Evento que se ejecuta cuando ocurre un error en la conexión WebSocket
  socket.onerror = (error) => {
    console.error('WebSocket error:', error);
    if (connectionStatus.value !== 'fallback') {
      handleConnectionError();
    }
    errorMessage.value = 'Failed to connect to WebSocket. Please try again later.';
    characters.value = [];
    isLoading.value = false;
  };

  // Evento que se ejecuta cuando la conexión WebSocket se cierra
  socket.onclose = () => {
    console.log('WebSocket connection closed');
    handleConnectionError();
    isLoading.value = false;
  };
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
  const classCache = new Set<string>();
  const familyCache = new Set<string>();

  for (const character of characters.value) {
    const className = character['Información del Personaje'].Clase;
    const familyName = character['Información Gens'].Familia;

    // Cargar las imágenes si no están en caché
    if (className && !classCache.has(className)) {
      classCache.add(className);
      classImages.value[className] = await getClassImage(className);
    }

    if (familyName && !familyCache.has(familyName)) {
      familyCache.add(familyName);
      familyImages.value[familyName] = await getFamilyImage(familyName);
    }
  }
};

const handleConnectionError = () => {
  connectionStatus.value = 'error';
  reconnectAttempts++;
  scheduleReconnect(); // Always schedule reconnect regardless of attempts

  // Schedule fallback to mock data after a certain duration if still disconnected
  if (!fallbackTimeout) {
    fallbackTimeout = setTimeout(() => {
      handleFallback();
    }, FALLBACK_DURATION);
  }
};

const handleFallback = () => {
  connectionStatus.value = 'fallback';
  characters.value = mockCharacters;
  if (reconnectTimeout) {
    clearTimeout(reconnectTimeout);
    reconnectTimeout = null;
  }
  if (fallbackTimeout) {
    clearTimeout(fallbackTimeout); // Clear fallback timeout if called
    fallbackTimeout = null;
  }
};

const scheduleReconnect = () => {
  if (!reconnectTimeout) {
    reconnectTimeout = setTimeout(() => {
      fetchData(); // Attempt to reconnect
    }, 5000) as any; // Try reconnecting every 5 seconds
  }
};

onMounted(() => {
  fetchData();
});

onUnmounted(() => {
  if (socket) {
    socket.close(); // Close socket on unmount
  }
  if (reconnectTimeout) {
    clearTimeout(reconnectTimeout); // Clear reconnect timeout
  }
  if (fallbackTimeout) {
    clearTimeout(fallbackTimeout); // Clear fallback timeout
  }
});
</script>

<template>
  <div class="container">
    <ThemeToggle />
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
            <img :src="classImages[character['Información del Personaje'].Clase]" alt="Class Image"
              v-if="classImages[character['Información del Personaje'].Clase]" />
          </td>
          <td>{{ character['Información del Personaje'].Ubicación }}</td>
          <td>
            <img :src="familyImages[character['Información Gens'].Familia]" alt="Family Image"
              v-if="familyImages[character['Información Gens'].Familia]" />
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
          <img :src="familyImage" alt="Family Image" class="family-image" />
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
/* Variables de color para tema claro y oscuro */
:root {
  --background-color: #ffffff; /* Color de fondo claro */
  --text-color: #333333; /* Color de texto claro */
  --primary-color: #4caf50; /* Color primario */
  --secondary-color: #007bff; /* Color secundario */
  --border-color: #333333; /* Color de borde */
  --hover-background: rgba(76, 175, 80, 0.2); /* Color de fondo al pasar el ratón */
}

.dark-theme {
  --background-color: #121212; /* Color de fondo oscuro */
  --text-color: #ffffff; /* Color de texto oscuro */
  --primary-color: #1db954; /* Color primario oscuro */
  --secondary-color: #0d6efd; /* Color secundario oscuro */
  --border-color: #ffffff; /* Color de borde oscuro */
  --hover-background: rgba(29, 185, 84, 0.2); /* Color de fondo al pasar el ratón en oscuro */
}

body {
  background-color: var(--background-color);
  color: var(--text-color);
  transition: background-color 0.3s, color 0.3s; /* Transiciones suaves */
  font-family: 'Arial', sans-serif; /* Fuente más moderna */
  line-height: 1.6; /* Mejora la legibilidad */
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  user-select: none;
}

.filters {
  margin-bottom: 20px;
}

.search-input,
.filter-select {
  margin-right: 10px;
  padding: 10px;
  border: 1px solid var(--border-color);
  border-radius: 5px;
  transition: border 0.3s;
}

.search-input:focus,
.filter-select:focus {
  border-color: var(--primary-color);
  outline: none;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

th,
td {
  border: 1px solid var(--border-color);
  padding: 12px; /* Mayor espacio interno */
  text-align: left;
  transition: background-color 0.3s; /* Para transiciones suaves */
}

th {
  background-color: var(--primary-color);
  color: #ffffff;
}

td {
  background-color: var(--background-color);
  color: var(--text-color);
}

/* Estilos para celdas de tabla en tema oscuro */
.dark-theme th {
  background-color: var(--primary-color);
}

.dark-theme td {
  background-color: #333; /* Fondo gris oscuro */
  color: #ffffff; /* Texto blanco */
}

/* Efecto hover para las celdas */
td:hover {
  background-color: rgba(76, 175, 80, 0.5); /* Color verde con algo de transparencia */
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
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.modal-content {
  background-color: rgba(255, 255, 255, 0.95); /* Fondo blanco con un poco de opacidad */
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.7); /* Sombra más oscura y más grande */
  max-width: 600px;
  width: 90%;
  max-height: 80%;
  overflow-y: auto;
}

h2,
h3 {
  margin: 0 0 10px;
  color: var(--text-color);
}

.character-info {
  display: flex;
  align-items: center;
  justify-content: center;
}

.family-image,
.class-image {
  width: 50px;
  height: auto;
  margin: 0 10px;
}

.close-button {
  background-color: var(--secondary-color);
  color: white;
  border: none;
  border-radius: 5px;
  padding: 10px 15px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.close-button:hover {
  background-color: darken(var(--secondary-color), 10%);
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 10px;
  margin-bottom: 20px;
}

.info-item {
  padding: 10px;
  border-radius: 5px;
  border: 1px solid rgba(0, 0, 0, 0.1); /* Borde suave alrededor de los elementos */
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.2); /* Sombra suave */
}

.info-grid,
.info-item {
  background-color: rgba(255, 255, 255, 0.9); /* Fondo blanco más opaco para los elementos internos */
}

.modal-content h2,
.modal-content h3 {
  color: #333; /* Color de texto más oscuro */
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
