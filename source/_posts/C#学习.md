---
title= C#学习笔记
---

# Type（类型）
## 类型的简介
小内存容纳大尺寸数据会丢失数据和精度...
大内存...小尺寸数据浪费内存
整数类型：int 4字节；long 8字节。
强类型语言：数据类型不能轻易转变；弱类型语言...
程序静止时，放在硬盘（外村）内，运行时，装载到内存。
类型中的信息：
	内存空间大小，
	储存值的大小范围，
	基类（父类），
	成员（事件，属性，方法）
	类型的变量分配在内存的位置
		Stack(栈，比较小)/Heap(堆，更大)
		Stack overflow栈溢出，堆不会溢出。
		堆上的内存如果忘记回收，会造成内存泄漏。
		cmd中调用performance monitor可以监测进程的内存。
		实例，分配到堆（heap）中去。
## 类型系统
### 五大数据类型 
类（class）：Window,Form,Console
结构体（structure）:Int,double
枚举（enumeration）
接口（interface）
委托（delegate） 

## 变量、对象与内存
### 变量
变量（名）表示了变量的值存储位置（从这个位置往后数多少节，用以保存这个变量，以其数据类型的要求），并且每个变量都有一个类型，决定什么样的值，能存入变量。
例如，int x =100;
类型：静态变量，实例变量（字段，成员变量），数组元素，值参数，引用参数(ref)，输出形参(out) ，局部变量。
狭义的变量，指局部变量（其他种类变量都有自己的约定名称，例如，静态变量称为静态字段）
局部变量：方法体，函数体内声明的变量。

### 值类型的变量
byte/short/ushort等
值类型的储存：在内存中分配空间存储值。
值类型没有实例，实例与变量合而为一了。

### 引用类型的变量
引用类型的储存：储存的是对象的地址。
例如：
Student stu;(分配四个字节的内存，用来指向实例化对象的地址)
stu =new Student();（开辟一段内存空间，用于储存Student类中的成员）

Class Student()
{
  unit ID;
  string name;
}

### 装箱/拆箱
装箱（Boxing）
int x=100;
Object obj =x;
将栈上的数据搬到堆上，然后将堆上存储这段数据的地址储存到栈上的obj上。
拆箱（Unboxing）
相反的操作。
int y =(int)obj;

# Method(方法)
## 由来
由C++和C语言中的function（函数）演变而来。
当函数成为类的成员之后，便变为了方法（或成员函数）。方法永远是类（class）的成员。 
## 声明与调用
C#声明与定义不分家。
声明结构：
函数头   函数体
函数头：特性  修饰符  （partial）  返回值   函数名  （）
	修饰符：有的可以组合
静态方法，隶属于类，而非声明的对象。不用实例化即可以调用。
例如：
Calculator c = new Calculator();
c.method1();
Calculator.method_2();

Class Calculator
{
 public int method()
 {
 return 0;
 }
 public static method_2()
 {
 return;
 } 
}

parameter：形参
argument：实参 

## 构造函数（构造器/constructor）
## 方法的重载（Overload）
## 方法的debug
设置断点（breakpoint）
观察方法调用时的call stack
Step in/over/out：
step in步进。
step over直接略过跳过断点的细节操作，执行下一语句。
step out跳到断点的上一层级。

## 方法的调用与栈
stack frame:方法被调用的时候，在栈当中的布局。

# 操作符（Operator）
## 概述
操作符有先后顺序，层级高的先运算，相同层级的从左往右依次运算。
赋值和lambda表达式，是从右向左运算（赋值），例如x = y;
操作符本质是函数的简记，例如3x5可以写为：Mul(3,5);
default操作符：获取当前类型的默认值。
new操作符：创造实例。 以及创造匿名类型的实例，然后用隐式类型变量引用这个实例。
new修饰符：在重写父类方法的时候，可以隐藏父类方法。
check操作符：检测程序是否溢出，若溢出则抛出异常。
unchecked 操作符：用于禁止溢出检查。它告诉编译器在执行某些可能导致溢出的数学运算时，不要抛出 OverflowException 异常，而是允许溢出发生，并对溢出的结果进行截断或回绕。
try-catch:当程序执行到 try 块中的代码时，如果发生了异常（错误），程序会立即跳转到 catch 块，而不会继续执行后续的代码。
sizeof操作符：获取结构体数据类型实例，在内存当中的字节数。
~操作符：在二进制上进行按位取反。对最小值用负号（-）来取反，得不到最大值，二进制编码取反原理造成的原因。

# 字段、属性、常量、索引器
## 字段（field）
表示与对象（类型）关联的变量，是类型的成员，旧称成员变量 。
实例字段：与对象相关联。在对象创建时才被初始化。
静态字段：与类型相关联，用static修饰。在类型被加载时才被初始化。
## 属性（property）
是用于访问对象或者类型的特征的成员，特征反映了状态。
属性是由字段发展而来，是字段的一种拓展。
字段：
```C#
//旧的写法
public int Age;
//get/set的写法
private int age;
public int GetAge()//获取字段的值
{
 return this.age;
}
public int SetAge(int value)//设置字段的值
{
 if(value>=0&&value<=120)
 {
  this.age=value;
 }
 ...
} 
```
属性现在的声明方法：
```C#
private int age;
private int Age
{
 get
 {
 return this.age;}
 }
 set
 {
  if(value>=0&&value<=120)
  {
   this.age=value;
  }
 }
}
```
value在get/set的语法中，是专属的关键字（上下文关键字），表示传进来的数值。
属性，也是一种语法糖，是编译器简化了编译过程。
prop/propfull：属性声明的简写
## 索引器
...

