# 一、关键字

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#final)final

**1\. 数据**

声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

* 对于基本类型，final 使数值不变；
* 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。

```source-java
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```

**2\. 方法** 

声明方法不能被子类覆盖。

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是覆盖基类方法，而是在子类中定义了一个新的方法。

**3\. 类**

声明类不允许被继承。

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#static)static

**1\. 静态变量**

静态变量在内存中只存在一份，只在类第一次实例化时初始化一次。

* 静态变量：类所有的实例都共享静态变量，可以直接通过类名来访问它；
* 实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死。

```source-java
public class A {
    private int x;        // 实例变量
    public static int y;  // 静态变量
}
```

**2\. 静态方法**

静态方法在类加载的时候就存在了，它不依赖于任何实例，所以 static 方法必须实现，也就是说它不能是抽象方法（abstract）。

**3\. 静态语句块**

静态语句块和静态变量一样在类第一次实例化时运行一次。

**4\. 初始化顺序**

静态数据优先于其它数据的初始化，静态变量和静态语句块哪个先运行取决于它们在代码中的顺序。

```source-java
public static String staticField = "静态变量";
```

```source-java
static {
    System.out.println("静态语句块");
}
```

实例变量和普通语句块的初始化在静态变量和静态语句块初始化结束之后。

```source-java
public String field = "实例变量";
```

```source-java
{
    System.out.println("普通语句块");
}
```

最后才是构造函数中的数据进行初始化

```source-java
public InitialOrderTest() {
    System.out.println("构造函数");
}
```

存在继承的情况下，初始化顺序为：

1. 父类（静态变量、静态语句块块）
2. 子类（静态变量、静态语句块）
3. 父类（实例变量、普通语句块）
4. 父类（构造函数）
5. 子类（实例变量、普通语句块）
6. 子类（构造函数）

**5\. 静态内部类**

内部类的一种，静态内部类不依赖外部类，且不能访问外部类的非 static 变量和方法。

**6\. 静态导包**

```
import static com.xxx.ClassName.*

```

在使用静态变量和方法时不用再指明 ClassName，从而简化代码，但可读性大大降低。

# [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E4%BA%8Cobject-%E9%80%9A%E7%94%A8%E6%96%B9%E6%B3%95)二、Object 通用方法

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E6%A6%82%E8%A7%88)概览

```source-java
public final native Class<?> getClass()

public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException

protected void finalize() throws Throwable {}
```

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#equals)equals()

**1\. equals() 与 == 的区别**

* 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
* 对于引用类型，== 判断两个实例是否引用同一个对象，而 equals() 判断引用的对象是否等价。

```source-java
Integer x = new Integer(1);
Integer y = new Integer(1);
System.out.println(x.equals(y)); // true
System.out.println(x == y);      // false
```

**2\. 等价关系**

（一）自反性

```source-java
x.equals(x); // true
```

（二）对称性

```source-java
x.equals(y) == y.equals(x) // true
```

（三）传递性

```source-java
if(x.equals(y) && y.equals(z)) {
    x.equals(z); // true;
}
```

（四）一致性

多次调用 equals() 方法结果不变

```source-java
x.equals(y) == x.equals(y); // true
```

（五）与 null 的比较

对任何不是 null 的对象 x 调用 x.equals(null) 结果都为 false

```source-java
x.euqals(null); // false;
```

**3\. 实现**

* 检查是否为同一个对象的引用，如果是直接返回 true；
* 检查是否是同一个类型，如果不是，直接返回 false；
* 将 Object 实例进行转型；
* 判断每个关键域是否相等。

```source-java
public class EqualExample {
    private int x;
    private int y;
    private int z;

    public EqualExample(int x, int y, int z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        EqualExample that = (EqualExample) o;

        if (x != that.x) return false;
        if (y != that.y) return false;
        return z == that.z;
    }
}
```

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#hashcode)hashCode()

hasCode() 返回散列值，而 equals() 是用来判断两个实例是否等价。等价的两个实例散列值一定要相同，但是散列值相同的两个实例不一定等价。

在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证相等的两个实例散列值也等价。

