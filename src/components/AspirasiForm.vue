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
      
      <!-- Input file untuk iOS (behavior default) -->
      <input 
        v-if="isIOS" 
        ref="fileInput" 
        type="file" 
        multiple 
        accept="image/*" 
        @change="handleFileChange" 
      />
      
      <!-- Input file tersembunyi untuk Android -->
      <template v-else>
        <input 
          ref="fileInput" 
          type="file" 
          multiple 
          accept="image/*" 
          @change="handleFileChange" 
          @click="handleAndroidFileClick"
          style="display: none;"
        />
        
        <input 
          ref="cameraInput" 
          type="file" 
          accept="image/*" 
          capture="environment"
          @change="handleFileChange" 
          style="display: none;"
        />
        
        <input 
          ref="galleryInput" 
          type="file" 
          multiple
          accept="image/*"
          @change="handleFileChange" 
          style="display: none;"
        />
        
        <!-- Button yang terlihat untuk Android -->
        <button type="button" class="android-file-btn" @click="showAndroidOptions">
          📁 Pilih File
          <span class="file-status">{{ previewGambar.length > 0 ? `${previewGambar.length} file dipilih` : 'tidak ada file yang dipilih' }}</span>
        </button>
      </template>
      
      <div class="preview-wrapper">
        <div v-for="(img, index) in previewGambar" :key="index" class="preview-box">
          <img :src="img" alt="Preview" class="preview-img" />
          <button type="button" class="remove-btn" @click="hapusGambar(index)">×</button>
        </div>
      </div>

      <!-- Android Options Modal -->
      <div v-if="showOptions && !isIOS" class="modal-overlay" @click="closeOptions">
        <div class="modal-content" @click.stop>
          <div class="modal-header">
            <h3>Pilih Sumber Gambar</h3>
            <button class="modal-close" @click="closeOptions">×</button>
          </div>
          <div class="option-list">
            <button class="option-item" @click="selectCamera">
              <span class="option-icon">📷</span>
              <span class="option-text">Ambil Foto</span>
            </button>
            <button class="option-item" @click="selectGallery">
              <span class="option-icon">🖼️</span>
              <span class="option-text">Pilih dari Galeri</span>
            </button>
            <button class="option-item" @click="selectFiles">
              <span class="option-icon">📁</span>
              <span class="option-text">Pilih File</span>
            </button>
          </div>
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
import { ref, defineProps, defineEmits, onMounted } from 'vue'

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
const isIOS = ref(false)
const showOptions = ref(false)

// Deteksi platform
onMounted(() => {
  const userAgent = navigator.userAgent || navigator.vendor || window.opera
  isIOS.value = /iPad|iPhone|iPod/.test(userAgent) && !window.MSStream
})

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
const cameraInput = ref(null)
const galleryInput = ref(null)

// Fungsi untuk Android - mencegah file picker default
function handleAndroidFileClick(e) {
  e.preventDefault()
  showAndroidOptions()
}

// Show Android options modal
function showAndroidOptions() {
  showOptions.value = true
}

function closeOptions() {
  showOptions.value = false
}

// Android option handlers
function selectCamera() {
  closeOptions()
  setTimeout(() => {
    cameraInput.value.click()
  }, 100)
}

function selectGallery() {
  closeOptions()
  setTimeout(() => {
    galleryInput.value.click()
  }, 100)
}

function selectFiles() {
  closeOptions()
  setTimeout(() => {
    fileInput.value.click()
  }, 100)
}

function hapusGambar(index) {
  gambarFiles.value.splice(index, 1)
  previewGambar.value.splice(index, 1)
  if (gambarFiles.value.length === 0) {
    if (fileInput.value) fileInput.value.value = ''
    if (cameraInput.value) cameraInput.value.value = ''
    if (galleryInput.value) galleryInput.value.value = ''
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
  
  // Reset input values setelah memproses file
  event.target.value = ''
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
  top: 60px;
  background: white;
  padding: 0.5rem 1rem;
  z-index: 9;
  text-align: left;
  box-shadow: none;
  border-radius: 6px;
}

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

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-4px); }
  to { opacity: 1; transform: translateY(0); }
}

.form-aspirasi {
  background: #fff;
  padding: 1rem;
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

/* Android File Button */
.android-file-btn {
  width: 100%;
  padding: 12px;
  margin-bottom: 1rem;
  border-radius: 10px;
  border: 1px solid #ccc;
  background: white;
  font-size: 0.95rem;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: all 0.2s;
}

.android-file-btn:hover {
  border-color: #d32f2f;
  background: #fafafa;
}

.file-status {
  color: #666;
  font-size: 0.85rem;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: flex-end;
  z-index: 1000;
  animation: modalFadeIn 0.3s ease;
}

.modal-content {
  background: white;
  width: 100%;
  max-width: 500px;
  border-radius: 16px 16px 0 0;
  padding: 0;
  animation: modalSlideUp 0.3s ease;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px 16px;
  border-bottom: 1px solid #eee;
}

.modal-header h3 {
  margin: 0;
  color: #333;
  font-size: 1.1rem;
  font-weight: 600;
}

.modal-close {
  background: none;
  border: none;
  font-size: 24px;
  color: #999;
  cursor: pointer;
  padding: 0;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all 0.2s;
}

.modal-close:hover {
  background: #f5f5f5;
  color: #666;
}

.option-list {
  padding: 8px 0 24px;
}

.option-item {
  width: 100%;
  padding: 16px 24px;
  border: none;
  background: none;
  display: flex;
  align-items: center;
  gap: 16px;
  cursor: pointer;
  transition: background 0.2s;
  font-size: 1rem;
}

.option-item:hover {
  background: #f8f8f8;
}

.option-item:active {
  background: #f0f0f0;
}

.option-icon {
  font-size: 24px;
  width: 40px;
  display: flex;
  justify-content: center;
}

.option-text {
  color: #333;
  font-weight: 500;
}

@keyframes modalFadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes modalSlideUp {
  from { transform: translateY(100%); }
  to { transform: translateY(0); }
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

.follow-up-options {
  display: flex;
  justify-content: flex-start;  
  flex-wrap: wrap;
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

.input-error {
  border-color: red;
  background-color: #ffe5e5;
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