## 常量（constant）
关键字：const;
隶属于，类型而非对象。 
局部常量，成员常量

## 各种“只读”
常量，只读字段，只读属性，静态只读字段。

# 参数
## 值参数
不带任何修饰符的参数。
## 引用参数
用ref修饰符声明的形参。不创建新的储存位置，引用参数所表示的储存位置是调用中作为实参给出变量所表示的储存位置。
变量在座位引用参数之前，必须明确赋值。
引用修饰符作用，强调该方法会改变传入的变量的值。
（难理解，比较绕）
## 输出形参
用out修饰符声明的形参，输出形参不创建新的储存位置，表示的储存位置恰是该方法调用中作为实参给出的那个变量所表示的储存位置。
会产生除return之外的额外返回值。
PS：params关键字
int res = Cal(1,2,3);
static int Cal(params int[] intArray)
{
 ...
}
params会自动声明，只需要输入数值即可。

## 具名调用
Printinfo(age:34,name:"Tim");
Static void Printinfo(string name,int age)
{
 ...
}
## 可选参数
Printinfo();
Static void Printinfo(string name="Tim",int age=34)

## 扩展方法（This)
扩展方法必须是public static
方法中的第一个参数必须被this修饰
必须由静态类（static）进行收纳

x作为Round方法的第一个参数（double intput），因此仅需要输入第二个参数即可。
在double intput前加上this之后，所有double类型的值均可以使用这个方法。
`double x =3.14159;
double y =x.Round(4);
static class DoubleExtension
{
 public static double Round(this double input,int digits)
{
 ...
}
}`

LINQ方法：

## 委托（delegate）
是（C/C++）函数指针的“升级版”、
变量（数据）是以某个地址为起点的一段内存中所存储的值；
变量，是寻找数据的地址。
函数（算法）是以某个地址为起点的一段内存中所存储的一组机器语言指令；
函数，是寻找算法的地址。
直接调用与间接调用：
直接调用：通过函数名来调用函数，CPU通过函数名获取函数所在地址。
间接调用：通过函数指针调用函数，CPU通过读取函数指针存储的值，获取函数所在地址。
### Action委托实例
Action委托：无返回值的委托类型。

```c#
static void Main(string[] args]
{
 Calculator calculator = new Calculator();
 Action action = new Action(calculator.Report);
 //calculator.Report不需要加()，因为只是需要方法名的地址，而不是需要调用它。
 calculator.Report();
 action.Invoke();
 action();
}
class Calculator
{
 public void Report()
 {
  Console. WriteLine("I have 3 methods.");
 }
}
```


### Func委托实例
有返回值的委托


### 委托的声明
委托是一种类（Class），但是声明方式和一般的类不同。
注意声明委托的位置：命名空间下，类/主函数之外，避免写错地方形成嵌套类型

```C#
namespace DelegateExample
{
 public delegate double Calc(double x, double y);
 class Program
 {
  static void Main(string[] args)
 {
   Calculator calculator = new Calculator();
   Calc calc1 = new Calc(calculator.Add); 
   Calc calc2 = new Calc(calculator.Sub);
   Calc calc3 = new Calc(calculator. Mul);
   Calc calc4 = new Calc(calculator.Div);
   double a = 100;
   double b = 200;
   double c = 0;
   c = calc1.Invokola, b);
   Console. WriteLine[c];
   c = calc2.Invoke(a, b);
   Console. WriteLine(c);
   c = calc3.Invoke(a, b);
   Console. WriteLine(c);
   c = calc4.Invoke(a, b);
   Console. WriteLine(c);
 }

 }
 class Calculator
 {
  ...
 }
}
```



### 委托的使用
1.把方法当作参数传给另外一个方法。
模板方法，“借用”指定的外部方法，来产生结果
相当于“填空题”，委托也有返回值，除了委托部分不确定，其他部分都是确定的。
2.回调（callback）方法，调用指定的外部方法
相当于“流水线”，委托不需要有返回值。你可以从许多方法中选择自己需要的委托方法。

### 委托使用的案例
```C#
namespace DelegateExample
{
 class Program
 static void Main(string[] args]
 {
   ProductFactory productFactory = new ProductFactory[);
   WrapFactory wrapFactory = new WrapFactory[];
   Func<Product> func1 = new Func<Product>(productFactory.MakePizza);
   Func<Product> func2 = new Func<Product>(productFactory.MakeToyCar);
   Logger logger =new Logger();
   Action<Product> log = new Action<Product>(logger.Log);
   Box box1 = wrapFactory.WrapProduct(func1,log);
   Box box2 = wrapFactory.WrapProduct(func2,log);
   Console. WriteLine(box1.Product.Name];
   Console. WriteLine(box2.Product.Name];
 }
 class Product
 {
  public string Name{get; set;}
 }
 class Box
 {
  public Product Product{get;set;}
 }
 class WrapFactory
 {
  public Box WrapProduct(Func<Product> getProduct)
  {
   Box box=new Box():
   Product product=getProduct.Invoke();
   box.Product=product;
   return box;
  }
 }
 
class ProductFactory
{
 public Product MakePizza()
 {
  Product product=new Product[];
  product.Name= "Pizza":
  return product;
 }
 public Product MakeToyCar()
 {
  Product product=new Product[];
  product.Name = "Toy Car";
  return product;
 }
} 
}
```



