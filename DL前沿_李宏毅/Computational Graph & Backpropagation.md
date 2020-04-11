# Computational Graph & Backpropagation
## Computational Graph 全连接
- ![](imgs/Computational-Graph-&-Backpropagation/计算图.png)
- ![](imgs/Computational-Graph-&-Backpropagation/Chain_Rule.png)
- ![](imgs/Computational-Graph-&-Backpropagation/Computational_Graph_example.png)
- ![](imgs/Computational-Graph-&-Backpropagation/Computational_Graph_反向传播求导数.png)
- ![](imgs/Computational-Graph-&-Backpropagation/Computational_Graph_权值共享情况.png)
  - 把每个x元素当成不同的x,所以这里v对x的导数是x
- 反向传播_传统方法(麻烦)
  - ![](imgs/Computational-Graph-&-Backpropagation/前向传播_反向传播.png)
- 计算图
  - ![](imgs/Computational-Graph-&-Backpropagation/反向传播.png)
- 向量求导
  - ![](imgs/Computational-Graph-&-Backpropagation/Jacobian_Matrix.png)
- 假设cost为交叉熵
  - ![](imgs/Computational-Graph-&-Backpropagation/假设cost为交叉熵.png)
- 激活函数求导
  - ![](imgs/Computational-Graph-&-Backpropagation/激活函数求导.png)
    - 这里的上标不是次方数,而是用于分辨变量的一个数
- 神经元求导
  - ![](imgs/Computational-Graph-&-Backpropagation/神经元求导.png)
- 权重求导
  - ![](imgs/Computational-Graph-&-Backpropagation/权重求导.png)
    - 拉直,看成向量,求雅可比矩阵
- cost对神经元某一项求导
  - ![](imgs/Computational-Graph-&-Backpropagation/cost对神经元某一项求导.png)
## Computational Graph RNN
- ![](imgs/Computational-Graph-&-Backpropagation/计算图_RNN.png)
  - 梯度下降可能会出现问题,导数连乘多次,多用LSTM、GRU