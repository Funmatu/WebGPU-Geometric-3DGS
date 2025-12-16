# WebGPU Geometric 3D Gaussian Splatting

![License](https://img.shields.io/badge/license-MIT-blue)
![WebGPU](https://img.shields.io/badge/Render-WebGPU-green)
![Status](https://img.shields.io/badge/Status-Experimental-orange)

**Real-time Surface Reconstruction from SLAM Point Clouds directly in the Browser.**

[**🚀 Live Demo**](https://funmatu.github.io/WebGPU-Geometric-3DGS/)

---

## 📖 Overview

This project is a high-performance **WebGPU implementation of 3D Gaussian Splatting (3DGS)** designed to work with sparse point cloud data (e.g., from **ZED2i**, **RTAB-map**, or standard `.ply` files).

Unlike standard 3DGS training which requires hundreds of images and minutes of offline training (CUDA), this renderer implements a **"Geometric Training"** phase that runs entirely on the GPU in real-time. It analyzes the spatial distribution of neighbors for each point to estimate scale, rotation, and opacity, turning a sparse point cloud into a continuous, splatted surface instantly.

## ✨ Key Features

* **WebGPU Compute Pipelines**: All logic (Sorting, Training, Rasterization) runs on the GPU.
* **Geometric Training (Blind Estimation)**:
    * Estimates 3DGS parameters (Covariance: Scale & Rotation) *without* reference images.
    * Uses stochastic neighbor search and iterative plane fitting to align splats with surface geometry.
* **Optimized Bitonic Sort**:
    * Implements a fully parallel GPU Bitonic Sort.
    * Utilizes **Dynamic Offsets** to batch sort commands, reducing CPU-GPU communication overhead by ~99% compared to naive implementations.
* **Universal PLY Loader**:
    * Auto-detects Binary/ASCII formats.
    * Auto-fills missing 3DGS attributes (Scale/Rot/Opacity) for standard Point Clouds.
    * Auto-downsamples massive files to fit your GPU limits.

## 🛠️ Technology Stack

* **Language**: JavaScript (ES Modules)
* **API**: WebGPU (WGSL)
* **Math**: Custom Matrix/Quaternion logic (No heavy external math libraries)

## 🚀 How to Use

1.  Open the [Live Demo](https://funmatu.github.io/WebGPU-Geometric-3DGS/).
2.  Click **"Choose File"** and load a `.ply` file (e.g., a point cloud from a SLAM session).
3.  Initially, points are rendered as small spheres (Phase 1).
4.  Click **"Start Geometric Training"**.
5.  Watch as the points adaptively stretch and rotate to form a solid surface (Phase 2).

---

## 🔍 Technical Details: Comparison with Procedural Sim

This project is a significant evolution from the previous [Procedural Gaussian Splatting Simulation](https://github.com/Funmatu/threejs-procedural-gaussian-splats).

| Feature | Previous Project (Procedural Sim) | **This Project (Geometric Architect)** |
| :--- | :--- | :--- |
| **Core Tech** | **WebGL 2.0** (GLSL) | **WebGPU** (WGSL) |
| **Rendering** | Standard Rasterization | **EWA Splatting** (Gaussian Rasterization) |
| **Sorting** | CPU / None (Additive Blending) | **GPU Parallel Bitonic Sort** (Compute Shader) |
| **Data Source** | Procedural Math (Sine waves) | **Real Data** (.ply / SLAM Point Clouds) |
| **Goal** | Visual Mimicry (VFX) | **Surface Reconstruction** (Scientific/Spatial) |
| **Bottleneck** | Fragment Shader Fill-rate | Compute Shader Memory Bandwidth |

### Why WebGPU?
To achieve "Real" 3DGS, correctly handling alpha blending requires sorting millions of particles every frame (Back-to-Front).
* **WebGL**: Requires sorting on CPU, which is too slow for >1M points.
* **WebGPU**: Allows us to implement **Bitonic Sort** directly in Compute Shaders, keeping all data on the VRAM.

### The "Geometric Training" Algorithm
Standard 3DGS optimizes parameters minimizing photometric error against photos. Since we only have points (XYZ), we minimize **Geometric Entropy**:
1.  **Search**: Find random neighbors within a radius.
2.  **Analyze**: Calculate vector to neighbor.
3.  **Adapt**:
    * If neighbor aligns with the tangent plane: **Stretch** scale (fill holes).
    * If neighbor is normal to surface: **Flatten** scale (form disk).
    * Update **Rotation** (Quaternion) to align the splat's normal with the estimated surface normal.

## 📦 Installation

No build tools required. Modern vanilla JS.

```bash
git clone [https://github.com/your-username/WebGPU-Geometric-3DGS.git](https://github.com/your-username/WebGPU-Geometric-3DGS.git)
cd WebGPU-Geometric-3DGS
# Start a local server (e.g., using Python)
python3 -m http.server
````

Open `http://localhost:8000` in a WebGPU-enabled browser (Chrome 113+, Edge).

## 📜 License

MIT License
