## ThreadLocal介绍
> ThreadLocal 很多地方叫线程本地变量，也有些地方叫线程本地存储。

#### 正确理解ThreadLocal
ThreadLocal并不解决多线程共享变量的问题。既然变量不共享，那就谈不上同步的问题。
> ThreadLocal的基本原理是：同一个ThreadLocal所包含的变量(对ThreadLocal<String>而言即String类型变量)，在不同的Thread中有不同的副本。ThreadLocal适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用，也即变量在线程间隔离而在方法或类间共享的场景。

综上所述，ThreadLocal适用于如下两种场景：
- 每个线程需要有自己独立的实例
- 实例需要在多个方法中共享，但不希望被多线程共享

#### 剖析ThreadLocal
**核心的几个方法**
```java
//用来获取ThreadLocal在当前线程中保存的变量副本
public T get() { }   
//用来设置当前线程中变量的副本
public void set(T value) { }   
//用来移除当前线程中变量的副本
public void remove() { }   
//是一个protected方法，一般是用来在使用时进行重写的，它是一个延迟加载方法
protected T initialValue() { }  
```
**get方法的实现**
```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this); //注意这里获取键值传入的是this，不是当前线程t
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue(); //如果map为空，则调用setInitialValue方法返回value
}
```
getMap方法做了什么呢？
> 返回当前线程t中的一个成员变量threadLocals，threadLocals实际就是一个ThreadLocalMap，ThreadLocalMap是ThreadLocal类中的一个内部类，做存储结构使用。
```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

static class ThreadLocalMap {
	static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
}
```
Entry类继承了WeakReference<ThreadLocal<?>>，即每个Entry对象都有一个ThreadLocal的弱引用（作为key），这是为了防止内存泄露。一旦线程结束，key变为一个不可达的对象，这个Entry就可以被GC了。

&emsp;**使用弱引用的原因**在于：当没有强引用指向ThreadLocal变量时，它可被回收，从而避免ThreadLocal不能被回收而造成的内存泄露问题。

&emsp;但是，这里有可能出现另外一种内存泄露的问题。ThreadLocalMap维护ThreadLocal变量与具体实例映射，当ThreadLocal变量被回收后，该映射的key变为null，该Entry无法被移除。从而使得实例被该Entry引用而无法被回收造成内存泄露。

**setInitialValue方法实现**
```java
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```
如果map不为空，就设置键值对，为空，则创建一个ThreadLocalMap，如下是createMap方法的实现
```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```
**set方法实现**
set方法的实现和setInitialValue方法如出一辙，区别在于value是通过方法传值进来
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```
每个Thread对象内部维护了一个ThreadLocalMap这样一个ThreadLocal的Map，可以存放若干个ThreadLocal
```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```
ThreadLocal是如何为每个线程创建变量副本的：
- 首先，在每个线程Thread的内部有一个ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的。键值为当前ThreadLocal变量，value为变量副本(即T类型的变量)。

- 初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存储到threadLocals。然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

#### 总结一下
1. 实际是通过ThreadLocal创建的副本存储在每个线程Thread自己的threadLocals成员变量中的；
2. 为何threadLocals的类型ThreadLocalMap的键值为ThreadLocal对象？因为每个线程中可有多个threadLocal变量。将ThreadLocal和线程巧妙的绑定在一起，既可以保证无用的ThreadLocal被及时回收，不会造成内存泄露，又可以提升性能。
3. 在进行get之前，必须先set，否则会报空指针异常；如果想在get之前不需要调用set就能正常访问，必须重写initialValue方法。
4. 因为threadLocal底层的ThreadLocalMap的key存的是this(ThreadLocal本身)，并且每个Thread线程存在一个threadLocalMap变量，所以同一个Thread可以存放多个值，颠覆的是大家认为的key存储的是thread，这样一个thread只能存放一个数据了。

#### 接下来说一下ThreadLocalMap的实现
**先看一下ThreadLocalMap的构造方法**
```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
```
构造函数的第一个参数就是ThreadLocal实例(this)，第二个参数就是要保存的线程本地变量。构造函数首先创建一个长度为16的Entry数组，然后计算出firstKey对象的hash值，然后存储到table中，并设置size和threshold。
&emsp;**注意**：在计算hash时，采用了hashCode&(INITIAL_CAPACITY-1)的算法，这相当于取模运算的高效实现，与HashMap中的思路一样，正是因为这种算法，要求数组的长度必须是2的指数，这样可以使得hash发生冲突的次数减小。

**在看一下ThreadLocalMap中的set方法**
```java
 private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```
看下nextIndex方法，ThreadLocalMap是如何计算hash的
```java
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}
```
我们看到ThreadLocalMap解决冲突的方法是**线性探测法（不断加1）**，而不是HashMap的链地址法，这一点也能从Entry的结构上推断出来。

#### 使用ThreadlLocal出现的问题
**ThreadlLocal为什么会出现内存泄露**

&emsp; ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用来引用它，那么系统GC的时候，这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value永远无法回收，造成内存泄漏。

&emsp; 其实，ThreadLocalMap的设计中已经考虑到这种情况，也加上了一些防护措施：在ThreadLocal的get(),set(),remove()的时候都会清除线程ThreadLocalMap里所有key为null的value。但是这些被动的预防措施并不能保证不会内存泄漏：使用static的ThreadLocal，延长了ThreadLocal的生命周期，可能导致的内存泄漏；分配使用了ThreadLocal又不再调用get(),set(),remove()方法，那么就会导致内存泄漏。

**ThreadLocal内存泄漏怎么避免呢？**
- 每次使用完ThreadLocal，都调用它的remove()方法，清除数据。
- 在使用线程池的情况下，没有及时清理ThreadLocal，不仅是内存泄漏的问题，更严重的是可能导致业务逻辑出现问题。所以，使用ThreadLocal就跟加锁完要解锁一样，用完就清理。


## 转载至
[Java-JUC-ThreadLocal深度剖析](http://ykblog.top/posts/Java/%E3%80%90JUC%E3%80%91ThreadLocal%E5%89%96%E6%9E%90/)