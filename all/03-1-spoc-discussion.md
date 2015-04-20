# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```
- [x]  

>  最先匹配优势:空闲分区列表按地址顺序排序，简单，在高地址空间有大块的空闲分区；劣势是会产生外部碎片，分配大块时很慢。   
最优匹配优势：可以避免大的空闲分区被拆分、可以减小外部碎片的大小、相对简单；劣势：会产生外部碎片、释放分区较慢、容易产生很多无用的小碎片。    
最差匹配优势：中等大小的分配较多时效果好、避免出现太多小碎片；劣势：释放分区较慢、会产生外部碎片、容易破坏大的空闲分区，后续难以分配大的分区。    
buddy system分配算法优势：分配大块分区效果好、资源利用率高；劣势：分配小块分区较慢，会产生内部碎片、不符合对齐条件时不能合并分区，导致不能分配较大的块。   
优点:分配和回收速度快、算法简单,当一个大小为2K字节的块释放后,存储管理只需要搜索2K字节大小的块以判定是否需要合并。而那些允许以任意形式分割内存的策略的算法需要搜索整个空闲块表。     
缺点:伴系统的内存利用率是低效的,这主要是所有的内存请求都必须以2的幂次方大小的空间来满足。比如应用程序申请20kB的空间,系统必须分配32kB的空间     
 

自己提出的一种分配算法：对分配块的大小进行分析，较大、中等时采用最差，较小时采用最优。

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)


```
如何表示空闲块？ 如何表示空闲块列表？ 
[(start0, size0),(start1,size1)...]
在一次malloc后，如果根据某种顺序查找符合malloc要求的空闲块？如何把一个空闲块改变成另外一个空闲块，或消除这个空闲块？如何更新空闲块列表？
在一次free后，如何把已使用块转变成空闲块，并按照某种顺序（起始地址，块大小）插入到空闲块列表中？考虑需要合并相邻空闲块，形成更大的空闲块？
如果考虑地址对齐（比如按照4字节对齐），应该如何设计？
如果考虑空闲/使用块列表组织中有部分元数据，比如表示链接信息，如何给malloc返回有效可用的空闲块地址而不破坏
元数据信息？
伙伴分配器的一个极简实现
http://coolshell.cn/tag/buddy
```

>  学号2012011278，实现2：最先匹配   

实现了初始化（内存预设为1024）、分配、释放合并空闲空间等功能    
测试用例为占用内存128、256、128、32、512的5个进程p1、p2、p3、p4、p5   
一开始先依次为其分配内存，其中前四个可以分配成功，第五个失败     
因此程序输出的前4行为1表示成功，第5行为0表示失败    
之后先后释放p2、p1、p3，因此前半段的空闲内存为256->354->512   
因此接下来3行的输出为1表示释放成功   
之后再次为p5分配内存，此时分配成功，最后一行输出为1，表示释放的空闲空间合并成功

