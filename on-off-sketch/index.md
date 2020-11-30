# On-Off Sketch


## 简介

[On-Off sketch](http://net.pku.edu.cn/~yangtong/uploads/OnOffSketch.pdf)是一个做persisence estimation和finding persistent items的算法。根据论文的介绍，这是第一个专门处理persistence esimation问题的算法，在finding persistent items方面，相比Small-Space和PIE也有着更好的表现。

我读了整篇论文，但是数学证明方面有一些地方没有看懂。所以本文只是对On-Off Sketch算法的过程和实验结果的分析进行描述。

## Persistence

Persistence是一个类似frequncy, cardinality, quantile的数据特征(characteristic)。它的定义如下：

> Given an item $e$ and a data stream with $T$ non-overlapping and contiguous time windows, the persistence of $e$ is defined as the number of time windows where $e$ appears.

## Persistence Estimation

如前文所述，论文中说没有专门在这个方面设计算法的。但是有一些先前用在别的问题上的算法可以去做persistence estimation。

### Prior Work

使用Bloom Filter+Count-Min sketch (CM sketch)。

Bloom Filter是用来处理membership问题的。CM Sketch是用来处理frequency问题的。这个两个算法结合是一种简单的处理persistence estimation的方式。假设当前item $e$到来，算法描述如下。

- 在每个时间窗口结束时，设置Bloom Filter所有位为0。不改变CM sketch。
- CM sketch初始化所有位为0。

1. 判断$e$是否在Bloom Filter中。如果在，说明当前时间窗口$e$已经出现过，不做处理；如果不在，就把$e$插入到Bloom Filter中，执行下一步。
2. 把$e$插入到CM sketch中。

{{< figure src="Figure1.jpg" height="350" width="350">}}

理想状况下，这个算法可以实现persistence estimation。通过Bloom Filter保证一个时间窗口一个item最多被记录一次，通过CM Sketch在所有时间窗口计算persistence值。但是同样，它有很大的局限性。

1. Bloom Filter会出现假阳性 (false positive)。这个时间窗口中没有出现的item被错误地认为是已经出现过。
2. CM sketch的哈希碰撞会导致persistence被严重高估。如下图，3个item被映射到同一位置，本来3个item的persistence应该各增加1，现在各增加了3。

{{< figure src="Figure2.jpg" title="Overestimation" height="350" width="350">}}



对此我们希望停止使用Bloom Filter，并保证每一个CM Sketch的计数器在一个时间窗口只增加1。

### On-Off Sketch: Persistence Estimation

#### 数据结构

On-Off Sketch提出了新的数据结构。On-Off sketch由$d$个数组组成，每一个数组包括$l$个计数器。其中每个计数器都有一个状态域 (state field)，有On和Off两个状态。第$i$个数组的第$j$个计数器表示为$C_i[j]$。

在每个时间窗口最后，把所有的state field设置为On。

{{< figure src="Figure3.jpg" height="350" width="350">}}

#### 添加item

添加item $e_i$到On-Off sketch。用$d$个哈希函数计算哈希值，把$e_i$映射到$d$个计数器$C_j[h_j(e_i)]$中。这$d$个计数器每一个都可能有两种情况：

1. 计数器的state为On。把计数器值增加1，把state变成Off。
2. 计数器的state为Off。说明**这个计数器**在当前时间窗口被访问过 。所以不再改变计数器。

#### 查询

和CM sketch类似，对要查询的item $e_i$做$d$个哈希找到相应计数器，然后取其中的最小值。把persistence的估计值表示为$\hat{p_i}$，也就是
$$
\hat{p_i} = min_{1 \le j \le d} \left(C_j[h_j(e_i)] \right)
$$

#### 算法分析

这个算法达成了前面我们说的两个目标。

1. 避免使用Bloom Filter，不会出现它产生的假阳性问题，从而不会有item的persistence被低估，只会被高估，只会产生单边错误 (one sided errors)。
2. 每一个计数器在一个时间窗口内只能增加1，避免了CM sketch哈希碰撞导致高估过多的情况。

将每一个数组的计数器个数$l$、数组个数$d$表示为
$$
\begin{aligned} l&=e/\epsilon \\\\ d&=ln \left( 1/\delta \right) \end{aligned}
$$
根据论文的分析，算法的时间复杂度为$O \left(ln \left(1/\delta \right) \right)$，空间复杂度为$O \left( \left( 1/\epsilon \right) \right) ln(1/\delta)$。此外可以证明，On-Off sketch的误差一定小于CM sketch。

## Finding Persistent Items

### Prior work

对于这个问题，先前的工作有Small Space和PIE两个。这两个可以看以前的文章。

简述这两个算法的缺点：

1. 大量空间存储的都是non-persistent items，造成严重浪费。
2. Small Space错误控制得不好。想要更精确就要花费大量的空间。
3. PIE相比之下空间需求很大。
4. PIE通过Raptor code恢复item ID需要用到矩阵乘法，速度慢。

### On-Off Sketch: Finding Persistent Items

为了避免出现上述问题，On-Off sketch的核心技术是通过persistence识别items，并且只储存persistent items的ID。

#### 数据结构

由两部分组成。

1. 一个由$l$个计数器组成的数组。第$i$个计数器表示为$C_1[i]$。这个数组使用一个哈希函数$h_1 \left( . \right)$，把每个item映射到数组的一个计数器中。
2. 一个由$l$个桶 (bucket) 构成的数组组成，第一部分中的每一个计数器对应一个桶。例如$B[i]$对应$C_1{i}$。每一个桶含有$w$个键值对 (key-value (KV) pairs)，键是item ID，值是一个计数器。这个计数器用$B[i][e_j]$表示。每个计数器也有它自己的state field。

初始化每一个计数器的state field为On，所有计数器值为0。

{{< figure src="Figure4.jpg" height="350" width="350">}}

#### 插入 {#fpi_insert}

要插入item $e_i$，先判断$e_i$是否已经在桶$B[h_1(e_i)]$中。

- 如果在桶中，且$B[h_1(e_i)][e_i]$的state为On，把$B[h_1(e_i)][e_i]$增加1，state设为Off；如果$B[h_1(e_i)][e_i]$的state为Off，不做操作。
- 如果不在，且$C_1[h_1(e_i)]$的state为On，把$C_1[h_1(e_i)]$增加1，state设为Off。
    - 如果此时$C_1[h_1(e_i)]>B[h_1(e_i)]^{min}$，就交换$C_1[h_1(e_i)]$和桶中值最小的那个计数器的值以及state field。然后把$B[h_1(e_i)]^{min}$对应的item ID换成$e_i$。
    - 如果此时$C_1[h_1(e_i)]\le B[h_1(e_i)]^{min}$，不做操作。
- 如果不在，且$C_1[h_1(e_i)]$的state为Off，不做操作。

在每个时间窗口最后，把所有计数器的state field设为On。

#### 查询persistent items

遍历所有桶，报告计数器的值大于给定阈值的item ID。

#### 算法分析

只用了一个计数器数组和它们附加的桶。减小数组的数量可以提升吞吐量*（关于吞吐量的内容我还不太明白原因）*。使用附加的桶，可以做到分离persistent items和non-persistent items，并且只记录item iD。如果一个item的persistence足够高，那它就不会被替换出去，也避免了哈希碰撞的可能。

设$l=2/\epsilon, w=1$，算法的空间复杂度为$O(1/\epsilon)$。

## 优化

使用SIMD进行加速。比如[上面判断是否已经在桶中](#fpi_insert)，就可以用SIMD进行并行匹配，加速这个过程。同样还是那个算法的比较过程，也可以加速。论文中特意提到Small Space和PIE没有顺序访问操作，大多只是读写一项内容 (read or write one item)，所以不能使用SIMD指令进行加速。

## 结果分析

基本全面优于先前的工作。论文的图就不放了。这里多说一下参数设置对实验的影响结果。

### Persistence Estimation

#### AAE (Average Absolute Error)

随着内存的增加，最优的$d$值也在增大。当内存比较小的时候，更大的$d$会造成AAE增大。但是随着内存的增加，AAE也会快速下降。

#### Insertion Throughput

内存的增大和$d$的增大会造成吞吐量的下降*（这里内存增大为什么会影响吞吐量下降不太明白）*。$d$对吞吐量影响更大，因为$d$变大代表更多的内存访问。

### Finding Persistent Items

#### Error

有足够内存的条件下，$w$增加，AAE和FNR (false negative rate) 往往会下降。论文里还有一段关于"randomness"和$w, l$对AAR, FNR, FPR (false positive rate)的影响，但是我没有看明白，在这里放上原文。

> With smaller $w$, the $l$ increases, and <u>the randomness of also increases</u> *(注：原文如此，没懂)*. When the memory is too small, such randomness can reduce FPR because there are often some buckets whose degree of overestimation is small, so the AAE and FNR decrease. When there is enough memory, randomness increases FPR because there are more buckets whose degree of overestimation is too high, so the AAE and FNR increase.

#### Insertion Throughput

$w$提高，吞吐量会降低。但是由于内存访问次数少，就算$w$比较大，吞吐量仍然比较高。可见这里$w$的选取就是精确度和吞吐量之间的trade-off。
