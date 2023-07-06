[TOC]

# C# 高级语法特性（一）

打算持续更新一个专栏，旨在分享对C#这门编程语言的理解，巩固自己的C#知识体系以用于之后可能的Unity游戏开发（**所以我对C#的讨论会更多地基于它在Unity这个游戏引擎的实际应用**），并讨论一些C#区别于其它语言的特点。既然是针对于C#，自然不涉及太多各语言都包含的语法知识和面向对象基础，更多会讨论一些进阶知识，整理C#的高级语法特性，故该专栏可能不适用于编程新手（基础语法教程随处可得，我觉得也没有必要再多费口舌）。其中可能会总结一些其他书籍、课程中分享的经验，甚至部分搬运到专栏中来。对文章中可能出现的任何错误和纰漏，欢迎大家指出和纠正。

![img](https://article.biliimg.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png)

## Unity对C#版本的支持

- Unity 4.x 支持到 C# 2.0
- Unity 5.x 支持到 C# 3.0，
- Unity 2017.x 支持到 C# 6.0
- Unity 2018.x 到 C# 7.2

  C#和其它语言一样，自然也经历过版本的迭代，而编程语言的第一版一般都是核心部分，也是最常用的部分。

## C# 1.0

罗列一下1.0版本中的语法特性：

- 类（class）
- 结构（struct）
- 接口（interface）
- 属性（property）
- 事件（event）
- 委托（delegates）
- 表达式
- 语句
- 特性（属性）（Attribute）

### 类

  大概会涉及到面向对象编程语言中共有的一些话题如：访问权限、抽象类、内部类、继承多态等，就无需过多介绍了。提一下C#中一个特殊的关键字partial，其实也很简单，它可以实现类的逻辑拆分到不同的文件。

### 结构

  熟悉C或者C++的开发者可能会对结构体比较熟悉，C#中也有结构体这个概念，其中最关键的一点：**值类型**。

  在Unity开发中，我们要对一个游戏对象的位置进行改变时，不得不写成如下形式：

>   Vector3 pos = transform.position; 
>
>   pos.x = 2.0f; 
>
>   transform.position = pos;

  而不是

>   transform.position.x = 2.0f;

  在这里，无法直接对结构体中的单一变量赋值。总结C#结构体的一些关键点：

- 值类型
- 结构不能为null
- 声明变量时，本身就有值了
- 分配给新变量时，是深拷贝，并且对新副本所做的任何修改不会更改原始副本的数据
- 在为属性器时，不能局部赋值

### 接口

  接口需要提一点：显式实现和隐式实现。显式实现在Unity中其主要的应用场景是**防止误调用**，如：

>   public interface ITask {   
>
> ​    void Run();   
>
>   }
>
>   public class Task : ITask{
>
> ​    void ITask.Run(){
>
> ​      ...
>
> ​    }
>
>   }
>
>   public class TaskRunner{   
>
> ​    public static void RunTask(ITask task){     
>
> ​      task.Run();     
>
> ​    } 
>
>   }
>
>   void Start(){   
>
> ​    var task = new Task();
>
> ​    // task.Run(); 显式实现，无法直接调用
>
> ​    TaskRunner.RunTask(task); // 用专门的执行器去调用
>
>   }

  接口的显式实现降低了方法调用的权限，避免开发者误调用。

### 属性器

  属性器的优势在于隔离类内部的变化，也体现了OOP的**封装性**。如：

>   public class A{
>
> ​    public int a; 
>
>   }

  此时在另一处访问了此属性：

>   void Start(){
>
> ​    var ob = new A() { a = 1 } ;
>
> ​    Debug.Log(ob.a);
>
>   }

  当我们需要对属性a的计算方法进行改变时：

>   public class A{
>
> ​    public int a{
>
> ​      get{
>
> ​        return 1 - b ;
>
> ​      }
>
> ​    }
>
> ​    public int b;
>
>   }

  此时我们只需在声明处传入b的值即可，而无需再在访问属性a的地方对其进行任何更改。

### 事件

  C#的事件和委托很相似，回顾一下事件的用法：

>   public class EventArgs {   
>
> ​    public int Data { get; set; } 
>
>   }
>
>   public event Action<EventArgs> SomeEvent;        
>
>   void Start() {   
>
> ​    SomeEvent += OnSomeEvent;   
>
> ​    SomeEvent.Invoke(new EventArgs()   {     
>
> ​      Data = 10   
>
> ​    });         
>
> ​    SomeEvent -= OnSomeEvent;         
>
> ​    if (SomeEvent != null)   {     
>
> ​      SomeEvent.Invoke(new EventArgs()     {       
>
> ​        Data = 20     
>
> ​      });   
>
> ​    } 
>
>   }
>
>   void OnSomeEvent(EventArgs eventArgs) {   
>
> ​    Debug.Log(eventArgs.Data); 
>
>   }
>
>   // 输出结果为 10

  事件和委托关键的区别在于事件只能在所声明的类的内部调用，但在外部可以进行增减操作。

### 委托  

  除了上述所讨论的和事件的区别之外，还有一些需要注意的点：

- 注册委托和注销委托最好成对出现
- 委托有可能为 null，最好在声明时设置初始值，减少空指针异常的风险，如:

>   public Action<int> del = (arg) => {};

### Attribute

  Unity中常用的Attribute 有 **[Serializable] (被标记的类可以进行序列化和反序列化)**和 **[SerializeField] (让私有的成员变量在 Inspector 上进行显示)。**

- 自定义 Attribute 只需要继承 System.Attribute
- 自定义 Attribute 通常需要与反射配合

  关于反射，会在下一篇专栏中集中总结C#中与反射有关的值得关注的知识点。

### 反射

​	Java中也有反射机制，和Java语言不同，C#中的反射并不是语言层面上的，而是.Net进行提供的。

**Type对象**

​	可以通过typeof(类名)这个方法获取一个type对象，进而访问type对象的程序集、模块、类名等，例如：

    var type = typeof(ClassA);
    Debug.Log(type.Assembly);
    Debug.Log(type.Module);
    Debug.Log(type.Name);

​	或者通过`object.GetType()`或者到Type对象，根据Type对象获得到的信息类别大致可将Type对象的方法分为：

-   类信息查询方法：各种与类相关的名字，如：
    -   `type.FullName`
    -   `type.Namespace`
-   类结构查询方法：获取父类、方法、成员变量等，如：
    -   `type.BaseType`
    -   `MethodInfo info = type.GetMethod()`（包括父类方法和private方法）
    -   `FieldInfo info = type.GetField()`
    -   `PropertyInfo info = type.GetPropertie()`
    -   `MemberInfo info = type.GetMember()`
-   检测方法：判断是否为某一个事物，如：
    -   `type.IsClass`
    -   `type.IsAbstract`
    -   `type.IsValueType`

​	结合BindingFlags对type对象方法进行限制

C#中的反射用法几乎和Java中的反射机制一样，反射可以应用的场景很多：依赖注入容器、代码生成、序列化、热更新等。