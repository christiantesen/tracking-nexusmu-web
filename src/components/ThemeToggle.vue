<template>
  <button @click="toggleTheme">{{ isDarkTheme ? 'Cambiar a Claro' : 'Cambiar a Oscuro' }}</button>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

const isDarkTheme = ref(false); // Estado del tema

// Cargar el tema guardado en localStorage al montar el componente
onMounted(() => {
  const savedTheme = localStorage.getItem('theme');
  isDarkTheme.value = savedTheme === 'dark';
  updateTheme(); // Aplicar el tema al cargar
});

// Función para alternar el tema
const toggleTheme = () => {
  isDarkTheme.value = !isDarkTheme.value;
  updateTheme();
};

// Función para aplicar el tema y guardar en localStorage
const updateTheme = () => {
  document.body.classList.toggle('dark-theme', isDarkTheme.value);
  localStorage.setItem('theme', isDarkTheme.value ? 'dark' : 'light');
};
</script>

<style scoped>
/* Estilos específicos para el botón de cambio de tema */
button {
  padding: 10px;
  margin: 20px;
  cursor: pointer;
}
</style>
