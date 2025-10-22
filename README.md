# UWIS: Underwater Image Stitching with Dynamic Enhancement

<div align="center">

[![OCEANS 2025](https://img.shields.io/badge/OCEANS-2025-blue.svg)](your-paper-link)
[![Project Page](https://img.shields.io/badge/Project-Page-green.svg)](your-project-page)
[![Presentation](https://img.shields.io/badge/Slides-PPT-orange.svg)](your-ppt-link)
[![License](https://img.shields.io/badge/License-MIT-red.svg)](LICENSE)

**[Jiayi Li](link)¹ · [Kaizhi Dong](link)² · [Guihui Li](link)¹ · [Mai Yan](link)¹ · [Haiyong Zheng](link)¹**

¹Ocean University of China, ²Ocean University of China

*Accepted at IEEE OCEANS 2025 (Oral Presentation)*

[Paper](your-arxiv-link) | [Slides](your-ppt-link) | [Video](your-video-link) | [Dataset](dataset-link)

</div>

---

## 📢 News

- **[2025.XX]** 🎉 Paper accepted to **OCEANS 2025** as **Oral Presentation**
- **[2025.XX]** 🌟 Code and dataset publicly released
- **[2025.XX]** 📊 UWIS dataset with 28,526 image pairs now available

---

## 🌊 Overview

Underwater image stitching is crucial for ocean exploration, marine habitat monitoring, and underwater robotics. However, unique challenges in underwater environments—such as severe light attenuation, color distortion, motion blur, and marine snow—make traditional stitching methods inadequate.

We introduce **UWIS**, the **first large-scale benchmark dataset** specifically designed for underwater image stitching, along with a **dynamic feature enhancement and stitching framework** that intelligently adapts to diverse underwater conditions.

<div align="center">
<img width="974" alt="Underwater Degradation Types" src="https://github.com/user-attachments/assets/55f145d6-ed31-4988-957a-714f3770bd7f" />

*Figure 1: Six typical underwater image degradation scenarios in the UWIS dataset: large parallax, low contrast, motion blur, color deterioration, non-uniform illumination, and marine snow artifacts.*
</div>

---

## ✨ Key Contributions

### 🎯 **1. UWIS Dataset - First Underwater Stitching Benchmark**

We present the first large-scale dataset tailored for underwater image stitching with **28,526 carefully annotated image pairs** covering six challenging underwater scenarios.

<div align="center">
<img width="1608" alt="Dataset Generation Pipeline" src="https://github.com/user-attachments/assets/4ec9324b-37fc-40df-ae8e-b2525f1bf884" />

*Figure 2: Our novel dataset generation pipeline combines non-rigid distortion, viewpoint transformation, and homography alignment to realistically simulate underwater parallax and geometric distortions.*
</div>

**Dataset Features:**
- 📦 **28,526 image pairs** with comprehensive annotations
- 🎨 Six degradation scenarios: parallax, blur, low contrast, color shift, uneven lighting, marine snow
- 🏷️ Complete labels: homography matrices, ground truth, enhancement references
- 🌍 Real-world underwater scenes from EUVP, MSRB, and UVEB datasets

---

### 🚀 **2. Dynamic Enhancement & Stitching Framework**

Unlike conventional methods that blindly enhance all images, our framework **intelligently decides** when enhancement helps and when it harms stitching quality.

<div align="center">
<img width="1313" alt="Framework Architecture" src="https://github.com/user-attachments/assets/62c7c5d4-673a-4f28-b63c-7962e70049d1" />

*Figure 3: Our three-stage framework: (a) Stage 1 - FUnIE-GAN enhancement, (b) Stage 2 - Dynamic Decision Making with adaptive input selection and matcher choice, (c) Stage 3 - Robust stitching with unsupervised refinement.*
</div>

**Framework Highlights:**

#### 🎨 **Stage 1: Adaptive Image Enhancement**
- Fine-tuned FUnIE-GAN for underwater color restoration
- Conservative training strategy preserving structural details

#### 🧠 **Stage 2: Dynamic Decision Making (DDM)**
Our key innovation - **DDM intelligently chooses the optimal processing strategy**:

<div align="center">
<img width="464" alt="Dynamic Decision Process" src="https://github.com/user-attachments/assets/9b6a5e4b-c512-4c28-a275-784b345d5a1c" />

*Figure 4: Dynamic decision process evaluating both matchability and color quality. The system automatically selects enhanced or original images based on quantitative assessment.*
</div>

**Key Finding:** 
- ✅ **64.1%** of cases benefit from enhancement
- ⚠️ **35.9%** perform better with original images
- 🎯 DDM automatically makes the right choice for each scenario

**Dual Dynamic Selection:**
1. **Image Selector**: Chooses between original/enhanced based on feature matchability + color quality
2. **Matcher Selector**: Switches between LoFTR (texture-rich) and SIFT (high-contrast) algorithms

#### 🔧 **Stage 3: Stitching & Refinement**
- Robust RANSAC-based homography estimation
- Unsupervised refinement network for seamless blending

---

### 📊 **3. Superior Performance**

<div align="center">
<img width="1002" alt="Qualitative Comparison" src="https://github.com/user-attachments/assets/7b8c43c4-8726-4730-a640-afdeb0508dd6" />

*Figure 5: Qualitative comparison on UWIS dataset. Our method achieves more natural transitions and better preserves structural details compared to APAP and UDIS++.*
</div>

**Quantitative Results:**

| Method | PSNR↑ | SSIM↑ | MSE↓ |
|--------|--------|--------|--------|
| APAP+RANSAC | 23.42 | 0.682 | 912.57 |
| UDIS++ | 26.51 | 0.736 | 643.88 |
| **Ours** | **26.97** | **0.777** | **626.51** |

**Improvements over UDIS++:**
- 📈 **+0.46 dB** PSNR
- 📈 **+5.57%** SSIM  
- 📉 **-2.77%** MSE

---

## 🔬 Technical Insights

### Feature Matching Comparison

<div align="center">
<img width="883" alt="Feature Matching Comparison" src="https://github.com/user-attachments/assets/847aa3c8-1596-4d82-95ad-3da69259fec2" />

*Figure 6: LoFTR (top) detects 2,071 high-confidence matches (avg. 0.776) with balanced distribution, significantly outperforming traditional methods (bottom) in challenging underwater conditions.*
</div>

### Complete Processing Pipeline

<div align="center">
<img width="1027" alt="Stitching and Refinement Process" src="https://github.com/user-attachments/assets/9852f454-201c-44f6-8ab9-cb1e8a9ab6ec" />

*Figure 7: Complete workflow from input images (a,b) through initial stitching (c) to refined result (d). The difference map (e) shows refinement focuses on seam regions while preserving structural integrity.*
</div>

---

## 🛠️ Installation

### Requirements
- Python 3.7+
- CUDA-capable GPU (RTX 3090 recommended)
- 64GB RAM (recommended)

### Quick Start

```bash
# Clone the repository
git clone https://github.com/your-username/UWIS.git
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

## 📁 Dataset Structure

```
UWIS/
├── data/
│   ├── train/                    # Training set (90%)
│   │   ├── input/                # Original image pairs
│   │   ├── enhanced/             # Enhanced images
│   │   ├── homography/           # Homography matrices
│   │   └── ground_truth/         # Stitching ground truth
│   └── test/                     # Test set (10%)
│       └── ...
├── models/
│   ├── funiegan/                 # Pre-trained FUnIE-GAN
│   └── refinement/               # Refinement network weights
└── results/
    └── visualizations/           # Output visualizations
```

**📥 [Download UWIS Dataset](dataset-download-link)** (X GB)

---

## 🚀 Usage

### Basic Stitching

```bash
# Process a single image pair
python main.py --mode process \
    --img1 data/test/coral_A.jpg \
    --img2 data/test/coral_B.jpg \
    --output results/coral_stitched.jpg

# Batch processing
python main.py --mode batch \
    --input_dir data/test/input \
    --output_dir results/batch_output
```

### Fine-tuning FUnIE-GAN

```bash
# Train on your data
python train_finetune.py \
    --input_dir data/finetune/input \
    --target_dir data/finetune/target \
    --epochs 10 \
    --batch_size 4
```

### Training Refinement Network

```bash
# Unsupervised refinement training
python main.py --mode train_refinement \
    --stitched_dir data/stitched \
    --epochs 20
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

**Metrics:**
- 📈 PSNR (Peak Signal-to-Noise Ratio)
- 📈 SSIM (Structural Similarity Index)
- 📉 MSE (Mean Squared Error)
- 🌊 UIQM (Underwater Image Quality Measure)

---

## 🎬 Demo & Visualization

### Interactive Demo
```bash
# Launch interactive GUI
python demo.py
```

### Video Processing
```bash
# Process underwater video sequences
python process_video.py \
    --input underwater_scene.mp4 \
    --output panoramic_result.mp4
```

---

## 📖 Citation

If you find our work useful in your research, please consider citing:

```bibtex
@inproceedings{li2025uwis,
  title={UWIS: Underwater Image Stitching Dataset and a Dynamic Pipeline for Feature Enhancement and Stitching},
  author={Li, Jiayi and Dong, Kaizhi and Li, Guihui and Yan, Mai and Zheng, Haiyong},
  booktitle={OCEANS 2025},
  year={2025},
  organization={IEEE}
}
```

---

## 🎓 Resources

- 📄 **Paper**: [arXiv](your-arxiv-link) | [IEEE Xplore](ieee-link)
- 🎤 **Presentation**: [Slides](your-ppt-link) | [Video](your-video-link)
- 💾 **Dataset**: [Download](dataset-link) | [Documentation](dataset-doc)
- 💬 **Discussion**: [GitHub Issues](issues-link)

---

## 🏆 Acknowledgments

This work was supported by:
- National Natural Science Foundation of China (No. 62171421)
- Taishan Scholars Youth Expert Program of Shandong Province (No. tsqn202306096)
- Fundamental Research Funds for the Central Universities (No. 202261007)

We thank the authors of EUVP, MSRB, and UVEB datasets for making their data publicly available.

---

## 📧 Contact

- **Jiayi Li**: jiayilee@stu.ouc.edu.cn
- **Kaizhi Dong**: dongkaizhi@stu.ouc.edu.cn
- **Guihui Li** (Corresponding): guihuilee@stu.ouc.edu.cn

**Ocean University of China**  
College of Electronic Engineering & Computer Science and Technology

---

## 📜 License

This project is released under the [MIT License](LICENSE).

---

## 🌟 Star History

[![Star History Chart](https://api.star-history.com/svg?repos=your-username/UWIS&type=Date)](https://star-history.com/#your-username/UWIS&Date)

---

<div align="center">

**Made with ❤️ for the underwater computer vision community**

*If you have any questions or suggestions, feel free to open an issue or reach out!*

</div>
