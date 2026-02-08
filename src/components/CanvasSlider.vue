<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from 'vue'

const canvasRef = ref<HTMLCanvasElement | null>(null)

// Canvas state
let ctx: CanvasRenderingContext2D | null = null
let rafId: number | null = null

// Canvas size
const CANVAS_WIDTH = 600
const CANVAS_HEIGHT = 380

const state = {
  dpr: 1, // Safe default
  needsRender: true, // Force first render
}

// Images
const images: HTMLImageElement[] = []
let imagesLoaded = 0
const imageUrls = [
  'https://images.unsplash.com/photo-1587300003388-59208cc962cb?w=800&h=600&fit=crop',
  'https://images.unsplash.com/photo-1552053831-71594a27632d?w=600&h=800&fit=crop',
  'https://images.unsplash.com/photo-1534361960057-19889db9621e?w=900&h=500&fit=crop',
  'https://images.unsplash.com/photo-1583511655857-d19b40a7a54e?w=700&h=700&fit=crop',
  'https://images.unsplash.com/photo-1477884213360-7e9d7dcc1e48?w=850&h=550&fit=crop',
]

// Slider state - free scrolling model
let scrollX = 0
let isDragging = false
let dragStartX = 0
let dragStartScrollX = 0

function loadImages() {
  imageUrls.forEach((url) => {
    const img = new Image()
    img.crossOrigin = 'anonymous'
    img.onload = () => {
      imagesLoaded++
      if (imagesLoaded === imageUrls.length) {
        state.needsRender = true
      }
    }
    img.onerror = () => {
      console.error('Failed to load image:', url)
      imagesLoaded++
    }
    img.src = url
    images.push(img)
  })
}

function drawImage(img: HTMLImageElement, x: number) {
  if (!ctx || !img.complete) return

  const W = CANVAS_WIDTH
  const H = CANVAS_HEIGHT

  // Calculate scaling to fit image within canvas while mintaining aspect ratio
  const imgAspect = img.width / img.height
  const canvasAspect = W / H

  let drawWidth = W
  let drawHeight = H
  let drawX = x
  let drawY = 0

  if (imgAspect > canvasAspect) {
    // Img is wider than canvas
    drawHeight = W / imgAspect
    drawY = (H - drawHeight) / 2
  } else {
    // Img is taller than canvas
    drawWidth = H * imgAspect
    drawX = x + (W - drawWidth) / 2
  }

  ctx.drawImage(img, drawX, drawY, drawWidth, drawHeight)
}

function render() {
  if (!ctx) return

  const W = CANVAS_WIDTH
  const H = CANVAS_HEIGHT

  // Clear canvas
  ctx.fillStyle = '#1a1a1a'
  ctx.fillRect(0, 0, W, H)

  if (imagesLoaded !== images.length) {
    // Loading...
    ctx.fillStyle = '#ffffff'
    ctx.font = '16px system-ui, -apple-system, sans-serif'
    ctx.textAlign = 'center'
    ctx.fillText(`Loading images... ${imagesLoaded}/${images.length}`, W / 2, H / 2)
    return
  }

  // Draw all images in a horizontal strip (offset by scrollX)
  images.forEach((img, i) => {
    const imageX = i * W - scrollX

    // Only draw images that are visible/partially visible
    if (imageX + W > 0 && imageX < W) {
      drawImage(img, imageX)
    }
  })
}

function loop() {
  if (state.needsRender) {
    state.needsRender = false
    render()
  }

  rafId = window.requestAnimationFrame(loop)
}

function handlePointerDown(e: PointerEvent) {
  const el = canvasRef.value
  if (!el) return

  isDragging = true
  dragStartX = e.clientX
  dragStartScrollX = scrollX
  el.setPointerCapture(e.pointerId)
  el.style.cursor = 'grabbing'
}

function handlePointerMove(e: PointerEvent) {
  if (!isDragging) return

  const deltaX = e.clientX - dragStartX

  // Update scroll position
  scrollX = dragStartScrollX - deltaX

  // Clamping to boundaries
  const maxScroll = (images.length - 1) * CANVAS_WIDTH
  scrollX = Math.max(0, Math.min(maxScroll, scrollX))

  state.needsRender = true
}

function handlePointerUp(e: PointerEvent) {
  if (!isDragging) return

  const el = canvasRef.value
  if (el) {
    el.releasePointerCapture(e.pointerId)
    el.style.cursor = 'grab'
  }

  isDragging = false
}

onMounted(() => {
  const el = canvasRef.value
  if (!el) return

  ctx = el.getContext('2d', { alpha: false })
  if (!ctx) return

  state.dpr = Math.max(1, window.devicePixelRatio || 1)

  // Set canvas size including dpr
  el.width = CANVAS_WIDTH * state.dpr
  el.height = CANVAS_HEIGHT * state.dpr
  ctx.setTransform(state.dpr, 0, 0, state.dpr, 0, 0)

  // Load images
  loadImages()

  // Start render loop
  rafId = window.requestAnimationFrame(loop)

  // Add pointer event listeners
  el.addEventListener('pointerdown', handlePointerDown)
  el.addEventListener('pointermove', handlePointerMove)
  el.addEventListener('pointerup', handlePointerUp)
  el.addEventListener('pointercancel', handlePointerUp)
})

onBeforeUnmount(() => {
  const el = canvasRef.value

  if (el) {
    el.removeEventListener('pointerdown', handlePointerDown)
    el.removeEventListener('pointermove', handlePointerMove)
    el.removeEventListener('pointerup', handlePointerUp)
    el.removeEventListener('pointercancel', handlePointerUp)
  }

  if (rafId != null) {
    window.cancelAnimationFrame(rafId)
    rafId = null
  }

  ctx = null
})
</script>

<template>
  <canvas ref="canvasRef" />
</template>

<style scoped>
canvas {
  display: block;
  margin: 0 auto;
  width: 600px;
  height: 380px;
  border-radius: 12px;
  cursor: grab;
  touch-action: none;
  user-select: none;
}

canvas:active {
  cursor: grabbing;
}
</style>