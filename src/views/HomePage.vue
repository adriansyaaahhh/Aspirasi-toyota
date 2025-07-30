<template>
  <div class="form-wrapper">
    
    <!-- ✅ Hanya tampil saat belum pilih role -->
    <section v-if="!selectedRole" class="hero-section">
      <div class="hero-content">
        <h1 class="hero-title">Selamat Datang di Sistem Aspirasi</h1>
        <p class="hero-subtitle">Pilih peran Anda untuk melanjutkan ke form kepuasan pelanggan</p>
      </div>
    </section>

    <!-- ✅ Pilih role (hilang jika sudah pilih) -->
    <section v-if="!selectedRole" class="role-selector">
      <div class="card-container">
        <div class="role-card" @click="selectRole('customer')">
          <img src="/src/assets/customer.png" alt="Customer" class="role-icon" />
          <h3>Pelanggan</h3>
        </div>
        <div class="role-card" @click="selectRole('employee')">
          <img src="/src/assets/karyawan.png" alt="Karyawan" class="role-icon" />
          <h3>Karyawan</h3>
        </div>
      </div>
    </section>

    <!-- ✅ Form akan tampil setelah role dipilih -->
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
.form-wrapper {
  padding: 2rem;
}

.hero-section {
  background: linear-gradient(rgba(255,255,255,0.8), rgba(255,255,255,0.8)), url('/src/assets/bg.') center/cover no-repeat;
  padding: 3rem 1rem;
  text-align: center;
  border-radius: 16px;
  margin-bottom: 2rem;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
}

.hero-title {
  font-size: 2rem;
  color: #d32f2f;
  margin-bottom: 0.5rem;
}

.hero-subtitle {
  font-size: 1.1rem;
  color: #555;
}

.role-selector {
  text-align: center;
}

.card-container {
  display: flex;
  justify-content: center;
  gap: 2rem;
  flex-wrap: wrap;
  margin-top: 2rem;
}

.role-card {
  background-color: #fff;
  border-radius: 16px;
  padding: 1.5rem;
  width: 200px;
  cursor: pointer;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease, box-shadow 0.3s ease;
  text-align: center;
}

.role-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2);
}

.role-icon {
  width: 64px;
  height: 64px;
  margin-bottom: 1rem;
}

h3 {
  color: #d32f2f;
  font-size: 1.2rem;
  margin: 0;
}

@media (max-width: 600px) {
  .hero-title {
    font-size: 1.5rem;
  }
  .hero-subtitle {
    font-size: 1rem;
  }
  .role-card {
    width: 90%;
  }
}
</style>