### 回调Callback方法案例
```c#
namespace DelegateExample
{
 class Program
 static void Main(string[] args]
 {
  ProductFactory productFactory = new ProductFactory[);
  WrapFactory wrapFactory = new WrapFactory[];
  Func<Product> func1 = new Func<Product>(productFactory.MalkePizza)
  Func<Product> func2 = new Func<Product>(productFactory.MaakeToyCar)
  Box box1 = wrapFactory.WrapProduct(func1);
  Box box2 = wrapFactory.WrapProduct(func2);
  Console. WriteLine(box1.Product.Name];
  Console. WriteLine(box2.Product.Name];
 }

 class Logger
 {
  public void Log(Product product)
  {
   Console.WriteLine("Product{0}...")
  }
 }
 class Product
 {
  public string Name{get; set;}
  public string Price{get;set;}
 }
 class Box
 {
  public Product Product{get;set;}
 }
 class WrapFactory
 {
 
 }
 public Box WrapProduct(Func<Product>getProduct,Action<Product>logCallback)
 {
  Box box=newBox():
  Product product=getProduct.Invoke();
  if(product.Price>=50)
  {
   logCallback(product);
  }
  box.Product=product;
  return box;
 }
class ProductFactory
 public Product MakePizza()
 {
  Product product=new Product[];
  product.Name= "Pizza";
  product.Price=12;
  return product;
 }
 public Product MakeToyCar()
 {
  Product product=new Product[];
  product.Name = "Toy Car";
  product.Price=100; 
  return product;
 }
}
```



### 委托使用的注意点
委托是一种方法级别的耦合，现实工作中要慎之又慎；
委托使用不当，会使可读性下降、debug难度增加；
把委托回调、异步调用、多线程纠缠在一起，会将代码变得难以阅读和维护；
委托使用不当可能造成内存泄漏和程序性能下降。（被占用的内存无法被释放）

### 多播委托（multicast）
一个委托内部，封装了不止一个方法。 

```C#
class Program
{
 static void Main(string[] args]
 {
  Student stu1 = new Student() {ID = 1, PenColor = ConsoleColorYellow}
  Student stu2 = new Student() {ID = 2, PenColor = ConsoleColorGreen}
  Student stu3 = new Student() {ID = 3, PenColor = ConsoleColor.Red } 
  Action action1 = new Action(stu1.DoHomework);
  Action action2 = new Action(stu2.DoHomework);
  Action action3 = new Action(stu3.DoHomework];
  action1 += action2;
  action1 += action3;
  action1.Invoke();
  }
}
```

### 隐式异步调用
- 同步与异步
同步：你做完了，我在你做的基础之上接着做。
异步：我们俩同时做。（同步进行）
- 同步调用与异步调用
每个程序都是一个进程（process）
每个进程可以有多个线程（Thread）
同步调用是在同一线程内，异步调用底层机理是多线程
- 隐式多线程与显示多线程
直接同步调用：直接通过方法名调用；
间接同步调用：使用委托进行；
隐式异步调用：使用委托的异步调用方法；
例子：
```C#
class Program
static void Main[string[] args]
Student stu1 = new Student() {ID = 1, PenColor = ConsoleCColor. Yellow
Student stu2 = new Student() {ID = 2, PenColor = ConsoleCColor.Green
Student stu3 = new Student() {ID = 3, PenColor = ConsoleCColor.Red
Action action1 = new Action(stu1.DoHomework];
Action action2=new Action(stu2.DoHomework);
Action action3 = new Action[stu3.DoHomework];
action1.BeginInvoke(null,null)
//生成一个分支线程。在分支线程里面调用方法。参数1：调用完之后的后续操作。
//.NET库不支持begininvoke，改用await Task.Run(Action1);
for (int i = 0; i < 10; i++)
Console.Foreground Color = ConsoleColor.Cyan;
Console.WriteLine["Main thread {0}.". i];
Thread.Sleep(1000);
class Student
{
 //...
}
```

显式异步调用：使用Thread或者Task；

```c#
Student stu1 = new Student() {ID = 1, PenColor = ConsoleCColor. Yellow

Student stu2 = new Student() {ID = 2, PenColor = ConsoleCColor.Green

Student stu3 = new Student() {ID = 3, PenColor = ConsoleColor.Red

Thread thread1 = new Thread[new ThreadStart(stu1.DoHomework)];

Thread thread2 = new Thread[new ThreadStart(stu2.DoHomework)]

Thread thread3 = new Thread[new ThreadStart(stu3.DoHomework)]

thread1.Start();

thread2.Start();

thread3.Start();

class Student

{

 //...

}
```



