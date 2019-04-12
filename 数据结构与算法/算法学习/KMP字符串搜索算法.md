> 这个算法总结梳理了半个月,虽然痛苦但是打通了好多个点,我认为最难的点在如何求`最大长度值表`,不管是`PMT表`还是`next表`,都是辅助我们求解问题,如果能看懂主要原理和流程的同学,我认为其实理解了KMP 就是不知道怎么实现?为什么这样实现?

# 暴力匹配
> 最简单的是使用暴力匹配,也就是不管前面匹配了多少位,只要发生了不匹配,主串(指针) i 就要回溯到开始的下一位继续匹配,这边代码就不放了=>[暴力破解字符串匹配代码](https://github.com/LowApe/JavaAlgorithmImplementation/blob/master/src/BruteForceStringMatching.java)

# KMP 算法

> KMP 算法通过匹配`要搜索字符串的子串`寻找不匹配前`相同前缀的位置`来提高效率,利用已经部分匹配这个有效信息，保持i指针不回溯，通过修改j指针，让模式串尽量地移动到有效的位置。下面举个例子


例如: 主串: asdasdasdadasda<br>
搜索字符串:asdasb<br>


![](http://ww1.sinaimg.cn/large/006rAlqhly1g1onlwz9klj30bn05bq2v.jpg)

## 那么什么是最大长度串?

基本流程通过上面了解,我们首先需要找到**不匹配位置前,子串的一个规律,从图中我们发现,这个子串有相同的内容,而我们需要移动的就是第二个相同的位置**,

> 不着急,我还需要通过具体描述来说明.
> 根据上图具体说明:当 i 和 j 下标的数据不相等(s[i]!=p[j]) 具体值使 d!=c,然后找到子串"a s d a s",有个`相同连续且最长字符串`的地方"as",这个子串(说的是"a s d a s")已经是匹配了的,所以整个搜索串移动到第二个"as"的位置.

那么也就是说,当我们发生不匹配的时候,就要查看子串有没有这个最大长度串,如果没有,整体移动到 i 之后,如果有,**j 就移动到最大长度串之后的位置**(因为最大长度串是匹配的,所以从它后以为开始比较)

整个流程得到一个小公式

```
j=PMT[j-1]
j:需要比较的位置(下一个比较的位置)
PMT:理解为发生不匹配位置前的`匹配串`最大长度值的数组(后面会说怎么求)
j-1:失配前的最大长度值下标
```

综上我们需要得到一个数组,存放最大长度值,`这个数组就是核心部分`.有了这个表,就能知道比较的下一个位置,所有问题好解决了

****

## 部分匹配表-PMT


那如何求这个最大长度的串呢?从而求出最大长度值呢?:
通过前后缀匹配的方式:
正确前缀: "ababa" 的正确前缀除最后一个之前的分解字符串:"a","ab","aba","abab";
正确后缀:"ababa" 的正确后缀除第一个之前的分解字符串:"a","ba","aba","baba";

![](http://ww1.sinaimg.cn/large/006rAlqhly1g1hhb67w8jj30fr05tweh.jpg)

部分匹配值=前后缀`交集`最长那个字符串的个数(上面的前后缀匹配得有"a","aba" 最大那个长度就是 3)
![](http://ww1.sinaimg.cn/large/006rAlqhly1g1hhd94u6yj30e504nmx3.jpg)


> 前缀和后缀数组交集中最长的串的个数,就是最大长度值,那如何求这个集合?下面是代码,其中有一个概念在代码之后讲解.

```java
public class PMT {
    static int[] pmt;
    public static int getPMT(char[] s,char[] p){
        int i=0;
        int j=0;
        int sLen = s.length;
        int pLen = p.length;
        while (i<sLen&&j<pLen){
            //j==0 遗漏第一位没有比较
//            表示没有最大长度值,整体 i 和 j 向后移
            if(j==-1||s[i]==p[j]){
                i++;
                j++;
            }else{
                //如果 s[i]!=p[j] 查看最大长度值
                j--;
                if(j==-1){
                    continue;
                }else{
                    j=pmt[j];
                }
            }
        }
        if(j==pLen){
            return i-j;
        }
        return -1;
    }

    public static void getMatchLength(char[] p,int pmt[]){
        //k是最大长度值 初始值0表示没有最大长度值
        int k=0;
        int pLen = p.length;
        //j初识下标为1,因为第一位没有前后缀,p[k],p[j]指向同一个数
        int j= 1;
        pmt[0]=0;
        //循环目的是把整个搜素串的各个位置的最大长度值求出来
        while(j<pLen){

            if(p[j]==p[k]){
                //如果 p[j]==p[k],表示当前符合前后缀k++,赋值
                k++;
                pmt[j]=k;

            }else {
                //如果 p[j]!=p[k],表示没有最大长度值,赋值k
                k=pmt[k];
            }
            j++;
        }
    }

    public static class TestPMT{
        public static void main(String[] args) {
            String s="dabadcabecbaabadcabfasdads";
            String p="abadcabf";
            pmt = new int[p.length()];
            getMatchLength(p.toCharArray(),pmt);
            System.out.println("PMT 局部匹配表");
            for (int i = 0; i < pmt.length; i++) {
                System.out.print(pmt[i]+"\t");
            }
            System.out.print("\n");
            System.out.println("首次出现的位置"+getPMT(s.toCharArray(),p.toCharArray()));
        }
    }
}


```
### 为什么进行p[k]==p[j]判断?

假设存在 k 值,`k为所求最大长度值`,则应该满足 `p[0~k-1]==p[j-k~j-1]`,如果p[k]==p[j] 说明前面那个式子 k 不是最大长度值,如果 p[k]!=p[j] ,k就是最大长度值,有点绕口,
为什么是这个等式:我们思考上面的特点,不管这个最大长度串有多长,前后缀都是`相同,连续且最长`,`p[0~k-1]`为前缀`p[j-k~j-1]`为后缀,注意j是不匹配位置所以这里是 j-1,p[k] 就是前缀下一个位置,p[j] 就是后缀下一个位置.

通过下图理解上面的公式:
![](http://ww1.sinaimg.cn/large/006rAlqhly1g1wt46wir5j30dv0fadg1.jpg)
![](http://ww1.sinaimg.cn/large/006rAlqhly1g1wt4gzjz9j30eh08rmx6.jpg)

> 说明:图上的例子很短,所以造成 k 值并不在前后缀中间,这没关系,一定要理解,我们是要求出最大长度值,也就是到 `p[k]!=p[j]`


****

## 从部分匹配表推出 next 数组
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
之前的公式
```
j=PMT[j-1]
因为next 数组整体后移一位,所有j刚好就是失配前最大长度值
j=next[j]
```

### 2. 为什么`next[0]= -1` ?
 next 数组[0] 索引下的值为 -1,我是这样理解的,当整体向后移动,第一位的值表示的含义就不是**失配前最大长度值**,因为在它前面没有子串,所有这表示没有可以匹配的子串,也就是意味无法找子串的移动位置,那找不到,就整体向后移,所有`next[0]= -1`是一个标志量,标志我们主串 i 向后移动.

> 别人的解释:当有匹配的位置就返回它的具体位置，否则返回-1（常用手段）

### 3. 如何理解next 优化 `next[j]=next[k]`?

![](http://ww1.sinaimg.cn/large/006rAlqhly1g1wyt1mr2nj30kq07wwfq.jpg)

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
> 这可能是本文我最不理解的地方,如果有弄懂的小伙伴,可以在[博客留言板留言](https://blogcode.top/guestbook/),谢谢了(*^__^*) 嘻嘻……

****

# 难点 list

## 1. 局部匹配值的理解
## 2. 注意移动位数与直接指向位置(本文全为指向位置)
## 3. 为什么 next 数组整体向后移位
## 4. 为什么 next 数组[0] 索引下的值为 -1
## 5. <div style="color:red;display:inline">*获取最大长度值的过程<div>

****

参考连接:<br>
- [所有代码 GitHub](https://github.com/LowApe/JavaAlgorithmImplementation)
- [blogcode.top](https://blogcode.top/)
- [jakeboxer 英文理解](http://jakeboxer.com/blog/2009/12/13/the-knuth-morris-pratt-algorithm-in-my-own-words/)<br>
- [阮一峰的理解](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)<br>
- [从头到尾彻底理解KMP（2014年8月22日版）](https://blog.csdn.net/v_july_v/article/details/7041827)
