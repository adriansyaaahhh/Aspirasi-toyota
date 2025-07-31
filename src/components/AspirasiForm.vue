<template>
      <header class="header">
      <div class="header-content">
        <img src="/src/assets/logo.png" alt="Logo Showroom" class="logo" />
        <h1 class="main-title">
  {{ role === 'customer' ? 'Form Aspirasi Pelanggan' : 'Form Aspirasi Karyawan' }}
</h1>

      </div>

    </header>
  <h2 class="form-title">Silakan Isi Aspirasi Anda</h2>
  <div class="form-aspirasi">
    <!-- Notifikasi toast -->
    <div v-if="showSuccess" class="toast success-toast">
      ✅ Aspirasi Anda telah berhasil dikirim!
    </div>

    <!-- Tombol kembali -->
    <button type="button" class="back-btn" @click="goBack">← Kembali</button>
    <form @submit.prevent="submitForm">
      <!-- Pelanggan -->
      <template v-if="role === 'customer'">
        <input v-model="form.nama" type="text" placeholder="Masukkan nama Anda" />
        <input v-model="form.noPolisi" type="text" placeholder="Nomor Polisi" />
        <input
          v-model="form.telepon"
          type="tel"
          placeholder="Nomor Telepon"
          @keypress="onlyNumber"
          @input="validateTelepon"
          :class="{ 'input-error': teleponError }"
        />
        <div v-if="teleponError" class="input-alert">
  ⚠️ Harap masukkan angka yang valid (hanya 0-9).
</div>

        <select v-model="form.layanan">
          <option disabled value="">Pilih layanan</option>
          <option>Servis</option>
          <option>Sales</option>
          <option>Sparepart</option>
          <option>Tukar Tambah</option>
          <option>Internal Toyota</option>
          <option>Lain lain</option>
        </select>
      </template>

      <!-- Karyawan -->
      <template v-else-if="role === 'employee'">
        <input v-model="form.nama" type="text" placeholder="Masukkan nama Anda" />
        <input v-model="form.jabatan" type="text" placeholder="Jabatan" />
      </template>

      <!-- Komentar wajib -->
      <textarea v-model="form.komentar" placeholder="Komentar atau Saran" required></textarea>

      <!-- Upload Gambar -->
      <label>Upload Gambar</label>
      <div v-if="isAndroid">
        <button type="button" class="upload-btn" @click="showAndroidOptions = !showAndroidOptions">
          📁 Pilih File
        </button>

        <div v-if="showAndroidOptions" class="android-options">
          <button type="button" @click="triggerFile('camera')">📷 Ambil Foto</button>
          <button type="button" @click="triggerFile('gallery')">🖼️ Pilih dari Galeri</button>
        </div>
      </div>

      <!-- iOS or default devices -->
      <div v-else>
        <input
          type="file"
          accept="image/*"
          multiple
          @change="handleFileChange"
        />
      </div>

      <input
        ref="fileInput"
        type="file"
        accept="image/*"
        multiple
        style="display: none"
        @change="handleFileChange"
      />

      <div class="preview-wrapper">
        <div v-for="(img, index) in previewGambar" :key="index" class="preview-box">
          <img :src="img" alt="Preview" class="preview-img" />
          <button type="button" class="remove-btn" @click="hapusGambar(index)">×</button>
        </div>
        
      </div>

      <!-- Pelanggan: Ingin dihubungi? -->
<div class="follow-up-group" v-if="role === 'customer'">
  <label class="follow-up-label">Apakah Anda ingin dihubungi untuk tindak lanjut? *</label>
  <div class="follow-up-options">
    <label class="follow-option" :class="{ selected: form.tindakLanjut === 'Ya' }">
      <input type="radio" value="Ya" v-model="form.tindakLanjut" required />
      <span>Ya</span>
    </label>
    <label class="follow-option" :class="{ selected: form.tindakLanjut === 'Tidak' }">
      <input type="radio" value="Tidak" v-model="form.tindakLanjut" />
      <span>Tidak</span>
    </label>
    <label class="follow-option" :class="{ selected: form.tindakLanjut === 'Mungkin' }">
      <input type="radio" value="Mungkin" v-model="form.tindakLanjut" />
      <span>Mungkin</span>
    </label>
  </div>
</div>


      <button type="submit" class="submit-btn">Kirim Aspirasi</button>
    </form>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from 'vue'
import { onMounted, watch,reactive } from 'vue'

const isAndroid = ref(false)
const showAndroidOptions = ref(false)


onMounted(() => {
  isAndroid.value = /android/i.test(navigator.userAgent)
})

const cameraInput = ref(null)
const galleryInput = ref(null)


function triggerFile(source) {
  showAndroidOptions.value = false
  if (fileInput.value) {
    if (source === 'camera') {
      fileInput.value.setAttribute('accept', 'image/*')
      fileInput.value.setAttribute('capture', 'environment')
    } else {
      fileInput.value.setAttribute('accept', 'image/*')
      fileInput.value.removeAttribute('capture')
    }
    fileInput.value.click()
  }
}

