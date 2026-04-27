# UWIS: Underwater Image Stitching with Dynamic Enhancement

<div align="center">

[![OCEANS 2025](https://img.shields.io/badge/OCEANS-2025-blue.svg)](https://brest25.oceansconference.org/)
[![Presentation](https://img.shields.io/badge/Slides-PPT-orange.svg)](https://docs.google.com/presentation/d/1xooebMjuQOqWVQYA39l-LfWS1ji_ffQ9/edit?usp=sharing&ouid=108431554049751899932&rtpof=true&sd=true)
[![Dataset](https://img.shields.io/badge/Dataset-UWIS-green.svg)](https://github.com/leanoLEE58/UWIS/blob/main/UWIS%20dataset)

**[Jiayi Li](mailto:leanolee58@gmail)¹ · [Kaizhi Dong](mailto:dongkaizhi@stu.ouc.edu.cn)² · [Guihui Li](mailto:guihuilee@stu.ouc.edu.cn)¹ · [Mai Yan](mailto:yanmai@ouc.edu.cn)¹ · [Haiyong Zheng](mailto:zhenghaiyong@ouc.edu.cn)¹**

¹College of Electronic Engineering, Ocean University of China  
²College of Computer Science and Technology, Ocean University of China


[📄 Paper](https://drive.google.com/file/d/1smfx3g0z5ALuykHzjahgh_nz-WG2WLUv/view?usp=sharing) | [🎤 Slides](https://docs.google.com/presentation/d/1xooebMjuQOqWVQYA39l-LfWS1ji_ffQ9/edit?usp=sharing&ouid=108431554049751899932&rtpof=true&sd=true) | [📦 Dataset](https://github.com/leanoLEE58/UWIS/blob/main/UWIS%20dataset)

</div>

---


## 🌊 Overview

Underwater image stitching is essential for ocean exploration, marine habitat monitoring, and underwater robotics. However, the unique challenges of underwater environments—**severe light attenuation, color distortion, motion blur, and marine snow**—render traditional stitching methods inadequate.

We introduce **UWIS**, the **first large-scale benchmark dataset** specifically designed for underwater image stitching, along with a **dynamic feature enhancement and stitching framework** that intelligently adapts to diverse underwater conditions.

<div align="center">
<img width="974" alt="Underwater Degradation Types" src="https://github.com/user-attachments/assets/55f145d6-ed31-4988-957a-714f3770bd7f" />

*Six typical underwater degradation scenarios in UWIS: **large parallax**, **low contrast**, **motion blur**, **color deterioration**, **non-uniform illumination**, and **marine snow artifacts**.*
</div>

---

## ✨ Key Contributions

### 🎯 **1. UWIS Dataset - First Underwater Stitching Benchmark**

We present the **first large-scale dataset** tailored for underwater image stitching with **28,526 carefully annotated image pairs** covering six challenging underwater scenarios.

<div align="center">
<img width="1608" alt="Dataset Generation Pipeline" src="https://github.com/user-attachments/assets/4ec9324b-37fc-40df-ae8e-b2525f1bf884" />

*Our novel dataset generation pipeline combines **non-rigid distortion**, **viewpoint transformation**, and **homography alignment** to realistically simulate underwater parallax and geometric distortions.*
</div>

**Dataset Highlights:**
- 📦 **28,526 image pairs** with comprehensive annotations
- 🎨 **Six degradation scenarios**: parallax, blur, low contrast, color shift, uneven lighting, marine snow
- 🏷️ **Complete labels**: homography matrices, ground truth, enhancement references
- 🌍 **Real-world scenes** from EUVP, MSRB, and UVEB datasets
- 📊 **90/10 train-test split** for standardized evaluation

**📥 [Download UWIS Dataset](https://github.com/leanoLEE58/UWIS/blob/main/UWIS%20dataset)**

---

### 🚀 **2. Dynamic Enhancement & Stitching Framework**

Unlike conventional methods that blindly enhance all images, our framework **intelligently decides when enhancement helps and when it harms** stitching quality.

<div align="center">
<img width="1313" alt="Framework Architecture" src="https://github.com/user-attachments/assets/62c7c5d4-673a-4f28-b63c-7962e70049d1" />

*Our three-stage framework: **(a) Stage 1** - FUnIE-GAN enhancement, **(b) Stage 2** - Dynamic Decision Making with adaptive input selection and matcher choice, **(c) Stage 3** - Robust stitching with unsupervised refinement.*
</div>

**Framework Highlights:**

#### 🎨 **Stage 1: Adaptive Image Enhancement**
- Fine-tuned **FUnIE-GAN** for underwater color restoration
- Conservative training strategy preserving structural details
- Improves contrast and visibility while maintaining texture

#### 🧠 **Stage 2: Dynamic Decision Making (DDM)**

Our **key innovation** - DDM intelligently chooses the optimal processing strategy:

<div align="center">
<img width="464" alt="Dynamic Decision Process" src="https://github.com/user-attachments/assets/9b6a5e4b-c512-4c28-a275-784b345d5a1c" />

*Dynamic decision process evaluating both **matchability** and **color quality**. The system automatically selects enhanced or original images based on quantitative assessment.*
</div>

**Critical Insight:** 
- ✅ **64.1%** of cases benefit from enhancement
- ⚠️ **35.9%** perform better with original images
- 🎯 **DDM automatically makes the right choice** for each scenario

**Dual Dynamic Selection:**
1. **Image Selector**: Chooses between original/enhanced based on:
   - Feature matchability (number and confidence of matches)
   - Color quality (contrast and color balance)
2. **Matcher Selector**: Switches between:
   - **LoFTR** for texture-rich, low-contrast scenes
   - **SIFT** for high-contrast, edge-dominant scenes

#### 🔧 **Stage 3: Stitching & Refinement**
- **Robust RANSAC** homography estimation
- **Unsupervised refinement network** for seamless blending without ground truth
- Specialized loss functions: boundary smoothness, structure preservation, color consistency

---



---

## 🎓 Resources

- 📄 **Full Paper**: [Google Drive](https://drive.google.com/file/d/1smfx3g0z5ALuykHzjahgh_nz-WG2WLUv/view?usp=sharing)
- 🎤 **Oral Presentation**: [Google Slides](https://docs.google.com/presentation/d/1xooebMjuQOqWVQYA39l-LfWS1ji_ffQ9/edit?usp=sharing&ouid=108431554049751899932&rtpof=true&sd=true)
- 📦 **UWIS Dataset**: [Download](https://github.com/leanoLEE58/UWIS/blob/main/UWIS%20dataset)
- 💬 **Discussion**: [GitHub Issues](https://github.com/leanoLEE58/UWIS/issues)

---



## 📧 Contact

**Authors:**
- **Jiayi Li** - leanolee58@gmail.com
- **Kaizhi Dong** - dongkaizhi@stu.ouc.edu.cn
- **Guihui Li** (Corresponding Author) - guihuilee@stu.ouc.edu.cn


**Institution:**  
Ocean University of China  
College of Electronic Engineering  
Qingdao, Shandong, China



---

## 🤝 Contributing

We welcome contributions! Please feel free to:
- 🐛 Report bugs via [Issues](https://github.com/leanoLEE58/UWIS/issues)
- 💡 Suggest new features
- 🔧 Submit pull requests
- 📖 Improve documentation

---

## ⭐ Star History

If you find this project helpful, please consider giving it a star ⭐!


<div align="center">

### 🌊 Made with ❤️ for the Underwater Computer Vision Community

**Questions? Suggestions? Feel free to [open an issue](https://github.com/leanoLEE58/UWIS/issues) or reach out!**

[⬆ Back to Top](#uwis-underwater-image-stitching-with-dynamic-enhancement)

</div>
