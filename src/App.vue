<template>
  <div class="page">
    <!-- ===== HERO ALBUM CARD ===== -->
    <div class="album-card">
      <img src="./assets/weeding.jpeg" alt="Wedding" class="album-image" />

      <div class="album-overlay">
        <h1 class="names">The Smiths</h1>
        <p class="date">June 18, 2025</p>
      </div>
    </div>

    <!-- ===== ACTION BUTTONS ===== -->
    <div class="actions">
      <label for="fileInput" class="btn primary"> ⬆ Upload Media </label>
      <input
        id="fileInput"
        type="file"
        accept="video/*"
        multiple
        hidden
        @change="onFilesChange"
      />

      <button class="btn secondary" @click="viewAlbum">🖼 View Album</button>
    </div>

    <!-- ===== PREVIEWS ===== -->
    <transition name="fade">
      <div v-if="previews.length && !hasJustUploaded" class="previews">
        <div v-for="(preview, i) in previews" :key="i" class="preview-item">
          <video :src="preview.url" controls></video>
          <p>{{ preview.name }}</p>
        </div>

        <button class="btn upload" :disabled="uploading" @click="uploadVideos">
          <span v-if="!uploading">
            ✨ Upload {{ selectedFiles.length }} Video{{
              selectedFiles.length > 1 ? "s" : ""
            }}
          </span>
          <span v-else>Uploading…</span>
        </button>
      </div>
    </transition>

    <!-- ===== PROGRESS ===== -->
    <div v-if="uploadProgress.length" class="progress-list">
      <div v-for="item in uploadProgress" :key="item.id">
        <div class="progress-info">
          <span>{{ item.name }}</span>
          <span>{{ item.progress }}%</span>
        </div>
        <div class="bar">
          <div class="fill" :style="{ width: item.progress + '%' }"></div>
        </div>
      </div>
    </div>

    <!-- ===== SUCCESS ===== -->
    <transition name="slide-up">
      <div v-if="uploadedVideos.length" class="success">
        <h3>🎉 Thank you!</h3>
        <p>{{ uploadedVideos.length }} video(s) uploaded successfully.</p>

        <div class="uploaded-grid">
          <div
            v-for="video in uploadedVideos"
            :key="video.filename"
            class="uploaded-item"
          >
            <video :src="video.url" controls muted loop></video>
            <a :href="video.url" target="_blank"> Watch full → </a>
          </div>
        </div>

        <label for="fileInput" class="btn upload-more"> ➕ Upload More </label>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

const api = axios.create({
  baseURL: "http://196.190.251.44:9000",
});

const selectedFiles = ref([]);
const previews = ref([]);
const uploading = ref(false);
const uploadProgress = ref([]);
const uploadedVideos = ref([]);
const hasJustUploaded = ref(false);

/* ===== FILE SELECT ===== */
const onFilesChange = (e) => {
  const files = Array.from(e.target.files);
  if (!files.length) return;

  selectedFiles.value = files;
  previews.value = files.map((f) => ({
    name: f.name,
    url: URL.createObjectURL(f),
  }));

  hasJustUploaded.value = false;
  uploadProgress.value = [];
};

/* ===== UPLOAD ===== */
const uploadVideos = async () => {
  uploading.value = true;

  uploadProgress.value = selectedFiles.value.map((f, i) => ({
    id: Date.now() + i,
    name: f.name,
    progress: 0,
  }));

  for (let i = 0; i < selectedFiles.value.length; i++) {
    const file = selectedFiles.value[i];
    const progressItem = uploadProgress.value[i];

    const formData = new FormData();
    formData.append("video", file);

    try {
      const res = await api.post("/upload", formData, {
        headers: {
          "Content-Type": "multipart/form-data",
        },
        onUploadProgress: (e) => {
          if (e.total) {
            progressItem.progress = Math.round((e.loaded * 100) / e.total);
          }
        },
      });

      uploadedVideos.value.push(res.data);
      progressItem.progress = 100;
    } catch (err) {
      progressItem.progress = 0;
      alert(`Failed to upload ${file.name}`);
    }
  }

  uploading.value = false;
  hasJustUploaded.value = true;
  previews.value = [];
};

/* ===== VIEW ALBUM ===== */
const viewAlbum = () => {
  alert("Navigate to album page");
};
</script>

<style scoped>
.page {
  padding: 20px;
  background: #f4f4f4;
  min-height: 100vh;
}

/* Album card */
.album-card {
  border-radius: 22px;
  overflow: hidden;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.15);
}

.album-image {
  width: 100%;
  height: 380px;
  object-fit: cover;
}

.album-overlay {
  position: absolute;
  bottom: 18px;
  left: 18px;
  color: white;
}

.names {
  margin: 0;
  font-size: 1.8rem;
}

.date {
  margin: 4px 0 0;
}

/* Buttons */
.actions {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.btn {
  height: 56px;
  border-radius: 14px;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.primary,
.secondary,
.upload,
.upload-more {
  background: #8fa98c;
  color: white;
}

.previews {
  margin-top: 30px;
}

.preview-item video {
  width: 100%;
  border-radius: 12px;
}

.progress-list {
  margin-top: 20px;
}

.bar {
  height: 10px;
  background: #ddd;
  border-radius: 5px;
  overflow: hidden;
}

.fill {
  height: 100%;
  background: #8fa98c;
}

.success {
  margin-top: 40px;
  padding: 20px;
  background: white;
  border-radius: 16px;
}

.uploaded-grid video {
  width: 100%;
  border-radius: 12px;
}

/* Animations */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-up-enter-active {
  transition: all 0.5s ease;
}
.slide-up-enter-from {
  opacity: 0;
  transform: translateY(30px);
}
</style>