```c#
Student stu1 = new Student() {ID = 1, PenColor = ConsoleColor. Yellow

Student stu2 = new Student() {ID = 2, PenColor = ConsoleColorGreen}

Student stu3 = new Student() {ID = 3, PenColor = ConsoleColor.FRed

Task task1 = new Task[new Action[stu1.DoHomework]];

Task task2 = new Task[new Action[stu2.DoHomework]];

Task task3 = new Task(new Action(stu3.DoHomework));

task1.Start();

task2.Start():

task3.Start[]:
```

- 适当的使用接口（interface）取代对委托的使用
- 接口可以取代委托的作用。



# 事件（Event）
事件是，使对象或类具备通知能力的成员（to provide notification）。
事件主要用于，对象或类之间的动作协调与信息传递（消息推送）。

事件参数（Event Args）
响应事件 

## 事件模型的五个组成部分
- 事件拥有者（event source ， 对象）
- 事件成员（event ，成员）
- 时间的响应者（event subscriber ， 对象）
- 事件处理器（event handler ， 成员）
- 事件订阅——把事件处理器与事件进行关联，本质是一种以委托类型为基础的“约定”。

```C#
using System;
using System.Timers;

namespace EventStudy
{
    internal class Program
    {
        static void Main(string[] args)
        {
            Timer timer = new Timer();
            timer.Interval = 1000;
            Boy boy = new Boy();
            timer.Elapsed += boy.Action;
            timer.Start();
            Console.ReadLine();
        }
    }
    class Boy
    {
        internal void Action(object sender, ElapsedEventArgs e)
        {
            Console.WriteLine("Jump!");
        }
    }
}

```

```C#
using System;
using System.Timers;
using System.Windows.Forms;

namespace EventStudy
{
    internal class Program
    {
        static void Main(string[] args)
        {
            Form form =new Form();
            Controller controller=new Controller(form);
            form.ShowDialog();
        }
    }
    class Controller
    {
        private Form form;
        public Controller(Form form)
        {
            this.form = form;
            this.form.Click += this.FormClick;
        }

        private void FormClick(object sender, EventArgs e)
        {
            this.form.Text = DateTime.Now.ToString();
        }
    }
}
```

```C#
using System;
using System.Timers;
using System.Windows.Forms;

namespace EventStudy
{
    internal class Program
    {
        static void Main(string[] args)
        {
            MyForm form = new MyForm();
            form.Click += form.FormClick;
            form.ShowDialog();
        }
    }
    class MyForm : Form
    {
        internal void FormClick(object sender, EventArgs e)
        {
            this.Text = DateTime.Now.ToString();
        }
    }
}
```
最常用用法：
```C#
using System;
using System.Timers;
using System.Windows.Forms;

namespace EventStudy
{
    internal class Program
    {
        static void Main(string[] args)
        {
            MyForm form = new MyForm();
            form.ShowDialog();
        }
    }
    class MyForm : Form
    {
        private TextBox textBox;
        private Button button;
        public MyForm()
        {
            this.textBox = new TextBox();
            this.button = new Button();
            this.Controls.Add(textBox);
            this.Controls.Add(button);
            this.button.Click += this.ButtonClicked;
        }

        private void ButtonClicked(object sender, EventArgs e)
        {
            this.textBox.Text = "Hello ,World!!!!!!!!";
        }
    }
}
```

## 事件的声明
完整声明
```C#
using System;
using System.Timers;
using System.Windows.Forms;

namespace EventStudy
{
    internal class Program
    {
        static void Main(string[] args)
        {
            Customer customer = new Customer();
            Waiter waiter = new Waiter();
            customer.Order += waiter.Action;
            Console.ReadLine();
            customer.Think();
            customer.Paythebill();
        }
    }

    public class OrderEventArgs : EventArgs
    {
        public string DishName { get; set; }
        public string Size { get; set; }
    }

    public delegate void OrderEventHandler(Customer customer, OrderEventArgs e);

    public class Customer
    {
        private OrderEventHandler orderEventHandler;

        public event OrderEventHandler Order
        {
            add
            {
                this.orderEventHandler += value;
            }
            remove
            {
                this.orderEventHandler -= value;
            }
        }

        public double Bill { get; set; }
        public void Paythebill()
        {
            Console.WriteLine("I will pay ${0}", this.Bill);
        }

        public void Think()
        {
            OrderEventArgs e = new OrderEventArgs();
            e.DishName = "baga";
            e.Size = "large";
            this.orderEventHandler.Invoke(this, e);
        }
    }

    public class Waiter
    {
        public void Action(Customer customer, OrderEventArgs e)
        {
            Console.WriteLine("I will serve you the dish : {0}", e.DishName);
            double price = 10;
            switch (e.Size)
            {
                case "small":
                    price = price * 0.5;
                    break;
                case "large":
                    price = price * 1.5;
                    break;
                default:
                    break;
            }
            customer.Bill = price;
        }
    }
}

```
简略声明（Field-like）
语法糖。
```C#
        public event OrderEventHandler Order;
        public void Think()
        {
            OrderEventArgs e = new OrderEventArgs();
            e.DishName = "baga";
            e.Size = "large";
            this.Order.Invoke(this, e);
            //简略后，用事件代替为之前的字段。
            //而在完整版的代码中，事件只能用来+=（订阅）运算。
        }
```
事件，让程序更加有逻辑，对象之间的关系更清晰。 
委托是方法与方法之间的强关联，事件是对象与对象之间。而委托可能造成，不同对象，对同一个方法进行滥用。
事件，只能访问事件的订阅操作，而想访问事件的其他层面则不允许。而委托，可以设定传入委托的具体参数和返回值。
事件的本质，是委托字段的包装器，对其进行封装（encapsulation），事件对外界隐藏了委托实例的大部分功能，而只能对其进行订阅。
protected,dynamic,partial，virtual

