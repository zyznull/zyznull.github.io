---

layout:     post # 使用的布局（不需要改）
title:      learn to rank总结# 标题 
subtitle:   #副标题
date:       2018-12-16  # 时间
author:     zyz  # 作者
header-img: #这篇文章标题背景图片
catalog: true  # 是否归档
tags:        #标签
        - 论文,机器学习,nlp

---

## 一. Learn to rank介绍
learn to rank是一个很经典的任务，假设我们有一个初始的候选文档集，给定一个查询，我们希望能得到与它相关的文档，并依照相关性高低排好序。本文主要介绍基于文本的一些机器学习方法在该任务上的应用。

## 二. 传统方法
### 1. tf-idf 与 bm25
首先，我们来看一下传统的信息检索方法怎样解决这样一个问题。
tf-idf基于一个很符合直觉的思想，一个词与一篇文档的相关性，用这个词在文档中出现的次数表示，即假设一个词在一篇文档中出现的次数越多，这个词与这篇文档越相关。但考虑到有些词可能很常用所以在每篇文档里都经常出现出现，所以还要考虑这个词在所有文档中出现的频率，用逆文档频率idf表示，公式为logN/（df+1)，df为这个词在多少篇文档中出现，N为总共的文档数，df+1是为了为了防止df为0。关于tf-idf，可以看一下[这篇论文]，从信息论的角度解释了tf-idf的合理性。
然而tf-idf忽略了这样一个问题，一篇文档越长，它出现查询相关词的概率越高，所以为了消除文档长度的影响，后来又提出了[bm25][bm25]算法。
$$  bm25 = idf * \frac{tf*(k+1)}{tf + k(1 - b + b * dl/avgdl} $$


## 三. 机器学习方法
本文只是对各个模型做一个简短的介绍，具体的模型描述还请参考原论文。
### 1. Lambdamart
lambamart 是一个基于mart(gbdt)的机器学习方法，用决策树作为基学习器，与gbdt的主要区别是提出了一个基于评测方法的lambda参数用于提高学习速率，具体可以参考[这篇文章](https://liam.page/2016/07/10/a-not-so-simple-introduction-to-lambdamart/)
### 2. [DRMM][DRMM]
从DRMM开始，下面介绍的算法就都是基于文本本身的算法了，不需要手工计算特征。
深度学习来作LTR任务通常有两种思路，一种是交互式的，一种是表示式的。交互式的方式即先利用文档与查询计算出一些交互特征如相似度等等，在此基础上运用深度网络进行学习建模，而表示式的是先将查询与文档各自学习一个表达，之后计算两个表示的相似度。
DRMM就是一个典型的交互式模型，它先计算每个查询词与文档的相似度，对每个词统计一个相似度直方图（有多少相似度在0-0.1，多少在0.1-0.2，等等，值得关注的是相似度为1要单独列出来）,文中为了防止直方图数据分布差异过大，选择了对统计数据取log，然后将每个词的相似度直方图送入一个神经玩过学习得到一个分数，最终将所有的分数相加，作为最终的得分。
![image_1cur09t0qs4u1mgdnkg112at1u9.png-81.4kB][1]
### 3. [PACRR][PACRR]
PACRR主要思想是应用CNN作n-gram卷积，提取词组特征。
首先，计算query 与 document每个单词的余弦相似度，为了确保矩阵大小固定，提出了两种方案：
PACRR-first 直接将超出部分截断，不足补0。
PACRR-kwindow，计算n个矩阵，分别用大小为2-n的窗口平行移动，取最大的k=ld/n个相连接。用来做n-gram卷积
之后在矩阵上做用大小为2-lg，每个大小nf个卷积核和做卷积（PACRR-frist步长为（1，1），kwindow上不上为（1，n））结合原始矩阵，得到lg个lq*ld*nf的矩阵，在nf上做max-pooling，在ld上做k-maxpooling。得到lq*lg*ns的矩阵，将每个query的lg*ns矩阵展平，链接上softmax后的idf值，送入lstm，将最终得分作为相似度
![image_1cur0aaa35n91g129qn1pht1rblm.png-62.8kB][2]
### 4. [DeepRank][DeepRank]
DeepRank的思路是模拟人类考虑相似度时候的步骤（1）找到可能相似的段落（2）判断是否相似（3）综合考虑整篇文章的相似性
deeprank将上述步骤转化为以下策略（1）找到文档中出现query词的位置，以查询词为中心向前后截取长度为2k+1的上下文作为一个段落（2）计算query与段落的相似度矩阵，并拼合query词向量和doc词向量，送入一个2-DGRU或者一个CNN用来得到一个段落的相似性(3)将每个词的所有段落相似性得分拼合位置信息送入一个GRU来学习全局相关性，并将所有词的相关性相加得到最终的结果
![image_1cur0ele61uoimb61mak1e8t1uld13.png-118kB][3]
### 5. [Hint][Hint]
Hint思路大体与DeepRank相似，不过它抛弃了寻找相关段落的方法，它直接将文章切分成多个段落，计算每个段落的相似性，然后，用k-max-pool等选出相关性最大的几个值，利用神经网络或者rnn进行学习
![image_1cur0f17crpp1no21gv3un3mur1g.png-61.3kB][4]
### 6. [NPRF][NPRF]
NPRF考虑到了其他相关文档可能的相关性对文档的影响，所以提出了一个框架，首先，用relq计算出query与每篇文档的相似性，之后，利用reld计算相关文档与目标文档的相似性，利用relq的结果线性变换后作为权重相加
![image_1cur1ebn1mmh109b18f42s1ovn2n.png-6.3kB][5]
![image_1cur153drft7ogmenn1uotaos2a.png-116.9kB][6]
### 7. 损失函数
简单来介绍一下LTR任务常见的损失函数的计算方法。
1）pointwise
将相关与否当作一个分类器学习，可以通过在最后的相关性得分后增加一个LR等，然后利用常见的损失函数如交叉熵等来学习。
2）pairwise
pointwise算法只考虑了单篇文档，而我们考虑的应该是整个排序的好坏，所以pairwise方法一定程度上考虑了排序相关的信息，将相关性转化为一个偏序对比较的任务，即比较两篇文档相关性得分的差值，常见的方法有Ranknet，margin loss等。
3）listwise
评估整个排序的好坏，但由于梯度无法直接计算，需要做一些改动，常见的有ListMLE，SofRank等。
## 四. 评测指标
最后，考虑如何评测一个排序的好坏呢？传统的准确率等方法不够使用，所以通常采取以下指标来进行评测
### 1. P@N
最直接的做法，看排序排在topk的几篇文档是否相关，用相关文档数/k来作为分数。
### 1. MAP
一个好的排序，越相关的文档应该排的越靠前。假设我们有五篇相关的文档，它们分别排在第1，2，6，8，20位置，则一个查询相应的ap得分为(1/1 + 2/2 + 3/6 + 4/8 + 5/20) / 5 = 0.65，map就是所有查询ap的平均值
### 2. NDCG
还是基于越相关的位置越靠前的思想，这次我们只比较topn篇文档。首先看dcg:
![image_1cur25so21q2dbe410h11v6417jh44.png-5.7kB][7]
rank就是每篇相关文档所在的位置，可以看到，相关的文档越靠前，分母越小，dcg值就越大。idcg是指理想状态下dcg的得分，用它来归一化dcg得分。最终的ndcg得分为所有查询ndcg得分的平均值，常用的有ndcg@20，ndcg@10。



[bm25]:https://dl.acm.org/citation.cfm?id=188561
[DRMM]:https://arxiv.org/abs/1711.08611
[PACRR]:https://arxiv.org/abs/1704.03940
[DeepRank]:https://arxiv.org/abs/1710.05649
[Hint]:https://arxiv.org/abs/1805.05737
[Nprf]:https://arxiv.org/abs/1810.12936


  [1]: http://static.zybuluo.com/zyz0/vdjunhr1idvyd908gzp736tw/image_1cur09t0qs4u1mgdnkg112at1u9.png
  [2]: http://static.zybuluo.com/zyz0/flfuhzlibmnt5nul1b53txi7/image_1cur0aaa35n91g129qn1pht1rblm.png
  [3]: http://static.zybuluo.com/zyz0/6zz3krsr6hfca47y93u132jz/image_1cur0ele61uoimb61mak1e8t1uld13.png
  [4]: http://static.zybuluo.com/zyz0/g3w9w6hcmw9hvvlr33xeybuq/image_1cur0f17crpp1no21gv3un3mur1g.png
  [5]: http://static.zybuluo.com/zyz0/eosijwrrocljqone1alhjndj/image_1cur1ebn1mmh109b18f42s1ovn2n.png
  [6]: http://static.zybuluo.com/zyz0/by38brjrqrdx8fee4jj11v2o/image_1cur153drft7ogmenn1uotaos2a.png
  [7]: http://static.zybuluo.com/zyz0/y6zl2qn1xoiqu6abrdkkqiyn/image_1cur25so21q2dbe410h11v6417jh44.png