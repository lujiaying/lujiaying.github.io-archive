title: Python 随机数标准库(2) -- shuffle()
date: 2016-08-18 23:56:04
tags: [Python, Math]
---
Python random包可以用来生成随机数。随机数不仅可以用于数学用途，还经常被嵌入到算法中，用以提高算法效率，并提高程序的安全性。如果想要更加高级的数学功能，可以考虑选择标准库之外的[numpy](http://www.numpy.org/)和[scipy](http://www.scipy.org/)项目，它们不但支持数组和矩阵运算，还有丰富的数学和物理方程可供使用。

本章节主要介绍`random.shuffle()`，也叫做洗牌函数，它被用来打乱列表元素的顺序。
<!-- more -->

## 洗牌的意义
现实生活中有不少需要洗牌算法的场景。

* 扑克牌
* 音乐播放器

扑克牌游戏中，每轮游戏开始前都需要洗牌，使每个人摸到每张牌的概率尽量相等，确保游戏的公平性，同时增加游戏的随机性。

音乐播放器一般会提供“单曲循环”、“列表循环”、“随机播放”等多种播放模式。有不少用户偏好随机播放，著名音乐播放产品[iPod Shuffle](http://baike.baidu.com/link?url=6UWaF33vGb8F99bS8cJBLV5CqhZh0gHgvvVcpJBJoZSa54TdSllZiURr9SIfUPgAyHwGtUee0q_pvsvOuORQRq)的卖点之一就是“你永远不知道你将要听到的下一首歌曲是什么”。

随机播放可以有两种实现方案: `random` 或 `shuffle`。

|名称|原理|优点|缺点|
|--|----|----|----|
|random|每次随机取歌播放|歌曲选取等概|可能出现AA、ABAB、ABCABC这种反人脑随机认知的序列|
|shuffle|按乱序列表顺序播放|避免random的缺陷|每轮播放中同一首歌只能出现一次|

## shuffle算法原理

### Knuth-Durstenfeld Shuffle
[Knuth-Durstenfeld Shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle#The_modern_algorithm) 在[Fisher-Yates shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)的基础上对算法进行了改进，形成了现代工业界常用的洗牌算法。
>Durstenfeld's solution is to move the "struck" numbers to the end of the list by swapping them with the last unstruck number at each iteration.

Durstenfeld的方法是：
1. 每次从未乱序列表中随机选择一个数字
2. 把该数字与未乱序列表的末尾数字交换，如此做则末尾存放的是已乱序处理过的数字
3. 循环，直至所有数字均被乱序处理过

这是一个in-place的算法，算法时间复杂度从Fisher算法的*O(n^2
)*提升到了*O(n)*。算法伪代码如下：

```
-- To shuffle an array a of n elements (indices 0..n-1):
for i from n−1 downto 1 do
     j ← random integer such that 0 ≤ j ≤ i
     exchange a[j] and a[i]
```

Python Random标准库中的shuffle使用的正是`Knuth-Durstenfeld Shuffle`算法。

```Python
def shuffle(self, x, random=None):
    """x, random=random.random -> shuffle list x in place; return None.
    Optional arg random is a 0-argument function returning a random
    float in [0.0, 1.0); by default, the standard random.random.
    """
    if random is None:
        random = self.random
    _int = int
    for i in reversed(xrange(1, len(x))):
        # pick an element in x[:i+1] with which to exchange x[i]
        j = _int(random() * (i+1))
        x[i], x[j] = x[j], x[i]
```

### 蓄水池算法

[蓄水池算法](https://en.wikipedia.org/wiki/Reservoir_sampling)从某种意义上来说，是另一种形式的shuffle算法。它能够从未知或者很大样本空间随机地取k个数，且保证样本空间中每一个数被抽取的概率是等概的。

在实际应用中，往往会遇到大数据流的情况。因此，我们无法先保存整个数据流然后再从中选取，而是期望有一种算法能将数据流遍历一遍就得到所选取的元素，并且保证得到的元素是随机的。这种算法正是蓄水池算法。

蓄水池算法的伪代码如下：
```
Init : a reservoir with the size: k
    for i= k+1 to N
        M=random(1, i);
        if (M < k)
             SWAP the Mth value and ith value
    end for
```

算法的核心思想分为两步：
1. 构建一个容积为k的蓄水池，即对一个数据流的前k个元素，保存在集合A中；
2. 从第k+1个元素开始，假设当前元素为数据流中的第m`(m>k)`个元素。先以 k/m的概率选择第m个元素，如果被选中，则从集合A中随机选择一个元素并用元素m替换之；否则淘汰元素m。 

下面给出每个元素被选中的概率为k/n的证明：
设某元素m，且 m <= n，那么`P(m最终被选中) = P(m被选中) * [P(m后面的元素没有被选中) + P(m后面的元素被选中) * P(m没有被替换)]`，即：

![证明](http://7xkdra.com1.z0.glb.clouddn.com/image/blog/%E8%93%84%E6%B0%B4%E6%B1%A0%E7%AE%97%E6%B3%95%E8%AF%81%E6%98%8E.png)

## 拓展阅读
* [Python 随机数标准库(1) -- random()](http://lujiaying.github.io/2016/06/02/Python-%E9%9A%8F%E6%9C%BA%E6%95%B0%E6%A0%87%E5%87%86%E5%BA%93-1-random/)
* Python 随机数标准库的应用 -- 彩票究竟是不是低智商税(计划写作中)

> 更多精彩内容，请关注我的[个人博客](http://lujiaying.github.io/)。
