<template>
  <div class="wedding-container">
    <!-- Floating Decorations -->
    <div class="bg-decoration">
      <div class="leaf leaf-1"></div>
      <div class="leaf leaf-2"></div>
      <div class="leaf leaf-3"></div>
      <div class="ring ring-1"></div>
      <div class="ring ring-2"></div>
    </div>

    <div class="card">
      <h1 class="title">
        <span class="icon">💍</span> Share Your Wedding Memories
      </h1>
      <p class="subtitle">
        Upload all your beautiful videos from our special day. We can't wait to
        relive them with you!
      </p>

      <!-- Styled File Input (Mobile-Friendly) -->
      <label for="video-input" class="upload-btn">
        <span v-if="selectedFiles.length === 0"> 📹 Choose Videos </span>
        <span v-else-if="!hasJustUploaded">
          ✓ {{ selectedFiles.length }} video{{
            selectedFiles.length > 1 ? "s" : ""
          }}
          ready
        </span>
        <span v-else> ➕ Upload More Videos </span>
      </label>
      <input
        id="video-input"
        type="file"
        accept="video/*"
        multiple
        @change="onFilesChange"
        hidden
      />

      <!-- Preview Section -->
      <transition name="fade">
        <div
          v-if="previews.length > 0 && !hasJustUploaded"
          class="previews-grid"
        >
          <div
            v-for="(preview, index) in previews"
            :key="index"
            class="preview-item"
          >
            <video :src="preview.url" controls class="video-thumb"></video>
            <p class="filename">{{ preview.name }}</p>
          </div>
        </div>
      </transition>

      <!-- Upload Progress -->
      <div v-if="uploadProgress.length > 0" class="progress-list">
        <div
          v-for="item in uploadProgress"
          :key="item.id"
          class="progress-item"
        >
          <div class="progress-info">
            <span class="progress-name">{{ item.name }}</span>
            <span class="progress-percent">{{ item.progress }}%</span>
          </div>
          <div class="progress-bar">
            <div
              class="progress-fill"
              :style="{ width: item.progress + '%' }"
            ></div>
          </div>
        </div>
      </div>

      <!-- Main Upload Button -->
      <button
        @click="uploadVideos"
        class="submit-btn"
        :disabled="selectedFiles.length === 0 || uploading || hasJustUploaded"
      >
        <span v-if="!uploading && !hasJustUploaded">
          ✨ Upload {{ selectedFiles.length }} Video{{
            selectedFiles.length > 1 ? "s" : ""
          }}
        </span>
        <span v-else-if="uploading">
          Uploading... <span class="dots"></span>
        </span>
        <span v-else> ✓ All Videos Uploaded Successfully! </span>
      </button>

      <!-- Success Section -->
      <transition name="slide-up">
        <div v-if="uploadedVideos.length > 0" class="success-section">
          <h3 class="success-title">🎉 Thank You So Much!</h3>
          <p>
            Your {{ uploadedVideos.length }} video{{
              uploadedVideos.length > 1 ? "s have" : " has"
            }}
            been uploaded.<br />
            Feel free to add even more memories!
          </p>

          <div class="uploaded-grid">
            <div
              v-for="video in uploadedVideos"
              :key="video.filename"
              class="uploaded-item"
            >
              <video
                :src="video.url"
                controls
                class="video-player uploaded-video"
                autoplay
                muted
                loop
              ></video>
              <a :href="video.url" target="_blank" class="video-link">
                Watch Full Video →
              </a>
            </div>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

const selectedFiles = ref([]);
const previews = ref([]);
const uploading = ref(false);
const uploadProgress = ref([]);
const uploadedVideos = ref([]);
const hasJustUploaded = ref(false);

const onFilesChange = (e) => {
  const files = Array.from(e.target.files);
  if (files.length > 0) {
    selectedFiles.value = files;
    previews.value = files.map((file) => ({
      name: file.name,
      url: URL.createObjectURL(file),
    }));
    hasJustUploaded.value = false;
    uploadProgress.value = [];
  }
};

