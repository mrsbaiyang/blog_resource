---
title: HashMap的小认识  
date: 2019-9-23 18:21:36  
tags: java map  
categories: 技术

---
想充分了解HashMap很久了，这个我们时刻使用，但是对于我来说总感觉有着迷雾的数据结构。  

##HashMap
下面是居于JDK7的，JDK8的红黑树结构暂时放弃。  
### 数据结构
HashMap继承了AbstractMap、实现了Map

    public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
    {
    /**
     * The table, resized as necessary. Length MUST Always be a power of two.
     */
    transient Entry<K,V>[] table;
重要的是数组table以及它的内部类Entry。  

    static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;
可以看到Entry里有一个next属性，所以它本身就是个单链表，以下就是我们常说的数组+链表的结构：  

![](https://upload-images.jianshu.io/upload_images/4843132-05b3a55bd2686dd3.png)

<!-- more -->

### put方法
代码还是很简单的，稍微一看都能清楚：  

	 public V put(K key, V value) {
	        // 对key为null的处理
	        if (key == null)
	            return putForNullKey(value);
	        // 根据key算出hash值
	        int hash = hash(key);
	        // 根据hash值和HashMap容量算出在table中应该存储的下标i
	        int i = indexFor(hash, table.length);
	        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
	            Object k;
	            // 先判断hash值是否一样，如果一样，再判断key是否一样
	            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
	                V oldValue = e.value;
	                e.value = value;
	                e.recordAccess(this);
	                return oldValue;
	            }
	        }
	
	        modCount++;
	        addEntry(hash, key, value, i);
	        return null;
	    }
整理出来的流程图如下：  
![](https://upload-images.jianshu.io/upload_images/4843132-3d5e78f87bb64216.png)  
值得注意点的地方是：   

    static int indexFor(int h, int length) {
        return h & (length-1);
    }

可以看到是用&的位运算，&运算是CPU的运算方式，所以效率是杠杠的，至于为什么是跟length-1进行&的位运呢？  
因为length为2的幂次方，即一定是偶数，偶数减1，即是奇数，这样保证了（length-1）在二进制中最低位是1，而&运算结果的最低位是1还是0完全取决于hash值二进制的最低位。如果length为奇数，则length-1则为偶数，则length-1二进制的最低位横为0，则&位运算的结果最低位横为0，即横为偶数。这样table数组就只可能在偶数下标的位置存储了数据，浪费了所有奇数下标的位置，这样也更容易产生hash冲突。**这也是HashMap的容量为什么总是2的平方数的原因。**  
![](https://upload-images.jianshu.io/upload_images/4843132-91b165309b5d447c.png)

### resize、get
这个的流程和put类似，都是先计算hash值以及对应的index下表，然后重新赋值或者获取元素，这里就不在啰嗦。

### iterate
这里我了解下，为什么map的遍历不是有序的，以及它是如何做的。  
核心代码如下：  

        final Entry<K,V> nextEntry() {
            // 保存下一个需要返回的Entry，作为返回结果
            Entry<K,V> e = next;
            // 如果遍历到table上单向链表的最后一个元素时
            if ((next = e.next) == null) {
                Entry[] t = table;
                // 继续往下寻找table上有元素的下标
                // 并且把下一个talbe上有单向链表的表头，作为下一个返回的Entry next
                while (index < t.length && (next = t[index++]) == null)
                    ;
            }
            current = e;
            return e;
        }
nextEntry的主要作用有两点

1. 把当前遍历到的Entry返回
2. 准备好下一个需要返回的Entry  

如果当前返回的Entry不是单向链表的最后一个元素，那只要让下一个返回的Entrynext为当前Entry的next属性（下图红色过程）；如果当前返回的Entry是单向链表的最后一个元素，那么它就没有next属性了，所以要寻找下一个table上有单向链表的表头（下图绿色过程）  
![](https://upload-images.jianshu.io/upload_images/4843132-3830a9227f7df10f.png)  

##LinkedHashMap
LinkedHashMap是在HashMap的基础上又维护了Entry之间的一个双向链表，其结构如下图所示：  
![](https://upload-images.jianshu.io/upload_images/4843132-7abca1abd714341d.png)  

    private static class Entry<K,V> extends HashMap.Entry<K,V> {
        // These fields comprise the doubly linked list used for iteration.
        Entry<K,V> before, after;

        Entry(int hash, K key, V value, HashMap.Entry<K,V> next) {
            super(hash, key, value, next);
        }
可以到相比HashMap.Entry，LinkedHashMap.Entry多了before和after，这就是用来维护双向链表的。

### put方法
新加入的元素维护到双向链表的末尾，代码如下：  

        private void addBefore(Entry<K,V> existingEntry) {
            after  = existingEntry;
            before = existingEntry.before;
            before.after = this;
            after.before = this;
        }

### iterate
从双向链表表头的第一个元素开始，遍历效率是大于HashMap的。

### 重排序
LinkedHashMap有一个比较特殊的构造函数：  

    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
accessOrder默认为false，标识元素顺序与插入顺序相关；如果为true，标识元素顺序与访问顺序相关。能做到这点，就是因为在accessOrder=true的情况下，每一次访问都会将元素重新排序：  

        void recordAccess(HashMap<K,V> m) {
            LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
            // 如果LinkedHashMap的accessOrder为true，则进行重排序
            // 比如前面提到LruCache中使用到的LinkedHashMap的accessOrder属性就为true
            if (lm.accessOrder) {
                lm.modCount++;
                // 把更新的Entry从双向链表中移除
                remove();
                // 再把更新的Entry加入到双向链表的表尾
                addBefore(lm.header);
            }
        }
其实到这里我就比较好奇了，根据插入顺序的HashMap还有一些场景，但是每次读顺序都会改变的HashMap有哪些使用场景呢？

<!-- more -->

## LinkedHashMap与LRU
每次访问的元素都会被放置在队列末尾，这和LRU（Least Recently Used最近最少算法）算法的特性是一致的。所以我们利用这个特性，能很轻易实现LRU-1算法，事实上java本身也给我们留好了实现的接口：  

	protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }
我们只要覆盖这个方法就是一个线程不安全的LRU了。  
LRU本身是内存淘汰算法，关于这个请听下回分解。