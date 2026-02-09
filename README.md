# Canvas Image Slider

 Drag-based image slider built with Vue 3, TypeScript, and HTML5 Canvas.

## 🐶🐶🐶 Live Demo
**[View Live Demo →](https://shevkate.github.io/publitas-slider/)**

## Features
- Free-scrolling carousel with drag interaction
- Canvas-based rendering for smooth performance
- Responsive image scaling
- Works with images of different dimensions

## Tech Stack
- Vue 3 (Composition API)
- TypeScript
- HTML5 Canvas
- Vite

## Browser Support
Tested and working on:
- Chrome 120+
- Firefox 121+
- Safari 17+


## Quick Start

### Preview Production Build
```bash
npm run build
npm run preview
```
Open http://localhost:4173

**Note:** The dist/ folder is configured for GitHub Pages deployment.
Use `npm run preview` to test the production build locally.

## Development

### Prerequisites
- Node.js >= 20.19 (tested with Node 20.19.0)
- npm or yarn

### Installation
```bash
npm install
```

### Run Development Server
```bash
npm run dev
```
Open http://localhost:5173

### Build for Production
```bash
npm run build
```


## Project Structure
```
src/
├── App.vue                 # Main app component
├── components/
│   └── CanvasSlider.vue   # Canvas slider implementation
└── main.ts                # App entry point
```


## Implementation Details

### Canvas Rendering
- Fixed-size canvas (600×380) with devicePixelRatio scaling for crisp rendering on different displays
- Images rendered with aspect ratio preservation
- Efficient rendering only redraws when needed

### Interaction
- Pointer Events API for unified mouse/touch/pen input
- Free-scrolling model (no snapping)
- Boundary enforcement (can't scroll past first/last image)

### Performance
- requestAnimationFrame-based render loop running at 60fps
- Visibility (only draws visible images)
- No unnecessary re-renders when idle

## Development Process

### Requirements Analysis
- Analyzed reference implementation behavior
- Identified key features: free-scrolling, drag interaction, aspect ratio preservation

### Key Decisions
See [DECISIONS.md](./DECISIONS.md) for detailed technical decisions and trade-offs.