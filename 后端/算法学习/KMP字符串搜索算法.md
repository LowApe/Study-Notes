# KMP 算法

> 如果想我一样懂原理,但是在找有几个难点的可以直接跳到下面的难点问题上,看看我的思路

## 暴力匹配

所谓暴力匹配就是通过匹配主串S,如果部分匹配,从匹配首位置的下一个开始重写匹配,这种匹配模式效率最低,如果匹配字符串P字段中并没有`重复的-部分匹配值为0`(下面的部分匹配表概念),就会存在不必要的比较.然后有了KMP算法

## KMP 算法
KMP 算法介绍

详细图解可以看看[阮一峰-字符串匹配的KMP算法](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)

下面给出原理的代码
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
               //关键词移动
               j=next[j];
           }
       }
       if(j==pLen){
           return i-j;
       }
       return -1;
   }
```

接下来就是难点部分,你说流程都懂,但是所谓那个表怎么求?next数组怎么得到?

## 部分匹配表-PMT数组
```
The length of the longest proper prefix in the (sub)pattern that matches a proper suffix in the same (sub)pattern.
```



这句话是部分匹配表的精髓,前缀和后缀匹配最大长度值,**这个最大长度值就是子匹配串出现的最大子串**,如何理解这句话,如下图"aba"的最大长度值为1,当出现不匹配可以移动到第二个"a","abab"最大长度值为2,当出现不匹配可移动到第二个的"ab";

![](http://ww1.sinaimg.cn/large/006rAlqhly1g1hhd94u6yj30e504nmx3.jpg)


先理解下面两个概念:
正确前缀: "ababa" 的正确前缀除最后一个之前的分解字符串:"a","ab","aba","abab";

正确后缀:"ababa" 的正确后缀除第一个之前的分解字符串:"a","ba","aba","baba";

部分匹配值=前后缀交集最大那个字符串的长度-上面的前后缀匹配得有"a","aba" 最大那个长度就是 3

![](http://ww1.sinaimg.cn/large/006rAlqhly1g1hhb67w8jj30fr05tweh.jpg)


现在得到了这个最大长度值,如何来用?

![](http://ww1.sinaimg.cn/large/006rAlqhly1g1itap47ddj30fx03kwed.jpg)
比如当在上图空格的位置上出现不匹配,之前都是匹配的,将下面的搜索串移动

```
移动的位数=已经匹配的个数-失配前一个的最大长度值;
```

## next数组
当出现部分匹配,可以通过上面的公式进行新位置的匹配


```
 具体的做法是，保持i指针不动，然后将j指针`指向`模式字符串的PMT[j −1]位即可。
```
`PMT`:我们需要算出的`部分匹配表`数组

已匹配的字符数: j-1<br>
失配所在的位置-next[失配所在的位置],且此值大于等于1<br>
PMT[j −1]相当于`移动位数`

通过`部分匹配表` 我们觉得每次去找上一个的最大长度值比较麻烦,于是我们直接将PMT数组整体后移1位(且首值赋值为-1)得到`next数组`,

**1. j 从0开始技术,所以j就是已经匹配的个数**

**2. 当具体某个位置不匹配,通过next数组对应的值,就是失配前一个的最大长度值**

这两个也就是为什么 PMT 转换成 next 的主要原因,我认为还是因为这样好算,不用考虑向前移动一个得到值
![](http://ww1.sinaimg.cn/large/006rAlqhly1g1hiuttf8gj30gh06zglq.jpg)

## 为什么next[0]= -1 ?
 next 数组[0] 索引下的值为 -1


```java
public static void getNext(char[] p,int next[]){
        int k=-1;
        int pLen=p.length;

        next[0]=-1;
        int j=0;
        while (j<pLen-1){
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



****

# 难点

## 1. 局部匹配值的理解
## 2. 移动位数
公式在算法里面 直接就是移动到了那个位置,最大长度值刚好是需要移动到下一个部分匹配的位置
## 2. next 数组整体向后移位
为了好得到匹配了的部分字符串的下一个位置(奇怪的是通过next数组后得到的那个最大长度值,就是对应到部分匹配的下一个的位置上)
## 3. next 数组[0] 索引下的值为 -1
## 4. 理解 k=next[k]


参考连接:<br>
[jakeboxer 英文理解](http://jakeboxer.com/blog/2009/12/13/the-knuth-morris-pratt-algorithm-in-my-own-words/)<br>
[阮一峰的理解](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)<br>
[如何更好的理解和掌握 KMP 算法?](https://www.zhihu.com/question/21923021)