# 类（class）
构造函数与析构函数
析构函数：`~Student(){...}`
## 类的访问级别修饰符
也就是，当其他项目（Aseembly）想要访问当前项目时，该类是否能被引用。
public：外部可以引用。
internal：仅在该项目内部有效。

## 类的继承
base class （基类）与derived class（派生类）
父类与子类
一个子类的实例，也是父类的一个实例。（例如，一个类的实例，是这个类的实例，也是object类的实例）
sealed：将该类进行封闭，不能再进行继承。
C#中一个类只支持一个基类，但是可以支持多个基接口。
public与internal修饰的类，需要注意访问级别的层级问题，子类访问级别不能高于父类。
在构造基类与派生类时，派生类会对基类的方法等进行改写和重构，如果再想通过该派生类去访问该派生类改写过的基类的方法等，是不行的（已被改写）。
父类的实例构造器，不能被子类继承。


## 类成员的访问
protected：将类成员的访问级别，限制在继承链上，protected 的类/方法，其子类可以对其进行访问。并且是跨程序集的。可以跟internal修饰符进行组合。

## 重写与多态（polymorphism）
```C#
class Vehicle 
{
 public virtual void Run()
 Console.WriteLine("I'm running!");
}
class Car: Vehicle 
{
 public override void Run() 
 {
  Console.WriteLine("Car is running!");
 }
}
```
如果不加virtual与override，则是子类对父类方法的隐藏。
在C#编程中，可以通过父类来声明一个子类的实例，这样做的用意是实现多态，也就是，在父类的引用中存储任何一个子类的对象，从而可以调用不同子类的实现。**（父类的方法或者属性，引用继承链上实例化的子类所带有的最新重写版本）**
通过父类声明子类实例可以让代码更加通用，尤其是在处理大量继承体系时。
在使用父类声明子类的实例时，最终得到的是 子类的实例，尽管变量的类型是父类。

```C#
public void MakeAnimalSound(Animal animal)
{
    animal.MakeSound();  // 不管是 Dog 还是 Cat，都会调用各自的 MakeSound 方法
}

public static void Main()
{
    Animal dog = new Dog();
    Animal cat = new Cat();

    MakeAnimalSound(dog);  // 输出: Bark
    MakeAnimalSound(cat);  // 输出: Meow
}
```

```C#
public class Animal
{
    public string Name { get; set; }

    public virtual void Speak()
    {
        Console.WriteLine("Animal speaks");
    }
}

public class Dog : Animal
{
    public void WagTail()
    {
        Console.WriteLine("Dog is wagging its tail");
    }

    public override void Speak()
    {
        Console.WriteLine("Dog barks");
    }
}

public class Program
{
    public static void Main()
    {
        Animal animal = new Dog(); // 父类类型的变量指向子类实例
        animal.Name = "Buddy";
        
        animal.Speak();  // 输出: Dog barks

        // animal.WagTail();  // 错误，Animal 类中没有 WagTail 方法
    }
}
```

属性也可以被重写。
# 接口、抽象类、SOLID、单元测试和反射

## 接口与抽象类
算法\设计原则
具体类 → 抽象类 → 接口：越来越抽象，内部实现的东西越来越少。
抽象类是未完全实现逻辑的类,（有被实现的部分也就是非abstract部分，也有部分abstract的）
### 抽象类与开放关闭（开闭）原则
**抽象类**不允许被实例化，但是可以作为基类。可以通过子类实例引用具体的方法。
**开闭原则**：没有特别情况（修BUG和增添新功能），尽可能不去修改已经写好的代码。也就是：我们应该封装那些不变的、稳定的、固定的和确定的成员，把那些不确定的、有可能改变的成员声明为抽象成员，并且留给子类去实现。

### 接口（Interface）与单元测试
- 接口是完全未实现逻辑的“类”，成员全部为public.
- 接口为解耦而生，方便单元测试，高内聚，低耦合。
- 接口是一个协约（contract）（有分工必有协作，有协作必有协约）
- 接口不能被实例化，只能用来声明变量、引用具体类（concrete class）的实例。
举例：
```C#
//接口-最抽象的类（所有方法等都是abstract）
interface IVehicle 
{
 void Stop();
 void Fill();
 void Run()
}
//基类
abstract class Vehicle : IVehicle 
{
 public void Stop()
 {
  Console.WriteLine('Stopped!');
 }
 public void Fill() 
 {
  Console.WriteLine("Pay and fill...");
 }
 abstract public void Run();
}
//实现类-两个
class Car : IVehicle 
{
 public override void Run() 
 {
  Console.WriteLine("Car is running...");
 }
}
class Truck : IVehicle 
{
 public override void Run() 
 {
  Console.WriteLine("Truck is running");
 }
}
```
接口在工作中的一般使用方法



