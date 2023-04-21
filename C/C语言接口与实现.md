# 我自己的总结

我想把口袋妖怪游戏移植到ESP32平台，这时候处理器、显示屏幕都变了，如果原游戏是模块化的，我只需要把改变的模块重新实现一遍就行了，从这里就可以看出来，模块化可以解耦，可以通用化，那该怎么做呢，也就是封装，游戏和显示分开实现，并不依赖，游戏调用不同的显示模块就可以在不同设备上显示

代码层级的实现呢，考虑socket，通过不同的参数可以获得不同的fd，后续的调用传入fd即可，所有的接口都是相同的，这就是抽象

什么时候需要模块化呢，游戏可能会在不同的屏幕显示、可能会在不同的处理器运行，我的ESP32播放可能会在不同的途径获取图片、可能使用不同的解码库，体会一下，当这个功能可能会有多种实现，或者说需要对接不同的方式时，就可以把这个功能实现为模块

模块化是一种思想，比如函数本身就是一种模块，传入不同的参数可以有不同的功能，可以复用

```c
typedef struct list_t *list_t;//这种类型无法解引用，虽然它是指针
list_t list;
*list;//非法的
```

## 19.2 信息隐藏

模块隐藏一些内部信息，比如实现方法，内置数据结构，技术上可以使用static定义私有变量与函数

# 接口

## 概念

为用户提供指定的功能，用户使用接口不依赖于具体实现，即用户不可以直接访问实现的数据，如果用户需要访问实现的数据，可以提供接口，如果没有提供接口，用户还需要访问实现的数据，说明接口设计不合理

## 封装

```c
link.h
	typedef struct link_t link_t;//头文件只声明了这种类型，但是没有定义
	void link_func(link_t *);
	link_t *link_new();
link.c
    struct link_t{
    	void *value;
    	struct link_t *next;
	};
main.c
	link_t link;//非法，因为不知道link_t的具体大小，无法分配内存
	link_t *link;//合法，因为指针大小是已知的，这也就要求库提供一个new函数
```

上面的方法确保了用户只可以实例化一个link_t类型的指针，但是因为无法访问link_t的成员变量，以此做到了对数据的封装

但是上述方法还有一定缺陷，比如我有两种链表（一个单链表一个双链表），如何实现？全部重新仿一遍吗？

```c
link.h
#define T link_t
//当typedef struct T T时，用户只能link_t *link;实例化
//当typedef struct T *T时，用户只能link_t link;实例化
//其实两种均可，用户都不可以解引用或访问成员变量，因为没有成员变量
typedef struct T *T;

struct T{
    void *(*insert)(T);
    ...
};
T t_new();
#undef T
```

```c
link.c
#include"list.h"
#define T link_t
//结构体t包含了T，也就是实现了接口T
typedef struct t
{
    struct T interface;
	...
}t;
//实现函数的参数类型依然是接口T，不可以是自己的类型
void t_insert(T interface)
{
    t *link = (t *)interface;//如果要访问t的成员变量，需要进行强转
    ...
}
//new函数返回类型依然是接口T，而不是自己本身
T t_new()
{
    t *link = (t *)calloc(1, sizeof(t));
    link->interface.insert = t_insert;//将实现函数注册
    return link;
}

#undef T
```

```c
main.c
#include"list.h"
int main()
{
    link_t link = t_new();
    link->insert(link);
    
    return 0;
}
```

# 反思

使用define和typedef是很好的封装技巧

使用接口而不是外部函数，调用者必须实例化才可以使用接口，并且封装在接口中函数避免了函数名重复的问题

接口必须通用化，谨慎设计接口函数。每个实现可能差异很大，如果需要一些特殊资源，可以通过初始化函数提供，因为初始化函数本身就是每个实现对外的接口

实现继承接口，实现实例化时申请的内存必不小于接口，所以将实现强制转换为接口类型是绝对安全的，前提是实现的第一个成员就是接口，保证首地址相同，而将实现转换为接口返回也是安全的，实际内存并不会截断，只是限制了调用者访问的范围，即只能访问接口的范围，这就做到了封装数据与方法，但是数据不对外开放

```c
#define T container_t
typedef struct T *T;
struct T
{
    void *(*add)(T, size_t);
    int   (*func)(T, int (*func)(void *));
    void  (*free)(T);
    int   (*num)(T);
    void *(*get)(T, int);
};
extern T new_link();
#undef T
```

```C
#define T container_t
#define L link_t
#define N node_t
typedef struct L *L;
typedef struct N *N;
struct L
{
	struct T interface;
	//调用者不可见节点
	struct N
	{
		void *item;
		N next;
	}*head;
	int num;
};
#undef N
#undef L
#undef T
```

