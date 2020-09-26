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

> ⚠️注意：
>
> next() 会移动迭代器位置
>
> - nextIndex() 获取游标前面(右边)索引，nextPrevious()获取后面(左边)，如果从左到右，再到坐可能存在索引为负
> - add()添加到**游标前面位置**

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



# RandomAccess

> 该接口不包含任何方法，用于测试特定具体集合是否支持高校的随机访问.在一些集合中，有两种有序集合，一种数组支持有序集合可以快速反问，一种链表支持有序

```java
if(c instanceof RandomAccess){
    use random access algorithm
}else{
    use sequential access algorithm
}
```



### 