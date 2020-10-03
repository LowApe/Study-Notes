# 集合

设计的内容

- Java 集合框架
- 视图与包装器
- 具体的集合
- 算法
- 映射
- 遗留的集合

# Java集合框架(接口)



### 将集合的接口与实现分离

> 与现代数据结构类库的常见情况一样，**Java集合类库也将接口(interface)与实现分离(implementation)**

**队列示例：**

```java
public interface Queue<E>{
    void add(E element);
    E remove();
    int size();
}
```

> 这个接口并没有说明怎么实现队列。**队列通常有两种实现方式:一种是使用循环数组；另一种是使用链表**

```java
/* 循环实现 */
public class CircularArrayQueue<E> implements Queue<E>{
    private int head;
    private int tail;
    private E[] elements;
    
    CircularArrayQueue(int capacity){...}
    public void add(E element){...}
    public E remove(){...}
    public int size(){...}
}

/* 链表实现 */
public class LinkedListQueue<E> implements Queue<E>{...}
```

> 可以使用**接口类型**存放集合的引用,这样的好处是如果改变实现方式，这需要改一个地方

```java
Queue<Customer> expressLane = new CircularArrayQueue<>(100);
Queue<Customer> expressLane = new LinkedListQueue<>();
```

> 书上说 API 文档，Abstract 开头的类是**为类库实现者而设计的**，那现阶段理解，实现是直接使用，抽象是增加功能

## Collection接口

这个接口比较重要的两个基本方法:

```java
public interface Collection<E>
{
    boolean add(E element);
    Iterator<E> iterator();
    ...
}
```

> 1. 向集合添加一个元素
> 2. 返回一个实现了 Iterator 接口的对象，通过这个迭代器对象依次访问集合的元素

**Collection 代码分析**

> 有些方法是 `destructive【毁灭性的】`,当操作集合的方法，不支持操作时，会抛出`UnsupportedOperationException` 异常
>
> For example, invoking the [`addAll(Collection)`](../../java/util/Collection.html#addAll-java.util.Collection-) method on an unmodifiable【不可修改的集合】 collection may, but is not required to, throw the exception if the collection to be added is empty.
>
> 实现了 Collection 的类对所包含的元素有些约束。比如，一些实现类禁止由 null  元素，或者其他约束元素的条件。如果尝试插入 ineligible 元素会抛出非检查异常，典型的异常 `NullPointException`和`ClassCastException`
>
> Such exceptions are marked as "optional" in the specification for this interface.
>
> 此类异常在接口中标记了可选的
>
> 一些方法在接口中都会用到 `equals` 方法，比如，具体方法 `contains(Object o)`方法返回包含元素的。他们则 `return (o==null?e==null : o.equals(e))` 在使用 equals 方法时使用`Object.hashCode)()`去判断对象是否相同



```java
package java.util;
public interface Collection<E> extends Iterable<E>{
    // 最大 Integer.MAX_VALUE;
    int size(); 
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    // 如果符合运行时具体的类型
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean addAll(Collection<? extends E> c);
    boolean remove(Object o);
    boolean removeAll(Collection<?> c);
    boolean containsAll(Collection<?> c);
    
    // 条件删除
    default boolean removeIf(Predicate<? super E> filter){
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while(each.hasNext()){
            if(filter.test(each.next())){
                each.remove();
                removed = true;
            }
        }
        return removed;
     }
    
    boolean retainAll(Collection<?> c);
    void clear();
    boolean equals(Object o);
    int hashCode;
    // 返回给定分隔符的分割迭代器
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }
    
    default Stream<E> stream(){
        return StreamSupport.stream(spliterator(), false);
    }
    
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
}
```

**Collection 类图**

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1giig6aaopfj30j60z2ju7.jpg)

**Collection 练习**

> 使用ArrayList练习：

