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
