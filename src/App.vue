<template>
  <div :class="['app-container', isDarkMode ? 'dark' : 'light']">
    <header class="header">
      <div class="header-content">
        <img src="/src/assets/logo.png" alt="Logo Showroom" class="logo" />
        <h1 class="main-title">Form Aspirasi Pelanggan</h1>
      </div>
      <button class="toggle-dark" @click="toggleDarkMode">
        {{ isDarkMode ? '🌙' : '☀️' }}
      </button>
    </header>

    <svg class="svg-decor top-right" viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
      <path fill="#f8d7da" d="M43.7,-64.7..." transform="translate(100 100)" />
    </svg>
    <svg class="svg-decor bottom-left" viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
      <path fill="#f8d7da" d="M43.7,-64.7..." transform="translate(100 100)" />
    </svg>

    <Homepage />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Homepage from './views/HomePage.vue'

const isDarkMode = ref(false)
function toggleDarkMode() {
  isDarkMode.value = !isDarkMode.value
  document.body.classList.toggle('dark', isDarkMode.value) // ✅ ini bikin semua dark style jalan
}

</script>

<style scoped>
.app-container {
  font-family: 'Segoe UI', sans-serif;
  position: relative;
  min-height: 100vh;
  transition: background 0.3s ease;
}

.light {
  background-color: #ffffff;
  color: #000000;
}

.dark {
  background-color: #1e1e1e;
  color: #ffffff;
}

.dark input, .dark textarea { color: white; background-color: #333; }


.header {
  position: sticky;
  top: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 1.5rem;
  background-color: inherit;
  z-index: 10;
}

.header-content {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.logo {
  width: 40px;
  height: auto;
}

.main-title {
  font-size: 1.5rem;
  color: #d32f2f;
  margin: 0;
}

.toggle-dark {
  background: transparent;
  border: 1px solid #ccc;
  font-size: 1rem;
  padding: 4px 10px;
  border-radius: 5px;
  cursor: pointer;
}

.form-wrapper {
  padding: 1rem;
}

.form-container {
  background-color: rgba(255, 0, 0, 0.03);
  border-radius: 16px;
  padding: 1.5rem;
  max-width: 600px;
  margin: 0 auto;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
}

.form-title {
  text-align: center;
  font-size: 1.2rem;
  color: #d32f2f;
  margin-bottom: 1rem;
}

input,
select,
textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 1rem;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-size: 1rem;
}

textarea {
  resize: vertical;
  min-height: 100px;
}

.submit-btn {
  background-color: #d32f2f;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  width: 100%;
  font-size: 1rem;
}

progress {
  width: 100%;
  margin-bottom: 1rem;
  height: 10px;
  border-radius: 8px;
}

.svg-decor {
  position: absolute;
  width: 300px;
  height: 300px;
  opacity: 0.3;
  z-index: 0;
  pointer-events: none;
}

.top-right {
  top: 0;
  right: 0;
}

.bottom-left {
  bottom: 0;
  left: 0;
}


/* Responsive */
@media (max-width: 600px) {
  .header {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
  }
  .main-title {
    font-size: 1.2rem;
  }
  .logo {
    width: 35px;
  }
  .toggle-dark {
    align-self: flex-end;
  }
}
</style>