```java
package core1.chapter9;


import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class CollectionPractice {
    public static Logger logger = LoggerFactory.getLogger(CollectionPractice.class);
    public static final int INIT_ARRAY_CAPACITY = 5;

    public static void main(String[] args) {
        // 初始化数据
        List<String> list = new ArrayList<>();
        for (int i = 0; i < INIT_ARRAY_CAPACITY; i++) {
            list.add(String.valueOf(i));
        }
        // list
        logger.info("list:{}\n", list);

        // size 长度
        logger.info("size:{}\n", list.size());

        // contains 是否包含单个迭代器
        logger.info("contains:{}\n", list.contains("value0"));

        // iterator 获得集合的迭代器
        logger.info("getIterator:{}\n", list.iterator());

        // toArray 转换数组
        Object[] a = list.toArray();
        logger.info("toArray:{}", Arrays.asList());
        logger.info("dealAfterSize:{}\n",a.length);

        // toArray(T[]) 场景：转换指定数组长度
        String[] b = new String[10];
        String[] afterArray = list.toArray(b);
        logger.info("toArray(T[]):{}",afterArray);
        logger.info("dealAfterSize:{}\n",afterArray.length);

        // add(E) 添加单个元素
        logger.info("add:{}",list.add("value5"));
        logger.info("list:{}", list);
        logger.info("size:{}\n", list.size());

        // remove(Object) 移除单个元素
        logger.info("remove:{}",list.remove("value5"));
        logger.info("list:{}", list);
        logger.info("size:{}\n", list.size());

        // containsAll(Collection<?>) 是否包含传入的集合
        List<String> containsList = new ArrayList<>();
        for (int i = 0; i < INIT_ARRAY_CAPACITY; i++) {
            containsList.add(String.valueOf(i));
        }
        logger.info("containsAll:{}",list.containsAll(containsList));

        // addAll(Collection<? extends E>) 添加所有元素
        List<String> addAllList = new ArrayList<>();
        for (int i = 0; i < INIT_ARRAY_CAPACITY; i++) {
            addAllList.add("value" + i);
        }
        logger.info("addAll:{}",list.addAll(addAllList));
        logger.info("list:{}", list);
        logger.info("size:{}\n", list.size());

        // removeAll(Collection<?>) 移除传入集合全部元素
        logger.info("removeAll:{}",list.removeAll(addAllList));
        logger.info("list:{}", list);
        logger.info("size:{}\n", list.size());

        // removeIf() 按条件的移除元素
        logger.info("removeIf():{}",list.removeIf(i->i.equals("2")));
        logger.info("list:{}", list);
        logger.info("size:{}\n", list.size());

        // retainAll() 只保留传入集合的元素
        List<String> retainList = new ArrayList<>();
        retainList.add("0");
        logger.info("retainAll():{}",list.retainAll(retainList));
        logger.info("list:{}", list);
        logger.info("size:{}\n", list.size());

        // clear()
        list.clear();
        logger.info("list:{}", list);
        logger.info("size:{}\n", list.size());

        // equals()
        List<String> equalList = new ArrayList<>();
        logger.info("equals:{}",list.equals(equalList));

        // hashCode()
        logger.info("hashCode:{}",list.hashCode());
        logger.info("hashCode:{}",addAllList.hashCode());
        logger.info("hashCode:{}",equalList.hashCode());
        logger.info("hashCode:{}",retainList.hashCode());

        // spliterator() 待定，等着 lambda 那边梳理出来
        list.spliterator();


        // stream()


        // parallelStream()


    }
}



```

**使用自定义的集合：**

```java

```



### List 接口

#### List 代码分析:

```java
public interface List<E> extends Collection<E>{
    // 最大 Integer.MAX_VALUE;
    int size(); 
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    // 如果符合运行时具体的类型
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean addAll(Collection<? extends E> c);
    boolean remove(Object o);
    boolean removeAll(Collection<?> c);
    boolean containsAll(Collection<?> c);
    
    // 条件删除
    default boolean removeIf(Predicate<? super E> filter){
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while(each.hasNext()){
            if(filter.test(each.next())){
                each.remove();
                removed = true;
            }
        }
        return removed;
     }
    
    boolean retainAll(Collection<?> c);
    void clear();
    boolean equals(Object o);
    int hashCode;
    // 返回给定分隔符的分割迭代器
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }
    
    default Stream<E> stream(){
        return StreamSupport.stream(spliterator(), false);
    }
    
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
    
    // 替换所有
    default void replaceAll(UnaryOperator<E> operator){
        Objects.requireNonNull(operator);
        final ListIterator<E> li = this.listIterator();
        while(li.hasNext()){
            li.set(operator.apply(li.next()));
        }
    }
    
    // 排序
    deafult void sort(Comparator<? super E> c){
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for(Object e: a){
            i.next();
            i.set((E)e);
        }
    }
    // 操作集合元素的接口
    E get(int index);
    E set(int index,E element);
    void add(int index,E element);
    E remove(int index);
    
    // 查询集合元素的接口
    int indexOf(Object o);
    int lastIndexOf(Object o);
    
    ListIterator<E> listIterator();
    // 指定位置的List迭代器
    ListIterator<E> listIterator(int index);
    
   	List<E> subList(int formIndex,int toIndex);
    
}
```



