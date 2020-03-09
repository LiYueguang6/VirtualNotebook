# QTune: 一种基于深度强化学习的查询感知数据库调优系统
论文发表于VLDB 2019

Q：什么是查询感知数据库？
A：query-aware database，不是查询感知数据库，而是数据库查询感知调优

Q：什么是NP问题？
A：

Q：DDPG相比于DQN、传统机器学习强在哪？
A：DDPG 在连续动作空间的任务中效果优于DQN而且收敛速度更快

Q：query特征化，向量化的好处
A：数据化才能计算使用

Q：query聚类怎么做的
A：随便做

Q：Double-state是什么意思？
A：增加了Predictor模块，预测query将会得到的△S，加上当前的S

Q：和CDBTune相比的好处？
A：

Q：初始的S值是什么？S'全是基于Predictor网络生成的吗？
A：S就是当前的状态值，S'是状态观察值S和Predictor预测的△S的和

Q：action是什么？S具体是什么？
A：action是参数值的向量（而不是参数增加或者减小或者不变）。

# Abstract
数据库调优是一个NP难问题，而且现有方法有很多限制。
首先，DBA们不能调大量的在不同环境下的数据库实例（例如，不同的数据库vendors？供应商？）
其次，传统的机器学习方法也找不到好的参数，或者太依赖难以获取的高质量的数据样本。
第三，他们只支持粗粒度的调优（例如workload层级的调优），而不能提供细粒度的调优（例如query层级的调优）

为了解决这些问题有了这个查询感知数据库调优系统QTune，使用了深度强化学习模型
流程：1、通过考虑大量的特征，先将queries特征化。2、然后将query特征喂入DRL模型中选择合适的参数。使用了DDPG模型它可以有效的利用actor critic网络，基于query特征向量和数据库states来调优。提供了三种粒度：query、workload、cluster（聚类不是集群，介于query和workload之间）。

# 1. Introduction
数据库几百个参数，参数空间大
1.DBA的只调一部分参数
2.DBA时间长，无法调多数据库实例
3.DBA只熟悉一个数据库
提及：
BestConfig：启发式搜索
OtterTune：高斯过程回归
CDBTune：DQN

query层级的调优，会获得低时延，但是吞吐量低。因为他不会并行的运行这些query。
workload层级的调优，会获得高吞吐，但是时延大。这样就没法为每一个query推荐一个优秀的参数。
cluster层级的调优，会将这些query聚类成几个组，为每个组的query找到一个好的参数。高吞吐、低时延。它可以为一个类别的query找到一个好的吞吐，并且并行的执行。（同时执行）
又权衡了吞吐量和时延、又提供了不同粒度的调优。
聚类用了深度学习，通过query适配的最佳参数的相似度
创新点：
1.基于query（第二章）
2.使用SQL特征将query特征化（第三章）
3.DDPG（四）
4.深度学习分类（第五章）
5.广泛的实验，在不同的query workload上和不同的数据库上，实验结果表明效果很好（六）

# 2. System Overview
query层级：对每一个query调优，执行这条query，请注意，会话级旋钮（例如，bulk write size）当前可以针对不同的查询进行调整，而系统级旋钮（例如，working memory size）不能同时进行调整，因为当我们针对查询调整这些旋钮时，系统无法处理其他查询。
workload层级：它为整个查询工作负载调整数据库旋钮。此方法无法优化查询延迟，因为不同的查询可能需要使用不同的最佳旋钮值。但是，这种方法可以获得高吞吐量，因为在设置新调整的旋钮之后，可以立即处理不同的查询。
cluster层级：分组，参数相近的一组，同时执行。

结构：
5个组件，

# 3. QUERY FEATURIZATION
query的向量化
1）首先要获取query的信息，例如query涉及到哪些表
2）要获取query的执行成本，例如查询成本和join成本
3）对query和成本信息进行统一的特征化，使不同query向量的每个特征有相同的含义
## 3.1 Query Information
query信息：（本文不采取3和4）
1）type（增删改查）
type查询类型很重要，因为不同的查询类型具有不同的查询成本（例如，OLTP和OLAP对数据库的影响不同）
2）表
表（数据量和数据结构）是个显著特征
3）attributes，属性
4）操作（选择、join、groupby等）

本文不采取3和4，因为：
1）query成本包含操作信息和操作成本
2）操作太具体了，会降低特征向量的泛化能力
3）属性和操作会频繁更新，需要不断重构模型。（6.1.2会比较考虑属性和操作的效果）
所以最后，本文提出了4+T的维度的向量，T是数据库里表的数量。4是增删改查。
## 3.2 Cost Information
Cost information成本信息包含了query的执行成本。然而，一个查询通常有许多可能的物理计划，每个计划都有不同的查询开销。预估查询成本是预估查询要涉及多少行，多少索引，预计时间。所以直接解析查询语句来提取查询成本是不现实的。相反，我们使用查询优化器生成的查询计划，它对每个操作都有一个成本估算。
由于每个数据库都有固定数量的操作，例如扫描、哈希连接、聚合，因此我们使用这些操作的成本来捕获成本信息。例如，在PostgreSQL中有38个操作。注意，一个操作可能出现在treeplan的不同节点上，同一个操作的代价应该和查询代价向量中相应的代价值相加，例如图3中（图里执行时间为totle-child），hash join的值等于两个hash join节点的代价之和。在得到查询成本后，我们通过减去平均值和除以标准差来规范化成本，总之，对于成本信息，我们维护了一个| P |维度向量，其中P是数据库中的操作集，而| P |是操作数。
## 3.3 Character Encoding
多个query vector构成一个query workload的向量。集合对于每个查询向量，我们需要考虑所有的查询类型和表，从而计算查询向量的并集。对于每个表来说，如果值为1，那么，就用这个歌表的行数代表这个数（1）。因此它可以捕获删除或者插入了多少行，这个信息，提高了系统的适应性。