const uploadVideos = async () => {
  if (selectedFiles.value.length === 0 || hasJustUploaded.value) return;

  uploading.value = true;
  hasJustUploaded.value = false;

  uploadProgress.value = selectedFiles.value.map((file, index) => ({
    id: Date.now() + index,
    name: file.name,
    progress: 0,
  }));

  for (let i = 0; i < selectedFiles.value.length; i++) {
    const file = selectedFiles.value[i];
    const progressItem = uploadProgress.value[i];

    const formData = new FormData();
    formData.append("video", file);

    try {
      const response = await axios.post("/upload", formData, {
        onUploadProgress: (progressEvent) => {
          const percent = Math.round(
            (progressEvent.loaded * 100) / progressEvent.total
          );
          progressItem.progress = percent;
        },
      });

      uploadedVideos.value.push(response.data);
      progressItem.progress = 100;
    } catch (error) {
      console.error("Upload failed:", error);
      alert(
        `Upload failed for ${file.name}: ${error.message || "Network error"}`
      );
      progressItem.progress = 0;
    }
  } // ✅ CLOSES for-loop

  uploading.value = false;
  hasJustUploaded.value = true;
};
</script>

<style scoped>
/* Global Mobile Fixes */
* {
  -webkit-tap-highlight-color: transparent;
}

button,
label {
  touch-action: manipulation;
}

/* Main Container */
.wedding-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #f9f5f0 0%, #f0e9e0 50%, #e8dfd5 100%);
  position: relative;
  overflow-x: hidden;
  font-family: "Georgia", "Playfair Display", serif;
  padding: 20px;
}

/* Decorations */
.bg-decoration {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 0;
}

.leaf {
  position: absolute;
  background: linear-gradient(45deg, #d4a574, #c19a6b);
  border-radius: 0 100% 0 100%;
  opacity: 0.12;
  animation: float 18s infinite ease-in-out;
}

.leaf-1 {
  width: 50px;
  height: 70px;
  top: 10%;
  left: 8%;
  transform: rotate(20deg);
}
.leaf-2 {
  width: 60px;
  height: 80px;
  top: 25%;
  right: 12%;
  animation-delay: 6s;
}
.leaf-3 {
  width: 50px;
  height: 70px;
  bottom: 18%;
  left: 15%;
  animation-delay: 12s;
}

.ring {
  position: absolute;
  width: 40px;
  height: 40px;
  border: 6px solid #c19a6b;
  border-radius: 50%;
  opacity: 0.15;
  animation: pulse 5s infinite;
}

.ring-1 {
  top: 15%;
  right: 10%;
}
.ring-2 {
  bottom: 25%;
  left: 10%;
  animation-delay: 2s;
}

/* Card */
.card {
  max-width: 1000px;
  margin: 20px auto;
  background: rgba(255, 255, 252, 0.95);
  border-radius: 28px;
  padding: 40px 24px;
  box-shadow: 0 25px 70px rgba(180, 140, 100, 0.18);
  position: relative;
  z-index: 1;
  backdrop-filter: blur(8px);
  text-align: center;
}

.title {
  font-size: 2.8rem;
  color: #8b6f47;
  margin-bottom: 16px;
  font-weight: 700;
  line-height: 1.2;
}

.icon {
  display: inline-block;
  animation: gentle-bounce 3s infinite;
}

.subtitle {
  font-size: 1.3rem;
  color: #a67c52;
  margin-bottom: 40px;
  line-height: 1.6;
  padding: 0 10px;
}

/* Upload Button - Critical Mobile Fix */
.upload-btn {
  display: block;
  width: 80%;
  max-width: 400px;
  margin: 0 auto 40px;
  padding: 20px 16px;
  background: #d4a574;
  color: white;
  font-size: 1.4rem;
  font-weight: 600;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 10px 30px rgba(212, 165, 116, 0.3);
  user-select: none;
  min-height: 32px; /* Larger touch target */
  display: flex;
  align-items: center;
  justify-content: center;
}

.upload-btn:active {
  transform: scale(0.96);
  background: #c19a6b;
}

/* Previews */
.previews-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
  margin: 40px 0;
}

