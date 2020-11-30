# PIE算法


## 简介

[PIE算法](http://www.vldb.org/pvldb/vol10/p289-dai.pdf)使用Space-Time Bloom Filter(STBF)和Raptor Code，寻找persistent items。

我只读了算法的过程，数学证明和其他内容没有看。

## Space-Time Bloom Filter

STBF是Bloom Filter的扩展。Bloom Filter是$m$ bits，STBF就是$m$个cell。PIE中STBF的一个cell包含3部分内容：

1. 1 bit的标志位$C_{iF}$。

    - 0代表cell未使用或者发生冲突。

    - 1代表cell已经被使用。

2. $r$ bits的Raptor Code $C_{iR}$。

3. 用一个特定的哈希函数计算的$p$ bits的hash-print $C_{iP}$。

和Bloom Filter一样，STBF使用$k$个哈希函数做映射。如果有两个item被映射到了同一个cell（发生了冲突），那么把$C_{iF}$置0，$C_{iR}$和$C_{iP}$所有位置1。

<img src="Space-Time Bloom Filter.jpg" style="zoom:67%;" />

## Raptor Code

Raptor code最重要的优势是对一个长度为$l$ bits的item ID，我们只需要存储$r$ bits （$r$比$l$小很多）的Raptor code就可以恢复出原本的item ID。

这个从原理上来说其实并不复杂，主要方式就是矩阵乘法。把item ID看作是$l \times 1$的向量，选取一个$r \times l$的矩阵
$$
A =
\begin{bmatrix} a_{i1}^1 & a_{i1}^2 & \cdots & a_{i1}^l \\\\ a_{i2}^1 & a_{i2}^2 & \cdots & a_{i2}^l \\\\ \vdots & \vdots & & \vdots \\\\ a_{ir}^1 & a_{ir}^2 & \cdots & a_{ir}^l \\\\ \end{bmatrix}
$$

$$
A \cdot
\begin{bmatrix} I_1^e \\\\ I_2^e \\\\ \vdots \\\\ I_l^e \\\\ \end{bmatrix} = \begin{bmatrix} R_{i1}^e \\\\ R_{i2}^e \\\\ \vdots \\\\ R_{ir}^e \end{bmatrix}
$$



## PIE算法过程

PIE算法分为两个阶段：

1. Recording Phase
2. Identification Phase

### Recording Phase

这个阶段用来记录各个时间窗口出现的item。每一个时间窗口都要构建一个STBF来做记录。这些STBF应该用的都是相同的$k$个哈希函数。

假设当前时间窗口为$t_i$，接收到了item $e$。

1. 用$e$的ID计算$r$-bit的Raptor code和$p$-bit的hash-print。
2. 用STBF的$k$个哈希函数计算要映射的cell。
3. 检查这些cell是空的(empty)，还是已经被一个item使用的(singleton)，还是已经发生冲突的(collided)。
    - 如果是空的，就把三个数值添加进去。标志位设为1之后，这个cell的状态就变成了singleton。
    - 如果已经被使用，检查储存在cell中的这个项目的hash-print和$e$的是否匹配。如果匹配不需要做其他操作；如果不匹配就要把cell的状态变成collided了，做法参照前文STBF的内容。
    - 如果已经发生冲突，不需要做其他操作。

### Identification Phase

这个过程用来找persistent items。

经过$T$个时间窗口，就建立了$T$个STBF。下面根据论文给出的例子讲解。

<img src="Cell Line.jpg" style="zoom:80%;" />

假设一个STBF长度为$m$-bit，新定义cell line，包含$STBF_1[x], STBF_2[x], \dots , STBF_T[x]$，其中$1 \le x \le m$。

下面对cell line进行处理。

#### Cell Line Processing

1. 丢弃cell line中所有为空(empty)和冲突(collide)的cell。
2. 按照hash-print为cell line中剩下的进行分组(group)。hash-print相同的分为一组。可以看上面的图，颜色相同的在一个group中。图中圈出的cell line一共有3组。这里可能会遇到hash-print碰撞导致不相同的item被分到了一组，这样的group称为mingled groups。我们将在[Mingled groups](#mingled_groups)讨论这一情况。
3. 将第$x$个cell line的groups和前面$x-1$个cell lines的groups比较，合并其中hash-print相同的组。

其中$x=1$的cell line不执行第3步，$x=2$到$x=m$的cell lines执行以上3步。

#### Recovering Item IDs

通过上面的步骤，对得到的groups计算item ID。**这一步好像没有详细说明是如何选取Raptor code进行计算的**（随机选一个item的还是用其他方法取得的）。因为如果是mingled groups，可能会出现group中两个item的hash-print相同而Raptor code不同的情况。

恢复出item ID后进行两步验证，确定item ID是否正确。

1. 计算恢复出的item  ID的hash-print，与group中的hash-prints比较是否相等。

2. （通过第一步验证后）使用$k$个哈希函数计算恢复的ID会被映射到的cell。

    如果和group中的cell都冲突了，所有这些cell的Raptor code都相同，所有这些cell的hash-print都和恢复出的item ID的hash-print相同才算验证通过。

    为了防止解释不清这里上原文吧。

    > For the recovered ID to pass the second test, unless the cells at these k locations in these STBFs are collided, the Raptor code fields of all these cells should be the same and the hash-prints of all these cells must match the hash-print of the recovered ID.

如果两步验证全都通过，那么这个恢复出的item ID有很大概率是正确的。

#### Mingled Groups {#mingled_groups}

最后要解决一个group里有不同item的情况。

从$g_r=1$开始，临时把$g_r$个cells从group的$g$个cells中移除。用其他的$g-g_r$个cells恢复item ID。然后把$g_r$增加1，重新执行临时移除直到阈值$g_T$。一般$g_T$不会超过2.

从$g$个items中移除$g_r$个，也就是$\left ( \begin{smallmatrix} g \\\\ g_r \end{smallmatrix} \right)$种移除方式。当$g_r=1$时就是要做$g$次不同的移除，每次都重新恢复一个item ID再验证。

## 算法的问题

PIE的优势在于一个item的persistence越高，成功恢复item ID的概率也越大。但是它也有明显的缺陷。

1. 使用Raptor code节省了空间，但是PIE要记录每一个时间窗口的所有不同的items(every distinct item)。其中大部分也都是non-persistent items，PIE浪费了大量空间在这些上面。
2. 计算Raptor code要进行矩阵乘法，相比数据流的速度来说矩阵乘法太慢了。
