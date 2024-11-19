
# OI教学笔记

标签（空格分隔）： 算法 数据结构 c++

---

[TOC]

## 程序测试
### 运行时间测试
c++的头文件ctime中的库函数clock()提供了测试函数运行时间的方法：
1、clock()返回类型为clock_t类型
2、clock_t实际为long 类型，   typedef long clock_t
3、clock() 函数，返回从  开启这个程序进程 到 程序中调用clock（）函数 时之间的CPU时钟计时单元（clock tick）数（挂钟时间），返回单位是毫秒
4、可以用常量CLOCKS_PER_SEC， 这个常量表示每一秒（per second）有多少个时钟计时单元
实例代码如下：
```cpp
#include <iostream>
#include <ctime>
using namespace std;
int main(){
clock_t start, end;
start = clock();
        //如果函数执行时间少的话，可能最后测出的结果为0,因为现在机子的运行速度很快
for(int j = 0; j < 1000; j++)
 for(int i = 0; i < 1000000; i++){
}
end = clock();
cout << (double)(end - start) / CLOCKS_PER_SEC << endl;
return 0;
}
//根据clock函数的定义，可以只用最后一个clock函数即可
```
运行结果：2.562

##ACM/OI 对拍程序的写法
原创 2016年04月14日 11:04:55 标签：数据 /测试 /调试 /批处理 4588
转载请注明出处：http://blog.csdn.net/wlx65003/article/details/51149196
搞程序设计竞赛的同学很多时候都会因为WA但苦苦找不到错误数据而苦恼，虽然肉眼debug的能力也很重要，但有的时候一直手打数据测试两三天也没有必要。这里就介绍一种对拍程序的写法，是我改进过的，自认为效率应该是比较高了。

如果你懒得学实现细节了，想直接使用，那么下面的内容可以略过了，到这去下载打包好的，里面有使用教程： 
http://pan.baidu.com/s/1boERyGZ

实现细节

首先对拍程序，顾名思义，一个输入给两个程序分别跑一遍，看看对不对的上。

那么牵扯到三个步骤：

生成一组输入数据
把这组数据分别给两个程序运行，并生成两组输出数据
比较两组输出数据
看到这个步骤很多人应该已经有想法了，没错用文件操作能实现，但太麻烦，因为你得修改你自己的代码把输出重定向到一个文件，这要是一不小心忘记删重定向直接交了又得WA一遍233（不对，是RE），这里介绍一种更方便更高逼格的批处理命令。

首先新建一个批处理文件，命名为简易对拍程序.bat，什么你不知道怎么新建？右键新建一个文本文档直接把后缀名改成bat就好啦，因为批处理文件本质上就是一堆命令文本嘛。

然后右键—编辑，开始打代码：

首先第一步：生成一组输入数据。 
我们假设你已经写好了一个数据生成器，编译成rand.exe并放在当前目录下了，那么我们只要把这个程序的输入重定向到一个文件就行了，如果你直接在源码里操作，还得各种文件流重定向烦得要死。在批处理命令里很简单，就一句话：
```bat
rand.exe > in.txt
```
是不是很简单明了？ 
那怎么把文件输入到一个程序里去呢？没错：
```bat
my.exe < in.txt
std.exe < in.txt
```
my.exe是你写的错误程序，std.exe是标程 
那怎么把两个程序的输出再重定向到文件里去呢？也很简单：
```bat
my.exe < in.txt > myout.txt
std.exe < in.txt > stdout.txt
```
是不是相当方便？ 
接下来就是比较myout.txt和stdout.txt了，也不用你手写判断程序，windows自带一个比较命令：fc(file compare)
```bat
fc myout.txt stdout.txt
```
如果两个没有差异，会显示：找不到差异，否则会显示不同的附近的几行的文本。

汇总一下：
```bat
rand.exe > in.txt
my.exe < in.txt > myout.txt
std.exe < in.txt > stdout.txt
fc myout.txt stdout.txt
```
好，这样一个简易版对拍程序就写好了。但这个功能也太简陋了，只能对拍一次，要是数据难找点岂不是要你运行到手酸？ 
有人就问了，能不能循环？答案是：可以！
```BAT
@echo off  
:loop  
    rand.exe > in.txt
    my.exe < in.txt > myout.txt
    std.exe < in.txt > stdout.txt
    fc myout.txt stdout.txt
if not errorlevel 1 goto loop  
pause
goto loop
```
别懵逼，一行行给你解释。 
首先@echo off 是关掉输入显示，不然你的所有命令都会显示出来的，防止刷屏。 
：loop是定位标记点，和c语言里的goto很像。 
中间是主体程序。 
if not errorlevel 1 goto loop ，errorlevel 是上一个命令的返回值，fc在文件不同时返回1，相同时返回0，这一行的意思就是，如果fc返回的不是1，就跳到:loop，使劲循环。 
pause，暂停，一旦fc返回1，就会执行到这一行，停住程序，给你时间看数据。 
goto loop，看完数据，按下任意键结束暂停，继续循环。

这样一来功能就顿时强大起来了，为了纪念这么伟大的改进，我们把文件名重命名为普通版对拍程序.bat。（网上流传的也大多就这个版本了）

但这还不够！ 为什么？ 我们看一下rand程序的写法。 
例如题目格式是，T组数据，每组数据一个n，一个m，然后n个1~m的整数 
你就这么写：
```C++
#include<bits/stdc++.h>
using namespace std;
#define random(a,b) ((a)+rand()%((b)-(a)+1))

int main( int argc, char *argv[] )
{ 
    int seed=time(NULL);
    srand(seed);

    printf("1\n");
    int n=10;
    int m=random(1,20);
    printf("%d %d\n",n,m);
    for(int i=0 ; i<n ; ++i)
    {
        printf(" %d ",random(0,m));
    }
    printf("\n");
    return 0;
}
```
这样的话有个缺点，time(NULL) 是一秒才更新一次的，也就是说我们的随机数据一秒才换一次，太慢了！ 
有没有什么变的更快的随机数种子？有！windows自带了一个随机数发生器：%random%，它的值就是一个随机整数，可以在命令行里调用。 
那接下来就好办了，我们把这个数传给rand.exe用来当随机数种子就行了。

什么？你不知道怎么传？ 
呃，你知不知道main函数里这两个参数干嘛用的:int argc, char *argv[]？ 
恐怕好多人还不知道，我这里解释下，这两个就是传入参数，argc 是参数个数，*argv[] 是参数表，从1开始。

知道了这个就好办了。
```bat
@echo off  
:loop  
    rand.exe %random% > data.in
    std.exe < data.in > std.out
    my.exe < data.in > my.out
    fc my.out std.out 
if not errorlevel 1 goto loop  
pause
goto loop
```
我们把%random%当参数传给rand.exe就行了。 
然后程序里这么写：
```C++
#include<bits/stdc++.h>
using namespace std;
#define random(a,b) ((a)+rand()%((b)-(a)+1))

stringstream ss;

int main( int argc, char *argv[] )
{ 
    int seed=time(NULL);
    if(argc)//如果有参数
    {
        ss.clear();
        ss<<argv[1];
        ss>>seed;//把参数转换成整数赋值给seed
    }
    srand(seed);
    //以上为随机数初始化，请勿修改
    //random(a,b)生成[a,b]的随机整数

    //以下写你自己的数据生成代码 
    printf("1\n");
    int n=10;
    int m=random(1,20);
    printf("%d %d\n",n,m);
    for(int i=0 ; i<n ; ++i)
    {
        printf(" %d ",random(0,m));
    }
    printf("\n");
    return 0;
}
```
我这里用stringstream把字符串转换成整数，你们也可以用其他办法。 
这么一来，数据测试效率就得到了上千倍的提升！为了纪念这么伟大的改进，我们把程序重命名为狂拽酷炫吊炸天对拍程序.bat。

这么一来对拍程序就彻底完成啦！完结撒花！ 
你只需把my.cpp和std.cpp放在和对拍程序相同的目录下 
my.cpp里放你自己的代码，编译成my.exe 
std.cpp里放标程，编译成std.exe 
（推荐用dev-c++，单文件编译方便）

然后双击运行对拍程序，等待它暂停，然后打开in.txt就能看到对拍出来的数据啦~
## STL
### vector
### list
### map
### set集合
set集合容器实现了红黑树（Red-Black Tree）的平衡二叉检索树的的数据结构，在插入元素时，它会自动调整二叉树的排列，把该元素放到适当的位置，以确保每个子树根节点的键值大于左子树所有节点的键值，而小于右子树所有节点的键值；另外，还得确保根节点的左子树的高度与有字数的高度相等，这样，二叉树的高度最小，从而检索速度最快。要注意的是，它不会重复插入相同键值的元素，而采取忽略处理。
平衡二叉检索树的检索使用中序遍历算法，检索效率高于vector、deque、和list的容器。另外，采用中序遍历算法可将键值由小到大遍历出来，所以，可以理解为平衡二叉检索树在插入元素时，就会自动将元素按键值从小到大的顺序排列。

Set容器和其他容器差不多，无非就是相同的值不存，存进去自动排序好了。

构造set集合的主要目的是为了快速检索，使用set前，需要在程序头文件中包含声明“#include<set>”。

1.创建set集合对象

创建set对象时，需要指定元素的类型，这一点和其他容器一样。
```cpp

#include<iostream>
#include<set>
using namespace std;
int main()
{
	set<int> s;
	return 0;
}
```
2.元素的插入与中序遍历
采用inset()方法把元素插入到集合中，插入规则在默认的比较规则下，是按元素值从小到大插入，如果自己指定了比较规则函数，则按自定义比较规则函数插入。使用前向迭代器对集合中序遍历，结果正好是元素排序后的结果。
```cpp

#include<iostream>
#include<set>
using namespace std;
int main()
{
	set<int> s;
	s.insert(5); //第一次插入5，可以插入
	s.insert(1);
	s.insert(6);
	s.insert(3);
	s.insert(5); //第二次插入5，重复元素，不会插入
	set<int>::iterator it; //定义前向迭代器
	//中序遍历集合中的所有元素
	for(it = s.begin(); it != s.end(); it++)
	{
		cout << *it << " ";
	}
	cout << endl;
	return 0;
}
//运行结果：1 3 5 6
```
3.元素的方向遍历
使用反向迭代器reverse_iterator可以反向遍历集合，输出的结果正好是集合元素的反向排序结果。它需要用到rbegin()和rend()两个方法，它们分别给出了反向遍历的开始位置和结束位置。
```cpp
#include<iostream>
#include<set>
using namespace std;
int main()
{
	set<int> s;
	s.insert(5); //第一次插入5，可以插入
	s.insert(1);
	s.insert(6);
	s.insert(3);
	s.insert(5); //第二次插入5，重复元素，不会插入
	set<int>::reverse_iterator rit; //定义反向迭代器
	//反向遍历集合中的所有元素
	for(rit = s.rbegin(); rit != s.rend(); rit++)
	{
		cout << *rit << " ";
	}
	cout << endl;
	return 0;
}
//运行结果：6 5 3 1
```
4.元素的删除
与插入元素的处理一样，集合具有高效的删除处理功能，并自动重新调整内部的红黑树的平衡。删除的对象可以是某个迭代器位置上的元素、等于某键值的元素、一个区间上的元素和清空集合。
```cpp

#include<iostream>
#include<set>
using namespace std;
int main()
{
	set<int> s;
	s.insert(5); //第一次插入5，可以插入
	s.insert(1);
	s.insert(6);
	s.insert(3);
	s.insert(5); //第二次插入5，重复元素，不会插入
	s.erase(6); //删除键值为6的元素
	set<int>::reverse_iterator rit; //定义反向迭代器
	//反向遍历集合中的所有元素
	for(rit = s.rbegin(); rit != s.rend(); rit++)
	{
		cout << *rit << " ";
	}
	cout << endl;	
	set<int>::iterator it;

	it = s.begin();
	for(int i = 0; i < 2; i++)
		it = s.erase(it); 
	for(it = s.begin(); it != s.end(); it++)
		cout << *it << " ";
	cout << endl;

	s.clear();
	cout << s.size() << endl;

	return 0;
}
/*
运行结果：
5 3 1
5
0    
*/
```
5.元素的检索
使用find()方法对集合进行检索，如果找到查找的的键值，则返回该键值的迭代器位置；否则，返回集合最后一个元素后面的一个位置，即end()。
```cpp

#include<iostream>
#include<set>
using namespace std;
int main()
{
	set<int> s;
	s.insert(5); //第一次插入5，可以插入
	s.insert(1);
	s.insert(6);
	s.insert(3);
	s.insert(5); //第二次插入5，重复元素，不会插入
	set<int>::iterator it;
	it = s.find(6); //查找键值为6的元素
	if(it != s.end())
		cout << *it << endl;
	else
		cout << "not find it" << endl;
	it = s.find(20);
	if(it != s.end())
		cout << *it << endl;
	else
		cout << "not find it" << endl;
	return 0;
}
/*
运行结果：
6
not find it   
*/
```
下面这种方法也能判断一个数是否在集合中：
```cpp

#include <cstdio>
#include <set>
using namespace std;
int main() {
    set <int> s;
    int a;
    for(int i = 0; i < 10; i++)
        s.insert(i);
    for(int i = 0; i < 5; i++) {
        scanf("%d", &a);
        if(!s.count(a)) //不存在
            printf("does not exist\n");
        else
            printf("exist\n");
    }
    return 0;
}
```
6.自定义比较函数

使用insert将元素插入到集合中去的时候，集合会根据设定的比较函数奖该元素放到该放的节点上去。在定义集合的时候，如果没有指定比较函数，那么采用默认的比较函数，即按键值从小到大的顺序插入元素。但在很多情况下，需要自己编写比较函数。

编写比较函数有两种方法。

(1)如果元素不是结构体，那么可以编写比较函数。下面的程序比较规则为按键值从大到小的顺序插入到集合中。
```cpp

#include<iostream>
#include<set>
using namespace std;
struct mycomp
{ //自定义比较函数，重载“（）”操作符
	bool operator() (const int &a, const int &b)
	{
		if(a != b)
			return a > b;
		else
			return a > b;
	}
};
int main()
{
	set<int, mycomp> s; //采用比较函数mycomp
	s.insert(5); //第一次插入5，可以插入
	s.insert(1);
	s.insert(6);
	s.insert(3);
	s.insert(5); //第二次插入5，重复元素，不会插入
	set<int,mycomp>::iterator it;
	for(it = s.begin(); it != s.end(); it++)
		cout << *it << " ";
	cout << endl;
	return 0;
}
/*
运行结果：6 5 3 1  
*/
```
(2)如果元素是结构体，那么可以直接把比较函数写在结构体内。
```cpp

#include<iostream>
#include<set>
#include<string>
using namespace std;
struct Info
{
	string name;
	double score;
	bool operator < (const Info &a) const // 重载“<”操作符，自定义排序规则
	{
		//按score由大到小排序。如果要由小到大排序，使用“>”即可。
		return a.score < score;
	}
};
int main()
{
	set<Info> s;
	Info info;

	//插入三个元素
	info.name = "Jack";
	info.score = 80;
	s.insert(info);
	info.name = "Tom";
	info.score = 99;
	s.insert(info);
	info.name = "Steaven";
	info.score = 60;
	s.insert(info);

	set<Info>::iterator it;
	for(it = s.begin(); it != s.end(); it++)
		cout << (*it).name << " : " << (*it).score << endl; 
	return 0;
}
/*
运行结果：
Tom : 99
Jack : 80
Steaven : 60
*/
```
Set常用函数：跟其他容器的函数差不多，好像都通用
c++ stl容器set成员函数:begin()--返回指向第一个元素的迭代器
c++ stl容器set成员函数:clear()--清除所有元素
c++ stl容器set成员函数:count()--返回某个值元素的个数
c++ stl容器set成员函数:empty()--如果集合为空，返回true
c++ stl容器set成员函数:end()--返回指向最后一个元素的迭代器
c++ stl容器set成员函数:equal_range()--返回集合中与给定值相等的上下限的两个迭代器
c++ stl容器set成员函数:erase()--删除集合中的元素
c++ stl容器set成员函数:find()--返回一个指向被查找到元素的迭代器
c++ stl容器set成员函数:get_allocator()--返回集合的分配器
c++ stl容器set成员函数:insert()--在集合中插入元素
c++ stl容器set成员函数:lower_bound()--返回指向大于（或等于）某值的第一个元素的迭代器
c++ stl容器set成员函数:key_comp()--返回一个用于元素间值比较的函数
c++ stl容器set成员函数:max_size()--返回集合能容纳的元素的最大限值
c++ stl容器set成员函数:rbegin()--返回指向集合中最后一个元素的反向迭代器
c++ stl容器set成员函数:rend()--返回指向集合中第一个元素的反向迭代器
c++ stl容器set成员函数:size()--集合中元素的数目
c++ stl容器set成员函数:swap()--交换两个集合变量
c++ stl容器set成员函数:upper_bound()--返回大于某个值元素的迭代器
c++ stl容器set成员函数:value_comp()--返回一个用于比较元素间的值的函数
##数据结构
时间复杂度和空间复杂度
###线性表

###单调栈
例题：Loongint的花篮
【Description】
Loongint要和MM结婚了。在两人的走进礼堂的红地毯两侧，需要摆一些装饰用的花篮，有一些不同高度的花篮，现在这些花篮被Loongint依照自己的美学观念编号为S1,S2,S3…Sn（两侧的花篮高度一样）。可Loongint的MM对这些花篮的摆放方式有不同的看法，她觉得满足以下条件的花篮摆放才是最好的。
如果对于区间[Si,Sj]$(1<=i<j<=n)$ 中任意的花篮都比Si高且比Sj低，那么这个区间称为一个美学区间。对于所有的美学区间，其长度（定义为j-i）都必须小于等于k，如果有长度大于k的美学区间，MM就会不高兴，Loongint就会有麻烦…
【Input】
第一行为m。表示有m组测试数据。
对于每一组：
第一行n，k，分别表示花篮的数量和美学区间的最大长度。
第二行为n个数，分别表示S1,S2,S3…Sn的值。
【Output】
如果根本不存在美学区间，输出-1。
如果存在美学区间，那么如果任意区间的长度都小于等于k，那么输出最大的长度，否则输出最大长度比k大多少（MaxLength-k）。
【Sample Input】
3
4 2
5 4 3 6
4 1
6 5 4 3
4 2
1 2 3 4
【Sample Output】
1
-1
1
【Hint】
对于30%的测试数据，1<=n<=100。
对于60%的测试数据，1<=n<=5555。
对于100%的测试数据，1<=n<=100000，0< Si<=100000，1<=m<=3。

###单调队列
###跳表和散列
### 树
#### Prufer序列
prufer数列是一种无根树的编码表示，对于一棵n个节点带编号的无根树，对应唯一一串长度为n-2的prufer编码。

（1）无根树转化为prufer序列。

首先定义无根树中度数为1的节点是叶子节点。

找到编号最小的叶子并删除，序列中添加与之相连的节点编号，重复执行直到只剩下2个节点。

如下图的树对应的prufer序列就是3，5，1，3。

