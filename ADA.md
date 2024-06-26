# 算法设计与分析
## 第九章 字符串算法 
#### 2 Rabin-Karp算法
滚动计算哈希值，但是人做的时候直接用除法求余数即可。
#### 3 有限状态自动机的算法
看待字符串匹配的角度：把主串逐个读入，与模式串作比较，把匹配上的字符个数定义为状态。状态为0到m，那么字符串匹配的过程就是逐个读入字符，更新当前状态的过程。
#### 4 KMP算法
构造前缀表的实现代码, nextchar为前缀表，预先构造的数组。 [Leetcode字符串识别题目，作业第三题](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/)
```python

        def getnext(t:str , nextchar):
            k = -1 
            j = 0
            nextchar[j] = -1
            while(j < len(t) - 1):
                if(k == -1 or t[j] == t[k]):
                     j += 1
                     k += 1
                     nextchar[j] = k
                else:
                    k = nextchar[k]


```
KMP算法的代码
```python
        for i in range(len(haystack)):
            # 先找最近的可能匹配位置看是否匹配，一开始是needle[j]，若needle[j]不匹配，找getnext[j]
            # 一直找到j == 0 的时候停。如果是j == 0 了而结束循环，那么之后会对needle[0]和当前haystack[i]做判断
            # 若二者相等，会匹配一位；若二者不相等，不会匹配，for循环自然进入下一轮, 新的haystack[i]和needle[0]
            # 重新开始比较
            while j > 0 and haystack[i] != needle[j]:
                j = nextchar[j]
            if(haystack[i] == needle[j]):
                j += 1
            if(j == len(needle)):
                index = i - j + 1
                break
```
首先把握模式串的下标，一般是0到m-1，那么需要用到前缀表的，是1到m-1的下标，因为这一个实现使用for语句实现，只有j>0没匹配上，才使用到前缀表查询；如果j=0，不管匹配上与否，下一轮主串都会到下一个字符。所以生成前缀表的时候，主要做1到m-1这些坐标。  
看kmp的主要代码，通过前缀表查询下一个比较的j的值时，输入的j大于0，换言之，只有大于0的坐标位置匹配失败才会访问前缀表，因此，前缀表的0位是什么根本不重要。在另外一个实现中，用的while循环，则是把`next[0]`设置为-1，当遇到-1时移动主串上的游标。真正查前缀表的，只会是从1坐标开始，就是从第二个开始。  
其次要注意到，前缀表1位置绝对是0.从上面的getnext函数一看就知道，`nextchar[1]`肯定是0.或者直接理解，现在状态为匹配成功了1个，第二个匹配失败，那肯定回到状态0。因此，如果要用ppt和书本上for循环的做法，是规定`next[1]`为0，从2号位开始生成。
```python
      def getnext(t:str , nextchar):
            k = 0
            nextchar[k] = -1
            n = len(t)
            if n > 1:   #leedcode上有测试样例数组越界，只好加一个判断条件
                nextchar[1] = 0
            for i in range(2 , n):
                while k > 0 and t[k] != t[i - 1]:
                    k = nextchar[k]
                if t[k] == t[i - 1]:
                    k += 1
                nextchar[i] = k
```
手算前缀表的过程，还是按照数据结构课整理的方法，求`next[i]`找前i个，一般是`pattern[0:i-1]`,肉眼观察出最大相同前后缀。