.preview-item {
  background: #fff;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.08);
}

.video-thumb {
  width: 100%;
  height: 180px;
  object-fit: cover;
  background: #000;
}

.filename {
  padding: 14px;
  font-size: 1rem;
  color: #8b6f47;
  margin: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Progress */
.progress-list {
  margin: 40px 0;
  max-width: 700px;
  margin-left: auto;
  margin-right: auto;
}

.progress-item {
  margin-bottom: 18px;
  background: #f5f0e8;
  border-radius: 12px;
  padding: 16px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
}

.progress-info {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
  font-weight: 600;
  color: #8b6f47;
  font-size: 1.1rem;
}

.progress-bar {
  height: 12px;
  background: #e8d9c5;
  border-radius: 6px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #d4a574, #c19a6b);
  width: 0%;
  transition: width 0.4s ease;
}

/* Submit Button */
.submit-btn {
  padding: 18px 50px;
  font-size: 1.5rem;
  font-weight: 600;
  background: linear-gradient(45deg, #b8860b, #d4a574);
  color: white;
  border: none;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 12px 35px rgba(184, 134, 11, 0.3);
  min-height: 64px;
  min-width: 220px;
  margin: 20px auto;
  display: block;
}

.submit-btn:active:not(:disabled) {
  transform: scale(0.95);
}

.submit-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
}

/* Success Section */
.success-section {
  margin-top: 60px;
  padding: 40px 20px;
  background: linear-gradient(135deg, #fdf8f0, #f5f0e8);
  border-radius: 24px;
  border: 3px dashed #d4a574;
}

.success-title {
  font-size: 2.2rem;
  color: #8b6f47;
  margin-bottom: 16px;
}

.uploaded-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 25px;
  margin-top: 30px;
}

.uploaded-video {
  width: 100%;
  max-height: 320px;
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  background: #000;
}

.video-link {
  display: block;
  margin-top: 14px;
  color: #b8860b;
  font-weight: 600;
  font-size: 1.1rem;
  text-decoration: none;
}

/* Animations */
@keyframes float {
  0%,
  100% {
    transform: translateY(0) rotate(0deg);
  }
  50% {
    transform: translateY(-25px) rotate(8deg);
  }
}

@keyframes pulse {
  0%,
  100% {
    transform: scale(1);
    opacity: 0.15;
  }
  50% {
    transform: scale(1.3);
    opacity: 0.25;
  }
}

@keyframes gentle-bounce {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-8px);
  }
}

@keyframes dots {
  0%,
  20% {
    content: ".";
  }
  40% {
    content: "..";
  }
  60%,
  100% {
    content: "...";
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.7s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-up-enter-active {
  transition: all 0.9s cubic-bezier(0.25, 0.8, 0.25, 1);
}
.slide-up-enter-from {
  opacity: 0;
  transform: translateY(50px);
}

/* Enhanced Mobile Responsive */
@media (max-width: 768px) {
  .card {
    padding: 30px 20px;
    margin: 10px;
    border-radius: 20px;
  }
  .title {
    font-size: 2.3rem;
  }
  .subtitle {
    font-size: 1.2rem;
  }
  .upload-btn,
  .submit-btn {
    font-size: 1.3rem;
    padding: 20px 16px;
    min-height: 68px;
  }
  .previews-grid,
  .uploaded-grid {
    grid-template-columns: 1fr;
    gap: 18px;
  }
  .progress-info {
    font-size: 1rem;
  }
}

@media (max-width: 480px) {
  .title {
    font-size: 2.1rem;
  }
  .upload-btn,
  .submit-btn {
    font-size: 1.2rem;
    padding: 18px 14px;
  }
}
</style>
