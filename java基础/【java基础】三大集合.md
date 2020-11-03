---
java基础之三大集合总结
---

#三大集合#
Java中的集合，从上层接口上看分为了两类，Map和Collection。也就是说，我们平时接触到的常用的集合，包括HashMap，ArrayList和HashSet等都直接或者间接的实现了这两个接口之一。而Collection接口的子接口又包括了Set和List接口。这样我们常见的Map，Set和List三大集合接口就出来了。接口类图如下所示：

![](https://img2018.cnblogs.com/i-beta/1417592/201912/1417592-20191211111343887-888025613.png)

为了方便查询，找到了java集合框架图，如下图：
![](https://pic002.cnblogs.com/images/2012/80896/2012053020261738.gif)


以下部分通过面试题来总结知识点

## 1.Map，List和Set都是Collection的子接口吗？##
**答：**Map是和Collection并列的集合上层接口，没有继承关系；List和Set是Collection的子接口。

## 2.说说Java中常见的集合吧。##
**答：**Java中的常见集合可以概括如下。
> - Map接口和Collection接口是所有集合框架的父接口
> - Collection接口的子接口包括：Set接口和List接口
> - Map接口的实现类主要有：HashMap、TreeMap、Hashtable、LinkedHashMap、ConcurrentHashMap以及Properties等
> - Set接口的实现类主要有：HashSet、TreeSet、LinkedHashSet等
> - List接口的实现类主要有：ArrayList、LinkedList、Stack以及Vector等


> **注：hashmap的底层实现**
> HashMap的主干是一个Entry数组。Entry是HashMap的基本组成单元，每一个Entry包含一个key-value键值对。
> hashmap整体结构如下：
> ![](https://images2015.cnblogs.com/blog/1024555/201611/1024555-20161113235348670-746615111.png)
> 简单来说，HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的，如果定位到的数组位置不含链表（当前entry的next指向null）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度为O(n)，首先遍历链表，存在即覆盖，否则新增；对于查找操作来讲，仍需遍历链表，然后通过key对象的equals方法逐一比对查找。所以，性能考虑，HashMap中的链表出现越少，性能才会越好。

## 3.HashMap和Hashtable的区别有哪些？##
**答：**HashMap和Hashtable之间的区别可以总结如下。
> - HashMap没有考虑同步，是线程不安全的；Hashtable使用了synchronized关键字，是线程安全的；
> - HashMap允许null作为Key；Hashtable不允许null作为Key，Hashtable的value也不可以为null

**HashMap是线程不安全的是吧？你可以举一个例子吗？**
答：**（注意，以下是候选人常见的错误理解！！！，因为上边的答案是大家背出来的）**
有一个快速失败fast-fail机制，当对HashMap遍历的时候，调用了remove方法使其迭代器发生改变的时候会抛出一个异常ConcurrentModificationException。Hashtable因为在方法上做了synchronized处理，所以不会抛出异常。**（自信的语气^_^感觉面试官很low）。**


**先给出正确的答案**
> - HashMap线程不安全主要是考虑到了多线程环境下进行扩容可能会出现HashMap死循环
> - Hashtable线程安全是由于其内部实现在put和remove等方法上使用synchronized进行了同步，所以对单个方法的使用是线程安全的。但是对多个方法进行复合操作时，线程安全性无法保证。 比如一个线程在进行get操作，一个线程在进行remove操作，往往会导致下标越界等异常。
> 



既然说到了这里，那么我们来看看大家一直想说的Java集合快速失败（fast-fail）机制是怎么回事儿吧~  

**Java集合中的快速失败（fast-fail）机制：**
答：快速失败是Java集合的一种错误检测机制，当多个线程对集合进行结构上的改变的操作时，有可能会产生fail-fast。

例如：

假设存在两个线程（线程1、线程2），线程1通过Iterator在遍历集合A中的元素，在某个时候线程2修改了集合A的结构（是结构上面的修改，而不是简单的修改集合元素的内容），那么这个时候程序就可能会抛出 ConcurrentModificationException异常，从而产生fast-fail快速失败。

**那么快速失败机制底层是怎么实现的呢？**

迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedModCount值，是的话就返回遍历；否则抛出异常，终止遍历。JDK源码中的判断大概是这样的：

![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572060550576_4A47A0DB6E60853DEDFCFDF08A5CA249)
我们再来接着看异常ConcurrentModificationException，JDK中是这么介绍该异常的：
![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572060665759_FB5C81ED3A220004B71069645F112867)

![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572060676903_10FB15C77258A991B0028080A64FB42D)
我来解释下JDK中的英文，大概意思就是说当检测到一个并发的修改，就可能会抛出该异常，一些迭代器的实现会抛出该异常，以便可以快速失败。但是你不可以为了便捷而依赖该异常，而应该仅仅作为一个程序的侦测。

前面常见的错误答案，错误的认为快速机制就是HashMap线程不安全的表现。并且坚定的认为Hashtable和Vector等线程安全的集合不会存在并发修改时候的快速失败，这是大错特错。概念和原理理解的不清晰导致掉入了面试官的陷阱里了，大家可以打开JDK源码，会发现Hashtable也会在迭代的时候抛出该异常，可能发生快速失败。

##HashMap底层实现结构有了解吗？##

**答：**HashMap底层实现数据结构为数组+链表的形式，JDK8及其以后的版本中使用了数组+链表+红黑树实现，解决了链表太长导致的查询速度变慢的问题。大概结构如下图所示：
![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572060763722_09DD8C2662B96CE14928333F055C5580)

**1.HashMap的初始容量，加载因子，扩容增量是多少？**

答：HashMap的初始容量16，加载因子为0.75，扩容增量是原容量的1倍。如果HashMap的容量为16，一次扩容后容量为32。HashMap扩容是指元素个数（包括数组和链表+红黑树中）超过了16*0.75=12之后开始扩容。

**2.HashMap的长度为什么是2的幂次方？**

**答：**

- 我们将一个键值对插入HashMap中，通过将Key的hash值与length-1进行&运算，实现了当前Key的定位，2的幂次方可以减少冲突（碰撞）的次数，提高HashMap查询效率

- 如果length为2的幂次方，则length-1 转化为二进制必定是11111……的形式，在与h的二进制与操作效率会非常的快，而且空间不浪费
- 如果length不是2的幂次方，比如length为15，则length-1为14，对应的二进制为1110，在与h与操作，最后一位都为0，而0001，0011，0101，1001，1011，0111，1101这几个位置永远都不能存放元素了，空间浪费相当大，更糟的是这种情况中，数组可以使用的位置比数组长度小了很多，这意味着进一步增加了碰撞的几率，减慢了查询的效率！这样就会造成空间的浪费。


**总的来说，2的N次幂有助于减少碰撞的几率，空间利用率比较大。至于加载因子，如果设置太小不利于空间利用，设置太大则会导致碰撞增多，降低了查询效率，所以设置了0.75。**

**3.HasMap的存储和获取原理：**
当调用put()方法传递键和值来存储时，先对键调用hashCode()方法，返回的hashCode用于找到bucket位置来储存Entry对象，也就是找到了该元素应该被存储的桶中（数组）。当两个键的hashCode值相同时，bucket位置发生了冲突，也就是发生了Hash冲突，这个时候，会在每一个bucket后边接上一个链表（JDK8及以后的版本中还会加上红黑树）来解决，将新存储的键值对放在表头（也就是bucket中）。

当调用get方法获取存储的值时，首先根据键的hashCode找到对应的bucket，然后根据equals方法来在链表和红黑树中找到对应的值。

**4.HasMap的扩容步骤：**
HashMap里面默认的负载因子大小为0.75，也就是说，当Map中的元素个数（包括数组，链表和红黑树中）超过了16*0.75=12之后开始扩容。将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。

但是，需要注意的是在多线程环境下，HashMap扩容可能会导致死循环。


前面我们介绍了在HashMap存储的时候，会发生Hash冲突，那么我们一起来看Hash冲突的解决办法吧。

**5.解决Hash冲突的方法有哪些？**
- 拉链法 （HashMap使用的方法） //在每个数组后边加上链表
- 开放地址法
	- 线性探测再散列法    //如果当前数组有值，则往后搜索直到找到空的
	- 二次探测再散列法    //二次探测用的算法是 i + 1^2，i-1^2的方法搜索 
	- 伪随机探测再散列法  //建立一个随机数发生器，生成随机序列，给定一个随机数做起点，每次加上这个伪随机数++就可以了。
- 再散列法 //如果冲突就再散列

***拉链法和开发定址法优缺点：***
**拉链法：**
1）优点： ①对于记录总数频繁可变的情况，处理的比较好（也就是避免了动态调整的开销） ②由于记录存储在结点中，而结点是动态分配，不会造成内存的浪费，所以尤其适合那种记录本身尺寸（size）很大的情况，因为此时指针的开销可以忽略不计了 ③删除记录时，比较方便，直接通过指针操作即可
 
2）缺点： ①存储的记录是随机分布在内存中的，这样在查询记录时，相比结构紧凑的数据类型（比如数组），哈希表的跳转访问会带来额外的时间开销 ②如果所有的 key-value 对是可以提前预知，并之后不会发生变化时（即不允许插入和删除），可以人为创建一个不会产生冲突的完美哈希函数（perfect hash function），此时封闭散列的性能将远高于开放散列 ③由于使用指针，记录不容易进行序列化（serialize）操作


**开发定址法：**
1）优点： ①记录更容易进行序列化（serialize）操作 ②如果记录总数可以预知，可以创建完美哈希函数，此时处理数据的效率是非常高的
 
2）缺点： ①存储记录的数目不能超过桶数组的长度，如果超过就需要扩容，而扩容会导致某次操作的时间成本飙升，这在实时或者交互式应用中可能会是一个严重的缺陷 ②使用探测序列，有可能其计算的时间成本过高，导致哈希表的处理性能降低 ③由于记录是存放在桶数组中的，而桶数组必然存在空槽，所以当记录本身尺寸（size）很大并且记录总数规模很大时，空槽占用的空间会导致明显的内存浪费 ④删除记录时，比较麻烦。比如需要删除记录a，记录b是在a之后插入桶数组的，但是和记录a有冲突，是通过探测序列再次跳转找到的地址，所以如果直接删除a，a的位置变为空槽，而空槽是查询记录失败的终止条件，这样会导致记录b在a的位置重新插入数据前不可见，所以不能直接删除a，而是设置删除标记。这就需要额外的空间和操作。


**6.哪些类适合做Hashmap的键**
**答：**
String和Interger这样的包装类很适合做为HashMap的键，因为他们是final类型的类，而且重写了equals和hashCode方法，避免了键值对改写，有效提高HashMap性能。
为了计算hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的hashCode的话，那么就不能从HashMap中找到你想要的对象。

##ConcurrentHashMap和Hashtable的区别？##
**答：** ConcurrentHashMap结合了HashMap和Hashtable二者的优势。HashMap没有考虑同步，Hashtable考虑了同步的问题。但是Hashtable在每次同步执行时都要锁住整个结构。

ConcurrentHashMap锁的方式是稍微细粒度的，ConcurrentHashMap将hash表分为16个桶（默认值），诸如get，put，remove等常用操作只锁上当前需要用到的桶。

**ConcurrentHashMap的具体实现方式(分段锁)：**
- 该类包含两个静态内部类MapEntry和Segment，前者用来封装映射表的键值对，后者用来充当锁的角色。
![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572061203649_8266E4BFEDA1BD42D8F9794EB4EA0A13)
- Segment是一种可重入的锁ReentrantLock，每个Segment守护一个HashEntry数组里得元素，当对HashEntry数组的数据进行修改时，必须首先获得对应的Segment锁。
![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572061252942_F19C9085129709EE14D013BE869DF69B)

	> 注：ConcurrentHashMap与Hashtable以及HashMap的比较是一个绝对高频的考察点，我们必须熟练掌握ConcurrentHashMap分段锁的实现方式。在实际的开发中，我们在单线程环境下可以使用HashMap，多线程环境下可以使用ConcurrentHashMap，至于Hashtable已经不被推荐使用了（也就是说Hashtable只存在于面试题目中了）


##TreeMap有哪些特性？##
**答：**TreeMap底层使用红黑树实现，TreeMap中存储的键值对按照键来排序。
- 如果Key存入的是字符串等类型，那么会按照字典默认顺序排序
- 如果传入的是自定义引用类型，比如说User，那么该对象必须实现Comparable接口，并且覆盖其compareTo方法；或者在创建TreeMap的时候，我们必须指定使用的比较器。如下所示：

```
	// 方式一：定义该类的时候，就指定比较规则
	class User implements Comparable{
	    @Override
	    public int compareTo(Object o) {
	        // 在这里边定义其比较规则
	        return 0;
	    }
	}
	public static void main(String[] args) {
	    // 方式二：创建TreeMap的时候，可以指定比较规则
	    new TreeMap<User, Integer>(new Comparator<User>() {
	        @Override
	        public int compare(User o1, User o2) {
	            // 在这里边定义其比较规则
	            return 0;
	        }
	    });
	}
```   
**解析：**
关于TreeMap的考察，会涉及到两个接口Comparable和Comparator的比较。Comparable接口的后缀是able大概表示可以的意思，也就是说一个类如果实现了这个接口，那么这个类就是可以比较的。类似的还有cloneable接口表示可以克隆的。而Comparator则是一个比较器，是创建TreeMap的时候传入，用来指定比较规则。

**1.那么Comparable接口和Comparator接口有哪些区别呢？**
- Comparable实现比较简单，但是当需要重新定义比较规则的时候，必须修改源代码，即修改User类里边的compareTo方法
- Comparator接口不需要修改源代码，只需要在创建TreeMap的时候重新传入一个具有指定规则的比较器即可。

**2..红黑树的特性**
（1）每个节点或者是黑色，或者是红色。
（2）根节点是黑色。
（3）每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！]
（4）如果一个节点是红色的，则它的子节点必须是黑色的。
（5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

红黑树的时间复杂度为: O(lgn)



##ArrayList和LinkedList有哪些区别？##

答：常用的ArrayList和LinkedList的区别总结如下。

- ArrayList底层使用了动态数组实现，实质上是一个动态数组
- LinkedList底层使用了双向链表实现，可当作堆栈、队列、双端队列使用
- ArrayList在随机访问方面（get，set）效率高于LinkedList
- LinkedList在节点的增删方面效率高于ArrayList
- ArrayList必须预留一定的空间，当空间不足的时候，会进行扩容操作
- LinkedList的开销是必须存储节点的信息以及节点的指针信息
 
解析：
该题是集合中最常见和最基础的题目之一，List集合也是我们平时使用很多的集合。List接口的常见实现就算ArrayList和LinkedList，我们必须熟练掌握其底层实现以及一些特性。其实还有一个集合Vector，它是线程安全的ArrayList，但是已经被废弃，不推荐使用了。**多线程环境下，我们可以使用CopyOnWriteArrayList替代ArrayList来保证线程安全。**


##HashSet和TreeSet有哪些区别？##
答： HashSet和TreeSet的区别总结如下。

- HashSet底层使用了Hash表实现。
 保证元素唯一性的原理：判断元素的hashCode值是否相同。如果相同，还会继续判断元素的equals方法，是否为true
- TreeSet底层使用了红黑树来实现。
 保证元素唯一性是通过Comparable或者Comparator接口实现
- TreeSet 是二差树实现的,Treeset中的数据是自动排好序的，不允许放入null值。 
- HashSet 是哈希表实现的,HashSet中的数据是无序的，可以放入null，但只能放入一个null，两者中的值都不能重复，就如数据库中唯一约束。 

解析：

其实，HashSet的底层实现还是HashMap，只不过其只使用了其中的Key，具体如下所示：

HashSet的add方法底层使用HashMap的put方法将key = e，value=PRESENT构建成key-value键值对，当此e存在于HashMap的key中，则value将会覆盖原有value，但是key保持不变，所以如果将一个已经存在的e元素添加中HashSet中，新添加的元素是不会保存到HashMap中，所以这就满足了HashSet中元素不会重复的特性。
HashSet的contains方法使用HashMap得containsKey方法实现

##LinkedHashMap和LinkedHashSet有了解吗？##
答：LinkedHashMap在面试题中还是比较常见的。LinkedHashMap可以记录下元素的插入顺序和访问顺序，具体实现如下：

- LinkedHashMap内部的Entry继承于HashMap.Node，这两个类都实现了Map.Entry<K,V>
- LinkedHashMap的Entry不光有value，next，还有before和after属性，这样通过一个双向链表，保证了各个元素的插入顺序
- 通过构造方法public LinkedHashMap(int initialCapacity,float loadFactor,boolean accessOrder)， accessOrder传入true可以实现LRU缓存算法（访问顺序）
- LinkedHashSet 底层使用LinkedHashMap实现，两者的关系类似与HashMap和HashSet的关系，大家可以自行类比。

**扩展： 什么是LRU算法？LinkedHashMap如何实现LRU算法？**
LRU（Least recently used，最近最少使用）算法根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”。

由于LinkedHashMap可以记录下Map中元素的访问顺序，所以可以轻易的实现LRU算法。只需要将构造方法的accessOrder传入true，并且重写removeEldestEntry方法即可。具体实现参考如下：
```
package pak2;

import java.util.LinkedHashMap;
import java.util.Map;

public class LRUTest {

    private static int size = 5;

    public static void main(String[] args) {
        Map<String, String> map = new LinkedHashMap<String, String>(size, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<String, String> eldest) {
                return size() > size;
            }
        };
        map.put("1", "1");
        map.put("2", "2");
        map.put("3", "3");
        map.put("4", "4");
        map.put("5", "5");
        System.out.println(map.toString());

        map.put("6", "6");
        System.out.println(map.toString());
        map.get("3");
        System.out.println(map.toString());
        map.put("7", "7");
        System.out.println(map.toString());
        map.get("5");
        System.out.println(map.toString());
    }
}
```

##List和Set的区别？##
答： List和Set的区别可以简单总结如下。

- List是有序的并且元素是可以重复的,Set是无序（LinkedHashSet除外）的，并且元素是不可以重复的.(此处的有序和无序是指放入顺序和取出顺序是否保持一致）
- list可以插入多个null元素，而set只允许插入一个null元素；
- Set：检索元素效率低下，删除和插入效率高，插入和删除不会引起元素位置改变。 
List：和数组类似，List可以动态增长，查找元素效率高，插入删除元素效率低，因为会引起其他元素位置改变。 


##Iterator和ListIterator的区别是什么？##
答： 常见的两种迭代器的区别如下。

- Iterator可以遍历list和set集合；ListIterator只能用来遍历list集合
- Iterator前者只能前向遍历集合；ListIterator可以前向和后向遍历集合
- ListIterator其实就是实现了前者，并且增加了一些新的功能。

解析：

Iterator其实就是一个迭代器，在遍历集合的时候需要使用。Demo实现如下：
```
        ArrayList<String> list =  new ArrayList<>();
        list.add("zhangsan");
        list.add("lisi");
        list.add("yangwenqiang");
        // 创建迭代器实现遍历集合
        Iterator<String> iterator = list.iterator();
        while(iterator.hasNext()){
            System.out.println(iterator.next());
        }
```

##数组和集合List之间的转换：##
答：数组和集合Lis的转换在我们的日常开发中是很常见的一种操作，主要通过Arrays.asList以及List.toArray方法来搞定。这里给出Demo演示：
```
package niuke;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class ConverTest {
    public static void main(String[] args) {
        // list集合转换成数组
        ArrayList<String> list =  new ArrayList<>();
        list.add("zhangsan");
        list.add("lisi");
        list.add("yangwenqiang");
        Object[] arr = list.toArray();
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
        System.out.println("---------------");
        // 数组转换为list集合
        String[] arr2 = {"niuke", "alibaba"};
        List<String> asList = Arrays.asList(arr2);
        for (int i = 0; i < asList.size(); i++) {
            System.out.println(asList.get(i));
        }

    }
}
```
输出结果如下：
![](https://uploadfiles.nowcoder.com/images/20191110/5459305_1573361088692_32D3CA5E23F4CCF1E4C8660C40E75F33)

解析：

关于数组和集合之间的转换是一个常用操作，这里主要讲解几个需要注意的地方吧。


**数组转为集合List：**
通过Arrays.asList方法搞定，转换之后不可以使用add/remove等修改集合的相关方法，因为该方法返回的其实是一个Arrays的内部私有的一个类ArrayList，该类继承于Abstractlist，并没有实现这些操作方法，调用将会直接抛出UnsupportOperationException异常。这种转换体现的是一种适配器模式，只是转换接口，本质上还是一个数组。


**集合转换数组：**
List.toArray方法搞定了集合转换成数组，这里最好传入一个类型一样的数组，大小就是list.size()。因为如果入参分配的数组空间不够大时，toArray方法内部将重新分配内存空间，并返回新数组地址；如果数组元素个数大于实际所需，下标为list.size()及其之后的数组元素将被置为null，其它数组元素保持原值。所以，建议该方法入参数组的大小与集合元素个数保持一致。

若是直接使用toArray无参方法，此方法返回值只能是Object[ ]类，若强转其它类型数组将出现ClassCastException错误。

##Collection和Collections有什么关系？##
答：
- java.util.Collection 是一个集合接口。它提供了对集合对象进行基本操作的通用接口方法。Collection接口在Java 类库中有很多具体的实现。Collection接口的意义是为各种具体的集合提供了最大化的统一操作方式。
List，Set，Queue接口都继承Collection。
直接实现该接口的类只有AbstractCollection类，该类也只是一个抽象类，提供了对集合类操作的一些基本实现。List和Set的具体实现类基本上都直接或间接的继承了该类。


- java.util.Collections 是一个包装类。它包含有各种有关集合操作的静态方法（对集合的搜索、排序、线程安全化等），大多数方法都是用来处理线性表的。此类不能实例化，就像一个工具类，服务于Java的Collection框架。




#补充知识点#
##为什么重写equals方法需同时重写hashCode方法##
答：HashMap中，如果要比较key是否相等，要同时使用这两个函数！因为自定义的类的hashcode()方法继承于Object类，其hashcode码为默认的内存地址，这样即便有相同含义的两个对象，比较也是不相等的。HashMap中的比较key是这样的，先求出key的hashcode(),比较其值是否相等，若相等再比较equals(),若相等则认为他们是相等的。若equals()不相等则认为他们不相等。如果只重写hashcode()不重写equals()方法，当比较equals()时只是看他们是否为同一对象（即进行内存地址的比较）,所以必定要两个方法一起重写。HashMap用来判断key是否相等的方法，其实是调用了HashSet判断加入元素 是否相等。重载hashCode()是为了对同一个key，能得到相同的Hash Code，这样HashMap就可以定位到我们指定的key上。重载equals()是为了向HashMap表明当前对象和key上所保存的对象是相等的，这样我们才真正
地获得了这个key所对应的这个键值对。






附图：
- HashMap的类图结构：
	![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572062458300_FB5C81ED3A220004B71069645F112867)
- ConcurrentHashMap的类图结构：
	![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572062471995_10FB15C77258A991B0028080A64FB42D)
- Hashtable的类图结构：
	![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572062499403_8266E4BFEDA1BD42D8F9794EB4EA0A13)
- TreeMap的类图结构：
	![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572062487909_09DD8C2662B96CE14928333F055C5580)
- LinkedHashMap的类图结构：
	![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572062511852_F19C9085129709EE14D013BE869DF69B)
- ArrayList的类图结构:
	![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572062525218_9EB9CD58B9EA5E04C890326B5C1F471F)
- LinkedList的类图结构：
	![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572062540745_602E8F042F463DC47EBFDF6A94ED5A6D)
- HashSet的类图结构：
	![](https://uploadfiles.nowcoder.com/images/20191106/5459305_1573054851175_10FB15C77258A991B0028080A64FB42D)
- TreeSet的类图结构：
	![](https://uploadfiles.nowcoder.com/images/20191026/5459305_1572062636452_EC41A1351C6E33E039A810BC03E4E161)