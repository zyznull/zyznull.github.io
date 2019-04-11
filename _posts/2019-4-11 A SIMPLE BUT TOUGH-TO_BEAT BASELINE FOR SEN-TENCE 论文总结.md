# A SIMPLE BUT TOUGH-TO-BEAT BASELINE FOR SEN- TENCE

标签（空格分隔）： 论文

---

## 介绍
介绍了一种无监督的句向量生成方法。简单来说就是对词向量的加权平均。并给出了理论推导，同时运用该理论解释了为什么word2vec中可以提升效果。


## 方法
首先，我们假设一句话有一个中心向量$c_s$,每个单词出现在这句话中的概率为:
$$ P(w|c_s) = \alpha p(w) + (1-\alpha)\frac{exp(<v_w, \tilde{c_s}>)}{Z} $$
$$ \tilde{c_s} = (1 - \beta) c_0 + \beta c_0$$
即每个词出现的概率和它的词频以及与中心向量的相似度有关，中心向量由两部分构成，一部分是这句话本身的语义向量，一部分是句子共有的语义向量。Z为normaliza参数，根据的定理，Z对于所有的句子是相似的。可以看作常数。
生成句向量的过程可以看作一个求最大似然估计的过程。即： 
$$ P(s) = \prod p(w|c_s) = \prod  (\alpha p(w) + (1-\alpha)\frac{exp(<v_w, \tilde{c_s}>)}{Z}) $$

令 $ f(c_s) = log P(w|c_s) $
则
![image_1d863r4k9v6s1j734s56bt5i9.png-19.3kB][1]
根据泰勒公式，我们有：
![image_1d863slb0vvt157s3ruh911fjgm.png-22.5kB][2]
令$ \alpha = \frac {1-\alpha}{\alpha Z} $
则$f(c_s) = \frac {\alpha <v_w,c_s>}{\alpha + p(w)} $
所以整个算法流程如下：
![image_1d86422171s5n1u507b7459plk13.png-104kB][3]
其中u为v构成矩阵的第一个特征向量，为的是从句子向量中减去$c_0$
## 试验
本文在20多项语义相似性检测任务上与许多其他监督和非监督的方法进行了比较，相较于其他非监督方法取得了很大的提升，在一些任务上甚至超过了监督方法。
![image_1d8647o2a1p5n1h5e170uvnv1f7i1g.png-113.1kB][4]


  [1]: http://static.zybuluo.com/zyz0/t9z5u6uurlmb019spb0xl89b/image_1d863r4k9v6s1j734s56bt5i9.png
  [2]: http://static.zybuluo.com/zyz0/6nu1usyn2v5n87q1q4do3x3f/image_1d863slb0vvt157s3ruh911fjgm.png
  [3]: http://static.zybuluo.com/zyz0/iqsib9ukp4fb1chyb93ke93b/image_1d86422171s5n1u507b7459plk13.png
  [4]: http://static.zybuluo.com/zyz0/pmx6x2t3ayy7wkb908j2h1rm/image_1d8647o2a1p5n1h5e170uvnv1f7i1g.png