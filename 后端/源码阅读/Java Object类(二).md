
```java
public final native void notify();
```
****

```java
public final native void notifyAll();
```
****

```java
public final native void wait(long timeout) throws InterruptedException;
```
****
```java
public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }
```
****
Causes the current thread to wait until another thread invokes the notify() method or the notifyAll() method for this object.

```java
public final void wait() throws InterruptedException {
        wait(0);
    }
```
> 1. wait() 出现在 synchronized 函数
2. this method should always be used in a loop

```java
synchronized (obj) {
             while (condition)
                  obj.wait();
              ... // Perform action appropriate to condition
         }
```

> 一定要在while 循环中使用 wait()和 notify() 的原因是,如果在if
****
```java
 protected void finalize() throws Throwable { }
```
