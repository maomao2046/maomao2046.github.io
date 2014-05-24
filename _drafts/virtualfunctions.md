---
layout: post
title: 从对象内存结构理解虚函数表
categories: cplusplus
---

最近抽空浏览了一遍《COM原理与应用》，一本老书了，COM技术在我工作中运用得不多，但是接口设计规范和标准这一套东西还是能带给我一些有用的实践经验和启发的。在读到第二章《COM对象和接口》的时候，看到虚函数表的一些相关知识，这些之前倒是也都知道，但是从来没有试着自己描述过，所以老觉得理解得不够彻底，那么……就试着结合一些小的代码段描述一下看，权当笔记加深记忆。  

*以下代码及运行结果基于Win7_64 + GCC 4.7.2环境，其它环境下可能程序运行结果等有出入，但是原理相通。*

###类对象的内存结构
先来看看有与没有虚函数的类的对象的内存结构的不同之处：  

* **无虚函数的对象的内存结构**   
无虚函数的对象的内存结构大致为：  
![Object with no virtual function](/images/posts/cplusplus/objectwithnovirtual.png)  
验证如下：  

```c++
#include <stdio.h>

class CObj
{
public:
    CObj(int nData1, int nData2) : m_nData1(nData1), m_nData2(nData2) {}
    ~CObj() {}

    void Func() { printf("CObj::Func()\n"); }

private:
    int m_nData1;
    int m_nData2;
};

int main()
{
    CObj obj(11 ,22);

    return 0;
}
```

来看看obj的实际内存分布：  
![Object with no virtual function](/images/posts/cplusplus/objectwithnovirtual_mem.png)

* **有虚函数的对象的内存结构**  
有虚函数的对象的内存结构大致为：  
![Object with virtual functions](/images/posts/cplusplus/objectwithvirtual.png)  
验证如下：  

```c++
#include <stdio.h>

class CObj
{
public:
    CObj(int nData1, int nData2) : m_nData1(nData1), m_nData2(nData2) {}
    ~CObj() {}

    virtual void FuncA() { printf("CObj::FuncA()\n"); }
    virtual void FuncB() { printf("CObj::FuncB()\n"); }

private:
    int m_nData1;
    int m_nData2;
};

int main()
{
    CObj obj1(10,20);
    CObj obj2(11,22);

    return 0;
}
```

来看看obj1和obj2的实际内存结构：  
![Object with virtual functions](/images/posts/cplusplus/objectwithvirtual_mem.png)  

###单继承的类对象的内存结构
子类覆盖父类虚函数之后虚函数表的变化可以通过对比明显的得出，这即是多态的实现机制。  
* **子类无覆盖父类的虚函数**  
这种类型的子类对象的内存结构如图：  
![derive no override](/images/posts/cplusplus/derivenooverride.png)  
验证如下：  

```c++
#include <stdio.h>

class CBase
{
public:
    CBase() { m_nData1 = 10; }
    virtual void FuncA() { printf("CBase::FuncA\n"); }

private:
    int m_nData1;
};

class CDerive : public CBase
{
public:
    CDerive() { m_nData2 = 20; }
    virtual void FuncB() { printf("CDerive::FuncB\n"); }
    
private:
    int m_nData2;
};

int main()
{
    CDerive *pDerive = new CDerive;

    typedef void (* FuncType) ();
    
    printf("pDerive->m_nData1 is %d\n", (int)(*((char*)pDerive + sizeof(FuncType*)/sizeof(char))));
    printf("pDerive->m_nData2 is %d\n", (int)(*((char*)pDerive + sizeof(FuncType*)/sizeof(char) + sizeof(int)/sizeof(char))));

    FuncType funcA = *(FuncType*)(*(int*)pDerive);
    funcA();
    FuncType funcB = *((FuncType*)(*(int*)pDerive) +1);
    funcB();

    return 0;
}
```

输出为：  

```
pDerive->m_nData1 is 10
pDerive->m_nData2 is 20
CBase::FuncA
CDerive::FuncB
```

* **子类有覆盖父类的虚函数**  
这种类型的子类对象的内存结构如图：  
![derive override](/images/posts/cplusplus/deriveoverride.png)  
验证如下：  

```c++
#include <stdio.h>

class CBase
{
public:
    CBase() { m_nData1 = 10; }
    virtual void FuncA() { printf("CBase::FuncA\n"); }

private:
    int m_nData1;
};

class CDerive : public CBase
{
public:
    CDerive() { m_nData2 = 20; }
    virtual void FuncA() { printf("CDerive::FuncA\n"); }    // add this line
    virtual void FuncB() { printf("CDerive::FuncB\n"); }
    
private:
    int m_nData2;
};

int main()
{
    CDerive *pDerive = new CDerive;

    typedef void (* FuncType) ();
    
    printf("pDerive->m_nData1 is %d\n", (int)(*((char*)pDerive + sizeof(FuncType*)/sizeof(char))));
    printf("pDerive->m_nData2 is %d\n", (int)(*((char*)pDerive + sizeof(FuncType*)/sizeof(char) + sizeof(int)/sizeof(char))));

    FuncType funcA = *(FuncType*)(*(int*)pDerive);
    funcA();
    FuncType funcB = *((FuncType*)(*(int*)pDerive) +1);
    funcB();

    return 0;
}
```

输出为：

```
pDerive->m_nData1 is 10
pDerive->m_nData2 is 20
CDerive::FuncA
CDerive::FuncB
```

###普通多继承的类对象的内存结构
* 

###钻石结构的类对象的内存结构