> 相比继承父类 Collection 多了用于随机访问的方法，和一些特殊方法,
>
> - ListIterator 接口是 Iterator 的一个子接口

#### List 类图

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj4cbuvfkzj30mc11s425.jpg)

#### List 练习

```java

```

### Set接口

#### Set 代码分析：

```java

```

#### Set接口类图





#### Set 练习

```java

```



## Map接口

### Map 代码分析：

>  A map cannot contain duplicate keys; 一个 map 对象不能包含完全相同的key
>
> 

### Map类图

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj4c2kh2wvj30re1bmq82.jpg)

### Map 练习



## Iterator迭代器

**Iterator 代码分析：**

```java
package java.util;

import java.util.function.Consumer;

public interface Iterator<E>{
    boolean hasNext();
    E next();
    default void remove(){
        throw new UnsupportedOperationException("remove");
    }
    default void forEachRemaining(Consumer<? super E> action){
        Objects.requireNonNull(action);
        while(hasNext())
            action.accept(next());
    }
}
```

> - next() 方法到达集合末尾，将抛出 `NoSuchElementException`，因此 `next` 之前调用 `hasNext()`方法。`next()`方法返回**越过的元素**
> - 因为所有的集合都实现了 Collection 接口，Collection 又扩展了带有 `forEach`的`Itrable`接口，所以集合都可以使用`for each` 来循环遍历
> - 除了`Iterable`接口的`forEach`,也可以通过获取集合迭代器的`forEachRemaining` lambda 循环遍历

> ⚠️注意：
>
> - 迭代器执行**向下进行**，操作不可逆，使用过后，不能继续使用
>
> - **元素顺序取决于集合类型**（如果对数组，迭代器则从索引0开始，如果是 `HashSet` **按照某种随机的次序出现**）
> - 虽然迭代过程中遍历所有元素，**但是无法确定元素被访问的次序**
> - **迭代器的查找和位置变更是紧密相连的**

**Iterator 类图**

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhly1gij9mpcc47j30n6090q3b.jpg)

**Iterator 练习**

> 使用 ArrayList 实现的 Iterator 方法

```java
package core1.chapter9;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.*;


public class IteratorPractice {

    public static Logger logger = LoggerFactory.getLogger(IteratorPractice.class);
    public static final Integer INIT_ARRAY_CAPACITY = 5;

    public static void main(String[] args) {
        // 初始化数据
        List<String> list = new ArrayList<>();
        for (int i = 0; i < INIT_ARRAY_CAPACITY; i++) {
            list.add("value" + i);
        }

        // hasNext next 两方法配合使用
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            String currentString = iterator.next();
            logger.info("default：{}", currentString);
            iterator.remove();
        }

        // forEachRemaining
        iterator.forEachRemaining(i -> {
            logger.info("forEachRemaining,{}", i);
        });

        // Iterable 的 forEach
        list.forEach(l -> {
            logger.info("forEach,{}", l);
        });

        System.out.println();

        // remove
        Iterator<String> remove1Iterator = list.iterator();

        // 使用 Iterator 的 forEachRemaining 移除
        remove1Iterator.forEachRemaining(r -> {
            remove1Iterator.remove();
        });
        logger.info("forEachRemainingResult:{}", list);

        // 使用 默认移除
        List<String> removeBeforeList = new ArrayList<>();
        for (int i = 0; i < INIT_ARRAY_CAPACITY; i++) {
            removeBeforeList.add("value" + i);
        }
        Iterator<String> remove2Iterator = removeBeforeList.iterator();
        while (remove2Iterator.hasNext()) {
            // remove 前如果没有 next() 会抛出 IllegalStateException
            remove2Iterator.next();
            remove2Iterator.remove();
        }
        logger.info("DefaultResult:{}", list);
    }
}

```

### ListIterator迭代器

#### ListIterator 代码分析：

```java
public interface ListIterator<E> extends Iterabtor<E>{
    boolean hasNext();
    E next();
    boolean hasPrevious();
    E previous();
    // 获取当前
    int nextIndex();
    int previousIndex();
    void remove();
    // set 替换游标越过的值
    void set(E e);
    // add 添加游标越过前的值
    void add(E e);
}
```

