# Abstract
- 大多数视频异常检测的缺陷
  - 摄像机固定,视频具有静态背景。不适用于车载摄像机
  - 将异常事件,如交通违章、自然驾驶场景的事故等,取用于训练的异常类别进行分类,这需要大量的标注数据
- 本文针对车载摄像头,通过预测交通参与者的未来位置检测异常,使用三种不同的策略监控预测的准确性和一致性指标
# Introduction
- 交通事故数据稀少,可训练识别正常安全路况模型,区分是否异常。这种方式无法准确识别发生了哪些异常。
- 本文假设通过查找预测位置和实际位置之间的偏差检测异常道路信息。(意外道路事件,如汽车撞击其他物体,会导致物体速度或位置突然发生意外变化)。
- 最相关的文章:[Future Frame Prediction for Anomaly Detection – A New Baseline](paper/Future%20Frame%20Prediction%20for%20Anomaly%20Detection%20–%20A%20New%20Baseline.pdf)
  - 该论文预测未来的RGB帧，然后寻找预测帧与实际帧的偏差
  - 本文先进行目标检测,然后进行轨迹预测
# Related Work
## 轨迹预测
- 静态相机
  - 考虑行人相互作用的Social LSTM->GAN
  - 考虑multimodal maneuver conditions的运用于车辆轨迹预测的Social pooling
  - 使用attention mechanisms获取scene context去协助轨迹预测的方式
  - 结合RNN和条件变分自编码器(CVAEs)生成mutimodal predictions,然后选择得分最高的模型方式
  - etc.
- 动态相机
  - 通过贝叶斯LSTM,对车载摄像头数据,预测行人位置,观察不确定性
  - 使用不同种类的cues输入到卷积-反卷积(Conv1D)中,预测行人未来位置
  - multi-stream RNN Encoder-Decoder架构,将过去车辆位置和图像特帧作为输入,预测车辆位置
## 视频异常检测
- 通常使用无监督学习方法重建正常训练数据
  - 3D convolutional Auto-Encoder对正常帧进行建模
  - Convolutional LSTM Auto-Encoder得到正常帧和运动模式
  - 时间相干稀疏编码TSC,保留正常事件和异常事件中帧之间的相似性
  - 通过寻找预测的未来框架和实际框架之间的差异来检测异常。(动态自动驾驶车运动剧烈,难以重建当前帧或者未来的RGB帧)
- 有监督学习,且做出不现实的假设
  - 动态摄像头dynamic-spatial-attention RNN交通事故检测
  - spatio-temporal action graph建立对象间的时空关系潜在图结构
# 本文
- 采用非异常的驾驶视频训练模型,学习物体和自我运动的正常模式,识别偏差。
- 考虑到自我运动对感知对象的影响,将未来的自我运动预测模块作为附加输入。
- 根据最后几帧预测对象当前位置,根据三种不同的异常检测策略确定是否发生了异常。
## **FOL**
- ![](imgs/Unsupervised%20Traffic%20Accident%20Detection%20in%20First-Person%20Videos/FOL.png)
### Bounding Box Prediction
- 上图上半部分
### Ego-Motion Cue
- 上图下半部分
### Missed Objects
- 正常运动模式目标的轨迹预测,感觉和deepsort类似,将deep改为FOL
  - ![](imgs/Unsupervised%20Traffic%20Accident%20Detection%20in%20First-Person%20Videos/FOL-Track%20Algorithm.png)
- 异常对象若采用FOL-track进行轨迹预测,边界框区域可能会完全移位导致不准确的预测,采用之后的度量标准消除一些误报
## 交通事故检测
1. Predicted Bounding Boxes - Accuracy
  - <details>
    <summary>原文</summary>
    The FOL model predicts bounding boxes of the next $\delta$ future frames, i.e., at each time t each object has $\delta$ bounding boxes predicted from time t − $\delta$ to t − 1, respectively.
    </details>
  - 取之前$\delta$帧预测当前帧的$\delta$个box的平均值作为当前box
  - 取当前帧的所有观测目标的box与预测的box之间iou的平均值
  - 取1-iou平均值作为聚合异常分数$L_{bbox}$
  - $L_{b b o x}=1-\frac{1}{N} \sum_{i=1}^{N} \mathbf{I} \mathbf{o} \mathbf{U}\left(\left(\frac{1}{\delta} \sum_{j=1}^{\delta} \hat{Y}_{t, t-j}^{i}\right), Y_{t_{0}}^{i}\right)$
2. Predicted Box Mask - Accuracy
  - Mask有什么作用?看上去只是计算t-1预测当前的box与当前box的iou
3. Predicted Bounding Boxes - Consistency
  - 外观和相互遮挡的变化,会影响异常参与者的检测跟踪
  - 假设异常的视觉和运动特征不仅在异常后发生,而且伴随明显的事前事件
  - 期望通过计算多个先前帧的未来对象定位输出的一致性来检测异常,缓解不准确的检测和跟踪的影响
  - $L_{p r e d}=\frac{1}{N} \sum_{i=1}^{N} \max _{\left\{c^{x}, c^{y}, w, h\right\}} \mathbf{S T D}\left(\hat{Y}_{t, t-j}\right)$
  - 计算先前多帧预测的当前帧box的$c^x,c^y,w,h$中最大的标准差
## EXPERIMENTS
- A3D异常数据集(测试集)
  - 标签:事故开始时间、所有交通工具恢复正常运动状态的时间
- 使用HEV-I数据集训练模型、COCO数据集训练的Mask-RCNN生成box、deep-sort进行跟踪得到ID
- ORB-SLAM2计算里程、FlowNet2计算光流