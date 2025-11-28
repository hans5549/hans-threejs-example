# Three.js Examples

A collection of Three.js examples demonstrating various 3D rendering techniques and post-processing effects.

## Examples

### 1. Interactive Raycasting Points

Interactive 3D point cloud with raycasting and hover effects.

📁 [examples/interactive-raycasting-points/](examples/interactive-raycasting-points/)

### 2. Unreal Bloom Post-processing

Demonstrates the UnrealBloomPass post-processing effect with an animated 3D model.

📁 [examples/unreal-bloom-postprocessing/](examples/unreal-bloom-postprocessing/)

**Features:**
- Real-time Bloom effect using UnrealBloomPass
- Animated Primary Ion Drive 3D model
- Interactive GUI for parameter adjustment
- OrbitControls for camera manipulation
- Responsive window resizing

**Quick Start:**
```bash
cd examples/unreal-bloom-postprocessing
python -m http.server 8080
# Open http://localhost:8080 in your browser
```

### 3. CSS3D Periodic Table

Interactive 3D periodic table with CSS3D transforms and multiple layout modes.

📁 [examples/css3d-periodic-table/](examples/css3d-periodic-table/)

### 4. WebGPU Volume Caustics 🆕

Real-time volumetric caustics effect using WebGPU and Three.js TSL (Three Shading Language).

📁 [examples/webgpu-volume-caustics/](examples/webgpu-volume-caustics/)

**Features:**
- WebGPU-powered rendering with Three.js
- Transparent glass duck model with caustic light projection
- Chromatic aberration (rainbow caustics) effect
- Volumetric lighting with 3D noise
- Bloom post-processing
- Interactive OrbitControls
- Fallback UI for unsupported browsers

**Quick Start:**
```bash
python -m http.server 8080
# Open http://localhost:8080/examples/webgpu-volume-caustics/ in Chrome 113+ or Edge 113+
```

**Note:** This example requires a browser with WebGPU support (Chrome 113+, Edge 113+).

## Technology Stack

- [Three.js](https://threejs.org/) - 3D JavaScript library
- ES Modules via CDN importmap
- lil-gui for parameter controls
- Stats.js for performance monitoring
- WebGPU (for advanced examples)

## Browser Support

Modern browsers with WebGL support:
- Chrome (latest 2 versions)
- Firefox (latest 2 versions)
- Safari (latest 2 versions)
- Edge (latest 2 versions)

**WebGPU Support (for example #4):**
- Chrome 113+
- Edge 113+