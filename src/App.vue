<template>
  <div :class="['app-container']">
    <Homepage />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Homepage from './views/HomePage.vue'



</script>

<style scoped>
.app-container {
  font-family: 'Segoe UI', sans-serif;
  position: relative;
  min-height: 100vh;
  transition: background 0.3s ease;

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
