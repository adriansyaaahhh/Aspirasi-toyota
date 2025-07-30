<template>
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
          type="text"
          placeholder="Nomor Telepon"
          @keypress="onlyNumber"
          @input="validateTelepon"
          :class="{ 'input-error': teleponError }"
        />
        <p v-if="teleponError" class="error-message">Masukkan angka yang valid</p>

        <select v-model="form.layanan">
          <option disabled value="">Pilih layanan</option>
          <option>Servis</option>
          <option>Jual-Beli</option>
          <option>Test Drive</option>
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
      <input  ref="fileInput" type="file" multiple accept="image/*" @change="handleFileChange" />
      <div class="preview-wrapper">
        <div v-for="(img, index) in previewGambar" :key="index" class="preview-box">
          <img :src="img" alt="Preview" class="preview-img" />
          <button type="button" class="remove-btn" @click="hapusGambar(index)">×</button>
        </div>
      </div>

      <!-- Pelanggan: Ingin dihubungi? -->
      <div v-if="role === 'customer'" class="follow-up-group">
        <label class="follow-up-label">Apakah Anda ingin dihubungi untuk tindak lanjut? *</label>
        <div class="follow-up-options">
          <label :class="{ selected: form.tindakLanjut === 'Ya' }">
            <input type="radio" value="Ya" v-model="form.tindakLanjut" required /> Ya
          </label>
          <label :class="{ selected: form.tindakLanjut === 'Tidak' }">
            <input type="radio" value="Tidak" v-model="form.tindakLanjut" /> Tidak
          </label>
          <label :class="{ selected: form.tindakLanjut === 'Mungkin' }">
            <input type="radio" value="Mungkin" v-model="form.tindakLanjut" /> Mungkin
          </label>
        </div>
      </div>

      <button type="submit" class="submit-btn">Kirim Aspirasi</button>
    </form>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from 'vue'

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

  const token = '8166400026:AAG9E_KMp0_H_wXgGNxtdeBkcsi-i7zurmk'
  const chatId = '892508199'

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
.form-aspirasi {
  background: #fff;
  padding: 1.5rem;
  border-radius: 16px;
  box-shadow: 0 0 10px rgba(0,0,0,0.1);
  max-width: 600px;
  margin: auto;
}
.form-title {
  text-align: center;
  color: #d32f2f;
  margin-bottom: 1rem;
  font-size: 1.4rem;
}
input, select, textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 1rem;
  border-radius: 10px;
  border: 1px solid #ccc;
  font-size: 1rem;
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
  padding: 12px;
  font-size: 1rem;
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
  font-size: 1rem;
  font-weight: 600;
  margin-bottom: 0.8rem;
  color: #111;
}
.dark .follow-up-label {
  color: #fff; 
}
.follow-up-options {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.follow-up-options label {
  display: flex;
  align-items: center;
  gap: 8px;
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
  font-weight: 600;
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
