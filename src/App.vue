<template>
  <div class="page">
    <!-- ===== HERO ALBUM CARD ===== -->
    <div class="album-card">
      <img src="./assets/weeding.jpeg" alt="Wedding" class="album-image" />
      <div class="album-overlay">
        <h1 class="names">Bruk & Tsion</h1>
        <p class="date">January 10, 2026</p>
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

    <!-- ===== UPLOAD BUTTON ===== -->
    <div v-if="selectedFiles.length" class="upload-box">
      <button class="btn upload" :disabled="uploading" @click="uploadVideos">
        <span v-if="!uploading">
          ✨ Upload {{ selectedFiles.length }} Video{{
            selectedFiles.length > 1 ? "s" : ""
          }}
        </span>

        <span v-else class="spinner-wrap">
          <span class="spinner"></span>
          Uploading…
        </span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

/* ================= AXIOS ================= */
const api = axios.create({
  baseURL: "", // same origin (http://196.190.251.44:5000)
  timeout: 0,
});

/* ================= STATE ================= */
const selectedFiles = ref([]);
const uploading = ref(false);

/* ================= FILE SELECT ================= */
const onFilesChange = (e) => {
  const files = Array.from(e.target.files);
  if (!files.length) return;
  selectedFiles.value = files;
};

/* ================= UPLOAD (FAST) ================= */
const uploadVideos = async () => {
  uploading.value = true;

  try {
    const uploads = selectedFiles.value.map((file) => {
      const formData = new FormData();
      formData.append("video", file);

      return api.post("/upload", formData, {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      });
    });

    await Promise.all(uploads);

    alert("🎉 Upload completed successfully!");
    selectedFiles.value = [];
  } catch (err) {
    console.error(err);
    alert("❌ Upload failed. Please try again.");
  } finally {
    uploading.value = false;
  }
};

/* ================= VIEW ALBUM ================= */
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
  position: relative;
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
.upload {
  background: #c29607;
  color: white;
}

.upload-box {
  margin-top: 30px;
}

/* Spinner */
.spinner-wrap {
  display: flex;
  align-items: center;
  gap: 10px;
}

.spinner {
  width: 18px;
  height: 18px;
  border: 3px solid rgba(255, 255, 255, 0.4);
  border-top: 3px solid white;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
