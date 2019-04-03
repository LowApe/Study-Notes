> 这个算法总结梳理了半个月,虽然痛苦但是打通了好多个点,我认为最难的点在如何求`最大长度值表`,不管是`PMT表`还是`next表`,都是辅助我们求解问题,如果能看懂主要原理和流程的同学,我认为其实理解了KMP 就是不知道怎么实现?为什么这样实现?

# KMP 算法
> KMP 算法通过匹配`关键字符串的子串`寻找不匹配前`相同前缀的位置`来提高效率


例如: 主串: asdasdasdadasda<br>
需要搜索的关键字符串:asdasb<br>
下图所示当最后移位不匹配时,搜索串不需要一位一位的移动,需要找到之前`匹配串`中最大长度的`子串`(重复出现的最大子串)

![](http://ww1.sinaimg.cn/large/006rAlqhly1g1onlwz9klj30bn05bq2v.jpg)

详细原理图解可以看看[阮一峰-字符串匹配的KMP算法](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)



基本流程通过上面了解,我们首先需要找到**不同不匹配位置的**最大长度的`子串`
从而需要得到一个表,这个表就是核心部分.

****

## 1. 部分匹配表-PMT

因为我们通过`匹配关键字的子串`寻找不匹配前相同前缀的位置
前缀和后缀匹配最大长度值,**就是前面说的`已匹配的子串`中出现`重复`的最大子串长度**

表中的部分匹配值是当不匹配的位置的前缀(已匹配的最大长度)

那如何求这个最大长度的串呢?从而求出最大长度值呢?:
通过前后缀匹配的方式:
正确前缀: "ababa" 的正确前缀除最后一个之前的分解字符串:"a","ab","aba","abab";
正确后缀:"ababa" 的正确后缀除第一个之前的分解字符串:"a","ba","aba","baba";

部分匹配值=前后缀交集最大那个字符串的长度-上面的前后缀匹配得有"a","aba" 最大那个长度就是 3

![](http://ww1.sinaimg.cn/large/006rAlqhly1g1hhb67w8jj30fr05tweh.jpg)


![](http://ww1.sinaimg.cn/large/006rAlqhly1g1hhd94u6yj30e504nmx3.jpg)

现在得到了这个最大长度值,当发生不匹配位置时,如何通过它得到需要移动的位置?
```
移动的位数=已经匹配的个数-失配前一个的最大长度值;
上图移动的位数就是=6-2=4
```
> **注意上面是移动的位数而不是移动的位置**

当出现部分匹配,可以通过上面的公式进行新位置的匹配
```
具体的做法是，保持i指针(主串的指针)不动，然后将j指针(搜素串)`指向`模式字符串的PMT[j −1]位即可。
```

`PMT`:我们需要算出的`部分匹配表`数组
已匹配的字符数: j-1<br>
PMT[j −1]相当于`需要比较的位置`

此时可以通过这个 PMT 数组就能进行算法的实现,这里不做讨论.
- [PMT 数组实现传送门]()

****

## 2. 从部分匹配表推出 next 数组
上面就是 kmp 的一个核心思想,那现在优化这个 pmt 数组

通过`部分匹配表` 我们觉得每次去找上一个的最大长度值比较麻烦,于是我们直接将PMT数组整体后移1位(且首值赋值为-1)得到`next数组`,

**1. j 从0开始计数,所以j就是已经匹配的个数**

**2. 当具体某个位置不匹配,通过next数组对应的值,就是失配前一个的最大长度值**

这两个也就是为什么 PMT 转换成 next 的主要原因,我认为还是因为这样好算,不用考虑向前移动一个得到值
![](http://ww1.sinaimg.cn/large/006rAlqhly1g1hiuttf8gj30gh06zglq.jpg)

****

## 3. 具体代码实现过程

下面给出原理的代码,先忽略next数组(把他理解为求得我们要移动的位置),还有一个`j==-1` 先理解为没有可匹配的子串
```java
public static int KMPSearch(char[] s,char[] p){
       int i=0;
       int j=0;
       int sLen=s.length;
       int pLen=p.length;
       while (i<sLen && j<pLen){
           if(j==-1 || p[j]==s[i]){
               i++;
               j++;

           }else{
               //关键词比较的位置
               j=next[j];
           }
       }
       if(j==pLen){
           return i-j;
       }
       return -1;
   }
```

```java
public static void getNext(char[] p,int next[]){
    // k 是最大长度值,通过下面求出k的值并存入数组
        int k=-1;
        int pLen=p.length;

        next[0]=-1;
        int j=0;
        while (j<pLen-1){
            //当前或
            if(k==-1||p[j]==p[k]){
                ++k;
                ++j;
                next[j]=k;
            }else{
                k = next[k];
            }
        }
    }
```

### 1. 为什么是  `j=next[j]`?


![图1](http://ww1.sinaimg.cn/large/006rAlqhly1g1itap47ddj30fx03kwed.jpg)

比如当在上图空格的位置上出现不匹配,之前都是匹配的,将下面的搜索串移动,需要弄明白一个点:**所谓移动也只是搜素关键串移动,j会发生变化,i 是不变的,移动的位数并不是j 的下标;**

![图2](http://ww1.sinaimg.cn/large/006rAlqhly1g1p7qyyc8ij30jq04f0t3.jpg)


这里因为如上图移动四位j的下标变成了2<p style="color:red;display:inline" >这刚好就是这个最大长度值</p>,所以`j=next[j]`

### 2. 为什么`next[0]= -1` ?
 next 数组[0] 索引下的值为 -1,我是这样理解的,当整体向后移动,第一位的值表示的含义就不是**失配前最大长度值**,因为在它前面没有子串,所有这表示没有可以匹配的子串,也就是意味无法找子串的移动位置,那找不到,就整体向后移,所有`next[0]= -1`是一个标志量,标志我们主串 i 向后移动.


### 3. 如何通过递推得到最大长度值

```java
public static void getNext(char[] p,int next[]){
    // k 是最大长度值,通过下面求出k的值并存入数组
        int k=-1;
        int pLen=p.length;

        next[0]=-1;
        int j=0;
        while (j<pLen-1){
            //当前或
            if(k==-1||p[j]==p[k]){
                ++k;
                ++j;
                /*
                next 优化
                */
                if (p[j]!=p[k]){
                    next[j]=k;
                }else{
                    next[j]=next[k];
                }
            }else{
                k = next[k];
            }
        }
    }
```

> flag: 最后只一步搞懂,就能修炼成功!!!!

****

# 难点 list

## 1. 局部匹配值的理解
## 2. 移动位数与比较位置区别
## 3. 为什么 next 数组整体向后移位
## 4. 为什么 next 数组[0] 索引下的值为 -1
## 5. <div style="color:red;display:inline">*获取最大长度值的过程<div>

****

参考连接:<br>
[jakeboxer 英文理解](http://jakeboxer.com/blog/2009/12/13/the-knuth-morris-pratt-algorithm-in-my-own-words/)<br>
[阮一峰的理解](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)<br>
[如何更好的理解和掌握 KMP 算法?](https://www.zhihu.com/question/21923021)