> 不同实现类的迭代器实现代码不同，
>
> - LinkedList 的链表迭代器的设计能够检测到其他迭代器修改过元素，会抛出一个 `ConcurrentModificationException`异常
> 	- 避免并发修改的异常，在每个迭代器方法开始处检查自己改写操作的技术值是否与集合的改写操作计数值一致，不一致，抛出一个 `ConcurrentModificationException`异常
> - 虽然链表的迭代器也提供随机访问元素get方法，但效率很低(size()/2 从尾部开始搜索),避免链表迭代器使用 get 方法
>
> ⚠️注意：
>
> next() 会移动迭代器位置
>
> - nextIndex() 获取游标前面(右边)索引，nextPrevious()获取后面(左边)，如果从左到右，再到坐可能存在索引为负(这是我的理解)
> 	- nextIndex 方法返回下一次调用 next 方法时返回元素的整数索引
> 	- previousIndex 方法返回下一次调用 previous 方法时返回元素的整数索引
> - add()添加到**游标前面位置**
> - 不能连续调用两次 remove
> - add 方法只依赖于迭代器的位置，而 remove 方法依赖于迭代器的状态

#### ListIterator 类图

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj4dc243g3j30ka0omjsx.jpg)

#### ListIterator 练习

> 使用 ArrayList 练习

```java
@Slf4j
public class ListIteratorPractice {
    private final static int INIT_CAPACITY = 5;

    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 1; i < INIT_CAPACITY; i++) {
            list.add("value" + i);
        }
        log.info("init List:{}", list);

        // 获得 ListIterator
        ListIterator<String> listIterator = list.listIterator();
        log.info("listIterator:{}", listIterator);

        // hasNext()
        while (listIterator.hasNext()) {
            String element = listIterator.next();
            log.info("nextIndex:{}", listIterator.nextIndex());
            log.info("hasNext-element:{}", element);
            // set 替换游标前面的元素
            listIterator.set("setValue");
        }

        System.out.println();
        // listIterator.add(()
        listIterator.add("addValue");

        log.info("{}",list);
        listIterator.previous();
        // remove()
        listIterator.remove();

        System.out.println();

        // hasPrevious()
        while (listIterator.hasPrevious()) {
            String element = listIterator.previous();

            log.info("previousIndex:{}", listIterator.previousIndex());
            log.info("hasPrevious-element:{}", element);
        }

        System.out.println();

        // 获取指定位置 ListIterator(int index);
        ListIterator<String> indexListIterator = list.listIterator(2);
        while (indexListIterator.hasNext()) {
            String element = indexListIterator.next();
            log.info("indexListIterator-element:{}", element);
        }


    }
}
```

> 使用 LinkedList 练习,见具体的集合类...

## RandomAccess

> 该接口不包含任何方法，用于测试特定具体集合是否支持高校的随机访问.在一些集合中，有两种有序集合，一种数组支持有序集合可以快速反问，一种链表支持有序

```java
if(c instanceof RandomAccess){
    use random access algorithm
}else{
    use sequential access algorithm
}
```



# 具体的集合类

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj5giji35fj31lm198n4k.jpg)

> 该图集合类承接上一节的接口

## LinkedList

> - 数组的缺陷在于删除一个元素，该元素之后所有元素都要向前移动，稍不注意可以造成删除时索引不正确
> - Java 程序设计这种，所有链表实际上都是**双向链接**的，每个结点还存放着指向前驱结点的引用
> - 只有对**自然有序**的集合使用迭代器添加元素才有实际意义。例如：Set时无序的，在Iterator 接口就没有 add 方法,而 ListIterator 接口包含 add 方法

