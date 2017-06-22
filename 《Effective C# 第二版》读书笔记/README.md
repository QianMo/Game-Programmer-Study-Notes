《Effective C\# 第二版》读书笔记
============================
<br>

原则1：尽可能地使用属性(property)，而不是可直接访问的数据成员
-------------------------------------------------------------
<br>

## 1. 概述

属性一直是C\#语言的一等公民。自C\#
1.0版本以来，C\#对属性进行了一系列的增强，让其表达能力不断提到。你甚至可以为setter和getter指定不同的访问权限。隐式属性也极大降低了声明属性时的工作量，不会比声明数据成员麻烦多少。

属性是一种全功能的、第一等的语言元素，能够以方法调用的形式访问或修改内部数据。成员函数中可以实现的任何功能都可以在属性中实现。

如果你的类里还有Public的变量，如果你还在手写get and set 方法，可以停下来了。

属性允许将数据成员作为共有接口的一部分暴露出去，同时仍旧提供面向对象环境下所需的封装。属性这个语言元素可以让你像访问数据成员一样使用，但其底层依然是用方法来实现的。

.Net Framework假定你使用Property来让外界访问你类里想让外界访问到的数据成员
。实际上也是这样的，因为.Net的数据绑定只支持Property，而不支持公有数据成员的访问。数据绑定的目的就是把一个对象的属性绑定到一个用户界面的control上。数据绑定是通过反射来实现的，如下例：

	textBoxCity.DataBindings.Add("Text", address, "City");

这段代码就是把textBoxCity的Text Property绑定到address这个对象的City
属性上。公有的数据成员并不推荐使用，因此Framework Class
Library设计器也不支持其实现绑定。这样的设计也保证了我们必须选择合适的面向对象技术。
<br>
<br>

## 2. 使用属性，可以方便地加入检查机制

如果你使用了Property，你可以非常轻松的添加一个检查机制，如下面这段代码:

	public class CustomerEx
	{
	    private string name;
	    public string Name
	    {
	        get { return name; }
	        set
	        {
	            if (string.IsNullOrEmpty(value))
	                throw new ArgumentException(
	                "Name cannot be blank",
	                "Name");
	            name = value;
	        }
	        //...
		}
	}



<br>

## 3. 属性可支持多线程

因为属性是用方法实现的，所以它拥有方法所拥有的一切语言特性。

比如，属性增加多线程的支持是非常方便的。你可以加强 get 和 set 访问器
（accessors）的实现来提供数据访问的同步：
		
	public class Customer
	{
	    private object syncHandle = new object();
	    private string name;
	    public string Name
	    {
	        get
	        {
	            lock (syncHandle)
	                return name;
	        }
	        set
	        {
	            if (string.IsNullOrEmpty(value))
	                throw new ArgumentException(
	                "Name cannot be blank",
	                "Name");
	            lock (syncHandle)
	                name = value;
	        }
	    }
	    // More Elided.
	}



<br>


## 4. 属性可以声明为virtual

同样，因为属性是用方法实现的，所以它拥有方法所拥有的一切语言特性，那么同样，属性可以被定义为virtual:

	public class Customer
	{
	    public virtual string Name
	    {
	        get;
	        set;
	    }
	}

<br>

## 5. 属性可以声明为接口，以及abstract

显而易见，你也可以把Property扩展为abstract，甚至成为interface的一部分：

	public interface INameValuePair
	{
	    object Name
	    {
	        get;
	    }
	    object Value
	    {
	        get;
	        set;
	    }
	}



<br>

## 6. 属性可以使用泛型

你可以用泛型+接口的属性类型：

	public interface INameValuePair<T>
	{
	    T Name
	    {
	        get;
	    }
	    T Value
	    {
	        get;
	        set;
	    }
	}

<br>

## 7. 给get 与set定义不同的访问权限

如前所述，因为属性是用方法实现的，所以它拥有方法所拥有的一切语言特性。

因为实现Property访问的方法get 与set是独立的两个方法，在C\#
2.0之后，你可以给它们定义不同的访问权限，来更好的控制类成员的可见性，如下：

	public class Customer
	{
	    private string _name;
	    public virtual string Name
	    {
	        get
	        {
	            return _name;
	        }
	        protected set
	        {
	            _name = value;
	        }
	    }
	    //...
	}




<br>

## 8. 在属性中使用索引器

想返回序列中的项，创建一个属性是非常不错的做法，如下例：

	public class Customer
	{
	    private int[] _theValues= new int[3] { 1,2,2};
	
	    public int this[int index]
	    {
	        get
	        {
	            return _theValues[index];
	        }
	        set
	        {
	            _theValues[index] = value;
	        }
	    }
	}

	//usage:
	Customer c1 = new Customer();
	c1[1] = 666;
	var cc1 = c1[1];



索引器和单元素Property有着相同的特性，他们都是作为方法实现的，因此可以在索引器内部实现任意的验证或者计算逻辑。索引器也可为虚的或抽象的，可以声明在接口中，可以为只读或读写。一维且使用数字作为参数的索引器也可以参与数据绑定。使用非整数参数的索引器可以用来定义其他的数据结构，如map和dictionary：

	public class AddressEx
	{
	    private Dictionary<int, string> _address = new Dictionary<int, string>();
	    public string this[int name]
	    {
	        get
	        {
	            return _address[name];
	        }
	        set
	        {
	            _address[name] = value;
	        }
	    }
	}

	//usage:
	AddressEx a1 = new AddressEx();
	a1[1] = "New York";
	a1[2] = "Hong Kong";
	var a11 = a1[1];




<br>

## 9. 创建多维索引器

为了和多维数组保持一致，我们可以创建多维索引器，在不同的维度上使用相同或不同类型：

	class ComputeValueClass
	{
	    public int this[int x, int y]
	    {
	        get { return ComputeValue(x, y); }
	    }
	
	    public int this[int x, string name]
	    {
	        get { return ComputeValue(x, name); }
	    }
	
	    private int ComputeValue(int x, int y)
	    {
	        //...
	    }
	
	    private int ComputeValue(int x, string y)
	    {
	        //...
	    }
	
	}


值得注意的是所有索引器必须使用 this 关键字声明。在 C\#
中你不能自己命名索引器。所以一个类型的索引器必须有不同的参数列表来避免歧义。几乎所有的属性的功能都适用索引器。索引器可以是
virtual 或 abstract ；索引器的 setters 和 getters
可以不同的访问限制。不过，你不能像创建隐式属性一样创建隐式索引器。

<br>

## 10. 总结

总而言之，无论什么时候，当你想让你类内部的数据被外界访问到时(无论是public还是protected)，一定要用属性。对于序列和字典，可以使用索引器。

使用Property，你可以得到如下好处：

1．数据绑定的支持

2．对于需求变化有更强的适应性，更方便的修改实现方法。

记住，现在多花1分钟使用Property，会在你修改程序以适应设计变化时，为你节约n小时。
