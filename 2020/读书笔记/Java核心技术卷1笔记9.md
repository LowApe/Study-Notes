# 集合

设计的内容

- Java 集合框架
- 视图与包装器
- 具体的集合
- 算法
- 映射
- 遗留的集合

# Java集合框架



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

### Collection接口

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

#### Collection 代码分析

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

#### Collection 类图



#### Collection 练习

### 迭代器

