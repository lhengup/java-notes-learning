## Collection下的List

### ArrayList和Vector区别

- `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[ ]`存储，适用于频繁的查找工作，线程不安全 ；

- `Vector` 是 `List` 的古老实现类，底层使用` Object[ ]` 存储，线程安全的。

  

### Arraylist 与 LinkedList 区别?

1. **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；
2. **底层数据结构：** `Arraylist` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。注意双向链表和双向循环链表的区别，下面有介绍到！）
3. **插入和删除是否受元素位置的影响：** ① **`ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。** 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。 ② **`LinkedList` 采用链表存储，所以对于`add(E e)`方法的插入，删除元素时间复杂度不受元素位置的影响，近似 O(1)，如果是要在指定位置`i`插入和删除元素的话（`(add(int index, E element)`） 时间复杂度近似为`o(n))`因为需要先移动到指定位置再插入。**
4. **是否支持快速随机访问：** `LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。
5. **内存空间占用：** ArrayList 的空 间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。



### ArrayList的扩容机制

##### 扩容机制总结

**扩容可分为两种情况：**

　　<font color = red>**第一种情况，当ArrayList的容量为0时，此时添加元素的话，需要扩容，三种构造方法创建的ArrayList在扩容时略有不同：**</font>

　　　　<font color=blue> 1.无参构造，创建ArrayList后容量为0，添加第一个元素后，容量变为10，此后若需要扩容，则正常扩容。</font>

　　　　<font color=blue>2.传容量构造，当参数为0时，创建ArrayList后容量为0，添加第一个元素后，容量为1，此时ArrayList是满的，下次添加元素时需正常扩容。</font>

　　　　<font color=blue>3.传列表构造，当列表为空时，创建ArrayList后容量为0，添加第一个元素后，容量为1，此时ArrayList是满的，下次添加元素时需正常扩容。</font>

　　<font color = red>**第二种情况，当ArrayList的容量大于0，并且ArrayList是满的时，此时添加元素的话，进行正常扩容，每次扩容到原来的1.5倍。**</font>

#### add方法

```java
public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}
```

这是最基本的add方法，当然，也是可以说明问题的。可以看到，此add方法的参数就是一个被加元素，moCount是记录ArrayList被修改次数的，可以不用管。然后是另一个add方法，所传的值是被加元素、当前数组和当前数组的元素个数，让我们来看看这个add方法吧。

```java
private void add(E e, Object[] elementData, int s) {
    // 判断元素个数是否等于当前容量
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}
```

首先，它判断了元素个数是否等于当前数组的容量，也就是判断当前数组是不是满的，如果当前空间是满的，就需要扩容了，grow函数就是扩容函数了，扩容后再将被加元素加到数组中。

　　下面我们来看看grow函数是什么样子的：

```java
private Object[] grow() {
    return grow(size + 1);
}
```

它里面又调用了一个带参的grow函数，参数是当前元素个数+1，也就是当前容量+1。返回的是这个函数的返回值，让我们进一步研究这个带参的函数。

```java
private Object[] grow(int minCapacity) {
    // 获取老容量，也就是当前容量
    int oldCapacity = elementData.length;
    // 如果当前容量大于0 或者 数组不是DEFAULTCAPACITY_EMPTY_ELEMENTDATA
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity, /* minimum growth */
                oldCapacity >> 1           /* preferred growth */);
        return elementData = Arrays.copyOf(elementData, newCapacity);
    // 如果 数组是DEFAULTCAPACITY_EMPTY_ELEMENTDATA（容量等于0的话，只剩这一种情况了）
    } else {
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}
```

　首先，它是记录了一下老容量的大小，然后再进行下面的操作。

　　如果当前容量大于0，或者当前数组不是DEFAULTCAPACITY_EMPTY_ELEMENTDATA，前面说明过，DEFAULTCAPACITY_EMPTY_ELEMENTDATA是一个被final修饰的空数组，在三个构造方法中，只有无参构造方法中elementData被赋予了DEFAULTCAPACITY_EMPTY_ELEMENTDATA。也就是说，这个if语句中不会处理用默认无参构造方法创建的数组的初始扩容情况，那么其余的扩容情况都是由此if语句来处理的。

　　我们来看一下if里面的操作，先创建一个新的数组，然后将旧数组拷贝到新数组并赋给elementData返回。ArraysSupport.newLength函数的作用是创建一个大小为oldCapacity + max（minimum growth， preferred growth）的数组。

　　minCapacity是传入的参数，我们上面看过，它的值是当前容量（老容量）+1，那么minCapacity - oldCapacity的值就恒为1，minimum growth的值也就恒为1。

　　oldCapacity >> 1的功能是将oldCapacity 进行位运算，右移一位，也就是减半，preferred growth的值即为oldCapacity大小的一半。





**扩容实质：**

当oldCapacity为0时，右移后还是0，也就是说此时扩容的大小为0+max（1,0）=1，容量从0扩展到1。那么什么时候是这种情况呢？

　　　　（1）传容量的构造方法传入的是0时，elementData被赋予的是EMPTY_ELEMENTDATA，此时数组容量为0，添加元素时，符合if的条件，会进入此扩容情况，容量从0扩展到1。

　　　　（2）传Collection元素列表的构造方法被传入空列表时，elementData被赋予的是EMPTY_ELEMENTDATA，数组容量为0，此时添加元素时，符合if的条件，会进入此扩容情况，容量从0扩展到1。

　　当oldCapacity大于0时，新创建的数组大小是老容量+老容量的一半，也就是老容量的1.5倍，每次扩容到原来的1.5倍。

　　else就剩一种情况了，也就是用默认无参构造方法创建的数组的初始扩容情况。此时的容量为0，添加一个元素时会创建一个新的数组，其大小为max(DEFAULT_CAPACITY, minCapacity)。

　　我们从上面的源码变量信息中可得知DEFAULT_CAPACITY是一个常量，其值为10，而minCapacity的值为（0+1），所以添加一个元素时，max(DEFAULT_CAPACITY, minCapacity)的值必为10。也就是说，当我们用默认无参构造方法创建的数组在添加元素前，ArrayList的容量为0，添加一个元素后，ArrayList的容量就变为10了。







