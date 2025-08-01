<template>
  <div class="container mt-4">
    <div class="row">
      <div class="col-lg-8 mx-auto">
        <div class="card shadow">
          <div class="card-header bg-primary text-white">
            <h2 class="card-title mb-0">
              <i class="bi bi-image"></i>
              Image Text Editor
            </h2>
          </div>
          <div class="card-body">
            <!-- File Upload Section -->
            <div class="mb-4">
              <label for="imageUpload" class="form-label fw-bold">Upload Image</label>
              <input
                type="file"
                class="form-control"
                id="imageUpload"
                accept="image/*"
                @change="handleImageUpload"
              />
              <div class="form-text">Select an image file (JPG, PNG, GIF)</div>
            </div>

            <!-- Text Input Section -->
            <div class="row mb-4">
              <div class="col-md-6">
                <label for="nameInput" class="form-label fw-bold">Name</label>
                <input
                  type="text"
                  class="form-control"
                  id="nameInput"
                  v-model="textSettings.name"
                  placeholder="Enter name"
                  @input="updateCanvas"
                />
              </div>
              <div class="col-md-6">
                <label for="companyInput" class="form-label fw-bold">Company</label>
                <input
                  type="text"
                  class="form-control"
                  id="companyInput"
                  v-model="textSettings.company"
                  placeholder="Enter company"
                  @input="updateCanvas"
                />
              </div>
            </div>

            <!-- Position Controls -->
            <div class="row mb-4">
              <div class="col-md-3">
                <label for="nameX" class="form-label fw-bold">Name X Position</label>
                <input
                  type="number"
                  class="form-control"
                  id="nameX"
                  v-model.number="textSettings.nameX"
                  min="0"
                  @input="updateCanvas"
                />
              </div>
              <div class="col-md-3">
                <label for="nameY" class="form-label fw-bold">Name Y Position</label>
                <input
                  type="number"
                  class="form-control"
                  id="nameY"
                  v-model.number="textSettings.nameY"
                  min="0"
                  @input="updateCanvas"
                />
              </div>
              <div class="col-md-3">
                <label for="companyX" class="form-label fw-bold">Company X Position</label>
                <input
                  type="number"
                  class="form-control"
                  id="companyX"
                  v-model.number="textSettings.companyX"
                  min="0"
                  @input="updateCanvas"
                />
              </div>
              <div class="col-md-3">
                <label for="companyY" class="form-label fw-bold">Company Y Position</label>
                <input
                  type="number"
                  class="form-control"
                  id="companyY"
                  v-model.number="textSettings.companyY"
                  min="0"
                  @input="updateCanvas"
                />
              </div>
            </div>

            <!-- Text Style Controls -->
            <div class="row mb-4">
              <div class="col-md-4">
                <label for="fontSize" class="form-label fw-bold">Font Size</label>
                <input
                  type="range"
                  class="form-range"
                  id="fontSize"
                  v-model.number="textSettings.fontSize"
                  min="12"
                  max="72"
                  @input="updateCanvas"
                />
                <div class="form-text">{{ textSettings.fontSize }}px</div>
              </div>
              <div class="col-md-4">
                <label for="textColor" class="form-label fw-bold">Text Color</label>
                <input
                  type="color"
                  class="form-control form-control-color"
                  id="textColor"
                  v-model="textSettings.color"
                  @input="updateCanvas"
                />
              </div>
              <div class="col-md-4">
                <label for="fontWeight" class="form-label fw-bold">Font Weight</label>
                <select
                  class="form-select"
                  id="fontWeight"
                  v-model="textSettings.fontWeight"
                  @change="updateCanvas"
                >
                  <option value="lighter">Lighter</option>
                  <option value="normal">Normal</option>
                  <option value="bold">Bold</option>
                </select>
              </div>
            </div>
            <!-- Download Button -->
            <div class="text-center" v-if="imageLoaded">
              <button
                class="btn btn-success btn-lg"
                @click="downloadImage"
                :disabled="!textSettings.name && !textSettings.company"
              >
                <i class="bi bi-download"></i>
                Download Edited Image
              </button>
              <div class="form-text mt-2" v-if="!textSettings.name && !textSettings.company">
                Add name or company text to enable download
              </div>
            </div>
            <!-- Canvas Container -->
            <div class="text-center mb-4" v-if="imageLoaded">
              <div class="border rounded p-3 bg-light">
                <canvas
                  ref="canvas"
                  :width="canvasWidth"
                  :height="canvasHeight"
                  class="border shadow-sm"
                  style="max-width: 100%; height: auto;"
                  @click="handleCanvasClick"
                ></canvas>
                <div class="form-text mt-2">
                  Click on the canvas to position text
                </div>
              </div>
            </div>

            <!-- Instructions -->
            <div class="alert alert-info mt-4" v-if="!imageLoaded">
              <h5 class="alert-heading">
                <i class="bi bi-info-circle"></i>
                How to use:
              </h5>
              <ol class="mb-0">
                <li>Upload an image using the file input above</li>
                <li>Enter the name and company text</li>
                <li>Adjust the X and Y positions manually or click on the image</li>
                <li>Customize text appearance with font size, color, and weight</li>
                <li>Download your edited image</li>
              </ol>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, nextTick, onMounted } from 'vue'
