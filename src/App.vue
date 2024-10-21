<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue';

const errorMessage = ref<string | null>(null);
const baseUrl_API = 'ws-rt-clock.onrender.com/';

const currentTime = ref(new Date());
let intervalId: ReturnType<typeof setInterval>;

const updateTime = () => {
  currentTime.value = new Date(); // Actualiza la hora actual
};

const startClock = () => {
  updateTime(); // Llama una vez al inicio
  intervalId = setInterval(updateTime, 1000); // Actualiza cada segundo
};

startClock();

const formattedTime = computed(() => {
  const seconds = currentTime.value.getSeconds();
  const minutes = currentTime.value.getMinutes();
  const hours = currentTime.value.getHours() % 12; // Modulo 12 para el formato de 12 horas
  const xhours = hours.toString().padStart(2, '0');
  const xminutes = minutes.toString().padStart(2, '0');
  const xseconds = seconds.toString().padStart(2, '0');
  return `${xhours}:${xminutes}:${xseconds}`; // Retorna el formato HH:MM:SS
});

const userInput = ref<string[]>([]);
let timer: ReturnType<typeof setTimeout>;
const isKeys = ref(false);

async function handleKeyPress(event: KeyboardEvent) {
  // Si ya se ha autenticado, no procesar más teclas
  if (isKeys.value) {
    return;
  }
  // Agregar la tecla presionada al array
  userInput.value.push(event.key);
  // Reiniciar el temporizador cada vez que se presiona una tecla
  clearTimeout(timer);
  timer = setTimeout(async () => {
    await sendKeysToAPI();
  }, 3000); // Espera 3 segundos sin pulsaciones
}

async function sendKeysToAPI() {
  try {
    const response = await fetch('https://' + baseUrl_API + 'unlock', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userInput.value),
    });
    if (response.ok) {
      isKeys.value = true; // Cambiar a true si la secuencia es correcta
    } else {
      userInput.value = []; // Limpiar el input si la secuencia es incorrecta
    }
  } catch (error: unknown) {
    //console.error('Error en la solicitud:', error);
    console.log('qwe');
  } finally {
    userInput.value = []; // Limpiar la lista de teclas después de la solicitud
  }
}

interface TokenResponse {
  access_token: string;
  token_type: string;
}

const username = ref<string>('');
const password = ref<string>('');
const isAuthenticated = ref<boolean>(false);
const tokenData = ref<TokenResponse | null>(null)
const access_token = ref<string>('');

async function handleLogin() {
  try {
    const response = await fetch('https://' + baseUrl_API + 'token', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: new URLSearchParams({
        username: username.value,
        password: password.value,
      }),
    });
    if (response.ok) {
      isAuthenticated.value = true; // Autenticado
      tokenData.value = await response.json(); // Guardar token de acceso
      access_token.value = tokenData.value!.access_token; // Guardar token de acceso
      errorMessage.value = ''; // Limpiar mensaje de error
      // Limpiar campos de entrada
      username.value = '';
      password.value = '';
      fetchData(); // Cargar datos si está autenticado
    } else {
      // Manejar errores del servidor
      const errorResponse = await response.json();
      errorMessage.value = errorResponse.detail || 'Access Denied'; // Mostrar error específico
    }
  } catch (error: unknown) { // Especificar el tipo de error como unknown
    // Manejar errores de conexión
    if (error instanceof Error) {
      errorMessage.value = 'An error occurred: ' + error.message; // Usar message de Error
    } else {
      errorMessage.value = 'An unknown error occurred.'; // Mensaje genérico para otros tipos de error
    }
    //console.error('Error en la solicitud:', error);
    console.log('asd');
  }
}

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
const classImages = ref<{ [key: string]: string }>({});
const familyImages = ref<{ [key: string]: string }>({});
const classImage = ref<string>('');
const familyImage = ref<string>('');

const searchQuery = ref('');
const selectedClass = ref('');
const selectedFamily = ref('');
const selectedGuild = ref('');
const selectedMap = ref('');
const selectedCharacter = ref<Character | null>(null);

const isLoading = ref(false);
const connectionStatus = ref<'connecting' | 'connected' | 'error' | 'fallback'>('connecting');
let socket: WebSocket | null = null;
let reconnectTimeout: any = null;
let fallbackTimeout: any = null;
let reconnectAttempts = 0;
const FALLBACK_DURATION = 30000;