带注释完整代码
```python
class Solution:
    def strStr(self, haystack: str, needle: str) -> int:
        # 先做好计算前缀表的函数
        def getnext(pattern):
            # 生成列表 ,下标1到n-1会用到查表，所以下标覆盖1到n-1即可
            n = len(pattern)
            Pi = [0 for _ in range(n)]
            # 用双探头来写，一个找位置，一个写入
            # 用i写入，由于写的是1到n-1范围，且每次i++完写入，因此i从0到n-2
            i = 0
            j = -1
            Pi[0] = -1
            while (i < n - 1):
                if j == -1 or pattern[i] == pattern[j]:
                    i += 1
                    j += 1
                    Pi[i] = j
                else:
                    j = Pi[j]
            return Pi

            
        Pi = getnext(needle)
        k = 0
        index = -1
        # 用for语句写,读主串的每一个字符
        n = len(haystack)
        m = len(needle)
        for i in range(n):
            # 后写这个：状态k > 0发生不匹配的时候，查前缀表更新状态
            while k > 0 and haystack[i] != needle[k]:
                k = Pi[k]
            # 先写这两句：当匹配上，状态更新成下一个，检查是否匹配完毕
            if haystack[i] == needle[k]:
                k += 1
            if k == m:
        # 写返回 , 当前i在最后一个（对应m-1），但是k也就是匹配状态已经超过了最后一个，k是m，所以i-(m-1)不用管k
        # 可以先让index = -1 ，匹配完成再更新成i-m+1 , 省的条件判断
                index = i - m + 1
                break

       
        return  index
            
```



#### 5 BMH逆向朴素匹配算法
此算法是模式串正向移动，每次逆向比较，发生不配对时，设主串上本次匹配段的最后一个字符是α，如果模式串的0到m-2段落（就是除了最后一个以外）有出现α，把最右边那个α直接移过去和主串上那个α对齐，然后下一轮匹配。为了方便高速移位，这是借助提前算好的偏移表完成。计算有一个公式。  
当`pattern[0:m-2]`出现了α，那么向右偏移量`shift[α]`等于模式串最后一个坐标m-1,减去最右侧出现的α的坐标。否则，直接偏移m，即模式串长度。

## 第八章 图算法
#### 1 Bellman-Ford 最短路径算法
设结点数有|V|,算法表述为每次松弛所以边，循环|V|-1次。之后对所有边检查一次，如果还有边没松弛，说明有负环。否则所有结点都求出了单源最短路径。  
每一轮要松弛|E|条边，做|V|-1轮，算法复杂度是O(VE)的。
```
进行|v|-1轮循环：
        每次松弛所有的边
对每一条边：
        如果还能松弛，就有负环
```