import defaultImage from './assets/invitation.jpg'

// Reactive data
const canvas = ref<HTMLCanvasElement | null>(null)
const imageLoaded = ref(false)
const canvasWidth = ref(900)
const canvasHeight = ref(1600)
const uploadedImage = ref<HTMLImageElement | null>(null)
const originalSize = reactive({ width: 0, height: 0 })

// Drag state
// Drag functionality has been removed

const textSettings = reactive({
  name: '',
  company: '',
  nameX: 530,
  nameY: 1150,
  companyX: 530,
  companyY: 1200,
  fontSize: 28,
  color: '#ffffff',
  fontWeight: 'bold',
  fontFamily: 'Nata Sans' as string
})

// Handle image upload
const handleImageUpload = (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  
  if (file) {
    const reader = new FileReader()
    reader.onload = (e) => {
      const img = new Image()
      img.onload = () => {
        uploadedImage.value = img
        
        // Calculate canvas dimensions to maintain aspect ratio
        const maxWidth = 900
        const maxHeight = 1600
        let { width, height } = img
        
        if (width > maxWidth) {
          height = (height * maxWidth) / width
          width = maxWidth
        }
        
        if (height > maxHeight) {
          width = (width * maxHeight) / height
          height = maxHeight
        }
        
        canvasWidth.value = width
        canvasHeight.value = height
        
        // Reset text positions to be within new dimensions
        textSettings.nameX = Math.min(textSettings.nameX, width - 100)
        textSettings.nameY = Math.min(textSettings.nameY, height - 50)
        textSettings.companyX = Math.min(textSettings.companyX, width - 100)
        textSettings.companyY = Math.min(textSettings.companyY, height - 50)
        
        imageLoaded.value = true
        
        nextTick(() => {
          updateCanvas()
        })
      }
      img.src = e.target?.result as string
    }
    reader.readAsDataURL(file)
  }
}

  function wrapText(
  ctx: CanvasRenderingContext2D,
  text: string,
  x: number,
  y: number,
  maxWidth: number,
  lineHeight: number
) {
  const words = text.split(' ')
  let line = ''
  for (let n = 0; n < words.length; n++) {
    const testLine = line ? line + ' ' + words[n] : words[n]
    const { width } = ctx.measureText(testLine)
    if (width > maxWidth && n > 0) {
      ctx.fillText(line, x, y)
      y += lineHeight
      line = words[n]
    } else {
      line = testLine
    }
  }
  ctx.fillText(line, x, y)
}
  
// Update canvas with image and text
const updateCanvas = () => {
  if (!canvas.value || !uploadedImage.value) return
  
  const ctx = canvas.value.getContext('2d')
  if (!ctx) return
  
  // Clear canvas
  ctx.clearRect(0, 0, canvasWidth.value, canvasHeight.value)
  
  // Draw image
  ctx.drawImage(uploadedImage.value, 0, 0, canvasWidth.value, canvasHeight.value)
  
  // Set text properties
  ctx.font = `${textSettings.fontWeight} ${textSettings.fontSize}px "${textSettings.fontFamily}", sans-serif`
  ctx.fillStyle = textSettings.color
  ctx.strokeStyle = '#ffffff'
  ctx.lineWidth = 2
  
  // Draw name text with outline for better visibility
  if (textSettings.name) {
    // ctx.strokeText(textSettings.name, textSettings.nameX, textSettings.nameY)
    // ctx.fillText(textSettings.name, textSettings.nameX, textSettings.nameY)
      const maxWidth   = 300           // <-- your fixed wrap width in px
      const lineHeight = textSettings.fontSize * 1.2
      wrapText(
        ctx,
        textSettings.name,
        textSettings.nameX,
        textSettings.nameY,
        maxWidth,
        lineHeight
      )
  }
  
  // Draw company text with outline for better visibility
  if (textSettings.company) {
    //ctx.strokeText(textSettings.company, textSettings.companyX, textSettings.companyY)
    //ctx.fillText(textSettings.company, textSettings.companyX, textSettings.companyY)

      const maxWidth   = 300           // <-- your fixed wrap width in px
      const lineHeight = textSettings.fontSize * 1.2
      wrapText(
        ctx,
        textSettings.company,
        textSettings.companyX,
        textSettings.companyY,
        maxWidth,
        lineHeight
      )
  }
}