组合向量

U：联合并集。就是所有满足条件的集合的并集。查询向量取并集，成本向量取和
我们讨论如何支持数据库的更新。数据库更新只能影响查询向量，因为成本向量是从优化器中实时计算出来的，优化器可以得到更新后的成本。对于查询向量，只有添加/删除表才会影响查询向量。为此，我们可以保留几个位置，以便将来捕获添加/删除表的更新。

# 4  DRL FOR KNOB TUNING
现有的DRL MODEL（16, 19, 12）不能利用查询特征，因为它们忽略了来自查询的环境状态的影响，并且我们提出了一种双状态的深度确定性策略梯度（DS-DDPG）模型来启用查询感知的调整。
## 4.1 DS-DDPG Model
5个组件：agent、environment、predictor、query2Vector、workload
状态是内部变量、reward是外部变量
predictor通过query的vector预测△S，Environment将△S和原始S结合起来，得到观测值S'，来模拟执行查询后的外部状态变量的度量。Agent基于S'来调内部变量参数，包含两个模块，Actor和Critic。Critic通过S'和action来得到Q值，Critic更新网络基于reward值，Actor更新网络基于Q值。actor生成一个action，Environment部署这个action，基于新参数配置的performance得到reward。

DS-DDPG是求解具有连续作用空间的最优问题的有效策略，借助于DS-DDPG同时训练Q值网络和action的策略网络。

神经网络适用于高维数据，所以也match数据库多参数调优这个问题
## 4.2 Training DS-DDPG
### 4.2.1 Predictor
Predictor旨在预测数据库的metrics变化，当query在数据库执行时。训练数据是
{v,S,I,△S}，通过v、S、I来预测△S（query级别的）
v：是一个query的向量
S：是metrics
I：是knobs
△S：是执行v之后的metrics
4个全连接层：输入层加两个隐藏层输出的张量限定在数据库metrics的范围内，并且生成代表预测的数据库metrics变化的vector。为了避免网络模型仅仅在学习线性变换，隐藏层使用ReLU作为激活函数，可以捕捉更复杂的模式（pattern）。正态分布初始化网络模型。
损失函数：

采用亚当优化器来训练Predictor。 

### 4.2.2 Actor-Critic 
这部分的目标是优化数据库参数配置。给定一个query workload，随机选取一个query子集生成一个样本workload。对这个样本，通过Query2Vector来得到特征向量，然后通过Predictor预测S'[1]，通过Actor得到A[1]，把这个action应用到数据库上，得到reward R[1]。在下一步中，基于S'[1]和action得到S'[2]，接着是A[2]、R[2]。通过迭代，得到一个三元组(S',A,R)集合T，直到平均奖励值足够好。


初始化模型，包括Environment和actor和critic网络，然后使用经验回放的方法训练actor和critic
使用集合T进行更新：
（1）更新actor网络的梯度值：

θ是actor的参数，Q是critic网络计算出来的Q值
（2）估计真实的参数调优action的价值Y。使用贝尔曼方程基于reward和Q值计算出Y
。。。
（3）基于Q和Y计算出loss。Critic网络更新的L如下：
。。。
i=t-1到i=1，执行这3步。（t是啥？）
算法终止于模型收敛（例如，性能提升小于阈值），否则选择下一个训练数据……

Target network
为了增强训练的稳定性，使用了这个

Reward Function
和CDBtune一样（改了一点）

Remark
首先，输入层大小决定了每一层神经元的数量。PostgreSQL 19->64
。。。。

Tuning



DDPG
# 5 Query Clustering
对于包括事务查询和分析查询的工作负载，如果用户希望优化时延，我们建议进行查询级别调整；如果用户希望提高吞吐量，我们建议进行工作负载级别调整。
对于纯分析查询，我们建议进行集群级调整，以平衡吞吐量和延迟。
聚类级优化的关键问题是（1）如何有效地为每个查询找到合适的配置模式；（2）如何基于配置模式对查询进行集群。本节研究这两个问题
## 5.1 Configuration Pattern
连续值离散化，设为-1,0,1，远超默认值的设为1，以此类推
避免维度太多，选用了模型常调的大约20个参数作为特征，但是3的20次方仍然很大
使用了深度学习来离散化（Vector2Pattern模块）
5层结构
## 5.2 Query Clustering
聚类算法不限，以DBSCAN[8]为例，根据距离和最小聚类数量聚类

# 6 Experiment
和ottertune、CDBtune、bestconfig进行对比。在大量的workload、数据库、硬件上进行测试。

实验部分就不多赘述了




