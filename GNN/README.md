# 这里是图和图神经网络学习的部分
## 图神经网路初探
### 什么是图神经网络
图神经网络基于图结构的的深度学习方法，近期被广泛应用在图像和nlp领域，图神经网络作为神经网络的扩展，它有很天然的优势去处理图类型的数据结构
在图中，每个节点的特性由它本身和响铃节点来定义，网络通过地柜的聚合和转化相邻节点的向量来计算节点的变化。  
在论文《Relational inductive biases, deep learning, and graph networks》中作者定义了图神经网络的通用结构，也就是现在的图神经网络的开山鼻祖
，在这个框架中定义了和扩展了各种图神经网络，GN framework的主要计算单元是GN block，一个图到图的模块，它的输入是一个图
在图结构上进行计算然后得到的输出仍然是图结构的输出。  
在GN framework下，一个图可以被定义为一个三元组$$G=(u,V,E)$$,$$u$$是图的全局表示，$$V={Vi},i=1:N^v$$代表图中$$N^v$$节点的集合，$$Vi$$
表示节点i，$$E={(e_k,r_k,s_k)},k=1:N^e$$表示图中$$N^e$$条边的表示，$$e_k$$为边的表示，$$r_k$$,$$s_k$$分别代表的是边的接收点和发送点。  
每一个GN block包含三个更新函数$$\Phi$$以及三个聚合函数$$\rho$$：  
![gnn1](https://github.com/opprash/braveRL/blob/master/datas/gnn1.png)  
上式中$$E'_i={(e_k,r_k,s_k)},r_k=i.k=1:N^e$$,$$V’={V_i},i=1:N^v$$,$$E'=U_iE_i'={(e'_k,r_k,s_k)},k=1:N^e$$。  
其中$$Φ^e$$应用于图中每条边的更新，$$Φ$$应用于图中每个节点的更新，$$Φ$$则用来更新图的全局表示；$$ρ$$函数将输入的表示集合整合为一个表示，该函数设计为
可以接收任意大小的集合输入，通常可以为加和、平均值或者最大值等不限输入个数的操作。  
当一个GN block得到一个图$$G$$的输入时，计算通常从边到节点，再到全局进行。下图给出了一个GN block的更新过程。  
![gnn2](https://github.com/opprash/braveRL/blob/master/datas/gnn2.png)  
一个GN block的计算可以被描述为如下几步：  
1. $$\Phi^e$$更新每一条边，输入参数为边表示$$e_k$$，接收和发送节点的表示$$v_r_k$$和$$v_s_k$$以及全局表示$$u$$，输出为更新过的边表示$$$e'_k$； 
2. $$ρ^(e \leftarrow v)$$用来有同一个聚合接收节点的边的信息，对节点i得到所有入边及邻近节点信息整合$$e'_i$$,用于下一步节点的更新；
3. $$\Phi^v$$更新每一个节点，输入为上文中的$$e'_i$$，节点表示$$v_i$$,以及全局表示$$u$$，输出为更新过的节点表示$$v'_i$$；
4. $$ρ^(e \leftarrow u)$$聚合图中所有边的信息得到边信息整合e(帽)；
5. $$ρ^(v \leftarrow u)$$通过聚合图中所有节点信息得到节点的信息整合$$\dot{v}$$（帽）；
6. $$\Phi^u$$更新全局的表示，输入为边信息整合e(帽)，节点信息整合$$\dot{v}$$（帽），以及节点表示$$u$$，输出为更新过的全局表示$$\dot{u}$$。  
可以通过上述的更新步骤来套用目前的图神经网络算法，当然这里的步骤的顺序并不是严格固定的，比如可以先更新全局信息，再更新节点信息以及边的信息。  
### 为什么要使用图神经网络
图神经网络有灵活的结构和更新方式，可以很好的表达一些数据本身的结构特性，除了一些自带图结构的数据集（如Cora，Citeseer等）以外，图神经网络目前
也被应用在更多的任务上，比如文本摘要，文本分类和序列标注任务等，目前图神经网络以及其变种在很多任务上都取得了目前最好的结果。  
比较常见的图神经网络算法主要有Graph Convolutional Network（GCN）和Graph Attention Network（GAT）等网络及其变种。  
### GCN
GCN的计算公式如下：  
![gnn3](https://github.com/opprash/braveRL/blob/master/datas/gnn3.png)  
可以看到在每一层图网络中，每个节点通过对邻近节点的信息聚合得到这层该节点的输出。$$H^(l)$$表示网络的第l层，$$σ$$代表非线性激活函数，$$W^(l)$$为该层的权重函数。$$D$$和$$A$$分别代表度矩阵以及邻接矩阵，~符号代表对每一个节点加上加上一个自环，即$$A$$
帽为(~),$$I$$为单位矩阵，这里由于邻接矩阵是没有进行正则化的，所以通过$$D^-1/2AD^-1/2$$帽为(~)使得结果中的每一行的和都为1.  
![gnn4](https://github.com/opprash/braveRL/blob/master/datas/gnn4.png)  
相比于前文给出的GCN基于邻接矩阵的公式定义，GCN的公式可以被更简洁的定义为以下两步：  
* 对于节点$$u$$,首先将节点邻居表示$$h_v$$聚合到一起，生成中间表示$$h_u$$帽为(^)， 
* 对于$$h_u$$帽为(^)通过一层非线性神经网络层$$h_u=f(W_uh_u)$$函数中的h的帽为(^)。