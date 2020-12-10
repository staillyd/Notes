# 数据
- 真实数据
![](imgs/Going%20Beyond%20Real%20Data%20A%20Robust%20Visual%20Representation%20for%20Vehicle/real-example.jpg)
  - 现实世界中40个相机采集的数据
  - 总共666辆车被标注，其中333辆作为训练集，333辆作为测试集
  - 总共56277张图片，训练集36935张图片，测试集18290张图片
  - 1052张图片作为queries

- 仿真数据
![](imgs/Going%20Beyond%20Real%20Data%20A%20Robust%20Visual%20Representation%20for%20Vehicle/simulation-example.jpg)
  - 由3d引擎生成的图片
  - 1362辆车，192150张图片

# 代码中采用的文件存储方式
## 训练
- 真实数据命名:id_相机id_当前相机下的第几张图片
- 仿真数据命名:id_相机id_颜色_车型_方向_当前相机下的第几张图片
```
pytorch2020
└───gallery
└───train  (真实数据里id<=400的数据)
│   └───000001
│       │   000001_c002_0.jpg
│       │   000001_c002_1.jpg
│       │   ...
└───query  (真实数据里400<id<=666,每相机只取一张的数据)
│   └───000401
│       │   000401_c016_8.jpg
│       │   000401_c017_26.jpg
│       │   ...
└───train_real_all  (真实数据，每id一个文件夹)
└───virtual  (仿真数据)
│   └───000667
│       │   000667_c006_10_10_4_13.jpg
│       │   ...
```

# 数据生成
![](imgs/Going%20Beyond%20Real%20Data%20A%20Robust%20Visual%20Representation%20for%20Vehicle/Style%20Transform%20Content%20Manipulation.png)
- **风格迁移**
  - CycleGAN:由于仿真数据和真实数据存在明显的样式差异，使用风格迁移，使仿真数据更加逼近真实世界的分布
- **Content Manipulation**
  - DGNet:生成具有不同视觉外观的样本。(对re-id特别有效)
    - 两个编码器，一个负责外观，一个负责结构
    - 解码器根据外观和结构嵌入生成图像
  - 对数据集中的高分辨率子集进行生成新个体，避免相似个体引起的歧义以及低分辨率导致的歧义
  - 仅选择一个目标图像为整个源图像提供外观嵌入
----------
![](imgs/Going%20Beyond%20Real%20Data%20A%20Robust%20Visual%20Representation%20for%20Vehicle/Copy%20&%20Paste.png)
- **Copy & Paste**
  - 将真实目标放到仿真图像的背景中，生成新的样本
  - MaskRCNN:通过实例分割从真实图像中分割出车辆，从仿真图像中分割出背景
  - DeepFill v2:在去除前景的空白区域进行图像修复
  - 无缝图像克隆(seamless image cloning):融合前景和背景图像

# 代码中有的数据增强
- [AutoAugment](https://zhuanlan.zhihu.com/p/62481481)
![](imgs/Going%20Beyond%20Real%20Data%20A%20Robust%20Visual%20Representation%20for%20Vehicle/AutoAugment.png)
  - shear:沿轴按比例裁剪
  - Translate:沿轴按比例平移
  - Rotate:旋转
  - AutoContrast:最暗的设为black,最亮的设为white
  - Invert:反转图像的像素
  - Equalize:均衡图像直方图
  - Solarize:反转幅度阈值以上的像素
  - Posterize:修改每个像素的位数
  - Contrast:对比度.0:灰度图,1:原始图像
  - Color:调整图像的色彩平衡,类似彩色电视机上的控件
  - Brightness:调整图像的亮度
  - Sharpness:调整图像清晰度
  - Cutout:将随机正方形色块设置为灰色
  - Sample Pairing:在不更改标签的情况下，将图像与具有权重的另一幅图像（从同一小批量中随机选择）线性相加
- 随机旋转
- 填充
- 随机裁剪
- 平移
- 旋转
- 随机裁剪并调整大小
- 随机水平翻转
- 随机擦除
- 随机更改亮度、对比度

# 训练
- 输入:图片
- 标签:id
- 损失:交叉熵损失


# Representation Learning
- 方式一、采用Person_reID_baseline_pytorch的网络进行特征提取。Person_reID_baseline_pytorch采用新的分类器替换ImageNet原始分类层。新的分类器包含fc1+bn+fc2，fc2可看作预测输出类别时的线性分类器，fc1输出512维特征。
- 方式二、sophisticated re-id network architecture
![](imgs/Going%20Beyond%20Real%20Data%20A%20Robust%20Visual%20Representation%20for%20Vehicle/network%20architecture.png)

# Optimization Functions
- 定义:
  - N:数据集中车辆id个数
  - x:给定的图像
  - y:图像对应的标签
- 交叉熵损失:类别预测
$$ loss_{c e}=-\sum_{i=1}^{N} p_{i} \log \left(\hat{p}_{i}\right) $$
where $p_i$ is the ground truth label of the input sample x.$p_i$ = 1 if i equals to the ground truth label y, else $p_i$ = 0. $\hat{p_i}$ is the predicted probability.

- Triplet loss:正样本距离小，负样本距离大
$$ loss_{\text {ranking}}=\left[D_{a p}-D_{a n}+m\right]_{+} $$

# 负样本挖掘
- 第一阶段，负样本挖掘：从小批量中随机抽取50%的图像，然后选择最相似的负样本对组成三元组
- 第二阶段，常规Triplet loss训练

# 辅助信息学习
- 车辆re-id容易被方向相似的不同样本混淆，采用方向分类模型预测车辆的方向，并在后处理阶段根据方向相似性来depress some vehicle pairs.
- 训练可识别相机的模型预测可以捕获图像的相机。
- 摄像机感知模型+方向感知模型 在后处理中进行摄像机验证

# 后处理
![](imgs/Going%20Beyond%20Real%20Data%20A%20Robust%20Visual%20Representation%20for%20Vehicle/pipepine.png)

- 图像对齐:原始数据集边界框较大，背景的影响可能较大，采用MaskRCNN重新检测车辆，在原图像和裁剪图像之间对车辆表示进行平均
- Model Ensemble:将12个不同模型归一化特征连接为最终的视觉representation
- Query Expansion & Re-ranking
  - The query feature is updated to the mean feature of the other queries in the same cluster.在计算the mean feature时不考虑低分辨率的图像的特征
  - 采用Re-ranking方法细化最终结果
- Camera Verification
  - 利用相机验证进一步去除一些hard-negative样本
- Group Distance
  - 对同一相机下的车辆进行跟踪得到小段轨迹
  - 采用图库扩展:将gallery特征更新为轨迹中其他图像的均值特征
  - 降低来自不同轨迹的相似性得分