##### 有向无环图(DAG)中的算法
```
生成一个拓扑排序  
按照拓扑序的每个点v：  
    松弛v的所有出边


```
就求好了，时间复杂度O(|E|)
#### 2 Dijkstra 算法
[leetcode上的链接](https://leetcode.cn/problems/network-delay-time/submissions/)
```
点分为两个集合，一个集合A表明已经确定了最小距离的点，一个集合B是尚未确定最小距离的点，初始化所有点在B中(上面的实现使用Bool数组来标记是否在A中)
初始化所有点的最小距离估计为无穷大。（上面的实现使用数组d[n]完成，实际上是把所有点的d做成一个优先级队列，这里就是Insert操作）
把源点的d设置为0
进入n次的循环：
        用优先级队列选出最小d是哪个节点  （上面的实现基于线性数组，就当作是O(n)复杂度的优先级队列 ， 此外应该用二叉堆、斐波那契堆来做，这里就是Extact-min操作）
        把这个点加入到集合A中  （上面实现就是把这个点对应的Bool数组元素改成True）
        松弛这个点u的所有出边(u,v)，如果有做松弛，应该把v的prev设置成这个u    （这就是对应优先级队列的Decrease-key操作）
返回d[target](就是目标节点target的最短路径值了)
       
```
分析复杂度：
decrase-key操作调用了多少次？这里用**聚合法**分析(第七章 平摊分析)，每一个点加入的时候松弛它所有出边，那么所有点的出边加起来就是整张图的所有边，也就是调用|E|次。   
Insert操作调用了多少次？每个点1次，就是|V|次。  
Exctract-min操作调用了多少次？显然是|V|次。  （上面伪代码的n就是结点数）   
当使用不同的优先级队列实现，这三个操作会有不一样的复杂度。  

#### 3 Floyd-Warshall 算法
[跳转](https://github.com/changdick/23Springsjjgsydm/blob/main/3%20%E9%87%8D%E8%A6%81%E7%AE%97%E6%B3%95%E9%83%A8%E5%88%86.md#8-floyd-%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84%E7%AE%97%E6%B3%95)  
可能考每一步的dp矩阵  
在写每一步的矩阵时，把上标k标上，如 $D^1,D^2,D^3,D^4...$，上标就是k，每次中间去绕这个点。

#### 4 最大流问题 Ford-Fulkerson算法
画图，余图，找增广路径，更新流，画余图，找增广路径，更新流，循环直到余图中没有增广路径，那么已经找到最大流。
#### 5 二部图最大匹配问题 匈牙利算法
匈牙利算法是基于交替路径、增广路径完成的。交替路径是指二部图中，匹配边、非匹配边交替的路径。而可增广的路径是指一条交替路径，如果最两端都是非匹配边，那么可增广。把这条路径中的匹配变非匹配边，非匹配边变匹配边，那么会多出来一个匹配。  
找最大匹配，只需找增广路径，取反增加匹配数，再找增广路径，再取反增加匹配数。一直到找不出来匹配边的增广路径，那么就找到了最大匹配。  
这是作业题，用dfs策略，执行匈牙利算法。
```python
    # 下面是在图中使用匈牙利算法
    def dfs( u ,visited):
        for v in dictionary[u]:
            if v not in visited:
                visited.add(v)
                if match[v] == -1 or dfs( match[v] , visited)
                    match[v] = u
                    return True
        return False

    for i in range(n):
        visited = set()
        dfs(i , visited)

```

## 第三章 分治策略
#### 1 快速排序
快速排序是分治策略，每次分成子问题，然后处理子问题，类似于先序遍历二叉树。  
下面是伪代码，partition函数用的是课本中的思路
```
quicksort(A,p,r)
if p < r
        q = partition(A,p,r)
        quicksort(A,p,q-1)
        quicksort(A,q+1,r)

partition(A,p,r)
        pivot = A[r]
        j = p - 1
        for i = p to r-1
                if A[i] < pivot
                        j = j + 1
                        swap(A[i] , A[j])
        swap(A[j+1] , A[r])
        return j+1
```
#### 2 归并排序

#### 3 选择顺序统计量

#### 4 二分查找的标准写法
```python
    def search(nums: List[int], target: int) -> int:
        left = 0
        right = len(nums) - 1
        while(left <= right):
            mid = (left + right) // 2
            if nums[mid] == target:
                return mid
            elif nums[mid] > target:
                right = mid - 1
            elif nums[mid] < target:
                left = mid + 1
        return -1
# 区间用闭区间， 闭区间就得用小于等于号控制条件 ， 分三个条件判断， mid加减1.
```

## 第四章 动态规划
用动态规划算法的问题特征： 最优化问题。  
适用动态规划需满足： 最优子结构性质，子问题重叠。  
做题关键：找出一种好的优化解的**定义**，这种定义有最优子结构性质，方便自底向上计算。  

做动态规划的题目，关键在于找出一种最优解的定义，分解子问题写出状态转移方程。只要写出了状态转移方程，让计算机自底向上计算就没有难度了。

> 编程题目足迹  
> [322零钱兑换](https://leetcode.cn/problems/coin-change/)  
> [300最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)  
> [53最大子数组和](https://leetcode.cn/problems/maximum-subarray/description/)  
> [198打家劫舍](https://leetcode.cn/problems/house-robber/)  
> [213打家劫舍2 作业题](https://leetcode.cn/problems/house-robber-ii/description/)  
> [64最小路径和](https://leetcode.cn/problems/minimum-path-sum/description/)  
> [编辑距离](https://leetcode.cn/problems/edit-distance/)  
> [最长公共子序列LCS问题](https://leetcode.cn/problems/longest-common-subsequence/)

#### 1 矩阵乘法问题
问题：矩阵 $A_1, A_2, ... , A_n$ ,用什么顺序计算 $A_1A_2...A_n$,代价最小。 
最优解定义和状态转移方程： 定义m[i,j]为 $A_iA_{i+1}...A_j$的最小代价，p[i]为矩阵 $A_i$的列数，那么m[i,j] = m[i,k]+m[k+1,j]+p[i-1]p[k]p[j]，其中k是遍历所有满足 $i \leq k < j$。 
#### 2 LCS问题
问题： text1和text2的最长公共子序列长度。
定义： dp[i,j]表示text1[0到i-1] 和text2[0到j-1]两个字符串的最长公共子序列。  
状态转移： 直接考察两字符串最后一个字符text[i-1] 和 text[j-1]，可以相等或不相等。
1. 选择text1[i-1] == text2[j-1], 那么这两个肯定是当前情况dp[i,j]所对应LCS的最后一个字符，dp[i,j] = dp[i-1 , j-1] + 1
2. 选择text1[i-1] != text2[j-1]，两个都有可能是LCS最后一个字符，选text1[i-1]是，那text2[j-1]可以离开， dp[i,j] = dp[i , j-1]
3. 选择text1[i-1] != text2[j-1]，两个都有可能是LCS最后一个字符，选text2[j-1]是，那text1[i-1]可以离开不在考虑，dp[i,j]=dp[i-1,j]  
三种选择对应依赖于三种前置状态，总是选最大的那个。因此状态转移方程dp[i][j] = max(dp[i - 1][j] , dp[i][j - 1] , dp[i - 1][j - 1] + (text1[i -1] == text2[j - 1]))。
由于每一次计算dp[i,j]依赖于三种前置的状态，分别在左上方，上方，左方，先初始化计算dp[0][0:n],dp[0:m][0]，全等于0， 再开始自底向上计算。

#### 3 背包问题
0-1背包问题：定义dp[i][j]表示装的物品为i到n，允许使用的最大容量为j时的最优解，那么我们要的答案dp[1][C] .  
状态转移：所做的选择就是第i个物品，拿与不拿的问题。  
1. 拿第i个物品， 那么 dp[i][j] = dp[i+1][j-w[i]] + v[i]
2. 不拿第i个物品， 那么dp[i][j] = dp[i+1][j]
所以dp[i][j] = max(dp[i+1][j-w[i]] + v[i] , dp[i+1][j]). 当`j - w[i] < 0` ,意思说装不下,就dp[i][j]=dp[i+1][j]，只能选择不装。

注意：背包问题在列举重量的那个装态的时候，正着遍历和倒过来遍历都可以。如果是倒过来遍历的话，可以压缩成一维数组。

>例.对于0-1背包问题，给定价值数组v={7, 3, 6, 10, 8},重量数组w={5, 2, 2, 6, 4},背包容量C=10。问：如何选择装入背包的物品，使得装入背包的物品的总价值最大？写出递归方程。  
> 递归方程dp[i][j] = max(dp[i+1][j] , dp[i+1][j-w[i]]+v[i]) ,先算i=5，再算i=4，i=3，i=2, i=1,  填表就知道最大价值19.

完全背包问题：物品可以拿多次，我们可以这样做：  
修改一下状态转移方程，当j-w[i] <0 ，dp[i][j] = dp[i+1][j]. 其他情况，dp[i][j]=max(dp[i+1][j],dp[i][j-w[i]]+v[i]),这里把放入i的情况改了下标，使得可以重复放。  0-1背包问题倒过来遍历压缩成一维数组，是类似的道理，如果正着遍历，会出现重复放，倒着就不会。如果要做完全背包问题，必须得正着遍历。

#### 4 最优二叉搜索树
把握住问题是如何定义的。对于包含关键字 $k_i,k_{i+1},...,k_{j-1},k_j$的最优二叉搜索树，定义问题的解是期望代价E[i,j]，下标i和j的定义为：i从1到n，j从0到n。数学记为 $i\geq 1, i-1\leq j\leq n$,并且定义下标 $i\leq j$时，二叉搜索树是包含了关键字 $k_i$到 $k_n$的，当且仅当 $j=i-1$,也就是下标 E[i,i-1]的情况，是仅仅包含一个为关键字 $d_{i-1}$的。 关键字的下标是i到j，伪关键字的下标是i-1到j。返回答案是返回E[1,n]，整个问题的关键字是1到n，伪关键字下标是0到n。  
问题的下标[i,j],表示二叉搜索树关键字从i到j。 j有严格的范围是0到n。 当出现i比j大1的时候，不包含关键字，仅仅包含为关键字 $d_j$。  
当构造一棵子树作为一个结点的子树时，其深度全加1，所以期望搜索代价的变化是W， W就是对该子树所有结点(关键字，伪关键字结点)的搜索概率求和，W[i,j]直接看i和j的值，关键字从i求到j，伪关键字是从i-1求到j。  
定义期望搜索代价E[i,j],要分情况， $i\leq j$，说明有关键字i到j， 当选取某个节点 $k_r$当树根的时候， 就会有最优子结构出现，因而有递归方程来求。递归方程为 $$E[i,j] = E[i,r-1]+E[r+1,j]+W[i,j]$$而特殊情况，i比j大，数学记作 $j=i-1$，也就是E[i,i-1]的情况，只有一个伪关键字 $d_{i-1}$，所以规定E[i,j]= $d_j=d_{i-1}$, 当 $i-1=j$.  
为了记录二叉搜索树构建的过程，定义root[i,j]来记录 $k_i$到 $k_j$的最优二叉搜索树的根是多少。  
计算W[i,j]的过程也是动态规划，W[i,j]=W[i,j-1]+ $p_j+q_j$. 同样是规定 j=i-1时，即W[i,i-1] = $q_{i-1}$

## 第五章 贪心算法
刷题足迹
> [455分发饼干 深圳18，19级考题](https://leetcode.cn/problems/assign-cookies/)  
> [134加油站 本部19年考题](https://leetcode.cn/problems/gas-station/)
> [55跳跃游戏](https://leetcode.cn/problems/jump-game/)  
> [45跳跃游戏II 作业题](https://leetcode.cn/problems/jump-game-ii/)


#### 1 活动安排问题
#### 2 Huffman编码
#### 3 最小生成树算法

## 第六章 搜索

#### 爬山法
爬山法是加了一点选择的深度优先搜索策略，用栈实现，但是每次扩展哪个子结点的的时候，参考某个评价函数，在当前节点的子结点中选优的那个。它的特点是一条搜索路径肯定走到底。与之相比，Best-First就不是深度优先策略，Best-First考虑的是全局最优，如果发现当前搜索路径不如之前的好，会回去搜索其他结点。但是爬山法不会回头搜索。
#### Best-First策略
搜索的过程，实际上是维护了一个优先队列，在这个优先级队列上对元素做操作。初始化只有一个树根在优先级队列。每次删除并搜索最优元素，把最优元素的子结点加入到优先级队列中。如果发现最优元素就是目标，那就找到了目标。  
从数据的结构看，就是在一个优先级队列上操作。从搜索树的角度看，总是所有的叶子结点在优先级队列中。找到叶子节点中最优的那个，继续探索它，它成为搜索树内部节点，探索它的子状态，即把它的子结点加入进来。Best-First，就意思是按全局考虑优先级来搜索。   
用二叉堆实现这个优先级队列，二叉堆的插入和删除操作都是O(log n)的。
#### 分支界限算法
为了求最优化问题，需要分支界限算法来减少搜索量。因为爬山法、Best-First策略都只是能够帮助更快的找到可能解，但是不保证最优。要想保证最优，只能求出所有解，再取最优的一个。但这样搜索量巨大，因此要用剪枝策略，即分支界限算法。  
分支界限算法先求出某个可能的解，然后这个解就是最优解的一个界限，然后继续搜索，如果搜索过程中发现评价测度已经超过了界限，那就不用继续搜索该条路径，直接剪枝，因为已经知道这条路径搜索下去不可能是最优解。此策略可以减少搜索量 。  
#### 剪枝技巧
用分支界限的思想剪枝，可以减少搜索量，但是减少了多少搜索量，与剪枝的实现是有关系的。同样的题目，用两种方法，一种可以很高效剪枝，但一种剪枝效果不好。  
例题：人员安排问题和旅行商问题，当代价是用一个矩阵表示的时候，把矩阵每行或每列减一个数，把矩阵做成每行都有一个0，每列都有一个0的样子，作为一个新的代价矩阵，而减去的数字总和，就是一个代价的下界。这样再搜索的时候，剪枝会更高效。
#### A* 算法
理解`A*`算法，首先要知道，`A*`算法是基于Best-First策略。 其次，`A*`算法是直接搜索出最优解的算法，这意味着没有剪枝策略，也不是某种加快搜索的策略，而是一种直接找到最优解的方法。ppt上的定理1，就是为了说明`A*`算法会直接发现最优解。  
A\*算法是做一个更好的评估函数，即考虑过去带来的影响，也考虑未来的影响。将已经明确的，过去搜索中积累过来的代价作为g(n)，将未来搜索的代价作为h\*n，真正的评价函数就是f\*(n)=g(n)+h\*(n).但是未来的这个代价是不保证的，我们只能做估计，所以实际上是估计当前能够看到的下界h(n)来参考,f(n)=g(n)+h(n).  
搜索的过程是对每个结点计算f(n)值，然后基于这个函数做Best-First策略搜索。如果Best-First策略选取了的结点，正好就是目标节点，那么我们就找到了目标节点，并且这是最优解。易错的地方，如果我们能够看到目标节点，并且计算了它的f(n)，此时还没有结束，应继续执行Best-First策略，直到真正选上目标节点为止。选到了目标节点，直接可以返回，没有再剪枝。


## 第二章 数学计算
### 递归树法
对于递归方程 $T(n)=aT(\frac{n}{b})+f(n)$,画出递归树。递归树的层数、叶子节点数求法：  
1. 递归树的层高：根结点到叶子，根节点对应问题规模为 $n$ ， 第二层的问题规模为 $\frac{n}{b}$ , 第三层的问题规模为 $\frac{n}{b^2}$,...,最后一层即叶子节点的问题规模是1，他就是 $\frac{n}{b^{log_bn}}$, 可见每一层的问题规模分母是 $b^0,b^1,b^2,...,b^{log_bn}$ ,所以层数总共 $1+log_bn$.
2. 叶子的个数：每一次递归，把问题分为a个子问题。相当于每往下一层，子问题数量是原来的a倍。 第一层问题数1，第二层问题数a,第三层问题数 $a^2$, ... , 最后一层问题数即叶子数是 $a^{log_bn}$. 在求完层数，肯定能把层号编成 $0,1,2,...,log_bn$, 第k层的问题数就是 $a^k$, 叶子数就是 $a^{log_bn}$. 根据 $a^{log_bn}=n^{log_ba}$，最后叶子数就是 $n^{log_ba}$

### Master定理
对于递归方程 $T(n)=aT(\frac{n}{b})+f(n)$, 直接比较 $n^{log_ba}$和 $f(n)$的大小。若 $n^{log_ba}$比 $f(n)$高阶，那么 $T(n)=\theta(n^{log_ba})$. 如果两者是同阶的，那么  $T(n)=\theta(n^{log_ba}log n)$, 若 $n^{log_ba}$比 $f(n)$低阶，那么 $T(n)=\theta(f(n))$.

## 第七章 平摊分析
三种平坦分析方法：聚集法，会计法，势能法。  
聚集法：直接算操作序列的总代价，然后平均分配。聚集法算出来每种操作的平摊代价是一样的。  
会计法：先给每种操作赋平摊代价，然后验证操作过程中存款总额总是非负。  
势能法：先定义势能函数，保证势能函数非负，然后用公式求平摊代价。  
#### 实例分析
栈操作势能函数：栈中的元素个数。二进制计数器的势能函数：'1'的个数。  
表的扩张与收缩的势能函数： 设表为T, T.num为表中已经装填的元素个数， T.size为表的大小，α为装填因子。那么势能函数是 2T.num-T.size , $(\alpha\geq\frac{1}{2})$，以及T.size/2-T.num ( $\frac{1}{4}\leq\alpha < \frac{1}{2}$)  
动态表的插入和删除，一共有八种情况，在动态表执行任意n个操作的实际运行时间是O(n).  
$\alpha\geq\frac{1}{2}$时插入，不扩张， $c_i=1,\Phi(D_i)-\Phi(D_{i-1})=(2T_i.num-T.size)-(2T_{i-1}.num-T.size)=2,c_i^{\prime}=c_i+2=3$    

$\alpha\geq\frac{1}{2}$时插入，扩张, 这里隐含着的等式有 $T_{i-1}.num=T_{i-1}.size,T_{i}.size=2T_{i-1}.size$, 先把元素拷贝到新表，再添加一个元素， $c_i=T_{i-1}.size+1,\Phi(D_i)-\Phi(D_{i-1})=(2T_i.num-T_i.size)-(2T_{i-1}.num-T_{i-1}.size)=2-T_{i-1}.size,c_i^{\prime}=c_i+2-T_{i-1}.size=3$  

$\alpha_{i-1}<\frac{1}{2}$时插入,并且 $alpha_{i}\geq\frac{1}{2}$时，这里隐含的等式 $T_i.num=T_{i-1}.num+1$,T.size是一样的，并且要用到 $\alpha_{i-1}=\frac{num}{size}<\frac{1}{2}$ .这样  $\Phi(D_i)-\Phi(D_{i-1})=(2T_i.num-T_i.size)-(\frac{1}{2}T_{i-1}.size-T_{i-1}.num)=3T_{i-1}.num+2-\frac{3}{2}T.size<\frac{3}{2}T.size+2-\frac{3}{2}T.size=2,c_i^{\prime}< c_i+2=3$ 。   

$\alpha_{i-1}<\frac{1}{2}$时插入,并且 $alpha_{i}<\frac{1}{2}$时, $c_i=1,\Phi(D_i)-\Phi(D_{i-1})=(\frac{1}{2}T_i.size-T_i.num)-(\frac{1}{2}T_{i-1}.size-T_{i-1}.num) = -1, c_i'=c_i+1=0$.   

$\alpha<\frac{1}{2}$时删除，不收缩， $c_i = 1, \Phi(D_i)-\Phi(D_{i-1})=(\frac{1}{2}T_i.size-T_i.num) -(\frac{1}{2}T_{i-1}.size-T_{i-1}.num)=1,c_i'=c_i+1=2$

$\alpha<\frac{1}{2}$时删除，导致了收缩，那么隐含的等式有 $2T_i.size = T_{i-1}.size, c_i=T_i.num=T_{i-1}.num-1,T_i.size=2T_{i-1}.num$,计算代价， $\Phi(D_i)-\Phi(D_{i-1})=(\frac{1}{2}T_i.size-T_i.num) -(\frac{1}{2}T_{i-1}.size-T_{i-1}.num)=-\frac{1}{2}T_i.size+1,c_i=\frac{1}{2}T_i.size,c_i'=1$

$\alpha_i\geq\frac{1}{2}$,删除，这种情况 $c_i=1,\Phi(D_i)-\Phi(D_{i-1})=2T_i.num-T_i.size-2T_{i-1}.num-T_{i-1}.size=-2 , c_i'=c_i-2=-1$ 

$\alpha_{i-1}\geq\frac{1}{2}, \alpha_i<\frac{1}{2}$,删除,  $c_i=1, \Phi(D_i)-\Phi(D_{i-1})=\frac{1}{2}T_i.size-T_i.num-2T_{i-1}.num+T_{i-1}.size=\frac{3}{2}T_i.size-3T_{i-1}.num+1<1$,所以 $c_i^{\prime}< c_i+1=2$