const props = defineProps({ role: String })
const emit = defineEmits(['kembaliKeRole'])

function goBack() {
  emit('kembaliKeRole')
}

const form = ref({
  nama: '',
  jabatan: '',
  noPolisi: '',
  telepon: '',
  layanan: '',
  tindakLanjut: '',
  komentar: ''
})
const teleponError = ref(false)
const showSuccess = ref(false)

function validateTelepon() {
  const angkaRegex = /^[0-9]*$/
  teleponError.value = !angkaRegex.test(form.value.telepon)
}

function onlyNumber(e) {
  if (!/[0-9]/.test(e.key)) {
    e.preventDefault()
  }
}

const gambarFiles = ref([])
const previewGambar = ref([])
const fileInput = ref(null)

function hapusGambar(index) {
  gambarFiles.value.splice(index, 1)
  previewGambar.value.splice(index, 1)
    if (gambarFiles.value.length === 0 && fileInput.value) {
    fileInput.value.value = ''
  }
}

function handleFileChange(event) {
  const files = Array.from(event.target.files)
  for (const file of files) {
    gambarFiles.value.push(file)
    const reader = new FileReader()
    reader.onload = (e) => previewGambar.value.push(e.target.result)
    reader.readAsDataURL(file)
  }
}



async function submitForm() {
  if (!form.value.komentar.trim()) {
    alert("Komentar atau Saran wajib diisi!")
    return
  }

  // Tampilkan toast
  showSuccess.value = true
  setTimeout(() => {
    showSuccess.value = false
  }, 4000)

  const token = '7388709190:AAH8Vo2OFy5qWpR7S12oEOJFrUVXevwv6xI'
  const chatId = '-4855857854'

  let caption = ''
  if (props.role === 'customer') {
    caption += `🧾 *Aspirasi dari Pelanggan*\n`
    caption += `👤 Nama: ${form.value.nama || '-'}\n`
    caption += `🚗 No Polisi: ${form.value.noPolisi || '-'}\n`
    caption += `📞 Telepon: ${form.value.telepon || '-'}\n`
    caption += `🔧 Layanan: ${form.value.layanan || '-'}\n`
    caption += `📬 Tindak Lanjut: ${form.value.tindakLanjut || '-'}\n`
  } else {
    caption += `🧾 *Aspirasi dari Karyawan*\n`
    caption += `👤 Nama: ${form.value.nama || '-'}\n`
    caption += `🧑‍💼 Jabatan: ${form.value.jabatan || '-'}\n`
  }
  caption += `💬 Komentar: ${form.value.komentar}`

  if (gambarFiles.value.length > 0) {
    const mediaGroup = []
    const formData = new FormData()

    gambarFiles.value.forEach((file, index) => {
      const fileId = `photo${index}`
      formData.append(fileId, file)
      mediaGroup.push({
        type: "photo",
        media: `attach://${fileId}`,
        ...(index === 0 ? { caption: caption, parse_mode: "Markdown" } : {})
      })
    })

    formData.append("chat_id", chatId)
    formData.append("media", JSON.stringify(mediaGroup))

    await fetch(`https://api.telegram.org/bot${token}/sendMediaGroup`, {
      method: 'POST',
      body: formData
    })
  } else {
    await fetch(`https://api.telegram.org/bot${token}/sendMessage`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        chat_id: chatId,
        text: caption,
        parse_mode: "Markdown"
      })
    })
  }

  form.value = {
    nama: '',
    jabatan: '',
    noPolisi: '',
    telepon: '',
    layanan: '',
    tindakLanjut: '',
    komentar: ''
  }
  gambarFiles.value = []
  previewGambar.value = []
}
</script>

<style scoped>

.image-upload-toggle {
  display: flex;
  gap: 10px;
  margin-bottom: 1rem;
}
.upload-btn {
  flex: 1;
  padding: 8px 10px;
  font-size: 0.95rem;
  border: 1px solid #ccc;
  border-radius: 10px;
  background: #f5f5f5;
  cursor: pointer;
  transition: background 0.3s;
}
.upload-btn:hover {
  background-color: #e0e0e0;
}
.header {
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
  top: 60px; /* agar tidak nabrak header logo */
  background: white;
  padding: 0.5rem 1rem;
  z-index: 9;
  text-align: left;
  box-shadow: none;
  border-radius: 6px;
}
/* Responsive untuk layar mobile */
@media (max-width: 600px) {
  .main-title {
    font-size: 1.2rem;
    padding: 0.4rem 0.8rem;
    top: 56px;
  }
}