### LinkedList 代码分析:

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>,Deque<E>,Cloneable,java.io.Serializable
{
    transient int size = 0;
    transient Node<E> first;
    transient Node<E> last;
    
    public LinedList(){
        
    }
    
    public LinedList(Collection<? extends E> c){
        this();
        addAll(c);
    }
    
    /**
     * 返回集合第一个元素
     * 1. 获取头结点
     * 2. 如果头结点为空说明链表为空，抛出不匹配元素异常
     * 3. 否则返回元素
     * @return the first element in this list
     * @throws NoSuchElementException if this list is empty
     */
    public E getFirst() {
        final Node<E> f = first;
        if(f == null)
            throw new NoSuchElementException();
        return f.item;
    }
    
    public E getLast() {
        final Node<E> l = last;
        if(l == null)
            throw new NoSuchElementException();
        return l.item;
    }
    
    public E removeFirst() {
        final Node<E> f = first;
        if(f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }
    
    public E removeLast() {
        final Node<E> l = last;
        if(l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
    
    public boolean contains(Object o){
        return indexOf(o) != -1;
    }
    
    public int size(){
        return size;
    }
    
    public boolean add(E e){
        linkLast(e);
        return true;
    }
    
    public boolean remove(Object o){
        if(o == null){
            for(Node<E> x = first,x!=null,x = x.next){
                if(x.item == null){
                    unlink(x);
                    return true;
                }
            }
        }else{
            for(Node<E> x = first,x!=null,x = x.next){
                if(o.equals(x.item)){
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
    
    public boolean addAll(Collection <? extends E> c){
        return addAll(size,c);
    }
    
     /**
     * 替换列表具体位置的元素
     * 1 检查
     * 2 查找具体索引下的结点
     * 3 获得旧元素
     * 4 替换新元素
     * 5 返回旧的元素
     * @param index index of the element to replace
     * @param element element to be stored at the specified position
     * @return 具体位置的元素先前的值
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E set(int index,E element){
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldValue = x.item;
        x.item = element;
        return oldValue;
    }
    /**
     * 列表插入具体的元素到具体的位置
     * 如果存在移动当前位置并且随后的元素在右边(添加一个索引)
     * 1 检查索引
     * 2 如果 index == size 尾结点插入
     *      linkLast 连接尾部
     * 3 else linkBefore(E e, Node<E> e); 在之前连接
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index,E e){
        checkPositionIndex(index);
        if(index == size){
            linklast(e);
        }else{
            linkBefore(e,node(index));
        }
    }
    /**
     * 列表具体位置移除元素。
     * 移动随后元素到左边(减去一个索引)
     * 返回列表这个移除的元素
     *
     * @param index the index of the element to be removed
     * @return the element previously at the specified position
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index){
        checkElementIndex(index);
        return unlink(node(index));
    }
    
    private int lastIndexOf(Object o){
        int index = size;
        if(o == null){
            for(Node<E> x = last;x!=null;x = x.prev){
                if(x.item == null){
                    return index;
                }
                
            }
        }else{
            for(Node<E> x = last;x!=null;x = x.prev){
                if(o.equals(x.item)){
                    return index;
                }
            }
        }
        return -1;
    }
    /**
     * 开始具体的位置插入所有元素。有重复则按照顺序
     * 1 检验位置索引 (不存在则抛出异常 IndexOutOfBoundsException)
     * 2 插入集合转为 Object[] 遍历使用
     * 3 定义添加集合数量
     * 4 如果数量为零，返回 false 表示没有添加内容
     * 5 定义两个结点: pred(插入前结点) succ(插入后结点)
     * 6 如果索引位置等于原大小总数(尾部插入)
     *      1. 设置succ = null  pred = last
     *      2. else succ = node(index)(拼接的尾部) pred =  succ.prev （拼接的头部）
     * 7 for 循环新的集合每一个结点并关联
     *      1 如果pred == null (集合为 null )
     *      2 first = newNode
     *      3 else pred.next = newNode (反向连接)
     *      4 pred = newNode (变动插入前结点)
     * 8 如果 succ == null (也就是全部插入尾部)
     *      else （也就是从中间插入）
     *      需要双向绑定
     * 9 总数量增加，修改数量修改
     * @param index index at which to insert the first element
     *              from the specified collection
     * @param c collection containing elements to be added to this list
     * @return {@code true} if this list changed as a result of the call
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @throws NullPointerException if the specified collection is null
     */
    public boolean addAll(int index,Collection <? extends E> c){
        checkPositionIndex(index);
        
        Object[] a = c.toArray();
        int newNum = a.length;
        if(newNum == 0){
            return false;
        }
        Node<E> pred,succ;
        if(index == size){
            succ = null;
            pred = last;
        }else{
            succ = node(index);
            pred = succ.prev;
        }
        for(Object o : a){
            E e = (E)o;
            Node<E> newNode = new Node<E>(pred,e,null);
            if(pred == null){
                first = newNode;
            }else{
                pred.next = newNode;
            }
            pred = newNode;
        }
        
        if(succ == null){
            last = pred;
        }else{
            pred.next = succ;
            succ.prev = pred;
        }
        size+=newNum;
        modCount++;
        return true;
    }
     /**
     * 从列表中移除所有元素
     * 在调用之后列表为空
     * 1 for 循环
     *      1 获得下一个结点，处理完后赋值
     *      2 三个数据域设置为 null
     *      3 赋值
     * 2 收尾结点设置为空
     * 3 总数设置为 0 ，modCount++;
     */
    public void clear(){
        for(Node<E> x = first;x!=null;){
            Node<E> next = x.next;
            x.item = null;
            x.prev = null;
            x.next = null;
            x = next;
        }
       first = last = null;
       size = 0;
        modCount++;
    }
    
    public E get(int index){
        checkPositionIndex(index);
        return node(index).item;
    }
    private void checkPositionIndex(int index){
        if(!isPositionIndex){
            throw new IndexOutOfBoundsException("这部分忽略");
        }
    }
    
    private void checkElementIndex(int index){
        if(!isElementIndex){
            throw new IndexOutOfBoundsException("这部分忽略");
        }
    }
    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }
    private boolean isPositionIndex(int index){
        return index>=0 && index<=size;
    }
    /**
     * 在具体索引返回非空结点
     * 1 size 右移1位( M >> n = M/2^n) (二分查找，>> 速度比算术运算符快)
     *      1 如果索引小于二分之一从前开始找
     *      2 else 从尾结点开始找
     * Returns the (non-null) Node at the specified element index.
     */
    Node<E> node(int index){
        if(index < (size >> 1)){
            Node<E> x = first;
            for(int i=0;i<index;i++){
                x = x.next;
            }
            return x;
        }else{
            Node<E> x = last;
            for(int i = size-1;i > index;i--){
                x = x.prev;
            }
            return x;
        }
    }
    /**
     * 断开结点连接
     * 1. 获得断开的元素
     * 2. 获得该结点的前后结点
     * 3. 分别处理前后结点是否为空(是否删除的是头结点或者尾结点)
     *      1. 如果为空
     *      2. 将next 设置头结点/将 prev设置为尾结点
     *      3. else 连接前后结点 并 断开前后结点的连接
     * 4. 修改数量，设置结点对象的数据引用为空
     */
    private E unlink(Node<E> x){
        E element = x.item;
        Node<E> next = x.next;
        Node<E> prev = x.prev;
        
        if(prev == null){
            first = next;
        }else{
            prev.next = next;
            x.prev = null;
        }
        
        if(next == null){
            last = prev;
        }else{
           next.prev = prev;
            x.next = null;
        }
        x.item = null;
        size--;
        modCount++;
        return element;
    }
    /**
     * 在列表返回当前第一个出现元素的索引,如果都不包含元素则返回 -1
     * 更正式的，最低会返回这样的
     * o == null ? get(i) == null:o.equals(get(i));
     * 否则没有包含返回 -1;
     * 1. 从索引 0 开始遍历
     * 2. 如果对象为 null
     *      1. for 循环遍历结点
     *      2. 如果找到 null 则返回 index (index 为正 true)
     * 3. else
     *      1. for 循环遍历结点
     *      2. 如果 o.equals(当前结点元素) 则返回 index (index 为正 true)
     * 4. return -1;（false）
     * @param o element to search for
     * @return the index of the first occurrence of the specified element in
     *         this list, or -1 if this list does not contain the element
     */
    public int indexOf(Object o){
        int index = 0;
        if(o ==null){
            for(Node<E> x = first,x != null,x = x.next){
                if(x==null){
                    return index;
                }
                index++;
            }
        }else{
            for(Node<E> x = first,x != null,x = x.next){
                if(o.equals(x.item)){
                    return index;
                }
                index++;
            }
        }
    }
    /**
     * 断开链表的第一个结点 f
     * 1. 检查f是头结点和第一个元素(目前确认是)
     * 2. 获得头结点的元素
     * 3. 获取下一结点(为断开作准备)
     * 4. 将结点的元素、下一集结点设置为空(单向断开)
     * 5. next 结点设置为头结点
     * 6. 如果 next 为空，则尾结点为空
     * 7. 否则下一结点前一结点设置为空(反向断开)
     * 8. 整体数量和修改数量改动
     */
    private E unlinkFirst(Node<E> f) {
        final E element = f.item;
        Node<E> next = f.next;
        f.item = null;
        f.next = null; // GC
        first = next;
        if(next == null){
            last = null;
        }else{
            next.prev = null;
        }
        size--;
        modCount++;
        return element;
    }
    
    public E unlinkLast(Node<E> l){
        final E element = l.item;
        Node<e> prev = l.prev;
        l.item = null;
        l.prev = null;
        last = prev;
        if(prev == null){
            first = null;
        }else{
            prev.next = null;
        }
        size--;
        modCount++;
        return element;
    }

     /**
     * 头结点链接(从头结点链接一个元素-头插法):
     * 1. 定义一个 f 变量获取头结点 first
     * 2. 创建一个新的结点与头结点进行关联
     * 3. 将新的结点赋值给头结点 (头插法，新的结点总是作为新的头部)
     * 4. 判断:
     *      1. 如果 f == null (头结点从来没使用过)
     *      2. 这个新的结点也是尾结点
     *      3. else (头结点已经存在了)
     *      4. f.prev = newNode (双向链表，所有新结点还需要链接到 first上)
     * 5. 链表 szie + 1
     * 6. 修改 modCount + 1
     * @param e 元素element
     */
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }

    /**
     * 尾部链接元素(尾插法)
     * 1. 定义变量 l 获取尾结点 last
     * 2. 构造这个新的结点
     * 3. last = newNode; 构造的结点就是新的尾结点
     * 4. 判断
     *      1. l == null
     *      2. 第一个结点即是头结点也是尾结点
     *      3. else
     *      4. l.next = newNode 单向尾结点，变成双向
     * @param e 元素
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
    
    
    // Queue operations. 队列的操作
    public E peek(){
        Node<E> f = first;
        return f == null ? null:f.item;
    }
    
        /**
     * 检索，但不移除列表第一个元素
     * 不能返回 null
     * Retrieves, but does not remove, the head (first element) of this list.
     *
     * @return the head of this list
     * @throws NoSuchElementException if this list is empty
     * @since 1.5
     */
    public E element() {
        return getFirst();
    }
    
    public E poll(){
        final Node<E> f = first;
        return f == null? null:unlinkFirst(f);
    }
    
    public E remove(){
        return removeFirst();
    }
    
    public boolean offer(E e) {
        return add(e);
    }
    
    // Deque operations 双端队列的操作
    // 在列表最前部分插入元素
    public boolean offerFirst(E e){
        addFirst(e);
        return true;
    }
    // 在列表尾部插入元素
    public boolean offerLast(E e){
        addLast(e);
        return true;
    }
    public E peekFirst(){
        final Node<E> f = first;
        reutrn f == null ? null : f.item;
    }
    
    public E peekLast(){
        final Node<E> l = last;
        reutrn l == null ? null : l.item;
    }
    
    public E pollFirst(){
        final Node<E> f = first;
        reutrn f == null ? null : unlinkFirst(f);
    }
    
    public E pollLast(){
        final Node<E> l = last;
        reutrn l == null ? null : unlinkLast(l);
    }
    // 将这个元素压入堆栈代表的列，换句话说，插入列表头部
    public void push(E e){
        addFirst(e);
    }
    public E pop() {
        return removeFirst();
    }
    
    public boolean removeFirstOccurrence(Object o) {
        return remove(o);
    }
    public boolean removeLastOccurrence(Object o) {
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }
    
private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : node(index);
            nextIndex = index;
        }

        public boolean hasNext() {
            return nextIndex < size;
        }

        /**
         * 1 检查修改数与期望数是否一，不一致抛出并发修改异常
         * 2 如果 hasNext 不存在，则抛出 NoSuchElementException
         * 3 lastReturned = next
         * 4 next = next.next
         * 5 nextIndex++: 下一个索引数+1
         * 6 返回上一个(越过的值)
         * @return
         */
        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }

        /**
         * 是否存在上一个
         * @return
         */
        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        /**
         * 返回上一个结点
         * 1 检查并发修改
         * 2 检查元素匹配
         * 3 lastReturned
         * 4 nextIndex--
         * 5 返回越过的元素
         * @return
         */
        public E previous() {
            checkForComodification();
            if (!hasPrevious())
                throw new NoSuchElementException();

            lastReturned = next = (next == null) ? last : next.prev;
            nextIndex--;
            return lastReturned.item;
        }

        public int nextIndex() {
            return nextIndex;
        }

        public int previousIndex() {
            return nextIndex - 1;
        }

        /**
         * 删除当前结点
         * 1 检查并发修改
         * 2 lastReturned 是否存在 抛出 IllegalStateExceptions
         * 3 获得 lastNext 结点(删除前下一个结点)
         * 4 断开 lastReturne（删除）
         * 5 如果 next ==  lastReturned （lastReturned 要被删除，不能为next结点）？
         *      next = lastNext
         * 5 else nextIndex-- 移除后索引减1
         * 6 移除的 lastReturned = null
         * 7 预期移动数量++
         */
        public void remove() {
            checkForComodification();
            if (lastReturned == null)
                throw new IllegalStateException();

            Node<E> lastNext = lastReturned.next;
            unlink(lastReturned);
            if (next == lastReturned)
                next = lastNext;
            else
                nextIndex--;
            lastReturned = null;
            expectedModCount++;
        }

        /**
         * 替换元素
         * 1 检查越过的结点
         * 2 检查并发修改
         * 3 替换越过元素
         * @param e
         */
        public void set(E e) {
            if (lastReturned == null)
                throw new IllegalStateException();
            checkForComodification();
            lastReturned.item = e;
        }

        /**
         * 1 检查并发修改
         * 2 设置 lastReturned = null ？
         * 3 if next == null 尾部连接
         * 4 else linkBefore(e,next) 中部连接
         * 5 nextIndex ++
         * 6 expectedModCount++;
         * @param e
         */
        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            if (next == null)
                linkLast(e);
            else
                linkBefore(e, next);
            nextIndex++;
            expectedModCount++;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (modCount == expectedModCount && nextIndex < size) {
                action.accept(next.item);
                lastReturned = next;
                next = next.next;
                nextIndex++;
            }
            checkForComodification();
        }

        /**
         * 检查修改的数量与期望是否一致
         * 1 modCount != expectedModCount throw ConcurrentModificationException
         */
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
    public Object[] toArray(){
        Object[] result = new Object[size];
        int i = 0;
        for(Node<E> x = first;x!=null; x = x.next){
            result[i++] = x.item;
        }
        return reuslt;
    }
    private static class Node<E>{
        E item;
        Node<E> next;
        Node<E> prev;
        Node(Node<E> prev,E element,Node<E> next){
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out any hidden serialization magic
        s.defaultWriteObject();

        // Write out size
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (Node<E> x = first; x != null; x = x.next)
            s.writeObject(x.item);
    }
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out any hidden serialization magic
        s.defaultWriteObject();

        // Write out size
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (Node<E> x = first; x != null; x = x.next)
            s.writeObject(x.item);
    }
}
```

> transient 关键字表示不序列化

### LinkedList 类图

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj5k3qaya3j314u0pwabx.jpg)

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj5k591synj30e21gawio.jpg)



### LinkedList 练习

```java
@Slf4j
public class LinkedListPractice {

    private final static Integer INIT_CAPACITY = 5;

        public static void main(String[] args) {
        LinkedList<String> linkedList = new LinkedList<>();
        // 头插入两个
        linkedList.addFirst("element1");
        linkedList.addFirst("element2");
        // 尾部插入一个
        linkedList.addLast("element3");
        log.info("init:{}",linkedList);
        // getFirst
        log.info("getFirst:{}",linkedList.getFirst());
        // getLast
        log.info("getLast:{}",linkedList.getLast());
        // removeFirst
        log.info("removeFirst:{}",linkedList.removeFirst());
        // removeLast
        log.info("removeLast:{}",linkedList.removeLast());
        // contains
        linkedList.addLast(new String());
        log.info("contains:{}",linkedList.contains(new String()));

    }
}
```



### LinkedList思考和问题

**1. LinkedList extends AbstractSequentialList的类实现了List的接口，为什么还实现List?**

> 因为继承了 AbstractSequentialList 的类间接实现 List 的接口，为了**避免代理时出现错误**

**2. LinkedList 为什么实现 Deque 接口？**

> java 的LinkedList是双向链表，本身就有addFirst addLast getFirst getLast等功能的需求，而队列是只是特定功能的LinkedList，二者实现的方法都是一样的，所以队列只是实现了接口

##Set接口

> Set 接口等同于 Collection 接口，不过其方法的行为有更严谨的定义



### Set 类图



![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj5ev3hxygj30gc19mae3.jpg)

# 相关链接

- [源码中文备注]()