方法\类之间的依赖或耦合
```C#
class Program
{
 static void Main(string[args] 
 {
  var engine = new Engine();
  var car = new Car(engine);
  car.Run(3);
  Console.WriteLine(car.Speed);
 }
}
//Engine与Car类有紧耦合，Car类对Engine类有强依赖
class Engine 
{
 public int RPM {get; private set;}
 public void Work[int gas] {
 this.RPM = 1000 * gas;
 //this.RPM = 0;
}
class Car 
{
 private Engine engine;
 public Car(Engine engine) {
  _engine = engine;
 }
 public int Speed {get; private set;}
 public void Run[int gas] {
  _engine.Work(gas);
 }
 this.Speed = engine.RPM / 100;
}
```
接口可以降低耦合度（保证给你提供的功能准确无误，可靠的）：
```C#
class Program {
 static void Main(stringħ args) {
  var user = new PhoneUser(new EricssonPhone)//只需要更改这一个地方，其他地方均不需要改动，因此耦合度低。
  user.UsePhone();
 }
}
class PhoneUser{
 private string _phone
 public PhoneUser(IPhone phone){
  _phone = phone;
 }
 public void UsePhone(){
  ...
 }
}
interface IPhone{...}
class NokiaPhone : IPhone{...}
class EricssonPhone : IPhone{...}
```

#### 依赖反转/倒置原则（Dependecy Inversion）：
之前是一个类依赖于另一个类，可以通过设计，改造为，一个类依赖于一个接口，而接口被依赖于其他类。
例如，司机依赖于交通工具接口，而不同种类的运输工具也依赖于交通工具接口。
#### 接口隔离原则（Interface Seperation）
接口和类的大小也不能太小，要把握好度。
接口提供的功能多于功能调用者
调用者不能多要功能，也就是调用最小的接口即可。
```C#
class Program 
{
 static void Main〔string[] args) {
 var driver = new Driver(new Car()//new Tank())};
 driver.Drive();
 }
}
class Driver 
{
 private IVehicle _vehicle;
 public Driver(IVehicle vehicle) 
 {
  _vehicle=vehicle;
 }
 public void Drive() 
 {
  _vehicle.Run();
 }
}
 interface IVehicle
 {
  void Run();
 }
  interface IWeapon
 {
  void Fire();
 }
 class Car : IVhicle{...}
 class Truck : IVhicle{...}

 interface ITank:IVehicle, IWeapon
 {
  //
 }
 class LightTank : ITank{...}
 class Medium Tank : ITank{...}
 class HeavyTank : ITank{...}
```

#### 单元测试：
在同一个solution下添加新的test project,命名为:**默认名称空间.test**;
...

## 反射、特性和依赖注入（Dependency Injection）

### 反射（Reflection）
是一种设计模式，通过依赖注入实现
### 依赖注入
将`Itank tank = new HeavyTank();`操作进行区分。
因为，当给程序进行升级的时候，很多时候接口后面所引用的实例具体类型需要大量更改，但是有的又不需要，因此，我们通过依赖注入，只需要将注册时候的引用实例类型更改一次即可，并且也可以和仍然需要`Itank tank = new HeavyTank();`的代码进行区分。

```C#
 interface ITank:IVehicle, IWeapon
 {
  //
 }
 interface IVehicle
 {
  void Run();
 }
 interface IWeapon
 {
  void Fire();
 }
class LightTank : ITank{...}
class Medium Tank : ITank{...}
class HeavyTank : ITank{...}
 
class Program 
{
 static void Main〔string[] args] 
 {
  //注册。
  var sc = new ServiceCollection();
  sc.AddScoped(typeof(ITank), typeof(Medium Tank)};
  //使用依赖注入后，只需要在这里更改ITank类引用的具体实例
  var sp = sc.BuildService Provider();
  //以下，是进行前台运行操作。
  ITank tank=sp.GetService<lTank>();
  tank.Fire();
  tank.Run();
 }
}

//-----------------------------------------
class Program 
{
 static void Main〔string[] args] 
 {
  //注册。
  var sc = new ServiceCollection();
  sc.AddScoped(typeof(IVehicle), typeof(Medium Tank)};
  sc.AddScoped<Driver>();
  var sp = sc.BuildService Provider();
  //以下，是进行前台运行操作。
  var driver=sp.GetService<Driver>();
  //当我们创建driver实例的时候，不再需要在构造函数里面去引用具体的类实例。
  //因为Driver类的构造函数中有传入参数的声明（IVehicle），因此DI会在注册列表中去找你的IVihecle，以及与之对应的类。
  driver.Drive();
 }
} 
```

