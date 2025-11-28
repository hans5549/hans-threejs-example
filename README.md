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

## Technology Stack

- [Three.js](https://threejs.org/) - 3D JavaScript library
- ES Modules via CDN importmap
- lil-gui for parameter controls
- Stats.js for performance monitoring

## Browser Support

Modern browsers with WebGL support:
- Chrome (latest 2 versions)
- Firefox (latest 2 versions)
- Safari (latest 2 versions)
- Edge (latest 2 versions)