const fetchData = () => {
  isLoading.value = true;
  connectionStatus.value = 'connecting';

  // Verifica si ya existe una conexión abierta y la cierra si es necesario
  if (socket && socket.readyState === WebSocket.OPEN) {
    socket.close();
  }

  socket = new WebSocket(`wss://${baseUrl_API}ws/scrape/?token=${access_token.value}`);
  // Evento que se ejecuta cuando se abre la conexión
  socket.onopen = () => {
    connectionStatus.value = 'connected';
    reconnectAttempts = 0;
    errorMessage.value = '';	// Limpiar mensaje de error
    if (fallbackTimeout) {
      clearTimeout(fallbackTimeout); // Clear fallback timeout on success
      fallbackTimeout = null;
    }
  };

  // Evento que se ejecuta cuando se recibe un mensaje desde el WebSocket
  socket!.onmessage = (event) => {
    try {
      const data = JSON.parse(event.data);
      // Validación de datos antes de usarlos
      if (data && typeof data === 'object') {
        
        Object.entries(data).forEach(([id, info]: [string, any]) => {
        const existingCharacterIndex = characters.value.findIndex(character => character.id === id);
        
        // Si el personaje ya existe, actualiza sus datos
        if (existingCharacterIndex !== -1) {
          characters.value[existingCharacterIndex] = {
            id,
            'Información del Personaje': info['Información del Personaje'] || {},
            'Información Gens': info['Información Gens'] || {},
            'Información del Guild': info['Información del Guild'] || {},
          };
        } else {
          // Si el personaje no existe, agrégalo a la lista
          characters.value.push({
            id,
            'Información del Personaje': info['Información del Personaje'] || {},
            'Información Gens': info['Información Gens'] || {},
            'Información del Guild': info['Información del Guild'] || {},
          });
        }
      });
        preloadImages();
      } else {
        //console.warn('Received invalid data:', data);
        console.log('zxc');
      }
    } catch (e) {
      //console.error('Error parsing WebSocket message:', e);
      console.log('rty');
    }
  };

  // Evento que se ejecuta cuando ocurre un error en la conexión WebSocket
  socket.onerror = () => {//(error) => {
    //console.error('WebSocket error:', error);
    console.log('fgh');
    handleConnectionError();
    errorMessage.value = 'Failed to connect to WebSocket. Please try again later.';
    isLoading.value = false;
  };

  // Evento que se ejecuta cuando la conexión WebSocket se cierra
  socket.onclose = () => {//(event) => {
    //console.log('WebSocket connection closed', event);
    console.log('vbn');
    handleConnectionError();
    isLoading.value = false;
    reconnect();
  };
};

if (isAuthenticated.value) {
  fetchData(); // Llama a la función fetchData si está autenticado
}

const reconnect = () => {
  if (reconnectAttempts < 5) {  // Limitar número de reconexiones
    reconnectAttempts++;
    setTimeout(() => {
      //console.log(`Reconnecting... Attempt ${reconnectAttempts}`);
      fetchData();  // Volver a intentar la conexión
    }, 3000);  // Esperar 3 segundos antes de reconectar
  } else {
    //console.error('Max reconnection attempts reached');
    errorMessage.value = 'Haz clic en el botón para Recargar/Actualizar los DATOS.';
  }
};

const openModal = async (character: Character) => {
  selectedCharacter.value = character;
  classImage.value = await getClassImage(character['Información del Personaje'].Clase);
  familyImage.value = await getFamilyImage(character['Información Gens'].Familia);
};

