<template>
  <div class="container">
    <h2>Upload an Image to Extract Text</h2>
    <input type="file" @change="onFileChange" class="file-input" />

    <!-- Display the image preview with cropping functionality -->
    <div v-if="imageSrc" class="image-preview">
      <h3>Select Area to Extract Text:</h3>
      <div class="crop-container">
        <img :src="imageSrc" alt="Uploaded Image" ref="image" />
      </div>
      <button @click="cropImage" class="btn-crop">Crop & Extract Text</button>
    </div>

    <div v-if="loading" class="loading">Processing Image...</div>

    <div v-if="text" class="text-output">
      <h3>Extracted Text:</h3>
      <p>{{ text }}</p>
    </div>
  </div>
</template>

<script>
import Cropper from 'cropperjs';
import Tesseract from 'tesseract.js';
import 'cropperjs/dist/cropper.css';

export default {
  data() {
    return {
      text: '',
      loading: false,
      imageSrc: '', // To store the image source for preview
      cropper: null, // To hold the cropper instance
    };
  },
  methods: {
    onFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        this.previewImage(file); // Show image preview
      }
    },
    previewImage(file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        this.imageSrc = e.target.result; // Set the image source
        this.$nextTick(() => {
          this.initCropper(); // Initialize cropper after image is loaded
        });
      };
      reader.readAsDataURL(file);
    },
    initCropper() {
      if (this.cropper) {
        this.cropper.destroy(); // Destroy the old cropper if it exists
      }
      const imageElement = this.$refs.image;
      this.cropper = new Cropper(imageElement, {
        aspectRatio: 0, // Free aspect ratio
        viewMode: 1,
        autoCrop: true,
      });
    },
    cropImage() {
      this.loading = true;
      const croppedCanvas = this.cropper.getCroppedCanvas();
      croppedCanvas.toBlob((blob) => {
        this.extractTextFromImage(blob);
      });
    },
    extractTextFromImage(file) {
      Tesseract.recognize(
        file,
        'eng', // Language code, you can change this based on your needs
        {
          logger: (m) => console.log(m), // Log progress
        }
      )
        .then(({ data: { text } }) => {
          this.text = text;
          this.loading = false;
        })
        .catch((err) => {
          console.error('Error extracting text:', err);
          this.loading = false;
        });
    },
  },
};
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
  font-family: Arial, sans-serif;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

h2 {
  margin-bottom: 20px;
  color: #333;
}

.file-input {
  display: block;
  margin: 20px auto;
  padding: 10px;
  font-size: 16px;
}

.image-preview {
  margin-top: 20px;
}

.crop-container {
  width: 100%;
  height: 400px;
  margin-top: 10px;
  overflow: hidden;
  position: relative;
}

.crop-container img {
  max-width: 100%;
  display: block;
}

.btn-crop {
  display: inline-block;
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 16px;
  color: #fff;
  background-color: #007bff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-crop:hover {
  background-color: #0056b3;
}

.loading {
  margin: 20px 0;
  color: #007bff;
  font-size: 18px;
}

.text-output {
  margin-top: 20px;
}

.text-output p {
  background-color: #f8f9fa;
  padding: 15px;
  border-radius: 4px;
  font-size: 16px;
  color: #555;
}
</style>
