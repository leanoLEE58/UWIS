# Underwater Image Stitching and Enhancement System (UWIS)

## Project Introduction

The Underwater Image Stitching and Enhancement System (UWIS) is an integrated solution specifically designed to address the challenges of image quality enhancement and stitching in underwater environments. The system integrates multiple modules including image enhancement, intelligent feature matching, dynamic decision-making, and unsupervised stitching optimization, capable of automatically processing the unique challenges of underwater scenes such as low contrast, color distortion, and limited visibility.

[img_3](https://github.com/user-attachments/assets/dbc62395-c783-432e-b294-ff84b0c3e8c9)

## Main Features

- **Intelligent Image Enhancement**: Underwater image enhancement based on FUnIE-GAN, supporting ultra-conservative fine-tuning
- **Dynamic Input Decision**: Automatic evaluation and selection of the best input images (original or enhanced)
- **Advanced Feature Matching**: Dynamic matcher selection combining LoFTR and SIFT
- **Precise Stitching**: Robust homography estimation based on RANSAC
- **Unsupervised Stitching Optimization**: Boundary optimization network designed specifically for underwater images
- **Evaluation System**: Integrated PSNR, SSIM
- **Visualization Diagnostics**: Detailed visualization display of each processing stage

## Installation and Dependencies

### System Requirements

- Python 3.7+
- GPU with CUDA support (recommended)

### Installation

1. Create a virtual environment (optional)
```bash
conda create -n underwater python=3.8
conda activate underwater
Install dependencies

pip install tensorflow==2.8.0
pip install torch==1.10.0 torchvision==0.11.0
pip install kornia==0.6.4
pip install opencv-python matplotlib numpy scikit-image tqdm
Directory Structure

underwater_stitching_system/
├── config.py                   # Configuration file
├── main.py                     # Main entry program
├── train_finetune.py           # FUnIE-GAN fine-tuning training program
├── test_finetune.py            # Fine-tuned model testing and evaluation program
├── components/                 # Component modules
│   ├── __init__.py
│   ├── feature_matching.py     # Feature matching module
│   ├── dynamic_decision.py     # Dynamic decision module
│   ├── ransac_stitcher.py      # RANSAC stitcher
│   ├── funiegan_enhancer.py    # FUnIE-GAN enhancer
│   ├── funiegan_finetuner.py   # FUnIE-GAN fine-tuner
│   └── unsupervised_refinement.py # Unsupervised optimization network
├── utils/                      # Utility functions
│   ├── __init__.py
│   ├── metrics.py              # Evaluation metrics calculation
│   └── visualization.py        # Visualization tools
└── data/                       # Data directory
    ├── input/                  # Input image pairs
    ├── enhanced/               # Enhanced images
    ├── stitched/               # Stitching results
    └── ground_truth/           # Optional reference ground truth
Usage
System Configuration
Before using the system, modify the config.py configuration file:

Path Configuration

ROOT_DIR = os.path.dirname(os.path.abspath(__file__))
DATA_DIR = os.path.join(ROOT_DIR, "data")
RESULTS_DIR = os.path.join(ROOT_DIR, "results")
MODELS_DIR = os.path.join(ROOT_DIR, "models")
FUnIE-GAN Model Paths

FUNIEGAN_MODEL_PATH = os.path.join(MODELS_DIR, "funiegan/model_15320_")
FUNIEGAN_FINETUNED_PATH = os.path.join(MODELS_DIR, "finetuned/underwater_funie_gan")
Fine-tuning Parameters

FINETUNE_CONFIG = {
    "input_dir": os.path.join(DATA_DIR, "finetune/input"),
    "target_dir": os.path.join(DATA_DIR, "finetune/target"),
    "batch_size": 4,
    "epochs": 10,
    "learning_rate": 0.00001,
    "save_interval": 2
}
Basic Operation
Interactive Mode (Default)

python main.py
Fine-tune FUnIE-GAN Enhancer

python main.py --mode finetune
Or use the dedicated training script:


python train_finetune.py
Test Fine-tuned Model

python test_finetune.py --model_path models/finetuned/underwater_funie_gan --test_dir data/test
Train Stitching Optimization Network

python main.py --mode train_refinement
Process a Single Pair of Images

python main.py --mode process --img1 path/to/image1.jpg --img2 path/to/image2.jpg
Batch Process Image Pairs

python main.py --mode batch --input_dir path/to/image_pairs
Data Preparation
Fine-tuning Dataset: Place original underwater images in data/finetune/input, corresponding target enhanced images in data/finetune/target
Stitching Test: Place image pairs in data/input following the format prefix_A.jpg and prefix_B.jpg
Reference Ground Truth: Optional reference ground truth images can be placed in the data/ground_truth directory
Key Technology Explanation
Dynamic Decision Module (DDM)
DDM automatically selects the most suitable input for stitching by analyzing the matchability and color quality of original and enhanced images:

Matchability Score: Evaluates the ability to establish feature matches between two images
Color Quality Score: Analyzes image contrast and color balance
Conservative Decision Mechanism: Selects enhanced images only when they significantly outperform the original
Dynamic Matcher Selection
The system intelligently selects the most appropriate feature matching algorithm after analyzing image features:

For texture-rich scenes, it tends to use LoFTR
For high-contrast scenes with obvious edges, it tends to use SIFT
Automatic failure switching mechanism ensures high matching success rate
Unsupervised Stitching Optimization
An unsupervised learning method designed specifically for underwater scenes, requiring no reference ground truth:

Mask-based Region Identification: Automatically detects stitching boundary regions
Multi-objective Loss Function: Boundary smoothness, structure preservation, color consistency, brightness balancing
Residual Learning: Only learns minor corrections, preserving the original structure
Fine-tuning Technique
FUnIE-GAN fine-tuning adopts an ultra-conservative strategy to ensure the original enhancement capability is not compromised:

Only trains the last 2% of network layers
Uses extremely low learning rate (0.00001)
Applies reference model constraints, maintaining 90% conservative loss weight
Batch processing and data augmentation to improve robustness
Evaluation Metrics
PSNR↑: Peak Signal-to-Noise Ratio, evaluates reconstruction quality
SSIM↑: Structural Similarity, evaluates structure preservation
CE↓: Contrast Entropy, evaluates contrast distribution
UIQM↑: Underwater Image Quality Metric, designed specifically for underwater scenes
Fine-tuning Training and Testing Functions
In addition to the main program, we provide two dedicated scripts for fine-tuning and testing FUnIE-GAN:

train_finetune.py
test_finetune.py
Enhancement-Stitching-Fine-tuning Process Example
Stitching Process Example

Stitching Effect Comparison Example
Stitching Effect Comparison

Detailed Analysis Report: evaluation_report.html

Performance and Limitations
Computational Requirements: LoFTR feature matching requires higher GPU memory; downsampling may be needed for high-resolution images
Tonal Consistency: Stitching optimization focuses on boundary regions, with limited improvement in overall tonal consistency
Real-time Performance: The current implementation emphasizes accuracy over speed and is not suitable for real-time systems
Maintainers
[Jiayi Li,  Guihui Li]
[{jiayilee, guihuilee}@stu.ouc.edu.cn]
```

**水下图像拼接增强系统 (UWIS)**
 
项目介绍
  
水下图像拼接增强系统(UWIS)是一个集成化的解决方案，专门解决水下环境图像的质量增强和拼接问题。系统集成了图像增强、智能特征匹配、动态决策和无监督拼接优化等多个模块，能够自动处理水下场景特有的挑战，如低对比度、颜色失真和可见度有限等问题。
[img_3](https://github.com/user-attachments/assets/dbc62395-c783-432e-b294-ff84b0c3e8c9)

系统架构

主要特性

* 智能图像增强：基于FUnIE-GAN的水下图像增强，支持超保守微调
* 动态输入决策：自动评估和选择最佳输入图像（原始或增强）
* 高级特征匹配：结合LoFTR和SIFT的动态匹配器选择
* 精确拼接：基于RANSAC的鲁棒单应性估计
* 无监督拼接优化：专为水下图像设计的边界优化网络
* 评估体系：集成PSNR、SSIM
* 可视化诊断：各处理阶段的详细可视化展示

**安装与依赖**
系统要求
* Python 3.7+
* CUDA支持的GPU (推荐)

**依赖安装**

# 创建虚拟环境(可选)
* conda create -n underwater python=3.8
* conda activate underwater

# 安装依赖
1. pip install tensorflow==2.8.0
2. pip install torch==1.10.0 torchvision==0.11.0
3. pip install kornia==0.6.4
4. pip install opencv-python matplotlib numpy scikit-image tqdm
目录结构

* underwater_stitching_system/
* ├── config.py                     # 配置文件
* ├── main.py                       # 主入口程序
* ├── train_finetune.py             # FUnIE-GAN微调训练程序
* ├── test_finetune.py              # 微调模型测试评估程序
* ├── components/                   # 组件模块
* │   ├── __init__.py
* │   ├── feature_matching.py       # 特征匹配模块
* │   ├── dynamic_decision.py       # 动态决策模块
* │   ├── ransac_stitcher.py        # RANSAC拼接器
* │   ├── funiegan_enhancer.py      # FUnIE-GAN增强器
* │   ├── funiegan_finetuner.py     # FUnIE-GAN微调器
* │   └── unsupervised_refinement.py# 无监督优化网络
* ├── utils/                        # 工具函数
* │   ├── __init__.py
* │   ├── metrics.py                # 评估指标计算
* │   └── visualization.py          # 可视化工具
* └── data/                         # 数据目录
*     ├── input/                    # 输入图像对
* 
*     ├── enhanced/                 # 增强图像
*     ├── stitched/                 # 拼接结果
*     └── ground_truth/             # 可选的参考真值
使用方法
配置系统
在使用系统前，先修改config.py配置文件：


# 路径配置
* ROOT_DIR = os.path.dirname(os.path.abspath(__file__))
* DATA_DIR = os.path.join(ROOT_DIR, "data")
* RESULTS_DIR = os.path.join(ROOT_DIR, "results")
* MODELS_DIR = os.path.join(ROOT_DIR, "models")

# FUnIE-GAN模型路径
FUNIEGAN_MODEL_PATH = os.path.join(MODELS_DIR, "funiegan/model_15320_")
FUNIEGAN_FINETUNED_PATH = os.path.join(MODELS_DIR, "finetuned/underwater_funie_gan")

# 微调参数
* FINETUNE_CONFIG = {
*     "input_dir": os.path.join(DATA_DIR, "finetune/input"),
*     "target_dir": os.path.join(DATA_DIR, "finetune/target"),
*     "batch_size": 4,
*     "epochs": 10,
*     "learning_rate": 0.00001,
*     "save_interval": 2
* }

基本运行
交互式模式（默认）

python main.py
微调FUnIE-GAN增强器

python main.py --mode finetune
# 或使用专用训练脚本
python train_finetune.py
测试微调后的模型

python test_finetune.py --model_path models/finetuned/underwater_funie_gan --test_dir data/test
训练拼接优化网络

python main.py --mode train_refinement
处理单对图像

python main.py --mode process --img1 path/to/image1.jpg --img2 path/to/image2.jpg
批量处理图像对

python main.py --mode batch --input_dir path/to/image_pairs
数据准备
微调数据集：将原始水下图像放在data/finetune/input，对应的目标增强图像放在data/finetune/target
拼接测试：将成对图像按prefix_A.jpg和prefix_B.jpg格式放在data/input
参考真值：可选的真值参考图像可放在data/ground_truth目录
关键技术解释
动态决策模块(DDM)
DDM通过分析原始和增强图像的匹配性和颜色质量，自动选择最适合拼接的输入：

匹配性评分：评估两幅图像间建立特征匹配的能力
颜色质量评分：分析图像的对比度和色彩平衡
保守决策机制：仅当增强版本显著优于原始版本时才选择增强图像
动态匹配器选择
系统分析图像特征后，智能选择最适合的特征匹配算法：

对于纹理丰富的场景，倾向于使用LoFTR
对于高对比度和明显边缘的场景，倾向于使用SIFT
自动故障切换机制确保匹配成功率
无监督拼接优化
专为水下场景设计的无监督学习方法，不需要参考真值：

基于蒙版的区域识别：自动检测拼接边界区域
多目标损失函数：边界平滑度、结构保持、颜色一致性、亮度均衡
残差学习：仅学习微小修正，保留原始结构
微调技术
FUnIE-GAN微调采用超保守策略，确保不破坏原有增强能力：

只训练最后2%的网络层
使用极低学习率(0.00001)
应用参考模型约束，保持90%的保守性损失权重
批量处理和数据增强以提高鲁棒性
评估指标
PSNR↑：峰值信噪比，评估重建质量
SSIM↑：结构相似性，评估结构保留程度
CE↓：对比度熵，评估对比度分布
UIQM↑：水下图像质量指标，专为水下场景设计
微调训练和测试函数
在主程序之外，我们提供了两个专用脚本用于FUnIE-GAN的微调和测试：
1.train_finetune.py
2.test_finetune.py

增强-拼接-微调流程示例
![img_2](https://github.com/user-attachments/assets/7dd5ca38-9c74-4587-b1ef-01750dfdfd32)

拼接效果对比示例
![img_1](https://github.com/user-attachments/assets/630e02bf-eef4-4416-8267-93edef7ff2eb)

详细分析报告：
[Uploading evaluation_report.html…]()



性能与局限性
* 计算需求：LoFTR特征匹配需要较高的GPU内存，对于高分辨率图像可能需要降采样
* 色调一致性：拼接优化专注于边界区域，对整体色调一致性的改善有限
* 实时性能：当前实现重点在准确性而非速度，不适用于实时系统

维护者
* [Jiayi Li1, Kaizhi Dong, Guihui Li]
* [{jiayilee, dongkaizhi, guihuilee}@stu.ouc.edu.cn

