# UWIS: Underwater Image Stitching with Dynamic Enhancement

<div align="center">

[![OCEANS 2025](https://img.shields.io/badge/OCEANS-2025-blue.svg)](https://brest25.oceansconference.org/)
[![Presentation](https://img.shields.io/badge/Slides-PPT-orange.svg)](https://docs.google.com/presentation/d/1xooebMjuQOqWVQYA39l-LfWS1ji_ffQ9/edit?usp=sharing&ouid=108431554049751899932&rtpof=true&sd=true)
[![Dataset](https://img.shields.io/badge/Dataset-UWIS-green.svg)](https://github.com/leanoLEE58/UWIS/blob/main/UWIS%20dataset)

**[Jiayi Li](mailto:jiayilee@stu.ouc.edu.cn)¹ · [Kaizhi Dong](mailto:dongkaizhi@stu.ouc.edu.cn)² · [Guihui Li](mailto:guihuilee@stu.ouc.edu.cn)¹ · [Mai Yan](mailto:yanmai@ouc.edu.cn)¹ · [Haiyong Zheng](mailto:zhenghaiyong@ouc.edu.cn)¹**

¹College of Electronic Engineering, Ocean University of China  
²College of Computer Science and Technology, Ocean University of China

*Accepted at IEEE OCEANS 2025 (Oral Presentation)*

[📄 Paper](https://drive.google.com/file/d/1smfx3g0z5ALuykHzjahgh_nz-WG2WLUv/view?usp=sharing) | [🎤 Slides](https://docs.google.com/presentation/d/1xooebMjuQOqWVQYA39l-LfWS1ji_ffQ9/edit?usp=sharing&ouid=108431554049751899932&rtpof=true&sd=true) | [📦 Dataset](https://github.com/leanoLEE58/UWIS/blob/main/UWIS%20dataset)

</div>

---

## 📢 News

- **[2025.01]** 🎉 Paper accepted to **IEEE OCEANS 2025** as **Oral Presentation**
- **[2025.01]** 🌟 Code and UWIS dataset publicly released
- **[2025.01]** 📊 UWIS benchmark with **28,526 annotated image pairs** now available

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

### 📊 **3. Superior Performance**

<div align="center">
<img width="1002" alt="Qualitative Comparison" src="https://github.com/user-attachments/assets/7b8c43c4-8726-4730-a640-afdeb0508dd6" />

*Qualitative comparison on UWIS dataset. Our method achieves **more natural transitions** and **better structural preservation** compared to APAP and UDIS++.*
</div>

**Quantitative Results:**

| Method | PSNR↑ | SSIM↑ | MSE↓ |
|--------|:--------:|:--------:|:--------:|
| APAP+RANSAC | 23.42 | 0.682 | 912.57 |
| UDIS++ | 26.51 | 0.736 | 643.88 |
| **Ours** | **26.97** | **0.777** | **626.51** |

**Improvements over State-of-the-Art (UDIS++):**
- 📈 **+0.46 dB** PSNR (better reconstruction quality)
- 📈 **+5.57%** SSIM (better structure preservation)
- 📉 **-2.77%** MSE (lower error)

---

## 🔬 Technical Insights

### Feature Matching Comparison

<div align="center">
<img width="883" alt="Feature Matching Comparison" src="https://github.com/user-attachments/assets/847aa3c8-1596-4d82-95ad-3da69259fec2" />

*LoFTR (green, top) detects **2,071 high-confidence matches** (avg. confidence 0.776) with balanced distribution across the entire overlapping region. Traditional methods (yellow, bottom) struggle with color distortion and low contrast, showing significantly fewer reliable matches.*
</div>

**Why LoFTR Excels in Underwater Scenes:**
- 🎯 Attention mechanisms handle color distortion robustly
- 🌐 Dense matching across entire image, not just high-contrast regions
- 💪 Resilient to marine snow and non-uniform illumination

### Complete Processing Pipeline

<div align="center">
<img width="1027" alt="Stitching and Refinement Process" src="https://github.com/user-attachments/assets/9852f454-201c-44f6-8ab9-cb1e8a9ab6ec" />

*Complete workflow: (a,b) Input images → (c) Initial stitching → (d) Refined result. The difference map (e) shows refinement **focuses on seam regions** while **preserving structural integrity**.*
</div>

**Refinement Network Benefits:**
- 🎨 Smooths visible seams and color transitions
- 🏗️ Preserves structural details in non-boundary regions
- 🔄 Unsupervised learning - no ground truth required
- ⚡ Lightweight design (1.4M parameters, 15.3ms processing time)

---

## 🛠️ Installation

### Requirements
- Python 3.7+
- CUDA-capable GPU (RTX 3090 recommended)
- 64GB RAM (recommended for large-scale processing)

### Quick Start

```bash
# Clone the repository
git clone https://github.com/leanoLEE58/UWIS.git
cd UWIS

# Create conda environment
conda create -n uwis python=3.8
conda activate uwis

# Install dependencies
pip install -r requirements.txt
```

### Dependencies
```bash
pip install tensorflow==2.8.0
pip install torch==1.10.0 torchvision==0.11.0
pip install kornia==0.6.4
pip install opencv-python matplotlib numpy scikit-image tqdm
```

---

## 📁 Project Structure

```
UWIS/
├── config.py                      # Configuration file
├── main.py                        # Main entry point
├── train_finetune.py             # FUnIE-GAN fine-tuning
├── test_finetune.py              # Model testing and evaluation
├── components/
│   ├── feature_matching.py       # Feature matching module
│   ├── dynamic_decision.py       # Dynamic Decision Making (DDM)
│   ├── ransac_stitcher.py        # RANSAC-based stitcher
│   ├── funiegan_enhancer.py      # FUnIE-GAN enhancer
│   └── unsupervised_refinement.py # Refinement network
├── utils/
│   ├── metrics.py                # Evaluation metrics
│   └── visualization.py          # Visualization tools
├── data/
│   ├── train/                    # Training set (90%)
│   ├── test/                     # Test set (10%)
│   └── finetune/                 # Fine-tuning data
├── models/
│   ├── funiegan/                 # Pre-trained FUnIE-GAN
│   └── refinement/               # Refinement network weights
└── results/
    ├── stitched/                 # Stitching results
    └── visualizations/           # Diagnostic visualizations
```

---

## 🚀 Usage

### 1. Basic Image Stitching

```bash
# Process a single image pair
python main.py --mode process \
    --img1 data/test/coral_A.jpg \
    --img2 data/test/coral_B.jpg \
    --output results/coral_stitched.jpg

# Interactive mode (recommended for beginners)
python main.py

# Batch processing
python main.py --mode batch \
    --input_dir data/test/input \
    --output_dir results/batch_output
```

### 2. Fine-tuning FUnIE-GAN

```bash
# Prepare your data:
# - Place degraded images in data/finetune/input
# - Place target enhanced images in data/finetune/target

# Fine-tune the model
python train_finetune.py \
    --input_dir data/finetune/input \
    --target_dir data/finetune/target \
    --epochs 10 \
    --batch_size 4 \
    --learning_rate 0.00001

# Test the fine-tuned model
python test_finetune.py \
    --model_path models/finetuned/underwater_funie_gan \
    --test_dir data/test
```

### 3. Training Refinement Network

```bash
# Unsupervised refinement training
python main.py --mode train_refinement \
    --stitched_dir data/stitched \
    --epochs 20 \
    --batch_size 4
```

### 4. Configuration

Edit `config.py` to customize paths and parameters:

```python
# Model paths
FUNIEGAN_MODEL_PATH = "models/funiegan/model_15320_"
FUNIEGAN_FINETUNED_PATH = "models/finetuned/underwater_funie_gan"

# Fine-tuning configuration
FINETUNE_CONFIG = {
    "batch_size": 4,
    "epochs": 10,
    "learning_rate": 0.00001,
    "save_interval": 2
}

# Dynamic Decision Making parameters
DDM_LAMBDA = 0.7          # Balance between matchability and color quality
DDM_THRESHOLD = 0.1       # Decision threshold
```

---

## 📊 Evaluation

```bash
# Evaluate on test set
python evaluate.py \
    --test_dir data/test \
    --model_path models/finetuned \
    --output results/evaluation_report.html
```

**Evaluation Metrics:**
- 📈 **PSNR** (Peak Signal-to-Noise Ratio) - Higher is better
- 📈 **SSIM** (Structural Similarity Index) - Higher is better
- 📉 **MSE** (Mean Squared Error) - Lower is better
- 🌊 **UIQM** (Underwater Image Quality Measure) - Higher is better
- 📉 **CE** (Contrast Entropy) - Lower is better

---

## 💡 Key Features

### ✅ What Makes UWIS Different?

| Feature | Traditional Methods | UWIS (Ours) |
|---------|-------------------|-------------|
| **Enhancement Strategy** | Always enhance or never enhance | ✅ Adaptive decision based on scene |
| **Feature Matching** | Fixed algorithm (SIFT/SURF) | ✅ Dynamic selection (LoFTR/SIFT) |
| **Refinement** | Manual blending or simple averaging | ✅ Unsupervised learning-based |
| **Underwater Focus** | Generic methods adapted | ✅ Designed for underwater scenes |
| **Dataset** | No dedicated benchmark | ✅ 28K+ annotated pairs |

### 🎯 When to Use UWIS?

- 🐠 **Marine biology research**: Creating panoramic views of coral reefs and habitats
- 🤖 **Underwater robotics**: Real-time scene reconstruction for AUVs/ROVs
- 🗺️ **3D reconstruction**: Building detailed underwater maps
- 📹 **Underwater cinematography**: Enhancing wide-angle underwater footage
- 🔬 **Scientific monitoring**: Long-term habitat surveillance

---

## 📚 Documentation

### Data Preparation

**For Stitching:**
1. Place image pairs in `data/input/`
2. Name format: `prefix_A.jpg` and `prefix_B.jpg`
3. Ensure sufficient overlap (30-70% recommended)

**For Fine-tuning:**
1. Input (degraded): `data/finetune/input/`
2. Target (enhanced): `data/finetune/target/`
3. Images should be paired with matching filenames

**For Refinement Training:**
1. Initial stitched results: `data/stitched/`
2. No ground truth required (unsupervised)

### Computational Requirements

| Component | Parameters | Memory (MB) | Time (ms) |
|-----------|------------|-------------|-----------|
| FUnIE-GAN | 5.8M | 87.4 | 28.6 |
| LoFTR | 15.2M | 256.3 | 94.2 |
| Refinement | 1.4M | 32.8 | 15.3 |
| **Total Pipeline** | **22.4M** | **376.5** | **~138** |

*Tested on NVIDIA RTX 3090 with 256×256 input images*

---

## 🎓 Citation

If you find our work useful in your research, please consider citing:

```bibtex
@inproceedings{li2025uwis,
  title={UWIS: Underwater Image Stitching Dataset and a Dynamic Pipeline for Feature Enhancement and Stitching},
  author={Li, Jiayi and Dong, Kaizhi and Li, Guihui and Yan, Mai and Zheng, Haiyong},
  booktitle={OCEANS 2025 - IEEE/MTS},
  year={2025},
  organization={IEEE}
}
```

---

## 🎓 Resources

- 📄 **Full Paper**: [Google Drive](https://drive.google.com/file/d/1smfx3g0z5ALuykHzjahgh_nz-WG2WLUv/view?usp=sharing)
- 🎤 **Oral Presentation**: [Google Slides](https://docs.google.com/presentation/d/1xooebMjuQOqWVQYA39l-LfWS1ji_ffQ9/edit?usp=sharing&ouid=108431554049751899932&rtpof=true&sd=true)
- 📦 **UWIS Dataset**: [Download](https://github.com/leanoLEE58/UWIS/blob/main/UWIS%20dataset)
- 💬 **Discussion**: [GitHub Issues](https://github.com/leanoLEE58/UWIS/issues)

---

## 🏆 Acknowledgments

This work was supported by:
- **National Natural Science Foundation of China** (No. 62171421)
- **Taishan Scholars Youth Expert Program** of Shandong Province (No. tsqn202306096)
- **Fundamental Research Funds for the Central Universities** (No. 202261007)

We thank the authors of **EUVP**, **MSRB**, and **UVEB** datasets for making their data publicly available, which greatly facilitated our research.

---

## 📧 Contact

**Authors:**
- **Jiayi Li** - jiayilee@stu.ouc.edu.cn
- **Kaizhi Dong** - dongkaizhi@stu.ouc.edu.cn
- **Guihui Li** (Corresponding Author) - guihuilee@stu.ouc.edu.cn
- **Mai Yan** (Corresponding Author) - yanmai@ouc.edu.cn
- **Haiyong Zheng** - zhenghaiyong@ouc.edu.cn

**Institution:**  
Ocean University of China  
College of Electronic Engineering  
Qingdao, Shandong, China

---

## 📜 License（pending）

This project is released under the [MIT License](LICENSE).

```
Copyright (c) 2025 Ocean University of China

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```

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
