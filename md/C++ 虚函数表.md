\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnblogs.com\](https://www.cnblogs.com/yunlambert/p/9876564.html)

虚函数表
----

C++ 中虚函数是通过一张虚函数表 (Virtual Table) 来实现的，在这个表中，主要是一个类的虚函数表的地址表；这张表解决了继承、覆盖的问题。在有虚函数的类的实例中这个表被分配在了这个实例的内存中，所以当我们用父类的指针来操作一个子类的时候，这张虚函数表就像一张地图一样指明了实际所应该调用的函数。

C++ 编译器是保证虚函数表的指针存在于对象实例中最前面的位置 (是为了保证取到虚函数表的最高的性能)，这样我们就能通过已经实例化的对象的地址得到这张虚函数表，再遍历其中的函数指针，并调用相应的函数。

下面先看一段代码:

```
class Base {
public:
	virtual void f() { cout << "Base::f()" << endl; }
	virtual void g() { cout << "Base::g()" << endl; }
	virtual void h() { cout << "Base::h()" << endl; }
};

typedef void(\*Fun)(void);

int main()
{
	Base b;
	Fun pFun = NULL;
	cout << "虚函数表的地址为:" << (int\*)(&b) << endl;
	cout << "虚函数表的第一个函数地址为:" << (int\*)\*(int\*)(&b) << endl;

	pFun = (Fun)\*((int\*)\*(int\*)(&b));
	pFun();
	system("pause");
	return 0;
}
```

运行结果如下:  
![](https://upload-images.jianshu.io/upload_images/7154520-cdec24ff8a6e9220.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
我们再追踪一下虚函数表的地址:

![](https://upload-images.jianshu.io/upload_images/7154520-9956cbfd47139f3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/7154520-ddc8627703878dc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/7154520-7df9f477061ef5f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

结合结果分析一下代码：首先是创建了一个 Base 的类，Base 类里面有三个成员函数，都为虚函数；然后`typedef void(*Fun)(void)`是利用类型别名声明一个函数指针，指向的地址为 NULL，可以等价成这样:`typedef decltype(void) *Fun。`然后再到 main 函数里，利用 Base 实例化了对象了 b；然后`Fun pFun=NULL`则是声明了一个返回指向函数的指针，该指针 pFun 此时也是 NULL，根据图 1 可以知道，他的类型是`void(*)()`，表示的就是函数指针，而当执行完这句后就会从原来没有分配的`0xcccccccc`变成`0x00000000`。接下来就是`(int*)(&b)`强行把`&b`转成`int *`，取得虚函数表的地址；再次取址就可以得到第一个虚函数的地址了，也就是 Base::f() 。接下来就是 `pFun = (Fun)*((int*)*(int*)(&b));` 把函数指针指向虚函数表的第一个函数，最后再`pFunc()`运行。

如果想让 pFun 调用其它的函数，可以是这样:

```
(Fun)\*((int\*)\*(int\*)(&b)+0);  // Base::f()

(Fun)\*((int\*)\*(int\*)(&b)+1);  // Base::g()

(Fun)\*((int\*)\*(int\*)(&b)+2);  // Base::h()
```

通过下图可以很好的进行理解:

![](https://upload-images.jianshu.io/upload_images/7154520-d6fc2ae8c52a8381.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对`(int*)*(int*)(&b)`可以这样理解，`(int*)(&b)`就是对象 b 的地址，只不过被强制转换成了`int*`了，如果直接调用`*(int*)(&b)`则是指向对象 b 地址所指向的数据，但是此处是个虚函数表呀，所以指不过去，必须通过`(int*)`将其转换成函数指针来进行指向就不一样了，它的指向就变成了对象 b 中第一个函数的地址，所以`(int*)*(int*)(&b)`就是独享 b 中第一个函数的地址；又因为`pFun`是由`Fun`这个函数声明的函数指针，所以相当于是 Fun 的实体，必须再将这个地址转换成`pFun`认识的，即加上`(Fun)*`进行强制转换: 简要概括就是从 b 地址开始  
读取四个字节的内容，然后将这个内容解释成一个内存地址，然后访问这个地址，然后将这个地址中存放的值再解释成一个函数的地址.

![](https://upload-images.jianshu.io/upload_images/7154520-6f1b90dfafaaf583.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面将对比说明有无虚函数覆盖情况下的虚函数表的样子:

### 一般继承 (无虚函数覆盖)

先写出一个继承关系：  
![](https://upload-images.jianshu.io/upload_images/7154520-1f06c2228a329248.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

写成代码如下:

```
class Base {
public:
	virtual void f() { cout << "Base::f()" << endl; }
	virtual void g() { cout << "Base::g()" << endl; }
	virtual void h() { cout << "Base::h()" << endl; }
};

class Derive :public Base{
public:
	virtual void f1() { cout << "Base::f1()" << endl; }
	virtual void g1() { cout << "Base::g1()" << endl; }
	virtual void h1() { cout << "Base::h1()" << endl; }
};

typedef void(\*Fun)(void);

int main()
{
	//Base b;
	Derive d;
	Fun pFun = NULL;
	cout << "虚函数表的地址为:" << (int\*)(&d) << endl;
	cout << "虚函数表的第一个函数地址为:" << (int\*)\*(int\*)(&d)<< endl;
	pFun =(Fun)\*((int\*)\*(int\*)(&d)+1);
	pFun();
	pFun = (Fun)\*((int\*)\*(int\*)(&d) + 3);
	pFun();

	system("pause");
	return 0;
}
```

执行结果如下:

![](https://upload-images.jianshu.io/upload_images/7154520-9705527b45d637bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过调试看一下相应的虚函数表:

![](https://upload-images.jianshu.io/upload_images/7154520-c951824c646ae61d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/7154520-ccdd646beb31ef8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个继承关系中，子类没有重载任何父类的函数，我们实例化了一个对象 d：`Derive d`，则它的虚函数表是如下的:

![](https://upload-images.jianshu.io/upload_images/7154520-3cdcadbb7c85dff4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以**虚函数按照其声明顺序放于表中**，并且**父类的虚函数在子类的虚函数前面**。

### 一般继承 (有虚函数覆盖)

这样的一个继承关系:

![](https://upload-images.jianshu.io/upload_images/7154520-8537a8df625af29a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个继承关系中，Derive 的`f()`重载了 Base 类中的`f()`, 下面我们来逐步调试:

![](C:%5CUsers%5CADMINI~1%5CAppData%5CLocal%5CTemp%5C1540901666638.png)

上图是刚刚通过`Derive d`声明的虚函数表的样子，我们再直接打印出对象 d 的第一个函数、第三个函数和第四个函数来看看:

![](https://upload-images.jianshu.io/upload_images/7154520-cff55d41faee7894.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

相应的虚函数表变成了:

![](https://upload-images.jianshu.io/upload_images/7154520-948ba0b25c781f9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以知道**覆盖的`f()`函数被放到了虚函数表中原来父类虚函数的位置，而没有被覆盖的函数依次往后排列**。

### 多重继承 (无虚函数覆盖)

现在用这样一个继承关系:

![](https://upload-images.jianshu.io/upload_images/7154520-af4b73882f1ba8f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

调试程序发现:

![](https://upload-images.jianshu.io/upload_images/7154520-51b380a2f5abec42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果我们访问第一个函数地址之后的第 6 个函数位置会发生什么呢？

![](https://upload-images.jianshu.io/upload_images/7154520-7c9c4f5f5cfdaf27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

是找不到的，说明虚函数表已经不是按原来的方式通过一个地址找到所有的函数，或者说所有的子函数实现是按照顺序排列来存放的了。

虚函数表是这样的:

![](https://upload-images.jianshu.io/upload_images/7154520-29f7e0901bcd5508.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是我们通过我们目前实现访问虚函数表的方式是访问不到下面两张虚函数表的，却可以通过这样来实现:

```
Derive d;
Base2 \*b2=&d;
b2->f();
b2->g();
```

![](https://upload-images.jianshu.io/upload_images/7154520-b64982b01a72cde7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/7154520-cb52347fd71c21ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在声明了 b2 并绑定 d 的时候，已经指向了 Base2 的虚函数表的地址，再通过`b2->f()`就可以访问 Base2 的虚函数表中第一个函数的位置。

### 多重继承 (有虚函数覆盖)

继承关系:

![](https://upload-images.jianshu.io/upload_images/7154520-ff63d60151243b1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里就不放出代码和调试内容了，直接给出虚函数表的样子:

![](https://upload-images.jianshu.io/upload_images/7154520-7f4ddec87aa333a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

几点注意
----

1\. 不能通过父类型的指针访问子类自己的虚函数，是非法的

```
Base \*b=new Derive();
b->f1();     //编译会出错
```

P.S: 我们可以通过指针的方式访问虚函数表来达到违反 C++ 语义的行为

```
class Base1 {
public:
	virtual void f() { cout << "Base1::f" << endl; }
	virtual void g() { cout << "Base1::g" << endl; }
	virtual void h() { cout << "Base1::h" << endl; }
};

class Base2 {
public:
	virtual void f() { cout << "Base2::f" << endl; }
	virtual void g() { cout << "Base2::g" << endl; }
	virtual void h() { cout << "Base2::h" << endl; }
};

class Base3 {
public:
	virtual void f() { cout << "Base3::f" << endl; }
	virtual void g() { cout << "Base3::g" << endl; }
	virtual void h() { cout << "Base3::h" << endl; }
};

class Derive : public Base1, public Base2, public Base3 {
public:
	virtual void f() { cout << "Derive::f" << endl; }
	virtual void g1() { cout << "Derive::g1" << endl; }
};

typedef void(\*Fun)(void);

int main()
{
	Fun pFun = NULL;
	Derive d;
	int\*\* pVtab = (int\*\*)&d;
	//Base1's vtable
	//pFun = (Fun)\*((int\*)\*(int\*)((int\*)&d+0)+0);
	pFun = (Fun)pVtab\[0\]\[0\];
	pFun();
	//pFun = (Fun)\*((int\*)\*(int\*)((int\*)&d+0)+1);
	pFun = (Fun)pVtab\[0\]\[1\];
	pFun();
	//pFun = (Fun)\*((int\*)\*(int\*)((int\*)&d+0)+2);
	pFun = (Fun)pVtab\[0\]\[2\];
	pFun();
	//Derive's vtable
	//pFun = (Fun)\*((int\*)\*(int\*)((int\*)&d+0)+3);
	pFun = (Fun)pVtab\[0\]\[3\];
	pFun();
	//The tail of the vtable
	pFun = (Fun)pVtab\[0\]\[4\];
	cout << pFun << endl;
	//Base2's vtable
	//pFun = (Fun)\*((int\*)\*(int\*)((int\*)&d+1)+0);
	pFun = (Fun)pVtab\[1\]\[0\];
	pFun();
	//pFun = (Fun)\*((int\*)\*(int\*)((int\*)&d+1)+1);
	pFun = (Fun)pVtab\[1\]\[1\];
	pFun();
	pFun = (Fun)pVtab\[1\]\[2\];
	pFun();
	//The tail of the vtable
	pFun = (Fun)pVtab\[1\]\[3\];
	cout << pFun << endl;

	//Base3's vtable
	//pFun = (Fun)\*((int\*)\*(int\*)((int\*)&d+1)+0);
	pFun = (Fun)pVtab\[2\]\[0\];
	pFun();
	//pFun = (Fun)\*((int\*)\*(int\*)((int\*)&d+1)+1);
	pFun = (Fun)pVtab\[2\]\[1\];
	pFun();
	pFun = (Fun)pVtab\[2\]\[2\];
	pFun();
	//The tail of the vtable
	pFun = (Fun)pVtab\[2\]\[3\];
	cout << pFun << endl;
	return 0;

}
```

在这里延伸一下关于`int **`与`int *`, 这里引用了知乎上的一片答案:

```
C语言里面的定义的指针，它除了表示一个地址，它还带有类型信息。这个类型信息，用来告诉你，在这个地址空间上，存放着什么类型的变量。打个比如，有如下的代码片段：int a;
int \*p = &a;
假设p的指针值为0x08004000，并且int类型长度为4字节。那么p将告诉你，\[0x08004000, 0x08004004) 内存空间上存放着一个int类型。如果你只从0x08004000地址中，只读取两个字节，那是错误的。同样道理，以下代码片段：int a;
int \*b = &a;
int \*\*c = &b;
是告诉你，c的指针值是另一个指针（b），而该指针则指向一个int变量。如果你刻意要较真，认为指针就是一个地址，并且举出下面的例子：int a;
int \*b = &a;
int \*c = &b;  // 请留意这里
最后一句赋值，实际上是将类型信息丢弃了。因为在编译器看来，C 就表示一个指针，它指向的是一个整数，只是你自己将它解释成指针而已。printf("%d\\n",\*(int \*)(\*c)); // 开发人员，将\*c解释成指针，编译器是不认帐的喔
为什么将一个整数解释成一个指针没有问题呢？那是因为凑巧，你把它放到64位架构上试试看看。
```

2\. 如果父类的虚函数是 private 或者是 protected 的，但这些非 public 的虚函数同样会存在于虚函数表中！  
3\. 虚函数表不一定是存在最开头，但是目前各个编译器大多是这样设置的。  
![](https://upload-images.jianshu.io/upload_images/7154520-e127a8b302145d35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)