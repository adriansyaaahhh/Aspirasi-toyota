<template>
  <div class="upload-container">
    <!-- Tombol Upload Kustom -->
    <button type="button" class="custom-upload-btn" @click="triggerFileInput">
      Pilih Gambar
    </button>

    <!-- Input file disembunyikan -->
    <input
      type="file"
      accept="image/*"
      multiple
      ref="fileInput"
      @change="handleUpload"
      class="hidden-input"
    />

    <!-- Preview Gambar -->
    <div class="preview-wrapper">
      <div
        class="image-preview"
        v-for="(gambar, index) in gambarList"
        :key="index"
      >
        <img :src="gambar.url" :alt="'Gambar ' + (index + 1)" />
        <button class="hapus-btn" @click="hapusGambar(index)">×</button>
        <p class="file-name">{{ gambar.name }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const gambarList = ref([])
const fileInput = ref(null)

function triggerFileInput() {
  fileInput.value?.click()
}

function handleUpload(e) {
  const files = Array.from(e.target.files)

  files.forEach((file) => {
    const reader = new FileReader()
    reader.onload = () => {
      gambarList.value.push({
        url: reader.result,
        name: file.name // simpan nama file
      })
    }
    reader.readAsDataURL(file)
  })

  // Reset input supaya bisa upload ulang file yang sama
  fileInput.value.value = ''
}

function hapusGambar(index) {
  gambarList.value.splice(index, 1)

  if (gambarList.value.length === 0) {
    fileInput.value.value = ''
  }
}
</script>

<style scoped>
.upload-container {
  margin: 1rem 0;
}

.custom-upload-btn {
  background-color: #d32f2f;
  color: white;
  border: none;
  padding: 10px 16px;
  font-size: 1rem;
  border-radius: 10px;
  cursor: pointer;
  margin-bottom: 1rem;
}

.custom-upload-btn:hover {
  background-color: #b71c1c;
}

.hidden-input {
  display: none;
}

.preview-wrapper {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.image-preview {
  position: relative;
  width: 120px;
  height: 150px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 0 8px rgba(0, 0, 0, 0.2);
  text-align: center;
}

.image-preview img {
  width: 100%;
  height: 120px;
  object-fit: cover;
}

.file-name {
  font-size: 12px;
  margin-top: 4px;
  color: #333;
  word-break: break-word;
  padding: 0 4px;
}

.hapus-btn {
  position: absolute;
  top: 5px;
  right: 5px;
  background: rgba(0, 0, 0, 0.6);
  color: white;
  border: none;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s ease;
}

.hapus-btn:hover {
  background: rgba(255, 0, 0, 0.8);
}
</style>
