[TOC]

# C# 高级语法特性（二）

​	1.0版本包含了C#语言最核心的部分，但仅有这些，会发现在实际开发中会遇到很多麻烦事，语言版本的迭代也是基于这些考虑在原有版本的基础上加入一些新特性，使得编程语言在处理一些问题上更加高效、方便、简洁和增强代码灵活性。

## C# 2.0

​	罗列一下2.0版本主要提供的特性：

-   泛型
-   partial
-   匿名方法
-   迭代器
-   协变和逆变
-   静态类
-   委托判断

### 泛型

​	泛型基本用法无需再赘述，与其它编程语言的泛型机制差别不大，泛型用的比较多的地方是底层、通用的模块。在设计一些框架或者库的底层时，对泛型类、泛型接口要求会比较高（包括**逆变和协变**），一般的开发其实无需过于深入理解泛型设计。

### 分部类型partial

​	微软官方文档指出partial关键字设计是用来把一个类的职责拆分给不同的开发者，而在Unity中，通常用于提供给UI类拆分出`View`和`Controller`的职责。例如拆分出`UI.cs`和`UI.Designer.cs`：

```c#
UI.cs(Controller的职责):
	
	public partial class UI{
		void Init(){}
        void Start(){}
        ...
    }

UI.Designer.cs(View的职责):
	
	public partial class UI{
        public Button Button;
        public Text Text;
        ...
    }
```

​	另外，在一个包含大量成员的静态类中使用partial，可以将它们拆分到不同的文件中，看起来可能会是：

```c#
StaticClass.XXX.cs:
	
	public static partial class StaticClass{
        public static X;
        ...
    }

StaticClass.YYY.cs:
	
	public static partial class StaticClass{
        public static Y;
        ...
    }
```

### 迭代器

​	多花一些篇幅讨论迭代器，基本形式看起来会是：

```c#
// IEnumerable接口定义
public interface IEnumerable{
    IEnumerator GetEnumerator();
}

// IEnumerator接口定义
public interface IEnumerator{
    bool MoveNext();
    object Current{get;}
    void Reset();
}

// 第一种形式
public class EnumClass : MonoBehaviour{
    void Start(){
		foreach(var v in EnumValue()){
            Debug.Log(v); // 将输出1 2 3
        }
    }
    IEnumerable EnumValue(){
        yield return 1;
        yield return 2;
        yield return 3;
    }
}

// 第二种形式
class EnumClass : IEnumerable
{
    public IEnumerator GetEnumerator()
    {
        yield return 1;
        yield return 2;
        yield return 3;
    }
}

void Start()
{
    foreach(var e in new EnumClass())
    {
       Debug.Log(e);
    }
    // foreach 同等实现
    var e = new EnumClass().GetEnumerator();
    while(e.MoveNext()){
        Debug.Log(e.Current);
    }
}
```

​	yield关键字本质是语法糖，其实是编译器自动实现IEnumerator等接口方法的关键字。

####  Coroutine

​		首先在此利用Unity中的Update()方法模拟Coroutine：

```c#
	...
	IEnumerator e; // 获取到某个IEnumerator
	void Update(){
        if(e.MoveNext()){
            ...
        }
    }
```

​		Update()方法将会在每一帧输出迭代器中的内容，再使用StartCoroutine()方法启动协程：

```c#
	StartCoroutine(e); // StartCoroutine方法会不停地调用MoveNext()方法直到结束
```

​		会发现两者效果一样，可以推断Unity也是通过枚举来运行程序块的。Coroutine则其实是一个迭代器模式+定时器的一种实现。

### 静态类

​	静态类稍微一提，关键的两点：

1.   不能被实例化
2.   静态构造函数在静态类成员被第一次访问时调用

2.0中其它新特性，如匿名方法、可空值类型、getter/setter单独可访问性、委托判断，目前我觉得没有什么需要说明的地方，一般的开发过程中也没有遇到什么相关问题。
