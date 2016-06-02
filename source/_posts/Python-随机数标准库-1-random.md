title: Python 随机数标准库(1) -- random()
date: 2016-06-02 14:03:53
tags: [Python, random, Math, 随机数, 数学, 算法, 技术笔记]
---
Python random包可以用来生成随机数。随机数不仅可以用于数学用途，还经常被嵌入到算法中，用以提高算法效率，并提高程序的安全性。如果想要更加高级的数学功能，可以考虑选择标准库之外的[numpy](http://www.numpy.org/)和[scipy](http://www.scipy.org/)项目，它们不但支持数组和矩阵运算，还有丰富的数学和物理方程可供使用。

本章节主要介绍`random.random()`，它被用来生成一个0~1之间的随机浮点数，是`randint(), shuffle(), choice()`等其他函数的基础。

## 随机数的意义
现实生活中，有很多场景需要用到“随机数”：

* 彩票
* 棋牌游戏中的洗牌和掷骰子
* 游戏掉宝率

<!-- more -->

其中大部分是靠计算机软件生成的的“伪随机数”。伪随机数一般是由随机种子和随机算法计算生成的，也就是说，在随机种子和随机算法一定的情况下，伪随机数是可重复、可预测的。
简单来说，伪随机数具有**循环长度**。什么叫循环长度？就是如果第一次产生数字55，第二个产生数字107，那么循环多少次后，会继续产生`55，107……`这样的序列。大部分简单算法的循环长度都是2^32左右。

这有什么影响吗？如果你使用伪随机数生成中奖号码，有心人可以根据历史结果推断出你的中奖号码规律。事实上，过去有些电话充值卡就是因此被破解的。大家可以思考其中的门道。

> 伪随机数的一个优点是它们的计算不需要外部的特殊硬件的支持，因此在计算机科学中伪随机数依然被使用。真正的随机数必须使用专门的设备，比如热噪讯号、量子力学的效应、放射性元素的衰退辐射，或使用无法预测的现象，譬如用户按键盘的位置与速度、用户运动鼠标的路径坐标等来产生。 [[wiki](https://zh.wikipedia.org/wiki/%E4%BC%AA%E9%9A%8F%E6%9C%BA%E6%80%A7) 伪随机性]

## 好的随机数算法
软件无法产生真正意义上的随机数，而只能产生伪随机数。
好的随机数算法应具有如下性质： 
(1)相同序列的概率非常低    
(2)符合统计学的平均性

好的伪随机数算法和差的具有天壤之别，其中差别之大用肉眼就可以观测到：

![C#的System.Random类生成的随机数填充的位图](http://7xkdra.com1.z0.glb.clouddn.com/image%2Fblog%2FC%23%E7%9A%84System.Random%E7%B1%BB%E7%94%9F%E6%88%90%E7%9A%84%E9%9A%8F%E6%9C%BA%E6%95%B0%E5%A1%AB%E5%85%85%E7%9A%84%E4%BD%8D%E5%9B%BE.png)
*C#的System.Random类生成的随机数填充的位图*


![php的rand函数生成的随机数填充的位图](http://7xkdra.com1.z0.glb.clouddn.com/image%2Fblog%2Fphp%E7%9A%84rand%E5%87%BD%E6%95%B0%E7%94%9F%E6%88%90%E7%9A%84%E9%9A%8F%E6%9C%BA%E6%95%B0%E5%A1%AB%E5%85%85%E7%9A%84%E4%BD%8D%E5%9B%BE.png)
*php的rand函数生成的随机数填充的位图*

## 常用的随机数算法
#### 线性同余
线性同余随机数生成器LCG（linear congruential generator）是最古老也最广为人知的位随机数生成器算法，它通过递归公式产生伪随机数。
公式：`X(i) = [a * X(i-1) + c] Mod m`产生一个`[0, m-1]`之间的随机数序列，种子值`X(0)`是用户提供的且是上式中唯一用作递归的随机数。该序列通过浮点数操作（除法）可以转化为在`[0, 1]`之间均匀分布的随机数序列。
LCG的循环长度对`a, c`敏感，选择合适的参数循环长度可达到`m`。

通过以上公式，我们很容易完成线性同余生成器的Python实现。
```Python
class Random(object):
    '''Random 库实现'''
    def __init__(self, x=None):
        if x is None:
            import time
            self.seed = long(time.time())
        else:
            self.seed = long(x)
        self._seed = self.seed
        
    def random_LCG(self):
        '''
        线性同余产生0~1之间的随机数

        X(k) = [a * X(k-1) + c] mod m
        '''
        # 以下参数值参照GCC编译器
        m = 2**32
        a = 1103515245
        c = 12345
        self._seed = (a * self._seed + c) % m
        return self._seed / float(m-1)
```

#### 平方取中
平方取中法是由冯·诺依曼在1946年提出的，其基本思想为：将数列中的第`X(i)`项（假设其有m位）平方，取得到的`2m`位数（若不足2m位，在最高位前补0）中间部分的`m`位数字，作为`X(i)`的下一项`X(i+1)`，由此产生一个伪随机数数列。即：
`x(i+1) = (10^(-m/2) * x(i) * x(i)) Mod (10^m)`
平方取中法计算较快，但在实际应用时会发现该方法容易产生周期性明显的数列，而且在某些情况下计算到一定步骤后始终产生相同的数甚至是零，或者产生的数字位数越来越小直至始终产生零。

在类 Random 中添加平方取中函数。
```Python
    def random_MSM(self):
        '''
        平方取中法产生0~1之间的随机数

        X(i+1) = [10^(-m/2) * X(i)^2] mod (10^m)
        '''
        m = 8
        self._seed = long((10**(-m/2) * self._seed * self._seed) % (10**m))
        return self._seed / float(10**m -1)
```

#### Wichmann-Hill
Python Random标准库中提供了基于Wichmann-Hill 算法实现的`random()`函数。

Wichman-Hill算法通过线性合并不同短周期随机数发生器的输出来产生长周期的随机数序列。如果把周期`N1`和`N2`的波形加起来那么得到的波形周期为
`N = lcm（N1,N2）`
这样，通过结合几个随机数发生器的输出，可以产生一个更长的序列。它结合3个随机数发生器的输出如下:
```
X(n) = 171 * X(n-1) Mod 30269
Y(n) = 172 * X(n-1) Mod 30307
Z(n )= 170 * X(n-1) Mod 30323
U(n) = {X(n)/30269 + Y(n)/30307 + Z(n)/30323}
```
源码如下：
```Python
def random(self):
        """Get the next random number in the range [0.0, 1.0)."""

        # Wichman-Hill random number generator.
        #
        # Wichmann, B. A. & Hill, I. D. (1982)
        # Algorithm AS 183:
        # An efficient and portable pseudo-random number generator
        # Applied Statistics 31 (1982) 188-190
        #
        # see also:
        #        Correction to Algorithm AS 183
        #        Applied Statistics 33 (1984) 123
        #
        #        McLeod, A. I. (1985)
        #        A remark on Algorithm AS 183
        #        Applied Statistics 34 (1985),198-200

        # This part is thread-unsafe:
        # BEGIN CRITICAL SECTION
        x, y, z = self._seed
        x = (171 * x) % 30269
        y = (172 * y) % 30307
        z = (170 * z) % 30323
        self._seed = x, y, z
        # END CRITICAL SECTION

        # Note:  on a platform using IEEE-754 double arithmetic, this can
        # never return 0.0 (asserted by Tim; proof too long for a comment).
        return (x/30269.0 + y/30307.0 + z/30323.0) % 1.0
```

#### Mersenne Twister
Mersenne Twister算法译为马特赛特旋转演算法，是伪随机数发生器之一，其主要作用是生成伪随机数。此算法是Makoto Matsumoto （松本）和Takuji Nishimura （西村）于1997年开发的，基于有限二进制字段上的矩阵线性再生。可以快速产生高质量的伪随机数，修正了古老随机数产生算法的很多缺陷。
Mersenne Twister有以下优点：随机性好，在计算机上容易实现，占用内存较少(mt19937的C程式码执行仅需624个字的工作区域)，与其它已使用的伪随机数发生器相比，产生随机数的速度快、周期长，可达到2^19937-1，且具有623维均匀分布的性质。
Mersenne Twister 目前被很多语言采用，作为随机数产生算法。以下是Python Random库中关于默认 Random.random() 函数的说明。
```Python
General notes on the underlying Mersenne Twister core generator:

* The period is 2**19937-1.
* It is one of the most extensively tested generators in existence.
* Without a direct way to compute N steps forward, the semantics of
  jumpahead(n) are weakened to simply jump to another distant state and rely
  on the large period to avoid overlapping sequences.
* The random() method is implemented in C, executes in a single Python step,
  and is, therefore, threadsafe.
```


## 性能比较

![随机数散点图](http://7xkdra.com1.z0.glb.clouddn.com/image%2Fblog%2F%E9%9A%8F%E6%9C%BA%E6%95%B0%E6%95%A3%E7%82%B9%E5%9B%BE.png)

根据随机数算法生成一组坐标(x,y)，使用[pylab](http://matplotlib.org/)绘制其在二维坐标系上的散点图。
上图的四个子图分别由不同的算法生成的**2000**个点绘制，散点图的特点是相同点数不堆积只绘制一次，因此在坐标轴上点数越多、分布越均匀代表算法性能越优秀。
可发现：
* 平方取中算法较其他算法出现了明显的稀疏分布，即算法在2000个点时已命中循环长度。
* 线性同余生成器算法在2000个点时与Python Random标准库提供的Wichmann-Hill、Mersenne Twister算法性能差距不大。

## 拓展阅读
Random 标准库`shuffle(), choice()` 函数原理分析（计划写作中）
[伪随机的上位和真随机的逆袭](http://blog.jobbole.com/83187/)
