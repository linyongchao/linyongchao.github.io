---
layout: post
title:  Redis 过期策略的实现
date:   2020-04-20 10:59:54
categories: Redis
---

* content
{:toc}

本文参考：[这里](https://www.cnblogs.com/xiaotime/p/9767253.html)

## 原理

### FIFO

按照“先进先出（First In，First Out）”的原理淘汰数据，正好符合队列的特性，数据结构上使用队列 Queue 来实现。

步骤如下：
1. 新访问的数据插入FIFO队列尾部，数据在FIFO队列中顺序移动
2. 淘汰FIFO队列头部的数据

### LRU

LRU（Least recently used，最近最少使用）算法根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”。

最常见的实现是使用一个链表保存缓存数据，详细算法实现如下：
1. 新数据插入到链表头部
2. 每当缓存命中（即缓存数据被访问），则将数据移到链表头部
3. 当链表满的时候，将链表尾部的数据丢弃

### LFU

LFU（Least Frequently Used）算法根据数据的历史访问频率来淘汰数据，其核心思想是“如果数据过去被访问多次，那么将来被访问的频率也更高”。

LFU的每个数据块都有一个引用计数，所有数据块按照引用计数排序，具有相同引用计数的数据块则按照时间排序。

具体实现如下：
1. 新加入数据插入到队列尾部（因为引用计数为1）
2. 队列中的数据被访问后，引用计数增加，队列重新排序
3. 当需要淘汰数据时，将已经排序的列表最后的数据块删除

## 实现

### FIFO

	#include "stdio.h"
	#include "stdlib.h"
	
	#define OK 1
	#define ERROR 0
	#define OVERFLOW -1
	
	typedef int status;
	typedef int ElemType;
	typedef status (*fp)(ElemType a,ElemType b);
	
	typedef struct QNode{
		ElemType data;
		QNode* next;
	}QNode;
	
	typedef struct Queue
	{
		// 缓存个数
		int nums;
		// 当前个数
		int totals;
		// 头结点
		QNode *front;
		// 尾结点
		QNode *rear;
	}Queue;
	
	status InitQueue(Queue &Q);
	status Locate_QNode(Queue Q,ElemType e,fp compare);
	status equal(ElemType a,ElemType b);
	status Add_QNode(Queue &Q,ElemType newQ,ElemType &oldQ);
	status EnQueue(Queue &Q,ElemType e);
	status DeQueue(Queue &Q,ElemType e);
	
	
	status InitQueue(Queue &Q){
		int nums;
		printf("please input cache nums:");
		scanf("%d",&nums);
		if(nums<1) return ERROR;
		Q.nums = nums;
		Q.totals = 0;
		Q.front = (QNode*)malloc(sizeof(QNode));
		Q.front->next =  NULL;
		Q.rear = Q.front;
		return OK;
	}
	
	// 查找队列是否有对应的缓存
	status Locate_QNode(Queue Q,ElemType e,fp compare){
		QNode *q1=Q.front;
		while(q1){
			if(compare(q1->data,e)) return OK;
			q1 = q1->next;
		}
		return ERROR;
	}
	
	status Add_QNode(Queue &Q,ElemType newQ,ElemType &oldQ){
		// 在尾部添加新的结点
		EnQueue(Q,newQ);
		// 缓存个数尚未满，个数增加
		if(Q.totals<Q.nums){
			Q.totals++;
		}else{
		// 缓存个数已满，去除头结点
			DeQueue(Q,oldQ);
		}
		return OK;
	}
	
	status EnQueue(Queue &Q,ElemType e){
		QNode *p;
		p = (QNode*)malloc(sizeof(QNode));
		if(!p) exit(OVERFLOW);
		p->data = e;
		p->next = NULL;
		Q.rear->next = p;
		Q.rear = p;
		return OK;
	}
	
	status DeQueue(Queue &Q,ElemType e){
		QNode *p;
		if(!Q.front->next) return ERROR;
		p = Q.front->next;
		Q.front->next = p->next;
		e = p->data;
		free(p);
		// 最后一个结点了
		if(!Q.front->next) Q.rear = Q.front;
		return OK;
	}
	
	status equal(ElemType a,ElemType b){
		return a==b?OK:ERROR;
	}
	
	int main(int argc, char const *argv[])
	{
		// 测试缓存页面队列,不同数字代表不同的缓存页面
		ElemType a[] = { 1,2,3,4,2,1,5,6,2,1,2,3,7,6,3,2,1,2,3,6};
		int count = sizeof(a)/sizeof(a[0]);
		// j为置换次数，temp为置换出来的页面
		int i, j=0,temp;
		Queue Q;
		InitQueue(Q);
		for(i = 0; i < count;i++){
			if(!Locate_QNode(Q,a[i],equal)){
				Add_QNode(Q,a[i],temp);
				j++;
			}
		}
		printf("页面置换次数%d\n",j);
		return 0;
	}

### LRU

	#include "stdio.h"
	#include "stdlib.h"
	
	#define OK 1
	#define ERROR 0
	#define OVERFLOW -1
	
	typedef int status;
	typedef int ELemType;
	typedef status (*fp) (ELemType, ELemType);
	
	// 缓存页面结点,用链表表示
	typedef struct LNode
	{
		ELemType data;
		struct LNode *next;
	}LNode,*LinkList;
	
	// 缓存页面
	typedef struct 
	{
		int num;// 允许的缓存数
		int total;// 当前缓存数
		struct LNode *Elem;
	}cache;
	
	status CreateCache(cache &C);
	status AddLNode_C(cache &C,ELemType e);
	status Locate_C(cache &C,ELemType e,fp compare);
	void ShowLink(cache c);
	status equal(ELemType a,ELemType b);
	
	// 初始化LRU算法
	status CreateCache(cache &C){
		int i;
		// 输入页面缓存个数
		printf("please input num:");
		scanf("%d",&i);
		if(i<1) return ERROR;
		C.num=i;
		C.total=0;
		C.Elem=NULL;
		return OK;
	}
	
	// 并查看当前内存个数是否大于可存放个数,大于则删除最后元素,最后增加新元素到第一节点,
	status AddLNode_C(cache &C,ELemType e){
		LNode *p,*q;
		// 把新结点放到开头
		p=(LinkList)malloc(sizeof(LNode));
		if(!p) exit(OVERFLOW);
		p->next=C.Elem;
		p->data=e;
		C.Elem=p;
		if(C.total==C.num){
			q=C.Elem;
			// 寻找最后一个结点
			while(q->next){
				p=q;
				q=q->next;
			}
			p->next=NULL;
			printf("删除了缓存页面尾部的结点\n");
			free(q);
		}else{
			C.total++;
		}
		printf("在缓存页面头部新增页面结点\n");
		return OK;
	}
	
	// 遍历缓存C查找是否有元素e,有则放到第一个结点,没有则返回错误
	status Locate_C(cache &C,ELemType e,fp compare){
		LNode *p,*q;
		q=C.Elem;
		p=C.Elem;
		// 不断遍历元素去寻找符合compare关系的元素
		while(p!=NULL){
			if(compare(p->data,e)){
				// 不是头结点,则需要移动到头结点
				if(q!=p){
					// 连接断点
					q->next=p->next;
					// 把p移到头结点前面,并把缓存记录的结点记录一下
					p->next=C.Elem;
					C.Elem=p;
				}
				printf("在原有的缓存页面里面找到了要使用的页面\n");
				return OK;
			}
			q=p;
			p=p->next;
		}
		printf("在原缓存里面没有找到要使用的页面\n");
		return ERROR;
	}
	void ShowLink(cache c){
		LinkList p;
		p=c.Elem;
		while(p!=NULL){
			printf("%d\n",p->data);
			p=p->next;
		}
	}
	
	// 判断两个缓存页面是否相同
	status equal(ELemType a,ELemType b){
		return a==b?OK:ERROR;
	}
	
	int main(int argc, char const *argv[])
	{
		// 测试缓存页面队列,不同数字代表不同的缓存页面
		ELemType a[] = { 1,2,3,4,2,1,5,6,2,1,2,3,7,6,3,2,1,2,3,6};
		int count = sizeof(a)/sizeof(a[0]);
		int i, j=0;
		cache C;
		CreateCache(C);
		for(i = 0; i < count;i++){
			if(!Locate_C(C,a[i],equal)){
				AddLNode_C(C,a[i]);
				j++;
			}
		}
		printf("页面置换次数%d\n",j);
		return 0;
	}

### LFU

	#include "stdio.h"
	#include "stdlib.h"
	
	#define OK 1
	#define ERROR 0
	#define OVERFLOW -1
	
	typedef int status;
	typedef int ElemType;
	
	typedef struct {
		ElemType data;
		int nums;
	}Node;
	
	typedef struct 
	{
		Node *base;
		int length;
		int listsize;
	}Cache;
	
	status InitCache(Cache &L);
	status ListInsert_Sq(Cache &L , int i, ElemType e);
	status Push(Cache &L ,  ElemType e);
	status Pop(Cache &L);
	status EQ(ElemType a,ElemType b);
	status LT(ElemType a,ElemType b);
	status ShowCache(Cache L);
	int LocateCache_nums(Cache L,ElemType e);
	int LocateCache_data(Cache L,ElemType e);
	status start(Cache &L , ElemType e);
	
	
	
	int main(int argc, char const *argv[])
	{
		Cache L;
		int arr[] = { 1,2,3,4,2,1,5,6,2,1,2,3,7,6,3,2,1,2,3,6};
		int count;
		int i;
		count = sizeof(arr)/sizeof(int);
		InitCache(L);
		for(i=0;i<count;i++){
			printf("-----%d\n",i+1);
			start(L,arr[i]);
			ShowCache(L);
		}
		return 0;
	}
	
	
	status InitCache(Cache &L){
		if(!L.base) exit(OVERFLOW);
		printf("please input cache nums:");
		scanf("%d",&L.listsize);
		L.base = (Node*)malloc(sizeof(Node)*L.listsize);
		L.length = 0;
		return OK;
	}
	/**
	 * 1.先查找是否在缓存里面
	 * 2.在找出位置，返回位置数
	 * 3.然后查出的相同次数缓存的第一个位置
	 * 4.把缓存调到该位置的前一个。：1.取出该元素，2该元素的原来位置到第一个元素的位置之间元素向后移一位，插入元素到第一个位置
	 * 5.不在则判断是否已满
	 * 6.不满直接插入元素到最后位置，否则删除最后元素，然后插入元素
	 * @param  L [description]
	 * @param  e [description]
	 * @return   [description]
	 */
	status start(Cache &L , ElemType e){
		int site,nums,firstElem,temp;
		Node p;
		// 查找有该元素
		if((site=LocateCache_data(L,e))!=-1){
			nums = L.base[site].nums;
			firstElem = LocateCache_nums(L,nums);
			printf("%d %d\n",nums,firstElem );
			p = L.base[site];
			// 把顺序表的结点一个个向后移一位
			for(temp=firstElem;temp<site;temp++){
				L.base[temp+1] = L.base[temp];
				printf("fuckfuck\n");
			}
			// 顺序表该增加调用的改为第一个
			L.base[firstElem] = p;
			L.base[firstElem].nums++;
		}else{
			// 没有该元素，直接删除最后一位，增加新的元素
			if(L.length==L.listsize)
				Pop(L);
			Push(L,e);
		}
	}
	
	
	// 查找第一个出现的对应元素，折半查找元素（根据使用次数）
	int LocateCache_nums(Cache L,int nums){
		int low,high,mid;
		low = L.length-1;
		high = 0;
		// printf("%d\n", mid);exit(-1);
		while(low >= high){
			mid = (high+low)/2;
			if(EQ(L.base[mid].nums,nums)){
				// 如果出现了相同的次数，查找第一个出现的次数
				while(mid!=0){
					// 查找前一个元素是否符合次数
					if(!EQ(L.base[--mid].nums,nums)){
						mid++;
						break;
					}
				}
				return mid;
			}else {
				if(LT(L.base[mid].nums,nums)) low = mid-1;
				else high = mid+1;
			}
		}
		return -1;
	}
	
	// 查找元素
	int LocateCache_data(Cache L,ElemType e){
		int i=0;
		// 设置哨兵
		L.base[L.length].data = e;
		for(;!EQ(L.base[i].data,e);i++);
		if(i==L.length) return -1;
		else return i;
	}
	
	// 在尾部插入元素，调用次数为1
	status Push(Cache &L , ElemType e){
		Node *p;
		if(L.length>=L.listsize) return ERROR;
		p = (Node*)malloc(sizeof(Node));
		p->data = e;
		p->nums = 1;
		L.base[L.length] = *p;
		L.length++;
	}
	
	// 删除尾部元素
	status Pop(Cache &L){
		if(L.length<1) return ERROR;
		L.length--;
	}
	
	status ShowCache(Cache L){
		int i=0;
		while(i<L.length){
			printf("cache Id nums:%d %d\n",L.base[i].data,L.base[i].nums);
			i++;
		}	
	}
	
	status EQ(ElemType a,ElemType b){
		return a==b?OK:ERROR;
	}
	status LT(ElemType a,ElemType b){
		return a<b?OK:ERROR;
	}

