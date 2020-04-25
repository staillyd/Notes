# Language Modeling
- ![](imgs/Language-Modeling/语言模型.png)
  - 语音识别:不同句子具有相同发音，语言模型可以识别到底是哪个句子
  - 语句生成

## N-gram
- ![](imgs/Language-Modeling/N-gram.png)
  - 数学简化假设
  - [N-gram 详细介绍](https://blog.csdn.net/songbinxu/article/details/80209197)
- ![](imgs/Language-Modeling/Challenge_of_N-gram.png)
### 可将N-gram学习的参数看作是学习一个矩阵
- ![](N-imgs/Language-Modeling/N-gram%20Matrix.png)

## NN-based LM
- ![](imgs/Language-Modeling/NN-based_LM_training.png)
- ![](imgs/Language-Modeling/NN-based_LM_predict.png)
  - NN为什么可以比N-gram效果好?
  - 用的参数更少
### 将NN-based LM看作是学习两组矩阵
- ![](imgs/Language-Modeling/NN-based%20LM%20Matrix.png)
  - 两组矩阵h,v:通过梯度下降得到
  - 真值是数据集中的P(输出|输入)
- ![](N-imgs/Language-Modeling/N-gram%20Matrix.png)
- ![](imgs/Language-Modeling/NN_LM.png)
  - N-gram:(m,n):假设输入编码为(m,1)、输出编码为(n,1)
  - NN:(m,x)+(x,n)

## RNN-based LM
- ![](imgs/Language-Modeling/RNN_based_LM.png)
- ![](imgs/Language-Modeling/RNN-Based_LM.png)
  - RNN参数比NN更少
- ![](imgs/Language-Modeling/RNN-based%20LM_training.png)