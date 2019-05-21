[TOC]

### 自动拆箱与装箱的原理

一段程序：

```java
public static void main(String[] args) {

    Integer i = 10;
    int n = i;

    Byte b = 'a';
    byte nb = b;
}
```

其字节码如下：

```
 0 bipush 10
 2 invokestatic #2 <java/lang/Integer.valueOf>
 5 astore_1
 6 aload_1
 7 invokevirtual #3 <java/lang/Integer.intValue>
10 istore_2
11 bipush 97
13 invokestatic #4 <java/lang/Byte.valueOf>
16 astore_3
17 aload_3
18 invokevirtual #5 <java/lang/Byte.byteValue>
21 istore 4
23 return
```

可看出：

- 自动装箱处，全部翻译成XXX.valueOf()方法调用
- 自动拆箱处，全部翻译成XXX.xxxValue()方法调用

### 一些需要注意的点

#### Integer等类的缓存机制

```java
public static void main(String[] args) {
         
   Integer i1 = 100;
   Integer i2 = 100;
   Integer i3 = 200;
   Integer i4 = 200;
         
   System.out.println(i1==i2); //true
   System.out.println(i3==i4); //false
}
```

在Integer.valueOf()方法中：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high) // [-128,127]
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

当值在[-128,127]之间时，首次调用会生成一个缓存，第二次调用时返回的同一个对象；不在[-128,127]区间时，每次都返回新对象

Integer、Short、Byte、Character、Long这几个类的valueOf方法都具有类似缓存机制

而Double、Float的valueOf方法的不具有缓存机制，即每次调用都返回新对象

#### Boolean类的特殊

```java
public static void main(String[] args) {
         
   Boolean i1 = false;
   Boolean i2 = false;
   Boolean i3 = true;
   Boolean i4 = true;
         
   System.out.println(i1==i2); // true
   System.out.println(i3==i4); // true
}
```

其valueOf方法：

```java
public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
}
```

其中TRUE和FALSE都是Boolean类中的静态单例对象，不管怎么调用都是同一个对象

#### 手动构建和自动装箱

`Integer i = new Integer(xxx);` 和 `Integer i =xxx;`

- 第一种方式不会触发自动装箱的过程；而第二种方式会触发
- 第二种方式的执行效率和资源占用在一般性情况下要优于第一种情况（但不是绝对）

#### == 和 equals

```java
public static void main(String[] args) {
        Integer a = 1;
        Integer b = 2;
        Integer c = 3;
        Integer d = 3;
        Integer e = 321;
        Integer f = 321;
        Long g = 3L;
        Long h = 2L;

        System.out.println(c==d);  // true , Integer的缓存机制
        System.out.println(e==f);  // false, 比较的是对象不是值 
        System.out.println(c==(a+b)); // true, 全都触发自动拆箱，比较值
        System.out.println(c.equals(a+b)); // true, 仅a,b触发自动拆箱和装箱，Integer和Integer比较，相等
        System.out.println(g==(a+b)); // true, 全都触发自动拆箱，比较值
        System.out.println(g.equals(a+b)); // false, 仅a,b触发自动拆箱和装箱，Long和Integer类型不相等
        System.out.println(g.equals(a+h)); // true, 仅a+h触发自动拆箱和装箱，其结果为Long类型，与Long类型比较，相等
}
```

“==”运算在没有遇到算数运算时不会自动拆箱，equals()方法不会处理数据转型的关系（值的表达式内会进行强制转换）