---
title: java核心卷Ⅰ-泛型
date: 2019-03-12 21:42:24
tags:
  - java
categories:
  - 读书笔记
  - java核心卷1
coments: true
---





本章节主要讲java核心卷Ⅰ泛型

<!--more-->

# 定义简单泛型类

```java
public class Pair<T>
{
    private T first;
    private T second;


    public Pair(){
   	    first=null;
    	second=null;
	}
    //这不算泛型方法 只用用类的泛型
    public Pair(T first,T second){
        this.first=first;
        this.second=second;
    }
    //同样也不是
    public T getFirstO {
        return first;
    }
    public T getSecondO {
        return second;
    }
    public void setFirst (T newValue) {
        first = newValue;
    }
    public void setSecond(T newValue) {
        second = newValue;
    }
}
```

# 泛型方法

```java
package faxing;


public class Test {
    //<T>是泛型的声明 第一个T是泛型的返回值 第二个T是泛型的形参
    public <T> T Test(T t){
        //？通配符，声明class1是Object或者继承于Object的子类类型
        Class<? extends Object> class1 = t.getClass();
        System.out.println(class1.getClass());
        return t;
    }
}
```

详细关于泛型方法与泛型类的语法参考

<https://blog.csdn.net/s10461/article/details/53941091>



# 泛型上下限

​	正如第二个代码所示使用extends表示泛型的上限制，这里extends后面不仅可以是类，也可以是接口。

但是不能一个是接口，一个是类，同时那个类是实现了接口的那种情况。



# 泛型diamagnetic与虚拟机

## 泛型擦出

在JVM中是不存在泛型的概念的，所有的泛型在编译时就会变成普通的类、接口、方法。上代码这是我们编辑代码时所见的，有泛型的概念

```java
public class Pair<T>
{
    private T first;
    private T second;


    public Pair(){
        first=null;
        second=null;
    }
    //这不算泛型方法 只用用类的泛型
    public Pair(T first,T second){
        this.first=first;
        this.second=second;
    }
    //同样也不是
    public T getFirstO {
        return first;
    }
    public T getSecondO {
        return second;
    }
    public void setFirst (T newValue) {
        first = newValue;
    }
    public void setSecond(T newValue) {
        second = newValue;
    }
    //这里的T不同于类中声明的那个T
    public <T> T Test(T t){
        //？通配符，声明class1是Object或者继承于Object的子类类型
        Class<? extends Object> class1 = t.getClass();
        System.out.println(class1.getClass());
        return t;
    }
}
```

在编译之后会变成下面代码的样子。其实是在编译时所有的泛型会变成其他类，规则是：