![](http://img.blog.csdn.net/20160216223141915?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

具体实现可以用一个set搞定，维护度数为1的节点。复杂度O(nlgn)。
```cpp
/**********************
给一棵无根树进行prufer编码
**********************/

#include <iostream>
#include <cmath>
#include <cstdlib>
#include <time.h>
#include <vector>
#include <cstdio>
#include <cstring>
#include <set>
using namespace std;
#define maxn 111111
#define maxm 211111

set <int> gg;
struct node {
	int u, v, next;
} edge[maxm];
int n, head[maxn], cnt, degree[maxn];
bool vis[maxn];

void add_edge (int u, int v) {
	edge[cnt].u = u, edge[cnt].v = v, edge[cnt].next = head[u], head[u] = cnt++;
}

int main () {
	cin >> n;
	memset (head, -1, sizeof head);
	memset (degree, 0, sizeof degree);
	memset (vis, 0, sizeof vis);
	gg.clear ();
	cnt = 0;
	for (int i = 1; i < n; i++) {
		int u, v;
		scanf ("%d%d", &u, &v);
		add_edge (u, v);
		add_edge (v, u);
		degree[u]++, degree[v]++;
	}
	for (int i = 1; i <= n; i++) {
		if (degree[i] == 1) {
			gg.insert (i);
			vis[i] = 1;
		}
	}

	set<int>::iterator it;
	int prufer[maxn], id = 0;
	for (; id <= n-3;) {
		int u = (*(it = gg.begin ()));
		gg.erase (u);
		for (int i = head[u]; i != -1; i = edge[i].next) {
			int v = edge[i].v;
			if (vis[v])
				continue;
			degree[v]--;
			prufer[++id] = v;
			if (degree[v] == 1) {
				gg.insert (v);
				vis[v] = 1;
			}
		}
	}
	for (int i = 1; i <= id; i++) {
		cout << prufer[i] << " ";
	} cout << endl;
	return 0;
}
```
（2）prufer序列转化为无根树。
设点集V={1,2,3,...,n}，每次取出prufer序列中最前面的元素u，在V中找到编号最小的没有在prufer序列中出现的元素v，给u，v连边然后分别删除，最后在V中剩下两个节点，给它们连边。最终得到的就是无根树。

具体实现也可以用一个set，维护prufer序列中没有出现的编号。复杂度O(nlgn)。
```cpp

/**********************
prufer序列解码
**********************/

#include <iostream>
#include <cmath>
#include <cstdlib>
#include <time.h>
#include <vector>
#include <cstdio>
#include <cstring>
#include <set>
using namespace std;
#define maxn 1111111
#define maxm 2111111

int n;
int prufer[maxn], node[maxn], cnt;
set<int> gg; //在prufer序列里没有出现的点
int vis[maxn]; //这个点是不是在prufer序列里面
bool used[maxn]; //这个点有没有使用过

int main () {
	cin >> n;
	gg.clear ();
	memset (vis, 0, sizeof vis);
	memset (used, 0, sizeof used);
	cnt = 0;
	for (int i = 1; i <= n-2; i++) {
		cin >> prufer[i];
		vis[prufer[i]]++;
	}
	for (int i = 1; i <= n; i++) {
		if (!vis[i]) {
			gg.insert (i);
		}
	}
	set<int>::iterator it;
	for (int i = 1; i <= n-2; i++) {
		int v = (*(it = gg.begin ())), u = prufer[i];
		cout << u << "-" << v << endl;
		used[v] = 1;
		gg.erase (v);
		vis[u]--;
		if (!vis[u] && !used[u]) {
			gg.insert (u);
		}
	}
	it = gg.begin ();
	cout << *it << "-" << *(++it) << endl;
	return 0;
}
```
最后有一个很重要的性质就是prufer序列中某个编号出现的次数+1就等于这个编号的节点在无根树中的度数

#### 经典证明：Prüfer编码与Cayley公式
![图](http://www.matrix67.com/blogimage/200808233.gif)
Cayley公式是说，一个完全图K_n有n^(n-2)棵生成树，换句话说n个节点的带标号的无根树有n^(n-2)个。今天我学到了Cayley公式的一个非常简单的证明，证明依赖于Prüfer编码，它是对带标号无根树的一种编码方式。
给定一棵带标号的无根树，找出编号最小的叶子节点，写下与它相邻的节点的编号，然后删掉这个叶子节点。反复执行这个操作直到只剩两个节点为止。由于节点数n>2的树总存在叶子节点，因此一棵n个节点的无根树唯一地对应了一个长度为n-2的数列，数列中的每个数都在1到n的范围内。下面我们只需要说明，任何一个长为n-2、取值范围在1到n之间的数列都唯一地对应了一棵n个节点的无根树，这样我们的带标号无根树就和Prüfer编码之间形成一一对应的关系，Cayley公式便不证自明了。

注意到，如果一个节点A不是叶子节点，那么它至少有两条边；但在上述过程结束后，整个图只剩下一条边，因此节点A的至少一个相邻节点被去掉过，节点A的编号将会在这棵树对应的Prüfer编码中出现。反过来，在Prüfer编码中出现过的数字显然不可能是这棵树（初始时）的叶子。于是我们看到，没有在Prüfer编码中出现过的数字恰好就是这棵树（初始时）的叶子节点。找出没有出现过的数字中最小的那一个（比如④），它就是与Prüfer编码中第一个数所标识的节点（比如③）相邻的叶子。接下来，我们递归地考虑后面n-3位编码（别忘了编码总长是n-2）：找出除④以外不在后n-3位编码中的最小的数（左图的例子中是⑦），将它连接到整个编码的第2个数所对应的节点上（例子中还是③）。再接下来，找出除④和⑦以外后n-4位编码中最小的不被包含的数，做同样的处理……依次把③⑧②⑤⑥与编码中第3、4、5、6、7位所表示的节点相连。最后，我们还有①和⑨没处理过，直接把它们俩连接起来就行了。由于没处理过的节点数总比剩下的编码长度大2，因此我们总能找到一个最小的没在剩余编码中出现的数，算法总能进行下去。这样，任何一个Prüfer编码都唯一地对应了一棵无根树，有多少个n-2位的Prüfer编码就有多少个带标号的无根树。

一个有趣的推广是，n个节点的度依次为D1, D2, …, Dn的无根树共有(n-2)! / [ (D1-1)!(D2-1)!..(Dn-1)!]个，因为此时Prüfer编码中的数字i恰好出现Di-1次。

#### 十三、二叉树的前序、中序和后序遍历
```c++
void preorder(int x)//二叉树的先序遍历
 {
   if(x==0) return;
   cout<<x;//先访问根
   preorder(a[x].ld);//再先序遍历根的左子树
   preorder(a[x].rd);//最后先序遍历根的右子树
}
 
void inorder(int x)//二叉树的中序遍历
 {
   if(x==0) return;
   preorder(a[x].ld);//先中序遍历根的左子树
   cout<<x;//再访问根
   preorder(a[x].rd);//最后中序遍历根的右子树
}
 
void reorder(int x)//二叉树的后序遍历
 {
   if(x==0) return;
   preorder(a[x].ld);//先后序遍历根的左子树
   preorder(a[x].rd);//再后序遍历根的右子树
   cout<<x;//最后访问根
}
``` 
 
####十四、树转换为二叉树算法
 
####十五、二叉排序树
##### 数组实现
1.结构定义：
```cpp
struct node{
	int key;
	int left,right,p;
}tree[1001];
int cnt=0;
```
2.遍历输出
```cpp
void inorder_print(int root)
{
	if(tree[root].left!=0)inorder_print(tree[root].left);
	printf("%d ",tree[root].key);
	if(tree[root].right!=0)inorder_print(tree[root].right);
}
```
3.插入结点
```
void insert(int root,int x,int cnt) 
{
	tree[cnt].key=x;
	if(root==cnt)return ;
	if(x<tree[root].key)
	{
		if(tree[root].left==0)
		{
			tree[root].left=cnt;
			tree[cnt].p=root;
		}
		else insert(tree[root].left,x,cnt);
	}
	else
	{
		if(tree[root].right==0)
		{
			tree[root].right=cnt;
			tree[cnt].p=root;
		}
		else insert(tree[root].right,x,cnt);
	}
}
```
4.综合实现代码
```cpp
#include <cstdio>
using namespace std;
struct node{
	int key;
	int left,right,p;
}tree[1001];
int cnt=0;
void insert(int root,int x,int cnt) 
{
	tree[cnt].key=x;
	if(root==cnt)return ;
	if(x<tree[root].key)
	{
		if(tree[root].left==0)
		{
			tree[root].left=cnt;
			tree[cnt].p=root;
		}
		else insert(tree[root].left,x,cnt);
	}
	else
	{
		if(tree[root].right==0)
		{
			tree[root].right=cnt;
			tree[cnt].p=root;
		}
		else insert(tree[root].right,x,cnt);
	}
}
void inorder_print(int root)
{
	if(tree[root].left!=0)inorder_print(tree[root].left);
	printf("%d ",tree[root].key);
	if(tree[root].right!=0)inorder_print(tree[root].right);
}
int main()
{
	int x;
	scanf("%d",&x);
	while(x!=-1)
	{
		insert(1,x,++cnt);
		scanf("%d",&x);
	}
	inorder_print(1);
	for(int i=1;i<=cnt;i++)
	{
		printf("key=%d left=%d right=%d p=%d\n",
			tree[i].key,tree[i].left,tree[i].right,tree[i].p);
	}
	return 0;
}
```
####十六、哈夫曼树
```c++
void haff(void) //构建哈夫曼树
{
   for(int i=n+1;i<=2*n-1;i++) //依次生成n-1个结点
     {int l=fmin(i-1); //查找权值最小的结点的编号l
       a[i].lchild=l; //把l作为结点i的左孩子
       a[l].father=i; //把l的父结点修改为i
       int r=fmin(i-1); //查找次小权值的编号r
       a[i].rchild=r; //把l作为结点i的右孩子
       a[r].father=i; //把r的父结点修改为i
       a[i].da=a[l].da+a[r].da; //合并l,j结点，生成新结点i
     }
}
int fmin(int k)//在1到K中寻找最小的权值的编号
       {
         int mins=0;
         for(int s=1;s<=k;s++)
           if((a[mins].da>a[s].da)&&(a[s].father==0)) //a[s].father=0,说明这个结点还不是别个结点
mins=s;                           //的孩子，不等于0说明这个结点已经用过。
         return mins;
       }
void inorder(int x)//递归生成哈夫曼编码
{
  if(a[x].father==0) {a[x].code=”“;}//根结点
  if(a[a[x].father].lchild==x)  a[x].code=a[a[x].father].code+'0';
  if(a[a[x].father].rchild==x)  a[x].code=a[a[x].father].code+'1';
  if(a[x].lchild!=0) inorder(a[x].lchild);//递归生成左子树
  if((a[x].lchild==0)&&(a[x].rchild==0))//输出叶子结点
     cout<<a[x].da<<':'<<a[x].code<<endl;
  if(a[x].rchild!=0) inorder(a[x].rchild);//递归生成右子树
}
```
####线段树
####左高树
####竞赛树（赢者树、输者树）
####搜索树
（二叉搜索树、平衡搜索树（AVL树、红黑树、分裂树、B-树、SBT

###图
####连通分量
####重连通分量
####十一、深度优先搜索
```c++
void dfs(int x)  \\以图的深度优先遍历为例。
    { 
      cout<<x<<‘ ‘; \\访问x顶点
      visited[x]=1; \\作已访问的标记
      for(int k=1;k<=n;k++) \\对与顶点x相邻而又没访问过的结点k进行深度优先搜索。
        if((a[x][k]==1)&&(visited[k]==0))
         dfs(k);
    }
```	
####十二、广度优先搜索
```c++
void  bfs(void) //按广度优先非递归遍历图G，n个顶点，编号为1..n。注：图不一定是连通的
{//使用辅助队列Q和访问标记数组visited。
    for(v=1;v<=n;v++)  visited[v]=0；//标记数组初始化
    for(v=1; v<=n; v++)
      if(visited[v]==0 ) {        //v尚未访问
         int h=1,r=1;    //置空的辅助队列q
         visited[v]=1；//顶点v,作访问标记
         cout<<v<<‘ ‘; //访问顶点v
         q[r]=v；    //v入队列
         while(h<=r) //当队列非空时循环
 {
             int tmp=q[h];  //队头元素出队，并赋值给tmp
             for(int j=1;j<=n;j++)
               if((visited[j]==0)&&(a[tmp][j]==1))
{//j为tmp的尚未访问的邻接顶点
                    visited[j]=1;  对j作访问标记
                    cout<<j<<‘ ‘; 访问j
                    r++; //队尾指针加1
q[r]=j; //j入队
}  //end-if
             h++;
           }//end -while
}
```

####十八、最小生成树之Prim算法
```c++
void prime(void) //prim算法求最小生成树，elist[i]是边集数组，a[i][j]为<I,j>的权值。edge为结构体类型。
{for (int i=1;i<=n-1;i++)//初始化结点1到其它n-1个结点形成的边集
   {elist[i].from=1;
elist[i].to=i+1;
elist[i].w=a[1][i+1];
   }
 for (int i=1;i<=n-1;i++)//依次确定n-1条边
  {int m=i;
   for(int j=i+1;j<=n-1;j++)//确定第i条边时，依次在i+1至n-1条边中找最小的那条边
     if(elist[j].w<elist[m].w) m=j;
   if(m!=i) //如果最小的边不是第i条边就交换
{edge tmp=elist[i];elist[i]=elist[m];elist[m]=tmp;}
   for(int j=i+1;j<=n-1;j++)//更新第i+1至n-1条边的最小距离。
     {if(elist[j].w>a[elist[i].to][elist[j].to]) elist[j].w=a[elist[i].to][elist[j].to];}
}                            
for(int i=1;i<=n-1;i++)//求最小生成树的值
ans=ans+elist[i].w;
}                       
``` 
//如果要求出哪些边构成最小生成树，在更新第i+1至n-1条边到已经生成的树中最小距离时(上面代码中加粗的部分)，还要加上elist[j].from=elist[i].to;语句，即在更新权值时，还应该更新起点。
//Prime算法适用于顶点不是太多的稠密图，如果对于顶点数较多的稀疏图，就不太适用了。
####二十、最小生成树之Kruscal算法
```c++
void qsort(int x,int y)//对边集数组进行快速排序
{int h=x,r=y,m=elist[(h+r)>>1].w;
 while(h<r)
  {while(elist[h].w<m) h++;
   while(elist[r].w>m) r--;
   if(h<=r)
    {edge tmp=elist[h];elist[h]=elist[r];elist[r]=tmp;h++;r--;}
  }
 if(x<r) qsort(x,r);
 if(h<y) qsort(h,y);
}
 
int getfather(int x)//找根结点，并压缩路径，此处用递归实现的。
{if(x==father[x]) return x;
 else {
        int f=getfather(father[x]);
        father[x]=f;
        return f;
      }
}
 
void merge(int x,int y)//合并x,y结点，在此题中的x,y为两个根结点。
{father[x]=y;}
 
void kruscal(void)
{int sum=0,ans=0;
qsort(1,t);//对t条边按权值大小按从小到大的次序进行快速排序
   for(int i=1;i<=t;i++)
     {int x1=getfather(elist[i].from);//取第i条边的起点所在的树的根
int x2=getfather(elist[i].to);// 取第i条边的终点所在的树的根
if(x1!=x2)
{sum++;merge(x1,x2);ans+=elist[i].w;}//不在同一个集合，合并，即第i条边可以选取。
if(sum>n-1)
break;//已经确定了n-1条边了，最小生成树已经生成了，可以提前退出循环了
}
   if(sum<n-1)
cout<<"Impossible"<<endl; //从t条边中无法确定n-1条边，说明无法生成最小生成树
   else  
cout<<ans<<endl;
  }
``` 
//克鲁斯卡尔算法，只用了边集数组，没有用到图的邻接矩阵，因此当图的结点数比较多的时候，输入数据又是边的信息时，就要考虑用Kruscal算法。对于岛国问题，我们就要选择此算法，如果用Prim算法，还要开一个二维的数组来//表示图的邻接矩阵，对于10000个点的数据，显然在空间上是无法容忍的。
####十九、最短路径之Dijkstra算法
```C++
void dijkstra(int x)  //求结点x到各个结点的最短路径
{
memset(vis,0,sizeof(vis)); //初始化，vis[i]＝0表示源点到结点i未求，否则已求
vis[x]=1;pre[x]=0; //初始化源点。
for(int i=1;i<=n;i++)   //对其它各点初始化。
    if(i!=x)
{
dis[i]=g[x][i];
pre[i]=x;
}
for(int i=1;i<=n-1;i++)   //对于n个结点的图，要求x到其它n-1个结点的最短距离
    {
int m=big; //虚拟一个最大的数big=99999999;
int k=x;
      for(int j=1;j<=n;j++)   //在未求出的结点中找一个源点到其距离最小的点
        if(vis[j]==0&&m>dis[j])
{
m=dis[j];
k=j;
}
      vis[k]=1;   //思考：如果k=X说明什么？说明后面的点，无解。
      for(int j=1;j<=n;j++)   //用当前找的结点更新未求结点到X的最短路径
      if((vis[j]==0)&&(dis[k]+g[k][j]<dis[j]))
        {   
         dis[j]=dis[k]+g[k][j];  //更新
         pre[j]=k;  //保存前趋结点，以便后面求路径
        }
    }
}
//说明：dis[i]表示x到i的最短距离，pre[i]表示i结点的前趋结点。
```
####最短路径之Bellman-Ford算法
 
####二十一、最短路径之Floyed算法
```C++
void floyed(void)// a[i][j]表示结点i到结点j的最短路径长度，初始时值为<I,J>的权值。
{for(int k=1;k<=n;k++) //枚举中间加入的结点不超过K时f[i][j]最短路径长度，K相当DP中的阶段
        for(int i=1;i<=n;i++) //i，j是结点i到结点J，相当于DP中的状态
for(int j=1;j<=n;j++)
      if (a[i][j]>a[i][k]+a[k][j]) a[i][j]=a[i][k]+a[k][j];//这是决策，加和不加中间点，取最小的值
}
``` 
//弗洛伊德算法适合于求没有负权回路的图的最短路径长度，利用FLOYED算法，可写出判断结点i和结点J是否连通的算法。

---
## 算法
### 求第k大或第k小或中位数的BFPRT算法
BFPRT算法又称为中位数的中位数算法，它的最坏时间复杂度为O(n)，它是由Blum、Floyd、Pratt、Rivest、Tarjan提出。BFPRT算法是解决从n个数中选择第k大或第k小的数这个经典问题的著名算法。
该算法的思想是修改快速选择算法的主元选取方法，提高算法在最坏情况下的时间复杂度。
一 基本思路
关于选择第k小的数字有许多方法，其效率和复杂度各不一样，可以根据实际情况进行选择。
 1. 将n个数排序(比如快速排序或归并排序)，选取排序后的第k个数，时间复杂度为O(nlogn)。使用STL函数sort可以大大减少编码量。
2. 将方法1中的排序方法改为线性时间排序算法(如基数排序或计数排序)，时间复杂度为O(n)。但线性时间排序算法使用限制较多，不常使用。
3. 维护一个k个元素的最大堆，存储当前遇到的最小的k个数，时间复杂度为O(nlogk)。这种方法同样适用于海量数据的处理。
4. 部分的选择排序，即把最小的放在第1位，第二小的放在第2位，直到第k位为止，时间复杂度为O(kn)。实现非常简单。
5. 部分的快速排序（快速选择算法），每次划分之后判断第k个数在左右哪个部分，然后递归对应的部分，平均时间复杂度为O(n)。但最坏情况下复杂度为O(n^2)。
6. BFPRT算法，修改快速选择算法的主元选取规则，使用中位数的中位数的作为主元，最坏情况下时间复杂度为O(n)。
二 快速选择算法
快速选择算法就是修改之后的快速排序算法，前面快速排序的实现与应用这篇文章中讲了它的原理和实现。

其主要思想就是在快速排序中得到划分结果之后，判断要求的第k个数是在划分结果的左边还是右边，然后只处理对应的那一部分，从而达到降低复杂度的效果。

在快速排序中，平均情况下数组被划分成相等的两部分，则时间复杂度为T(n)=2*T(n/2)+O(n)，可以解得T(n)=nlogn。
在快速选择中，平均情况下数组也是非常相等的两部分，但是只处理其中一部分，于是T(n)=T(n/2)+O(n)，可以解得T(n)=O(n)。

但是两者在最坏情况下的时间复杂度均为O(n^2)，出现在每次划分之后左右总有一边为空的情况下。为了避免这个问题，需要谨慎地选取划分的主元，一般的方法有：

- 固定选择首元素或尾元素作为主元。
- 随机选择一个元素作为主元。
- 三数取中，选择三个数的中位数作为主元。一般是首尾数，再加中间的一个数或者随机的一个数。

在BFPTR算法中，仅仅是改变了快速排序Partion中的pivot值的选取，在快速排序中，我们始终选择第一个元素或者最后一个元素作为pivot，而在BFPTR算法中，每次选择五分中位数的中位数作为pivot，这样做的目的就是使得划分比较合理，从而避免了最坏情况的发生。算法步骤如下：
（1）将输入数组的n个元素划分为$\lfloor n/5 \rfloor$组，每组5个元素，且至多只有一个组由剩下的n%5个元素组成。
（2）寻找$\lfloor n/5 \rfloor$个组中每一个组的中位数，首先对每组的元素进行插入排序，然后从排序过的序列中选出中位数。
（3）对于（2）中找出的$\lceil n/5 \rceil$个中位数，递归进行步骤（1）和（2），直到只剩下一个数即为这$\lceil n/5 \rceil$个元素的中位数，找到中位数后并找到对应的下标。

（4）进行Partion划分过程，Partion划分中的pivot元素下标为p。

（5）进行高低区判断即可。

本算法的最坏时间复杂度为，值得注意的是通过BFPTR算法将数组按第K小（大）的元素划分为两部分，而这高低两部分不一定是有序的，通常我们也不需要求出顺序，而只需要求出前K大的或者前K小的。
```cpp
#include <iostream>  
#include <string.h>  
#include <stdio.h>  
#include <time.h>  
#include <algorithm>  
   
using namespace std;  
const int N = 10005;  
   
int a[N];  
   
//插入排序  
void InsertSort(int a[], int l, int r)  
{  
    for(int i = l + 1; i <= r; i++)  
    {  
        if(a[i - 1] > a[i])  
        {  
            int t = a[i];  
            int j = i;  
            while(j > l && a[j - 1] > t)  
            {  
                a[j] = a[j - 1];  
                j--;  
            }  
            a[j] = t;  
        }  
    }  
}  
   
//寻找中位数的中位数  
int FindMid(int a[], int l, int r)  
{  
    if(l == r) return a[l];  
    int i = 0;  
    int n = 0;  
    for(i = l; i < r - 5; i += 5)  
    {  
        InsertSort(a, i, i + 4);  
        n = i - l;  
        swap(a[l + n / 5], a[i + 2]);  
    }  
   
    //处理剩余元素  
    int num = r - i + 1;  
    if(num > 0)  
    {  
        InsertSort(a, i, i + num - 1);  
        n = i - l;  
        swap(a[l + n / 5], a[i + num / 2]);  
    }  
    n /= 5;  
    if(n == l) return a[l];  
    return FindMid(a, l, l + n);  
}  
   
//寻找中位数的所在位置  
int FindId(int a[], int l, int r, int num)  
{  
    for(int i = l; i <= r; i++)  
        if(a[i] == num)  
            return i;  
    return -1;  
}  
   
//进行划分过程  
int Partion(int a[], int l, int r, int p)  
{  
    swap(a[p], a[l]);  
    int i = l;  
    int j = r;  
    int pivot = a[l];  
    while(i < j)  
    {  
        while(a[j] >= pivot && i < j)  
            j--;  
        a[i] = a[j];  
        while(a[i] <= pivot && i < j)  
            i++;  
        a[j] = a[i];  
    }  
    a[i] = pivot;  
    return i;  
}  
   
int BFPTR(int a[], int l, int r, int k)  
{  
    int num = FindMid(a, l, r);    //寻找中位数的中位数  
    int p =  FindId(a, l, r, num); //找到中位数的中位数对应的id  
    int i = Partion(a, l, r, p);  
   
    int m = i - l + 1;  
    if(m == k) return a[i];  
    if(m > k)  return BFPTR(a, l, i - 1, k);  
    return BFPTR(a, i + 1, r, k - m);  
}  
   
int main()  
{  
    int n, k;  
    scanf("%d", &n);  
    for(int i = 0; i < n; i++)  
        scanf("%d", &a[i]);  
    scanf("%d", &k);  
    printf("The %d th number is : %d\n", k, BFPTR(a, 0, n - 1, k));  
    for(int i = 0; i < n; i++)  
        printf("%d ", a[i]);  
    puts("");  
    return 0;  
}  
   
/** 
10 
72 6 57 88 60 42 83 73 48 85 
5 
*/  
```
### 排序
#### 二、冒泡排序
```c++
void paopao(void) //待排序的数据存放在a[1]..a[n]数组中
{
  for(int i=1;i<n;i++)  //控制循环（冒泡）的次数，n个数，需要n-1次冒泡
    for(int j=1;j<=n-i;j++) //相邻的两两比较
      if(a[j]<a[j+1]) 
      {
         int temp=a[j];
         a[j]=a[j+1];
         a[j+1]=temp;
      }
}
```
//或者
```c++
void BubbleSort3(int a[], int n)
{
	int j, k;
	int flag;
	
	flag = n;
	while (flag > 0)
	{
		k = flag;
		flag = 0;
		for (j = 1; j < k; j++)
			if (a[j - 1] > a[j])
			{
				Swap(a[j - 1], a[j]);
				flag = j;
			}
	}
}
```
//调用：paopao()，适用于n比较小的排序

#### 插入排序
#### 选择排序
#### 一、快速排序
```c++
void qsort(int x,int y) //待排序的数据存放在a[1]..a[n]数组中
{
   int h=x,r=y;
   int m=a[(x+y)>>1]; //取中间的那个位置的值
   while(h<r)
   {
      while (a[h]<m) h++; //比中间那个位置的值小，循环直到找一个比中间那个值大的
      while (a[r]>m) r--; //比中间那个位置的值大，循环直到找一个比中间那个值小的
      if(h<=r)
      {
         int temp=a[h];//如果此时h<=r，交换a[h]和a[r]
         a[h]=a[r];
         a[r]=temp;
         h++;r--; //这两句必不可少哦
      }
   }
   if(r>x) qsort(x,r); //注意此处，尾指针跑到前半部分了
   if(h<y) qsort(h,y); //注意此处，头指针跑到后半部分了
}
//调用：qsort(1,n)即可实现数组a中元素有序。适用于n比较大的排序
```
#### 计数排序
 
#### 三、桶排序
```c++
void bucketsort(void)//a的取值范围已知。如a<=cmax。
{
  memset(tong,0,sizeof(tong));//桶初始化
  for(int i=1;i<=n;i++)//读入n个数
  {
    int a
    cin>>a;
    tong[a]++;
  }//相应的桶号计数器加1
  for(int i=1;i<=cmax;i++)
  {
     if(tong[i]>0) //当桶中装的树大于0，说明i出现过tong[i]次，否则没出现过i
	 {
       while (tong[i]!=0)
       {
	     tong[i]--;
         cout<<i<<" ";
	    }
	
  }
}
```
//桶排序适用于那些待排序的关键字的值在已知范围的排序。

#### 基数排序
 
#### 四、合（归）并排序
```c++
void merge(int l,int m,int r)//合并[l,m]和[m+1,r]两个已经有序的区间
{ int b[101];//借助一个新的数组B，使两个有序的子区间合并成一个有序的区间，b数组的大小要注意
  int h,t,k;
  k=0;//用于新数组B的指针
  h=l;t=m+1;//让h指向第一个区间的第一个元素，t指向第二个区间的第一个元素。
  while((h<=m)&&(t<=r))//在指针h和t没有到区间尾时，把两个区间的元素抄在新数组中
    {k++;       //新数组指针加1
     if (a[h]<a[t]){b[k]=a[h];h++;}       //抄第一个区间元素到新数组
     else{b[k]=a[t];t++;}   //抄第二个区间元素到新数组
    }
  while(h<=m){k++;b[k]=a[h];h++;}  //如果第一个区间没有抄结束，把剩下的抄在新数组中
  while(t<=r){k++;b[k]=a[t];t++;}   //如果第二个区间没有抄结束，把剩下的抄在新数组中
  for(int o=1;o<=k;o++)//把新数组中的元素，再抄回原来的区间，这两个连续的区间变为有序的区间。
a[l+o-1]=b[o];
}
void mergesort(int x,int y)//对区间[x,y]进行二路归并排序
{
  int mid;
  if(x>=y) return;
  mid=(x+y)/2;//求[x,y]区间，中间的那个点mid,mid把x,y区间一分为二
  mergesort(x,mid);//对前一段进行二路归并
  mergesort(mid+1,y);//对后一段进行二路归并
  merge(x,mid,y);//把已经有序的前后两段进行合并
}
```
//归并排序应用了分治思想，把一个大问题，变成两个小问题。二分是分治的思想。
 
### 二分
#### 二分查找
```c++
int find(int x,int y,int m) //在[x,y]区间查找关键字等于m的元素下标
{ int head,tail,mid;
  head=x;tail=y;mid=((x+y)/2);//取中间元素下标
  if(a[mid]==m) return mid;//如果中间元素值为m返回中间元素下标mid
  if(head>tail) return 0;//如果x>y，查找失败，返回0
  if(m>a[mid])  //如果m比中间元素大，在后半区间查找，返回后半区间查找结果
    return find(mid+1,tail);
  else //如果m比中间元素小，在前半区间查找，返回后前区间查找结果
    return find(head,mid-1);
}
```
#### 二分答案
### 三分
### 字符串匹配
#### KMP（Knuth-Morris-Pratt）算法
### 高精度算法
#### 六、高精度加法
```c++
#include<iostream>
#include<cstring>
using namespace std;
int main()
{
  string str1,str2;
  int a[250],b[250],len;   //数组的大小决定了计算的高精度最大位数
  int i;
  memset(a,0,sizeof(a));
  memset(b,0,sizeof(b));
  cin>>str1>>str2;   //输入两个字符串
  a[0]=str1.length();  //取得第一个字符串的长度
  for(i=1;i<=a[0];i++)  //把第一个字符串转换为整数，存放在数组a中
    a[i]=str1[a[0]-i]-'0';
  b[0]=str2.length();   //取得第二个字符串长度
  for(i=1;i<=b[0];i++)   //把第二个字符串中的每一位转换为整数，存放在数组B中
    b[i]=str2[b[0]-i]-'0';
  len=(a[0]>b[0]?a[0]:b[0]);   //取两个字符串最大的长度
  for(i=1;i<=len;i++)   //做按位加法，同时处理进位
  {
    a[i]+=b[i];
    a[i+1]+=a[i]/10;
    a[i]%=10;   
  }
  len++;    //下面是去掉最高位的0，然后输出。
  while((a[len]==0)&&(len>1)) len--;
  for(i=len;i>=1;i--)
    cout<<a[i];
  return 0; 
}
```
//注意：两个数相加，结果的位数，应该比两个数中大的那个数多一位。
 
#### 七、高精度减法
```c++
#include<iostream>
using namespace std;
int compare(string s1,string s2);
int main()
{
  string str1,str2;
  int a[250],b[250],len;
  int i;
  memset(a,0,sizeof(a));
  memset(b,0,sizeof(b));
  cin>>str1>>str2;
  a[0]=str1.length();
  for(i=1;i<=a[0];i++)
    a[i]=str1[a[0]-i]-'0';
  b[0]=str2.length();
  for(i=1;i<=b[0];i++)
    b[i]=str2[b[0]-i]-'0';
  if((compare(str1,str2))==0)  //大于等于，做按位减，并处理借位。
  {
    for(i=1;i<=a[0];i++)
      {a[i]-=b[i];
       if (a[i]<0) {a[i+1]--;a[i]+=10;}
      }
    a[0]++;
    while((a[a[0]]==0)&&(a[0]>1)) a[0]--;
    for(i=a[0];i>=1;i--)
      cout<<a[i];
    cout<<endl; 
  }                          
  else
  {
    cout<<'-';  //小于就输出负号
    for(i=1;i<=b[0];i++)  //做按位减，大的减小的
      {b[i]-=a[i];
       if (b[i]<0) {b[i+1]--;b[i]+=10;}
      }
    b[0]++;
    while((b[b[0]]==0)&&(b[0]>1)) b[0]--;
    for(i=b[0];i>=1;i--)
      cout<<b[i];
    cout<<endl;        
  }
  return 0; 
}
int compare(string s1,string s2)  //比较字符串（两个数）数字的大小，大于等于返回0，小于返回1。
{
  if(s1.length()>s2.length()) return 0;  //先比较长度，哪个字符串长，对应的那个数就大
  if(s1.length()<s2.length()) return 1;
  for(int i=0;i<=s1.length();i++)  //长度相同时，就一位一位比较。
  {
    if(s1[i]>s2[i]) return 0;
    if(s1[i]<s2[i]) return 1;                          
  }
  return 0;   //如果长度相同，每一位也一样，就返回0，说明相等
}
``` 
//做减法时，首先要判断两个字符串的大小，决定是否输出负号，然后就是按位减法，注意处理借位。
 
#### 八、高精度乘法
```c++
#include<iostream>
#include<cstring>
using namespace std;
int main()
{
  string str1,str2;
  int a[250],b[250],c[500],len;    //250位以内的两个数相乘
  int i,j;
  memset(a,0,sizeof(a));
  memset(b,0,sizeof(b));
  cin>>str1>>str2;
  a[0]=str1.length();
  for(i=1;i<=a[0];i++)
    a[i]=str1[a[0]-i]-'0';
  b[0]=str2.length();
  for(i=1;i<=b[0];i++)
    b[i]=str2[b[0]-i]-'0';
  memset(c,0,sizeof(c));
  for(i=1;i<=a[0];i++)   //做按位乘法同时处理进位，注意循环内语句的写法。
    for(j=1;j<=b[0];j++)
    {
    c[i+j-1]+=a[i]*b[j];
    c[i+j]+=c[i+j-1]/10;
    c[i+j-1]%=10;   
    }
  len=a[0]+b[0]+1;  //去掉最高位的0，然后输出
  while((c[len]==0)&&(len>1)) len--;   //为什么此处要len>1??
  for(i=len;i>=1;i--)
    cout<<c[i];
  return 0; 
}
``` 
//注意：两个数相乘，结果的位数应该是这两个数的位数和减1。
//优化：万进制
```c++
#include<iostream>
#include<cstring>
using namespace std;
void num1(int s[],string st1);
int a[2501],b[2501],c[5002];//此处可以进行2500位万进制乘法，即10000位十进制乘法。
Int main()
{  
    string str1,str2;
    int len;
    cin>>str1>>str2;
    memset(a,0,sizeof(a));
    memset(b,0,sizeof(b));
    memset(c,0,sizeof(c));
    num1(a,str1); //把str1从最低位开始，每4位存放在数组a中
    num1(b,str2); //把str2从最低位开始，每4位存放在数组b中
    for(int i=1;i<=a[0];i++) //作按位乘法并处理进位，此处是万进制进位
      for(int j=1;j<=b[0];j++)
        {
          c[i+j-1]+=a[i]*b[j];
          c[i+j]+=c[i+j-1]/10000;
          c[i+j-1]%=10000;
        }
    len=a[0]+b[0];//a[0]和b[0]存放的是每个数按4位处理的位数
    while ((c[len]==0)&&(len>1)) len--;//去掉高位的0，并输出最高位
      cout<<c[len];
    for(int i=len-1;i>=1;i--)//把剩下来的每一位还原成4位输出
      {
        if (c[i]<1000) cout<<’0’;
        if (c[i]<100) cout<<’0’;
        if (c[i]<10) cout<<’0’;        
        cout<<c[i];
      }
    cout<<endl;
    return 0;
}
void num1(int s[],string st1)//此函数的作用就是把字符串st1，按4位一组存放在数组s中
{   int k=1,count=1;
    s[0]=st1.length();//存放st1的长度，省去一长度变量
    for(int i=s[0]-1;i>=0;i--) //从最低位开始，处理每一位
    { if (count%4==0) {s[k]+=(st1[i]-‘0’)*1000; if(i!=0) k++;}
      if (count%4==1) s[k]=(st1[i]-‘0’);
      if (count%4==2) s[k]+=(st1[i]-‘0’)*10;
      if (count%4==3) s[k]+=(st1[i]-‘0’)*100;
      count++;
    }
    s[0]=k; //存放数组的位数，就是按4位处理后的万进制数的位数。  
    Return;
}
``` 
#### 九、高精度除法（无代码）
 
#### 十、筛选法建立素数表
```c++
void maketable(int x)//建立X以内的素数表prim，prim[i]为0，表示i为素数，为1表示不是质数
{
 memset(prim,0,sizeof(prim));//初始化质数表
 prim[0]=1;prim[1]=1;prim[2]=0;//用筛选法求X以内的质数表
 for(int i=2;i<=x;i++)
    if (prim[i]==0)
     {int j=2*i;
      while(j<=x)
       {prim[j]=1;j=j+i;}
}
}
``` 
//对于那些算法中，经常要判断素数的问题，建立一个素数表，可以达到一劳永逸的目的。
 

#### 十七、并查集
```c++
int getfather(int x)//非递归求X结点的根结点的编号
{while(x!=father[x])
  x=father[x];
 return x;
}
 
int getfather(int x)//递归求X结点的根结点的编号
{if(x==father[x]) return x;
 else return getfather(father[x]); 
}
 
int getfather(int x)//非递归求X结点的根结点编号同时进行路径压缩
{int p=x;
while(p!=father[p])//循环结束后，P即为根结点
   p=father[p];
 while(x!=father[x])//从X结点沿X的父结点进行路径压缩
   {int temp=father[x];//暂存X没有修改前的父结点
father[x]=p;//把X的父结点指向P
x=temp;
   }
 return p;
}
 
int getfather(int x)//递归求X结点的根结点编号同时进行路径压缩
{if(x==father[x]) return x;
 else {
       int temp=getfather(father[x]);
       father[x]=temp;
       return temp;
}
}
 
void merge(int x,int y)//合并x,y两个结点
 {int x1,x2;
  x1=getfather(x);//取得X的父结点
  x2=getfather(y);//取得Y的父结点
  if(x1!=x2) father[x1]=x2; //两个父结点不同的话就合并，注意：合并的是X，Y两个结点的根。
}
``` 

 
#### 二十二、01背包问题
```c++
//n为物品的数量，w[i]表示第i个物品的重量，c[i]表示第i个物品的价值，v为背包的最大重量。
//有状态转移方程f[i][j]=max{f[i-1][j],f[i-1][j-w[i]]+c[i]}。f[i][j]表示前i个物品，在背包载重为j时获得的最大价值。显然f[n][v]即为所求。边界条件为f[0][s]=0,s=0,1,…,v。
for(int i=1;i<=n;i++)//枚举阶段
 for(int j=0;j<=v;j++)//枚举状态，当然此处也可写成:for(int j=v;j>=0;j--)
   {
f[i][j]=f[i-1][j];//不选第i个物品
if(f[i][j]<f[i-1][j-w[i]]+c[i]) f[i][j]=f[i-1][j-w[i]]+c[i];//选第i个物品
}
cout<<f[n][v]<<endl;//输出结果。
 
//优化：用一维数组实现，把第i-1阶段和第i阶段数据存在一块。
for(int i=1;i<=n;i++)//枚举阶段
 for(int j=v;j>=0;j--)//枚举状态，当然此处也可写成:for(int j=v;j>=0;j--)
   {
f[j]=f[j];//不选第i个物品，可省略此语句。
     if((j>w[i])&&(f[j]<f[j-w[i]]+c[i])) f[j]=f[j-w[i]]+c[i];//选第i个物品
}
cout<<f[v]<<endl;//输出结果。
``` 
//对比优化前后，我们不难发现，优化后的代码实际上就是在原来基本的代码基础上，减少了阶段这一维，同时在枚举状态时，为了保证结果的正确性，枚举的顺序只能是v到0，而不能是0到v。大家细想一下为什么？就是保证在求第i///阶段j状态时，f[j-w[i]]为第i-1阶段的值。
 
//进一步优化，在上面代码中，枚举状态时，还可以写成for(int j=v;j>=w[i];j--)，此时下面的判断条件j>=w[i]就可以省略了。
 
#### 二十三、完全背包问题

//和01背包问题不同的是，完全背包，对于任何一个物品i，只要背包重量允许，可以多次选取，也就是在决策上，可以选0个，1个，2个，…，v/w[i]个。
//状态转移方程f[i][j]=max{f[i-1][j],f[i-1][j-w[i]]+c[i]，f[i-1][j-2*w[i]]+2*c[i]，…，f[i-1][j-k*w[i]]+k*c[i]}。k=0，1，2，…，v/w[i]。f[i][j]表示前i个物品，在背包载重为j时获得的最大价值。显然f[n][v]即为所///求。边界条件为f[0][s]=0,s=0,1,…,v。
```c++
for(int i=1;i<=n;i++)//枚举阶段
 for(int j=0;j<=v;j++)//枚举状态，当然此处也可写成:for(int j=v;j>=0;j--)
    {f[i][j]=f[i-1][j];//k=0的情况作为f[i][j]的初始值，然后在k=1,2,…,v/w[i]中找最大值
    for(int k=1;k<=v/w[i];k++)  
if(f[i][j]<f[i-1][j-k*w[i]]+k*c[i]) f[i][j]=f[i-1][j-k*w[i]]+k*c[i];//选第i个物品
}
cout<<f[n][v]<<endl;//输出结果。
``` 
#### 二十四、多属性背包问题
 
#### 二十五、多背包问题
 

   
### 动态规划
动态规划算法通常用于求解具有某种最优性质的问题。在这类问题中，可能会有许多可靠解。每一个解都对应于一个值，希望找到具有最优值（最大值或最小值）的解。
动态规划算法的有效性依赖于问题的两个重要性质：最优子结构和子问题重叠性质。
1 、最优子结构性质。如果问题的最优解所包含的子问题的解也是最优的，我们就称该问题具有最优子结构性质（ 即满足最优化原理）。最优子结构性质为动态规划算法解决问题提供了重要线索。 
2 、子问题重叠性质。子问题重叠性质是指在用递归算法自顶向下对问题进行求解时，每次产生的子问题并不总是新问题，有些子问题会被重复计算多次。动态规划算法正是利用了这种子问题的重叠性质，对每一个子问题只计算一次，然后将其计算结果保存在一个表格中，当再次需要计算已经计算过的子问题时，只是在表格中简单地查看一下结果，从而获得较高的解题效率。

例：最大子段和。给出一段序列，选出其中连续且非空的一段使得这段和最大。([洛谷P1115](https://www.luogu.org/problem/show?pid=1115))
**以下代码不考虑最大子段和为负的情况**
1、简单算法$O(n^3)$
```C++
template<class T>
T maxsum(T a[],int n,int &start,int &end)
//返回数组a的最大子段和，并用start和end返回子段的位置
{
    T sum=0;//最大子段和
	for(int  i=0;i<n;i++)
	{//子段起始位置
		for(int j=i;j<n;j++)
		{//子段结束位置
			T subSum=0;//子段和
			for(int k=i;i<=j;k++)subSum+=a[k];//求子段和 
			if(subSum>sum)
			{//修改最大子段和及位置 
				sum=subSum;
				start=i;
				end=j;
			}
		}
	}
	return sum;
}
```
2、改进算法$O(n^2)$
通过如下式子，可省去最后一个for循环。

$$\sum_{k=i}^j a_k=a_j+\sum_{k=i}^ {j-1} a_k$$
```c++
template<class T>
T maxsum(T a[],int n,int &start,int &end)
//返回数组a的最大子段和，并用start和end返回子段的位置
{
    T sum=0;//最大子段和
	for(int  i=0;i<n;i++)
	{//子段起始位置
		T subSum=0;//子段和
		for(int j=i;j<n;j++)
		{//子段结束位置
			subSum+=a[j]; 
			if(subSum>sum)
			{//修改最大子段和及位置 
				sum=subSum;
				start=i;
				end=j;
			}
		}
	}
	return sum;
}
```
3、动态规划算法$O(n)$
定义b[j]为前j项的最大子段和，则
b[j]=max{b[j-1]+a[j],a[j]}
即当b[j-1]>0时，b[j]=b[j-1]+a[j]，否则b[j]=a[j]。
```c++
template<class T>
template<class T>
T maxsum(T a[],int n,int &start,int &end)
//返回数组a的最大子段和，并用start和end返回子段的位置
{
    T sum=0;//最大子段和
    T bsum=0;//上界为i的最大子段和 
    int bstart=0,bend=n-1;//注意此处应该初始化
	for(int  i=0;i<n;i++)
	{//子段起始位置
		if(bsum>0)
		{
			bsum+=a[i];
			bend=i;
		}
		else 
		{
			bsum=a[i];
			bstart=i;
		}
		if(bsum>sum)
		{
			sum=bsum;
			start=bstart;
			end=bend;
		}
	}
	
	return sum;
}
```
#### 线型动态规划
最长不下降子序列（[导弹拦截NOIP1999](https://www.luogu.org/problem/show?pid=1020)、[合唱队形NOIP2004](https://www.luogu.org/problem/show?pid=1091)）
最长公共子序列([洛谷P3402](https://www.luogu.org/problem/show?pid=3402),[洛谷P2516](https://www.luogu.org/problem/show?pid=2516))
二十六、最长不降（上升）子序列问题
 
   //f[i]表示从第1个数开始，以第i个数结尾的最长递增子序列。
 
//状态转移方程：f[i]=max{f[j]}+1 （1≤j≤i-1，1≤i≤n，a[i]≥a[j]）
//临界状态：f[1]=1;
 
二十七、最长公共子序列问题
 
   //f[i][j]表示第一个串前i个字符和第二个串前j个字符的最长公共子序列数。
   //状态转移方程： 
       f[i-1][j-1]                 (若a[i]==b[j])
f[i][j]=
                          max{f[i-1][j],f[i][j-1]}+1     (若a[i]≠b[j])
 
   //临界状态:f[0][j]=0，f[i][0]=0

#### 资源背包动态规划
01背包（装箱问题、采药问题、开心的金明）、完全背包、多重背包、分组背包、带附件的背包（金明的预算方案）

#### 区间类型动态规划
石子合并、能量项链、乘积最大

#### 树型动态规划
参考网址：
http://blog.csdn.net/txl199106/article/details/45372337
加分二叉树
题目：
poj2342
hdu1520
poj1192
hdu1561 树形DP The More The Better
####子集动态规划
####状态压缩动态规划
####坐标规则型动态规划
数塔、过河卒

#### 多进程动态规划
方格取数、传纸条

#### 动态规划的优化
##### 四边形不等式
##### 函数的凸凹性
##### 状态设计
##### 规划方向



### 递归与分治策略
当我们求解某些问题时，由于这些问题要处理的数据相当多，或求解过程相当复杂，使得直接求解法在时间上相当长，或者根本无法直接求出。对于这类问题，我们往往先把它分解成几个子问题，找到求出这几个子问题的解法后，再找到合适的方法，把它们组合成求整个问题的解法。如果这些子问题还较大，难以解决，可以再把它们分成几个更小的子问题，以此类推，直至可以直接求出解为止。这就是分治策略的基本思想。
分治法解题的一般步骤：
（1）分解，将要解决的问题划分成若干规模较小的同类问题；
（2）求解，当子问题划分得足够小时，用较简单的方法解决；
（3）合并，按原问题的要求，将子问题的解逐层合并构成原问题的解。
### 贪心算法
### 回溯法
八皇后问题
骑士周游问题
24点问题
N个数的排列与组合
### 分枝限界法
### 线性规划与网络流
#### 最大网络流问题
##### 网络与流
##### 增广路算法
##### 预流推进算法
##### 最大流问题的变换与应用
#### 最小费用流问题
##### 最小费用流
##### 消圈算法
##### 最小费用路算法
##### 网络单纯形算法
##### 最小流问题的变换与应用
### 概率算法
### 树链剖分
### 树上倍增
### 求最值的ST算法
### Tarjan算法
#### 双连通分量
#### 最小割
### 最近公共祖先（LCA——ST算法）
LCA（Least Common Ancestors），即最近公共祖先，是指这样一个问题：在有根树中，找出某两个结点u和v最近的公共祖先（另一种说法，离树根最远的公共祖先）。
对于该问题，最容易想到的算法是分别从节点u和v回溯到根节点，获取u和v到根节点的路径P1，P2，其中P1和P2可以看成两条单链表，这就转换成常见的一道面试题：【判断两个单链表是否相交，如果相交，给出相交的第一个点。】。该算法总的复杂度是O（n）（其中n是树节点个数）。
本节介绍了两种比较高效的算法解决这个问题，其中一个是在线算法（DFS+ST），另一个是离线算法（Tarjan算法）。
在线算法DFS+ST描述(思想是：将树看成一个无向图，u和v的公共祖先一定在u与v之间的最短路径上)：
（1）DFS：从树T的根开始，进行深度优先遍历（将树T看成一个无向图），并记录下每次到达的顶点。第一个的结点是root(T)，每经过一条边都记录它的端点。由于每条边恰好经过2次，因此一共记录了2n-1个结点，用E[1, ... , 2n-1]来表示。
（2）计算R：用R[i]表示E数组中第一个值为i的元素下标，即如果R[u] < R[v]时，DFS访问的顺序是E[R[u], R[u]+1, …, R[v]]。虽然其中包含u的后代，但深度最小的还是u与v的公共祖先。
（3）RMQ：当R[u] ≥ R[v]时，LCA[T, u, v] = RMQ(L, R[v], R[u])；否则LCA[T, u, v] = RMQ(L, R[u], R[v])，计算RMQ。
由于RMQ中使用的ST算法是在线算法，所以这个算法也是在线算法。
【举例说明】
T=<V,E>，其中V={A,B,C,D,E,F,G},E={AB,AC,BD,BE,EF,EG},且A为树根。则图T的DFS结果为：A->B->D->B->E->F->E->G->E->B->A->C->A，要求D和G的最近公共祖先, 则LCA[T, D, G] = RMQ(L, R[D], R[G])= RMQ(L, 3, 8)，L中第4到7个元素的深度分别为：1,2,3,3，则深度最小的是B。
离线算法（Tarjan算法）描述：
所谓离线算法，是指首先读入所有的询问（求一次LCA叫做一次询问），然后重新组织查询处理顺序以便得到更高效的处理方法。Tarjan算法是一个常见的用于解决LCA问题的离线算法，它结合了深度优先遍历和并查集，整个算法为线性处理时间。
Tarjan算法是基于并查集的，利用并查集优越的时空复杂度，可以实现LCA问题的O(n+Q)算法，这里Q表示询问 的次数。更多关于并查集的资料，可阅读这篇文章：《数据结构之并查集》。
同上一个算法一样，Tarjan算法也要用到深度优先搜索，算法大体流程如下：对于新搜索到的一个结点，首先创建由这个结点构成的集合，再对当前结点的每一个子树进行搜索，每搜索完一棵子树，则可确定子树内的LCA询问都已解决。其他的LCA询问的结果必然在这个子树之外，这时把子树所形成的集合与当前结点的集合合并，并将当前结点设为这个集合的祖先。之后继续搜索下一棵子树，直到当前结点的所有子树搜索完。这时把当前结点也设为已被检查过的，同时可以处理有关当前结点的LCA询问，如果有一个从当前结点到结点v的询问，且v已被检查过，则由于进行的是深度优先搜索，当前结点与v的最近公共祖先一定还没有被检查，而这个最近公共祖先的包涵v的子树一定已经搜索过了，那么这个最近公共祖先一定是v所在集合的祖先。
算法伪代码：
```
LCA(u)
{
Make-Set(u)
ancestor[Find-Set(u)]=u
对于u的每一个孩子v
{
LCA(v)
Union(u,v)
ancestor[Find-Set(u)]=u
}
checked[u]=true
对于每个(u,v)属于P // (u,v)是被询问的点对
{
if checked[v]=true
 
then {
回答u和v的最近公共祖先为ancestor[Find-Set(v)]
}
      }
}
```
【举例说明】 
根据实现算法可以看出，只有当某一棵子树全部遍历处理完成后，才将该子树的根节点标记为黑色（初始化是白色），假设程序按上面的树形结构进行遍历，首先从节点1开始，然后递归处理根为2的子树，当子树2处理完毕后，节点2， 5， 6均为黑色；接着要回溯处理3子树，首先被染黑的是节点7（因为节点7作为叶子不用深搜，直接处理），接着节点7就会查看所有询问(7, x)的节点对，假如存在(7, 5)，因为节点5已经被染黑，所以就可以断定(7, 5)的最近公共祖先就是find(5).ancestor，即节点1（因为2子树处理完毕后，子树2和节点1进行了union，find(5)返回了合并后的树的根1，此时树根的ancestor的值就是1）。有人会问如果没有(7, 5)，而是有(5, 7)询问对怎么处理呢? 我们可以在程序初始化的时候做个技巧，将询问对(a, b)和(b, a)全部存储，这样就能保证完整性。


CODEVS 2370 小机房的树 [传送门](http://codevs.cn/problem/2370/)
CODEVS 1036 商务旅行 [传送门](http://codevs.cn/problem/1036/)
METO CODE 223 拉力赛 [传送门](http://oj.fjaxyz.com:3389/problem.php?id=223/)
HDU 2586 How far way? [传送门](http://acm.hdu.edu.cn/showproblem.php?pid=2586)
ZOJ 3195 Design the city [传送门](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=3195)
　　　　　　　　　　
### 区间最值查询（RMQ——ST算法）
RMQ（Range Minimum/Maximum Query），即区间最值查询，是指这样一个问题：对于长度为n的数列A，回答若干询问RMQ（A,i,j）(i,j<=n)，返回数列A中下标在i，j之间的最小/大值。
对于该问题，最容易想到的解决方案是遍历，复杂度是O(n)。但当数据量非常大且查询很频繁时，该算法也许会存在问题。
本节介绍了一种比较高效的在线算法（ST算法）解决这个问题。所谓在线算法，是指用户每输入一个查询便马上处理一个查询。该算法一般用较长的时间做预处理，待信息充足以后便可以用较少的时间回答每个查询。ST（Sparse Table）算法是一个非常有名的在线处理RMQ问题的算法，它可以在O(nlogn)时间内进行预处理，然后在O(1)时间内回答每个查询。
首先是预处理，用动态规划（DP）解决。设A[i]是要求区间最值的数列，F[i, j]表示从第i个数起连续2^j个数中的最大值。例如数列3 2 4 5 6 8 1 2 9 7，F[1，0]表示第1个数起，长度为2^0=1的最大值，其实就是3这个数。 F[1，2]=5，F[1，3]=8，F[2，0]=2，F[2，1]=4……从这里可以看出F[i,0]其实就等于A[i]。这样，DP的状态、初值都已经有了，剩下的就是状态转移方程。我们把F[i，j]平均分成两段（因为f[i，j]一定是偶数个数字），从i到i+2^(j-1)-1为一段，i+2^(j-1)到i+2^j-1为一段(长度都为2^（j-1）)。用上例说明，当i=1，j=3时就是3,2,4,5 和 6,8,1,2这两段。F[i，j]就是这两段的最大值中的最大值。于是我们得到了动态规划方程F[i, j]=max（F[i，j-1], F[i + 2^(j-1)，j-1]）。
然后是查询。取k=[log2(j-i+1)]，则有：RMQ(A, i, j)=min{F[i,k],F[j-2^k+1,k]}。 举例说明，要求区间[2，8]的最大值，就要把它分成[2,5]和[5,8]两个区间，因为这两个区间的最大值我们可以直接由f[2，2]和f[5，2]得到。
算法伪代码：
```
//初始化


INIT_RMQ
 
//max[i][j]中存的是重j开始的2^i个数据中的最大值，最小值类似，num中存有数组的值
 
for i : 1 to n
 
  max[0][i] = num[i]
 
for i : 1 to log(n)/log(2)
 
  for j : 1 to (n+1-2^i)
 
     max[i][j] = MAX(max[i-1][j], max[i-1][j+2^(i-1)]
 
//查询
 
RMQ(i, j)
 
k = log(j-i+1) / log(2)
 
return MAX(max[k][i], max[k][j-2^k+1])
```
当然，该问题也可以用线段树（也叫区间树）解决，算法复杂度为：O(N)~O(logN)，具体可阅读这篇文章：《数据结构之线段树》。

### Google方程式
### 朱刘算法（省选）
最小有向生成树：给定一个有向带权图G和其中一个点u，找出一个以u为跟结点，权和最小的有向生成树。有向生成树也叫树形图，是指一个类似树的有向图，满足以下条件：

 
1.恰好有一个入度为0的点，称为根结点 
2.其他结点的入度均为1 
3.可以从根结点到达其他结点

算法的主过程如下: 
1.找到除了root以为其他点的权值最小的入边。用In[i]记录 
2.如果出现除了root以为存在其他孤立的点，则不存在最小树形图。 
3.找到图中所有的环，并对环进行缩点，重新编号。 
4.更新其他点到环上的点的距离 
5.重复3，4直到没有环为止。

```C++
#include <cstdio>#include <cstring>

const int MAXNODE = 1010;
const int MAXEDGE = 100010;
typedef int Type;
const Type INF = 0x3f3f3f3f;
struct Edge {    
int u, v;
    Type dis;
    Edge() {}
    Edge(int u, int v, Type dis): u(u), v(v), dis(dis) {}
};
struct Directed_MT{    
    int n, m;
   Edge edges[MAXEDGE];    
    int vis[MAXNODE];    
    int pre[MAXNODE];    
    int id[MAXNODE];
    Type in[MAXNODE];    
        void init(int n) {        
            this->n = n;
        m = 0;
    }    
            void AddEdge(int u, int v, Type dis) {
        edges[m++] = Edge(u, v, dis);
    }

    Type DirMt(int root) {
        Type ans = 0;        
                while (1) {            //初始化
            for (int i = 0; i < n; i++) in[i] = INF;            for (int i = 0; i < m; i++) {                int u = edges[i].u;                int v = edges[i].v;                //找寻最小入边，删除自环
                if (edges[i].dis < in[v] && u != v) {
                    in[v] = edges[i].dis;
                    pre[v] = u;
                }
            } //如果没有最小入边，表示该点不连通，则最小树形图形成失败
            for (int i = 0; i < n; i++) {                if (i == root) continue;                if (in[i] == INF) return -1;
            }            
                      int cnt = 0;//记录缩点
                memset(id, -1, sizeof(id));
                memset(vis, -1, sizeof(vis));
                in[root] = 0;//树根不能有入边
            for (int i = 0; i < n; i++) {
                ans += in[i];
                        int v = i;       //找寻自环
                while (vis[v] != i && 
                        id[v] == -1 
                        && v != root) {
                    vis[v] = i;
                    v = pre[v];
                }                //找到自环
                if (v != root && id[v] == -1) {                    //这里不能从i开始找，因为i有可能不在自环内
                    for (int u = pre[v]; 
                                u != v; u = pre[u]) 
                      id[u] = cnt;                    id[v] = cnt++;
                }
            }     //如果没有自环了，表示最小树形图形成成功了
            if (cnt == 0) break;            //找到那些不是自环的，重新给那些点进行标记
            for (int i = 0; i < n; i++) 
                if (id[i] == -1) id[i] = cnt++;            for (int i = 0; i < m; i++) {                int v = edges[i].v;
                edges[i].v = id[edges[i].v];
                edges[i].u = id[edges[i].u];                if (edges[i].u != edges[i].v) 
                    edges[i].dis -= in[v];
            }    //缩点完后，点的数量就边了
            n = cnt;
            root = id[root];
        }        return ans;
    }
}MT;
int main() {    
return 0;
}
```
### 平衡树（Treap+Splay+SBT）（省选）
#### Treap树

核心是 利用随机数的二叉排序树的各种操作复杂度平均为O(lgn)

Treap模板：

```C++
#include <cstdio>
#include <cstring>
#include <ctime>
#include <iostream>
#include <algorithm>
#include <cstdlib>
#include <cmath>
#include <utility>
#include <vector>
#include <queue>
#include <map>
#include <set>
#define max(x,y) ((x)>(y)?(x):(y))
#define min(x,y) ((x)>(y)?(y):(x))
#define INF 0x3f3f3f3f
#define MAXN 100005
using namespace std;
int cnt=1,rt=0; //节点编号从1开始struct Tree
{	int key, size, pri, son[2]; //保证父亲的pri大于儿子的pri
	void set(int x, int y, int z)
	{
		key=x;
		pri=y;
		size=z;
		son[0]=son[1]=0;
	}
}T[MAXN];

void rotate(int p, int &x){	

        int y=T[x].son[!p];
	T[x].size=T[x].size-T[y].size+T[T[y].son[p]].size;
	T[x].son[!p]=T[y].son[p];
	T[y].size=T[y].size-T[T[y].son[p]].size+T[x].size;
	T[y].son[p]=x;
	x=y;
}void ins(int key, int &x){	

        if(x == 0)
		T[x = cnt++].set(key, rand(), 1);	

         else
	{
		T[x].size++;		
                int p=key < T[x].key;
		ins(key, T[x].son[!p]);		
                if(T[x].pri < T[T[x].son[!p]].pri)
			rotate(p, x);
	}
}

void del(int key, int &x) //删除值为key的节点{	if(T[x].key == key)
	{		if(T[x].son[0] && T[x].son[1])
		{			

int p=T[T[x].son[0]].pri > T[T[x].son[1]].pri;
			rotate(p, x);
			del(key, T[x].son[p]);
		}		else
		{			if(!T[x].son[0])
				x=T[x].son[1];			

else
				x=T[x].son[0];
		}
	}	else
	{
		T[x].size--;		int p=T[x].key > key;
		del(key, T[x].son[!p]);
	}
}int find(int p, int &x) //找出第p小的节点的编号{	if(p == T[T[x].son[0]].size+1)		return x;	if(p > T[T[x].son[0]].size+1)
		find(p-T[T[x].son[0]].size-1, T[x].son[1]);	else
		find(p, T[x].son[0]);
}int find_NoLarger(int key, int &x) //找出值小于等于key的节点个数{	if(x == 0)		return 0;	if(T[x].key <= key)		return T[T[x].son[0]].size+1+find_NoLarger(key, T[x].son[1]);	else
		return find_NoLarger(key, T[x].son[0]);	
}
```
相关题解：

POJ 3481 treap

POJ 1442 treap

POJ 2352 treap

#### Splay Tree（伸展树）

核心就是 过程Splay(x, y)，即将x节点转移到y节点的子节点上面（其中y是x的祖先）。

利用其中双旋的优势能够保证查询复杂度均摊为O(lgn)

一开始理解有些困难，其实实际上不做深入的理解就是，双旋的过程就是一个建立相对平衡的二叉树的一个过程。

》对于二叉树，最极端的情况就是线性插入，使得整棵二叉树退化为一条链。比如你查询链的最后一个节点，之后再次查询第一个节点。

1）若只是单旋通过Splay(x, 0)将最后一个节点移动到根节点，需要O(n)复杂度，而查询第一个节点时又需要O(n)复杂度，来来往往就退化成一条链了。

2）若是双旋Splay(x, 0)将最后一个节点移动到根节点上时，移动过程中建立起了相对平衡的二叉树，需要O(n)，也就是查询第一个节点时，大概是需要O(lgn)复杂度。这就降低了复杂度。可以证明，总的每个操作的均摊复杂度是O(lgn)。

具体证明可以参见 杨思雨《伸展树的基本操作与应用》

I 用于维护单调队列 ：（以key为维护对象保证单调）

常用版：（支持相同值）

```C++

Struct Tree{

int key, size, fa, son[2];

}

void PushUp(int x);

void Rotate(int x, int p); //0左旋 1右旋

void Splay(int x, int To) //将x节点插入到To的子节点中

int find(int key) //返回值为key的节点 若无返回0 若有将其转移到根处

int prev() //返回比根值小的最大值 若无返回0 若有将其转移到根处

int succ() //返回比根值大的最小值 若无返回0 若有将其转移到根处

void Insert(int key) //插入key 并且将该节点转移到根处

void Delete(int key) //删除值为key的节点 若有重点只删其中一个 x的前驱移动到根处

int GetPth(int p) //获得第p小的节点 并将其转移到根处

int GetRank(int key) //获得值<=key的节点个数 并将其转移到根处 若<key只需将<=换为<

模板：


int cnt=1, rt=0;struct Tree
{	int key, size, fa, son[2];	void set(int _key, int _size, int _fa)
	{
		key=_key;
		size=_size;
		fa=_fa;
		son[0]=son[1]=0;
	}
}T[MAXN];

inline void PushUp(int x)
{	T[x].size=T[T[x].son[0]].size+T[T[x].son[1]].size+1;
}

inline void Rotate(int x, int p) //0左旋 1右旋{	int y=T[x].fa;	T[y].son[!p]=T[x].son[p];	T[T[x].son[p]].fa=y;	T[x].fa=T[y].fa;	if(T[x].fa)		T[T[x].fa].son[T[T[x].fa].son[1] == y]=x;	T[x].son[p]=y;	T[y].fa=x;
	PushUp(y);
	PushUp(x);
}void Splay(int x, int To) //将x节点插入到To的子节点中{	while(T[x].fa != To)
	{		if(T[T[x].fa].fa == To)
			Rotate(x, T[T[x].fa].son[0] == x);		else
		{			int y=T[x].fa, z=T[y].fa;			int p=(T[z].son[0] == y);			if(T[y].son[p] == x)
				Rotate(x, !p), Rotate(x, p); //之字旋
			else
				Rotate(y, p), Rotate(x, p); //一字旋		}
	}	if(To == 0) rt=x;
}int find(int key) //返回值为key的节点 若无返回0 若有将其转移到根处{	int x=rt;	while(x && T[x].key != key)
		x=T[x].son[key > T[x].key];	if(x) Splay(x, 0);	return x;
}int prev() //返回比根值小的最大值 若无返回0 若有将其转移到根处{	int x=T[rt].son[0];	if(!x) return 0;	while(T[x].son[1])
		x=T[x].son[1];
	Splay(x, 0);	return x;
}int succ() //返回比根值大的最小值 若无返回0 若有将其转移到根处{	int x=T[rt].son[1];	if(!x) return 0;	while(T[x].son[0])
		x=T[x].son[0];
	Splay(x, 0);	return x;
}void Insert(int key) //插入key 并且将该节点转移到根处{	if(!rt)		T[rt = cnt++].set(key, 1, 0);	else
	{		int x=rt, y=0;		while(x)
		{
			y=x;
			x=T[x].son[key > T[x].key];
		}		T[x = cnt++].set(key, 1, y);		T[y].son[key > T[y].key]=x;
		Splay(x, 0);
	}
}void Delete(int key) //删除值为key的节点 若有重点只删其中一个 x的前驱移动到根处{	int x=find(key);	if(!x) return;	int y=T[x].son[0];	while(T[y].son[1])
		y=T[y].son[1];	int z=T[x].son[1];	while(T[z].son[0])
		z=T[z].son[0];	if(!y && !z)
	{
		rt=0;		return;
	}	if(!y)
	{
		Splay(z, 0);		T[z].son[0]=0;
		PushUp(z);		return;
	}	if(!z)
	{
		Splay(y, 0);		T[y].son[1]=0;
		PushUp(y);		return;
	}
	Splay(y, 0);
	Splay(z, y);	T[z].son[0]=0;
	PushUp(z);
	PushUp(y);
}int GetPth(int p) //获得第p小的节点 并将其转移到根处{	if(!rt) return 0;	int x=rt, ret=0;	while(x)
	{		if(p == T[T[x].son[0]].size+1)			break;		if(p>T[T[x].son[0]].size+1)
		{
			p-=T[T[x].son[0]].size+1;
			x=T[x].son[1];
		}		else
			x=T[x].son[0];
	}
	Splay(x, 0);	return x;
}int GetRank(int key) //获得值<=key的节点个数 并将其转移到根处 若<key只需将<=换为<{	if(!rt) return 0;	int x=rt, ret=0, y;	while(x)
	{
		y=x;		if(T[x].key <= key)
		{
			ret+=T[T[x].son[0]].size+1;
			x=T[x].son[1];
		}		else
			x=T[x].son[0];
	}
	Splay(y, 0);	return ret;
}
```
完全版：（支持相同值，支持区间删除，支持懒惰标记）

```C++
Struct Tree{

int key, num, size, fa, son[2];

}

void PushUp(int x);

void PushDown(int x);

int Newnode(int key, int fa); //新建一个节点并返回

void Rotate(int x, int p); //0左旋 1右旋

void Splay(int x, int To); //将x节点移动到To的子节点中

int GetPth(int p, int To); //返回第p小的节点 并移动到To的子节点中

int Find(int key); //返回值为key的节点 若无返回0 若有将其转移到根处

int Prev(); //返回根节点的前驱

int Succ(); //返回根结点的后继

void Insert(int key); //插入key值

void Delete(int key); //删除值为key的节点

int GetRank(int key); //获得值<=key的节点个数

void Delete(int l, int r); //删除值在[l, r]中的节点




int cnt, rt;int Add[MAXN];struct Tree{	int key, num, size, fa, son[2];
}T[MAXN];

inline void PushUp(int x)
{	T[x].size=T[T[x].son[0]].size+T[T[x].son[1]].size+T[x].num;
}

inline void PushDown(int x)
{	if(Add[x])
	{		if(T[x].son[0])
		{			T[T[x].son[0]].key+=Add[x];
			Add[T[x].son[0]]+=Add[x];
		}		if(T[x].son[1])
		{			T[T[x].son[1]].key+=Add[x];
			Add[T[x].son[1]]+=Add[x];
		}
		Add[x]=0;
	}
}

inline int Newnode(int key, int fa) //新建一个节点并返回{	++cnt;	T[cnt].key=key;	T[cnt].num=T[cnt].size=1;	T[cnt].fa=fa;	T[cnt].son[0]=T[cnt].son[1]=0;	return cnt;
}

inline void Rotate(int x, int p) //0左旋 1右旋{	int y=T[x].fa;
	PushDown(y);
	PushDown(x);	T[y].son[!p]=T[x].son[p];	T[T[x].son[p]].fa=y;	T[x].fa=T[y].fa;	if(T[x].fa)		T[T[x].fa].son[T[T[x].fa].son[1] == y]=x;	T[x].son[p]=y;	T[y].fa=x;
	PushUp(y);
	PushUp(x);
}void Splay(int x, int To) //将x节点移动到To的子节点中{	while(T[x].fa != To)
	{		if(T[T[x].fa].fa == To)
			Rotate(x, T[T[x].fa].son[0] == x);		else
		{			int y=T[x].fa, z=T[y].fa;			int p=(T[z].son[0] == y);			if(T[y].son[p] == x)
				Rotate(x, !p), Rotate(x, p); //之字旋
			else
				Rotate(y, p), Rotate(x, p); //一字旋		}
	}	if(To == 0) rt=x;
}int GetPth(int p, int To) //返回第p小的节点 并移动到To的子节点中{	if(!rt || p > T[rt].size) return 0;	int x=rt;	while(x)
	{
		PushDown(x);		if(p >= T[T[x].son[0]].size+1 && p <= T[T[x].son[0]].size+T[x].num)			break;		if(p > T[T[x].son[0]].size+T[x].num)
		{
			p-=T[T[x].son[0]].size+T[x].num;
			x=T[x].son[1];
		}		else
			x=T[x].son[0];
	}
	Splay(x, 0);	return x;
}int Find(int key) //返回值为key的节点 若无返回0 若有将其转移到根处{	if(!rt) return 0;	int x=rt;	while(x)
	{
		PushDown(x);		if(T[x].key == key) break;
		x=T[x].son[key > T[x].key];
	}	if(x) Splay(x, 0);	return x;
}int Prev() //返回根节点的前驱 非重点{	if(!rt || !T[rt].son[0]) return 0;	int x=T[rt].son[0];	while(T[x].son[1])
	{
		PushDown(x);
		x=T[x].son[1];
	}
	Splay(x, 0);	return x;
}int Succ() //返回根结点的后继 非重点{	if(!rt || !T[rt].son[1]) return 0;	int x=T[rt].son[1];	while(T[x].son[0])
	{
		PushDown(x);
		x=T[x].son[0];
	}
	Splay(x, 0);	return x;
}void Insert(int key) //插入key值{	if(!rt)
		rt=Newnode(key, 0);	else
	{		int x=rt, y=0;		while(x)
		{
			PushDown(x);
			y=x;			if(T[x].key == key)
			{				T[x].num++;				T[x].size++;				break;
			}			T[x].size++;
			x=T[x].son[key > T[x].key];
		}		if(!x)
			x=T[y].son[key > T[y].key]=Newnode(key, y);
		Splay(x, 0);
	}
}void Delete(int key) //删除值为key的节点1个{	int x=Find(key);	if(!x) return;	if(T[x].num>1)
	{		T[x].num--;
		PushUp(x);		return;
	}	int y=T[x].son[0];	while(T[y].son[1])
		y=T[y].son[1];	int z=T[x].son[1];	while(T[z].son[0])
		z=T[z].son[0];	if(!y && !z)
	{
		rt=0;		return;
	}	if(!y)
	{
		Splay(z, 0);		T[z].son[0]=0;
		PushUp(z);		return;
	}	if(!z)
	{
		Splay(y, 0);		T[y].son[1]=0;
		PushUp(y);		return;
	}
	Splay(y, 0);
	Splay(z, y);	T[z].son[0]=0;
	PushUp(z);
	PushUp(y);
}int GetRank(int key) //获得值<=key的节点个数{	if(!Find(key))
	{
		Insert(key);		int tmp=T[T[rt].son[0]].size;
		Delete(key);		return tmp;
	}	else
		return T[T[rt].son[0]].size+T[rt].num;
}void Delete(int l, int r) //删除值在[l, r]中的所有节点 l!=r{	if(!Find(l)) Insert(l);	int p=Prev();	if(!Find(r)) Insert(r);	int q=Succ();	if(!p && !q)
	{
		rt=0;		return;
	}	if(!p)
	{		T[rt].son[0]=0;
		PushUp(rt);		return;
	}	if(!q)
	{
		Splay(p, 0);		T[rt].son[1]=0;
		PushUp(rt);		return;
	}
	Splay(p, q);	T[p].son[1]=0;
	PushUp(p);
	PushUp(q);
}
```
(经测NOI2004郁闷的出纳员 POJ3481 POJ2352 POJ1442)

速度相对来说都还不错，POJ这些都3~500ms，郁闷的出纳员900多ms

相关题解：

HNOI 2002 营业额统计

POJ 3481 splay

POJ 2352 splay

POJ 1442 splay

NOI2004 郁闷的出纳员

II 用于维护序列： （以序列下标为对象维护，相当于对区间操作）（能够完成线段树的操作及其不能完成的操作）

```C++

Struct Tree{

int key, sum, size, fa, son[2];

}
```
支持操作：
```c++
void PushUp(int x);

void PushDown(int x);

int MakeTree(int l, int r, int a[]); //新建一个子树返回根节点

void Rotate(int x, int p); //0左旋 1右旋

void Splay(int x, int To); //将x节点移动到To的子节点中

int Select(int p, int To); //将第p个数移动到To的子节点中 并返回该节点

int Find(int key); //返回值为key的节点 若无返回0 若有将其转移到根处

int Prev(); //返回根节点的前驱

int Succ(); //返回根结点的后继

void Insert(int p, int l, int r, int a[]) //将a[l .. r]的数插入到下标为p后面

void Delete(int l, int r); //删除区间[l, r]中的节点

int Query(int l, int r); //返回[l, r]的和
```
待补充。。

#### Size Balance Tree

和上述两种二叉树比起来，SBT可能是最像真正平衡二叉树吧。

SBT能够保证树的高度在lgn，这样对于插入，删除操作都能够准确保证时间复杂度在O(lgn)

Maintain操作事实上理解起来也是挺简单的，至于证明参见CQF神牛的 《SBT》

```C++
int cnt, rt;struct Tree
{	int key, size, son[2];
}T[MAXN];

inline void PushUp(int x)
{	T[x].size=T[T[x].son[0]].size+T[T[x].son[1]].size+1;
}

inline int Newnode(int key)
{	++cnt;	T[cnt].key=key;	T[cnt].size=1;	T[cnt].son[0]=T[cnt].son[1]=0;	return cnt;
}void Rotate(int p, int &x)
{	int y=T[x].son[!p];	T[x].son[!p]=T[y].son[p];	T[y].son[p]=x;
	PushUp(x);
	PushUp(y);
	x=y;
}void Maintain(int &x, int p) //维护SBT的!p子树{	if(T[T[T[x].son[p]].son[p]].size > T[T[x].son[!p]].size)
		Rotate(!p, x);	else if(T[T[T[x].son[p]].son[!p]].size > T[T[x].son[!p]].size)
		Rotate(p, T[x].son[p]), Rotate(!p, x);	else return;
	Maintain(T[x].son[0], 0);
	Maintain(T[x].son[1], 1);
	Maintain(x, 0);
	Maintain(x, 1);
}

inline int Prev() //返回比根值小的最大值 若无返回0{	int x=T[rt].son[0];	if(!x) return 0;	while(T[x].son[1])
		x=T[x].son[1];	return x;
}

inline int Succ() //返回比根值大的最小值 若无返回0{	int x=T[rt].son[1];	if(!x) return 0;	while(T[x].son[0])
		x=T[x].son[0];	return x;
}void Insert(int key, int &x)
{	if(!x) x=Newnode(key);	else
	{		T[x].size++;
		Insert(key, T[x].son[key > T[x].key]);
		Maintain(x, key > T[x].key);
	}
}bool Delete(int key, int &x) //删除值为key的节点 key可以不存在{	if(!x) return 0;	if(T[x].key == key)
	{		if(!T[x].son[0])
		{
			x=T[x].son[1];			return 1;
		}		if(!T[x].son[1])
		{
			x=T[x].son[0];			return 1;
		}		int y=Prev();		T[x].size--;		return Delete(T[x].key, T[x].son[0]);
	}	else
		if(Delete(key, T[x].son[key > T[x].key]))
		{			T[x].size--;			return 1;
		}
}int GetPth(int p, int &x) //返回第p小的节点{	if(!x) return 0;	if(p == T[T[x].son[0]].size+1)		return x;	if(p > T[T[x].son[0]].size+1)		return GetPth(p-T[T[x].son[0]].size-1, T[x].son[1]);	else
		return GetPth(p, T[x].son[0]);
}int GetRank(int key, int &x) //找出值<=key的节点个数{	if(!x) return 0;	if(T[x].key <= key)		return T[T[x].son[0]].size+1+GetRank(key, T[x].son[1]);	else
		return GetRank(key, T[x].son[0]);
}
```
相关题解：

POJ 3481 SBT做法

上述题均为用于测试平衡树基本操作的题目。

提高题：（暂时未写）

[NOI2005]维修数列

[POJ3580]SuperMemo

[HNOI2004]宠物收养所

本部分内容引用自推库
### 可持久化Trie树（省选）
### 树状数组（省选）
参考：[“高级”数据结构——树状数组](https://www.cnblogs.com/RabbitHu/p/BIT.html)
#### 1. 单点修改 + 区间查询
最简单的树状数组就是这样的：
```cpp
void add(int p, int x){ //给位置p增加x
    while(p <= n) sum[p] += x, p += p & -p;
}
int ask(int p){ //求位置p的前缀和
    int res = 0;
    while(p) res += sum[p], p -= p & -p;
    return res;
}
int range_ask(int l, int r){ //区间求和
    return ask(r) - ask(l - 1);
}
```
#### 2. 区间修改 + 单点查询
通过“差分”（就是记录数组中每个元素与前一个元素的差），可以把这个问题转化为问题1。

查询

设原数组为$a[i]$, 设数组$d[i]=a[i]−a[i−1](a[0]=0)$，则 $a[i]=\sum_{j=1}^id[j]$，可以通过求$d[i]$的前缀和查询。

修改

当给区间$[l,r]$加上x的时候，$a[l]$ 与前一个元素 $a[l−1]$ 的差增加了x，$a[r+1]$ 与 $a[r]$ 的差减少了x。根据d[i]数组的定义，只需给a[l]加上 x, 给a[r+1] 减去 x 即可。
```cpp
void add(int p, int x){ //这个函数用来在树状数组中直接修改
    while(p <= n) sum[p] += x, p += p & -p;
}
void range_add(int l, int r, int x){ //给区间[l, r]加上x
    add(l, x), add(r + 1, -x);
}
int ask(int p){ //单点查询
    int res = 0;
    while(p) res += sum[p], p -= p & -p;
    return res;
}
```
#### 3. 区间修改 + 区间查询
这是最常用的部分，也是用线段树写着最麻烦的部分——但是现在我们有了树状数组！

怎么求呢？我们基于问题2的“差分”思路，考虑一下如何在问题2构建的树状数组中求前缀和：

位置p的前缀和 =
$∑_{i=1}^pa[i]=∑_{i=1}^p∑_{j=1}^id[j]$

在等式最右侧的式子$∑_{i=1}^p∑_{j=1}^id[j]$中，$d[1]$ 被用了p次，$d[2]$被用了p−1次……那么我们可以写出：

位置p的前缀和 =
$∑_{i=1}^p∑_{j=1}^id[j]=∑_{i=1}^pd[i]*(p-i+1)=(p+1)*∑_{i=1}^pd[i]-∑_{i=1}^pd[i]*i$
那么我们可以维护两个数组的前缀和：
一个数组是 sum1[i]=d[i]，
另一个数组是 sum2[i]=d[i]∗i。

查询

位置p的前缀和即： (p + 1) * sum1数组中p的前缀和 - sum2数组中p的前缀和。

区间[l, r]的和即：位置r的前缀和 - 位置l的前缀和。

修改

对于sum1数组的修改同问题2中对d数组的修改。

对于sum2数组的修改也类似，我们给 sum2[l] 加上 l * x，给 sum2[r + 1] 减去 (r + 1) * x。
```cpp
void add(ll p, ll x){
    for(int i = p; i <= n; i += i & -i)
        sum1[i] += x, sum2[i] += x * p;
}
void range_add(ll l, ll r, ll x){
    add(l, x), add(r + 1, -x);
}
ll ask(ll p){
    ll res = 0;
    for(int i = p; i; i -= i & -i)
        res += (p + 1) * sum1[i] - sum2[i];
    return res;
}
ll range_ask(ll l, ll r){
    return ask(r) - ask(l - 1);
}
```
用这个做区间修改区间求和的题，无论是时间上还是空间上都比带lazy标记的线段树要优。

#### 4. 二维树状数组
我们已经学会了对于序列的常用操作，那么我们不由得想到（谁会想到啊喂）……能不能把类似的操作应用到矩阵上呢？这时候我们就要写二维树状数组了！

在一维树状数组中，tree[x]（树状数组中的那个“数组”）记录的是右端点为x、长度为lowbit(x)的区间的区间和。
那么在二维树状数组中，可以类似地定义tree[x][y]记录的是右下角为(x, y)，高为lowbit(x), 宽为 lowbit(y)的区间的区间和。

#### 单点修改 + 区间查询
```cpp
void add(int x, int y, int z){ //将点(x, y)加上z
    int memo_y = y;
    while(x <= n){
        y = memo_y;
        while(y <= n)
            tree[x][y] += z, y += y & -y;
        x += x & -x;
    }
}
void ask(int x, int y){//求左上角为(1,1)右下角为(x,y) 的矩阵和
    int res = 0, memo_y = y;
    while(x){
        y = memo_y;
        while(y)
            res += tree[x][y], y -= y & -y;
        x -= x & -x;
    }
}
```
#### 区间修改 + 单点查询

我们对于一维数组进行差分，是为了使差分数组前缀和等于原数组对应位置的元素。

那么如何对二维数组进行差分呢？可以针对二维前缀和的求法来设计方案。

二维前缀和：
sum[i][j]=sum[i−1][j]+sum[i][j−1]−sum[i−1][j−1]+a[i][j]

那么我们可以令差分数组d[i][j] 表示 a[i][j] 与 a[i−1][j]+a[i][j−1]−a[i−1][j−1] 的差。

例如下面这个矩阵
```
 1  4  8
 6  7  2
 3  9  5
```
对应的差分数组就是
```
 1  3  4
 5 -2 -9
-3  5  1
```
当我们想要将一个矩阵加上x时，怎么做呢？
下面是给最中间的3*3矩阵加上x时，差分数组的变化：

0  0  0  0  0
0 +x  0  0 -x
0  0  0  0  0
0  0  0  0  0
0 -x  0  0 +x
这样给修改差分，造成的效果就是：

0  0  0  0  0
0  x  x  x  0
0  x  x  x  0
0  x  x  x  0
0  0  0  0  0
那么我们开始写代码吧！
```cpp
void add(int x, int y, int z){ 
    int memo_y = y;
    while(x <= n){
        y = memo_y;
        while(y <= n)
            tree[x][y] += z, y += y & -y;
        x += x & -x;
    }
}
void range_add(int xa, int ya, int xb, int yb, int z){
    add(xa, ya, z);
    add(xa, yb + 1, -z);
    add(xb + 1, ya, -z);
    add(xb + 1, yb + 1, z);
}
void ask(int x, int y){
    int res = 0, memo_y = y;
    while(x){
        y = memo_y;
        while(y)
            res += tree[x][y], y -= y & -y;
        x -= x & -x;
    }
}
```
#### 区间修改 + 区间查询

类比之前一维数组的区间修改区间查询，下面这个式子表示的是点(x, y)的二维前缀和：

$∑_{i=1}^x∑_{j=1}^y∑_{k=1}^i∑_{h=1}^jd[h][k]$
(d[h][k]为点(h, k)对应的“二维差分”(同上题))

这个式子炒鸡复杂( $O(n^4)$ 复杂度！)，但利用树状数组，我们可以把它优化到 $O(log^2n)$！

首先，类比一维数组，统计一下每个d[h][k]出现过多少次。$d[1][1]$出现了x∗y次，$d[1][2]$出现了x∗(y−1)次……$d[h][k]$ 出现了 (x−h+1)∗(y−k+1)次。

那么这个式子就可以写成：

$∑_{i=1}^x∑_{j=1}^yd[i][j]∗(x+1−i)∗(y+1−j)
$

把这个式子展开，就得到：

$(x+1)*(y+1)*∑_{i=1}^x∑_{j=1}^yd[i][j]−(y+1)∗∑_{i=1}^x∑_{j=1}^yd[i][j]∗i−(x+1)∗∑_{i=1}^x∑_{j=1}^yd[i][j]∗j+∑_{i=1}^x∑_{j=1}^yd[i][j]∗i∗j$

那么我们要开四个树状数组，分别维护：

d[i][j],d[i][j]∗i,d[i][j]∗j,d[i][j]∗i∗j
这样就完成了！
```cpp
#include <cstdio>
#include <cmath>
#include <cstring>
#include <algorithm>
#include <iostream>
using namespace std;
typedef long long ll;
ll read(){
    char c; bool op = 0;
    while((c = getchar()) < '0' || c > '9')
        if(c == '-') op = 1;
    ll res = c - '0';
    while((c = getchar()) >= '0' && c <= '9')
        res = res * 10 + c - '0';
    return op ? -res : res;
}
const int N = 205;
ll n, m, Q;
ll t1[N][N], t2[N][N], t3[N][N], t4[N][N];
void add(ll x, ll y, ll z){
    for(int X = x; X <= n; X += X & -X)
        for(int Y = y; Y <= m; Y += Y & -Y){
            t1[X][Y] += z;
            t2[X][Y] += z * x;
            t3[X][Y] += z * y;
            t4[X][Y] += z * x * y;
        }
}
void range_add(ll xa, ll ya, ll xb, ll yb, ll z){ //(xa, ya) 到 (xb, yb) 的矩形
    add(xa, ya, z);
    add(xa, yb + 1, -z);
    add(xb + 1, ya, -z);
    add(xb + 1, yb + 1, z);
}
ll ask(ll x, ll y){
    ll res = 0;
    for(int i = x; i; i -= i & -i)
        for(int j = y; j; j -= j & -j)
            res += (x + 1) * (y + 1) * t1[i][j]
                - (y + 1) * t2[i][j]
                - (x + 1) * t3[i][j]
                + t4[i][j];
    return res;
}
ll range_ask(ll xa, ll ya, ll xb, ll yb){
    return ask(xb, yb) - ask(xb, ya - 1) - ask(xa - 1, yb) + ask(xa - 1, ya - 1);
}
int main(){
    n = read(), m = read(), Q = read();
    for(int i = 1; i <= n; i++){
        for(int j = 1; j <= m; j++){
            ll z = read();
            range_add(i, j, i, j, z);
        }
    }
    while(Q--){
        ll ya = read(), xa = read(), yb = read(), xb = read(), z = read(), a = read();
        if(range_ask(xa, ya, xb, yb) < z * (xb - xa + 1) * (yb - ya + 1))
            range_add(xa, ya, xb, yb, a);
    }
    for(int i = 1; i <= n; i++){
        for(int j = 1; j <= m; j++)
            printf("%lld ", range_ask(i, j, i, j));
        putchar('\n');
    }
    return 0;
}
```
### 线段树（省选）
### 主席树（省选）
### 莫队算法+分块实现（省选）
### 详解配对堆
### 可合并堆——左偏树（Leftist Heap)
#### 左偏树的定义
节选：[一篇讲左偏树的好文章](http://blog.csdn.net/zy_discovery/article/details/52145143)
左偏树(Leftist Tree)是一种可并堆的实现。左偏树是一棵二叉树，它的节点除了和二叉树的节点一样具有左右子树指针( left, right )外，还有两个属性：键值和距离(dist)。键值上面已经说过，是用于比较节点的大小。距离则是如下定义的：
节点i称为外节点(external node)，当且仅当节点i的左子树或右子树为空 ( left(i) = NULL或right(i) = NULL )；节点i的距离(dist(i))是节点i到它的后代中，最近的外节点所经过的边数。特别的，如果节点i本身是外节点，则它的距离为0；而空节点的距离规定为-1 (dist(NULL) = -1)。在本文中，有时也提到一棵左偏树的距离，这指的是该树根节点的距离。
 
左偏树满足下面两条基本性质：
 
**[性质1] 节点的键值小于或等于它的左右子节点的键值。**
即key(i)≤key(parent(i))  这条性质又叫堆性质。符合该性质的树是堆有序的(Heap-Ordered)。有了性质1，我们可以知道左偏树的根节点是整棵树的最小节点，于是我们可以在O(1)的时间内完成取最小节点操作。
 
**[性质2] 节点的左子节点的距离不小于右子节点的距离。**
即dist(left(i))≥dist(right(i))  这条性质称为左偏性质。性质2是为了使我们可以以更小的代价在优先队列的其它两个基本操作（插入节点、删除最小节点）进行后维持堆性质。在后面我们就会看到它的作用。
 
这两条性质是对每一个节点而言的，因此可以简单地从中得出，左偏树的左右子树都是左偏树。
由这两条性质，我们可以得出左偏树的定义：左偏树是具有左偏性质的堆有序二叉树。
我们知道，一个节点必须经由它的子节点才能到达外节点。由于性质2，一个节点的距离实际上就是这个节点一直沿它的右边到达一个外节点所经过的边数，也就是说，我们有
 
**[性质3] 节点的距离等于它的右子节点的距离加1。**
即 dist( i ) = dist( right( i ) ) + 1  外节点的距离为0，由于性质2，它的右子节点必为空节点。为了满足性质3，故前面规定空节点的距离为-1。
 
我们的印象中，平衡树是具有非常小的深度的，这也意味着到达任何一个节点所经过的边数很少。左偏树并不是为了快速访问所有的节点而设计的，它的目的是快速访问最小节点以及在对树修改后快速的恢复堆性质。从图中我们可以看到它并不平衡，由于性质2的缘故，它的结构偏向左侧，不过距离的概念和树的深度并不同，左偏树并不意味着左子树的节点数或是深度一定大于右子树。
 
下面我们来讨论左偏树的距离和节点数的关系。
 
**[引理1] 若左偏树的距离为一定值，则节点数最少的左偏树是完全二叉树。**
证明：由性质2可知，当且仅当对于一棵左偏树中的每个节点i，都有 dist(left(i)) = dist(right(i))时，该左偏树的节点数最少。显然具有这样性质的二叉树是完全二叉树。
 
**[定理1] 若一棵左偏树的距离为k，则这棵左偏树至少有2k+1-1个节点。**
证明：由引理1可知，当这样的左偏树节点数最少的时候，是一棵完全二叉树。距离为k的完全二叉树高度也为k，节点数为2k+1-1，所以距离为k的左偏树至少有2k＋1-1个节点。
 
作为定理1的推论，我们有：
 
**[性质4] 一棵N个节点的左偏树距离最多为ëlog(N+1)û -1。**
证明：设一棵N个节点的左偏树距离为k，由定理1可知，N ≥ 2k+1-1，因此k ≤ ëlog(N+1)û -1。
 
左偏树的操作

本章将讨论左偏树的各种操作，包括插入新节点、删除最小节点、合并左偏树、构建左偏树和删除任意节点。由于各种操作都离不开合并操作，因此我们先讨论合并操作。
3.1 左偏树的合并

C ← Merge(A,B)
Merge( ) 把A,B两棵左偏树合并，返回一棵新的左偏树C，包含A和B中的所有元素。在本文中，一棵左偏树用它的根节点的指针表示。
在合并操作中，最简单的情况是其中一棵树为空（也就是，该树根节点指针为NULL）。这时我们只须要返回另一棵树。
 
若A和B都非空，我们假设A的根节点小于等于B的根节点（否则交换A,B），把A的根节点作为新树C的根节点，剩下的事就是合并A的右子树right(A) 和B了。
right(A) ← Merge(right(A), B)
 
合并了right(A) 和B之后，right(A) 的距离可能会变大，当right(A) 的距离大于left(A) 的距离时，左偏树的性质2会被破坏。在这种情况下，我们只须要交换left(A) 和right(A)。
若dist(left(A)) > dist(right(A))，交换left(A) 和right(A)
 

最后，由于right(A) 的距离可能发生改变，我们必须更新A的距离：
dist(A) ← dist(right(A)) + 1
不难验证，经这样合并后的树C符合性质1和性质2，因此是一棵左偏树。至此左偏树的合并就完成了。
 
下图是一个合并过程的示例：
合并流程

我们可以用下面的代码描述左偏树的合并过程：
```basic
Function Merge(A, B)
       If A = NULL Then return B
       If B = NULL Then return A
       If key(B) < key(A) Then swap(A, B)
       right(A) ← Merge(right(A), B)
       If dist(right(A)) > dist(left(A)) Then
              swap(left(A), right(A))
       If right(A) = NULL Then dist(A) ← 0
       Else dist(A) ← dist(right(A)) + 1
       return A
End Function
```
 
下面我们来分析合并操作的时间复杂度。从上面的过程可以看出，每一次递归合并的开始，都需要分解其中一棵树，总是把分解出的右子树参加下一步的合并。根据性质3，一棵树的距离决定于其右子树的距离，而右子树的距离在每次分解中递减，因此每棵树A或B被分解的次数分别不会超过它们各自的距离。根据性质4，分解的次数不会超过ëlog(N1+1)û + ëlog(N2+1)û -2，其中N1和N2分别为左偏树A和B的节点个数。因此合并操作最坏情况下的时间复杂度为O( ëlog(N1+1)û +ëlog(N2+1)û -2) = O(log N1 + log N2)。
 
3.2 插入新节点

单节点的树一定是左偏树，因此向左偏树插入一个节点可以看作是对两棵左偏树的合并。下面是插入新节点的代码：
```
Procedure Insert(x, A)
       B ← MakeIntoTree(x)
       A ← Merge(A, B)
End Procedure
```
 
由于合并的其中一棵树只有一个节点，因此插入新节点操作的时间复杂度是O(logn)。
3.3 删除最小节点

由性质1，我们知道，左偏树的根节点是最小节点。在删除根节点后，剩下的两棵子树都是左偏树，需要把他们合并。删除最小节点操作的代码也非常简单：
Function DeleteMin(A)
       t ← key(root(A))
       A ← Merge(left(A), right(A))
       return t
End Function
 
由于删除最小节点后只需进行一次合并，因此删除最小节点的时间复杂度也为O(logn)。
 
3.4 左偏树的构建

将n个节点构建成一棵左偏树，这也是一个常用的操作。
 
算法一  暴力算法——逐个节点插入，时间复杂度为O(nlogn)。
 
算法二  仿照二叉堆的构建算法，我们可以得到下面这种算法：
Ø        将n个节点（每个节点作为一棵左偏树）放入先进先出队列。
Ø        不断地从队首取出两棵左偏树，将它们合并之后加入队尾。
Ø        当队列中只剩下一棵左偏树时，算法结束。
下面分析算法二的时间复杂度。假设n=2k，则：
 
前 次和并的是两棵只有1个节点的左偏树。
接下来的 次合并的是两棵有2个节点的左偏树。
接下来的 次合并的是两棵有4个节点的左偏树。
……
接下来的 次合并的是两棵有2i-1个节点的左偏树。
 
合并两棵2i个节点的左偏树时间复杂度为O(i)，因此算法二的总时间复杂度为：
3.5 删除任意已知节点

接下来是关于删除任意已知节点的操作。之所以强调“已知”，是因为这里所说的任意节点并不是根据它的键值找出来的，左偏树本身除了可以迅速找到最小节点外，不能有效的搜索指定键值的节点。故此，我们不能要求：请删除所有键值为100的节点。
 
前面说过，优先队列是一种容器。对于通常的容器来说，一旦节点被放进去以后，容器就完全拥有了这个节点，每个容器中的节点具有唯一的对象掌握它的拥有权（ownership）。对于这种容器的应用，优先队列只能删除最小节点，因为你根本无从知道它的其它节点是什么。
 
但是优先队列除了作为一种容器外还有另一个作用，就是可以找到最小节点。很多应用是针对这个功能的，它们并没有将拥有权完全转移给优先队列，而是把优先队列作为一个最小节点的选择器，从一堆节点中依次将它们选出来。这样一来节点的拥有权就可能同时被其它对象掌握。也就是说某个节点虽不是最小节点，不能从优先队列那里“已知”，但却可以从其它的拥有者那里“已知”。
 
这种优先队列的应用也是很常见的。设想我们有一个闹钟，它可以记录很多个响铃时间，不过由于时间是线性的，铃只能一个个按先后次序响，优先队列就很适合用来作这样的挑选。另一方面使用者应该可以随时取消一个“已知”的响铃时间，这就需要进行任意已知节点的删除操作了。
 
我们的这种删除操作需要指定被删除的节点，这和原来的删除根节点的操作是兼容的，因为根节点肯定是已知的。上面已经提过，在删除一个节点以后，将会剩下它的两棵子树，它们都是左偏树，我们先把它们合并成一棵新的左偏树。
p ← Merge(left(x), right(x))
现在p指向了这颗新的左偏树，如果我们删除的是根节点，此时任务已经完成了。不过，如果被删除节点x不是根节点就有点麻烦了。这时p指向的新树的距离有可能比原来x的距离要大或小，这势必有可能影响原来x的父节点q的距离，因为q现在成为新树p的父节点了。于是就要仿照合并操作里面的做法，对q的左右子树作出调整，并更新q的距离。这一过程引起了连锁反应，我们要顺着q的父节点链一直往上进行调整。
x
q
q
p
 
新树p的距离为dist(p)，如果dist(p)+1等于q的原有距离dist(q)，那么不管p是q的左子树还是右子树，我们都不需要对q进行任何调整，此时删除操作也就完成了。
 
如果dist(p)+1小于q的原有距离dist(q)，那么q的距离必须调整为dist(p)+1，而且如果p是左子树的话，说明q的左子树距离比右子树小，必须交换子树。由于q的距离减少了，所以q的父节点也要做出同样的处理。
 
剩下就是另外一种情况了，那就是p的距离增大了，使得dist(p)+1大于q的原有距离dist(q)。在这种情况下，如果p是左子树，那么q的距离不会改变，此时删除操作也可以结束了。如果p是右子树，这时有两种可能：一种是p的距离仍小于等于q的左子树距离，这时我们直接调整q的距离就行了；另一种是p的距离大于q的左子树距离，这时我们需要交换q的左右子树并调整q的距离，交换完了以后q的右子树是原来的左子树，它的距离加1只能等于或大于q的原有距离，如果等于成立，删除操作可以结束了，否则q的距离将增大，我们还要对q的父节点做出相同的处理。
 
删除任意已知节点操作的代码如下：
Procedure Delete(x)
       q ← parent(x)
       p ← Merge(left(x), right(x))
       parent(p) ← q
       If q ≠ NULL and left(q) = x Then
              left(q) ← p
       If q ≠ NULL and right(q) = x Then
              right(q) ← p
       While q ≠ NULL Do
              If dist(left(q)) < dist(right(q)) Then
                     swap(left(q), right(q))
              If dist(right(q))+1 = dist(q) Then
                     Exit Procedure
              dist(q) ← dist(right(q))+1
              p ← q
              q ← parent(q)
       End While
End Procedure
 
下面分两种情况讨论删除操作的时间复杂度。
 
情况1：p的距离减小了。在这种情况下，由于q的距离只能缩小，当循环结束时，要么根节点处理完了，q为空；要么p是q的右子树并且dist(p)+1=dist(q)；如果dist(p)+1>dist(q)，那么p一定是q的左子树，否则会出现q的右子树距离缩小了，但是加1以后却大于q的距离的情况，不符合左偏树的性质3。不论哪种情况，删除操作都可以结束了。注意到，每一次循环，p的距离都会加1，而在循环体内，dist(p)+1最终将成为某个节点的距离。根据性质4，任何的距离都不会超过logn，所以循环体的执行次数不会超过logn。
 
情况2：p的距离增大了。在这种情况下，我们将必然一直从右子树向上调整，直至q为空或p是q的左子树时停止。一直从右子树升上来这个事实说明了循环的次数不会超过logn（性质4）。
 
最后我们看到这样一个事实，就是这两种情况只会发生其中一个。如果某种情况的调整结束后，我们已经知道要么q为空，要么dist(p)+1 = dist(q)，要么p是q的左子树。这三种情况都不会导致另一情况发生。直观上来讲，如果合并后的新子树导致了父节点的一系列距离调整的话，要么就一直是往小调整，要么是一直往大调整，不会出现交替的情况。
 
我们已经知道合并出新子树p的复杂度是O(logn)，向上调整距离的复杂度也是O(logn)，故删除操作的最坏情况的时间复杂度是O(logn)。如果左偏树非常倾斜，实际应用情况下要比这个快得多。
 
3.6 小结

本章介绍了左偏树的各种操作，我们可以看到，左偏树作为可并堆的实现，它的各种操作性能都十分优秀，且编程复杂度比较低，可以说是一个“性价比”十分高的数据结构。左偏树之所以是很好的可并堆实现，是因为它能够捕捉到具有堆性质的二叉树里面的一些其它有用信息，没有将这些信息浪费掉。根据堆性质，我们知道，从根节点向下到任何一个外节点的路径都是有序的。存在越长的路径，说明树的整体有序性越强，与平衡树不同（平衡树根本不允许有很长的路径），左偏树尽大约一半的可能保留了这个长度，并将它甩向左侧，利用它来缩短节点的距离以提高性能。这里我们不进行严格的讨论，左偏树作为一个例子大致告诉我们：放弃已有的信息意味着算法性能上的牺牲。下面是最好的左偏树：有序表（插入操作是按逆序发生的，自然的有序性被保留了）和最坏的左偏树：平衡树（插入操作是按正序发生的，自然的有序性完全被放弃了）。
1
2
3
4
5
6
7
有序表 
平衡树 
1
2
3
4
5
6
7

3.7 附代码 
```cpp
#include <iostream>
#include <cstdio>
#include <cstdlib>
using namespace std;
/*基本的存储结构*/
struct LTNode
{
	int data;
	int dist;
	struct LTNode *lchild;
	struct LTNode *rchild;
};


/*简单的中序遍历操作，方便调试用的*/
void InOrder(LTNode *t)
{
	if(t)
	{
		InOrder(t->lchild);
		printf("%d ",t->data);
		InOrder(t->rchild);
	}
}
/*简单的先序遍历操作，方便调试用的*/
void PreOrder(LTNode *t)
{
	if(t)
	{
		printf("%d ",t->data);
		PreOrder(t->lchild);
		PreOrder(t->rchild);
	}
}

/*核心操作，一定要小心实现*/
LTNode* merge(LTNode* &A,LTNode* &B)
{
	if(A==NULL || B==NULL)
		return A==NULL?B:A;

	if(A->data > B->data)    /*确保B->data >= A->data*/
	{	swap<LTNode*>(A,B);	}//swap函数包含在<iostream>与std中 

	A->rchild = merge(A->rchild,B); /*新来个左偏树始终合并到右侧*/

	/*由于新结点合并到右侧，右侧结点现在一定存在了,但左侧不一定*/
	if(A->lchild==NULL ||          /*左侧为空，一定小于右侧*/
	   A->rchild->dist > A->lchild->dist)/*右侧大于了左侧*/
		swap<LTNode*>(A->lchild,A->rchild);

	if(A->rchild==NULL){A->dist = 0;}     /*右子树为空*/
	else               {A->dist = A->rchild->dist + 1;}

	return A;
}

void insert(LTNode* &t,int x)
{
	/*initiate the node*/
	LTNode *p = new LTNode;
	p->data = x;
	p->dist = 0;
	p->lchild = NULL;
	p->rchild = NULL;

	/*merge into a leftist tree*/
	t = merge(t,p);
	printf("insert %d success\n",x);
}

LTNode* ExtractMin(LTNode* &t)
{  
	/*根元素始终都是最小的元素*/
	LTNode *p = t;
	/*合并根的左右子树*/
	t = merge(t->lchild,t->rchild);
	/*返回指向最小节点的指针*/
	return p;
}
int main()
{
	LTNode *t=new LTNode();
	while(true)
	{
		int x;
		scanf("%d",&x);
		insert(t,x);
		printf("pre-order:\n");
		PreOrder(t);
		printf("\n");
		printf("In-order:\n");
		InOrder(t);
		printf("\n");
	}	
}
```
四、左偏树的应用



4.1 例——数字序列（Baltic 2004）

[问题描述]
给定一个整数序列a1, a2, … , an，求一个不下降序列b1 ≤ b2 ≤ … ≤ bn，使得数列{ai}和{bi}的各项之差的绝对值之和 |a1 - b1| + |a2 - b2| + … + |an - bn| 最小。
[数据规模] 1 ≤ n ≤ 10^6, 0 ≤ ai ≤ 2*10^9
[初步分析]
我们先来看看两个最特殊的情况：
1．a[1]≤a[2]≤…≤a[n]，在这种情况下，显然最优解为b[i]=a[i]；
2．a[1]≥a[2]≥…≥a[n]，这时，最优解为b[i]=x，其中x是数列a的中位数[†]。
于是我们可以初步建立起这样一个思路：
把1…n划分成m个区间：[q[1],q[2]-1]，[q[2],q[3]-1]，……，[q[m],q[m+1]-1][‡]。每个区间对应一个解，b[q[i]] = b[q[i]+1] = … = b[q[i+1]-1] = w[i]，其中w[i] 为a[q[i]], a[q[i]+1], ... , a[q[i+1]-1] 的中位数。
显然，在上面第一种情况下m=n，q[i]=i；在第二种情况下m=1，q[1]=1。
这样的想法究竟对不对呢？应该怎样实现？
若某序列前半部分a[1], a[2], … , a[n] 的最优解为 (u,u,…,u)，后半部分a[n+1], a[n+2], ... , a[m]的最优解为 (v,v,…,v)，那么整个序列的最优解是什么呢？若u≤v，显然整个序列的最优解为 (u,u,…,u,v,v,…,v)。否则，设整个序列的最优解为 (b[1],b[2],…,b[m])，则显然b[n]≤u（否则我们把前半部分的解 (b[1],b[2],…,b[n]) 改为 (u,u,…,u)，由题设知整个序列的解不会变坏），同理b[n+1]≥v。接下来，我们将看到下面这个事实：
对于任意一个序列a[1],a[2],…,a[n]，如果最优解为 (u,u,…,u)，那么在满足u≤u′≤b[1]或b[n]≤u′≤u的情况下，(b[1],b[2],…,b[n]) 不会比 (u′,u′,…,u′) 更优。
我们用归纳法证明u≤u′≤b[1] 的情况，b[n]≤u′≤u的情况可以类似证明。
当n=1时，u=a[1]，命题显然成立。
当n>1时，假设对于任意长度小于n的序列命题都成立，现在证明对于长度为n的序列命题也成立。首先把 (b[1], b[2], … b[n]) 改为 (b[1], b[1], … b[1])，这一改动将不会导致解变坏，因为如果解变坏了，由归纳假设可知a[2],a[3],…,a[n]的中位数w>u，这样的话，最优解就应该为(u,u,…,u,w,w,…,w)，矛盾。然后我们再把(b[1],b[1],…,b[1])改为 (u′,u′,…,u′)，由于| a[1] - x | + | a[2] - x | + … + | a[n] - x | 的几何意义为数轴上点x到点a[1], a[2], … a[n]的距离之和，且u≤u′≤b[1]，显然点u′到各点的距离之和不会比点b[1]到各点的距离之和大，也就是说，(b[1],b[1],…,b[n]) 不会比 (v,v,…,v) 更优。（证毕）
再回到之前的论述，由于b[n]≤u，作为上述事实的结论，我们可以得知，将(b[1],b[2],…,b[n])改为(b[n],b[n],…,b[n])，再将(b[n+1],b[n+2],…,b[m])改为(b[n+1],b[n+1],…,b[n+1])，并不会使解变坏。也就是说，整个序列的最优解为(b[n],b[n],…,b[n],b[n+1],b[n+1],…,b[n+1])。再考虑一下该解的几何意义，设整个序列的中位数为w，则显然令b[n]=b[n+1]=w将得到整个序列的最优解，即最优解为 (w,w,…,w)。
分析到这里，我们一开始的想法已经有了理论依据，算法也不难构思了。
[算法描述]
延续我们一开始的思路，假设我们已经找到前k个数a[1], a[2], … , a[k] (k<n) 的最优解，得到m个区间组成的队列，对应的解为 (w[1],w[2],…,w[m])，现在要加入a[k+1]，并求出前k+1个数的最优解。首先我们把a[k+1]作为一个新区间直接加入队尾，令w[m+1]=a[k+1]，然后不断检查队尾两个区间的解w[m]和w[m+1]，如果w[m]>w[m+1]，我们需要将最后两个区间合并，并找出新区间的最优解（也就是序列a中，下标在这个新区间内的各项的中位数）。重复这个合并过程，直至w[1]≤w[2]≤…≤w[m]时结束，然后继续处理下一个数。
这个算法的正确性前面已经论证过了，现在我们需要考虑一下数据结构的选取。算法中涉及到以下两种操作：合并两个有序集以及查询某个有序集内的中位数。能较高效地支持这两种操作的数据结构有不少，一个比较明显的例子是二叉检索树(BST)，它的询问操作复杂度是O(logn)，但合并操作不甚理想，采用启发式合并，总时间复杂度为O(nlog2n)。
有没有更好的选择呢？通过进一步分析，我们发现，只有当某一区间内的中位数比后一区间内的中位数大时，合并操作才会发生，也就是说，任一区间与后面的区间合并后，该区间内的中位数不会变大。于是我们可以用最大堆来维护每个区间内的中位数，当堆中的元素大于该区间内元素的一半时，删除堆顶元素，这样堆中的元素始终为区间内较小的一半元素，堆顶元素即为该区间内的中位数。考虑到我们必须高效地完成合并操作，左偏树是一个理想的选择[§]。左偏树的询问操作时间复杂度为O(1)，删除和合并操作时间复杂度都是O(logn)，而询问操作和合并操作少于n次，删除操作不超过n/2次（因为删除操作只会在合并两个元素个数为奇数的堆时发生），因此用左偏树实现，可以把算法的时间复杂度降为O(nlogn)。
[小结]
这道题的解题过程对我们颇有启示。在应用左偏树解题时，我们往往会觉得题目无从下手，甚至与左偏树毫无关系，但只要我们对题目深入分析，加以适当的转化，问题终究会迎刃而解。这需要我们具有敏捷的思维以及良好的题感。
用左偏树解本题，相比较于前面BST的解法，时间复杂度和编程复杂度更低，这使我们不得不感叹于左偏树的神奇威力。这不是说左偏树就一定是最好的解法，就本题来说，解法有很多种，光是可并堆的解法，就可以用多种数据结构来实现，但左偏树相对于它们，还是有一定的优势的，这将在下一章详细讨论。
五、左偏树与各种可并堆的比较

我们知道，左偏树是一种可并堆的实现。但是为什么我们要用左偏树实现可并堆呢？左偏树相对于其他可并堆有什么优点？本章将就这个问题展开讨论，介绍各种可并堆的特点，并对它们做出比较。
5.1 左偏树的变种——斜堆

这里我们要介绍左偏树的一个变种——斜堆(Skew Heap)。斜堆是一棵堆有序的二叉树，但是它不满足左偏性质，或者说，斜堆根本就没有“距离”这个概念——它不需要记录任何一个节点的距离。从结构上来说，所有的左偏树都是斜堆，但反之不然。
类似于左偏树，斜堆的各种操作也是在合并操作的基础上完成的，因此这里只介绍斜堆的合并操作，其他操作读者都可以仿照左偏树完成。
斜堆合并操作的递归合并过程和左偏树完全一样。假设我们要合并A和B两个斜堆，且A的根节点比B的根节点小，我们只需要把A的根节点作为合并后新堆的根节点，并将A的右子树与B合并。由于合并都是沿着最右路径进行的，经过合并之后，新堆的最右路径长度必然增加，这会影响下一次合并的效率。为了解决这一问题，左偏树在进行合并的同时，检查最右路径节点的距离，并通过交换左右子树，使整棵树的最右路径长度非常小。然而斜堆不记录节点的距离，那么应该怎样维护最右路径呢？我们采取的办法是，从下往上，沿着合并的路径，在每个节点处都交换左右子树。
下面是斜堆合并操作的代码：
Function Merge(A,B)
       If A = NULL Then return B
       If B = NULL Then return A
       If key(B) < key(A) Then swap(A,B)
       right(A) ← Merge(right(A), B)
       swap(left(A), right(A))
       return A
End Function
 
斜堆的这种维护方法也是行之有效的。通过不断交换左右子树，斜堆把最右路径甩向左边了。可以证明，斜堆合并操作的平摊时间复杂度为O(logn)。这里略去详细的复杂度分析，感兴趣的读者可以自行参考相关的资料。
前面说过，斜堆的其他各种操作都和左偏树类似，因此斜堆各项操作的平摊时间复杂度都与左偏树相同。在空间上，由于斜堆不用记录节点的距离，因此它比左偏树的空间需求小一点。至于编程复杂度，两者都十分的低，不过斜堆的代码还是要比左偏树稍微简洁一些。至于斜堆的不足，大概可以算是它单次合并操作的时间复杂度可能退化为O(n)，不过通常这并不影响算法的总时间复杂度。
 
总的来说，斜堆作为左偏树的变种，与左偏树并无优劣之分。在斜堆能派上用场的地方，左偏树同样能出色地实现算法。斜堆与左偏树之间的关系，可以类比于伸展树与AVL树之间的关系，只不过前两者的编程复杂度，并没有多大的差别。
 
5.2 左偏树与二叉堆的比较

二叉堆大概是我们最常用的一种优先队列了，它具有简洁，高效的特点，应用十分广泛。二叉堆和左偏树相比，除了合并操作外的各种操作，时间复杂度都一样，但在实际测试中，二叉堆往往比左偏树快一些。在空间上，由于二叉堆是完全二叉树，用数组表示法，可以剩下左右子树指针的空间，在这点上左偏树又吃了一亏。
介绍了二叉堆的优点，下面的这两个缺点也是不容忽视的：
1．在进行插入、删除等操作后，二叉堆中元素的物理位置会发生改变。
2．二叉堆的合并操作太慢，时间复杂度高达O(n)。
当元素本身所占空间比较大时，频繁地移动元素物理位置将会影响时间效率。而有些时候，我们需要随时掌握某些元素的物理位置，以便对它进行访问或修改（比如实现删除任意已知节点操作），这时，二叉堆的第一个缺点将给我们造成一定的麻烦，我们不得不通过增加元素指针等手段来解决这个问题。而二叉堆合并效率的低下则是由其本身性质所造成的，采用启发式合并可以作为大多数情况下的一个优化，但效果仍不能令人满意。
至于二叉堆空间上的优势，也不是绝对的。数组表示法并不是在任何场合都适用的，比如当我们需要动态维护多个优先队列的时候，二叉堆不得不使用传统的指针表示法，这时二叉堆和左偏树在空间上就相差无几了。
 
二叉堆的上述特点，决定了它不适合作为可并堆的实现。客观的说，需要实现可并堆的题目在竞赛中并不多见，不过当我们遇到这类题目或者某些特殊的情况时，左偏树将会是比二叉堆更好的选择。
 
5.3 左偏树与其他可并堆的比较

前面已经提到过，可并堆可以用多种数据结构实现，左偏树并非唯一的选择。二项堆，Fibonacci堆都是时间效率十分优秀的可并堆。
二项堆是由若干棵深度不同的二项树组成的森林。深度为k的二项树Bk由两棵二项树Bk-1连接根节点而成，有2k个节点，并且满足堆性质。二项树组成二项堆的形式可以看作总节点数n的二进制表示形式，两个二项堆的合并可以看作二进制数的加法，这点很有启发意义。与左偏树类似，二项堆的各种操作也是在合并操作的基础上完成的。二项堆的结构和性质决定了它可以很高效地完成合并操作，与左偏树相比，虽然复杂度相同，但一般实际情况下二项堆比较快。但二项树实现取最小节点操作需要检查所有二项树的根节点，因此这项操作时间复杂度比左偏树高[**]。
Fibonacci堆是一个很复杂的数据结构，与二项堆一样，它也是由一组堆有序的树构成的。所不同的是，Fibonacci堆的树不一定是二项树，而且这些树是无序的，且树中兄弟节点的联系用双向循环链表来表示。Fibonacci堆实现插入操作和合并操作只是简单地将两个Fibonacci堆的根表连在一起，因此这两个操作比其他可并堆都快。但在删除操作时，我们需要对Fibonacci堆进行维护，合并所有度数相同的树，这一步异常复杂，并且是最慢的。
有关二项堆和Fibonacci堆的详细介绍，有兴趣的读者可以参考相关资料，本文不再赘述。下表列出了各种可并堆的各项操作的时间复杂度[††]，已及它们的空间需求和编程复杂度。
|项目|二叉堆|左偏树|二项堆|Fibonacci堆|
|:-:|:-:|:-:|:-:|:-:|
|构建|O(n)|O(n)|O(n)|O(n)|
|插入|O(logn)|O(logn)|O(logn)|O(1)|
|取最小节点|O(1)|O(1)|O(logn)|O(1)|
|删除最小节点|O(logn)|O(logn)|O(logn)|O(logn)|
|删除任意节点|O(logn)|O(logn)|O(logn)|O(logn)|
|合并|O(n)|O(logn)|O(logn)|O(1)|
|空间需求|最小|较小|一般|较大|
|编程复杂度|最低|较低|较高|很高|
 
从表中我们可以看出，Fibonacci堆的时间复杂度非常低，如果我们不需要进行频繁的删除操作，用Fibonacci堆实现可并堆将会降低算法的时间复杂度。但实际上，删除操作往往是很重要的操作，而Fibonacci堆的删除操作比表中其它可并堆都慢，至于它的空间需求，也大得使人望而却步。更槽糕的是Fibonacci堆的编程复杂度太高了，在竞赛中使用实在是不理智的行为。
至于二项堆，虽然各项操作的时间效率都十分优秀，但空间需求和编程复杂度仍然比不上左偏树。和左偏树相比，二项堆在时间效率上的优势微乎其微，用左偏树代替二项堆，的确会牺牲一些算法性能，但换来的却是简洁的代码，便于实现和调试的程序。在时间有限的竞赛中，左偏树无疑是更好的选择。
 
最后，我们应该认识到，虽然二项堆和Fibonacci堆某些操作的时间复杂度比左偏树低，但是在实际应用中，那些时间复杂度较高的操作往往会成为算法的瓶颈。比如前面的例题《数字序列》，改用二项堆或Fibonacci堆实现，总时间复杂度仍为O(nlogn)，并不能起到降低算法时间复杂度的作用，实现难度反而增加了不少。
六、总结

至此，我们已经对左偏树有了深刻的认识。左偏树在时间效率上不如二项堆和Fibonacci堆，在空间效率上不如二叉堆，这样看来左偏树没有任何独树一帜的地方，似乎是个相当平庸的数据结构，但其实这正是左偏树的优势所在。正所谓“鱼与熊掌不可兼得”，时间复杂度、空间复杂度和编程复杂度，这三者之间很多时候是矛盾的。Fibonacci堆时间复杂度最低，但编程复杂度让人无法接受；二叉堆的空间复杂度和编程复杂度都很低，但时间复杂度却是它的致命弱点。左偏树很好地协调了三者之间的矛盾，并且在存储性质上，没有二叉堆那样的缺陷，因此左偏树的适用范围十分广。左偏树不但可以高效方便地实现可并堆，更可以作为二叉堆的替代品，应用于各种优先队列，很多时候甚至比二叉堆更方便。

### 可合并堆——二项堆（Binomial Heap）
### 可合并堆——斜堆
### 可合并堆——斐波那契堆（Fibonacci Heap）
### pairing 堆
### d叉堆
### 详解KDTree
### AC自动机算法
### 搞懂后缀数组！
### 字典树，后缀树
### 图论-SPFA算法
### 图论-第K短路

### 强连通分量（Tarjan）
### A*搜索算法
### 教你初步了解KMP算法
### 由KMP算法谈到BM算法
### Dijkstra 算法+fibonacci堆的逐步c实现
### Dijkstra 算法+Heap堆的完整c源码

### Link-Cut Tree(LCT)动态树
### 博弈论
### 快速傅利叶变换
### 前缀和
#### 一维前缀和

这个优化主要是用来在O（1）时间内求出一个序列a中,a[i]+a[i+1]+……+a[j]的和。

具体原理十分简单：用sum[i]表示（a[1]+a[2]+……+a[i])，其中sum[0]=0，则（a[i]+a[i+1]+……+a[j]）即等于sum[j]-sum[i-1]。

#### 二维前缀和

同理，有一维就有二维。对于一个矩阵a，我们也能在O（1）时间内求出子矩阵[x1~x2][y1~y2]的和。

设sum[i][j]为子矩阵[1~i][1~j]的和。则由容斥原理得：

sum[0][j]=sum[i][0]=0

a[x1~x2][y1~y2]=sum[x2][y2]-sum[x1-1][y2]-sum[x2][y1-1]+sum[x1-1][y1-1]

#### 应用问题

核心就两个字：降维。

面对许多高维问题，往往前缀和是最先想到的降维方法。 
这样在降维的基础上，许多更进一步的优化才能实现。

例题：
BZOJ1218: [HNOI2003]激光炸弹([http://www.lydsy.com/JudgeOnline/problem.php?id=1218])

## 数学知识
### 欧几里德与扩展欧几里德算法
欧几里德算法又称辗转相除法，用于计算两个整数a,b的最大公约数。其计算原理依赖于下面的定理：
定理：gcd(a,b) = gcd(b,a mod b)
证明：a可以表示成a = kb + r，则r = a mod b 
假设d是a,b的一个公约数，则有 
d|a, d|b，而r = a - kb，因此d|r 
因此d是(b,a mod b)的公约数
假设d 是(b,a mod b)的公约数，则 
d | b , d |r ，但是a = kb +r 
因此d也是(a,b)的公约数
因此(a,b)和(b,a mod b)的公约数是一样的，其最大公约数也必然相等，得证
时间复杂度O（logN),N=min(a,b)
用 C++ 语言描述如下：
```cpp
int gcd(int a,int b)
{
    while(b)
    {
        int r=b;
        b=a%b;
        a=r;
    }
}
```
```cpp
int gcd(int a,int b)
{
    if(b==0)return a;
    else return gcd(b,a%b);
}
```

扩展欧几里德

    现在我们知道了 a 和 b 的最大公约数是 gcd ，那么，我们一定能够找到这样的 x 和 y ，使得: a*x + b*y = gcd 这是一个不定方程（其实是一种丢番图方程），有多解是一定的，但是只要我们找到一组特殊的解 x0 和 y0 那么，我们就可以用 x0 和 y0 表示出整个不定方程的通解：
        x = x0 + (b/gcd)*t
        y = y0 – (a/gcd)*t
    为什么不是：
        x = x0 + b*t
        y = y0 – a*t
    这个问题也是在今天早上想通的，想通之后忍不住喷了自己一句弱逼。那是因为：
    b/gcd 是 b 的因子， a/gcd 是 a 的因子是吧？那么，由于 t的取值范围是整数，你说 (b/gcd)*t 取到的值多还是 b*t 取到的值多？同理，(a/gcd)*t 取到的值多还是 a*gcd 取到的值多？那肯定又要问了，那为什么不是更小的数，非得是 b/gcd 和a/gcd ？
    注意到：我们令 B = b/gcd ， A = a、gcd ， 那么，A 和 B 一定是互素的吧？这不就证明了 最小的系数就是 A 和 B 了吗？要是实在还有什么不明白的，看看《基础数论》（哈尔滨工业大学出版社），这本书把关于不定方程的通解讲的很清楚
    现在，我们知道了一定存在 x 和 y 使得 ： a*x + b*y = gcd ， 那么，怎么求出这个特解 x 和 y 呢？只需要在欧几里德算法的基础上加点改动就行了。
    我们观察到：欧几里德算法停止的状态是： a= gcd ， b = 0 ，那么，这是否能给我们求解 x y 提供一种思路呢？因为，这时候，只要 a = gcd 的系数是 1 ，那么只要 b 的系数是 0 或者其他值（无所谓是多少，反正任何数乘以 0 都等于 0 但是a 的系数一定要是 1），这时，我们就会有： a*1 + b*0 = gcd
    当然这是最终状态，但是我们是否可以从最终状态反推到最初的状态呢？
    假设当前我们要处理的是求出 a 和 b的最大公约数，并求出 x 和 y 使得 a*x + b*y= gcd ，而我们已经求出了下一个状态：b 和 a%b 的最大公约数，并且求出了一组x1 和y1 使得： b*x1 + (a%b)*y1 = gcd ， 那么这两个相邻的状态之间是否存在一种关系呢？
    我们知道： a%b = a - (a/b)*b（这里的 “/” 指的是整除，例如 5/2=2 , 1/3=0），那么，我们可以进一步得到：
        gcd = b*x1 + (a-(a/b)*b)*y1
            = b*x1 + a*y1 – (a/b)*b*y1
            = a*y1 + b*(x1 – a/b*y1)
    对比之前我们的状态：求一组 x 和 y 使得：a*x + b*y = gcd ，是否发现了什么？
    这里：
        x = y1
        y = x1 – a/b*y1
    以上就是扩展欧几里德算法的全部过程，依然用递归写：
```cpp
int e_gcd(int a,int b,int &x,int &y)
{
    if(b==0)
    {
        x=1;
        y=0;
        return a;
    }
    int ans=e_gcd(b,a%b,x,y);
    int temp=x;
    x=y;
    y=temp-x/b*y;
    return ans;
}
```

依然很简短，相比欧几里德算法，只是多加了几个语句而已。
    这就是理论部分，欧几里德算法部分我们好像只能用来求解最大公约数，但是扩展欧几里德算法就不同了，我们既可以求出最大公约数，还可以顺带求解出使得： a*x + b*y = gcd 的通解 x 和 y
    扩展欧几里德有什么用处呢？
    求解形如 a*x +b*y = c 的通解，但是一般没有谁会无聊到让你写出一串通解出来，都是让你在通解中选出一些特殊的解，比如一个数对于另一个数的乘法逆元
    什么叫乘法逆元？
    
这里，我们称 x 是 a 关于 m 的乘法逆元
    这怎么求？可以等价于这样的表达式： a*x + m*y = 1
    看出什么来了吗？没错，当gcd(a , m) != 1 的时候是没有解的这也是 a*x + b*y = c 有解的充要条件： c % gcd(a , b) == 0
    接着乘法逆元讲，一般，我们能够找到无数组解满足条件，但是一般是让你求解出最小的那组解，怎么做？我们求解出来了一个特殊的解 x0 那么，我们用 x0 % m其实就得到了最小的解了。为什么？
可以这样思考：
    x 的通解不是 x0 + m*t 吗？
    那么，也就是说， a 关于 m 的逆元是一个关于 m 同余的，那么根据最小整数原理，一定存在一个最小的正整数，它是 a 关于m 的逆元，而最小的肯定是在（0 , m）之间的，而且只有一个，这就好解释了。
    可能有人注意到了，这里，我写通解的时候并不是 x0 + (m/gcd)*t ，但是想想一下就明白了，gcd = 1，所以写了跟没写是一样的，但是，由于问题的特殊性，有时候我们得到的特解 x0 是一个负数，还有的时候我们的 m 也是一个负数这怎么办？
    当 m 是负数的时候，我们取 m 的绝对值就行了，当 x0 是负数的时候，他模上 m 的结果仍然是负数（在计算机计算的结果上是这样的，虽然定义的时候不是这样的），这时候，我们仍然让 x0 对abs(m) 取模，然后结果再加上abs(m) 就行了，于是，我们不难写出下面的代码求解一个数 a 对于另一个数 m 的乘法逆元：
    
还有最小整数解之类的问题，但都是大同小异，只要细心的推一推就出来了，这里就不一一介绍了，下面给一些题目还有AC代码，仅供参考
    ZOJ 3609 ：http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemId=4712 求最小逆元
    
```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cmath>
#include <vector>
#include <string>
#include <queue>
#include <stack>
#include <algorithm>

#define INF 0x7fffffff
#define EPS 1e-12
#define MOD 1000000007
#define PI 3.141592653579798
#define N 100000

using namespace std;

typedef long long LL;
typedef double DB;

LL e_gcd(LL a,LL b,LL &x,LL &y)
{
    if(b==0)
    {
        x=1;
        y=0;
        return a;
    }
    LL ans=e_gcd(b,a%b,x,y);
    LL temp=x;
    x=y;
    y=temp-a/b*y;
    return ans;
}

LL cal(LL a,LL b,LL c)
{
    LL x,y;
    LL gcd=e_gcd(a,b,x,y);
    if(c%gcd!=0) return -1;
    x*=c/gcd;
    b/=gcd;
    if(b<0) b=-b;
    LL ans=x%b;
    if(ans<=0) ans+=b;
    return ans;
}

int main()
{
    LL a,b,t;
    scanf("%lld",&t);
    while(t--)
    {
        scanf("%lld%lld",&a,&b);
        LL ans=cal(a,b,1);
        if(ans==-1) printf("Not Exist\n");
        else printf("%lld\n",ans);
    }
    return 0;
}
```
### 费马小定理

### 字符串哈希
例题：POJ2503
BKDRhash(效果最佳）代码解读
hdu 1800 字符串的hash(BKDRHash 模版)
ELFhash（效果最差）代码解读
解释：
首先我们的hash结果是一个unsigned int类型的数据：
0000 0000 0000 0000
1.hash左移4位，将str插入（一个char有八位）这里我开始也一直是怀疑的态度，那么第一个字节的高四位不就乱了吗
实际上这也是我们的第一次杂糅，我们是故意这么做的，这里我们需要注意标记一下，我们在第一个字节的高四位做了第一次杂糅
2.x这里用0xf0000000获取了hash的第四个字节的高四位，并用高四位作为掩码做第二次杂糅
在这里我们首先声明一下，因为我们的ELFhash强调的是每个字符都要对最后的结构有影响，所以说我们左移到一定程度是会吞掉最高的四位的，所以说我们要将最高的四位先对串产生影响，再让他被吞掉，之后的所有的影响都是叠加的，这就是多次的杂糅保证散列均匀，防止出现冲突的大量出现
3.x掩码右移24位移动到刚才的5-8位哪里在对5-8位进行第二次杂糅
4.我们定时清空高四位，实际上这部操作我们完全没有必要，但是算法要求，因为我们下一次的左移会自动吞掉这四位//这里存疑，不会减少我们的hash的范围？
5.str递增，引入下一个字符进行杂糅
6.返回一个缺失了最高符号位的无符号数（为了之后防止用到了有符号的时候造成的溢出）作为最后的hash值
```cpp
// ELF Hash Function
unsigned int ELFHash(char *str)
{
	unsigned int hash = 0;
	unsigned int x = 0;

	while (*str)
	{
		hash = (hash << 4) + (*str++);//hash左移4位，把当前字符ASCII存入hash低四位。 
		if ((x = hash & 0xF0000000L) != 0)
		{
			//如果最高的四位不为0，则说明字符多余7个，现在正在存第7个字符，如果不处理，再加下一个字符时，第一个字符会被移出，因此要有如下处理。
			//该处理，如果最高位为0，就会仅仅影响5-8位，否则会影响5-31位，因为C语言使用的算数移位
			//因为1-4位刚刚存储了新加入到字符，所以不能>>28
			hash ^= (x >> 24);
			//上面这行代码并不会对X有影响，本身X和hash的高4位相同，下面这行代码&~即对28-31(高4位)位清零。
			hash &= ~x;
		}
	}
	//返回一个符号位为0的数，即丢弃最高位，以免函数外产生影响。(我们可以考虑，如果只有字符，符号位不可能为负)
	return (hash & 0x7FFFFFFF);
}
```
[^嵩天-Python语言程序设计]:嵩天-Python语言程序设计


  [1]: http://img.blog.csdn.net/20160216223141915?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center
  [2]: http://www.matrix67.com/blogimage/200808233.gif

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI3MTgwODc0OF19
-->