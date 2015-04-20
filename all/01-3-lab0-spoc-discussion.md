# lab0 SPOC思考题

## 个人思考题

---

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。
- [x]  

>  http://www.imada.sdu.dk/Courses/DM18/Litteratur/IntelnATT.htm
>   基本看懂了

虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？
- [x]  

>   中断、虚存管理、段页式管理、进程间交互、资源互斥、用户态与和核心态之间的切换、分时


哪些困难（请分优先级）会阻碍你自主完成lab实验？
- [x]  

>   实验庞大、实验环境搭配困难、实验使用到的软件不熟悉、不能熟练使用git

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？
- [x]  

>   逻辑地址通过段式管理映射得到线性地址，线性地址通过页式管理映射得到物理地址

了解函数调用栈对lab实验有何帮助？
- [x]  

>   了解资源的分配与互斥、进程间的调度

你希望从lab中学到什么知识？
- [x]  

>   中断、虚存管理、段页式管理、进程间交互、资源互斥、用户态与和核心态之间的切换、分时

---

## 小组讨论题

---

搭建好实验环境，请描述碰到的困难和解决的过程。
- [x]  

> 基本搭建完成。网络不好，下载太慢--等；不太熟悉步骤--请教同学

熟悉基本的git命令行操作命令，从github上的[ucore git repo](http://www.github.com/chyyuu/ucore_lab)下载ucore lab实验
- [x]  

> git下载完成，并基本熟悉git的使用；实验下载完成

尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1
- [x]  

> 尝试调试完成，效果还可以

对于如下的代码段，请说明”：“后面的数字是什么含义
```
/* Gate descriptors for interrupts and traps */
struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
};
```
- [x]  

> 位域大小，表示实际需要用于存储信息的空间大小，可以节约空间

对于如下的代码段，
```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 0,1,2,3);
```
请问执行上述指令后， intr的值是多少？
- [x]  

> intr = 65538

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
- [x]  

> 该.h文件中定义了双向的链表（形成一个环），并定义了链表的初始化、增加（向前加、向后加）、删除、判断是否为空等函数功能。
我编写了一个小程序，有3个链表，首位相连形成一个环，然后对这个环形链表进行计数，结果为3，程序正确。
代码如下：
```
#include<iostream>
#include "list.h"
using namespace std;
int main()
{	
	list_entry_t* list0 = new list_entry_t;
	list_entry_t* list1 = new list_entry_t;
	list_entry_t* list2 = new list_entry_t;	
	list_init(list0);
	list_add(list0,list1);
	list_add(list1,list2);	
	list_entry_t* temp = list_next(list0);
	int num;
	num = 1;	
	while(temp != list0)
	{
		num ++;
		temp = list_next(temp);
	}	
	cout << num << endl;	
	return 0;
}


---

## 开放思考题

---

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。
- [x]  

>  由于本人的能力有限，所以不参与大实验

---
