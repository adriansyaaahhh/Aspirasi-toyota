<template>
  <div class="form-wrapper">
    <div v-if="!selectedRole" class="role-selector">
      <h2>Pilih Peran Anda</h2>
      <button @click="selectRole('customer')" class="role-btn">Pelanggan</button>
      <button @click="selectRole('employee')" class="role-btn">Karyawan</button>
    </div>

    <!-- Tampilkan form berdasarkan peran -->
    <FormAspirasi
      v-if="selectedRole"
      :role="selectedRole"
      @kembaliKeRole="selectedRole = ''"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import FormAspirasi from '../components/AspirasiForm.vue'

const selectedRole = ref('')
function selectRole(role) {
  selectedRole.value = role
}
</script>

<style scoped>
.role-selector {
  text-align: center;
  margin-top: 2rem;
}

.role-btn {
  margin: 10px;
  padding: 10px 20px;
  background-color: #d32f2f;
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 1rem;
  cursor: pointer;
}
</style>