下面的代码中，新建了两个等价的实例，并将它们添加到 HashSet 中。我们希望将这两个实例当成一样的，只在集合中添加一个实例，但是因为 EqualExample 没有实现 hasCode() 方法，因此这两个实例的散列值是不同的，最终导致集合添加了两个等价的实例。

```source-java
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size()); // 2
```

理想的散列函数应当具有均匀性，即不相等的实例应当均匀分不到所有可能的散列值上。这就要求了散列函数要把所有域的值都考虑进来，可以将每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。R 一般取 31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位。

一个数与 31 相乘可以转换成移位和减法：31*x == (x [深入理解 abstract class 和 interface](https://www.ibm.com/developerworks/cn/java/l-javainterface-abstract/) 
> [When to Use Abstract Class and Interface](https://dzone.com/articles/when-to-use-abstract-class-and-intreface)

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#super)super

* 访问父类的构造函数：可以使用 super() 函数访问父类的构造函数，从而完成一些初始化的工作。
* 访问父类的成员：如果子类覆盖了父类的中某个方法的实现，可以通过使用 super 关键字来引用父类的方法实现。

```source-java
public class SuperExample {
    protected int x;
    protected int y;

    public SuperExample(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public void func() {
        System.out.println("SuperExample.func()");
    }
}
```

```source-java
public class SuperExtendExample extends SuperExample {
    private int z;

    public SuperExtendExample(int x, int y, int z) {
        super(x, y);
        this.z = z;
    }

    @Override
    public void func() {
        super.func();
        System.out.println("SuperExtendExample.func()");
    }
}
```

```source-java
SuperExample e = new SuperExtendExample(1, 2, 3);
e.func();
```

```text-html-basic
SuperExample.func()
SuperExtendExample.func()
```

> [Using the Keyword super](https://docs.oracle.com/javase/tutorial/java/IandI/super.html)

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E8%A6%86%E7%9B%96%E4%B8%8E%E9%87%8D%E8%BD%BD)覆盖与重载

* 覆盖（Override）存在于继承体系中，指子类实现了一个与父类在方法声明上完全相同的一个方法；

* 重载（Overload）存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是参数类型、个数、顺序至少有一个不同。应该注意的是，返回值不同，其它都相同不算是重载。

# [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E4%BA%94string)五、String

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#string-stringbuffer-and-stringbuilder)String, StringBuffer and StringBuilder

**1\. 是否可变**

* String 不可变
* StringBuffer 和 StringBuilder 可变

**2\. 是否线程安全**

* String 不可变，因此是线程安全的
* StringBuilder 不是线程安全的
* StringBuffer 是线程安全的，内部使用 synchronized 来同步

> [String, StringBuffer, and StringBuilder](https://stackoverflow.com/questions/2971315/string-stringbuffer-and-stringbuilder)

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#string-%E4%B8%8D%E5%8F%AF%E5%8F%98%E7%9A%84%E5%8E%9F%E5%9B%A0)String 不可变的原因

**1\. 可以缓存 hash 值**

因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

**2\. String Pool 的需要**

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

[![](/img/f76067a5-7d5f-4135-9549-8199c77d8f1c.jpg)](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/pics/f76067a5-7d5f-4135-9549-8199c77d8f1c.jpg)

**3\. 安全性**

String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

**4\. 线程安全**

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

> [Why String is immutable in Java?](https://www.programcreek.com/2013/04/why-string-is-immutable-in-java/)

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#stringintern)String.intern()

使用 String.intern() 可以保证相同内容的字符串实例引用相同的内存对象。

下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同对象，而 s3 是通过 s1.intern() 方法取得一个对象引用，这个方法首先把 s1 引用的对象放到 String Poll（字符串常量池）中，然后返回这个对象引用。因此 s3 和 s1 引用的是同一个字符串常量池的对象。

```source-java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
System.out.println(s1.intern() == s3);  // true
```

如果是采用 "bbb" 这种使用双引号的形式创建字符串实例，会自动地将新建的对象放入 String Poll 中。

```source-java
String s4 = "bbb";
String s5 = "bbb";
System.out.println(s4 == s5);  // true
```

Java 虚拟机将堆划分成新生代、老年代和永久代（PermGen Space）。在 Java 7 之前，字符串常量池被放在永久代中，而在 Java 7，它被放在堆的其它位置。这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。

> [What is String interning?](https://stackoverflow.com/questions/10578984/what-is-string-interning) 
> [深入解析 String#intern](https://tech.meituan.com/in_depth_understanding_string_intern.html)

# [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E5%85%AD%E5%9F%BA%E6%9C%AC%E7%B1%BB%E5%9E%8B%E4%B8%8E%E8%BF%90%E7%AE%97)六、基本类型与运算

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E5%8C%85%E8%A3%85%E7%B1%BB%E5%9E%8B)包装类型

八个基本类型：

* boolean/1
* byte/8
* char/16
* short/16
* int/32
* float/32
* long/64
* double/64

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。

```source-java
Integer x = 2;     // 装箱
int y = x;         // 拆箱
```

new Integer(123) 与 Integer.valueOf(123) 的区别在于，new Integer(123) 每次都会新建一个对象，而 Integer.valueOf(123) 可能会使用缓存对象，因此多次使用 Integer.valueOf(123) 会取得同一个对象的引用。

```source-java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);   // true
```

编译器会在自动装箱过程调用 valueOf() 方法，因此多个 Integer 实例使用自动装箱来创建并且值相同，那么就会引用相同的对象。

```source-java
Integer m = 123;
Integer n = 123;
System.out.println(m == n); // true
```

valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接使用缓存池的内容。

```source-java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

在 Java 8 中，Integer 缓存池的大小默认为 -128~127。

```source-java
static final int low = -128;
static final int high;
static final Integer cache[];

static {
    // high value may be configured by property
    int h = 127;
    String integerCacheHighPropValue =
        sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    if (integerCacheHighPropValue != null) {
        try {
            int i = parseInt(integerCacheHighPropValue);
            i = Math.max(i, 127);
            // Maximum array size is Integer.MAX_VALUE
            h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        } catch( NumberFormatException nfe) {
            // If the property cannot be parsed into an int, ignore it.
        }
    }
    high = h;

    cache = new Integer[(high - low) + 1];
    int j = low;
    for(int k = 0; k < cache.length; k++)
        cache[k] = new Integer(j++);

    // range [-128, 127] must be interned (JLS7 5.1.7)
    assert IntegerCache.high >= 127;
}
```

Java 还将一些其它基本类型的值放在缓冲池中，包含以下这些：

* boolean values true and false
* all byte values
* short values between -128 and 127
* int values between -128 and 127
* char in the range \u0000 to \u007F

因此在使用这些基本类型对应的包装类型时，就可以直接使用缓冲池中的对象。

> [Differences between new Integer(123), Integer.valueOf(123) and just 123](https://stackoverflow.com/questions/9030817/differences-between-new-integer123-integer-valueof123-and-just-123)

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#switch)switch

从 Java 7 开始，可以在 switch 条件判断语句中使用 String 对象。

```source-java
String s = "a";
switch (s) {
    case "a":
        System.out.println("aaa");
        break;
    case "b":
        System.out.println("bbb");
        break;
}
```

switch 不支持 long，是因为 swicth 的设计初衷是为那些只需要对少数的几个值进行等值判断，如果值过于复杂，那么还是用 if 比较合适。

```source-java
//        long x = 111;
//        switch (x) { // Incompatible types. Found: 'long', required: 'char, byte, short, int, Character, Byte, Short, Integer, String, or an enum'
//            case 111:
//                System.out.println(111);
//                break;
//            case 222:
//                System.out.println(222);
//                break;
//        }
```

> [Why can't your switch statement data type be long, Java?](https://stackoverflow.com/questions/2676210/why-cant-your-switch-statement-data-type-be-long-java)

# [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E4%B8%83%E5%8F%8D%E5%B0%84)七、反射

每个类都有一个 **Class** 对象，包含了与类有关的信息。当编译一个新类时，会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。

类加载相当于 Class 对象的加载。类在第一次使用时才动态加载到 JVM 中，可以使用 Class.forName("com.mysql.jdbc.Driver") 这种方式来控制类的加载，该方法会返回一个 Class 对象。

反射可以提供运行时的类信息，并且这个类可以在运行时才加载进来，甚至在编译时期该类的 .class 不存在也可以加载进来。

Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：

1. **Field** ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
2. **Method** ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
3. **Constructor** ：可以用 Constructor 创建新的对象。

IDE 使用反射机制获取类的信息，在使用一个类的对象时，能够把类的字段、方法和构造函数等信息列出来供用户选择。

**Advantages of Using Reflection:**

* **Extensibility Features** : An application may make use of external, user-defined classes by creating instances of extensibility objects using their fully-qualified names.
* **Class Browsers and Visual Development Environments** : A class browser needs to be able to enumerate the members of classes. Visual development environments can benefit from making use of type information available in reflection to aid the developer in writing correct code.
* **Debuggers and Test Tools** : Debuggers need to be able to examine private members on classes. Test harnesses can make use of reflection to systematically call a discoverable set APIs defined on a class, to insure a high level of code coverage in a test suite.

**Drawbacks of Reflection:**

Reflection is powerful, but should not be used indiscriminately. If it is possible to perform an operation without using reflection, then it is preferable to avoid using it. The following concerns should be kept in mind when accessing code via reflection.

* **Performance Overhead** : Because reflection involves types that are dynamically resolved, certain Java virtual machine optimizations can not be performed. Consequently, reflective operations have slower performance than their non-reflective counterparts, and should be avoided in sections of code which are called frequently in performance-sensitive applications.
* **Security Restrictions** : Reflection requires a runtime permission which may not be present when running under a security manager. This is in an important consideration for code which has to run in a restricted security context, such as in an Applet.
* **Exposure of Internals** :Since reflection allows code to perform operations that would be illegal in non-reflective code, such as accessing private fields and methods, the use of reflection can result in unexpected side-effects, which may render code dysfunctional and may destroy portability. Reflective code breaks abstractions and therefore may change behavior with upgrades of the platform.

> [Trail: The Reflection API](https://docs.oracle.com/javase/tutorial/reflect/index.html) 
> [深入解析 Java 反射（1）- 基础](http://www.sczyh30.com/posts/Java/java-reflection-1/)

# [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E5%85%AB%E5%BC%82%E5%B8%B8)八、异常

Throwable 可以用来表示任何可以作为异常抛出的类，分为两种： **Error** 和 **Exception**。其中 Error 用来表示 JVM 无法处理的错误，Exception 分为两种：

1. **受检异常** ：需要用 try...catch... 语句捕获并进行处理，并且可以从异常中恢复；
2. **非受检异常** ：是程序运行时错误，例如除 0 会引发 Arithmetic Exception，此时程序奔溃并且无法恢复。

[![](/img/PPjwP.png)](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/pics/PPjwP.png)

> [Java 入门之异常处理](https://www.tianmaying.com/tutorial/Java-Exception) 
> [Java 异常的面试问题及答案 -Part 1](http://www.importnew.com/7383.html)

# [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E4%B9%9D%E6%B3%9B%E5%9E%8B)九、泛型

```source-java
public class Box<T> {
    // T stands for "Type"
    private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

> [Java 泛型详解](https://www.ziwenxie.site/2017/03/01/java-generic/)
> [10 道 Java 泛型面试题](https://cloud.tencent.com/developer/article/1033693)

# [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E5%8D%81%E6%B3%A8%E8%A7%A3)十、注解

Java 注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。

> [注解 Annotation 实现原理与自定义注解例子](https://www.cnblogs.com/acm-bingzi/p/javaAnnotation.html)

# [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E5%8D%81%E4%B8%80%E7%89%B9%E6%80%A7)十一、特性

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E4%B8%89%E5%A4%A7%E7%89%B9%E6%80%A7)面向对象三大特性

> [封装、继承、多态](https://github.com/CyC2018/Interview-Notebook/blob/master/notes/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E6%80%9D%E6%83%B3.md#%E5%B0%81%E8%A3%85%E7%BB%A7%E6%89%BF%E5%A4%9A%E6%80%81)

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#java-%E5%90%84%E7%89%88%E6%9C%AC%E7%9A%84%E6%96%B0%E7%89%B9%E6%80%A7)Java 各版本的新特性

**New highlights in Java SE 8**

1. Lambda Expressions
2. Pipelines and Streams
3. Date and Time API
4. Default Methods
5. Type Annotations
6. Nashhorn JavaScript Engine
7. Concurrent Accumulators
8. Parallel operations
9. PermGen Error Removed

**New highlights in Java SE 7**

1. Strings in Switch Statement
2. Type Inference for Generic Instance Creation
3. Multiple Exception Handling
4. Support for Dynamic Languages
5. Try with Resources
6. Java nio Package
7. Binary Literals, Underscore in literals
8. Diamond Syntax

> [Difference between Java 1.8 and Java 1.7?](http://www.selfgrowth.com/articles/difference-between-java-18-and-java-17)
> [Java 8 特性](http://www.importnew.com/19345.html)

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#java-%E4%B8%8E-c-%E7%9A%84%E5%8C%BA%E5%88%AB)Java 与 C++ 的区别

Java 是纯粹的面向对象语言，所有的对象都继承自 java.lang.Object，C++ 为了兼容 C 即支持面向对象也支持面向过程。

| Java | C++ |
| --- | --- |
| Java does not support pointers, templates, unions, operator overloading, structures etc. The Java language promoters initially said "No pointers!", but when many programmers questioned how you can work without pointers, the promoters began saying "Restricted pointers." Java supports what it calls "references". References act a lot like pointers in C++ languages but you cannot perform arithmetic on pointers in Java. References have types, and they're type-safe. These references cannot be interpreted as raw address and unsafe conversion is not allowed. | C++ supports structures, unions, templates, operator overloading, pointers and pointer arithmetic. |
| Java support automatic garbage collection. It does not support destructors as C++ does. | C++ support destructors, which is automatically invoked when the object is destroyed. |
| Java does not support conditional compilation and inclusion. | Conditional inclusion (#ifdef #ifndef type) is one of the main features of C++. |
| Java has built in support for threads. In Java, there is a `Thread` class that you inherit to create a new thread and override the `run()` method. | C++ has no built in support for threads. C++ relies on non-standard third-party libraries for thread support. |
| Java does not support default arguments. There is no scope resolution operator (::) in Java. The method definitions must always occur within a class, so there is no need for scope resolution there either. | C++ supports default arguments. C++ has scope resolution operator (::) which is used to to define a method outside a class and to access a global variable within from the scope where a local variable also exists with the same name. |
| There is no *goto* statement in Java. The keywords `const` and `goto` are reserved, even though they are not used. | C++ has *goto* statement. However, it is not considered good practice to use of *goto*statement. |
| Java doesn't provide multiple inheritance, at least not in the same sense that C++ does. | C++ does support multiple inheritance. The keyword `virtual` is used to resolve ambiguities during multiple inheritance if there is any. |
| Exception handling in Java is different because there are no destructors. Also, in Java, try/catch must be defined if the function declares that it may throw an exception. | While in C++, you may not include the try/catch even if the function throws an exception. |
| Java has method overloading, but no operator overloading. The `String` class does use the `+` and `+=` operators to concatenate strings and `String`expressions use automatic type conversion, but that's a special built-in case. | C++ supports both method overloading and operator overloading. |
| Java has built-in support for documentation comments (`/** ... */`); therefore, Java source files can contain their own documentation, which is read by a separate tool usually `javadoc` and reformatted into HTML. This helps keeping documentation maintained in easy way. | C++ does not support documentation comments. |
| Java is interpreted for the most part and hence platform independent. | C++ generates object code and the same code may not run on different platforms. |

> [What are the main differences between Java and C++?](http://cs-fundamentals.com/tech-interview/java/differences-between-java-and-cpp.php)

## [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#jre-or-jdk)JRE or JDK

* JRE is the JVM program, Java application need to run on JRE.
* JDK is a superset of JRE, JRE + tools for developing java programs. e.g, it provides the compiler "javac"

# [](https://github.com/CyC2018/Interview-Notebook/blob/2512b291d3e7129de5468010f5b4d3b1fda85c03/notes/Java%20%E5%9F%BA%E7%A1%80.md#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99)参考资料

* Eckel B. Java 编程思想[M]. 机械工业出版社, 2002.
* Bloch J. Effective java[M]. Addison-Wesley Professional, 2017.

