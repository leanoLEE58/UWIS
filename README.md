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

[Uploadi
        <!DOCTYPE html>
        <html>
        <head>
            <title>水下图像拼接评估报告</title>
            <style>
                body { font-family: Arial, sans-serif; margin: 20px; }
                h1, h2, h3 { color: #2c3e50; }
                table { border-collapse: collapse; width: 100%; margin-bottom: 20px; }
                th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
                th { background-color: #f2f2f2; }
                tr:nth-child(even) { background-color: #f9f9f9; }
                .summary { margin: 20px 0; }
                .summary img { max-width: 500px; display: inline-block; margin: 10px; }
                .gallery { display: flex; flex-wrap: wrap; gap: 20px; }
                .gallery-item { width: 45%; margin-bottom: 20px; border: 1px solid #ddd; padding: 10px; }
                .gallery-item img { width: 100%; height: auto; }
                .good { color: green; }
                .average { color: orange; }
                .poor { color: red; }
            </style>
        </head>
        <body>
            <h1>水下图像拼接评估报告</h1>
            <p>评估时间: 2025-04-06 21:45:20</p>
            <p>样本数量: 515 个图像对</p>

            <h2>评估概要</h2>
            <table>
                <tr>
                    <th>指标</th>
                    <th>平均值</th>
                    <th>标准差</th>
                    <th>最小值</th>
                    <th>最大值</th>
                    <th>中位数</th>
                </tr>
                <tr>
                    <td>SSIM</td>
                    <td>0.7766</td>
                    <td>0.2482</td>
                    <td>0.0708</td>
                    <td>0.9982</td>
                    <td>0.9053</td>
                </tr>
                <tr>
                    <td>PSNR</td>
                    <td>26.9675</td>
                    <td>8.5205</td>
                    <td>9.2323</td>
                    <td>47.9018</td>
                    <td>28.1014</td>
                </tr>
                <tr>
                    <td>MSE</td>
                    <td>626.5101</td>
                    <td>1098.5598</td>
                    <td>1.0541</td>
                    <td>7759.7989</td>
                    <td>100.6802</td>
                </tr>
            </table>

            <h2>可视化结果</h2>
            <div class="summary">
                <img src="plots/metrics_boxplot.png" alt="指标箱线图">
                <img src="plots/ssim_histogram.png" alt="SSIM直方图">
                <img src="plots/psnr_histogram.png" alt="PSNR直方图">
                <img src="plots/ssim_vs_psnr.png" alt="SSIM vs PSNR">
            </div>

            <h2>各图像评估结果</h2>
            <table>
                <tr>
                    <th>图像ID</th>
                    <th>拼接文件</th>
                    <th>真值文件</th>
                    <th>SSIM</th>
                    <th>PSNR</th>
                    <th>MSE</th>
                    <th>RMSE</th>
                </tr>
        
                <tr>
                    <td>000009</td>
                    <td>pair_000009/stitched.png</td>
                    <td>000009.jpg</td>
                    <td class="poor">0.6956</td>
                    <td class="poor">19.6584</td>
                    <td>703.4592</td>
                    <td>26.5228</td>
                </tr>
            
                <tr>
                    <td>000010</td>
                    <td>pair_000010/stitched.png</td>
                    <td>000010.jpg</td>
                    <td class="poor">0.3955</td>
                    <td class="average">20.1916</td>
                    <td>622.1836</td>
                    <td>24.9436</td>
                </tr>
            
                <tr>
                    <td>000022</td>
                    <td>pair_000022/stitched.png</td>
                    <td>000022.jpg</td>
                    <td class="poor">0.5213</td>
                    <td class="poor">16.5932</td>
                    <td>1424.8120</td>
                    <td>37.7467</td>
                </tr>
            
                <tr>
                    <td>000028</td>
                    <td>pair_000028/stitched.png</td>
                    <td>000028.jpg</td>
                    <td class="poor">0.3653</td>
                    <td class="poor">18.1097</td>
                    <td>1004.8600</td>
                    <td>31.6995</td>
                </tr>
            
                <tr>
                    <td>000032</td>
                    <td>pair_000032/stitched.png</td>
                    <td>000032.jpg</td>
                    <td class="poor">0.5647</td>
                    <td class="poor">18.8400</td>
                    <td>849.3280</td>
                    <td>29.1432</td>
                </tr>
            
                <tr>
                    <td>000035</td>
                    <td>pair_000035/stitched.png</td>
                    <td>000035.jpg</td>
                    <td class="poor">0.5730</td>
                    <td class="poor">16.1123</td>
                    <td>1591.6433</td>
                    <td>39.8954</td>
                </tr>
            
                <tr>
                    <td>000036</td>
                    <td>pair_000036/stitched.png</td>
                    <td>000036.jpg</td>
                    <td class="poor">0.6043</td>
                    <td class="poor">17.3922</td>
                    <td>1185.3868</td>
                    <td>34.4294</td>
                </tr>
            
                <tr>
                    <td>000041</td>
                    <td>pair_000041/stitched.png</td>
                    <td>000041.jpg</td>
                    <td class="poor">0.5000</td>
                    <td class="poor">17.4546</td>
                    <td>1168.4777</td>
                    <td>34.1830</td>
                </tr>
            
                <tr>
                    <td>000042</td>
                    <td>pair_000042/stitched.png</td>
                    <td>000042.jpg</td>
                    <td class="poor">0.2841</td>
                    <td class="poor">14.5605</td>
                    <td>2275.2750</td>
                    <td>47.6998</td>
                </tr>
            
                <tr>
                    <td>000053</td>
                    <td>pair_000053/stitched.png</td>
                    <td>000053.jpg</td>
                    <td class="poor">0.4461</td>
                    <td class="poor">13.8435</td>
                    <td>2683.6792</td>
                    <td>51.8042</td>
                </tr>
            
                <tr>
                    <td>000072</td>
                    <td>pair_000072/stitched.png</td>
                    <td>000072.jpg</td>
                    <td class="poor">0.3327</td>
                    <td class="poor">16.6438</td>
                    <td>1408.3323</td>
                    <td>37.5278</td>
                </tr>
            
                <tr>
                    <td>000109</td>
                    <td>pair_000109/stitched.png</td>
                    <td>000109.jpg</td>
                    <td class="poor">0.4137</td>
                    <td class="poor">18.3284</td>
                    <td>955.5317</td>
                    <td>30.9117</td>
                </tr>
            
                <tr>
                    <td>000112</td>
                    <td>pair_000112/stitched.png</td>
                    <td>000112.jpg</td>
                    <td class="average">0.8571</td>
                    <td class="average">24.0125</td>
                    <td>258.1266</td>
                    <td>16.0663</td>
                </tr>
            
                <tr>
                    <td>000113</td>
                    <td>pair_000113/stitched.png</td>
                    <td>000113.jpg</td>
                    <td class="average">0.7786</td>
                    <td class="average">21.9899</td>
                    <td>411.2309</td>
                    <td>20.2788</td>
                </tr>
            
                <tr>
                    <td>000125</td>
                    <td>pair_000125/stitched.png</td>
                    <td>000125.jpg</td>
                    <td class="poor">0.3852</td>
                    <td class="poor">16.6170</td>
                    <td>1417.0370</td>
                    <td>37.6436</td>
                </tr>
            
                <tr>
                    <td>000136</td>
                    <td>pair_000136/stitched.png</td>
                    <td>000136.jpg</td>
                    <td class="poor">0.5526</td>
                    <td class="poor">19.5149</td>
                    <td>727.0901</td>
                    <td>26.9646</td>
                </tr>
            
                <tr>
                    <td>000158</td>
                    <td>pair_000158/stitched.png</td>
                    <td>000158.jpg</td>
                    <td class="poor">0.5545</td>
                    <td class="poor">19.5429</td>
                    <td>722.4163</td>
                    <td>26.8778</td>
                </tr>
            
                <tr>
                    <td>000159</td>
                    <td>pair_000159/stitched.png</td>
                    <td>000159.jpg</td>
                    <td class="poor">0.5327</td>
                    <td class="average">20.2634</td>
                    <td>611.9776</td>
                    <td>24.7382</td>
                </tr>
            
                <tr>
                    <td>000161</td>
                    <td>pair_000161/stitched.png</td>
                    <td>000161.jpg</td>
                    <td class="poor">0.5829</td>
                    <td class="poor">18.3405</td>
                    <td>952.8670</td>
                    <td>30.8685</td>
                </tr>
            
                <tr>
                    <td>000177</td>
                    <td>pair_000177/stitched.png</td>
                    <td>000177.jpg</td>
                    <td class="poor">0.5425</td>
                    <td class="poor">18.9004</td>
                    <td>837.5983</td>
                    <td>28.9413</td>
                </tr>
            
                <tr>
                    <td>000179</td>
                    <td>pair_000179/stitched.png</td>
                    <td>000179.jpg</td>
                    <td class="poor">0.3154</td>
                    <td class="poor">14.5376</td>
                    <td>2287.2834</td>
                    <td>47.8256</td>
                </tr>
            
                <tr>
                    <td>000184</td>
                    <td>pair_000184/stitched.png</td>
                    <td>000184.jpg</td>
                    <td class="poor">0.3425</td>
                    <td class="poor">18.5004</td>
                    <td>918.4077</td>
                    <td>30.3052</td>
                </tr>
            
                <tr>
                    <td>000190</td>
                    <td>pair_000190/stitched.png</td>
                    <td>000190.jpg</td>
                    <td class="poor">0.2604</td>
                    <td class="poor">14.6638</td>
                    <td>2221.7534</td>
                    <td>47.1355</td>
                </tr>
            
                <tr>
                    <td>000191</td>
                    <td>pair_000191/stitched.png</td>
                    <td>000191.jpg</td>
                    <td class="poor">0.5247</td>
                    <td class="poor">17.2347</td>
                    <td>1229.1631</td>
                    <td>35.0594</td>
                </tr>
            
                <tr>
                    <td>000192</td>
                    <td>pair_000192/stitched.png</td>
                    <td>000192.jpg</td>
                    <td class="poor">0.3562</td>
                    <td class="poor">16.3664</td>
                    <td>1501.1990</td>
                    <td>38.7453</td>
                </tr>
            
                <tr>
                    <td>000193</td>
                    <td>pair_000193/stitched.png</td>
                    <td>000193.jpg</td>
                    <td class="poor">0.6867</td>
                    <td class="poor">18.0910</td>
                    <td>1009.2020</td>
                    <td>31.7679</td>
                </tr>
            
                <tr>
                    <td>000199</td>
                    <td>pair_000199/stitched.png</td>
                    <td>000199.jpg</td>
                    <td class="poor">0.4188</td>
                    <td class="poor">17.7192</td>
                    <td>1099.4000</td>
                    <td>33.1572</td>
                </tr>
            
                <tr>
                    <td>000222</td>
                    <td>pair_000222/stitched.png</td>
                    <td>000222.jpg</td>
                    <td class="poor">0.4587</td>
                    <td class="poor">15.9759</td>
                    <td>1642.4597</td>
                    <td>40.5273</td>
                </tr>
            
                <tr>
                    <td>000226</td>
                    <td>pair_000226/stitched.png</td>
                    <td>000226.jpg</td>
                    <td class="poor">0.3561</td>
                    <td class="poor">13.1106</td>
                    <td>3177.0554</td>
                    <td>56.3654</td>
                </tr>
            
                <tr>
                    <td>000228</td>
                    <td>pair_000228/stitched.png</td>
                    <td>000228.jpg</td>
                    <td class="poor">0.4808</td>
                    <td class="poor">17.0110</td>
                    <td>1294.1277</td>
                    <td>35.9740</td>
                </tr>
            
                <tr>
                    <td>000229</td>
                    <td>pair_000229/stitched.png</td>
                    <td>000229.jpg</td>
                    <td class="poor">0.4194</td>
                    <td class="poor">17.0258</td>
                    <td>1289.7352</td>
                    <td>35.9129</td>
                </tr>
            
                <tr>
                    <td>000241</td>
                    <td>pair_000241/stitched.png</td>
                    <td>000241.jpg</td>
                    <td class="average">0.8449</td>
                    <td class="average">22.8269</td>
                    <td>339.1482</td>
                    <td>18.4160</td>
                </tr>
            
                <tr>
                    <td>000254</td>
                    <td>pair_000254/stitched.png</td>
                    <td>000254.jpg</td>
                    <td class="poor">0.6050</td>
                    <td class="poor">16.6915</td>
                    <td>1392.9455</td>
                    <td>37.3222</td>
                </tr>
            
                <tr>
                    <td>000256</td>
                    <td>pair_000256/stitched.png</td>
                    <td>000256.jpg</td>
                    <td class="poor">0.3380</td>
                    <td class="poor">13.4902</td>
                    <td>2911.1191</td>
                    <td>53.9548</td>
                </tr>
            
                <tr>
                    <td>000257</td>
                    <td>pair_000257/stitched.png</td>
                    <td>000257.jpg</td>
                    <td class="poor">0.5519</td>
                    <td class="poor">17.2020</td>
                    <td>1238.4516</td>
                    <td>35.1916</td>
                </tr>
            
                <tr>
                    <td>000260</td>
                    <td>pair_000260/stitched.png</td>
                    <td>000260.jpg</td>
                    <td class="poor">0.5131</td>
                    <td class="poor">16.9379</td>
                    <td>1316.1075</td>
                    <td>36.2782</td>
                </tr>
            
                <tr>
                    <td>000263</td>
                    <td>pair_000263/stitched.png</td>
                    <td>000263.jpg</td>
                    <td class="poor">0.6559</td>
                    <td class="poor">19.5701</td>
                    <td>717.9182</td>
                    <td>26.7940</td>
                </tr>
            
                <tr>
                    <td>000282</td>
                    <td>pair_000282/stitched.png</td>
                    <td>000282.jpg</td>
                    <td class="poor">0.5400</td>
                    <td class="poor">16.9893</td>
                    <td>1300.6118</td>
                    <td>36.0640</td>
                </tr>
            
                <tr>
                    <td>000283</td>
                    <td>pair_000283/stitched.png</td>
                    <td>000283.jpg</td>
                    <td class="poor">0.6529</td>
                    <td class="poor">17.6155</td>
                    <td>1125.9799</td>
                    <td>33.5556</td>
                </tr>
            
                <tr>
                    <td>000285</td>
                    <td>pair_000285/stitched.png</td>
                    <td>000285.jpg</td>
                    <td class="poor">0.5913</td>
                    <td class="poor">18.1181</td>
                    <td>1002.9234</td>
                    <td>31.6690</td>
                </tr>
            
                <tr>
                    <td>000304</td>
                    <td>pair_000304/stitched.png</td>
                    <td>000304.jpg</td>
                    <td class="poor">0.3217</td>
                    <td class="poor">14.3854</td>
                    <td>2368.8507</td>
                    <td>48.6708</td>
                </tr>
            
                <tr>
                    <td>000313</td>
                    <td>pair_000313/stitched.png</td>
                    <td>000313.jpg</td>
                    <td class="poor">0.3971</td>
                    <td class="poor">16.0279</td>
                    <td>1622.9093</td>
                    <td>40.2853</td>
                </tr>
            
                <tr>
                    <td>000320</td>
                    <td>pair_000320/stitched.png</td>
                    <td>000320.jpg</td>
                    <td class="poor">0.3598</td>
                    <td class="poor">18.8846</td>
                    <td>840.6555</td>
                    <td>28.9941</td>
                </tr>
            
                <tr>
                    <td>000326</td>
                    <td>pair_000326/stitched.png</td>
                    <td>000326.jpg</td>
                    <td class="poor">0.2825</td>
                    <td class="poor">17.3323</td>
                    <td>1201.8400</td>
                    <td>34.6676</td>
                </tr>
            
                <tr>
                    <td>000337</td>
                    <td>pair_000337/stitched.png</td>
                    <td>000337.jpg</td>
                    <td class="poor">0.6501</td>
                    <td class="poor">18.6415</td>
                    <td>889.0667</td>
                    <td>29.8172</td>
                </tr>
            
                <tr>
                    <td>000338</td>
                    <td>pair_000338/stitched.png</td>
                    <td>000338.jpg</td>
                    <td class="average">0.7797</td>
                    <td class="average">25.5800</td>
                    <td>179.9218</td>
                    <td>13.4135</td>
                </tr>
            
                <tr>
                    <td>000339</td>
                    <td>pair_000339/stitched.png</td>
                    <td>000339.jpg</td>
                    <td class="poor">0.5903</td>
                    <td class="average">20.0772</td>
                    <td>638.7949</td>
                    <td>25.2744</td>
                </tr>
            
                <tr>
                    <td>000363</td>
                    <td>pair_000363/stitched.png</td>
                    <td>000363.jpg</td>
                    <td class="poor">0.5778</td>
                    <td class="poor">19.3601</td>
                    <td>753.4748</td>
                    <td>27.4495</td>
                </tr>
            
                <tr>
                    <td>000366</td>
                    <td>pair_000366/stitched.png</td>
                    <td>000366.jpg</td>
                    <td class="poor">0.6902</td>
                    <td class="poor">18.7089</td>
                    <td>875.3577</td>
                    <td>29.5864</td>
                </tr>
            
                <tr>
                    <td>000371</td>
                    <td>pair_000371/stitched.png</td>
                    <td>000371.jpg</td>
                    <td class="poor">0.5843</td>
                    <td class="poor">15.9180</td>
                    <td>1664.4737</td>
                    <td>40.7980</td>
                </tr>
            
                <tr>
                    <td>000375</td>
                    <td>pair_000375/stitched.png</td>
                    <td>000375.jpg</td>
                    <td class="poor">0.6154</td>
                    <td class="average">21.6137</td>
                    <td>448.4416</td>
                    <td>21.1764</td>
                </tr>
            
                <tr>
                    <td>000392</td>
                    <td>pair_000392/stitched.png</td>
                    <td>000392.jpg</td>
                    <td class="poor">0.4467</td>
                    <td class="poor">17.7050</td>
                    <td>1103.0093</td>
                    <td>33.2116</td>
                </tr>
            
                <tr>
                    <td>000393</td>
                    <td>pair_000393/stitched.png</td>
                    <td>000393.jpg</td>
                    <td class="poor">0.3350</td>
                    <td class="poor">14.8958</td>
                    <td>2106.2196</td>
                    <td>45.8936</td>
                </tr>
            
                <tr>
                    <td>000394</td>
                    <td>pair_000394/stitched.png</td>
                    <td>000394.jpg</td>
                    <td class="poor">0.5153</td>
                    <td class="average">21.6191</td>
                    <td>447.8906</td>
                    <td>21.1634</td>
                </tr>
            
                <tr>
                    <td>000396</td>
                    <td>pair_000396/stitched.png</td>
                    <td>000396.jpg</td>
                    <td class="poor">0.2999</td>
                    <td class="poor">16.2538</td>
                    <td>1540.6401</td>
                    <td>39.2510</td>
                </tr>
            
                <tr>
                    <td>000423</td>
                    <td>pair_000423/stitched.png</td>
                    <td>000423.jpg</td>
                    <td class="poor">0.2644</td>
                    <td class="poor">14.1092</td>
                    <td>2524.4342</td>
                    <td>50.2437</td>
                </tr>
            
                <tr>
                    <td>000443</td>
                    <td>pair_000443/stitched.png</td>
                    <td>000443.jpg</td>
                    <td class="poor">0.5409</td>
                    <td class="poor">18.8496</td>
                    <td>847.4588</td>
                    <td>29.1111</td>
                </tr>
            
                <tr>
                    <td>000444</td>
                    <td>pair_000444/stitched.png</td>
                    <td>000444.jpg</td>
                    <td class="poor">0.3647</td>
                    <td class="poor">15.2513</td>
                    <td>1940.6481</td>
                    <td>44.0528</td>
                </tr>
            
                <tr>
                    <td>000459</td>
                    <td>pair_000459/stitched.png</td>
                    <td>000459.jpg</td>
                    <td class="poor">0.5604</td>
                    <td class="poor">14.2508</td>
                    <td>2443.4193</td>
                    <td>49.4310</td>
                </tr>
            
                <tr>
                    <td>000461</td>
                    <td>pair_000461/stitched.png</td>
                    <td>000461.jpg</td>
                    <td class="poor">0.2899</td>
                    <td class="poor">13.4528</td>
                    <td>2936.3227</td>
                    <td>54.1878</td>
                </tr>
            
                <tr>
                    <td>000465</td>
                    <td>pair_000465/stitched.png</td>
                    <td>000465.jpg</td>
                    <td class="poor">0.4582</td>
                    <td class="poor">16.9955</td>
                    <td>1298.7783</td>
                    <td>36.0386</td>
                </tr>
            
                <tr>
                    <td>000479</td>
                    <td>pair_000479/stitched.png</td>
                    <td>000479.jpg</td>
                    <td class="poor">0.7086</td>
                    <td class="average">21.1805</td>
                    <td>495.4876</td>
                    <td>22.2596</td>
                </tr>
            
                <tr>
                    <td>000486</td>
                    <td>pair_000486/stitched.png</td>
                    <td>000486.jpg</td>
                    <td class="poor">0.2483</td>
                    <td class="poor">15.9283</td>
                    <td>1660.5403</td>
                    <td>40.7497</td>
                </tr>
            
                <tr>
                    <td>000496</td>
                    <td>pair_000496/stitched.png</td>
                    <td>000496.jpg</td>
                    <td class="poor">0.4477</td>
                    <td class="poor">15.6375</td>
                    <td>1775.5491</td>
                    <td>42.1373</td>
                </tr>
            
                <tr>
                    <td>000497</td>
                    <td>pair_000497/stitched.png</td>
                    <td>000497.jpg</td>
                    <td class="poor">0.3556</td>
                    <td class="poor">17.7611</td>
                    <td>1088.8575</td>
                    <td>32.9978</td>
                </tr>
            
                <tr>
                    <td>000498</td>
                    <td>pair_000498/stitched.png</td>
                    <td>000498.jpg</td>
                    <td class="poor">0.5026</td>
                    <td class="poor">18.4305</td>
                    <td>933.3303</td>
                    <td>30.5505</td>
                </tr>
            
                <tr>
                    <td>000506</td>
                    <td>pair_000506/stitched.png</td>
                    <td>000506.jpg</td>
                    <td class="poor">0.4736</td>
                    <td class="poor">14.8937</td>
                    <td>2107.2181</td>
                    <td>45.9044</td>
                </tr>
            
                <tr>
                    <td>000507</td>
                    <td>pair_000507/stitched.png</td>
                    <td>000507.jpg</td>
                    <td class="poor">0.5517</td>
                    <td class="poor">17.5085</td>
                    <td>1154.0526</td>
                    <td>33.9714</td>
                </tr>
            
                <tr>
                    <td>000516</td>
                    <td>pair_000516/stitched.png</td>
                    <td>000516.jpg</td>
                    <td class="poor">0.4816</td>
                    <td class="poor">17.8726</td>
                    <td>1061.2666</td>
                    <td>32.5771</td>
                </tr>
            
                <tr>
                    <td>001909</td>
                    <td>pair_001909/stitched.png</td>
                    <td>001909.jpg</td>
                    <td class="poor">0.2870</td>
                    <td class="poor">10.9771</td>
                    <td>5192.4829</td>
                    <td>72.0589</td>
                </tr>
            
                <tr>
                    <td>001910</td>
                    <td>pair_001910/stitched.png</td>
                    <td>001910.jpg</td>
                    <td class="poor">0.5245</td>
                    <td class="poor">13.9428</td>
                    <td>2623.0320</td>
                    <td>51.2155</td>
                </tr>
            
                <tr>
                    <td>001911</td>
                    <td>pair_001911/stitched.png</td>
                    <td>001911.jpg</td>
                    <td class="poor">0.5046</td>
                    <td class="poor">13.5357</td>
                    <td>2880.7611</td>
                    <td>53.6727</td>
                </tr>
            
                <tr>
                    <td>001947</td>
                    <td>pair_001947/stitched.png</td>
                    <td>001947.jpg</td>
                    <td class="poor">0.3808</td>
                    <td class="poor">15.3767</td>
                    <td>1885.4338</td>
                    <td>43.4216</td>
                </tr>
            
                <tr>
                    <td>002020</td>
                    <td>pair_002020/stitched.png</td>
                    <td>002020.jpg</td>
                    <td class="poor">0.2296</td>
                    <td class="poor">15.4440</td>
                    <td>1856.4206</td>
                    <td>43.0862</td>
                </tr>
            
                <tr>
                    <td>002021</td>
                    <td>pair_002021/stitched.png</td>
                    <td>002021.jpg</td>
                    <td class="poor">0.2402</td>
                    <td class="poor">15.1145</td>
                    <td>2002.7689</td>
                    <td>44.7523</td>
                </tr>
            
                <tr>
                    <td>002062</td>
                    <td>pair_002062/stitched.png</td>
                    <td>002062.jpg</td>
                    <td class="poor">0.5163</td>
                    <td class="average">20.1649</td>
                    <td>626.0188</td>
                    <td>25.0204</td>
                </tr>
            
                <tr>
                    <td>002090</td>
                    <td>pair_002090/stitched.png</td>
                    <td>002090.jpg</td>
                    <td class="poor">0.3271</td>
                    <td class="poor">16.2520</td>
                    <td>1541.2591</td>
                    <td>39.2589</td>
                </tr>
            
                <tr>
                    <td>002097</td>
                    <td>pair_002097/stitched.png</td>
                    <td>002097.jpg</td>
                    <td class="poor">0.4147</td>
                    <td class="poor">13.6073</td>
                    <td>2833.6822</td>
                    <td>53.2323</td>
                </tr>
            
                <tr>
                    <td>002143</td>
                    <td>pair_002143/stitched.png</td>
                    <td>002143.jpg</td>
                    <td class="average">0.7531</td>
                    <td class="average">21.8287</td>
                    <td>426.7844</td>
                    <td>20.6588</td>
                </tr>
            
                <tr>
                    <td>002145</td>
                    <td>pair_002145/stitched.png</td>
                    <td>002145.jpg</td>
                    <td class="average">0.7914</td>
                    <td class="average">25.9415</td>
                    <td>165.5491</td>
                    <td>12.8666</td>
                </tr>
            
                <tr>
                    <td>002147</td>
                    <td>pair_002147/stitched.png</td>
                    <td>002147.jpg</td>
                    <td class="poor">0.3377</td>
                    <td class="poor">18.5362</td>
                    <td>910.8788</td>
                    <td>30.1808</td>
                </tr>
            
                <tr>
                    <td>002168</td>
                    <td>pair_002168/stitched.png</td>
                    <td>002168.jpg</td>
                    <td class="poor">0.3975</td>
                    <td class="poor">14.7345</td>
                    <td>2185.9175</td>
                    <td>46.7538</td>
                </tr>
            
                <tr>
                    <td>002175</td>
                    <td>pair_002175/stitched.png</td>
                    <td>002175.jpg</td>
                    <td class="poor">0.5048</td>
                    <td class="poor">13.4873</td>
                    <td>2913.0928</td>
                    <td>53.9731</td>
                </tr>
            
                <tr>
                    <td>002176</td>
                    <td>pair_002176/stitched.png</td>
                    <td>002176.jpg</td>
                    <td class="poor">0.3173</td>
                    <td class="poor">13.5288</td>
                    <td>2885.3454</td>
                    <td>53.7154</td>
                </tr>
            
                <tr>
                    <td>002248</td>
                    <td>pair_002248/stitched.png</td>
                    <td>002248.jpg</td>
                    <td class="poor">0.4272</td>
                    <td class="poor">14.9554</td>
                    <td>2077.5131</td>
                    <td>45.5797</td>
                </tr>
            
                <tr>
                    <td>002250</td>
                    <td>pair_002250/stitched.png</td>
                    <td>002250.jpg</td>
                    <td class="poor">0.2843</td>
                    <td class="poor">14.4384</td>
                    <td>2340.1491</td>
                    <td>48.3751</td>
                </tr>
            
                <tr>
                    <td>002251</td>
                    <td>pair_002251/stitched.png</td>
                    <td>002251.jpg</td>
                    <td class="poor">0.2477</td>
                    <td class="poor">12.8753</td>
                    <td>3353.8844</td>
                    <td>57.9127</td>
                </tr>
            
                <tr>
                    <td>002253</td>
                    <td>pair_002253/stitched.png</td>
                    <td>002253.jpg</td>
                    <td class="poor">0.2800</td>
                    <td class="poor">13.1250</td>
                    <td>3166.5035</td>
                    <td>56.2717</td>
                </tr>
            
                <tr>
                    <td>002263</td>
                    <td>pair_002263/stitched.png</td>
                    <td>002263.jpg</td>
                    <td class="poor">0.3116</td>
                    <td class="poor">18.9515</td>
                    <td>827.8106</td>
                    <td>28.7717</td>
                </tr>
            
                <tr>
                    <td>002273</td>
                    <td>pair_002273/stitched.png</td>
                    <td>002273.jpg</td>
                    <td class="poor">0.4294</td>
                    <td class="poor">16.3165</td>
                    <td>1518.5448</td>
                    <td>38.9685</td>
                </tr>
            
                <tr>
                    <td>002280</td>
                    <td>pair_002280/stitched.png</td>
                    <td>002280.jpg</td>
                    <td class="poor">0.2562</td>
                    <td class="poor">13.5734</td>
                    <td>2855.8620</td>
                    <td>53.4403</td>
                </tr>
            
                <tr>
                    <td>002284</td>
                    <td>pair_002284/stitched.png</td>
                    <td>002284.jpg</td>
                    <td class="poor">0.1727</td>
                    <td class="poor">13.3565</td>
                    <td>3002.1276</td>
                    <td>54.7917</td>
                </tr>
            
                <tr>
                    <td>002291</td>
                    <td>pair_002291/stitched.png</td>
                    <td>002291.jpg</td>
                    <td class="poor">0.4163</td>
                    <td class="poor">16.3985</td>
                    <td>1490.1509</td>
                    <td>38.6025</td>
                </tr>
            
                <tr>
                    <td>002297</td>
                    <td>pair_002297/stitched.png</td>
                    <td>002297.jpg</td>
                    <td class="poor">0.4670</td>
                    <td class="poor">13.8476</td>
                    <td>2681.1470</td>
                    <td>51.7798</td>
                </tr>
            
                <tr>
                    <td>002306</td>
                    <td>pair_002306/stitched.png</td>
                    <td>002306.jpg</td>
                    <td class="poor">0.4350</td>
                    <td class="poor">16.7122</td>
                    <td>1386.3185</td>
                    <td>37.2333</td>
                </tr>
            
                <tr>
                    <td>002307</td>
                    <td>pair_002307/stitched.png</td>
                    <td>002307.jpg</td>
                    <td class="poor">0.2523</td>
                    <td class="poor">14.2027</td>
                    <td>2470.6706</td>
                    <td>49.7058</td>
                </tr>
            
                <tr>
                    <td>002319</td>
                    <td>pair_002319/stitched.png</td>
                    <td>002319.jpg</td>
                    <td class="poor">0.3688</td>
                    <td class="poor">12.5652</td>
                    <td>3602.1037</td>
                    <td>60.0175</td>
                </tr>
            
                <tr>
                    <td>002320</td>
                    <td>pair_002320/stitched.png</td>
                    <td>002320.jpg</td>
                    <td class="poor">0.1205</td>
                    <td class="poor">10.5428</td>
                    <td>5738.5077</td>
                    <td>75.7529</td>
                </tr>
            
                <tr>
                    <td>002321</td>
                    <td>pair_002321/stitched.png</td>
                    <td>002321.jpg</td>
                    <td class="poor">0.0708</td>
                    <td class="poor">9.2323</td>
                    <td>7759.7989</td>
                    <td>88.0897</td>
                </tr>
            
                <tr>
                    <td>002322</td>
                    <td>pair_002322/stitched.png</td>
                    <td>002322.jpg</td>
                    <td class="poor">0.1123</td>
                    <td class="poor">9.4159</td>
                    <td>7438.5469</td>
                    <td>86.2470</td>
                </tr>
            
                <tr>
                    <td>002370</td>
                    <td>pair_002370/stitched.png</td>
                    <td>002370.jpg</td>
                    <td class="poor">0.2814</td>
                    <td class="poor">14.0209</td>
                    <td>2576.2590</td>
                    <td>50.7569</td>
                </tr>
            
                <tr>
                    <td>002383</td>
                    <td>pair_002383/stitched.png</td>
                    <td>002383.jpg</td>
                    <td class="poor">0.3891</td>
                    <td class="poor">13.7099</td>
                    <td>2767.5408</td>
                    <td>52.6074</td>
                </tr>
            
                <tr>
                    <td>002471</td>
                    <td>pair_002471/stitched.png</td>
                    <td>002471.jpg</td>
                    <td class="poor">0.4219</td>
                    <td class="poor">15.3547</td>
                    <td>1894.9916</td>
                    <td>43.5315</td>
                </tr>
            
                <tr>
                    <td>002476</td>
                    <td>pair_002476/stitched.png</td>
                    <td>002476.jpg</td>
                    <td class="poor">0.2997</td>
                    <td class="poor">15.5794</td>
                    <td>1799.4657</td>
                    <td>42.4201</td>
                </tr>
            
                <tr>
                    <td>002492</td>
                    <td>pair_002492/stitched.png</td>
                    <td>002492.jpg</td>
                    <td class="poor">0.5669</td>
                    <td class="poor">17.6553</td>
                    <td>1115.7179</td>
                    <td>33.4024</td>
                </tr>
            
                <tr>
                    <td>002499</td>
                    <td>pair_002499/stitched.png</td>
                    <td>002499.jpg</td>
                    <td class="poor">0.2976</td>
                    <td class="poor">15.7680</td>
                    <td>1722.9970</td>
                    <td>41.5090</td>
                </tr>
            
                <tr>
                    <td>002544</td>
                    <td>pair_002544/stitched.png</td>
                    <td>002544.jpg</td>
                    <td class="poor">0.4992</td>
                    <td class="poor">16.6477</td>
                    <td>1407.0622</td>
                    <td>37.5108</td>
                </tr>
            
                <tr>
                    <td>002588</td>
                    <td>pair_002588/stitched.png</td>
                    <td>002588.jpg</td>
                    <td class="poor">0.2257</td>
                    <td class="poor">13.5665</td>
                    <td>2860.4331</td>
                    <td>53.4830</td>
                </tr>
            
                <tr>
                    <td>002589</td>
                    <td>pair_002589/stitched.png</td>
                    <td>002589.jpg</td>
                    <td class="poor">0.2271</td>
                    <td class="poor">13.7425</td>
                    <td>2746.8312</td>
                    <td>52.4102</td>
                </tr>
            
                <tr>
                    <td>002625</td>
                    <td>pair_002625/stitched.png</td>
                    <td>002625.jpg</td>
                    <td class="poor">0.3318</td>
                    <td class="poor">17.0214</td>
                    <td>1291.0291</td>
                    <td>35.9309</td>
                </tr>
            
                <tr>
                    <td>002650</td>
                    <td>pair_002650/stitched.png</td>
                    <td>002650.jpg</td>
                    <td class="poor">0.2778</td>
                    <td class="poor">12.2705</td>
                    <td>3855.0798</td>
                    <td>62.0893</td>
                </tr>
            
                <tr>
                    <td>002675</td>
                    <td>pair_002675/stitched.png</td>
                    <td>002675.jpg</td>
                    <td class="poor">0.3245</td>
                    <td class="poor">14.6147</td>
                    <td>2247.0394</td>
                    <td>47.4029</td>
                </tr>
            
                <tr>
                    <td>002705</td>
                    <td>pair_002705/stitched.png</td>
                    <td>002705.jpg</td>
                    <td class="poor">0.2039</td>
                    <td class="poor">13.1105</td>
                    <td>3177.0799</td>
                    <td>56.3656</td>
                </tr>
            
                <tr>
                    <td>002741</td>
                    <td>pair_002741/stitched.png</td>
                    <td>002741.jpg</td>
                    <td class="poor">0.4339</td>
                    <td class="poor">17.1298</td>
                    <td>1259.2127</td>
                    <td>35.4854</td>
                </tr>
            
                <tr>
                    <td>002804</td>
                    <td>pair_002804/stitched.png</td>
                    <td>002804.jpg</td>
                    <td class="poor">0.2782</td>
                    <td class="poor">15.2273</td>
                    <td>1951.4173</td>
                    <td>44.1748</td>
                </tr>
            
                <tr>
                    <td>002838</td>
                    <td>pair_002838/stitched.png</td>
                    <td>002838.jpg</td>
                    <td class="poor">0.6252</td>
                    <td class="average">20.2706</td>
                    <td>610.9667</td>
                    <td>24.7177</td>
                </tr>
            
                <tr>
                    <td>002851</td>
                    <td>pair_002851/stitched.png</td>
                    <td>002851.jpg</td>
                    <td class="poor">0.1868</td>
                    <td class="poor">13.6646</td>
                    <td>2796.5066</td>
                    <td>52.8820</td>
                </tr>
            
                <tr>
                    <td>002915</td>
                    <td>pair_002915/stitched.png</td>
                    <td>002915.jpg</td>
                    <td class="poor">0.4313</td>
                    <td class="poor">18.5245</td>
                    <td>913.3358</td>
                    <td>30.2214</td>
                </tr>
            
                <tr>
                    <td>002917</td>
                    <td>pair_002917/stitched.png</td>
                    <td>002917.jpg</td>
                    <td class="poor">0.3332</td>
                    <td class="poor">13.7837</td>
                    <td>2720.8637</td>
                    <td>52.1619</td>
                </tr>
            
                <tr>
                    <td>002936</td>
                    <td>pair_002936/stitched.png</td>
                    <td>002936.jpg</td>
                    <td class="poor">0.3360</td>
                    <td class="poor">18.0365</td>
                    <td>1021.9599</td>
                    <td>31.9681</td>
                </tr>
            
                <tr>
                    <td>003052</td>
                    <td>pair_003052/stitched.png</td>
                    <td>003052.jpg</td>
                    <td class="poor">0.2293</td>
                    <td class="poor">16.0034</td>
                    <td>1632.0768</td>
                    <td>40.3990</td>
                </tr>
            
                <tr>
                    <td>003053</td>
                    <td>pair_003053/stitched.png</td>
                    <td>003053.jpg</td>
                    <td class="poor">0.2734</td>
                    <td class="poor">16.2836</td>
                    <td>1530.1135</td>
                    <td>39.1167</td>
                </tr>
            
                <tr>
                    <td>003055</td>
                    <td>pair_003055/stitched.png</td>
                    <td>003055.jpg</td>
                    <td class="poor">0.2088</td>
                    <td class="poor">15.4071</td>
                    <td>1872.2832</td>
                    <td>43.2699</td>
                </tr>
            
                <tr>
                    <td>003059</td>
                    <td>pair_003059/stitched.png</td>
                    <td>003059.jpg</td>
                    <td class="poor">0.2914</td>
                    <td class="poor">15.1326</td>
                    <td>1994.4255</td>
                    <td>44.6590</td>
                </tr>
            
                <tr>
                    <td>003109</td>
                    <td>pair_003109/stitched.png</td>
                    <td>003109.jpg</td>
                    <td class="poor">0.3326</td>
                    <td class="poor">12.5371</td>
                    <td>3625.4906</td>
                    <td>60.2120</td>
                </tr>
            
                <tr>
                    <td>003115</td>
                    <td>pair_003115/stitched.png</td>
                    <td>003115.jpg</td>
                    <td class="poor">0.3474</td>
                    <td class="poor">17.3696</td>
                    <td>1191.5645</td>
                    <td>34.5190</td>
                </tr>
            
                <tr>
                    <td>003117</td>
                    <td>pair_003117/stitched.png</td>
                    <td>003117.jpg</td>
                    <td class="poor">0.3095</td>
                    <td class="poor">13.6792</td>
                    <td>2787.1689</td>
                    <td>52.7936</td>
                </tr>
            
                <tr>
                    <td>003156</td>
                    <td>pair_003156/stitched.png</td>
                    <td>003156.jpg</td>
                    <td class="poor">0.3246</td>
                    <td class="poor">13.4404</td>
                    <td>2944.7198</td>
                    <td>54.2653</td>
                </tr>
            
                <tr>
                    <td>003245</td>
                    <td>pair_003245/stitched.png</td>
                    <td>003245.jpg</td>
                    <td class="poor">0.2614</td>
                    <td class="poor">16.7590</td>
                    <td>1371.4523</td>
                    <td>37.0331</td>
                </tr>
            
                <tr>
                    <td>003327</td>
                    <td>pair_003327/stitched.png</td>
                    <td>003327.jpg</td>
                    <td class="poor">0.4213</td>
                    <td class="poor">15.7460</td>
                    <td>1731.7402</td>
                    <td>41.6142</td>
                </tr>
            
                <tr>
                    <td>003345</td>
                    <td>pair_003345/stitched.png</td>
                    <td>003345.jpg</td>
                    <td class="poor">0.6372</td>
                    <td class="poor">19.3342</td>
                    <td>757.9793</td>
                    <td>27.5314</td>
                </tr>
            
                <tr>
                    <td>003427</td>
                    <td>pair_003427/stitched.png</td>
                    <td>003427.jpg</td>
                    <td class="poor">0.5217</td>
                    <td class="poor">13.3658</td>
                    <td>2995.6872</td>
                    <td>54.7329</td>
                </tr>
            
                <tr>
                    <td>003435</td>
                    <td>pair_003435/stitched.png</td>
                    <td>003435.jpg</td>
                    <td class="poor">0.3053</td>
                    <td class="poor">14.7712</td>
                    <td>2167.4988</td>
                    <td>46.5564</td>
                </tr>
            
                <tr>
                    <td>003449</td>
                    <td>pair_003449/stitched.png</td>
                    <td>003449.jpg</td>
                    <td class="poor">0.3235</td>
                    <td class="poor">13.9901</td>
                    <td>2594.6138</td>
                    <td>50.9374</td>
                </tr>
            
                <tr>
                    <td>003454</td>
                    <td>pair_003454/stitched.png</td>
                    <td>003454.jpg</td>
                    <td class="poor">0.3747</td>
                    <td class="poor">13.2655</td>
                    <td>3065.7169</td>
                    <td>55.3689</td>
                </tr>
            
                <tr>
                    <td>003484</td>
                    <td>pair_003484/stitched.png</td>
                    <td>003484.jpg</td>
                    <td class="poor">0.2235</td>
                    <td class="poor">12.1054</td>
                    <td>4004.4481</td>
                    <td>63.2807</td>
                </tr>
            
                <tr>
                    <td>003491</td>
                    <td>pair_003491/stitched.png</td>
                    <td>003491.jpg</td>
                    <td class="poor">0.3977</td>
                    <td class="poor">15.9550</td>
                    <td>1650.3766</td>
                    <td>40.6248</td>
                </tr>
            
                <tr>
                    <td>003507</td>
                    <td>pair_003507/stitched.png</td>
                    <td>003507.jpg</td>
                    <td class="poor">0.3733</td>
                    <td class="poor">14.1834</td>
                    <td>2481.6477</td>
                    <td>49.8161</td>
                </tr>
            
                <tr>
                    <td>003520</td>
                    <td>pair_003520/stitched.png</td>
                    <td>003520.jpg</td>
                    <td class="poor">0.1228</td>
                    <td class="poor">9.6584</td>
                    <td>7034.5375</td>
                    <td>83.8721</td>
                </tr>
            
                <tr>
                    <td>003521</td>
                    <td>pair_003521/stitched.png</td>
                    <td>003521.jpg</td>
                    <td class="poor">0.2071</td>
                    <td class="poor">11.0282</td>
                    <td>5131.6487</td>
                    <td>71.6355</td>
                </tr>
            
                <tr>
                    <td>003522</td>
                    <td>pair_003522/stitched.png</td>
                    <td>003522.jpg</td>
                    <td class="poor">0.1874</td>
                    <td class="poor">10.2964</td>
                    <td>6073.5690</td>
                    <td>77.9331</td>
                </tr>
            
                <tr>
                    <td>003528</td>
                    <td>pair_003528/stitched.png</td>
                    <td>003528.jpg</td>
                    <td class="poor">0.5572</td>
                    <td class="poor">14.7681</td>
                    <td>2169.0453</td>
                    <td>46.5730</td>
                </tr>
            
                <tr>
                    <td>003529</td>
                    <td>pair_003529/stitched.png</td>
                    <td>003529.jpg</td>
                    <td class="poor">0.5434</td>
                    <td class="poor">17.7992</td>
                    <td>1079.3500</td>
                    <td>32.8535</td>
                </tr>
            
                <tr>
                    <td>003570</td>
                    <td>pair_003570/stitched.png</td>
                    <td>003570.jpg</td>
                    <td class="poor">0.3194</td>
                    <td class="poor">15.1772</td>
                    <td>1974.0751</td>
                    <td>44.4306</td>
                </tr>
            
                <tr>
                    <td>003589</td>
                    <td>pair_003589/stitched.png</td>
                    <td>003589.jpg</td>
                    <td class="average">0.7669</td>
                    <td class="average">22.3213</td>
                    <td>381.0226</td>
                    <td>19.5198</td>
                </tr>
            
                <tr>
                    <td>003686</td>
                    <td>pair_003686/stitched.png</td>
                    <td>003686.jpg</td>
                    <td class="poor">0.2991</td>
                    <td class="poor">16.7718</td>
                    <td>1367.4053</td>
                    <td>36.9784</td>
                </tr>
            
                <tr>
                    <td>003755</td>
                    <td>pair_003755/stitched.png</td>
                    <td>003755.jpg</td>
                    <td class="poor">0.3872</td>
                    <td class="poor">12.8804</td>
                    <td>3349.9575</td>
                    <td>57.8788</td>
                </tr>
            
                <tr>
                    <td>001909</td>
                    <td>pair_001909/stitched.png</td>
                    <td>001909.jpg</td>
                    <td class="good">0.9285</td>
                    <td class="average">23.2781</td>
                    <td>305.6842</td>
                    <td>17.4838</td>
                </tr>
            
                <tr>
                    <td>001910</td>
                    <td>pair_001910/stitched.png</td>
                    <td>001910.jpg</td>
                    <td class="good">0.9272</td>
                    <td class="average">24.4064</td>
                    <td>235.7417</td>
                    <td>15.3539</td>
                </tr>
            
                <tr>
                    <td>001911</td>
                    <td>pair_001911/stitched.png</td>
                    <td>001911.jpg</td>
                    <td class="good">0.9633</td>
                    <td class="average">26.1033</td>
                    <td>159.4979</td>
                    <td>12.6292</td>
                </tr>
            
                <tr>
                    <td>001947</td>
                    <td>pair_001947/stitched.png</td>
                    <td>001947.jpg</td>
                    <td class="average">0.8923</td>
                    <td class="average">26.6532</td>
                    <td>140.5278</td>
                    <td>11.8544</td>
                </tr>
            
                <tr>
                    <td>002020</td>
                    <td>pair_002020/stitched.png</td>
                    <td>002020.jpg</td>
                    <td class="average">0.8687</td>
                    <td class="average">28.3263</td>
                    <td>95.5990</td>
                    <td>9.7775</td>
                </tr>
            
                <tr>
                    <td>002021</td>
                    <td>pair_002021/stitched.png</td>
                    <td>002021.jpg</td>
                    <td class="average">0.8947</td>
                    <td class="average">29.3379</td>
                    <td>75.7334</td>
                    <td>8.7025</td>
                </tr>
            
                <tr>
                    <td>002062</td>
                    <td>pair_002062/stitched.png</td>
                    <td>002062.jpg</td>
                    <td class="good">0.9031</td>
                    <td class="average">29.1549</td>
                    <td>78.9934</td>
                    <td>8.8878</td>
                </tr>
            
                <tr>
                    <td>002090</td>
                    <td>pair_002090/stitched.png</td>
                    <td>002090.jpg</td>
                    <td class="average">0.8321</td>
                    <td class="average">26.6050</td>
                    <td>142.0944</td>
                    <td>11.9203</td>
                </tr>
            
                <tr>
                    <td>002097</td>
                    <td>pair_002097/stitched.png</td>
                    <td>002097.jpg</td>
                    <td class="average">0.8102</td>
                    <td class="average">23.7418</td>
                    <td>274.7241</td>
                    <td>16.5748</td>
                </tr>
            
                <tr>
                    <td>002143</td>
                    <td>pair_002143/stitched.png</td>
                    <td>002143.jpg</td>
                    <td class="good">0.9350</td>
                    <td class="good">34.4769</td>
                    <td>23.1948</td>
                    <td>4.8161</td>
                </tr>
            
                <tr>
                    <td>002145</td>
                    <td>pair_002145/stitched.png</td>
                    <td>002145.jpg</td>
                    <td class="good">0.9354</td>
                    <td class="good">34.8985</td>
                    <td>21.0489</td>
                    <td>4.5879</td>
                </tr>
            
                <tr>
                    <td>002147</td>
                    <td>pair_002147/stitched.png</td>
                    <td>002147.jpg</td>
                    <td class="good">0.9117</td>
                    <td class="good">30.6134</td>
                    <td>56.4597</td>
                    <td>7.5140</td>
                </tr>
            
                <tr>
                    <td>002168</td>
                    <td>pair_002168/stitched.png</td>
                    <td>002168.jpg</td>
                    <td class="average">0.7963</td>
                    <td class="average">22.2045</td>
                    <td>391.4046</td>
                    <td>19.7839</td>
                </tr>
            
                <tr>
                    <td>002175</td>
                    <td>pair_002175/stitched.png</td>
                    <td>002175.jpg</td>
                    <td class="good">0.9618</td>
                    <td class="good">35.2038</td>
                    <td>19.6200</td>
                    <td>4.4294</td>
                </tr>
            
                <tr>
                    <td>002176</td>
                    <td>pair_002176/stitched.png</td>
                    <td>002176.jpg</td>
                    <td class="good">0.9599</td>
                    <td class="good">31.7421</td>
                    <td>43.5381</td>
                    <td>6.5983</td>
                </tr>
            
                <tr>
                    <td>002248</td>
                    <td>pair_002248/stitched.png</td>
                    <td>002248.jpg</td>
                    <td class="good">0.9202</td>
                    <td class="average">26.4764</td>
                    <td>146.3655</td>
                    <td>12.0982</td>
                </tr>
            
                <tr>
                    <td>002250</td>
                    <td>pair_002250/stitched.png</td>
                    <td>002250.jpg</td>
                    <td class="average">0.8555</td>
                    <td class="average">23.1838</td>
                    <td>312.3894</td>
                    <td>17.6745</td>
                </tr>
            
                <tr>
                    <td>002251</td>
                    <td>pair_002251/stitched.png</td>
                    <td>002251.jpg</td>
                    <td class="average">0.7789</td>
                    <td class="poor">19.6483</td>
                    <td>705.1002</td>
                    <td>26.5537</td>
                </tr>
            
                <tr>
                    <td>002253</td>
                    <td>pair_002253/stitched.png</td>
                    <td>002253.jpg</td>
                    <td class="good">0.9136</td>
                    <td class="average">24.3692</td>
                    <td>237.7723</td>
                    <td>15.4199</td>
                </tr>
            
                <tr>
                    <td>002263</td>
                    <td>pair_002263/stitched.png</td>
                    <td>002263.jpg</td>
                    <td class="average">0.8168</td>
                    <td class="average">24.5544</td>
                    <td>227.8467</td>
                    <td>15.0946</td>
                </tr>
            
                <tr>
                    <td>002273</td>
                    <td>pair_002273/stitched.png</td>
                    <td>002273.jpg</td>
                    <td class="average">0.8339</td>
                    <td class="average">24.0503</td>
                    <td>255.8911</td>
                    <td>15.9966</td>
                </tr>
            
                <tr>
                    <td>002280</td>
                    <td>pair_002280/stitched.png</td>
                    <td>002280.jpg</td>
                    <td class="average">0.8067</td>
                    <td class="average">24.5888</td>
                    <td>226.0478</td>
                    <td>15.0349</td>
                </tr>
            
                <tr>
                    <td>002284</td>
                    <td>pair_002284/stitched.png</td>
                    <td>002284.jpg</td>
                    <td class="good">0.9451</td>
                    <td class="average">28.3925</td>
                    <td>94.1518</td>
                    <td>9.7032</td>
                </tr>
            
                <tr>
                    <td>002291</td>
                    <td>pair_002291/stitched.png</td>
                    <td>002291.jpg</td>
                    <td class="poor">0.5796</td>
                    <td class="poor">19.7504</td>
                    <td>688.7102</td>
                    <td>26.2433</td>
                </tr>
            
                <tr>
                    <td>002297</td>
                    <td>pair_002297/stitched.png</td>
                    <td>002297.jpg</td>
                    <td class="average">0.8997</td>
                    <td class="good">30.4626</td>
                    <td>58.4545</td>
                    <td>7.6456</td>
                </tr>
            
                <tr>
                    <td>002306</td>
                    <td>pair_002306/stitched.png</td>
                    <td>002306.jpg</td>
                    <td class="good">0.9000</td>
                    <td class="good">30.9623</td>
                    <td>52.1014</td>
                    <td>7.2181</td>
                </tr>
            
                <tr>
                    <td>002307</td>
                    <td>pair_002307/stitched.png</td>
                    <td>002307.jpg</td>
                    <td class="average">0.8245</td>
                    <td class="average">28.4291</td>
                    <td>93.3626</td>
                    <td>9.6624</td>
                </tr>
            
                <tr>
                    <td>002319</td>
                    <td>pair_002319/stitched.png</td>
                    <td>002319.jpg</td>
                    <td class="average">0.7776</td>
                    <td class="average">25.2625</td>
                    <td>193.5654</td>
                    <td>13.9128</td>
                </tr>
            
                <tr>
                    <td>002320</td>
                    <td>pair_002320/stitched.png</td>
                    <td>002320.jpg</td>
                    <td class="good">0.9408</td>
                    <td class="average">24.5246</td>
                    <td>229.4159</td>
                    <td>15.1465</td>
                </tr>
            
                <tr>
                    <td>002321</td>
                    <td>pair_002321/stitched.png</td>
                    <td>002321.jpg</td>
                    <td class="poor">0.5928</td>
                    <td class="poor">14.1657</td>
                    <td>2491.7759</td>
                    <td>49.9177</td>
                </tr>
            
                <tr>
                    <td>002322</td>
                    <td>pair_002322/stitched.png</td>
                    <td>002322.jpg</td>
                    <td class="average">0.8143</td>
                    <td class="poor">16.7619</td>
                    <td>1370.5227</td>
                    <td>37.0206</td>
                </tr>
            
                <tr>
                    <td>002370</td>
                    <td>pair_002370/stitched.png</td>
                    <td>002370.jpg</td>
                    <td class="average">0.8809</td>
                    <td class="average">26.8290</td>
                    <td>134.9524</td>
                    <td>11.6169</td>
                </tr>
            
                <tr>
                    <td>002383</td>
                    <td>pair_002383/stitched.png</td>
                    <td>002383.jpg</td>
                    <td class="average">0.8945</td>
                    <td class="average">26.9628</td>
                    <td>130.8568</td>
                    <td>11.4393</td>
                </tr>
            
                <tr>
                    <td>002471</td>
                    <td>pair_002471/stitched.png</td>
                    <td>002471.jpg</td>
                    <td class="good">0.9208</td>
                    <td class="average">28.4464</td>
                    <td>92.9901</td>
                    <td>9.6431</td>
                </tr>
            
                <tr>
                    <td>002476</td>
                    <td>pair_002476/stitched.png</td>
                    <td>002476.jpg</td>
                    <td class="average">0.7678</td>
                    <td class="average">22.2006</td>
                    <td>391.7634</td>
                    <td>19.7930</td>
                </tr>
            
                <tr>
                    <td>002492</td>
                    <td>pair_002492/stitched.png</td>
                    <td>002492.jpg</td>
                    <td class="average">0.8997</td>
                    <td class="average">28.0060</td>
                    <td>102.9161</td>
                    <td>10.1448</td>
                </tr>
            
                <tr>
                    <td>002499</td>
                    <td>pair_002499/stitched.png</td>
                    <td>002499.jpg</td>
                    <td class="good">0.9045</td>
                    <td class="average">29.8053</td>
                    <td>68.0067</td>
                    <td>8.2466</td>
                </tr>
            
                <tr>
                    <td>002544</td>
                    <td>pair_002544/stitched.png</td>
                    <td>002544.jpg</td>
                    <td class="good">0.9032</td>
                    <td class="average">26.2351</td>
                    <td>154.7297</td>
                    <td>12.4390</td>
                </tr>
            
                <tr>
                    <td>002588</td>
                    <td>pair_002588/stitched.png</td>
                    <td>002588.jpg</td>
                    <td class="average">0.8564</td>
                    <td class="average">23.7499</td>
                    <td>274.2116</td>
                    <td>16.5593</td>
                </tr>
            
                <tr>
                    <td>002589</td>
                    <td>pair_002589/stitched.png</td>
                    <td>002589.jpg</td>
                    <td class="poor">0.7038</td>
                    <td class="average">20.5056</td>
                    <td>578.7930</td>
                    <td>24.0581</td>
                </tr>
            
                <tr>
                    <td>002625</td>
                    <td>pair_002625/stitched.png</td>
                    <td>002625.jpg</td>
                    <td class="average">0.8117</td>
                    <td class="average">24.3569</td>
                    <td>238.4460</td>
                    <td>15.4417</td>
                </tr>
            
                <tr>
                    <td>002650</td>
                    <td>pair_002650/stitched.png</td>
                    <td>002650.jpg</td>
                    <td class="good">0.9623</td>
                    <td class="average">27.8691</td>
                    <td>106.2120</td>
                    <td>10.3059</td>
                </tr>
            
                <tr>
                    <td>002675</td>
                    <td>pair_002675/stitched.png</td>
                    <td>002675.jpg</td>
                    <td class="average">0.8553</td>
                    <td class="average">25.6865</td>
                    <td>175.5622</td>
                    <td>13.2500</td>
                </tr>
            
                <tr>
                    <td>002705</td>
                    <td>pair_002705/stitched.png</td>
                    <td>002705.jpg</td>
                    <td class="average">0.7637</td>
                    <td class="poor">18.5108</td>
                    <td>916.2264</td>
                    <td>30.2692</td>
                </tr>
            
                <tr>
                    <td>002741</td>
                    <td>pair_002741/stitched.png</td>
                    <td>002741.jpg</td>
                    <td class="average">0.8138</td>
                    <td class="average">27.1206</td>
                    <td>126.1889</td>
                    <td>11.2334</td>
                </tr>
            
                <tr>
                    <td>002804</td>
                    <td>pair_002804/stitched.png</td>
                    <td>002804.jpg</td>
                    <td class="average">0.8237</td>
                    <td class="average">24.2724</td>
                    <td>243.1291</td>
                    <td>15.5926</td>
                </tr>
            
                <tr>
                    <td>002838</td>
                    <td>pair_002838/stitched.png</td>
                    <td>002838.jpg</td>
                    <td class="good">0.9240</td>
                    <td class="good">31.2155</td>
                    <td>49.1510</td>
                    <td>7.0108</td>
                </tr>
            
                <tr>
                    <td>002851</td>
                    <td>pair_002851/stitched.png</td>
                    <td>002851.jpg</td>
                    <td class="good">0.9480</td>
                    <td class="average">27.8497</td>
                    <td>106.6875</td>
                    <td>10.3290</td>
                </tr>
            
                <tr>
                    <td>002915</td>
                    <td>pair_002915/stitched.png</td>
                    <td>002915.jpg</td>
                    <td class="average">0.7807</td>
                    <td class="average">24.1255</td>
                    <td>251.4966</td>
                    <td>15.8586</td>
                </tr>
            
                <tr>
                    <td>002917</td>
                    <td>pair_002917/stitched.png</td>
                    <td>002917.jpg</td>
                    <td class="average">0.7777</td>
                    <td class="average">22.6153</td>
                    <td>356.0840</td>
                    <td>18.8702</td>
                </tr>
            
                <tr>
                    <td>002936</td>
                    <td>pair_002936/stitched.png</td>
                    <td>002936.jpg</td>
                    <td class="good">0.9484</td>
                    <td class="good">32.9927</td>
                    <td>32.6442</td>
                    <td>5.7135</td>
                </tr>
            
                <tr>
                    <td>003052</td>
                    <td>pair_003052/stitched.png</td>
                    <td>003052.jpg</td>
                    <td class="poor">0.7406</td>
                    <td class="average">23.2042</td>
                    <td>310.9273</td>
                    <td>17.6331</td>
                </tr>
            
                <tr>
                    <td>003053</td>
                    <td>pair_003053/stitched.png</td>
                    <td>003053.jpg</td>
                    <td class="average">0.8082</td>
                    <td class="average">24.6644</td>
                    <td>222.1494</td>
                    <td>14.9047</td>
                </tr>
            
                <tr>
                    <td>003055</td>
                    <td>pair_003055/stitched.png</td>
                    <td>003055.jpg</td>
                    <td class="poor">0.7164</td>
                    <td class="average">22.2627</td>
                    <td>386.2023</td>
                    <td>19.6520</td>
                </tr>
            
                <tr>
                    <td>003059</td>
                    <td>pair_003059/stitched.png</td>
                    <td>003059.jpg</td>
                    <td class="average">0.8332</td>
                    <td class="average">24.4153</td>
                    <td>235.2593</td>
                    <td>15.3382</td>
                </tr>
            
                <tr>
                    <td>003109</td>
                    <td>pair_003109/stitched.png</td>
                    <td>003109.jpg</td>
                    <td class="good">0.9363</td>
                    <td class="average">24.9108</td>
                    <td>209.8952</td>
                    <td>14.4878</td>
                </tr>
            
                <tr>
                    <td>003115</td>
                    <td>pair_003115/stitched.png</td>
                    <td>003115.jpg</td>
                    <td class="average">0.8370</td>
                    <td class="average">27.2415</td>
                    <td>122.7248</td>
                    <td>11.0781</td>
                </tr>
            
                <tr>
                    <td>003117</td>
                    <td>pair_003117/stitched.png</td>
                    <td>003117.jpg</td>
                    <td class="average">0.8455</td>
                    <td class="average">27.2703</td>
                    <td>121.9126</td>
                    <td>11.0414</td>
                </tr>
            
                <tr>
                    <td>003156</td>
                    <td>pair_003156/stitched.png</td>
                    <td>003156.jpg</td>
                    <td class="good">0.9485</td>
                    <td class="average">28.5271</td>
                    <td>91.2781</td>
                    <td>9.5540</td>
                </tr>
            
                <tr>
                    <td>003245</td>
                    <td>pair_003245/stitched.png</td>
                    <td>003245.jpg</td>
                    <td class="good">0.9345</td>
                    <td class="good">31.5767</td>
                    <td>45.2281</td>
                    <td>6.7252</td>
                </tr>
            
                <tr>
                    <td>003327</td>
                    <td>pair_003327/stitched.png</td>
                    <td>003327.jpg</td>
                    <td class="average">0.8076</td>
                    <td class="average">26.4593</td>
                    <td>146.9433</td>
                    <td>12.1220</td>
                </tr>
            
                <tr>
                    <td>003345</td>
                    <td>pair_003345/stitched.png</td>
                    <td>003345.jpg</td>
                    <td class="good">0.9612</td>
                    <td class="good">33.2320</td>
                    <td>30.8946</td>
                    <td>5.5583</td>
                </tr>
            
                <tr>
                    <td>003427</td>
                    <td>pair_003427/stitched.png</td>
                    <td>003427.jpg</td>
                    <td class="good">0.9172</td>
                    <td class="average">28.8335</td>
                    <td>85.0614</td>
                    <td>9.2229</td>
                </tr>
            
                <tr>
                    <td>003435</td>
                    <td>pair_003435/stitched.png</td>
                    <td>003435.jpg</td>
                    <td class="average">0.7591</td>
                    <td class="average">22.9819</td>
                    <td>327.2573</td>
                    <td>18.0903</td>
                </tr>
            
                <tr>
                    <td>003449</td>
                    <td>pair_003449/stitched.png</td>
                    <td>003449.jpg</td>
                    <td class="good">0.9298</td>
                    <td class="average">26.2420</td>
                    <td>154.4812</td>
                    <td>12.4290</td>
                </tr>
            
                <tr>
                    <td>003454</td>
                    <td>pair_003454/stitched.png</td>
                    <td>003454.jpg</td>
                    <td class="good">0.9510</td>
                    <td class="good">30.7087</td>
                    <td>55.2340</td>
                    <td>7.4320</td>
                </tr>
            
                <tr>
                    <td>003484</td>
                    <td>pair_003484/stitched.png</td>
                    <td>003484.jpg</td>
                    <td class="good">0.9584</td>
                    <td class="good">30.1006</td>
                    <td>63.5359</td>
                    <td>7.9709</td>
                </tr>
            
                <tr>
                    <td>003491</td>
                    <td>pair_003491/stitched.png</td>
                    <td>003491.jpg</td>
                    <td class="good">0.9226</td>
                    <td class="average">29.4427</td>
                    <td>73.9288</td>
                    <td>8.5982</td>
                </tr>
            
                <tr>
                    <td>003507</td>
                    <td>pair_003507/stitched.png</td>
                    <td>003507.jpg</td>
                    <td class="average">0.8472</td>
                    <td class="average">27.4352</td>
                    <td>117.3705</td>
                    <td>10.8338</td>
                </tr>
            
                <tr>
                    <td>003520</td>
                    <td>pair_003520/stitched.png</td>
                    <td>003520.jpg</td>
                    <td class="average">0.7643</td>
                    <td class="poor">16.4078</td>
                    <td>1486.9776</td>
                    <td>38.5613</td>
                </tr>
            
                <tr>
                    <td>003521</td>
                    <td>pair_003521/stitched.png</td>
                    <td>003521.jpg</td>
                    <td class="good">0.9539</td>
                    <td class="average">24.4604</td>
                    <td>232.8322</td>
                    <td>15.2588</td>
                </tr>
            
                <tr>
                    <td>003522</td>
                    <td>pair_003522/stitched.png</td>
                    <td>003522.jpg</td>
                    <td class="average">0.7645</td>
                    <td class="poor">16.7448</td>
                    <td>1375.9288</td>
                    <td>37.0935</td>
                </tr>
            
                <tr>
                    <td>003528</td>
                    <td>pair_003528/stitched.png</td>
                    <td>003528.jpg</td>
                    <td class="average">0.8363</td>
                    <td class="average">25.4951</td>
                    <td>183.4743</td>
                    <td>13.5453</td>
                </tr>
            
                <tr>
                    <td>003529</td>
                    <td>pair_003529/stitched.png</td>
                    <td>003529.jpg</td>
                    <td class="good">0.9375</td>
                    <td class="average">29.8187</td>
                    <td>67.7968</td>
                    <td>8.2339</td>
                </tr>
            
                <tr>
                    <td>003570</td>
                    <td>pair_003570/stitched.png</td>
                    <td>003570.jpg</td>
                    <td class="good">0.9295</td>
                    <td class="average">28.1014</td>
                    <td>100.6802</td>
                    <td>10.0340</td>
                </tr>
            
                <tr>
                    <td>003589</td>
                    <td>pair_003589/stitched.png</td>
                    <td>003589.jpg</td>
                    <td class="good">0.9388</td>
                    <td class="average">29.1890</td>
                    <td>78.3748</td>
                    <td>8.8530</td>
                </tr>
            
                <tr>
                    <td>003686</td>
                    <td>pair_003686/stitched.png</td>
                    <td>003686.jpg</td>
                    <td class="average">0.8546</td>
                    <td class="average">25.7965</td>
                    <td>171.1697</td>
                    <td>13.0832</td>
                </tr>
            
                <tr>
                    <td>003755</td>
                    <td>pair_003755/stitched.png</td>
                    <td>003755.jpg</td>
                    <td class="average">0.7581</td>
                    <td class="average">24.1231</td>
                    <td>251.6357</td>
                    <td>15.8630</td>
                </tr>
            
                <tr>
                    <td>000009</td>
                    <td>pair_000009/stitched.png</td>
                    <td>000009.jpg</td>
                    <td class="good">0.9767</td>
                    <td class="good">38.9188</td>
                    <td>8.3406</td>
                    <td>2.8880</td>
                </tr>
            
                <tr>
                    <td>000010</td>
                    <td>pair_000010/stitched.png</td>
                    <td>000010.jpg</td>
                    <td class="good">0.9215</td>
                    <td class="good">34.1931</td>
                    <td>24.7614</td>
                    <td>4.9761</td>
                </tr>
            
                <tr>
                    <td>000022</td>
                    <td>pair_000022/stitched.png</td>
                    <td>000022.jpg</td>
                    <td class="good">0.9938</td>
                    <td class="good">42.9868</td>
                    <td>3.2689</td>
                    <td>1.8080</td>
                </tr>
            
                <tr>
                    <td>000028</td>
                    <td>pair_000028/stitched.png</td>
                    <td>000028.jpg</td>
                    <td class="good">0.9472</td>
                    <td class="good">31.6382</td>
                    <td>44.5922</td>
                    <td>6.6777</td>
                </tr>
            
                <tr>
                    <td>000032</td>
                    <td>pair_000032/stitched.png</td>
                    <td>000032.jpg</td>
                    <td class="good">0.9413</td>
                    <td class="good">33.5365</td>
                    <td>28.8024</td>
                    <td>5.3668</td>
                </tr>
            
                <tr>
                    <td>000035</td>
                    <td>pair_000035/stitched.png</td>
                    <td>000035.jpg</td>
                    <td class="good">0.9438</td>
                    <td class="average">29.1203</td>
                    <td>79.6243</td>
                    <td>8.9232</td>
                </tr>
            
                <tr>
                    <td>000036</td>
                    <td>pair_000036/stitched.png</td>
                    <td>000036.jpg</td>
                    <td class="good">0.9329</td>
                    <td class="average">28.3375</td>
                    <td>95.3518</td>
                    <td>9.7648</td>
                </tr>
            
                <tr>
                    <td>000041</td>
                    <td>pair_000041/stitched.png</td>
                    <td>000041.jpg</td>
                    <td class="good">0.9583</td>
                    <td class="good">32.3612</td>
                    <td>37.7537</td>
                    <td>6.1444</td>
                </tr>
            
                <tr>
                    <td>000042</td>
                    <td>pair_000042/stitched.png</td>
                    <td>000042.jpg</td>
                    <td class="good">0.9973</td>
                    <td class="good">45.2418</td>
                    <td>1.9449</td>
                    <td>1.3946</td>
                </tr>
            
                <tr>
                    <td>000053</td>
                    <td>pair_000053/stitched.png</td>
                    <td>000053.jpg</td>
                    <td class="good">0.9081</td>
                    <td class="average">28.5321</td>
                    <td>91.1737</td>
                    <td>9.5485</td>
                </tr>
            
                <tr>
                    <td>000072</td>
                    <td>pair_000072/stitched.png</td>
                    <td>000072.jpg</td>
                    <td class="good">0.9011</td>
                    <td class="good">30.2247</td>
                    <td>61.7460</td>
                    <td>7.8579</td>
                </tr>
            
                <tr>
                    <td>000109</td>
                    <td>pair_000109/stitched.png</td>
                    <td>000109.jpg</td>
                    <td class="good">0.9674</td>
                    <td class="good">36.3584</td>
                    <td>15.0397</td>
                    <td>3.8781</td>
                </tr>
            
                <tr>
                    <td>000112</td>
                    <td>pair_000112/stitched.png</td>
                    <td>000112.jpg</td>
                    <td class="good">0.9856</td>
                    <td class="good">38.6897</td>
                    <td>8.7924</td>
                    <td>2.9652</td>
                </tr>
            
                <tr>
                    <td>000113</td>
                    <td>pair_000113/stitched.png</td>
                    <td>000113.jpg</td>
                    <td class="good">0.9529</td>
                    <td class="good">32.2333</td>
                    <td>38.8824</td>
                    <td>6.2356</td>
                </tr>
            
                <tr>
                    <td>000125</td>
                    <td>pair_000125/stitched.png</td>
                    <td>000125.jpg</td>
                    <td class="good">0.9491</td>
                    <td class="good">32.7086</td>
                    <td>34.8511</td>
                    <td>5.9035</td>
                </tr>
            
                <tr>
                    <td>000136</td>
                    <td>pair_000136/stitched.png</td>
                    <td>000136.jpg</td>
                    <td class="good">0.9026</td>
                    <td class="average">28.6058</td>
                    <td>89.6406</td>
                    <td>9.4679</td>
                </tr>
            
                <tr>
                    <td>000158</td>
                    <td>pair_000158/stitched.png</td>
                    <td>000158.jpg</td>
                    <td class="good">0.9241</td>
                    <td class="average">27.8660</td>
                    <td>106.2875</td>
                    <td>10.3096</td>
                </tr>
            
                <tr>
                    <td>000159</td>
                    <td>pair_000159/stitched.png</td>
                    <td>000159.jpg</td>
                    <td class="good">0.9172</td>
                    <td class="average">27.9167</td>
                    <td>105.0523</td>
                    <td>10.2495</td>
                </tr>
            
                <tr>
                    <td>000161</td>
                    <td>pair_000161/stitched.png</td>
                    <td>000161.jpg</td>
                    <td class="good">0.9970</td>
                    <td class="good">45.2104</td>
                    <td>1.9590</td>
                    <td>1.3997</td>
                </tr>
            
                <tr>
                    <td>000177</td>
                    <td>pair_000177/stitched.png</td>
                    <td>000177.jpg</td>
                    <td class="good">0.9337</td>
                    <td class="good">33.7872</td>
                    <td>27.1869</td>
                    <td>5.2141</td>
                </tr>
            
                <tr>
                    <td>000179</td>
                    <td>pair_000179/stitched.png</td>
                    <td>000179.jpg</td>
                    <td class="average">0.8552</td>
                    <td class="average">25.9654</td>
                    <td>164.6428</td>
                    <td>12.8313</td>
                </tr>
            
                <tr>
                    <td>000184</td>
                    <td>pair_000184/stitched.png</td>
                    <td>000184.jpg</td>
                    <td class="good">0.9638</td>
                    <td class="good">33.6345</td>
                    <td>28.1599</td>
                    <td>5.3066</td>
                </tr>
            
                <tr>
                    <td>000190</td>
                    <td>pair_000190/stitched.png</td>
                    <td>000190.jpg</td>
                    <td class="good">0.9861</td>
                    <td class="average">28.1925</td>
                    <td>98.5891</td>
                    <td>9.9292</td>
                </tr>
            
                <tr>
                    <td>000191</td>
                    <td>pair_000191/stitched.png</td>
                    <td>000191.jpg</td>
                    <td class="good">0.9352</td>
                    <td class="average">26.1364</td>
                    <td>158.2847</td>
                    <td>12.5811</td>
                </tr>
            
                <tr>
                    <td>000192</td>
                    <td>pair_000192/stitched.png</td>
                    <td>000192.jpg</td>
                    <td class="average">0.8490</td>
                    <td class="average">22.8760</td>
                    <td>335.3377</td>
                    <td>18.3122</td>
                </tr>
            
                <tr>
                    <td>000193</td>
                    <td>pair_000193/stitched.png</td>
                    <td>000193.jpg</td>
                    <td class="good">0.9050</td>
                    <td class="good">31.0029</td>
                    <td>51.6164</td>
                    <td>7.1845</td>
                </tr>
            
                <tr>
                    <td>000199</td>
                    <td>pair_000199/stitched.png</td>
                    <td>000199.jpg</td>
                    <td class="good">0.9214</td>
                    <td class="good">31.1306</td>
                    <td>50.1214</td>
                    <td>7.0797</td>
                </tr>
            
                <tr>
                    <td>000222</td>
                    <td>pair_000222/stitched.png</td>
                    <td>000222.jpg</td>
                    <td class="good">0.9218</td>
                    <td class="good">32.1888</td>
                    <td>39.2829</td>
                    <td>6.2676</td>
                </tr>
            
                <tr>
                    <td>000226</td>
                    <td>pair_000226/stitched.png</td>
                    <td>000226.jpg</td>
                    <td class="average">0.8877</td>
                    <td class="average">27.3788</td>
                    <td>118.9050</td>
                    <td>10.9044</td>
                </tr>
            
                <tr>
                    <td>000228</td>
                    <td>pair_000228/stitched.png</td>
                    <td>000228.jpg</td>
                    <td class="good">0.9550</td>
                    <td class="good">30.7967</td>
                    <td>54.1264</td>
                    <td>7.3571</td>
                </tr>
            
                <tr>
                    <td>000229</td>
                    <td>pair_000229/stitched.png</td>
                    <td>000229.jpg</td>
                    <td class="good">0.9191</td>
                    <td class="average">25.3358</td>
                    <td>190.3254</td>
                    <td>13.7958</td>
                </tr>
            
                <tr>
                    <td>000241</td>
                    <td>pair_000241/stitched.png</td>
                    <td>000241.jpg</td>
                    <td class="good">0.9544</td>
                    <td class="good">30.9259</td>
                    <td>52.5400</td>
                    <td>7.2484</td>
                </tr>
            
                <tr>
                    <td>000254</td>
                    <td>pair_000254/stitched.png</td>
                    <td>000254.jpg</td>
                    <td class="good">0.9466</td>
                    <td class="good">34.8843</td>
                    <td>21.1179</td>
                    <td>4.5954</td>
                </tr>
            
                <tr>
                    <td>000256</td>
                    <td>pair_000256/stitched.png</td>
                    <td>000256.jpg</td>
                    <td class="good">0.9258</td>
                    <td class="good">30.5009</td>
                    <td>57.9415</td>
                    <td>7.6119</td>
                </tr>
            
                <tr>
                    <td>000257</td>
                    <td>pair_000257/stitched.png</td>
                    <td>000257.jpg</td>
                    <td class="good">0.9374</td>
                    <td class="good">32.3111</td>
                    <td>38.1918</td>
                    <td>6.1800</td>
                </tr>
            
                <tr>
                    <td>000260</td>
                    <td>pair_000260/stitched.png</td>
                    <td>000260.jpg</td>
                    <td class="good">0.9938</td>
                    <td class="good">42.2795</td>
                    <td>3.8471</td>
                    <td>1.9614</td>
                </tr>
            
                <tr>
                    <td>000263</td>
                    <td>pair_000263/stitched.png</td>
                    <td>000263.jpg</td>
                    <td class="good">0.9866</td>
                    <td class="good">34.5372</td>
                    <td>22.8750</td>
                    <td>4.7828</td>
                </tr>
            
                <tr>
                    <td>000282</td>
                    <td>pair_000282/stitched.png</td>
                    <td>000282.jpg</td>
                    <td class="good">0.9890</td>
                    <td class="good">43.3022</td>
                    <td>3.0399</td>
                    <td>1.7435</td>
                </tr>
            
                <tr>
                    <td>000283</td>
                    <td>pair_000283/stitched.png</td>
                    <td>000283.jpg</td>
                    <td class="average">0.7796</td>
                    <td class="average">24.1697</td>
                    <td>248.9475</td>
                    <td>15.7781</td>
                </tr>
            
                <tr>
                    <td>000285</td>
                    <td>pair_000285/stitched.png</td>
                    <td>000285.jpg</td>
                    <td class="good">0.9837</td>
                    <td class="good">37.1909</td>
                    <td>12.4161</td>
                    <td>3.5237</td>
                </tr>
            
                <tr>
                    <td>000304</td>
                    <td>pair_000304/stitched.png</td>
                    <td>000304.jpg</td>
                    <td class="good">0.9448</td>
                    <td class="average">29.6322</td>
                    <td>70.7725</td>
                    <td>8.4126</td>
                </tr>
            
                <tr>
                    <td>000313</td>
                    <td>pair_000313/stitched.png</td>
                    <td>000313.jpg</td>
                    <td class="good">0.9579</td>
                    <td class="good">34.8189</td>
                    <td>21.4381</td>
                    <td>4.6301</td>
                </tr>
            
                <tr>
                    <td>000320</td>
                    <td>pair_000320/stitched.png</td>
                    <td>000320.jpg</td>
                    <td class="average">0.8891</td>
                    <td class="average">28.2517</td>
                    <td>97.2545</td>
                    <td>9.8618</td>
                </tr>
            
                <tr>
                    <td>000326</td>
                    <td>pair_000326/stitched.png</td>
                    <td>000326.jpg</td>
                    <td class="good">0.9299</td>
                    <td class="good">30.5218</td>
                    <td>57.6628</td>
                    <td>7.5936</td>
                </tr>
            
                <tr>
                    <td>000337</td>
                    <td>pair_000337/stitched.png</td>
                    <td>000337.jpg</td>
                    <td class="average">0.8662</td>
                    <td class="average">25.7515</td>
                    <td>172.9539</td>
                    <td>13.1512</td>
                </tr>
            
                <tr>
                    <td>000338</td>
                    <td>pair_000338/stitched.png</td>
                    <td>000338.jpg</td>
                    <td class="average">0.8346</td>
                    <td class="average">27.3125</td>
                    <td>120.7350</td>
                    <td>10.9879</td>
                </tr>
            
                <tr>
                    <td>000339</td>
                    <td>pair_000339/stitched.png</td>
                    <td>000339.jpg</td>
                    <td class="good">0.9769</td>
                    <td class="good">36.2156</td>
                    <td>15.5423</td>
                    <td>3.9424</td>
                </tr>
            
                <tr>
                    <td>000363</td>
                    <td>pair_000363/stitched.png</td>
                    <td>000363.jpg</td>
                    <td class="good">0.9972</td>
                    <td class="good">47.9018</td>
                    <td>1.0541</td>
                    <td>1.0267</td>
                </tr>
            
                <tr>
                    <td>000366</td>
                    <td>pair_000366/stitched.png</td>
                    <td>000366.jpg</td>
                    <td class="good">0.9144</td>
                    <td class="good">33.1093</td>
                    <td>31.7796</td>
                    <td>5.6373</td>
                </tr>
            
                <tr>
                    <td>000371</td>
                    <td>pair_000371/stitched.png</td>
                    <td>000371.jpg</td>
                    <td class="good">0.9607</td>
                    <td class="good">30.6480</td>
                    <td>56.0122</td>
                    <td>7.4841</td>
                </tr>
            
                <tr>
                    <td>000375</td>
                    <td>pair_000375/stitched.png</td>
                    <td>000375.jpg</td>
                    <td class="average">0.8891</td>
                    <td class="average">25.9335</td>
                    <td>165.8549</td>
                    <td>12.8785</td>
                </tr>
            
                <tr>
                    <td>000392</td>
                    <td>pair_000392/stitched.png</td>
                    <td>000392.jpg</td>
                    <td class="good">0.9399</td>
                    <td class="good">31.5870</td>
                    <td>45.1209</td>
                    <td>6.7172</td>
                </tr>
            
                <tr>
                    <td>000393</td>
                    <td>pair_000393/stitched.png</td>
                    <td>000393.jpg</td>
                    <td class="good">0.9905</td>
                    <td class="good">36.2385</td>
                    <td>15.4607</td>
                    <td>3.9320</td>
                </tr>
            
                <tr>
                    <td>000394</td>
                    <td>pair_000394/stitched.png</td>
                    <td>000394.jpg</td>
                    <td class="good">0.9767</td>
                    <td class="good">33.7503</td>
                    <td>27.4191</td>
                    <td>5.2363</td>
                </tr>
            
                <tr>
                    <td>000396</td>
                    <td>pair_000396/stitched.png</td>
                    <td>000396.jpg</td>
                    <td class="good">0.9697</td>
                    <td class="good">34.1805</td>
                    <td>24.8333</td>
                    <td>4.9833</td>
                </tr>
            
                <tr>
                    <td>000423</td>
                    <td>pair_000423/stitched.png</td>
                    <td>000423.jpg</td>
                    <td class="good">0.9413</td>
                    <td class="good">30.2109</td>
                    <td>61.9428</td>
                    <td>7.8704</td>
                </tr>
            
                <tr>
                    <td>000443</td>
                    <td>pair_000443/stitched.png</td>
                    <td>000443.jpg</td>
                    <td class="good">0.9545</td>
                    <td class="good">33.2549</td>
                    <td>30.7323</td>
                    <td>5.5437</td>
                </tr>
            
                <tr>
                    <td>000444</td>
                    <td>pair_000444/stitched.png</td>
                    <td>000444.jpg</td>
                    <td class="good">0.9559</td>
                    <td class="good">33.2520</td>
                    <td>30.7525</td>
                    <td>5.5455</td>
                </tr>
            
                <tr>
                    <td>000459</td>
                    <td>pair_000459/stitched.png</td>
                    <td>000459.jpg</td>
                    <td class="good">0.9877</td>
                    <td class="good">39.0511</td>
                    <td>8.0904</td>
                    <td>2.8444</td>
                </tr>
            
                <tr>
                    <td>000461</td>
                    <td>pair_000461/stitched.png</td>
                    <td>000461.jpg</td>
                    <td class="good">0.9555</td>
                    <td class="average">24.5766</td>
                    <td>226.6854</td>
                    <td>15.0561</td>
                </tr>
            
                <tr>
                    <td>000465</td>
                    <td>pair_000465/stitched.png</td>
                    <td>000465.jpg</td>
                    <td class="good">0.9816</td>
                    <td class="good">37.7217</td>
                    <td>10.9879</td>
                    <td>3.3148</td>
                </tr>
            
                <tr>
                    <td>000479</td>
                    <td>pair_000479/stitched.png</td>
                    <td>000479.jpg</td>
                    <td class="good">0.9777</td>
                    <td class="good">34.7501</td>
                    <td>21.7806</td>
                    <td>4.6670</td>
                </tr>
            
                <tr>
                    <td>000486</td>
                    <td>pair_000486/stitched.png</td>
                    <td>000486.jpg</td>
                    <td class="average">0.8512</td>
                    <td class="average">24.9436</td>
                    <td>208.3128</td>
                    <td>14.4330</td>
                </tr>
            
                <tr>
                    <td>000496</td>
                    <td>pair_000496/stitched.png</td>
                    <td>000496.jpg</td>
                    <td class="good">0.9024</td>
                    <td class="good">30.2361</td>
                    <td>61.5845</td>
                    <td>7.8476</td>
                </tr>
            
                <tr>
                    <td>000497</td>
                    <td>pair_000497/stitched.png</td>
                    <td>000497.jpg</td>
                    <td class="good">0.9587</td>
                    <td class="good">34.0401</td>
                    <td>25.6489</td>
                    <td>5.0645</td>
                </tr>
            
                <tr>
                    <td>000498</td>
                    <td>pair_000498/stitched.png</td>
                    <td>000498.jpg</td>
                    <td class="good">0.9033</td>
                    <td class="good">30.6558</td>
                    <td>55.9112</td>
                    <td>7.4774</td>
                </tr>
            
                <tr>
                    <td>000506</td>
                    <td>pair_000506/stitched.png</td>
                    <td>000506.jpg</td>
                    <td class="good">0.9757</td>
                    <td class="good">36.3052</td>
                    <td>15.2252</td>
                    <td>3.9019</td>
                </tr>
            
                <tr>
                    <td>000507</td>
                    <td>pair_000507/stitched.png</td>
                    <td>000507.jpg</td>
                    <td class="good">0.9675</td>
                    <td class="good">32.5709</td>
                    <td>35.9745</td>
                    <td>5.9979</td>
                </tr>
            
                <tr>
                    <td>000516</td>
                    <td>pair_000516/stitched.png</td>
                    <td>000516.jpg</td>
                    <td class="good">0.9811</td>
                    <td class="good">37.7921</td>
                    <td>10.8110</td>
                    <td>3.2880</td>
                </tr>
            
                <tr>
                    <td>000518</td>
                    <td>pair_000518/stitched.png</td>
                    <td>000518.jpg</td>
                    <td class="good">0.9748</td>
                    <td class="average">29.8082</td>
                    <td>67.9605</td>
                    <td>8.2438</td>
                </tr>
            
                <tr>
                    <td>000519</td>
                    <td>pair_000519/stitched.png</td>
                    <td>000519.jpg</td>
                    <td class="good">0.9742</td>
                    <td class="average">29.0902</td>
                    <td>80.1796</td>
                    <td>8.9543</td>
                </tr>
            
                <tr>
                    <td>000522</td>
                    <td>pair_000522/stitched.png</td>
                    <td>000522.jpg</td>
                    <td class="good">0.9872</td>
                    <td class="good">43.8196</td>
                    <td>2.6985</td>
                    <td>1.6427</td>
                </tr>
            
                <tr>
                    <td>000526</td>
                    <td>pair_000526/stitched.png</td>
                    <td>000526.jpg</td>
                    <td class="good">0.9754</td>
                    <td class="good">35.1403</td>
                    <td>19.9093</td>
                    <td>4.4620</td>
                </tr>
            
                <tr>
                    <td>000527</td>
                    <td>pair_000527/stitched.png</td>
                    <td>000527.jpg</td>
                    <td class="good">0.9964</td>
                    <td class="good">42.9154</td>
                    <td>3.3231</td>
                    <td>1.8229</td>
                </tr>
            
                <tr>
                    <td>000533</td>
                    <td>pair_000533/stitched.png</td>
                    <td>000533.jpg</td>
                    <td class="good">0.9284</td>
                    <td class="good">30.1983</td>
                    <td>62.1226</td>
                    <td>7.8818</td>
                </tr>
            
                <tr>
                    <td>000535</td>
                    <td>pair_000535/stitched.png</td>
                    <td>000535.jpg</td>
                    <td class="average">0.8836</td>
                    <td class="average">28.0422</td>
                    <td>102.0605</td>
                    <td>10.1025</td>
                </tr>
            
                <tr>
                    <td>000537</td>
                    <td>pair_000537/stitched.png</td>
                    <td>000537.jpg</td>
                    <td class="good">0.9226</td>
                    <td class="good">30.1041</td>
                    <td>63.4856</td>
                    <td>7.9678</td>
                </tr>
            
                <tr>
                    <td>000541</td>
                    <td>pair_000541/stitched.png</td>
                    <td>000541.jpg</td>
                    <td class="good">0.9533</td>
                    <td class="good">36.3414</td>
                    <td>15.0986</td>
                    <td>3.8857</td>
                </tr>
            
                <tr>
                    <td>000550</td>
                    <td>pair_000550/stitched.png</td>
                    <td>000550.jpg</td>
                    <td class="good">0.9674</td>
                    <td class="good">34.7232</td>
                    <td>21.9161</td>
                    <td>4.6815</td>
                </tr>
            
                <tr>
                    <td>000589</td>
                    <td>pair_000589/stitched.png</td>
                    <td>000589.jpg</td>
                    <td class="good">0.9933</td>
                    <td class="good">39.5874</td>
                    <td>7.1506</td>
                    <td>2.6741</td>
                </tr>
            
                <tr>
                    <td>000590</td>
                    <td>pair_000590/stitched.png</td>
                    <td>000590.jpg</td>
                    <td class="good">0.9925</td>
                    <td class="good">40.9468</td>
                    <td>5.2288</td>
                    <td>2.2867</td>
                </tr>
            
                <tr>
                    <td>000607</td>
                    <td>pair_000607/stitched.png</td>
                    <td>000607.jpg</td>
                    <td class="average">0.8495</td>
                    <td class="average">23.2952</td>
                    <td>304.4795</td>
                    <td>17.4493</td>
                </tr>
            
                <tr>
                    <td>000609</td>
                    <td>pair_000609/stitched.png</td>
                    <td>000609.jpg</td>
                    <td class="average">0.7612</td>
                    <td class="average">21.5231</td>
                    <td>457.8990</td>
                    <td>21.3986</td>
                </tr>
            
                <tr>
                    <td>000627</td>
                    <td>pair_000627/stitched.png</td>
                    <td>000627.jpg</td>
                    <td class="good">0.9675</td>
                    <td class="good">33.3646</td>
                    <td>29.9653</td>
                    <td>5.4741</td>
                </tr>
            
                <tr>
                    <td>000630</td>
                    <td>pair_000630/stitched.png</td>
                    <td>000630.jpg</td>
                    <td class="good">0.9545</td>
                    <td class="average">29.7732</td>
                    <td>68.5104</td>
                    <td>8.2771</td>
                </tr>
            
                <tr>
                    <td>000631</td>
                    <td>pair_000631/stitched.png</td>
                    <td>000631.jpg</td>
                    <td class="good">0.9435</td>
                    <td class="good">33.2655</td>
                    <td>30.6570</td>
                    <td>5.5369</td>
                </tr>
            
                <tr>
                    <td>000633</td>
                    <td>pair_000633/stitched.png</td>
                    <td>000633.jpg</td>
                    <td class="good">0.9686</td>
                    <td class="good">36.6871</td>
                    <td>13.9434</td>
                    <td>3.7341</td>
                </tr>
            
                <tr>
                    <td>000638</td>
                    <td>pair_000638/stitched.png</td>
                    <td>000638.jpg</td>
                    <td class="good">0.9411</td>
                    <td class="good">32.5875</td>
                    <td>35.8370</td>
                    <td>5.9864</td>
                </tr>
            
                <tr>
                    <td>000644</td>
                    <td>pair_000644/stitched.png</td>
                    <td>000644.jpg</td>
                    <td class="good">0.9762</td>
                    <td class="good">40.5584</td>
                    <td>5.7180</td>
                    <td>2.3912</td>
                </tr>
            
                <tr>
                    <td>000648</td>
                    <td>pair_000648/stitched.png</td>
                    <td>000648.jpg</td>
                    <td class="good">0.9718</td>
                    <td class="good">33.8101</td>
                    <td>27.0439</td>
                    <td>5.2004</td>
                </tr>
            
                <tr>
                    <td>000659</td>
                    <td>pair_000659/stitched.png</td>
                    <td>000659.jpg</td>
                    <td class="good">0.9891</td>
                    <td class="good">39.6780</td>
                    <td>7.0030</td>
                    <td>2.6463</td>
                </tr>
            
                <tr>
                    <td>000661</td>
                    <td>pair_000661/stitched.png</td>
                    <td>000661.jpg</td>
                    <td class="good">0.9023</td>
                    <td class="average">29.4053</td>
                    <td>74.5677</td>
                    <td>8.6353</td>
                </tr>
            
                <tr>
                    <td>000685</td>
                    <td>pair_000685/stitched.png</td>
                    <td>000685.jpg</td>
                    <td class="good">0.9334</td>
                    <td class="average">24.8447</td>
                    <td>213.1116</td>
                    <td>14.5983</td>
                </tr>
            
                <tr>
                    <td>000690</td>
                    <td>pair_000690/stitched.png</td>
                    <td>000690.jpg</td>
                    <td class="average">0.8713</td>
                    <td class="average">26.8749</td>
                    <td>133.5350</td>
                    <td>11.5557</td>
                </tr>
            
                <tr>
                    <td>000704</td>
                    <td>pair_000704/stitched.png</td>
                    <td>000704.jpg</td>
                    <td class="good">0.9624</td>
                    <td class="good">34.0989</td>
                    <td>25.3038</td>
                    <td>5.0303</td>
                </tr>
            
                <tr>
                    <td>000727</td>
                    <td>pair_000727/stitched.png</td>
                    <td>000727.jpg</td>
                    <td class="poor">0.6682</td>
                    <td class="average">25.0998</td>
                    <td>200.9579</td>
                    <td>14.1760</td>
                </tr>
            
                <tr>
                    <td>000728</td>
                    <td>pair_000728/stitched.png</td>
                    <td>000728.jpg</td>
                    <td class="good">0.9115</td>
                    <td class="good">30.5128</td>
                    <td>57.7837</td>
                    <td>7.6016</td>
                </tr>
            
                <tr>
                    <td>000733</td>
                    <td>pair_000733/stitched.png</td>
                    <td>000733.jpg</td>
                    <td class="good">0.9007</td>
                    <td class="good">31.5762</td>
                    <td>45.2339</td>
                    <td>6.7256</td>
                </tr>
            
                <tr>
                    <td>000743</td>
                    <td>pair_000743/stitched.png</td>
                    <td>000743.jpg</td>
                    <td class="good">0.9002</td>
                    <td class="good">33.8371</td>
                    <td>26.8765</td>
                    <td>5.1843</td>
                </tr>
            
                <tr>
                    <td>000750</td>
                    <td>pair_000750/stitched.png</td>
                    <td>000750.jpg</td>
                    <td class="good">0.9674</td>
                    <td class="good">36.3345</td>
                    <td>15.1227</td>
                    <td>3.8888</td>
                </tr>
            
                <tr>
                    <td>000778</td>
                    <td>pair_000778/stitched.png</td>
                    <td>000778.jpg</td>
                    <td class="good">0.9804</td>
                    <td class="good">36.0931</td>
                    <td>15.9870</td>
                    <td>3.9984</td>
                </tr>
            
                <tr>
                    <td>000788</td>
                    <td>pair_000788/stitched.png</td>
                    <td>000788.jpg</td>
                    <td class="average">0.8159</td>
                    <td class="average">25.1792</td>
                    <td>197.3134</td>
                    <td>14.0468</td>
                </tr>
            
                <tr>
                    <td>000791</td>
                    <td>pair_000791/stitched.png</td>
                    <td>000791.jpg</td>
                    <td class="good">0.9390</td>
                    <td class="average">26.1874</td>
                    <td>156.4357</td>
                    <td>12.5074</td>
                </tr>
            
                <tr>
                    <td>000797</td>
                    <td>pair_000797/stitched.png</td>
                    <td>000797.jpg</td>
                    <td class="good">0.9809</td>
                    <td class="good">32.9115</td>
                    <td>33.2607</td>
                    <td>5.7672</td>
                </tr>
            
                <tr>
                    <td>000803</td>
                    <td>pair_000803/stitched.png</td>
                    <td>000803.jpg</td>
                    <td class="good">0.9672</td>
                    <td class="good">33.6047</td>
                    <td>28.3535</td>
                    <td>5.3248</td>
                </tr>
            
                <tr>
                    <td>000806</td>
                    <td>pair_000806/stitched.png</td>
                    <td>000806.jpg</td>
                    <td class="good">0.9279</td>
                    <td class="good">30.2804</td>
                    <td>60.9594</td>
                    <td>7.8076</td>
                </tr>
            
                <tr>
                    <td>000807</td>
                    <td>pair_000807/stitched.png</td>
                    <td>000807.jpg</td>
                    <td class="good">0.9318</td>
                    <td class="good">30.3656</td>
                    <td>59.7753</td>
                    <td>7.7315</td>
                </tr>
            
                <tr>
                    <td>000813</td>
                    <td>pair_000813/stitched.png</td>
                    <td>000813.jpg</td>
                    <td class="good">0.9595</td>
                    <td class="good">32.3031</td>
                    <td>38.2622</td>
                    <td>6.1856</td>
                </tr>
            
                <tr>
                    <td>000842</td>
                    <td>pair_000842/stitched.png</td>
                    <td>000842.jpg</td>
                    <td class="good">0.9321</td>
                    <td class="good">32.1332</td>
                    <td>39.7891</td>
                    <td>6.3079</td>
                </tr>
            
                <tr>
                    <td>000847</td>
                    <td>pair_000847/stitched.png</td>
                    <td>000847.jpg</td>
                    <td class="good">0.9319</td>
                    <td class="good">32.1258</td>
                    <td>39.8570</td>
                    <td>6.3132</td>
                </tr>
            
                <tr>
                    <td>000879</td>
                    <td>pair_000879/stitched.png</td>
                    <td>000879.jpg</td>
                    <td class="average">0.8986</td>
                    <td class="average">28.5827</td>
                    <td>90.1178</td>
                    <td>9.4930</td>
                </tr>
            
                <tr>
                    <td>000892</td>
                    <td>pair_000892/stitched.png</td>
                    <td>000892.jpg</td>
                    <td class="good">0.9456</td>
                    <td class="good">32.7447</td>
                    <td>34.5628</td>
                    <td>5.8790</td>
                </tr>
            
                <tr>
                    <td>000893</td>
                    <td>pair_000893/stitched.png</td>
                    <td>000893.jpg</td>
                    <td class="good">0.9916</td>
                    <td class="good">42.0475</td>
                    <td>4.0582</td>
                    <td>2.0145</td>
                </tr>
            
                <tr>
                    <td>000894</td>
                    <td>pair_000894/stitched.png</td>
                    <td>000894.jpg</td>
                    <td class="good">0.9054</td>
                    <td class="good">30.2855</td>
                    <td>60.8879</td>
                    <td>7.8031</td>
                </tr>
            
                <tr>
                    <td>000904</td>
                    <td>pair_000904/stitched.png</td>
                    <td>000904.jpg</td>
                    <td class="good">0.9933</td>
                    <td class="good">41.5779</td>
                    <td>4.5216</td>
                    <td>2.1264</td>
                </tr>
            
                <tr>
                    <td>000905</td>
                    <td>pair_000905/stitched.png</td>
                    <td>000905.jpg</td>
                    <td class="good">0.9310</td>
                    <td class="good">32.4908</td>
                    <td>36.6437</td>
                    <td>6.0534</td>
                </tr>
            
                <tr>
                    <td>000908</td>
                    <td>pair_000908/stitched.png</td>
                    <td>000908.jpg</td>
                    <td class="good">0.9898</td>
                    <td class="good">38.2214</td>
                    <td>9.7936</td>
                    <td>3.1295</td>
                </tr>
            
                <tr>
                    <td>000909</td>
                    <td>pair_000909/stitched.png</td>
                    <td>000909.jpg</td>
                    <td class="good">0.9866</td>
                    <td class="good">35.5222</td>
                    <td>18.2332</td>
                    <td>4.2700</td>
                </tr>
            
                <tr>
                    <td>000913</td>
                    <td>pair_000913/stitched.png</td>
                    <td>000913.jpg</td>
                    <td class="average">0.8225</td>
                    <td class="average">29.9316</td>
                    <td>66.0567</td>
                    <td>8.1275</td>
                </tr>
            
                <tr>
                    <td>000914</td>
                    <td>pair_000914/stitched.png</td>
                    <td>000914.jpg</td>
                    <td class="good">0.9320</td>
                    <td class="good">33.7711</td>
                    <td>27.2878</td>
                    <td>5.2238</td>
                </tr>
            
                <tr>
                    <td>000915</td>
                    <td>pair_000915/stitched.png</td>
                    <td>000915.jpg</td>
                    <td class="average">0.8901</td>
                    <td class="good">32.8625</td>
                    <td>33.6383</td>
                    <td>5.7999</td>
                </tr>
            
                <tr>
                    <td>000925</td>
                    <td>pair_000925/stitched.png</td>
                    <td>000925.jpg</td>
                    <td class="good">0.9136</td>
                    <td class="average">24.6446</td>
                    <td>223.1637</td>
                    <td>14.9387</td>
                </tr>
            
                <tr>
                    <td>000927</td>
                    <td>pair_000927/stitched.png</td>
                    <td>000927.jpg</td>
                    <td class="average">0.8910</td>
                    <td class="average">24.2919</td>
                    <td>242.0408</td>
                    <td>15.5577</td>
                </tr>
            
                <tr>
                    <td>000932</td>
                    <td>pair_000932/stitched.png</td>
                    <td>000932.jpg</td>
                    <td class="good">0.9405</td>
                    <td class="good">34.1978</td>
                    <td>24.7342</td>
                    <td>4.9734</td>
                </tr>
            
                <tr>
                    <td>000933</td>
                    <td>pair_000933/stitched.png</td>
                    <td>000933.jpg</td>
                    <td class="good">0.9949</td>
                    <td class="good">44.5382</td>
                    <td>2.2870</td>
                    <td>1.5123</td>
                </tr>
            
                <tr>
                    <td>000946</td>
                    <td>pair_000946/stitched.png</td>
                    <td>000946.jpg</td>
                    <td class="average">0.8810</td>
                    <td class="average">23.3048</td>
                    <td>303.8095</td>
                    <td>17.4301</td>
                </tr>
            
                <tr>
                    <td>000952</td>
                    <td>pair_000952/stitched.png</td>
                    <td>000952.jpg</td>
                    <td class="good">0.9872</td>
                    <td class="good">36.8672</td>
                    <td>13.3770</td>
                    <td>3.6575</td>
                </tr>
            
                <tr>
                    <td>000954</td>
                    <td>pair_000954/stitched.png</td>
                    <td>000954.jpg</td>
                    <td class="good">0.9595</td>
                    <td class="good">33.3155</td>
                    <td>30.3058</td>
                    <td>5.5051</td>
                </tr>
            
                <tr>
                    <td>000961</td>
                    <td>pair_000961/stitched.png</td>
                    <td>000961.jpg</td>
                    <td class="good">0.9313</td>
                    <td class="average">29.4472</td>
                    <td>73.8520</td>
                    <td>8.5937</td>
                </tr>
            
                <tr>
                    <td>000981</td>
                    <td>pair_000981/stitched.png</td>
                    <td>000981.jpg</td>
                    <td class="average">0.8026</td>
                    <td class="average">23.5199</td>
                    <td>289.1298</td>
                    <td>17.0038</td>
                </tr>
            
                <tr>
                    <td>000986</td>
                    <td>pair_000986/stitched.png</td>
                    <td>000986.jpg</td>
                    <td class="average">0.8682</td>
                    <td class="average">24.3736</td>
                    <td>237.5322</td>
                    <td>15.4121</td>
                </tr>
            
                <tr>
                    <td>001000</td>
                    <td>pair_001000/stitched.png</td>
                    <td>001000.jpg</td>
                    <td class="good">0.9381</td>
                    <td class="good">31.4773</td>
                    <td>46.2753</td>
                    <td>6.8026</td>
                </tr>
            
                <tr>
                    <td>001007</td>
                    <td>pair_001007/stitched.png</td>
                    <td>001007.jpg</td>
                    <td class="good">0.9632</td>
                    <td class="good">32.4141</td>
                    <td>37.2971</td>
                    <td>6.1071</td>
                </tr>
            
                <tr>
                    <td>001014</td>
                    <td>pair_001014/stitched.png</td>
                    <td>001014.jpg</td>
                    <td class="good">0.9479</td>
                    <td class="good">32.5379</td>
                    <td>36.2485</td>
                    <td>6.0207</td>
                </tr>
            
                <tr>
                    <td>001015</td>
                    <td>pair_001015/stitched.png</td>
                    <td>001015.jpg</td>
                    <td class="good">0.9036</td>
                    <td class="average">29.8620</td>
                    <td>67.1246</td>
                    <td>8.1930</td>
                </tr>
            
                <tr>
                    <td>001016</td>
                    <td>pair_001016/stitched.png</td>
                    <td>001016.jpg</td>
                    <td class="good">0.9346</td>
                    <td class="good">31.7722</td>
                    <td>43.2373</td>
                    <td>6.5755</td>
                </tr>
            
                <tr>
                    <td>001021</td>
                    <td>pair_001021/stitched.png</td>
                    <td>001021.jpg</td>
                    <td class="good">0.9418</td>
                    <td class="good">34.0009</td>
                    <td>25.8813</td>
                    <td>5.0874</td>
                </tr>
            
                <tr>
                    <td>001035</td>
                    <td>pair_001035/stitched.png</td>
                    <td>001035.jpg</td>
                    <td class="average">0.8833</td>
                    <td class="average">27.6145</td>
                    <td>112.6242</td>
                    <td>10.6125</td>
                </tr>
            
                <tr>
                    <td>001039</td>
                    <td>pair_001039/stitched.png</td>
                    <td>001039.jpg</td>
                    <td class="good">0.9879</td>
                    <td class="good">37.3427</td>
                    <td>11.9897</td>
                    <td>3.4626</td>
                </tr>
            
                <tr>
                    <td>001040</td>
                    <td>pair_001040/stitched.png</td>
                    <td>001040.jpg</td>
                    <td class="good">0.9944</td>
                    <td class="good">43.3013</td>
                    <td>3.0406</td>
                    <td>1.7437</td>
                </tr>
            
                <tr>
                    <td>001045</td>
                    <td>pair_001045/stitched.png</td>
                    <td>001045.jpg</td>
                    <td class="good">0.9952</td>
                    <td class="good">40.8924</td>
                    <td>5.2947</td>
                    <td>2.3010</td>
                </tr>
            
                <tr>
                    <td>001047</td>
                    <td>pair_001047/stitched.png</td>
                    <td>001047.jpg</td>
                    <td class="average">0.8910</td>
                    <td class="average">26.7305</td>
                    <td>138.0466</td>
                    <td>11.7493</td>
                </tr>
            
                <tr>
                    <td>001048</td>
                    <td>pair_001048/stitched.png</td>
                    <td>001048.jpg</td>
                    <td class="good">0.9958</td>
                    <td class="good">41.7137</td>
                    <td>4.3824</td>
                    <td>2.0934</td>
                </tr>
            
                <tr>
                    <td>001055</td>
                    <td>pair_001055/stitched.png</td>
                    <td>001055.jpg</td>
                    <td class="good">0.9500</td>
                    <td class="good">36.7586</td>
                    <td>13.7159</td>
                    <td>3.7035</td>
                </tr>
            
                <tr>
                    <td>001061</td>
                    <td>pair_001061/stitched.png</td>
                    <td>001061.jpg</td>
                    <td class="good">0.9083</td>
                    <td class="average">25.0800</td>
                    <td>201.8749</td>
                    <td>14.2083</td>
                </tr>
            
                <tr>
                    <td>001062</td>
                    <td>pair_001062/stitched.png</td>
                    <td>001062.jpg</td>
                    <td class="good">0.9931</td>
                    <td class="good">36.4427</td>
                    <td>14.7507</td>
                    <td>3.8407</td>
                </tr>
            
                <tr>
                    <td>001065</td>
                    <td>pair_001065/stitched.png</td>
                    <td>001065.jpg</td>
                    <td class="good">0.9735</td>
                    <td class="good">37.0745</td>
                    <td>12.7535</td>
                    <td>3.5712</td>
                </tr>
            
                <tr>
                    <td>001071</td>
                    <td>pair_001071/stitched.png</td>
                    <td>001071.jpg</td>
                    <td class="good">0.9600</td>
                    <td class="average">27.1875</td>
                    <td>124.2586</td>
                    <td>11.1471</td>
                </tr>
            
                <tr>
                    <td>001080</td>
                    <td>pair_001080/stitched.png</td>
                    <td>001080.jpg</td>
                    <td class="good">0.9645</td>
                    <td class="good">32.9329</td>
                    <td>33.0971</td>
                    <td>5.7530</td>
                </tr>
            
                <tr>
                    <td>001082</td>
                    <td>pair_001082/stitched.png</td>
                    <td>001082.jpg</td>
                    <td class="good">0.9898</td>
                    <td class="good">36.1853</td>
                    <td>15.6514</td>
                    <td>3.9562</td>
                </tr>
            
                <tr>
                    <td>001093</td>
                    <td>pair_001093/stitched.png</td>
                    <td>001093.jpg</td>
                    <td class="good">0.9968</td>
                    <td class="good">43.3202</td>
                    <td>3.0273</td>
                    <td>1.7399</td>
                </tr>
            
                <tr>
                    <td>001103</td>
                    <td>pair_001103/stitched.png</td>
                    <td>001103.jpg</td>
                    <td class="good">0.9693</td>
                    <td class="good">34.3551</td>
                    <td>23.8545</td>
                    <td>4.8841</td>
                </tr>
            
                <tr>
                    <td>001155</td>
                    <td>pair_001155/stitched.png</td>
                    <td>001155.jpg</td>
                    <td class="good">0.9609</td>
                    <td class="good">32.4141</td>
                    <td>37.2964</td>
                    <td>6.1071</td>
                </tr>
            
                <tr>
                    <td>001185</td>
                    <td>pair_001185/stitched.png</td>
                    <td>001185.jpg</td>
                    <td class="good">0.9546</td>
                    <td class="good">32.7400</td>
                    <td>34.6004</td>
                    <td>5.8822</td>
                </tr>
            
                <tr>
                    <td>001193</td>
                    <td>pair_001193/stitched.png</td>
                    <td>001193.jpg</td>
                    <td class="good">0.9982</td>
                    <td class="good">47.2911</td>
                    <td>1.2133</td>
                    <td>1.1015</td>
                </tr>
            
                <tr>
                    <td>001195</td>
                    <td>pair_001195/stitched.png</td>
                    <td>001195.jpg</td>
                    <td class="average">0.8270</td>
                    <td class="average">27.0725</td>
                    <td>127.5942</td>
                    <td>11.2958</td>
                </tr>
            
                <tr>
                    <td>001196</td>
                    <td>pair_001196/stitched.png</td>
                    <td>001196.jpg</td>
                    <td class="poor">0.6541</td>
                    <td class="poor">18.4442</td>
                    <td>930.3748</td>
                    <td>30.5020</td>
                </tr>
            
                <tr>
                    <td>001197</td>
                    <td>pair_001197/stitched.png</td>
                    <td>001197.jpg</td>
                    <td class="average">0.8563</td>
                    <td class="average">27.2617</td>
                    <td>122.1556</td>
                    <td>11.0524</td>
                </tr>
            
                <tr>
                    <td>001200</td>
                    <td>pair_001200/stitched.png</td>
                    <td>001200.jpg</td>
                    <td class="good">0.9403</td>
                    <td class="good">34.3464</td>
                    <td>23.9023</td>
                    <td>4.8890</td>
                </tr>
            
                <tr>
                    <td>001201</td>
                    <td>pair_001201/stitched.png</td>
                    <td>001201.jpg</td>
                    <td class="good">0.9361</td>
                    <td class="good">31.0791</td>
                    <td>50.7188</td>
                    <td>7.1217</td>
                </tr>
            
                <tr>
                    <td>001202</td>
                    <td>pair_001202/stitched.png</td>
                    <td>001202.jpg</td>
                    <td class="good">0.9419</td>
                    <td class="good">31.0642</td>
                    <td>50.8938</td>
                    <td>7.1340</td>
                </tr>
            
                <tr>
                    <td>001211</td>
                    <td>pair_001211/stitched.png</td>
                    <td>001211.jpg</td>
                    <td class="good">0.9255</td>
                    <td class="good">34.6142</td>
                    <td>22.4728</td>
                    <td>4.7406</td>
                </tr>
            
                <tr>
                    <td>001213</td>
                    <td>pair_001213/stitched.png</td>
                    <td>001213.jpg</td>
                    <td class="good">0.9394</td>
                    <td class="good">33.1803</td>
                    <td>31.2648</td>
                    <td>5.5915</td>
                </tr>
            
                <tr>
                    <td>001215</td>
                    <td>pair_001215/stitched.png</td>
                    <td>001215.jpg</td>
                    <td class="good">0.9859</td>
                    <td class="good">41.6924</td>
                    <td>4.4039</td>
                    <td>2.0986</td>
                </tr>
            
                <tr>
                    <td>001218</td>
                    <td>pair_001218/stitched.png</td>
                    <td>001218.jpg</td>
                    <td class="average">0.7580</td>
                    <td class="average">26.5630</td>
                    <td>143.4754</td>
                    <td>11.9781</td>
                </tr>
            
                <tr>
                    <td>001231</td>
                    <td>pair_001231/stitched.png</td>
                    <td>001231.jpg</td>
                    <td class="good">0.9662</td>
                    <td class="good">33.0428</td>
                    <td>32.2702</td>
                    <td>5.6807</td>
                </tr>
            
                <tr>
                    <td>001237</td>
                    <td>pair_001237/stitched.png</td>
                    <td>001237.jpg</td>
                    <td class="good">0.9852</td>
                    <td class="good">34.6770</td>
                    <td>22.1505</td>
                    <td>4.7064</td>
                </tr>
            
                <tr>
                    <td>001250</td>
                    <td>pair_001250/stitched.png</td>
                    <td>001250.jpg</td>
                    <td class="good">0.9306</td>
                    <td class="good">30.2052</td>
                    <td>62.0235</td>
                    <td>7.8755</td>
                </tr>
            
                <tr>
                    <td>001251</td>
                    <td>pair_001251/stitched.png</td>
                    <td>001251.jpg</td>
                    <td class="average">0.8150</td>
                    <td class="average">25.6113</td>
                    <td>178.6266</td>
                    <td>13.3651</td>
                </tr>
            
                <tr>
                    <td>001255</td>
                    <td>pair_001255/stitched.png</td>
                    <td>001255.jpg</td>
                    <td class="good">0.9452</td>
                    <td class="good">31.8209</td>
                    <td>42.7558</td>
                    <td>6.5388</td>
                </tr>
            
                <tr>
                    <td>001256</td>
                    <td>pair_001256/stitched.png</td>
                    <td>001256.jpg</td>
                    <td class="good">0.9150</td>
                    <td class="average">29.8920</td>
                    <td>66.6622</td>
                    <td>8.1647</td>
                </tr>
            
                <tr>
                    <td>001275</td>
                    <td>pair_001275/stitched.png</td>
                    <td>001275.jpg</td>
                    <td class="good">0.9773</td>
                    <td class="good">32.0863</td>
                    <td>40.2211</td>
                    <td>6.3420</td>
                </tr>
            
                <tr>
                    <td>001276</td>
                    <td>pair_001276/stitched.png</td>
                    <td>001276.jpg</td>
                    <td class="good">0.9406</td>
                    <td class="average">29.2898</td>
                    <td>76.5777</td>
                    <td>8.7509</td>
                </tr>
            
                <tr>
                    <td>001277</td>
                    <td>pair_001277/stitched.png</td>
                    <td>001277.jpg</td>
                    <td class="average">0.8628</td>
                    <td class="average">25.9999</td>
                    <td>163.3387</td>
                    <td>12.7804</td>
                </tr>
            
                <tr>
                    <td>001281</td>
                    <td>pair_001281/stitched.png</td>
                    <td>001281.jpg</td>
                    <td class="good">0.9925</td>
                    <td class="good">40.7790</td>
                    <td>5.4348</td>
                    <td>2.3313</td>
                </tr>
            
                <tr>
                    <td>001291</td>
                    <td>pair_001291/stitched.png</td>
                    <td>001291.jpg</td>
                    <td class="good">0.9891</td>
                    <td class="good">38.9968</td>
                    <td>8.1921</td>
                    <td>2.8622</td>
                </tr>
            
                <tr>
                    <td>001293</td>
                    <td>pair_001293/stitched.png</td>
                    <td>001293.jpg</td>
                    <td class="good">0.9178</td>
                    <td class="average">29.6820</td>
                    <td>69.9656</td>
                    <td>8.3645</td>
                </tr>
            
                <tr>
                    <td>001321</td>
                    <td>pair_001321/stitched.png</td>
                    <td>001321.jpg</td>
                    <td class="good">0.9279</td>
                    <td class="good">31.0323</td>
                    <td>51.2685</td>
                    <td>7.1602</td>
                </tr>
            
                <tr>
                    <td>001342</td>
                    <td>pair_001342/stitched.png</td>
                    <td>001342.jpg</td>
                    <td class="good">0.9680</td>
                    <td class="good">34.5184</td>
                    <td>22.9743</td>
                    <td>4.7931</td>
                </tr>
            
                <tr>
                    <td>001366</td>
                    <td>pair_001366/stitched.png</td>
                    <td>001366.jpg</td>
                    <td class="good">0.9107</td>
                    <td class="average">29.5524</td>
                    <td>72.0835</td>
                    <td>8.4902</td>
                </tr>
            
                <tr>
                    <td>001367</td>
                    <td>pair_001367/stitched.png</td>
                    <td>001367.jpg</td>
                    <td class="good">0.9447</td>
                    <td class="good">30.6806</td>
                    <td>55.5936</td>
                    <td>7.4561</td>
                </tr>
            
                <tr>
                    <td>001368</td>
                    <td>pair_001368/stitched.png</td>
                    <td>001368.jpg</td>
                    <td class="good">0.9918</td>
                    <td class="good">41.6420</td>
                    <td>4.4553</td>
                    <td>2.1108</td>
                </tr>
            
                <tr>
                    <td>001373</td>
                    <td>pair_001373/stitched.png</td>
                    <td>001373.jpg</td>
                    <td class="good">0.9506</td>
                    <td class="good">32.7444</td>
                    <td>34.5655</td>
                    <td>5.8792</td>
                </tr>
            
                <tr>
                    <td>001379</td>
                    <td>pair_001379/stitched.png</td>
                    <td>001379.jpg</td>
                    <td class="good">0.9348</td>
                    <td class="good">31.2367</td>
                    <td>48.9118</td>
                    <td>6.9937</td>
                </tr>
            
                <tr>
                    <td>001383</td>
                    <td>pair_001383/stitched.png</td>
                    <td>001383.jpg</td>
                    <td class="good">0.9732</td>
                    <td class="good">34.3198</td>
                    <td>24.0492</td>
                    <td>4.9040</td>
                </tr>
            
                <tr>
                    <td>001387</td>
                    <td>pair_001387/stitched.png</td>
                    <td>001387.jpg</td>
                    <td class="good">0.9329</td>
                    <td class="good">36.2494</td>
                    <td>15.4221</td>
                    <td>3.9271</td>
                </tr>
            
                <tr>
                    <td>001393</td>
                    <td>pair_001393/stitched.png</td>
                    <td>001393.jpg</td>
                    <td class="good">0.9589</td>
                    <td class="good">34.0826</td>
                    <td>25.3991</td>
                    <td>5.0398</td>
                </tr>
            
                <tr>
                    <td>001394</td>
                    <td>pair_001394/stitched.png</td>
                    <td>001394.jpg</td>
                    <td class="good">0.9049</td>
                    <td class="good">34.9174</td>
                    <td>20.9577</td>
                    <td>4.5780</td>
                </tr>
            
                <tr>
                    <td>001397</td>
                    <td>pair_001397/stitched.png</td>
                    <td>001397.jpg</td>
                    <td class="good">0.9896</td>
                    <td class="good">41.5313</td>
                    <td>4.5704</td>
                    <td>2.1379</td>
                </tr>
            
                <tr>
                    <td>001398</td>
                    <td>pair_001398/stitched.png</td>
                    <td>001398.jpg</td>
                    <td class="good">0.9762</td>
                    <td class="good">38.7680</td>
                    <td>8.6354</td>
                    <td>2.9386</td>
                </tr>
            
                <tr>
                    <td>001400</td>
                    <td>pair_001400/stitched.png</td>
                    <td>001400.jpg</td>
                    <td class="good">0.9617</td>
                    <td class="good">32.1982</td>
                    <td>39.1973</td>
                    <td>6.2608</td>
                </tr>
            
                <tr>
                    <td>001404</td>
                    <td>pair_001404/stitched.png</td>
                    <td>001404.jpg</td>
                    <td class="average">0.8620</td>
                    <td class="average">24.2501</td>
                    <td>244.3831</td>
                    <td>15.6328</td>
                </tr>
            
                <tr>
                    <td>001405</td>
                    <td>pair_001405/stitched.png</td>
                    <td>001405.jpg</td>
                    <td class="good">0.9389</td>
                    <td class="average">27.8317</td>
                    <td>107.1298</td>
                    <td>10.3504</td>
                </tr>
            
                <tr>
                    <td>001432</td>
                    <td>pair_001432/stitched.png</td>
                    <td>001432.jpg</td>
                    <td class="good">0.9851</td>
                    <td class="good">35.8612</td>
                    <td>16.8641</td>
                    <td>4.1066</td>
                </tr>
            
                <tr>
                    <td>001433</td>
                    <td>pair_001433/stitched.png</td>
                    <td>001433.jpg</td>
                    <td class="good">0.9840</td>
                    <td class="good">36.5318</td>
                    <td>14.4511</td>
                    <td>3.8015</td>
                </tr>
            
                <tr>
                    <td>001434</td>
                    <td>pair_001434/stitched.png</td>
                    <td>001434.jpg</td>
                    <td class="good">0.9558</td>
                    <td class="good">31.6001</td>
                    <td>44.9852</td>
                    <td>6.7071</td>
                </tr>
            
                <tr>
                    <td>001438</td>
                    <td>pair_001438/stitched.png</td>
                    <td>001438.jpg</td>
                    <td class="good">0.9282</td>
                    <td class="average">29.1907</td>
                    <td>78.3457</td>
                    <td>8.8513</td>
                </tr>
            
                <tr>
                    <td>001444</td>
                    <td>pair_001444/stitched.png</td>
                    <td>001444.jpg</td>
                    <td class="good">0.9103</td>
                    <td class="average">26.7090</td>
                    <td>138.7338</td>
                    <td>11.7785</td>
                </tr>
            
                <tr>
                    <td>001466</td>
                    <td>pair_001466/stitched.png</td>
                    <td>001466.jpg</td>
                    <td class="good">0.9665</td>
                    <td class="good">34.0439</td>
                    <td>25.6265</td>
                    <td>5.0623</td>
                </tr>
            
                <tr>
                    <td>001481</td>
                    <td>pair_001481/stitched.png</td>
                    <td>001481.jpg</td>
                    <td class="average">0.7781</td>
                    <td class="average">26.3788</td>
                    <td>149.6922</td>
                    <td>12.2349</td>
                </tr>
            
                <tr>
                    <td>001482</td>
                    <td>pair_001482/stitched.png</td>
                    <td>001482.jpg</td>
                    <td class="average">0.7798</td>
                    <td class="average">26.2665</td>
                    <td>153.6149</td>
                    <td>12.3941</td>
                </tr>
            
                <tr>
                    <td>001497</td>
                    <td>pair_001497/stitched.png</td>
                    <td>001497.jpg</td>
                    <td class="good">0.9186</td>
                    <td class="average">26.7845</td>
                    <td>136.3433</td>
                    <td>11.6766</td>
                </tr>
            
                <tr>
                    <td>001513</td>
                    <td>pair_001513/stitched.png</td>
                    <td>001513.jpg</td>
                    <td class="good">0.9642</td>
                    <td class="average">26.8140</td>
                    <td>135.4191</td>
                    <td>11.6370</td>
                </tr>
            
                <tr>
                    <td>001514</td>
                    <td>pair_001514/stitched.png</td>
                    <td>001514.jpg</td>
                    <td class="average">0.8550</td>
                    <td class="average">24.3340</td>
                    <td>239.7088</td>
                    <td>15.4825</td>
                </tr>
            
                <tr>
                    <td>001524</td>
                    <td>pair_001524/stitched.png</td>
                    <td>001524.jpg</td>
                    <td class="good">0.9477</td>
                    <td class="good">33.4565</td>
                    <td>29.3379</td>
                    <td>5.4164</td>
                </tr>
            
                <tr>
                    <td>001526</td>
                    <td>pair_001526/stitched.png</td>
                    <td>001526.jpg</td>
                    <td class="good">0.9864</td>
                    <td class="good">35.3807</td>
                    <td>18.8368</td>
                    <td>4.3401</td>
                </tr>
            
                <tr>
                    <td>001527</td>
                    <td>pair_001527/stitched.png</td>
                    <td>001527.jpg</td>
                    <td class="good">0.9941</td>
                    <td class="good">37.9314</td>
                    <td>10.4698</td>
                    <td>3.2357</td>
                </tr>
            
                <tr>
                    <td>001528</td>
                    <td>pair_001528/stitched.png</td>
                    <td>001528.jpg</td>
                    <td class="good">0.9551</td>
                    <td class="good">34.3174</td>
                    <td>24.0626</td>
                    <td>4.9054</td>
                </tr>
            
                <tr>
                    <td>001529</td>
                    <td>pair_001529/stitched.png</td>
                    <td>001529.jpg</td>
                    <td class="good">0.9410</td>
                    <td class="good">32.9857</td>
                    <td>32.6971</td>
                    <td>5.7181</td>
                </tr>
            
                <tr>
                    <td>001530</td>
                    <td>pair_001530/stitched.png</td>
                    <td>001530.jpg</td>
                    <td class="good">0.9609</td>
                    <td class="good">34.6680</td>
                    <td>22.1961</td>
                    <td>4.7113</td>
                </tr>
            
                <tr>
                    <td>001536</td>
                    <td>pair_001536/stitched.png</td>
                    <td>001536.jpg</td>
                    <td class="good">0.9630</td>
                    <td class="good">39.0524</td>
                    <td>8.0879</td>
                    <td>2.8439</td>
                </tr>
            
                <tr>
                    <td>001540</td>
                    <td>pair_001540/stitched.png</td>
                    <td>001540.jpg</td>
                    <td class="good">0.9918</td>
                    <td class="good">39.4965</td>
                    <td>7.3017</td>
                    <td>2.7022</td>
                </tr>
            
                <tr>
                    <td>001550</td>
                    <td>pair_001550/stitched.png</td>
                    <td>001550.jpg</td>
                    <td class="average">0.8728</td>
                    <td class="average">27.9182</td>
                    <td>105.0161</td>
                    <td>10.2477</td>
                </tr>
            
                <tr>
                    <td>001597</td>
                    <td>pair_001597/stitched.png</td>
                    <td>001597.jpg</td>
                    <td class="good">0.9227</td>
                    <td class="good">32.0402</td>
                    <td>40.6504</td>
                    <td>6.3758</td>
                </tr>
            
                <tr>
                    <td>001685</td>
                    <td>pair_001685/stitched.png</td>
                    <td>001685.jpg</td>
                    <td class="average">0.7848</td>
                    <td class="average">23.0558</td>
                    <td>321.7358</td>
                    <td>17.9370</td>
                </tr>
            
                <tr>
                    <td>001717</td>
                    <td>pair_001717/stitched.png</td>
                    <td>001717.jpg</td>
                    <td class="poor">0.7404</td>
                    <td class="average">21.5344</td>
                    <td>456.7125</td>
                    <td>21.3708</td>
                </tr>
            
                <tr>
                    <td>001736</td>
                    <td>pair_001736/stitched.png</td>
                    <td>001736.jpg</td>
                    <td class="average">0.7656</td>
                    <td class="average">26.2063</td>
                    <td>155.7588</td>
                    <td>12.4803</td>
                </tr>
            
                <tr>
                    <td>001770</td>
                    <td>pair_001770/stitched.png</td>
                    <td>001770.jpg</td>
                    <td class="good">0.9303</td>
                    <td class="good">30.4358</td>
                    <td>58.8173</td>
                    <td>7.6692</td>
                </tr>
            
                <tr>
                    <td>001788</td>
                    <td>pair_001788/stitched.png</td>
                    <td>001788.jpg</td>
                    <td class="average">0.8686</td>
                    <td class="average">26.5651</td>
                    <td>143.4060</td>
                    <td>11.9752</td>
                </tr>
            
                <tr>
                    <td>001804</td>
                    <td>pair_001804/stitched.png</td>
                    <td>001804.jpg</td>
                    <td class="average">0.8361</td>
                    <td class="average">26.6867</td>
                    <td>139.4487</td>
                    <td>11.8088</td>
                </tr>
            
                <tr>
                    <td>001806</td>
                    <td>pair_001806/stitched.png</td>
                    <td>001806.jpg</td>
                    <td class="good">0.9418</td>
                    <td class="good">31.4204</td>
                    <td>46.8861</td>
                    <td>6.8473</td>
                </tr>
            
                <tr>
                    <td>001859</td>
                    <td>pair_001859/stitched.png</td>
                    <td>001859.jpg</td>
                    <td class="average">0.8177</td>
                    <td class="average">24.5260</td>
                    <td>229.3423</td>
                    <td>15.1441</td>
                </tr>
            
                <tr>
                    <td>001885</td>
                    <td>pair_001885/stitched.png</td>
                    <td>001885.jpg</td>
                    <td class="good">0.9740</td>
                    <td class="good">39.5555</td>
                    <td>7.2033</td>
                    <td>2.6839</td>
                </tr>
            
                <tr>
                    <td>001887</td>
                    <td>pair_001887/stitched.png</td>
                    <td>001887.jpg</td>
                    <td class="good">0.9168</td>
                    <td class="good">35.0578</td>
                    <td>20.2910</td>
                    <td>4.5046</td>
                </tr>
            
                <tr>
                    <td>001900</td>
                    <td>pair_001900/stitched.png</td>
                    <td>001900.jpg</td>
                    <td class="average">0.7848</td>
                    <td class="average">24.0424</td>
                    <td>256.3542</td>
                    <td>16.0111</td>
                </tr>
            
                <tr>
                    <td>000009</td>
                    <td>pair_000009/stitched.png</td>
                    <td>000009.jpg</td>
                    <td class="good">0.9767</td>
                    <td class="good">38.9188</td>
                    <td>8.3406</td>
                    <td>2.8880</td>
                </tr>
            
                <tr>
                    <td>000010</td>
                    <td>pair_000010/stitched.png</td>
                    <td>000010.jpg</td>
                    <td class="good">0.9215</td>
                    <td class="good">34.1931</td>
                    <td>24.7614</td>
                    <td>4.9761</td>
                </tr>
            
                <tr>
                    <td>000022</td>
                    <td>pair_000022/stitched.png</td>
                    <td>000022.jpg</td>
                    <td class="good">0.9938</td>
                    <td class="good">42.9868</td>
                    <td>3.2689</td>
                    <td>1.8080</td>
                </tr>
            
                <tr>
                    <td>000028</td>
                    <td>pair_000028/stitched.png</td>
                    <td>000028.jpg</td>
                    <td class="good">0.9472</td>
                    <td class="good">31.6382</td>
                    <td>44.5922</td>
                    <td>6.6777</td>
                </tr>
            
                <tr>
                    <td>000032</td>
                    <td>pair_000032/stitched.png</td>
                    <td>000032.jpg</td>
                    <td class="good">0.9413</td>
                    <td class="good">33.5365</td>
                    <td>28.8024</td>
                    <td>5.3668</td>
                </tr>
            
                <tr>
                    <td>000035</td>
                    <td>pair_000035/stitched.png</td>
                    <td>000035.jpg</td>
                    <td class="good">0.9438</td>
                    <td class="average">29.1203</td>
                    <td>79.6243</td>
                    <td>8.9232</td>
                </tr>
            
                <tr>
                    <td>000036</td>
                    <td>pair_000036/stitched.png</td>
                    <td>000036.jpg</td>
                    <td class="good">0.9329</td>
                    <td class="average">28.3375</td>
                    <td>95.3518</td>
                    <td>9.7648</td>
                </tr>
            
                <tr>
                    <td>000041</td>
                    <td>pair_000041/stitched.png</td>
                    <td>000041.jpg</td>
                    <td class="good">0.9583</td>
                    <td class="good">32.3612</td>
                    <td>37.7537</td>
                    <td>6.1444</td>
                </tr>
            
                <tr>
                    <td>000042</td>
                    <td>pair_000042/stitched.png</td>
                    <td>000042.jpg</td>
                    <td class="good">0.9973</td>
                    <td class="good">45.2418</td>
                    <td>1.9449</td>
                    <td>1.3946</td>
                </tr>
            
                <tr>
                    <td>000053</td>
                    <td>pair_000053/stitched.png</td>
                    <td>000053.jpg</td>
                    <td class="good">0.9081</td>
                    <td class="average">28.5321</td>
                    <td>91.1737</td>
                    <td>9.5485</td>
                </tr>
            
                <tr>
                    <td>000072</td>
                    <td>pair_000072/stitched.png</td>
                    <td>000072.jpg</td>
                    <td class="good">0.9011</td>
                    <td class="good">30.2247</td>
                    <td>61.7460</td>
                    <td>7.8579</td>
                </tr>
            
                <tr>
                    <td>000109</td>
                    <td>pair_000109/stitched.png</td>
                    <td>000109.jpg</td>
                    <td class="good">0.9674</td>
                    <td class="good">36.3584</td>
                    <td>15.0397</td>
                    <td>3.8781</td>
                </tr>
            
                <tr>
                    <td>000112</td>
                    <td>pair_000112/stitched.png</td>
                    <td>000112.jpg</td>
                    <td class="good">0.9856</td>
                    <td class="good">38.6897</td>
                    <td>8.7924</td>
                    <td>2.9652</td>
                </tr>
            
                <tr>
                    <td>000113</td>
                    <td>pair_000113/stitched.png</td>
                    <td>000113.jpg</td>
                    <td class="good">0.9529</td>
                    <td class="good">32.2333</td>
                    <td>38.8824</td>
                    <td>6.2356</td>
                </tr>
            
                <tr>
                    <td>000125</td>
                    <td>pair_000125/stitched.png</td>
                    <td>000125.jpg</td>
                    <td class="good">0.9491</td>
                    <td class="good">32.7086</td>
                    <td>34.8511</td>
                    <td>5.9035</td>
                </tr>
            
                <tr>
                    <td>000136</td>
                    <td>pair_000136/stitched.png</td>
                    <td>000136.jpg</td>
                    <td class="good">0.9026</td>
                    <td class="average">28.6058</td>
                    <td>89.6406</td>
                    <td>9.4679</td>
                </tr>
            
                <tr>
                    <td>000158</td>
                    <td>pair_000158/stitched.png</td>
                    <td>000158.jpg</td>
                    <td class="good">0.9220</td>
                    <td class="good">30.0307</td>
                    <td>64.5663</td>
                    <td>8.0353</td>
                </tr>
            
                <tr>
                    <td>000159</td>
                    <td>pair_000159/stitched.png</td>
                    <td>000159.jpg</td>
                    <td class="good">0.9053</td>
                    <td class="average">29.0545</td>
                    <td>80.8405</td>
                    <td>8.9911</td>
                </tr>
            
                <tr>
                    <td>000161</td>
                    <td>pair_000161/stitched.png</td>
                    <td>000161.jpg</td>
                    <td class="good">0.9970</td>
                    <td class="good">45.2104</td>
                    <td>1.9590</td>
                    <td>1.3997</td>
                </tr>
            
                <tr>
                    <td>000177</td>
                    <td>pair_000177/stitched.png</td>
                    <td>000177.jpg</td>
                    <td class="good">0.9214</td>
                    <td class="good">30.1052</td>
                    <td>63.4683</td>
                    <td>7.9667</td>
                </tr>
            
                <tr>
                    <td>000179</td>
                    <td>pair_000179/stitched.png</td>
                    <td>000179.jpg</td>
                    <td class="average">0.8552</td>
                    <td class="average">25.9654</td>
                    <td>164.6428</td>
                    <td>12.8313</td>
                </tr>
            
                <tr>
                    <td>000184</td>
                    <td>pair_000184/stitched.png</td>
                    <td>000184.jpg</td>
                    <td class="good">0.9638</td>
                    <td class="good">33.6345</td>
                    <td>28.1599</td>
                    <td>5.3066</td>
                </tr>
            
                <tr>
                    <td>000190</td>
                    <td>pair_000190/stitched.png</td>
                    <td>000190.jpg</td>
                    <td class="good">0.9937</td>
                    <td class="good">38.7881</td>
                    <td>8.5955</td>
                    <td>2.9318</td>
                </tr>
            
                <tr>
                    <td>000191</td>
                    <td>pair_000191/stitched.png</td>
                    <td>000191.jpg</td>
                    <td class="good">0.9467</td>
                    <td class="average">29.6220</td>
                    <td>70.9387</td>
                    <td>8.4225</td>
                </tr>
            
                <tr>
                    <td>000192</td>
                    <td>pair_000192/stitched.png</td>
                    <td>000192.jpg</td>
                    <td class="good">0.9624</td>
                    <td class="good">30.8981</td>
                    <td>52.8776</td>
                    <td>7.2717</td>
                </tr>
            
                <tr>
                    <td>000193</td>
                    <td>pair_000193/stitched.png</td>
                    <td>000193.jpg</td>
                    <td class="good">0.9050</td>
                    <td class="good">31.0029</td>
                    <td>51.6164</td>
                    <td>7.1845</td>
                </tr>
            
                <tr>
                    <td>000199</td>
                    <td>pair_000199/stitched.png</td>
                    <td>000199.jpg</td>
                    <td class="good">0.9214</td>
                    <td class="good">31.1306</td>
                    <td>50.1214</td>
                    <td>7.0797</td>
                </tr>
            
                <tr>
                    <td>000222</td>
                    <td>pair_000222/stitched.png</td>
                    <td>000222.jpg</td>
                    <td class="good">0.9218</td>
                    <td class="good">32.1888</td>
                    <td>39.2829</td>
                    <td>6.2676</td>
                </tr>
            
                <tr>
                    <td>000226</td>
                    <td>pair_000226/stitched.png</td>
                    <td>000226.jpg</td>
                    <td class="average">0.8877</td>
                    <td class="average">27.3788</td>
                    <td>118.9050</td>
                    <td>10.9044</td>
                </tr>
            
                <tr>
                    <td>000228</td>
                    <td>pair_000228/stitched.png</td>
                    <td>000228.jpg</td>
                    <td class="good">0.9550</td>
                    <td class="good">30.7967</td>
                    <td>54.1264</td>
                    <td>7.3571</td>
                </tr>
            
                <tr>
                    <td>000229</td>
                    <td>pair_000229/stitched.png</td>
                    <td>000229.jpg</td>
                    <td class="good">0.9816</td>
                    <td class="good">37.0935</td>
                    <td>12.6978</td>
                    <td>3.5634</td>
                </tr>
            
                <tr>
                    <td>000241</td>
                    <td>pair_000241/stitched.png</td>
                    <td>000241.jpg</td>
                    <td class="good">0.9544</td>
                    <td class="good">30.9259</td>
                    <td>52.5400</td>
                    <td>7.2484</td>
                </tr>
            
                <tr>
                    <td>000254</td>
                    <td>pair_000254/stitched.png</td>
                    <td>000254.jpg</td>
                    <td class="good">0.9466</td>
                    <td class="good">34.8843</td>
                    <td>21.1179</td>
                    <td>4.5954</td>
                </tr>
            
                <tr>
                    <td>000256</td>
                    <td>pair_000256/stitched.png</td>
                    <td>000256.jpg</td>
                    <td class="good">0.9258</td>
                    <td class="good">30.5009</td>
                    <td>57.9415</td>
                    <td>7.6119</td>
                </tr>
            
                <tr>
                    <td>000257</td>
                    <td>pair_000257/stitched.png</td>
                    <td>000257.jpg</td>
                    <td class="good">0.9374</td>
                    <td class="good">32.3111</td>
                    <td>38.1918</td>
                    <td>6.1800</td>
                </tr>
            
                <tr>
                    <td>000260</td>
                    <td>pair_000260/stitched.png</td>
                    <td>000260.jpg</td>
                    <td class="good">0.9938</td>
                    <td class="good">42.2795</td>
                    <td>3.8471</td>
                    <td>1.9614</td>
                </tr>
            
                <tr>
                    <td>000263</td>
                    <td>pair_000263/stitched.png</td>
                    <td>000263.jpg</td>
                    <td class="good">0.9866</td>
                    <td class="good">34.5372</td>
                    <td>22.8750</td>
                    <td>4.7828</td>
                </tr>
            
                <tr>
                    <td>000282</td>
                    <td>pair_000282/stitched.png</td>
                    <td>000282.jpg</td>
                    <td class="good">0.9890</td>
                    <td class="good">43.3022</td>
                    <td>3.0399</td>
                    <td>1.7435</td>
                </tr>
            
                <tr>
                    <td>000283</td>
                    <td>pair_000283/stitched.png</td>
                    <td>000283.jpg</td>
                    <td class="average">0.7796</td>
                    <td class="average">24.1697</td>
                    <td>248.9475</td>
                    <td>15.7781</td>
                </tr>
            
                <tr>
                    <td>000285</td>
                    <td>pair_000285/stitched.png</td>
                    <td>000285.jpg</td>
                    <td class="good">0.9837</td>
                    <td class="good">37.1909</td>
                    <td>12.4161</td>
                    <td>3.5237</td>
                </tr>
            
                <tr>
                    <td>000304</td>
                    <td>pair_000304/stitched.png</td>
                    <td>000304.jpg</td>
                    <td class="good">0.9448</td>
                    <td class="average">29.6322</td>
                    <td>70.7725</td>
                    <td>8.4126</td>
                </tr>
            
                <tr>
                    <td>000313</td>
                    <td>pair_000313/stitched.png</td>
                    <td>000313.jpg</td>
                    <td class="good">0.9579</td>
                    <td class="good">34.8189</td>
                    <td>21.4381</td>
                    <td>4.6301</td>
                </tr>
            
                <tr>
                    <td>000320</td>
                    <td>pair_000320/stitched.png</td>
                    <td>000320.jpg</td>
                    <td class="average">0.8891</td>
                    <td class="average">28.2517</td>
                    <td>97.2545</td>
                    <td>9.8618</td>
                </tr>
            
                <tr>
                    <td>000326</td>
                    <td>pair_000326/stitched.png</td>
                    <td>000326.jpg</td>
                    <td class="good">0.9299</td>
                    <td class="good">30.5218</td>
                    <td>57.6628</td>
                    <td>7.5936</td>
                </tr>
            
                <tr>
                    <td>000337</td>
                    <td>pair_000337/stitched.png</td>
                    <td>000337.jpg</td>
                    <td class="average">0.8662</td>
                    <td class="average">25.7515</td>
                    <td>172.9539</td>
                    <td>13.1512</td>
                </tr>
            
                <tr>
                    <td>000338</td>
                    <td>pair_000338/stitched.png</td>
                    <td>000338.jpg</td>
                    <td class="average">0.8346</td>
                    <td class="average">27.3125</td>
                    <td>120.7350</td>
                    <td>10.9879</td>
                </tr>
            
                <tr>
                    <td>000339</td>
                    <td>pair_000339/stitched.png</td>
                    <td>000339.jpg</td>
                    <td class="good">0.9769</td>
                    <td class="good">36.2156</td>
                    <td>15.5423</td>
                    <td>3.9424</td>
                </tr>
            
                <tr>
                    <td>000363</td>
                    <td>pair_000363/stitched.png</td>
                    <td>000363.jpg</td>
                    <td class="good">0.9972</td>
                    <td class="good">47.9018</td>
                    <td>1.0541</td>
                    <td>1.0267</td>
                </tr>
            
                <tr>
                    <td>000366</td>
                    <td>pair_000366/stitched.png</td>
                    <td>000366.jpg</td>
                    <td class="good">0.9144</td>
                    <td class="good">33.1093</td>
                    <td>31.7796</td>
                    <td>5.6373</td>
                </tr>
            
                <tr>
                    <td>000371</td>
                    <td>pair_000371/stitched.png</td>
                    <td>000371.jpg</td>
                    <td class="good">0.9607</td>
                    <td class="good">30.6480</td>
                    <td>56.0122</td>
                    <td>7.4841</td>
                </tr>
            
                <tr>
                    <td>000375</td>
                    <td>pair_000375/stitched.png</td>
                    <td>000375.jpg</td>
                    <td class="average">0.8398</td>
                    <td class="average">24.8172</td>
                    <td>214.4654</td>
                    <td>14.6446</td>
                </tr>
            
                <tr>
                    <td>000392</td>
                    <td>pair_000392/stitched.png</td>
                    <td>000392.jpg</td>
                    <td class="good">0.9399</td>
                    <td class="good">31.5870</td>
                    <td>45.1209</td>
                    <td>6.7172</td>
                </tr>
            
                <tr>
                    <td>000393</td>
                    <td>pair_000393/stitched.png</td>
                    <td>000393.jpg</td>
                    <td class="good">0.9905</td>
                    <td class="good">36.2385</td>
                    <td>15.4607</td>
                    <td>3.9320</td>
                </tr>
            
                <tr>
                    <td>000394</td>
                    <td>pair_000394/stitched.png</td>
                    <td>000394.jpg</td>
                    <td class="good">0.9845</td>
                    <td class="good">35.0465</td>
                    <td>20.3436</td>
                    <td>4.5104</td>
                </tr>
            
                <tr>
                    <td>000396</td>
                    <td>pair_000396/stitched.png</td>
                    <td>000396.jpg</td>
                    <td class="good">0.9697</td>
                    <td class="good">34.1805</td>
                    <td>24.8333</td>
                    <td>4.9833</td>
                </tr>
            
                <tr>
                    <td>000423</td>
                    <td>pair_000423/stitched.png</td>
                    <td>000423.jpg</td>
                    <td class="good">0.9413</td>
                    <td class="good">30.2109</td>
                    <td>61.9428</td>
                    <td>7.8704</td>
                </tr>
            
                <tr>
                    <td>000009</td>
                    <td>pair_000009/stitched.png</td>
                    <td>000009.jpg</td>
                    <td class="good">0.9767</td>
                    <td class="good">38.9188</td>
                    <td>8.3406</td>
                    <td>2.8880</td>
                </tr>
            
                <tr>
                    <td>000010</td>
                    <td>pair_000010/stitched.png</td>
                    <td>000010.jpg</td>
                    <td class="good">0.9215</td>
                    <td class="good">34.1931</td>
                    <td>24.7614</td>
                    <td>4.9761</td>
                </tr>
            
                <tr>
                    <td>000022</td>
                    <td>pair_000022/stitched.png</td>
                    <td>000022.jpg</td>
                    <td class="good">0.9938</td>
                    <td class="good">42.9868</td>
                    <td>3.2689</td>
                    <td>1.8080</td>
                </tr>
            
                <tr>
                    <td>000028</td>
                    <td>pair_000028/stitched.png</td>
                    <td>000028.jpg</td>
                    <td class="good">0.9472</td>
                    <td class="good">31.6382</td>
                    <td>44.5922</td>
                    <td>6.6777</td>
                </tr>
            
                <tr>
                    <td>000032</td>
                    <td>pair_000032/stitched.png</td>
                    <td>000032.jpg</td>
                    <td class="good">0.9413</td>
                    <td class="good">33.5365</td>
                    <td>28.8024</td>
                    <td>5.3668</td>
                </tr>
            
                <tr>
                    <td>000035</td>
                    <td>pair_000035/stitched.png</td>
                    <td>000035.jpg</td>
                    <td class="good">0.9438</td>
                    <td class="average">29.1203</td>
                    <td>79.6243</td>
                    <td>8.9232</td>
                </tr>
            
                <tr>
                    <td>000036</td>
                    <td>pair_000036/stitched.png</td>
                    <td>000036.jpg</td>
                    <td class="good">0.9329</td>
                    <td class="average">28.3375</td>
                    <td>95.3518</td>
                    <td>9.7648</td>
                </tr>
            
                <tr>
                    <td>000041</td>
                    <td>pair_000041/stitched.png</td>
                    <td>000041.jpg</td>
                    <td class="good">0.9583</td>
                    <td class="good">32.3612</td>
                    <td>37.7537</td>
                    <td>6.1444</td>
                </tr>
            
                <tr>
                    <td>000042</td>
                    <td>pair_000042/stitched.png</td>
                    <td>000042.jpg</td>
                    <td class="good">0.9973</td>
                    <td class="good">45.2418</td>
                    <td>1.9449</td>
                    <td>1.3946</td>
                </tr>
            
            </table>

            <h2>样本图像对比</h2>
            <div class="gallery">
        
                <div class="gallery-item">
                    <h3>图像 001193</h3>
                    <p>SSIM: 0.9982 | PSNR: 47.2911</p>
                    <img src="comparison_images\001193_comparison.png" alt="图像对比">
                    <h4>差异热力图</h4>
                    <img src="comparison_images\001193_diff.png" alt="差异热力图">
                </div>
            
                <div class="gallery-item">
                    <h3>图像 000042</h3>
                    <p>SSIM: 0.9973 | PSNR: 45.2418</p>
                    <img src="comparison_images\000042_comparison.png" alt="图像对比">
                    <h4>差异热力图</h4>
                    <img src="comparison_images\000042_diff.png" alt="差异热力图">
                </div>
            
                <div class="gallery-item">
                    <h3>图像 000042</h3>
                    <p>SSIM: 0.9973 | PSNR: 45.2418</p>
                    <img src="comparison_images\000042_comparison.png" alt="图像对比">
                    <h4>差异热力图</h4>
                    <img src="comparison_images\000042_diff.png" alt="差异热力图">
                </div>
            
                <div class="gallery-item">
                    <h3>图像 000042</h3>
                    <p>SSIM: 0.9973 | PSNR: 45.2418</p>
                    <img src="comparison_images\000042_comparison.png" alt="图像对比">
                    <h4>差异热力图</h4>
                    <img src="comparison_images\000042_diff.png" alt="差异热力图">
                </div>
            
                <div class="gallery-item">
                    <h3>图像 000363</h3>
                    <p>SSIM: 0.9972 | PSNR: 47.9018</p>
                    <img src="comparison_images\000363_comparison.png" alt="图像对比">
                    <h4>差异热力图</h4>
                    <img src="comparison_images\000363_diff.png" alt="差异热力图">
                </div>
            
                <div class="gallery-item">
                    <h3>图像 000363</h3>
                    <p>SSIM: 0.9972 | PSNR: 47.9018</p>
                    <img src="comparison_images\000363_comparison.png" alt="图像对比">
                    <h4>差异热力图</h4>
                    <img src="comparison_images\000363_diff.png" alt="差异热力图">
                </div>
            
                <div class="gallery-item">
                    <h3>图像 000161</h3>
                    <p>SSIM: 0.9970 | PSNR: 45.2104</p>
                    <img src="comparison_images\000161_comparison.png" alt="图像对比">
                    <h4>差异热力图</h4>
                    <img src="comparison_images\000161_diff.png" alt="差异热力图">
                </div>
            
                <div class="gallery-item">
                    <h3>图像 000161</h3>
                    <p>SSIM: 0.9970 | PSNR: 45.2104</p>
                    <img src="comparison_images\000161_comparison.png" alt="图像对比">
                    <h4>差异热力图</h4>
                    <img src="comparison_images\000161_diff.png" alt="差异热力图">
                </div>
            
                <div class="gallery-item">
                    <h3>图像 001093</h3>
                    <p>SSIM: 0.9968 | PSNR: 43.3202</p>
                    <img src="comparison_images\001093_comparison.png" alt="图像对比">
                    <h4>差异热力图</h4>
                    <img src="comparison_images\001093_diff.png" alt="差异热力图">
                </div>
            
                <div class="gallery-item">
                    <h3>图像 000527</h3>
                    <p>SSIM: 0.9964 | PSNR: 42.9154</p>
                    <img src="comparison_images\000527_comparison.png" alt="图像对比">
                    <h4>差异热力图</h4>
                    <img src="comparison_images\000527_diff.png" alt="差异热力图">
                </div>
            
            </div>
        </body>
        </html>
        ng evaluation_report.html…]()

性能与局限性
* 计算需求：LoFTR特征匹配需要较高的GPU内存，对于高分辨率图像可能需要降采样
* 色调一致性：拼接优化专注于边界区域，对整体色调一致性的改善有限
* 实时性能：当前实现重点在准确性而非速度，不适用于实时系统

维护者
* [Jiayi Li1, Kaizhi Dong, Guihui Li]
* [{jiayilee, dongkaizhi, guihuilee}@stu.ouc.edu.cn

