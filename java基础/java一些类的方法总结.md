---
title: java工具类的一些方法
---


#StringBuffer#
```
StringBuffer sb = new StringBuffer();

sb.append();  //在最后追加字符串

sb.setCharAt(int index,char ch);//替换字符

sb.reverse();//反转字符串

sb.deleteCharAt();删除指定位置的字符

sb.delete(int start,int end);删除2-5的字符，左闭右开
```

#双端队列#
Deque是Queue的子接口,我们知道Queue是一种队列形式,而Deque则是双向队列,它支持从两个端点方向检索和插入元素,因此Deque既可以支持LIFO形式也可以支持LIFO形式.Deque接口是一种比Stack和Vector更为丰富的抽象数据形式,因为它同时实现了以上两者.

| 修饰符和返回值|	方法名|	描述 |
| :-----| ----: | :----: |
| void | push(E) | 向队列头部插入一个元素,失败时抛出异常  |
| void | addFirst(E) | 向队列头部插入一个元素,失败时抛出异常 |
| void | addLast(E) | 向队列尾部插入一个元素,失败时抛出异常  |
| boolean | offerFirst(E) | 向队列头部加入一个元素,失败时返回false |
| boolean | offerLast(E) | 向队列尾部加入一个元素,失败时返回false |
| E | getFirst() | 获取队列头部元素,队列为空时抛出异常  |
| E | getLast() | 获取队列尾部元素,队列为空时抛出异常 |
| E | peekLast() | 获取队列头部元素,队列为空时返回null  |
| E | peekFirst() | 获取队列头部元素,队列为空时返回null |
| boolean | removeFirstOccurrence(Object) | 删除第一次出现的指定元素,不存在时返回false |
| boolean | removeLastOccurrence(Object) | 删除最后一次出现的指定元素,不存在时返回false |
| E | pop() | 弹出队列头部元素,队列为空时抛出异常  |
| E | removeFirst() | 弹出队列头部元素,队列为空时抛出异常 |
| E | removeLast() | 	弹出队列尾部元素,队列为空时抛出异常  |
| E | pollFirst() | 弹出队列头部元素,队列为空时返回null |
| E | pollLast() | 弹出队列尾部元素,队列为空时返回null  |
| Iterator<E> | descendingIterator() | 返回队列反向迭代器  |



#Hashmap迭代#
增加for循环迭代Hashmap
Map.Entry<> e : map.entrySet()
```
for (Map.Entry<String, Integer> pair: items.entrySet()) {
    System.out.format("key: %s, value: %d%n", pair.getKey(), pair.getValue());
}
```


#List,Integer[],int[]转换#
[https://www.cnblogs.com/cat520/p/10299879.html](https://www.cnblogs.com/cat520/p/10299879.html)