```
#include<iostream>
using namespace std;

struct process
{
	int size;
	int pos;
	process()
	{
		size = -1;
		pos = -1;
	}
	process(int s)
	{
		size = s;
		pos = -1;
	}
};

struct block
{
	int start;
	int size;
	block* prev;
	block* next;	
	block(int a, int b)
	{
		start = a;
		size = b;
		prev = NULL;
		next = NULL;
	}	
	block()
	{		
		start = 0;
		size = 0;
		prev = NULL;
		next = NULL;
	}
	
	void operator = (block newone)
	{
		start = newone.start;
		size = newone.size;
		prev = newone.prev;
		next = newone.next;		
	}
	
	static block* init()
	{
		block* header = new block(0, 1024);
		return header;
	}
	
	bool merge(block* current)	
	{
		if(current->prev != NULL)
		{
			block* temp = current->prev;
			if(temp->start + temp->size == current->start)
			{
				temp->size += current->size;
				if(current->next!=NULL)
					current->next->prev = temp;
				temp->next = current->next;
				current = temp;
			}
		}
		if(current->next != NULL)
		{
			block* temp = current->next;
			if(current->start + current->size == temp->start)
			{
				current->size += temp->size;
				if(temp->next!=NULL)
					temp->next->prev = current;
				current->next = temp->next;
			}
		}
		return true;
	}
	
	bool alloc(process& p)
	{
		block* temp;
		if(size >= p.size)
		{
			p.pos = start;
			start += p.size;
			size -=p.size;
			return true;
		}
		temp = next;
		while(temp != NULL)
		{
			if(size >= p.size)
			{
				p.pos = start;
				start += p.size;
				size -=p.size;
				return true;
			}
			else 
				temp = temp->next;
		}
		return false;
	}
	
	bool free(process& p)
	{	
		block* temp;
		block* pretemp;
		temp = this;
		
		if(start > p.pos)//在开头加
		{
			block* newone = new block(start, size);
			newone->next = next;
			newone->prev = this;
			if(next != NULL)
				next -> prev = newone;
			next = newone;
			start = p.pos;
			size = p.size;
			//cout << "在开头加\t" << newone->start << " " << newone->size << endl;
			return merge(newone);
		}
		
		while((temp != NULL) && (temp->start < p.pos))
		{
			pretemp = temp;
			temp = temp->next;
		}
		if(temp == NULL)//表明要在最后加
		{
			block* newone = new block(p.pos, p.size);
			newone->prev = pretemp;
			pretemp->next = newone;
			return merge(newone);
		}
		else//在中间加
		{
			block* newone = new block(p.pos, p.size);
			newone->next = temp;
			newone->prev = pretemp;
			pretemp->next = newone;
			temp->prev = newone;
			return merge(newone);
		}
	}
};

int main()
{
	process p1(128);
	process p2(256);
	process p3(128);
	process p4(32);
	process p5(512);
	block* qw = block::init();
	//cout << qw->start << endl << qw->size << endl;
	cout << qw->alloc(p1) << " " << p1.pos << endl;
	cout << qw->alloc(p2) << " " << p2.pos << endl;
	cout << qw->alloc(p3) << " " << p3.pos << endl;
	cout << qw->alloc(p4) << " " << p4.pos << endl;
	cout << qw->alloc(p5) << " " << p5.pos << endl;
	cout << qw->free(p2) << " " << qw->size << endl;
	cout << qw->free(p1) << " " << qw->size << endl;
	cout << qw->free(p3) << " " << qw->start << " " << qw->size << endl;
	cout << qw->alloc(p5) << endl;
	system("pause");
}
```

--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

>  把逻辑地址空间转换成物理地址空间，可以保护独立地址空间，也可以共享相同的内存。

L1和L2高速缓存的结构和工作机理？

>  结构：同CPU一起在处理器中，由硬件实现其操作。   
工作机理：CPU需要访问数据或者指令时，先访问L!与L2，若已经有相应的内容，则可以直接拿到，否则去内存查找。

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  程序用高级语言进行编写，高级语言进行编译转换成机器认识的汇编指令；   
再进行汇编变成二进制代码，符号由字符串变成地址空间里的位置；   
然后进行链接，将不同的模块与函数库排在一起，之后就可以使用到其他模块里的位置；  
程序加载运行时，位置会发生变化，进行重定位

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  内碎片：分配单元内部的未被使用内存（因 取整对齐而无法使用）   
外碎片：分配单元之间的未被使用内存（由于过小而无法使用）

为什么最先匹配会越用越慢？

- [x]  

>  从空闲分区列表里找，使用最先找到的比其大的，因为使用了前面地址空间，越往后能使用的空间越靠后，查找时间越长。

为什么最差匹配会的外碎片少？

- [x]  

>  每次都使用最大的空闲分区，使用后会留下较大的空闲分区，减少了小碎片。

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  最先：前后检查时候有空闲分区可以合并     
最佳：查找临近（地址临近）的空闲分区合并，合并后并调整空闲分区列表顺序，比较复杂
最差：同最佳


### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