### 通过反射实现更松的耦合
```C#

using System;
using System.Runtime.Loader;

namespace ReflectionStudy
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine();
            //在程序路径下创建Animals文件夹
            var folder = Path.Combine(Environment.CurrentDirectory, "Animals");
            //获取Animals文件夹里面的文件
            var files = Directory.GetFiles(folder);
            //创建动物的集合
            var animalTypes = new List<Type>();
            foreach (var file in files)
            {
                //将文件夹内的文件整合成一个assembly类型的实例
                var assembly = AssemblyLoadContext.Default.LoadFromAssemblyPath(file);
                //获得assembly里面，每个元素的所有信息（属性和方法等）
                var types = assembly.GetTypes();
                //将每个符合要求的type类型加到我们需要的type集合当中去。
                foreach (var t in types)
                {
                    if (t.GetMethod("Voice") != null)
                    {
                        animalTypes.Add(t);
                    }
                }
            }

            while (true)
            {
                //将animalsType中的动物逐一打印出名字
                for (int i = 0; i < animalTypes.Count; i++)
                {
                    Console.WriteLine($"{i + 1}。{animalTypes[i].Name}");
                }
                Console.WriteLine("=========================");
                //选择动物，然后判断是否超出范围，没有则为其选择对应的动物
                Console.WriteLine("Please choose animal:");
                int index = int.Parse(Console.ReadLine());
                if (index > animalTypes.Count || index < 1)
                {
                    Console.WriteLine("No such an animal. Try again!");
                    continue;                    
                };
                int Clicktimes = int.Parse(Console.ReadLine());
                //获取对应动物类型的typeinfo信息，然后创建反射。
                var t = animalTypes[index - 1];
                //获取t中，我们需要的方法。
                var m = t.GetMethod("Voice");
                //创建object实例o，并通过它来调用t实例的方法或者属性
                var o = Activator.CreateInstance(t);
                //将o传给新创建的object，然后触发m所反射的方法
                m.Invoke(o, new object[] { Clicktimes });
            }
        }
    }
}
```
**Assembly** 类可以加载以下类型的文件：
.exe 文件（可执行文件）
.dll 文件（动态链接库）
.winmd 文件（Windows Metadata 文件）
特定平台的程序集（如 .a、.so、.dylib）
加载这些文件时，Assembly 类会解析文件中的元数据，并允许你通过反射访问其中的类型、方法等信息。

- **.dll文件制作**
需要另起一个工程，选择class lib。
- ** SDK制作**
在总解决方案中，添加Interface工程
- **制作SDK插件**
在项目中添加依赖（dependence）中，选择browse，引用即可。

```C#
namespace Animals.Lib2 {
 [Unfinished]
 public class Cow : IAnimal 
 {
  public void Voice(int times) 
  {
   for (inti=0; i < times; i++)
   {
    Console.WriteLine("Moo!");
   }
  }
 }
}
```
- **Attribute**
在制作插件的过程中，未完成的部分，用[Unfinished]标注，最后在主程序这边会过滤掉。
前提是，在SDK接口开发中，声明一个Unfinished的特性，声明如下：
```C#
namespace BabyStroller.SDK {
 public class UnfinishedAttribute : Attribute {
 }
}
```

if continue

当我们采用接口和SDK了之后，代码会发生改动
比如过滤无效类型：
```C#
foreach (var t in types) 
{
 if (t.GetInterfaces().Contains(typeof(IAnimal))) 
 {
  var isUnfinished = t.GetCustomAttributes(false).Any(a   => a.GetType() == typeof(UnfinishedAttribute)};
  if (isUnfinished) continue;
  animalTypes.Add(t);
 }
}
```

以及最调用时：
```C#
int times = int.Parse(Console.ReadLine()};
var t = animalTypes[index - 1];
var m = t.GetMethod("Voice");
var o = Activator.Createlnstance(t);
var a = o as IAnimal;
a.Voice(times);
```

>接口`IAnimal` 定义了 `Voice `方法签名，例如：
```C#
public interface IAnimal
{
    void Voice(int times);
}
```
类实现接口：
假设有一个类`Dog`实现了 IAnimal 接口，并提供了 Voice 方法的实现：
```C#
public class Dog : IAnimal
{
    public void Voice(int times)
    {
        Console.WriteLine($"The dog barks {times} times.");
    }
}
```
这个类实现了 IAnimal 接口，提供了 Voice 方法的具体实现。
`var o = Activator.CreateInstance(t);`
o 是 Dog 类型的实例（如果 t 是 Dog）。此时，o 既是 Dog 类型的对象，也是 IAnimal 类型的对象，因为 Dog 实现了 IAnimal 接口。
接口转换和方法调用：你将 o 强制转换为 IAnimal 类型：
`var a = o as IAnimal;`
如果 o 实现了 IAnimal 接口，转换会成功，a 就变成了一个 IAnimal 类型的引用，指向 o。然后你可以调用 a.Voice(times)，实际上是在调用 Dog 类中实现的 Voice 方法。

# 泛型、partial类、枚举、结构体
## 泛型（generic）
为什么需要泛型：避免成员膨胀或者类型膨胀
正交性：泛型类型（类/接口/委托/...）、泛型成员（属性/方法/字段/...） 
### 泛型类型
```C#
internal class Program{
 public static void Main(string[] args) args) {
  Apple apple=new AppleO{Color= "Red"};
  Book book = new BookO {Name = "New Book"};
  Box<Apple> box1 = new Box<Apple>(){Cargo = apple};
  Box<Book> box2=new Box<Book>Q{Cargo = book};
 }
}

class Apple {
public string Color {get; set;}
}

class Book {
 public string Name { get; set;}
}
class Box<TCargo>{
 public TCargo Cargo {get; set;}
}
```