// Handle canvas click to position text
const handleCanvasClick = (event: MouseEvent) => {
  if (!canvas.value) return
  
  const coords = getCanvasCoordinates(event)
  
  // Determine which text to position based on current selection
  // For simplicity, we'll alternate between name and company
  if (!textSettings.name) {
    textSettings.nameX = coords.x
    textSettings.nameY = coords.y
  } else if (!textSettings.company) {
    textSettings.companyX = coords.x
    textSettings.companyY = coords.y
  } else {
    // If both texts exist, position the one that's closer to the click
    const nameDistance = Math.sqrt(
      Math.pow(coords.x - textSettings.nameX, 2) + Math.pow(coords.y - textSettings.nameY, 2)
    )
    const companyDistance = Math.sqrt(
      Math.pow(coords.x - textSettings.companyX, 2) + Math.pow(coords.y - textSettings.companyY, 2)
    )
    
    if (nameDistance < companyDistance) {
      textSettings.nameX = coords.x
      textSettings.nameY = coords.y
    } else {
      textSettings.companyX = coords.x
      textSettings.companyY = coords.y
    }
  }
  
  updateCanvas()
}

// Drag and drop functionality has been removed

// Get canvas coordinates from mouse event
const getCanvasCoordinates = (event: MouseEvent) => {
  if (!canvas.value) return { x: 0, y: 0 }
  
  const rect = canvas.value.getBoundingClientRect()
  const scaleX = canvasWidth.value / rect.width
  const scaleY = canvasHeight.value / rect.height
  
  return {
    x: Math.round((event.clientX - rect.left) * scaleX),
    y: Math.round((event.clientY - rect.top) * scaleY)
  }
}

// Drag functionality has been removed

// Drag functionality has been removed

// Drag functionality has been removed

// Download the edited image
const downloadImage = () => {
  if (!canvas.value) {
    console.log('canvas is null');
    return;
  }

  // redraw to be safe
  updateCanvas();

  // 1. Grab a data-URL of the current canvas
  const dataUrl = canvas.value.toDataURL('image/jpg');

  // 2. Create an <a> that we can “click” to download
  const link = document.createElement('a');
  link.href = dataUrl;
  link.download = `${textSettings.name} Invitation.jpg`;       // file name

  // 3. Append, click, and remove
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
};

onMounted(() => {
  const img = new Image()
  img.src = defaultImage
  img.onload = () => {
    // reuse your existing logic for sizing:
    uploadedImage.value   = img
    originalSize.width    = img.naturalWidth
    originalSize.height   = img.naturalHeight

    // compute display canvasWidth/Height exactly as in handleImageUpload
    const maxW = 900, maxH = 1600
    let w = img.naturalWidth, h = img.naturalHeight
    if (w > maxW) { h = (h * maxW) / w; w = maxW }
    if (h > maxH) { w = (w * maxH) / h; h = maxH }
    canvasWidth.value  = w
    canvasHeight.value = h

    imageLoaded.value = true
    nextTick(updateCanvas)
  }
})

</script>
<style scoped>
.card {
  border: none;
}

.form-control:focus,
.form-select:focus {
  border-color: #0d6efd;
  box-shadow: 0 0 0 0.2rem rgba(13, 110, 253, 0.25);
}

canvas {
  cursor: crosshair;
  transition: transform 0.2s ease;
  user-select: none;
}

canvas:hover {
  transform: scale(1.02);
}

.btn:disabled {
  opacity: 0.6;
}

.alert-info {
  border-left: 4px solid #0dcaf0;
}

.form-range::-webkit-slider-thumb {
  background-color: #0d6efd;
}

.form-range::-moz-range-thumb {
  background-color: #0d6efd;
  border: none;
}
</style>