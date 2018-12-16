
---

layout:     post # 使用的布局（不需要改）
title:      论文总结：A strong baseline for question relevancy ranki  # 标题 
subtitle:   #副标题
date:       2018-11-23  # 时间
author:     zyz  # 作者
header-img: #这篇文章标题背景图片
catalog: true  # 是否归档
tags:        #标签
        - 论文

---
## 1. 论文动机
在QQrelevence问题（两个问题的相关性检验）上，许多复杂的模型经常用一些十分常规的IR模型做对比，却并没有取得很大的提升。而谷歌提出了一种基于IR特征的简单的多任务神经网络模型（利用其他任务QA，fake news等作为辅助训练）作为baseline，取得了很好的效果。（啧啧，满脑子都是谷歌一脸中二的甩出模型：“好好看好好学，效果比这差的就不要拿出来丢人现眼了”）

## 2. 论文模型
将两个输入句子（query）分别用三种形式表示，averaged word embeddings（词向量平均值, binary unigram vectors（2ngram）, and trigram vectors（3ngram）。ps：看了下代码，用的是1-3ngram的tf-idf和3char-ngram的tf-idf表示作为向量
计算了五种距离：cosine distance，Manhattan distance，Bhattacharya distance，Euclidean
distance ，Jaccard index（只能用于计算ngram），实际代码中还用了互信息距离。
将各种问题表述方法计算出的距离作为features 送入一个mlp层，然后经过一个公共的隐层，之后是一个特定任务的隐层和一个特定任务的分类器（模型利用了其他任务作为副主任务训练，增加了模型的效果和泛化能力

![image_1cstt92j58ll11cmsvv1v5vej99.png-100.3kB][1]

## 3. 实验结果
![image_1csttfjouufkpmki55s9f1lkr13.png-208.9kB][2]
可以看到，实验结果基本与最好的模型相似
![image_1csttgpk8aag1lndcpjb8bqmg1g.png-60kB][3]
利用辅助任务提升了小幅度的模型效果，不过训练的稳定性得到了很大的提高

## 4. 总结
学习了各种向量距离的计算，可以发现很多时候只需要对基础的IR特征采取简单的操作就可以得到很好的效果，未必需要复杂的模型。多任务辅助用来提高模型的稳定性和性能是个很值得的学习的方法

## 5. 向量距离的总结(主要参考自[博客](https://blog.csdn.net/v_july_v/article/details/8203674))

1. 欧式距离
两点之间的直线距离
$$ D(x,y) = \sqrt{\sum_i(x_i - y_i)^2} $$
标准欧式距离: $ D(x,y) = \sqrt{\sum_i\frac{(x_i - y_i)^2}{s_i}} ,其中s_i为第i维的标准差 $
2. 曼哈顿距离
有时也叫街区距离，当两点之间直线不可达时的距离测量方式
$$ Man(x,y) = \sum_i|x_i - y_i| $$
3. 切比雪夫距离
$$ D(x,y) = \max{|x_i - y_i|} $$
数学上，可以将切比雪夫距离定义为 
$$ D(x,y) = \lim_{k \to +\infty}(\sum_i(|x_i - y_i|)^k)^{1/k} $$
类似国际象棋中，王可以向八个方向行走，斜线行走一个单位的长度与直线相同，这是衡量两点之间的距离可以用切比雪夫距离
4. 马氏距离
X为n个向量样本，S为协方差矩阵，$\mu$为均值向量，则
$$ D(X) = \sqrt{(X-\mu)^TS^{-1}(X-\mu)}$$
$$ D(x_i,x_j) = \sqrt{(x_i-x_j)^TS^{-1}(x_i-x_j)} $$
当各个维度服从独立分布的时候，马氏距离相当于标准欧式距离
5. cos距离
衡量的是两个向量的夹角大小，夹角越小，说明两个向量越相似
$$ cos(x,y) = \frac{\sum_i x_i * y_i}{||x||+||y||} $$
6. 巴式距离
衡量两个分布的相似性
$$ Bha(x,y) = -ln(\sum_i\sqrt{x_iy_i} )$$ 
严格来说x_i,y_i 应该是概率，$Bha(P,Q) = -ln(\sum_{x \in X}\sqrt{P(x)Q(x)})$

7. 雅卡尔指数
设两个集合为A，B，每个元素的取值均为0或1，$M_{11}$为AB均取1的元素个数，$M_{10}$是A1B0的元素个数
 $M_{01}$为A0B1
$$ J(A,B) = \frac{M_{11}}{M_{11} + M_{10} + M_{01}} $$
衡量了两个集合的重叠性，雅卡尔指数严格上不是距离，因为不具备对称性，而且不关注全为0的点
8. kl散度（相对熵）
$$ P(p|q) = \sum_ip(x_i)log\frac{p(x_i)}{q(x_i)} $$
$$ 相对熵=交叉熵-熵=H(p,q) - H(p) = -\sum_{x \in X} p(x)log(q(x))+\sum_{x \in  X} p(x)log(p(x))$$
衡量q分布与p的相似程度
9. 互信息
$$ I(x,y) = p(x,y)log\frac{p(x,y)}{p(x)p(y)} $$ 
$$ I(X;Y) = \sum_{x \in X}\sum_{y \in Y}I(x,y) $$
$$ I(X,Y) = H(X) - H(X|Y) = H(Y) - H(Y|X) = H(X) + H(Y) - H(X,Y) $$ 
互信息相当于信息增益
![image_1csvhs0fk33g4vc6mk114ic4bf.png-151.4kB][5]
互信息存在只能度量两个分布相关情况的大小，没办法衡量两者不同部分的情况（或者说，互信息无法很好的衡量一个分布是否能很好的表示另一个分布的能力，具体例子可以参考[这篇文章](https://www.douban.com/note/621588501/)
解决办法时通过标准化之后的互信息：
![image_1csvl5ljlm021eqa53g16t614b0s.png-58.2kB][4]



  [1]: http://static.zybuluo.com/zyz0/6f0mpogj9ou2bo8qm3qmrlpi/image_1cstt92j58ll11cmsvv1v5vej99.png
  [2]: http://static.zybuluo.com/zyz0/cumgzrolsst0z0vjuvcpb18z/image_1csttfjouufkpmki55s9f1lkr13.png
  [3]: http://static.zybuluo.com/zyz0/myllvt2r6no1sgspg1n8z4si/image_1csttgpk8aag1lndcpjb8bqmg1g.png
  [4]: http://static.zybuluo.com/zyz0/hf9af44h3sw21qenmffuykmm/image_1csvl5ljlm021eqa53g16t614b0s.png
  [5]: http://static.zybuluo.com/zyz0/3vxqfpfm4bxshhswn3d4omc1/image_1csvhs0fk33g4vc6mk114ic4bf.png