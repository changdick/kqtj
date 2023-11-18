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

## 第四章 动态规划
用动态规划算法的问题特征： 最优化问题。  
适用动态规划需满足： 最优子结构性质，子问题重叠。  
做题关键：找出一种好的优化解的**定义**，这种定义有最优子结构性质，方便自底向上计算。  