* 如果在类中指明了他继承的那个类，形式如下：

  ![](https://i.loli.net/2019/03/12/5c87b8fb1326d.png)

* 会编译成以他的继承的类、接口（其实不能说是继承，只是因为用extends更能体现继承的思想）为上限的类型，及那个类及那个类的子类，就是那个家伙或他的子子孙孙（应该优先子孙的）,就是下面这种类型

  ```java
  public class Pair
  {
      private Object first;
      private Object second;
  
  
      public Pair(){
      first=null;
      second=null;
      }
      //这不算泛型方法 只用用类的泛型
      public Pair(Object first,Object second){
          this.first=first;
          this.second=second;
      }
      //同样也不是
      public Object getFirstO {
          return first;
      }
      public Object getSecondO {
          return second;
      }
      public void setFirst (Object newValue) {
          first = newValue;
      }
      public void setSecond(Object newValue) {
          second = newValue;
      }
      //这里的T不同于类中声明的那个T
      public Object Test(Object t){
          //？通配符，声明class1是Object或者继承于Object的子类类型
          Object class1 = t.getClass();
          System.out.println(class1.getClass());
          return t;
      }
  }
  ```

* 带来两个问题

  1. 继承泛型带来的麻烦，(子类没有覆盖住父类的方法)

     <https://blog.csdn.net/weixin_37770552/article/details/77692449>

     上面那个代码，他在编译时变成了Object类型，包括里面的泛型参数。如果我们想继承他的方法，比如

     ```java
     class SonPair extends Pair<String>{
     	public void setFirst(String fir){....}
     }
     ```

     很明显能看出来的问题，我们准备覆盖setFirst方法，然而父类的那个方法的类型是Object，然而这里是String类型，覆盖失败。

     来看看编译器解决这个问题

     ```java
     编译器 会自动在 SonPair中生成一个桥方法(bridge method ) 强转
     public void setFirst(Object fir){
         setFirst((String) fir)
     }
     
     ```

     这样也带来一个问题，如果我们想覆盖getFirst方法，看下面代码

     ```java
     class SonPair extends Pair<String>{
     	public String getFirst(){....}
     }
     ```

     和之前生成的代码对比，我们生成的同名不同返回值的两个方法

     ```java
     ①String getFirst() // 自己定义的方法
     ②Object getFirst() // 编译器生成的桥方法
     ```

     这里给自己补充一个知识点：方法签名=方法名+参数列表。上图：

     ![](https://i.loli.net/2019/03/12/5c87ba0aa21ee.png)

     这样就带来了问题，我们不能写出方法签名相同的代码。

     但是jvm会用类型参数和返回值确定一个方法 。所以这里是可行的

  2. 泛型类型中的方法冲突

     在第一个代码中添加一下方法

     ```java
     public boolean equals(T value){
     	return (first.equals(value));
     }
     ```

     ![](https://i.loli.net/2019/03/12/5c87ba6b8499c.png)

     Name clash: The method equals(T) of type Pair has the same erasure as equals(Object) of type Object but does not override it

     所以我也不知道这里怎么解决。

  参考链接：

  关于泛型基本语法：<https://blog.csdn.net/s10461/article/details/53941091>

  关于类型擦除：<https://blog.csdn.net/weixin_37770552/article/details/77692449>



# 通配符类型

全文摘自这里，此处只是备份，怕链接失效，没有这么好的博客

<https://segmentfault.com/a/1190000005337789>



## 数组的协变

在了解通配符之前，先来了解一下数组。Java 中的数组是协变的，什么意思？看下面的例子：

```java
class Fruit {}
class Apple extends Fruit {}
class Jonathan extends Apple {}
class Orange extends Fruit {}


public class CovariantArrays {
    public static void main(String[] args) {
        Fruit[] fruit = new Apple[10];
        fruit[0] = new Apple(); // OK
        fruit[1] = new Jonathan(); // OK
        	// Runtime type is Apple[], not Fruit[] or Orange[]:
        try {
        	// Compiler allows you to add Fruit:
            fruit[0] = new Fruit(); // ArrayStoreException
        } catch(Exception e) { System.out.println(e); }
        try {
            // Compiler allows you to add Oranges:
            fruit[0] = new Orange(); // ArrayStoreException
        } catch(Exception e) { System.out.println(e); }
    }
} /* Output:
java.lang.ArrayStoreException: Fruit
java.lang.ArrayStoreException: Orange
*///:~
```

​	main 方法中的第一行，创建了一个 Apple 数组并把它赋给 Fruit 数组的引用。这是有意义的，Apple 是 Fruit 的子类，一个 Apple 对象也是一种 Fruit 对象，所以一个 Apple 数组也是一种 Fruit 的数组。这称作数组的协变，Java 把数组设计为协变的，对此是有争议的，有人认为这是一种缺陷。

​	尽管 Apple[] 可以 “向上转型” 为 Fruit[]，但数组元素的实际类型还是 Apple，我们只能向数组中放入 Apple或者 Apple 的子类。在上面的代码中，向数组中放入了 Fruit 对象和 Orange 对象。对于编译器来说，这是可以通过编译的，但是在运行时期，JVM 能够知道数组的实际类型是 Apple[]，所以当其它对象加入数组的时候就会抛出异常。

​	泛型设计的目的之一是要使这种运行时期的错误在编译期就能发现，看看用泛型容器类来代替数组会发生什么：

```java
// Compile Error: incompatible types:
ArrayList<Fruit> flist = new ArrayList<Apple>();
```

​	上面的代码根本就无法编译。当涉及到泛型时， 尽管 Apple 是 Fruit 的子类型，但是 ArrayList<Apple> 不是 ArrayList<Fruit> 的子类型，泛型不支持协变。

## 使用通配符

​	从上面我们知道，List\<Number\> list = ArrayList\<Integer\>\>这样的语句是无法通过编译的，尽管 Integer 是 Number 的子类型。那么如果我们确实需要建立这种 “向上转型” 的关系怎么办呢？这就需要通配符来发挥作用了。

## 上边界限定通配符

​	利用 <? extends Fruit> 形式的通配符，可以实现泛型的向上转型：

```java
public class GenericsAndCovariance {
	public static void main(String[] args) {
        // Wildcards allow covariance:
        List<? extends Fruit> flist = new ArrayList<Apple>();
        // Compile Error: can’t add any type of object:
        // flist.add(new Apple());
        // flist.add(new Fruit());
        // flist.add(new Object());
        flist.add(null); // Legal but uninteresting
        // We know that it returns at least Fruit:
        Fruit f = flist.get(0);
    }
}
```

​	上面的例子中， flist 的类型是 List<? extends Fruit>，我们可以把它读作：一个类型的 List， 这个类型可以是继承了 Fruit 的某种类型。注意，这并不是说这个 List 可以持有 Fruit 的任意类型。通配符代表了一种特定的类型，它表示 “某种特定的类型，但是 flist 没有指定”。这样不太好理解，具体针对这个例子解释就是，flist 引用可以指向某个类型的 List，只要这个类型继承自 Fruit，可以是 Fruit 或者 Apple，比如例子中的 new ArrayList<Apple>，但是为了向上转型给 flist，flist 并不关心这个具体类型是什么。

​	如上所述，通配符 List<? extends Fruit> 表示某种特定类型 ( Fruit 或者其子类 ) 的 List，但是并不关心这个实际的类型到底是什么，反正是 Fruit 的子类型，Fruit 是它的上边界。那么对这样的一个 List 我们能做什么呢？其实如果我们不知道这个 List 到底持有什么类型，怎么可能安全的添加一个对象呢？在上面的代码中，向 flist 中添加任何对象，无论是 Apple 还是 Orange 甚至是 Fruit 对象，编译器都不允许，唯一可以添加的是 null。所以如果做了泛型的向上转型 (List<? extends Fruit>>flist = new ArrayList\<Apple\>())，那么我们也就失去了向这个 List 添加任何对象的能力，即使是 Object 也不行。

​	另一方面，如果调用某个返回 Fruit 的方法，这是安全的。因为我们知道，在这个 List 中，不管它实际的类型到底是什么，但肯定能转型为 Fruit，所以编译器允许返回 Fruit。

​	了解了通配符的作用和限制后，好像任何接受参数的方法我们都不能调用了。其实倒也不是，看下面的例子：

```java
public class CompilerIntelligence {
    public static void main(String[] args) {
        List<? extends Fruit> flist =
        Arrays.asList(new Apple());
        Apple a = (Apple)flist.get(0); // No warning
        flist.contains(new Apple()); // Argument is ‘Object’
        flist.indexOf(new Apple()); // Argument is ‘Object’


        //flist.add(new Apple()); 无法编译


    }
}
```

​	在上面的例子中，flist 的类型是 List<? extends Fruit>，泛型参数使用了受限制的通配符，所以我们失去了向其中加入任何类型对象的例子，最后一行代码无法编译。

​	但是 flist 却可以调用 contains 和 indexOf 方法，它们都接受了一个 Apple 对象做参数。如果查看 ArrayList 的源代码，可以发现 add() 接受一个泛型类型作为参数，但是 contains 和 indexOf 接受一个 Object 类型的参数，下面是它们的方法签名：

```java
public boolean add(E e)
public boolean contains(Object o)
public int indexOf(Object o)
```

​	所以如果我们指定泛型参数为 <? extends Fruit>时，add() 方法的参数变为 ? extends Fruit，编译器无法判断这个参数接受的到底是 Fruit 的哪种类型，所以它不会接受任何类型。

​	然而，contains 和 indexOf 的类型是 Object，并没有涉及到通配符，所以编译器允许调用这两个方法。这意味着一切取决于泛型类的编写者来决定那些调用是 “安全” 的，并且用 Object 作为这些安全方法的参数。如果某些方法不允许类型参数是通配符时的调用，这些方法的参数应该用类型参数，比如 add(E e)。

​	当我们自己编写泛型类时，上面介绍的就有用了。下面编写一个 Holder 类：

```java
public class Holder<T> {
    private T value;
    public Holder() {}
    public Holder(T val) { value = val; }
    public void set(T val) { value = val; }
    public T get() { return value; }
    public boolean equals(Object obj) {
        return value.equals(obj);
    }
    public static void main(String[] args) {
        Holder<Apple> Apple = new Holder<Apple>(new Apple());
        Apple d = Apple.get();
        Apple.set(d);
        // Holder<Fruit> Fruit = Apple; // Cannot upcast
        Holder<? extends Fruit> fruit = Apple; // OK
        Fruit p = fruit.get();
        d = (Apple)fruit.get(); // Returns ‘Object’
        try {
       	 	Orange c = (Orange)fruit.get(); // No warning
        } catch(Exception e) { System.out.println(e); }
      	  // fruit.set(new Apple()); // Cannot call set()
       	 // fruit.set(new Fruit()); // Cannot call set()
    	System.out.println(fruit.equals(d)); // OK
    }
} /* Output: (Sample)
java.lang.ClassCastException: Apple cannot be cast to Orange
true
*///:~
```

​	在 Holer 类中，set() 方法接受类型参数 T 的对象作为参数，get() 返回一个 T 类型，而 equals() 接受一个 Object 作为参数。fruit 的类型是 Holder<? extends Fruit>，所以set()方法不会接受任何对象的添加，但是 equals() 可以正常工作。

## 下边界限

通配符的另一个方向是　“超类型的通配符“: ? super T，T 是类型参数的下界。使用这种形式的通配符，我们就可以 ”传递对象” 了。还是用例子解释：

```java
public class SuperTypeWildcards {
    static void writeTo(List<? super Apple> apples) {
        apples.add(new Apple());
        apples.add(new Jonathan());
        // apples.add(new Fruit()); // Error
    }
}
```

riteTo 方法的参数 apples 的类型是 List<? super Apple>，它表示某种类型的 List，这个类型是 Apple 的基类型。也就是说，我们不知道实际类型是什么，但是这个类型肯定是 Apple 的父类型。因此，我们可以知道向这个 List 添加一个 Apple 或者其子类型的对象是安全的，这些对象都可以向上转型为 Apple。但是我们不知道加入 Fruit 对象是否安全，因为那样会使得这个 List 添加跟 Apple 无关的类型。

在了解了子类型边界和超类型边界之后，我们就可以知道如何向泛型类型中 “写入” ( 传递对象给方法参数) 以及如何从泛型类型中 “读取” ( 从方法中返回对象 )。下面是一个例子：

```java
public class Collections {
    public static <T> void copy(List<? super T> dest, List<? extends T> src)
    {
        for (int i=0; i<src.size(); i++)
        dest.set(i,src.get(i));
    }
}
```

src 是原始数据的 List，因为要从这里面读取数据，所以用了上边界限定通配符：<? extends T>，取出的元素转型为 T。dest 是要写入的目标 List，所以用了下边界限定通配符：<? super T>，可以写入的元素类型是 T 及其子类型。

## 无边界通配符

有一种通配符是无边界通配符，它的使用形式是一个单独的问号：List<?>，也就是没有任何限定。不做任何限制，跟不用类型参数的 List 有什么区别呢？

List<?> list 表示 list 是持有某种特定类型的 List，但是不知道具体是哪种类型。那么我们可以向其中添加对象吗？当然不可以，因为并不知道实际是哪种类型，所以不能添加任何类型，这是不安全的。而单独的 List list ，也就是没有传入泛型参数，表示这个 list 持有的元素的类型是 Object，因此可以添加任何类型的对象，只不过编译器会有警告信息。

总结：

使用 List<? extends C>>list 这种形式，表示 list 可以引用一个 ArrayList ( 或者其它 List 的 子类 ) 的对象，这个对象包含的元素类型是 C 的子类型 ( 包含 C 本身）的一种。

使用 List<? super C> list 这种形式，表示 list 可以引用一个 ArrayList ( 或者其它 List 的 子类 ) 的对象，这个对象包含的元素就类型是 C 的超类型 ( 包含 C 本身 ) 的一种。

《java核心卷》

带有超类型限定的通配符可以向泛型对象写人，super。

带有子类型限定的通配符可以从泛型对象读取，extends。



# 泛型的继承规则

```java
public class Father {
    public void fatherPrint() {
    	System.out.println("I am father");
    }
}
```

```java
public class Son extends Father{
    public void sonPrint() {
    	System.out.println("I am son");
    }
}
```

```java
public class FatherSon {
    public static void main(String[] args) {
        Pair<Father> fPair=new Pair<Son>(new Son(), new Son());


        Pair<Son> fPair2=new Pair<Son>(new Son(), new Son());
    }
}
```

报错：

![](https://i.loli.net/2019/03/13/5c885f153851b.png)

Type mismatch: cannot convert from Pair Son to Pair Father

虽然泛型中类型T是继承关系，但是在泛型中这种写法是不允许的

实际上，无论类型变量T和S之间有什么关系，包括继承关系还是实现关系，Pair和Pair之间都不会有什么联系

下面这种继承是可以的

```java
class BasicPair<T> {


}


class Pair<T> extends BasicPair<T>{


    private T first;
    private T second;


    public Pair(T first, T second) {
        super();
        this.first = first;
        this.second = second;
    }


    public T getFirst() {
        return first;
    }


    public void setFirst(T first) {
        this.first = first;
    }


    public T getSecond() {
        return second;
    }


    public void setSecond(T second) {
        this.second = second;
    }
}
```



![](https://i.loli.net/2019/03/13/5c885f872d5da.png)

引用一句话泛型的继承只是在类这个维度上是可以的，他们的参数类型必须是一致的

参考链接：

<https://blog.csdn.net/l294265421/article/details/46413975>

<https://blog.csdn.net/csdn_of_coder/article/details/52563982>



# 泛型的约束与局限性



>泛型的一个原则：**如果一段代码在编译时没有提出“未经检查的转换”警告，则程序在运行时不会引发ClasscastException异常**



## 不能用基本类型实例化类型参数

​		因为泛型在编译时存在类型擦除，会将类型转换成原始类型，如果是原始类型，就无法转换，但是基本类型是有8种，我们可以使用他们的封装类代替



## 运行时类型查询只适合原始类型

因为泛型的擦除，使代码运行时用的是原始类型，而其他类型无法使用instanceof 上代码

```java
//泛型类
public class OriginTest<T> {
    private T id;
    private T name;
    private T salary;
    public T getId() {
        return id;
    }
    public void setId(T id) {
        this.id = id;
    }
    public T getName() {
        return name;
    }
    public void setName(T name) {
        this.name = name;
    }
    public T getSalary() {
        return salary;
    }
    public void setSalary(T salary) {
        this.salary = salary;
    }
    public OriginTest(T id, T name, T salary) {
        super();
        this.id = id;
        this.name = name;
        this.salary = salary;
    }
    public OriginTest() {
        super();
    }
}
```

```java
//测试
public class OriginTest1 {
    public static void main(String[] args) {
        OriginTest<String> test = new OriginTest<>();
        if(test instanceof OriginTest<String>) {
        	System.out.println(true);
        }else {
            System.out.println(false);
        }
    }
}
```

报错：

![](https://i.loli.net/2019/03/13/5c886062ce4d7.png)

Cannot perform instanceof check against parameterized type OriginTest. Use the form OriginTest<?> instead since further generic type information will be erased at runtime

无法对参数化类型OriginTest 执行instanceof检查。 请使用OriginTest <？>形式，因为在运行时将删除其他泛型类型信息



## 不能创建参数化类型的数组

同样也是擦除搞得鬼，

个人理解，new的时候实例化的是Object类型的，然而我们使用变量是String类型，我们不能直接用父类类型变量指向子类的对象。需要强制类型转换，所以这里是不允许的

![](https://i.loli.net/2019/03/13/5c8860c048e5b.png)

但是在想如果我们用的变量是Object类型的呢，实验的一般发现这样仍然不行，编译器不允许这样的存在。

![](https://i.loli.net/2019/03/13/5c8860f35b340.png)

报错：Cannot create a generic array of OriginTest 上面两个报错一样

那么我们用强制类型转换（其实就是向上转型，向下转型问题）：

声明通配类型的数组，然后后进行类型转换

```java
OriginTest<String>[] test1=(OriginTest<String>[]) new OriginTest<?>[10];
```

但是这里存在警告：Type safety: Unchecked cast from OriginTest<?>[] to OriginTest[]

这是不安全的，上截图

![](https://i.loli.net/2019/03/13/5c88613026afc.png)

对于这类问题，使用集合代替数组，解决不同类型的问题：

```java
new ArrayList<OriginTest<String>>();
```



## varargs警告(可变参数警告)

补充知识点：可变参数其实就是数组

所以当我们使用可变参数泛型的类时，会出现上面那种问题，这里规则放松，没有报错只是警告。但是在后面赋值的时候还是会出现异常，应该是类型转换异常



##  不能实例化类型变量

我们不知只有new T(…) new T[…]或者T.class这样的类型变量，原因就是编译器不知道T是什么类型。这里我有个疑问，既然T默认在没有指定类型的情况下，是编译成Object类型，这里为什么不行？？？

当我们想定义未知类型的 对象时，有两种方法：

![](https://i.loli.net/2019/03/13/5c8861b27ab2d.png)

第二种方法也可以表示为：

```java
Pair<String> t=Pair.makePair(String::new)
```

makePair 方法接收一个Supplier， 这是一个函数式接口， 表示一个无参数而且返回类型为T 的函数

![](https://i.loli.net/2019/03/13/5c8861f37db14.png)



## 不能构造泛型数组

数组是可以由父类指向子类实体的

```java
Father[] f= new Son[];
```

​	然而如果可以使用泛型数组，在编译时泛型数组会被擦除为object类型，意味着此时数组中可以存储任意类型的数据，让我们在使用指向该数组的变量类型时就会报错(可以理解为类型转换错误)，出现极大问题，因此，java不允许使用泛型数组。详细代码看参考链接

参考链接：

<http://www.blogjava.net/sean/archive/2005/08/09/9630.html>

<https://blog.csdn.net/x_iya/article/details/79550667>