.input-alert {
  margin-top: -0.6rem;
  margin-bottom: 1rem;
  padding: 8px 12px;
  background-color: #fff3cd;
  color: #856404;
  border: 1px solid #ffeeba;
  border-radius: 8px;
  font-size: 0.95rem;
  display: flex;
  align-items: center;
  gap: 8px;
  animation: fadeIn 0.3s ease-in-out;
}

/* Animasi agar alert muncul lembut */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-4px); }
  to { opacity: 1; transform: translateY(0); }
}


.form-aspirasi {
  background: #fff;
  padding: 1 rem;
  border-radius: 16px;
  box-shadow: 0 0 10px rgba(0,0,0,0.1);
  max-width: 600px;
  margin: auto;
}
.form-title {
  text-align: center;
  color: #d32f2f;
  margin-bottom: 0.8rem;
  font-size: 1rem;
}
input, select, textarea {
  width: 100%;
  padding: 8px;
  margin-bottom: 1rem;
  border-radius: 10px;
  border: 1px solid #ccc;
  font-size: 0.95rem;
}
textarea { resize: vertical; }
.radio-group {
  display: flex;
  gap: 15px;
  margin-bottom: 1rem;
}
.preview-wrapper {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 1rem;
}
.preview-box {
  position: relative;
  width: 100px;
  height: 100px;
}
.preview-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 8px;
  border: 1px solid #ccc;
}
.remove-btn {
  position: absolute;
  top: 5px;
  right: 5px;
  background: transparent;
  color: #d32f2f;
  border: none;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  padding: 0;
  line-height: 1;
  transition: color 0.2s;
}

.remove-btn:hover {
  color: #9a0007;
  transform: scale(1.2);
}

.submit-btn {
  width: 100%;
  background-color: #d32f2f;
  color: white;
  padding: 10px;
  font-size: 0.95rem;
  border-radius: 10px;
  border: none;
  cursor: pointer;
}
.back-btn {
  background: none;
  border: none;
  color: #d32f2f;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  margin-bottom: 1rem;
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 4px 8px;
  border-radius: 8px;
  transition: background 0.2s;
}

.back-btn:hover {
  background: rgba(211, 47, 47, 0.1);
}


.peran-toggle {
  display: flex;
  gap: 1rem;
  justify-content: center;
  margin-bottom: 1rem;
}

.radio-group {
  margin-bottom: 1rem;
}

.radio-label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}
.follow-up-options input[type="radio"] {
  accent-color: #d32f2f;
  width: 16px;
  height: 16px;
  margin: 0;
  flex-shrink: 0;
}
.radio-options {
  display: flex;
  gap: 1.5rem;
  align-items: center;
  flex-wrap: wrap;
}
.follow-up-group {
  margin-bottom: 1.5rem;
}

.follow-up-label {
  display: block;
  font-size: 0.95rem;
  font-weight: 600;
  margin-bottom: 0.8rem;
  color: #111;
}
.dark .follow-up-label {
  color: #fff; 
}
.follow-up-options {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  justify-content: flex-start;  
  gap: 1rem;

}

.follow-option {
  flex: 1 1 100px;
  max-width: 100px;
  padding: 8px;
  border: 2px solid #ccc;
  border-radius: 12px;
  background: #fafafa;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s;
  position: relative;
}

.follow-option.selected {
  border-color: #d32f2f;
  background: #ffe5e5;
  font-weight: bold;
  color: #b71c1c;
}

.follow-option input[type="radio"] {



  border: 2px solid #d32f2f;
  border-radius: 50%;
  outline: none;
  margin-right: 8px;
  position: relative;
  top: 3px;
  cursor: pointer;
  accent-color: #d32f2f;
  transform: scale(1.2);
}

.follow-option input[type="radio"]:checked::before {
    content: '';    
    width: 10px;
    height: 10px;
    background: #d32f2f;
    border-radius: 50%;
    margin: auto;
    position: relative;
    top: 4px;
    left: 4px;
    accent-color: #d32f2f;
  transform: scale(1.2);
}

.follow-up-options label {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 10px 16px;
  border: 2px solid #ccc;
  border-radius: 10px;
  background-color: #fafafa;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  font-size: 1rem;
  color: #333;
  user-select: none;
}
.follow-up-options label:hover {
  border-color: #d32f2f;
  background-color: #fff1f1;
}

.follow-up-options label.selected {
  border-color: #d32f2f;
  background-color: #ffe5e5;
  color: #b71c1c;
  font-weight: 200;
}

.follow-up-options input[type="radio"] {
  accent-color: #d32f2f;
  transform: scale(1.2);
}

.input-error {
  border-color: red;
  background-color: #ffe5e5;
}

.error-message {
  color: red;
  font-size: 0.9rem;
  margin-top: -0.5rem;
  margin-bottom: 1rem;
}
.toast {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 9999;
  padding: 12px 20px;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 500;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  transition: all 0.3s ease;
  opacity: 0.95;
}
.success-toast {
  background-color: #4caf50;
  color: white;
}
</style>