const copyLocation = async (character: Character) => {
  // Persona | Ubicación | Guild
  const message = `${character['Información del Personaje'].Personaje} | ${character['Información del Personaje'].Ubicación
    } | ${character['Información del Guild'].Guild}`;
  await navigator.clipboard.writeText(message);
  alert('Location copied to clipboard!');
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

const uniqueMaps = computed(() => {
  const maps = new Set(
    characters.value.map((char) => {
      const ubicacion = char['Información del Personaje'].Ubicación;
      const mapName = ubicacion.replace(/\s*\(.*\)$/, '').trim();
      return mapName;
    })
  );
  return ['', ...Array.from(maps)];
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
    const matchesMap =
      selectedMap.value === '' ||
      char['Información del Personaje'].Ubicación.includes(selectedMap.value);

    return matchesSearch && matchesClass && matchesFamily && matchesGuild && matchesMap;
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
  errorMessage.value = 'Haz clic en el botón para Recargar/Actualizar los DATOS.';
  scheduleReconnect();

  if (!fallbackTimeout) {
    fallbackTimeout = setTimeout(() => {
      handleFallback();
    }, FALLBACK_DURATION);
  }
};

const handleFallback = () => {
  connectionStatus.value = 'fallback';
  characters.value = [];
  if (reconnectTimeout) {
    clearTimeout(reconnectTimeout);
    reconnectTimeout = null;
  }
  if (fallbackTimeout) {
    clearTimeout(fallbackTimeout);
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
  intervalId = setInterval(updateTime, 1000); // Llama a updateTime cada segundo
  window.addEventListener('keydown', handleKeyPress);
});

onUnmounted(() => {
  clearInterval(intervalId); // Limpia el intervalo si fuera necesario
  window.removeEventListener('keydown', handleKeyPress);
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

const reloadData = () => {
  fetchData(); // Llama a la función fetchData para recargar los datos
};
</script>

<template>
  <div class="digital-clock" v-if="!isKeys">
    <div class="time-display">{{ formattedTime }}</div> <!-- Muestra el tiempo formateado -->
  </div>
  <div v-else>
    <div class="content" v-if="!isAuthenticated">
      <h2>Sign In</h2>
      <div class="form">
        <div class="inputBox">
          <input type="text" v-model="username" required /> <i>Username</i>
        </div>
        <div class="inputBox">
          <input type="password" v-model="password" required /> <i>Password</i>
        </div>
        <div class="inputBox">
          <input type="submit" value="Login" @click.prevent="handleLogin" />
        </div>
        <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
      </div>
    </div>

    <div v-else class="container">
      <h1 style="color: greenyellow;">Tracking Table</h1>
      <div class="inputBox">
        <input type="submit" @click="reloadData" value=" 🔃 ">
      </div>
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
        <select v-model="selectedMap" class="filter-select">
          <option value="">All Maps</option>
          <option v-for="map in uniqueMaps" :key="map" :value="map">
            {{ map }}
          </option>
        </select>
      </div>
      <div class="table-container">
        <table v-if="filteredCharacters.length > 0">
          <thead>
            <tr>
              <th
                style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                Character</th>
              <th
                style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                Class</th>
              <th
                style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                Location</th>
              <th
                style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                Gens</th>
              <th
                style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                Guild</th>
              <th
                style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                Options</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(character) in filteredCharacters" :key="character.id">
              <td
                style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                {{ character['Información del Personaje'].Personaje }}</td>
              <td
                style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                <img :src="classImages[character['Información del Personaje'].Clase]" alt="Class Image"
                  v-if="classImages[character['Información del Personaje'].Clase]" />
              </td>
              <td
                style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                {{ character['Información del Personaje'].Ubicación }}</td>
              <td
                style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                <img :src="familyImages[character['Información Gens'].Familia]" alt="Family Image"
                  v-if="familyImages[character['Información Gens'].Familia]" />
              </td>
              <td
                style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                {{ character['Información del Guild'].Guild }}</td>
              <td>
                <div class="inputBox">
                  <input type="submit" @click="openModal(character)" value=" 🕵️ ">
                </div>
                <div class="inputBox">
                  <input type="submit" @click="copyLocation(character)" value=" 🚩 ">
                </div>
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
      </div>
      <div v-if="selectedCharacter" class="modal" @click.self="closeModal">
        <div class="modal-content">
          <div class="character-info">
            <img :src="familyImage" alt="Family Image" class="family-image" />
            <h1 style="color: greenyellow;">{{ selectedCharacter['Información del Personaje'].Personaje }}</h1>
            <img :src="classImage" alt="Class Image" class="class-image" />
          </div>
          <table class="table-container">
            <tbody>
              <tr>
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                  <strong>Guild:</strong>
                </td>
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                  {{ selectedCharacter['Información del Guild'].Guild }}
                </td>
              </tr>
              <tr>
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                  <strong>Location:</strong>
                </td>
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                  {{ selectedCharacter['Información del Personaje'].Ubicación }}</td>
              </tr>
              <tr>
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                  <strong>Log Login:</strong>
                </td>
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                  {{ selectedCharacter['Información del Personaje']['Último Ingreso'] }}</td>
              </tr>
              <tr>
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                  <strong>Level:</strong>
                </td>
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
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
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                  <strong>Stats Pack:</strong>
                </td>
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
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
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
                  <strong>Stats GR Full:</strong>
                </td>
                <td
                  style="color: greenyellow; text-shadow: 1px 1px 0px black, -1px -1px 0px black, -1px 1px 0px black, 1px -1px 0px black;">
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
          <div class="inputBox">
            <input type="submit" @click="closeModal" value=" ❌ ">
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Quicksand:wght@300;400;500;600;700&display=swap');

.table-container {
  max-height: 55vh;
  max-width: 80vh;
  /* Ajusta la altura máxima según sea necesario */
  overflow-y: auto;
  /* Habilita el desplazamiento vertical */
  margin-top: 20px;
  /* Espacio superior */
  border: 1px solid #ccc;
  /* Opcional: borde para visualizar mejor el contenedor */
  border-radius: 4px;
  /* Opcional: bordes redondeados */
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Quicksand', sans-serif;
}

/* Variables de color para tema claro y oscuro */
:root {
  --background-color: #ffffff;
  /* Color de fondo claro */
  --text-color: #333333;
  /* Color de texto claro */
  --primary-color: #4caf50;
  /* Color primario */
  --secondary-color: #007bff;
  /* Color secundario */
  --border-color: #333333;
  /* Color de borde */
  --hover-background: rgba(76, 175, 80, 0.2);
  /* Color de fondo al pasar el ratón */
}

.dark-theme {
  --background-color: #121212;
  /* Color de fondo oscuro */
  --text-color: #ffffff;
  /* Color de texto oscuro */
  --primary-color: #1db954;
  /* Color primario oscuro */
  --secondary-color: #0d6efd;
  /* Color secundario oscuro */
  --border-color: #ffffff;
  /* Color de borde oscuro */
  --hover-background: rgba(29, 185, 84, 0.2);
  /* Color de fondo al pasar el ratón en oscuro */
}

body {
  color: var(--text-color);
  transition: background-color 0.3s, color 0.3s;
  /* Transiciones suaves */
  font-family: 'Arial', sans-serif;
  /* Fuente más moderna */
  line-height: 1.6;
  /* Mejora la legibilidad */
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: #000;
}

section {
  position: absolute;
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 2px;
  flex-wrap: wrap;
  overflow: hidden;
}

section::before {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  background: linear-gradient(#000, #0f0, #000);
  animation: animate 5s linear infinite;
}

@keyframes animate {
  0% {
    transform: translateY(-100%);
  }

  100% {
    transform: translateY(100%);
  }
}

section span {
  position: relative;
  display: block;
  width: calc(6.25vw - 2px);
  height: calc(6.25vw - 2px);
  background: #181818;
  z-index: 2;
  transition: 1.5s;
}

section span:hover {
  background: #0f0;
  transition: 0s;
}

section .signin {
  position: absolute;
  width: 400px;
  background: #222;
  z-index: 1000;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px;
  border-radius: 4px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 9);
}

section .signin .content {
  position: relative;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  gap: 40px;
}

section .signin .content h2 {
  font-size: 2em;
  color: #0f0;
  text-transform: uppercase;
}

section .signin .content .form {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 25px;
}

section .signin .content .form .inputBox {
  position: relative;
  width: 100%;
}

section .signin .content .form .inputBox input {
  position: relative;
  width: 100%;
  background: #333;
  border: none;
  outline: none;
  padding: 25px 10px 7.5px;
  border-radius: 4px;
  color: #fff;
  font-weight: 500;
  font-size: 1em;
}

section .signin .content .form .inputBox i {
  position: absolute;
  left: 0;
  padding: 15px 10px;
  font-style: normal;
  color: #aaa;
  transition: 0.5s;
  pointer-events: none;
}

.signin .content .form .inputBox input:focus~i,
.signin .content .form .inputBox input:valid~i {
  transform: translateY(-7.5px);
  font-size: 0.8em;
  color: #fff;
}

.signin .content .form .links {
  position: relative;
  width: 100%;
  display: flex;
  justify-content: space-between;
}

.signin .content .form .links a {
  color: #fff;
  text-decoration: none;
}

.signin .content .form .links a:nth-child(2) {
  color: #0f0;
  font-weight: 600;
}

.signin .content .form .inputBox input[type="submit"] {
  padding: 10px;
  background: #0f0;
  color: #000;
  font-weight: 600;
  font-size: 1.35em;
  letter-spacing: 0.05em;
  cursor: pointer;
}

input[type="submit"]:active {
  opacity: 0.6;
}

@media (max-width: 900px) {
  section span {
    width: calc(10vw - 2px);
    height: calc(10vw - 2px);
  }
}

@media (max-width: 600px) {
  section span {
    width: calc(20vw - 2px);
    height: calc(20vw - 2px);
  }
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
  border-bottom: 1px solid #ddd;
  /* Línea divisoria entre filas */
  padding: 8px;
  /* Mayor espacio interno */
  text-align: left;
  transition: background-color 0.3s;
  /* Para transiciones suaves */
}

th {
  background-color: #f2f2f2;
  /* Color de fondo para el encabezado */
  color: #88a07e;
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
  background-color: #333;
  /* Fondo gris oscuro */
  color: #ffffff;
  /* Texto blanco */
}

/* Efecto hover para las celdas */
td:hover {
  background-color: rgba(76, 175, 80, 0.5);
  /* Color verde con algo de transparencia */
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
  background-color: #333;
  /* Fondo blanco con un poco de opacidad */
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.7);
  /* Sombra más oscura y más grande */
  max-width: 600px;
  width: 90%;
  max-height: 80%;
  overflow-y: auto;
}

h1,
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
  border: 1px solid rgba(0, 0, 0, 0.1);
  /* Borde suave alrededor de los elementos */
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.2);
  /* Sombra suave */
}

.info-grid,
.info-item {
  background-color: rgba(255, 255, 255, 0.9);
  /* Fondo blanco más opaco para los elementos internos */
}

.modal-content h2,
.modal-content h3 {
  color: #333;
  /* Color de texto más oscuro */
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

section .signin .content .form .inputBox {
  position: relative;
  width: 100%;
}

section .signin .content .form .inputBox input {
  position: relative;
  width: 100%;
  background: #333;
  border: none;
  outline: none;
  padding: 25px 10px 7.5px;
  border-radius: 4px;
  color: #fff;
  font-weight: 500;
  font-size: 1em;
}

section .signin .content .form .inputBox i {
  position: absolute;
  left: 0;
  padding: 15px 10px;
  font-style: normal;
  color: #aaa;
  transition: 0.5s;
  pointer-events: none;
}

.signin .content .form .inputBox input:focus~i,
.signin .content .form .inputBox input:valid~i {
  transform: translateY(-7.5px);
  font-size: 0.8em;
  color: #fff;
}


/* Combine the clock and hour hand into one element */
.digital-clock {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #1c1c1c; /* Color de fondo negro oscuro */
  padding: 20px; /* Espaciado interno */
  border-radius: 15px; /* Bordes redondeados */
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.6); /* Sombra para dar profundidad */
  transition: background-color 0.3s ease; /* Transición suave para el fondo */
}

.digital-clock:hover {
  background-color: #333; /* Color de fondo al pasar el mouse */
}

.time-display {
  font-size: 4em; /* Tamaño de la fuente */
  color: #00ff00; /* Color verde fosforescente */
  font-family: 'Courier New', Courier, monospace; /* Fuente monoespaciada */
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5); /* Sombra en el texto para más legibilidad */
  animation: pulse 1s infinite; /* Efecto de pulso */
}