### 泛型接口
```c#
internal class Program{
 public static void Main(string[] args) args) {
  Student<int> stu = new Student<int>();
  stu.ID = 101;
  stu.Name= "Timothy";
 }

interface IUnique<Tld> {
 TId ID{get;set;}
}

class Student<TId>:lUnique<TId> {
 public TId ID { get; set;}
 public string Name {get; set;}
}
```
### 泛型集合
带有一个类型参数的泛型类型
```C#
internal class Program {
 public static void Main(string[] args) {
 IList<int> list = new List<int>();
 for (int i = 0;i<100; i++) {
  list.Add(i);
 }
 foreach (var item in list) {
  Console.WriteLine(item);
 }
}
```
带有两个类型参数的泛型类型
```c#
Dictionary<int, string> dict = new Dictionary<int, string>
```

### 泛型方法
```C#
internal class Program{
 public static void Main(string[] args) args) {
  int[] a1 = {1,2,3, 4, 5};
  int[] a2 = {1, 2, 3, 4, 5,6};
  double[] a3 = {1.1, 2.2, 3.3, 4.4, 5.5};
  double[] a4 = {1.1, 2.2, 3.3, 4.4, 5.5, 6.6};
  var result=Zip(a1,a2);
  //var result=Zip(a3,a4);
  Console.WriteLine(string.Join(",", result)};
 }
}

//方法：把两个数组组合成一个数组。
static T[] Zip<T>(T[] a, T[] b){
 T[] zipped = new T[a.Length + b.Length];
 int ai = 0, bi = 0, zi = 0;
 do {
  if (ai < a.Length) zipped[zi++]= a[ai++];
  if (bi < b.Length) zipped[zi++] = b[bi++];
 } 
 while (ai < a.Length || bi < b.Length);
 return zipped;
}
```

### 泛型委托
Action泛型委托
```C#
internal class Program
{
 public static void Main(string[] args) args) 
 {
  //Action泛型委托声明的方式和Action普通委托声明方式有差异。
  Action<string> a1 = Say;
  //Action a1 = new Action(Say)
  a1();
 }
 static void Say(string str)
 {
  Console.WriteLine(...);
 }
 static void Mul(int x)
 {
  ...;
 }
}

```
Func泛型委托
```c#
internal class Program
{
 public static void Main(string[] args) args) 
 {
  Func<double,double,double> func1 = Add;
  var result = func1(100, 200);
  Console.WriteLine(result); 
 }
 static int Add(int a,int b)
 {
  ...;
 }
 static double Add(double a,double b)
 {
  ...;
 }
```
泛型委托与lambda表达式：`Func<double,double,double> func1 = {a, b}=>{return a+b;};`
可以简化语句，节省声明方法所可能污染代码的可能。

## partial类
减少类的派生，当原类会自动更新覆盖等时，把自己添加的方法等放到其他名称空间下，则不会被删除或者重写。

partial类必须要保证和原类型在同一个名称空间之下。

## 枚举
枚举的每个值都有大小，大小也可以自定义设置。
`Console.WriteLine((int)Level.Employee);`
```C#
enum Level
{
 Employee = 100,
 Boss = 200,
}
/////////////////////////////
enum Level
{
 Employee = 100,
 Boss,
}
//不给BOSS,额外设定值的话，那么bOSS是101.
```
枚举的比特位编程
```C#
internal class Program
{
 public static void Main(string[] args) args) 
 {
  //假设这个人四个技能都会。
  person.Skill = Skill.Drive | Skill.Cook| Skill.Program| SkiTeach;
  Console.WriteLine();
  //按或运算符计算。
 }
enum Skill 
{
 Drive = 1,
 Cook=2,
 Program=4,
 Teach=8
}
class Person
{
 public Skill skill;
}
```
## 结构体（struct）
结构体是值类型，存放值类型数据。
装箱与拆箱操作：
```C#
internal class Program 
{
 public static void Main(string[] args) args) 
 {
  Student student = new Student(KID = 101, Name = "Timtohy223;
  object object = student;//装箱
  Student student2 = (Student) obj;//拆箱
  Console.WriteLine($"#{student2.ID} Name:(student2Name}")
 }
}

struct Student 
{
 public int ID { get; set;}
 public string Name { get; set;}
}
```
结构体对象，进行复制时，是完全复制了其所有数据，而不是只复制了其引用的地址。
结构体类型，只能由接口派生。
结构体不能拥有显示的无参构造器：`public Studnt(){}`.（备注，有参的可以。）




# LINQ、lambda表达式与委托
## lambda
- 匿名方法
- Inline方法

**泛型委托的类型推断**
```C#
//两种表达方式：
//Func<int, int, int> func = new Func<int,int,int>((a, b) => { return a + b;});
Func<int, int, int> func = (a, b) => { return a + b;};
int res = func(100, 200);
Console.WriteLine(res);
func=(x,y)=>{returnx
res = func(3,4);
y;};
Console.WriteLine(res);
```
另外一种使用方式：
```C#
class Program 
{
 static void Main(string[] args) 
 {
  DoSomeCalc<int>((int a, int b) => (return a b; }, 100,200);
  //高级写法：
  //DoSomeCalc((a,b) => (return a b; }, 100,200);
  static void DoSomeCalc<T>(Func<T, T, T  T> func,T x, Ty)
 {
  T res=func(x,y);
  Console.WriteLine(res);
 }
}
```
## LINQ(.NET Language Integrated Query)
将C#语句转换为sql语句，进行查询，但是sql的查询会更准确，LINQ只是临时简单使用。
........
