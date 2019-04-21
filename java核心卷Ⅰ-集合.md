---
title: java核心卷Ⅰ-集合
date: 2019-03-10 15:03:13
tags:
  - java
categories:
  - 读书笔记
  - java核心卷1
coments: true
---



本章节主要讲java核心卷1的集合框架

这篇很水，请看链接<a href="<http://www.cnblogs.com/ysocean/p/6555373.html>">一只渣渣威</a>

<!--more-->



## java集合框架



### 集合框架中的接口

![集合框架中的接口](https://i.loli.net/2019/03/10/5c8501eb4bb1b.png)
***集合两个基本接口Collection Map***

![集合框架中的类](https://i.loli.net/2019/03/11/5c8628b7430f5.jpg)

![java库中的具体集合](https://i.loli.net/2019/03/11/5c86292530aca.jpg)

#### Iterable接口

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Iterable接口中只定义了一个方法：iterator，返回集合Iterator对象，所有实现了Iterator接口的集合都可以使用foreach循环进行遍历。

```java
public interface Iterable<T> {
    Iterator<T> iterator();
}
```

#### Collection接口

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这是集合类的基本接口，其中声明了很多有用的方法，所有的类都需要实现里面的方法，如果我们实现这些接口无疑增加很多的工作量，所以Java 类库提供了一个类AbstractCollection，它将基础方法size 和iterator 抽象化了。

#### List接口

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有序集合，加到容器中的特定位置。可以采用两种访问方式，一种是使用一个迭代器访问，这样必须顺序访问；另一种成为随机访问，是使用整数索引访问，这样可以按任意循序访问元素。

#### Set接口

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Set接口相当于Collection接口,set中不允许添加重复内容。

#### SortSet SortMap接口

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用于提供排序的比较器对象

#### NavigableSet 和NavigableMap

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接口NavigableSet 和NavigableMap, 其中包含一些用于搜索和遍历有序集和映射的方法。



## 具体的集合

### 链表

List接口 实现接口的类LinkedList

- java中的链表是双向链表，每个节点及存放着他的前驱结点，也存放着他的后驱节点
- 链表可以理解为队列，先进先出。当然他也提供了反向遍历链表的方法。
- 集合类库中提供了子接口ListIterator，其中提供了一些有用的方法，比如反向遍历链表

```java
E previous()
boolean hasPrevious()
```

- **Add方法在迭代器位置之前添加一个新对象**
- 如果迭代器发现它的集合被另一个迭代器修改了， 或是被该集合自身的方法修改了， 就会抛出一个ConcurrentModificationException 异常。这类问题，可以根据需要给容器附加许多迭代器，但是这些迭代器只能读取列表，弄外，在单独设置一个既能读又能写的迭代器。
- **链表只负责跟踪对列表的结构性修改， 例如， 添加元素、删除元素。set 方法不被视为结构性修改。**
- 不支持快速随机访问，读取效率低，但是修改方便
- 元素访问方式一种是用**迭代器**， 另一种是用get 和set 方法随机地访问每个元素。后者不适用于链表，其实get方法还是顺序访问。效率底下
- 列表迭代器接口可以获取当前位置的索引。nextIndex方法返回下一次调用next方法是返回元素的位置。previousIndex返回下一次调用previous方法是返回元素的索引。
  理解：
  比如光标在这里
  A B | C D	此时读取的位置是B的位置
  下次调用next 光标变化 A B C | D 读取的位置是C的位置
  下次调用previous 光标变化 A | B C D 读取的位置是A的位置

代码练习

```java
package jiheTest;

import java.util.Iterator;
import java.util.LinkedList;
import java.util.List;
import java.util.ListIterator;

public class LinkedListTest {
	public static void main(String[] args) {
		List<String> a=new LinkedList<>();
		a.add("a");
		a.add("b");
		a.add("c");
		List<String> b=new LinkedList<>();
		b.add("A");
		b.add("B");
		b.add("C");
		
		ListIterator<String> aIter=a.listIterator();//跳过下一个节点，不读取
		Iterator<String> bIter=b.listIterator();
		/**
		 * 变化过程
		 * 起始		| a b c	越过第一个元素
		 * 		a | b c
		 * 		a A | b	c 	添加A元素 越过下一个元素
		 * 		a A b | c	
		 * 		a A b B | c
		 * 以下类推
		 */
		while (bIter.hasNext()) {
			if(aIter.hasNext())
				aIter.next();//跳过下一个元素
			aIter.add(bIter.next());//添加b元素
		}

		System.out.println(a);
		
		bIter=b.iterator();
		while (bIter.hasNext()) {
			bIter.next();//跳过第一个元素
			if(bIter.hasNext()) {
				bIter.next();//跳过下一个元素
				bIter.remove();//移除当前元素
			}
		}
		
		System.out.println(b);
		a.removeAll(b);//移除b中与a相同的元素
		System.out.println(a);
	}
}
```

结果
[a, A, b, B, c, C]
[A, C]
[a, b, B, c]

### 数组列表

ArrayList实现List接口

- 一般使用get set方法访问内部元素
- ArrayList封装了一个动态再分配的对象数组
- **Vector 类的所有方法都是同步**的。可以由两个线程安全地访问一个Vector 对象。但是， 如果由一个线程访问Vector, 代码要在同步操作上耗费大量的时间。这种情况还是很常见的。而**ArrayList 方法不是同步的**，因此， 建议在不需要同步时使用ArrayList, 而不要使用Vector。

### 散列集

散列表用***链表数组***表示

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;链表和数组是按照人们的意愿存储的，如果我们想查看某个指定的元素又记位置，我们必须遍历他们，消耗很多时间，而使用散列集我们可以快速查找我们需要的内容。

1. 使用散列集我们无法控制元素出现的位置，他们将按照有利于操作目的的原则组织数据
2. 散列码定义：

- 散列表示为每一个对象计算一个整数，称为散列码散列码是由对象域产生的一个整数。
- 具有不同的数据域的对象将产生不同的散列码
  ep:使用String类的HashCode方法生成的
  ![1552096929030](https://i.loli.net/2019/03/10/5c8501eac3813.png)

3. 自定义类，我们需要自己负责实现这个类的HashCode方法。

- 自己实现的hashcode方法应该和equals方法兼容，即**如果e.equals(b)为true，a b对象拥有相同的散列码**
- 散列码的计算只与散列的对象有关，与散列表中的其他对象无关

4. java中，散列表用**链表数组**表示。

- 每个列表称为**桶**（bucket），要想査找表中对象的位置， 就要**先计算它的散列码， 然后与桶的总数取余， 所得到的结果就是保存这个元素的桶的索引**。例如， 如果某个对象的散列码为76268, 并且有128 个桶， 对象应该保存在第108 号桶中（ 76268除以128 余108 )。或许会很幸运， 在这个桶中没有其他元素， 此时将元素直接插人到桶中就可以了。当然， 有时候会遇到桶被占满的情况， 这也是不可避免的。这种现象被称为**散列冲突**（ hashcollision) o 这时， 需要**用新对象与桶中的所有对象进行比较， 査看这个对象是否已经存在**。如果散列码是合理且随机分布的， 桶的数目也足够大， 需要比较的次数就会很少。
  ![1552096956820](https://i.loli.net/2019/03/10/5c8501eb34b3a.png)
- 设置桶数：一般设置为预计元素的75%—150%，最好将桶数设置成一个素数，防止**键的集聚** 。
- 事先不知道存储多少个元素，如果散列表太满，就要再散列。如果对散列表再散列，就需要创建一个桶数更多的列表。然后将所有元素填入该表中，然后丢弃该表。
- **填充因子：决定何时对散列表在散列**，例如， 如果装填因子为0.75 (默认值)，而表中超过75%的位置已经填人元素， 这个表就会用双倍的桶数自动地进行再散列。对于大多数应用程序来说， 装填因子为0.75 是比较合理的。

5. 散列表实例
   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;散列表可以用于实现几个重要的数据结构。其中最简单的是**set 类型**。set 是没有重复元素的元素集合。set 的add 方法首先在集中查找要添加的对象， 如果不存在，就将这个对象添加进去。

- HashSet 类，它实现了基于散列表的集。可以用add 方法添加元素。contains 方法已经被重新定义， 用来快**速地查看是否某个元素已经出现在集中。它只在某个桶中査找元素，而不必查看集合中的所有元素**（add执行原理）。
- **散列集迭代器将依次访问所有的桶。由于散列将元素分散在表的各个位置上，所以访问它们的顺序几乎是随机的**。只有不关心集合中元素的顺序时才应该使用HashSet。

栗子：

```
public class HashSetTest {
	public static void main(String[] args) {
		Set<String> words=new HashSet<>();//HashSet继承set
		long totalTime=0;
		
		try(Scanner in=new Scanner(System.in)){		//输入
			while(in.hasNext()) {
				String word=in.next();//读取文件中的单词
				long callTime=System.currentTimeMillis();
				words.add(word);
				callTime=System.currentTimeMillis()-callTime;
				totalTime+=callTime;//计算总读取时间
			}
			
			Iterator<String> iter=words.iterator();
			for (int i=0;i<=20 && iter.hasNext();i++) {
				System.out.println(iter.next());//打印集合中的元素
			}
		}
		System.out.println("...");
		System.out.println(words.size()+" distinct words."+totalTime+" milliseconds.");
	}
}
```

> - 补充知识点：在命令行调用某个程序输入文件内容是，不能再当级目录下
> - 使用 > 表示输入文件内容

![1552097000408](https://i.loli.net/2019/03/10/5c8501ea916db.png)

![1552097106176](https://i.loli.net/2019/03/10/5c8501ea2ffb4.png)

文件一test.txt&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;文件二 test1.txt




&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![1552097213238](https://i.loli.net/2019/03/10/5c8501ea8ed60.png)

​	编译输出结果，可以看出在文件二中，输出的内容顺序打乱了，证明了文章开始的那句话***使用散列集我们无法控制元素出现的位置，他们将按照有利于操作目的的原则组织数据***
参考链接：
https://blog.csdn.net/PacosonSWJTU/article/details/50317553



### 树集TreeSet

1. 树集是一个有序集合（sorted collection）。可以以任意顺序将元素插入到集合中，在对集合进行遍历时，每个值将自动de按照排序后的顺序出现。使用**红黑树**

```
SortedSet<String> sorter = new TreeSet<>(); // TreeSet implements SortedSet
sorter.add("bob");
sorter.add("car");
sorter.add("amy");
for(String s: sorter)
    System.println(s);
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结果：amy bob car

- 将元素添加到树中比添加到散列集中慢，但是比将元素添加到数组或链表中要快的多。
- 迭代器总是以排好序的方式访问每个元素
- 使用树集必须**实现Comparable接口或者构造集的时候提供一个Comparator**
- 树的排序必须是全序的，任意两个元素必须是可比的
  更详细的内容可以见：https://blog.csdn.net/PacosonSWJTU/article/details/50319649
  关于对象的比较

```java
package hashSet;

import java.util.Objects;

public class Item implements Comparable<Item>{//实现Comparable接口 完成compare方法

	private String description;
	private int partNumber;
	
	public Item() {
		super();
	}

	public Item(String description, int partNumber) {
		super();
		this.description = description;
		this.partNumber = partNumber;
	}

	public String getDescription() {
		return description;
	}

	public void setDescription(String description) {
		this.description = description;
	}

	public int getPartNumber() {
		return partNumber;
	}

	public void setPartNumber(int partNumber) {
		this.partNumber = partNumber;
	}

	@Override
	public String toString() {
		return "Item [description=" + description + ", partNumber=" + partNumber + "]";
	}

	@Override
	public int hashCode() {//Hashcode覆写
		/*final int prime = 31;
		int result = 1;
		result = prime * result + ((description == null) ? 0 : description.hashCode());
		result = prime * result + partNumber;
		return result;*/
		return Objects.hash(description,partNumber);
	}

	/**
	 * ==比较的是对象的存储地址
	 * equals没有覆写的情况下也是比较对象的存储地址
	 */
	@Override
	public boolean equals(Object obj) {//equals覆写
		if (this == obj)	//如果引用地址相同，返回true
			return true;
		if (obj == null)	//判断是否为空
			return false;
		if (getClass() != obj.getClass())//判断类型是否相同
			return false;
		Item other = (Item) obj;		//强制类型转换
		/*if (description == null) {
			if (other.description != null)
				return false;
		} else if (!description.equals(other.description))
			return false;
		if (partNumber != other.partNumber)
			return false;
		return true;*/
		return Objects.equals(description,other.description)&&partNumber==other.partNumber;
	}

	@Override
	public int compareTo(Item o) {
		// TODO Auto-generated method stub
		int diff=Integer.compare(partNumber, o.partNumber);
		return diff!=0?diff:description.compareTo(o.description);
	}

}

```

上面注释的内容是自动生成代码，有一定的缺陷

```java
package hashSet;

import java.util.Comparator;
import java.util.NavigableSet;
import java.util.SortedSet;
import java.util.TreeSet;

public class TreeSetTest {

	public static void main(String[] args) {
		SortedSet<Item> parts=new TreeSet<>();		//SortSet比较器
		parts.add(new Item("Toaster",1234));
		parts.add(new Item("Widget",4562));
		parts.add(new Item("Modem",9912));
		System.out.println(parts);
		
		//便于定位元素以及反向遍历
		NavigableSet<Item> sortByDescription =new TreeSet<>(Comparator.comparing(Item::getDescription));
		sortByDescription.addAll(parts);
		System.out.println(sortByDescription);
	}

}

```

结果：

```java
[Item [description=Toaster, partNumber=1234], Item [description=Widget, partNumber=4562], Item [description=Modem, partNumber=9912]]
[Item [description=Modem, partNumber=9912], Item [description=Toaster, partNumber=1234], Item [description=Widget, partNumber=4562]]

```

关于Comparable和Comparator区别见链接：https://blog.csdn.net/pacosonswjtu/article/details/50320775

### 队列与双端队列

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;队列可以让人们有效地在尾部添加一个元素， 在头部删除一个元素。有两个端头的队列， 即双端队列， 可以让人们有效地在头部和尾部同时添加或删除元素。不支持在队列中间添加元素。在Java SE 6 中引人了Deque 接口， 并由ArrayDeque 和LinkedList 类实现。这两个类都提供了双端队列， 而且在必要时可以增加队列的长度。

### 优先级队列

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;优先级队列（priority queue) 中的元素**可以按照任意的顺序插人**，却总是按照排序的顺序进行检索。也就是说，**无论何时调用remove 方法， 总会获得当前优先级队列中最小的元素**。然而， 优先级队列并没有对所有的元素进行排序。如果用迭代的方式处理这些元素，并不需要对它们进行排序。优先级队列使用了一个优雅且高效的数据结构，称为**堆（ heap)**。堆是一个可以自我调整的二叉树，**对树执行添加（ add) 和删除（ remore) 操作， 可以让最小的元素移动到根，而不必花费时间对元素进行排序。**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;与TreeSet—样，一个优先级队列既**可以保存实现了Comparable 接口的类对象， 也可以保存在构造器中提供的Comparator 对象。**



## 映射

映射Map （key-value）

### 基本映射操作

* 通用映射：HashMap TreeMap

* 散列映射对键散列。树映射用键的整体顺序对元素进行排序，并将其组织成搜索树

* **散列或比较函数只能用于键。值不能散列或比较(只与key有关， 与value 无关)**

* 如果在映射中没有给定与键对应的信息，get将返回null

* 迭代处理键和值

```java
Map<String,Integer> scores= new HashMap<>();
scores.put("1", 1);
scores.put("2", 2);
scores.put("3", 3);
scores.put("4", 4);
scores.put("5", 5);
scores.forEach((k,v)->{
    k+=1;
    v++;
    System.out.println("key="+k+" value="+v);
});
```

结果：

```java
key=11 value=2
key=21 value=3
key=31 value=4
key=41 value=5
key=51 value=6
```

也可以使用以下方法

```java
for(Map.Entry<String , Employee> entry: staff.entrySet())
{
    String key = entry.getKey();
    Employee e = entry.getValue();
    do sth with key
}
```



### 更新映射项

不是很懂这部分内容，下面代码是网上copy下来的，如果在经常更新Map映射项的话，可以使用下面的方法

```java
private Map<String,int> table = new HashMap<String,int>();
public void update(String key, int val) {
    if( !table.containsKey(key) ) return;
    Entry<String,int> entry;
    for( entry : table.entrySet() ) {
        if( entry.getKey().equals(key) ) {
            entry.setValue(val);
            break;
        }
    }
}
```

当然也可以使用，get 再put的方法

链接：[更新映射项](https://codeday.me/bug/20170712/41026.html)

### <span style="color:red">映射视图</span>

* Map映射视图右三种，键集、值集、键值集

* 获取三种视图的方法

  ![三种方法](https://i.loli.net/2019/03/11/5c85de31c6043.jpg)

  栗子：

```java
package mapTest;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class MapTest {

	public static void main(String[] args) {
		Map<String,String> map=new HashMap<>();
		map.put("1", "1");
		map.put("2", "2");
		map.put("3", "3");
		map.put("4", "4");
		map.put("5", "5");
		
		//输出键集
		System.out.println("输出键集");
		Set<String> keys=map.keySet();
		for (String string : keys) {
			System.out.println(string);
		}
		System.out.println();
		
		//输出值集
		System.out.println("输出值集");
		Collection<String> values=map.values();
		for (String s : values) {
			System.out.println(s);
		}
		System.out.println();
		
		//输出键值集
		System.out.println("输出键值集");
		Set<Map.Entry<String, String>> entries=map.entrySet();
		for (Map.Entry entry : entries) {
			System.out.println(entry);
			System.out.println(entry.getKey()+ "  " +entry.getValue());
		}
		System.out.println(map);
	}
}

```

​	如果在键集视图上调用迭代器的remove 方法， 实际上会从映射中删除这个键和与它关联的值。不过，不能向键集视图增加元素。另外， 如果增加一个键而没有同时增加值也是没有意义的。如果试图调用add 方法， 它会抛出一个UnsupportedOperationException。条目集视图有同样的限制，尽管理论上增加一个新的键/ 值对好像是有意义的。







### 专用映射散列集

#### <span style="color:red">弱散映射</span>

引入问题：

* 如果有一个值，对应的键已经不再使用了， 将会出现什么情况呢？ 假定对某个键的最后一次引用已经消亡，不再有任何途径引用这个值的对象了。但是， 由于在程序中的任何部分没有再出现这个键， 所以， 这个键/ 值对无法从映射中删除。
* 垃圾回收器无法回收，垃圾回收器跟踪活动的对象。只要映射对象是活动的，其中的所有桶也是活动的， 它们不能被回收(<span style="color:red">不能被原因</span>)

解决方案：

* 程序删除长期无用的值
* 使用WeakHashMap。当对键的唯一引用来自**散列条目**时， 这一数据结构将与垃圾回收器协同工作一起删除键/ 值对。

WeakHashMap原理

​	WeakHashMap 使用弱引用（ weak references) 保存键。**WeakReference 对象将引用保存到另外一个对象中， 在这里， 就是<span style="color:red">散列表键</span>**。对于这种类型的对象， 垃圾回收器用一种特有的方式进行处理。通常， 如果垃圾回收器发现某个特定的对象已经没有他人引用了， 就将其回收。然而， 如果某个对象只能由WeakReference 引用， **垃圾回收器仍然回收它，但要将引用这个对象的弱引用放人队列中。WeakHashMap 将周期性地检查队列， 以便找出新添加的弱引用**。一个弱引用进人队列意味着这个键不再被他人使用， 并且已经被收集起来。于是， WeakHashMap 将删除对应的条目。



#### <span style="color:red">链接散列集与映射</span>

本章节不是很懂，有待深究

* <span style="color:red">**LinkedHashSet和LinkedHashMap**，用来记住元素项的插入顺序</span>，避免散列表中的项从表面上看是随机排列的，当条目插入到表中时，就会并入到双向链表中。个人理解就是用了两种存储方式，Hash表（至于是链表还是数组表就不得而至）存储方式和双向链表

  ![](https://i.loli.net/2019/03/11/5c861f403e0fd.jpg)

* 链接散列映射将用访问顺序，而不是插入顺序，对映射条目进行迭代，每次调用get或put，收影响的条目将从当前的位置删除，并放到条目链表的 尾部（只有条目在链表中的位置会受影响， 而散列表中的桶不会受影响。一个条目总位于与键散列码对应的桶中，个人理解：<span style="color:blue">其实就是修改了双向链表的存储顺序，有之前的乱序改为顺序，而Hash桶中的位置不变</span>）

  实现方法：

```java
LinkedHashMap<K, V>(initialCapacity, loadFactor, true)
```



* **访问顺序对于实现高速缓存的“ 最近最少使用” 原则十分重要**。例如， 可能希望将访问频率高的元素放在内存中， 而访问频率低的元素则从数据库中读取。当在表中找不到元素项且表又已经满时， 可以将迭代器加入到表中， 并将枚举的前几个元素删除掉。这些是近期最少使用的几个元素。

  自动化实现该过程，构建LinkedHashMap子类。覆盖以下方法：

```java
protected boolean removeEldestEntry(Map.Entry<K， V> eldest)
```



* 栗子：每当方法返回true 时， 就添加一个新条目，从而导致删除eldest 条目。例如，下面的高速缓存可以存放100 个元素：

```java
Map<K, V> cache = new
LinkedHashMapo(128, 0.75F, true)
{
	protected boolean removeEldestEntry(Map.Entry<K, V> eldest)
	{
		return size() > 100;
	}
}();
```





#### 枚举集和映射

此处不是很重要，只是使用方法

* EnumSet是一个枚举类型元素集的高效实现。由于枚举类型只有有限个实例，所以EnumSet内部用位序列实现。如果对应的值在集中，则相应的位被置为1。 

* EnumSet类没有公共的构造器。可以使用静态工厂方法构造这个集：

```java
enum Weekday{MonDay,TuesDay,WenesDay,ThursDay,FriDay,SaturDay,SunDay};

EnumSet<Weekday> enumSet=EnumSet.allOf(Weekday.class);// 创建一个包含指定元素类型的所有元素的枚举 set

EnumSet<Weekday> never=EnumSet.noneOf(Weekday.class);// 创建一个具有指定元素类型的空枚举 set

 EnumSet<Weekday> workday=EnumSet.range(Weekday.MonDay, Weekday.FriDay);//创建一个最初包含由两个指定端点所定义范围内的所有元素的枚举 set

 EnumSet<Weekday> mwf=EnumSet.of(Weekday.MonDay, Weekday.FriDay,Weekday.SunDay,Weekday.SaturDay);//创建一个最初包含指定元素的枚举 set
```



栗子：

```java
import java.util.EnumSet;
public class Test {

    public Test() {
        // TODO Auto-generated constructor stub
    }

    enum Weekday{MonDay,TuesDay,WenesDay,ThursDay,FriDay,SaturDay,SunDay};

    public static void main(String[] args) {
        // TODO Auto-generated method stub
        EnumSet<Weekday> enumSet=EnumSet.allOf(Weekday.class);// 创建一个包含指定元素类型的所有元素的枚举 set
        enumSet.remove(Weekday.MonDay);//创建的枚举元素，可以被删除
        System.out.println(enumSet);

        EnumSet<Weekday> never=EnumSet.noneOf(Weekday.class);// 创建一个具有指定元素类型的空枚举 set
        never.add(Weekday.FriDay);
        never.add(Weekday.SunDay);      
        never.add(Weekday.MonDay);//会自动排序
        System.out.println(never);

        EnumSet<Weekday> workday=EnumSet.range(Weekday.MonDay, Weekday.FriDay);//创建一个最初包含由两个指定端点所定义范围内的所有元素的枚举 set
        System.out.println(workday);//输出MonDay~FriDay之间的枚举变量


        EnumSet<Weekday> mwf=EnumSet.of(Weekday.MonDay, Weekday.FriDay,Weekday.SunDay,Weekday.SaturDay);//创建一个最初包含指定元素的枚举 set
        System.out.println(mwf);

        mwf.add(Weekday.WenesDay);//添加元素
        System.out.println(mwf);
    }
}
```

结果：

```java
[TuesDay, WenesDay, ThursDay, FriDay, SaturDay, SunDay]
[MonDay, FriDay, SunDay]
[MonDay, TuesDay, WenesDay, ThursDay, FriDay]
[MonDay, FriDay, SaturDay, SunDay]
[MonDay, WenesDay, FriDay, SaturDay, SunDay]
```

* **可以修改 Set接口的常用方法来修改 EnumSet**

* **EnumMap 是一个键类型为 枚举类型的映射表。** 它可以直接且高效地用一个值数组实现， 在使用时， 需要再构造器中指定键类型：

```java
EnumMap<Weekday, Employee> map = new EnumMap<>(Weekday.class);
```



个人理解：其实EnumSet、EnumMap用法类推Set、Map，只是增加了Enum枚举定义

参考链接：

https://blog.csdn.net/PacosonSWJTU/article/details/50331019

https://blog.csdn.net/u014322541/article/details/45677283

#### 标识散列映射

不是很重要

​	类IdentityHashMap有特殊的作用。在这个类中，键的散列值不是用hashCode函数计算的，而是用System.identityHashCode方法计算的。这是**Object.hashCode方法根据对象的内存地址类计算散列码所使用的方式**。而且，在对两个对象进行比较时，**IdentityHashMap类使用==，而不是用equals**。



## 视图和包装器

视图理解：

摘取理解

* java中的视图，可以说其实就是一个具有限制的集合对象，只不过这里的不是集合对象，而是一个视图对象。

* 集合视图就是把集合里面的东西给你展示出来。然后功能没有原始集合多。

* 是查看集合中部分或者全部数据的窗口。

* 对原集合的一层包装 ，不同的集合视图有不同的用途，有的是只读有的是同步的。针对视图的操作会影响到集合的数据。

  链接：https://blog.csdn.net/weixin_38201813/article/details/70665574

原文：

* **使用视图可以获得其他的实现了集合接口Collection和映射表Map接口的对象**

  栗子：映射表类的keySet方法，看起来好像它创建了一个新集(Set),并将映射表中的所有键都填进去，然后返回这个集。事实是该方法返回一个实现Set接口的类对象，这个类方法对原映射表进行操作。这种集合就称为视图。

个人理解：

​	视图获得其他实现集合接口Collection、Map接口的对象；可以通过这些访问他们的方法以及他们的子类、实现接口的方法，但是有的方法是无法使用的，因为数据类型的原因，可以在轻量级集合包装器中看到。



写在前面的话：

​	视图以下部分大多是在讲视图的适应范围，不同的情况不同使用，应该在结合项目写代码的时候有更深入的理解。视图可以设置子范围，试图有只能修改或者只能读的，或者转化异常的。。。大概就是下面的内容，个人理解的不是很深，以后还得深究。



### 轻量级集合包装器

* Arrays 类的静态方法asList 将返回一个包装了普通Java 数组的List 包装器。这个方法可以将数组传递给一个期望得到列表或集合参数的方法。

* 返回的对象不是ArrayList。它是一个视图对象， 带有访问底层数组的get 和set 方法。改变数组大小的所有方法（例如， 与迭代器相关的add 和remove 方法）都会抛出一个Unsupported OperationException 异常

```java
public class Test {
	public static void main(String[] args) {
        Map<String,String> a=new HashMap<>();
        String[] s=new String[10];
        /*自动补全变量快捷键Ctrl+Alt+v*/
        List<String> list = Arrays.asList(s);
        list.set(2,"a");
        //list.add("10");//此处报错remove方法同样报错
        System.out.println(list.get(2));
        System.out.println(list.size());
        //将返回一个实现了List 接口的不可修改的对象， 并给人一种包含》个元素， 每个元素都像是一个anObject 的错觉。 创建了100个default，但是对象只存储一次，字符串值存储一次，存储代价小
        List<String> set= Collections.nCopies(100,"default");
        System.out.println(set.get(0)==set.get(10));
    }
}
```



结果：

```java
a
10
true
```

* asList方法可以接收可变数目的参数。源码：

```java
    /**
     * Returns a fixed-size list backed by the specified array.  (Changes to
     * the returned list "write through" to the array.)  This method acts
     * as bridge between array-based and collection-based APIs, in
     * combination with {@link Collection#toArray}.  The returned list is
     * serializable and implements {@link RandomAccess}.
     *
     * <p>This method also provides a convenient way to create a fixed-size
     * list initialized to contain several elements:
     * <pre>
     *     List&lt;String&gt; stooges = Arrays.asList("Larry", "Moe", "Curly");
     * </pre>
     *
     * @param <T> the class of the objects in the array
     * @param a the array by which the list will be backed
     * @return a list view of the specified array
     */
    @SafeVarargs
    @SuppressWarnings("varargs")
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
```



* <span style="color:red">**区分Collection和Collections **</span>

  * java.util.Collection 是一个**集合接口**。它提供了对集合对象进行基本操作的通用接口方法。Collection接口在Java 类库中有很多具体的实现。Collection接口的意义是为各种具体的集合提供了最大化的统一操作方式。

    Collection   
    ├List   
    │├LinkedList   
    │├ArrayList   
    │└Vector   
    │　└Stack   
    └Set 

    ![Collection方法接口](https://i.loli.net/2019/03/12/5c87561922435.jpg)

    继承接口见开始的那张图

    关于Collection接口官方解释：

  ```java
   * The root interface in the <i>collection hierarchy</i>.  A collection
   * represents a group of objects, known as its <i>elements</i>.  Some
   * collections allow duplicate elements and others do not.  Some are ordered
   * and others unordered.  The JDK does not provide any <i>direct</i>
   * implementations of this interface: it provides implementations of more
   * specific subinterfaces like <tt>Set</tt> and <tt>List</tt>.  This interface
   * is typically used to pass collections around and manipulate them where
   * maximum generality is desired.
  ```

  

  ​	Collection 层次结构 中的**根接口**。**Collection 表示一组对象**，这些对象也称为 collection 的元素。一些 collection 允许有重复的元素，而另一些则不允许。一些 collection 是有序的，而另一些则是无序的。JDK 不提供此接口的任何直接 实现：它提供更具体的子接口（如 Set 和 List）实现。此接口通常用来传递 collection，并在需要最大普遍性的地方操作这些 collection。

  

  * java.util.Collections 是一个包装类。它包含有各种有关集合操作的**静态多态方法**。此类**不能实例化**，就像一**个工具类**，服务于Java的Collection框架。

    关于Colections：

  ```java
   * This class consists exclusively of static methods that operate on or return
   * collections.  It contains polymorphic algorithms that operate on
   * collections, "wrappers", which return a new collection backed by a
   * specified collection, and a few other odds and ends.
  ```

  

  ​	**此类完全由在 collection 上进行操作或返回 collection 的静态方法组成**。它**包含在 collection 上操作的多态算法，即“包装器”，**包装器返回由指定 collection 支持的新 collection，以及少数其他内。

* 调用以下代码： **Collections.singleton(anObject); 则返回一个视图对象。** 这个对象实现了Set 接口。 返回的对象实现了一个不可修改的单元素集， 而不需要付出建立数据结构的 开销。 singletonList 和 singletonMap 方法类似； 

  ![](https://i.loli.net/2019/03/12/5c87599660c84.jpg)



### 子范围

试图接下来内容看不懂了，很是模糊，原文摘要参看以下链接

* 可以为很多集合建立子范围视图；

  如， List g2 = staff.subList(10,20); 从列表staff 中取出 第10个~第19个元素；（第一个 索引包含在内，第二个索引不包含在内）

* 可以将任何操作应用到子范围， 并且能够自动地反应整个列表的情况；

* 对于有序集合映射表， 可以使用 排序顺序而不是元素位置建立子范围。

  * SortedSet 接口说明了3个方法	返回 大于等于from 小于to的所有元素子集

```java
SortedSet<E> subSet(E from ,E to)
SortedSet<E> headSet(E to)
SortedSet<E> tailSet(E from)
```

​	* 有序映射表有类似方法 返回映射表视图， 该映射表包含键落在指定范围内的所有元素

![](https://i.loli.net/2019/03/12/5c875c578f98e.jpg)

摘自：https://blog.csdn.net/PacosonSWJTU/article/details/50333509 视图部分不是很理解，有需求的请参照该链接

### 不可修改的视图

* Collections 还有几个方法， 用于产生集合的不可修改视图。 这些视图对现有集合增加了一个运行时的检查。 如果发现试图对集合进行修改， 就抛出一个异常， 同时这个集合将保持未修改状态

  * 一下方法获取不可修改视图

```java
Collections.unmodifiableCollection
Collections.unmodifiableList
Collections.unmodifiableSet
Collections.unmodifiableSortedSet
Collections.unmodifiableMap
Collections.unmodifiableSortedMap
```

​	关于这六中方法，可以使用时查看源码注释

​	这里放一个：

```java
* Returns an unmodifiable view of the specified list.  This method allows
* modules to provide users with "read-only" access to internal
* lists.  Query operations on the returned list "read through" to the
* specified list, and attempts to modify the returned list, whether
* direct or via its iterator, result in an
```

* **不可修改视图并不是 集合本身不可修改（只是无法通过其投影出的视图修改原始集合）**。仍然可以通过集合的原始引用对集合进行修改。并且仍然可以让集合的元素调用更改方法；

* **由于视图只是包装了 接口而不是 实际的集合对象， 所以只能访问 接口中定义的方法；** 如， LinkedList 类有一些非常方便的方法， addFirst 和 addLast ， 它们都不是 List接口的方法，不能通过修改视图进行访问；

* waning：

  ​	unmodifiableCollection 方法 ，它的equals 方法不调用底层集合的equals 方法。相反， 它继承了Object 类的equals 方法， 这个方法只是检测两个对象是否是同一个对象。如果将集或列表转换成集合， 就再也无法检测其内容是否相同了。• 视图就是以这种方式运行的， 因为内容是否相等的检测在分层结构的这一层上没有定义妥当。视图将以同样的方式处理hashCode 方法。

  ​	unmodifiableSet 类和unmodifiableList 类却使用底层集合的equals 方法和hashCode 方法。

### 同步视图

多线程问题，锁机制？ 其实就是不能同步修改内容，有个锁机制

问题：

​	如果由多个线程访问集合，就必须确保集不会被意外地破坏。例如， 如果一个线程试图将元素添加到散列表中，同时另一个线程正在对散列表进行再散列，其结果将是灾难性的。

解决办法：

* 类库的设计者使用视图机制来确保常规集合的线程安全， 而不是实现线程安全的集合类

  栗子：

```java
Map<String, Employee〉map = Collections.synchronizedMap(new HashMap<String, Employee>0)；
```




  现在， 就可以由多线程访问map 对象了。像get 和put 这类方法都是同步操作的， **即在另一个线程调用另一个方法之前，刚才的方法调用必须彻底完成**

### 受查视图

提出问题：

```java
ArrayList<String> strings = new ArrayListo()；
ArrayList rawList = strings; // warning only, not an error, for compatibility with legacy code
rawList.add(new DateO) ; // now strings contains a Date object!
```

解决办法：

```java
List<String> safestrings = Collections.checkedList(strings，String,class);
```

视图的add 方法将检测插人的对象是否属于给定的类。如果不属于给定的类， 就立即抛出一个ClassCastException。这样做的好处是错误可以在正确的位置得以报告：

```java
ArrayList rawList = safestrings;
rawList.add(new DateO)；// checked list throws a ClassCastException
```



* 受查视图受限于虚拟机可以运行的运行时检查。例如， 对于ArrayList <Pair<String>>, 由于虚拟机有一个单独的“ 原始” Pair 类， 所以，无法阻止插入Pair <Date>。





## 算法

### 排序与混排

* Collections 类中的sort 方法可以对实现了List 接口的集合进行排序

```java
Collections.sort(xxx);
```

* 如果想采用**其他方式对列表进行排序**，可以使用List 接口的sort 方法并传入一个Comparator 对象。可以如下按工资对一个员工列表排序：

```java
staff.sort(Comparator.comparingDouble(Employee::getSalary));
```

<span style="color:red">注意上面的写法，注意！！！很值得学习使用</span>

* 如果想按照**降序**对列表进行排序， 可以使用一种非常方便的静态方法Collections.reverseOrder()。 这个方法将返回一个比较器， 比较器则返回b.compareTo(a)。例如:

```java
staff.sort(Comparator.reverseOrder())
```

* **java的排序实现**

  直接将所有元素转人一个数组， 对数组进行排序，然后，再将排序后的序列复制回列表。

* shuffle方法随机混排列表中元素的顺序。

### 二分查询

* Collections的binarySearch方法实现该方法

* 要想查找某个元素， 必须提供集合以及要查找的元素。如果集合没有采用Comparable 接口的compareTo 方法进行排序， 就还要提供一个比较器对象。

```java
i = Collections.binarySearch(c, element) ;
i = Collections.binarySearch(c, element, comparator);
```

* 如果binarySearch 方法返回的数值大于等于0, 则表示匹配对象的索引。也就是说,c.get(i) 等于在这个比较顺序下的element。如果返回负值， 则表示没有匹配的兀素。但是，可以利用返回值计算应该将element 插人到集合的哪个位置， 以保持集合的有序性。插人的位置是

```java
insertionPoint = -i - 1;
```

```java
if (i < 0)
	c.add(-i - 1, element) ;
```

<span style="color:red">使用技巧</span>

### 简单算法

看源码去，用到算法时候，先baidu ，看源码说明！。主要是我太懒了，懒得将这些东西弄上去

### 批操作

* 成批删除

  colll.removeAll (coll2);

  将从coll1 中删除coll2中出现的所有元素。

* colli.retainAll(coll2);

  会从coll1 中删除所有**未**在coll2 中出现的元素

* 找出两个集合的交集 <span style="color:red">技巧！</span>

  1. 建立一个新集来存放结果

     ```java
     Set<String> result = new HashSeto(a);
     ```

  2. retainAll()

     ```java
     result.retainAll(b);
     ```

* 一个很有技巧的例子：假设有一个映射， 将员工ID映射到员工对象， 而且建立了一个将不再聘用的所有员工的ID。

  ```java
  Map<String, Employee> staffMap = . . .;
  Set<String> terainatedlDs = . . .;
  ```

  直接建立一个键集，并删除终止聘用关系的所有员工的ID。

  ```java
  staffMap.keySet().removeAll (terminatedIDs);
  ```

  

### 集合和数组的转换

* 数组转换成集合

  ```java
  String[] value=...
  HashSet<String> staff=new HashSet<>(Arrays.adList(values));//注意Arrays.adList(values)返回的是一个视图，前面有说
  ```

* 集合转换成数组

  ```java
  Object[] values=staff.toArray();//注意返回类型为Object，而不是我们需要的类型，不能强制类型转换
  ```

  解决办法：

  ```java
  String[] values=staff.toArray(new String[0]);
  ```

  或者这种指定长度数组

  ```java
  staff.toArray(new String[staff.size()]) ;
  ```

  



## 遗留的集合

挂个图，用的不多，暂时不深究，以后又需要在学

![](https://i.loli.net/2019/03/12/5c876e9d5e5bd.jpg)

## 写在尾巴这里的话

​	耗时1周将集合这部分内容看完了，可以说是很慢了，确实有很多东西不是很了解，以前看视频、看李兴华的那本书也是摸棱两可，很多概念不是很了解，看完了这一部分，对集合视图有了大致的了解，但是还需要深入学习，每种方法的实现，以及集合接口，集合类之间的关系，以及他们的适用范围，这将会是我看完核心卷在深入java学习的新章节。





附带集合类相关链接。

https://blog.csdn.net/sunhuaqiang1/article/details/52142873



![](https://i.loli.net/2019/03/12/5c877133c4b95.jpg)

​	上述类图中，**实线边框的是实现类**，比如ArrayList，LinkedList，HashMap等，**折线边框的是抽象类**，比如AbstractCollection，AbstractList，AbstractMap等，而点线边框的是接口，比如Collection，Iterator，List等。

​	 我们可以看到Collection是List、Set、Queue接口的父接口，故该接口中定义的方法可用于操作List、Set、Queue集合。

​	 发现一个特点，上述所有的集合类，都实现了Iterator接口，这是一个用于遍历集合中元素的接口，主要包含hashNext(),next(),remove()三种方法。它的一个子接口LinkedIterator在它的基础上又添加了三种方法，分别是add(),previous(),hasPrevious()。也就是说如果是先Iterator接口，那么在遍历集合中元素的时候，只能往后遍历，被遍历后的元素不会在遍历到，通常无序集合实现的都是这个接口，比如HashSet，HashMap；而那些元素有序的集合，实现的一般都是LinkedIterator接口，实现这个接口的集合可以双向遍历，既可以通过next()访问下一个元素，又可以通过previous()访问前一个元素，比如ArrayList。

​	 还有一个特点就是抽象类的使用。如果要自己实现一个集合类，去实现那些抽象的接口会非常麻烦，工作量很大。这个时候就可以使用抽象类，这些抽象类中给我们提供了许多现成的实现，我们只需要根据自己的需求重写一些方法或者添加一些方法就可以实现自己需要的集合类，工作流昂大大降低。