/* Animación de pulso */
@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

/* Estilo para texto secundario o etiquetas, si es necesario */
.secondary-text {
  font-size: 1.2em;
  color: #ccc; /* Color gris claro */
  margin-top: 10px;
}

.filters {
  width: 80vh;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  /* Espaciado entre los elementos */
  margin: 20px 0;
  /* Margen superior e inferior */
  padding: 10px;
  border: 1px solid #ccc;
  /* Borde alrededor del contenedor */
  border-radius: 8px;
  /* Esquinas redondeadas */
  background-color: rgba(255, 255, 255, 0.9);
  /* Fondo semitransparente */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  /* Sombra sutil */
}

.search-input,
.filter-select {
  padding: 8px 12px;
  /* Espaciado interno */
  border: 1px solid #aaa;
  /* Borde de los inputs */
  border-radius: 4px;
  /* Esquinas redondeadas */
  font-size: 16px;
  /* Tamaño de fuente */
  outline: none;
  /* Sin contorno al hacer foco */
  transition: border-color 0.3s;
  /* Transición suave para el borde */
  color: #333;
  /* Color del texto */
  background-color: #f9f9f9;
  /* Fondo de los inputs */
}

.search-input:focus,
.filter-select:focus {
  border-color: greenyellow;
  /* Color del borde al enfocar */
  box-shadow: 0 0 5px rgba(0, 255, 85, 0.5);
  /* Sombra al enfocar */
}

/* Mejora de la legibilidad del texto */
.filter-select option {
  padding: 10px;
  /* Espaciado en las opciones del select */
  color: #333;
  /* Color del texto de las opciones */
}

/* Sombra sutil en el texto */
.filter-select,
.search-input {
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
  /* Sombra sutil para mejorar la legibilidad */
}
</style>
