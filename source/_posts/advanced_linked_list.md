---
title: 数据结构四之链表进阶
date: 2014-12-22 23:19:21
tags:
- 数据结构
- c++
categories:
- c++
---

只谈一下单链表, 链表实在是太重要, 是前面两篇说算法博客的基础, 了解了其应用和衍生, 再去了解其本身就有动力了

这是一篇偏向单链表进阶的博客, 并不会讲单链表的建立/增加/删除等等, 而且这篇博客大多数只说思想不写代码(因为其实蛮简单的..)

# 存储结构
```
typedef struct Node
{
	DataType data;
	struct Node *next;
}Node, *Node_Ptr;
```

{% asset_img advanced_linked_list_1.png %}

<!-- more -->

# 在O(1)时间删除链表结点

给定单链表的头指针和一个结点指针, 定义一个函数在 O(1)时间删除该结点.

思路 : 

如果要遍历找到该结点的前一个结点p, 来改变结点p的下一个结点指向的这种解法肯定是O(n)时间复杂度了. 
**那是不是一定需要得到被删除的结点的前一个结点呢？**
答案是否定的。我们可以很方便地得到要删除的结点的下一个结点。如果我们把下一个结点的内容复制到需要删除的结点上覆盖原有的内容，再把下一个结点删除，那是不是就相当于把当前需要删除的结点删除了？

# 找一个单链表的中间结点


算法思想 : 

(**快慢指针的使用**)设置两个指针，一个每次移动两个位置，一个每次移动一个位置，当第一个指针到达尾节点时，第二个指针就达到了中间节点的位置

# 判断链表中是否有环


算法思想 : 

(**快慢指针的使用**)链表中有环，其实也就是自相交. 用两个指针pslow和pfast从头开始遍历链表，pslow每次前进一个节点，pfast每次前进两个结点，若存在环，则pslow和pfast肯定会在环中相遇，若不存在，则pslow和pfast能正常到达最后一个节点

# 判断两个链表是否相交, 假设两个链表均不带环


算法思想 : 

如果两个链表相交于某一节点，那么在这个相交节点之后的所有节点都是两个链表所共有的。也就是说，如果两个链表相交，那么最后一个节点肯定是共有的。先遍历第一个链表，记住最后一个节点，然后遍历第二个链表，到最后一个节点时和第一个链表的最后一个节点做比较，如果相同，则相交，否则不相交。

# 从尾到头打印链表

有两种解法 : 

- **反转链表解法** : 反转链表之后再从头到尾打印 (这样会改变原来的链表)
- **栈存储解法** : 用栈存储之后再逐个出栈一一打印 (这样不会改变原来的链表)

## 反转链表解法

比如一个链表:
头指针->A->B->C->D->E
反转成为:
头指针->E->D->C->B->A

### 算法思想 : 

1. 取一个指针 pFirst 来一直指向A
2. 取一个指针 pTemp 来临时保存一下头指针后面的那个元素

第一轮 : 头指针->A->B->C->D->E
第二轮 : 头指针->B->A->C->D->E
第三轮 : 头指针->C->B->A->D->E
第四轮 : 头指针->D->C->B->A->E
第五轮 : 头指针->E->D->C->B->A

### 算法cpp实现：

手写的代码， 已经跑过了，可直接用
下面代码中反转函数为 ReverseList ， 且有详细注释以及总结

``` c++
#include <stdio.h>

struct TList
{
	struct TList *pNext;
	void *pData;
};
typedef struct TList *LPTLIST;

/* 将单链表反转, 要求只能扫描链表一次.
*@param ppstHead 指向链表首节点的指针
*/
void ReverseList(LPTLIST *ppstHead)
{
	if (!ppstHead)
		return;

	// 我们只用上述算法思想中第二轮来说明一下此算法, 即为 "第二轮 : 头指针->B->A->C->D->E"
	LPTLIST pFirst = (*ppstHead)->pNext; // 这个指针一直指向着原来链表头指针后面的那个元素（即原第一个元素， 这个指针的指向一直都不会变， 一直都是指向A）

	while (pFirst && pFirst->pNext) // 需要两个判断, 不然当pFirst为NULL的时候会出错, 且pFirst->pNext为NULL的时候也没必要继续循环了
	{
		LPTLIST pTemp = (*ppstHead)->pNext; // 临时保存一下头指针后面的那个元素A
		(*ppstHead)->pNext = pFirst->pNext; // 把目前第一个元素A替换为原第一个元素的后面那个元素B : 头指针->B (第1步)
		pFirst->pNext = pFirst->pNext->pNext; // 原第一个元素A的pnext指到它后面的后面那个元素C : A->C (第2步)
		(*ppstHead)->pNext->pNext = pTemp; // 让B指向A : B->A (第3步)
		// 综上所述只需要3步
	}
	// 总结， 链表反转需要两个指针， 见上面两个指针, 一个pFirst, 一个pTemp
}

LPTLIST print_tlist(LPTLIST *ppstHead)
{
	LPTLIST origin_tlist = *ppstHead;
	if (*ppstHead)
	{

		while ((*ppstHead)->pNext)
		{
			*ppstHead = (*ppstHead)->pNext;
			printf("%d ->", *(int*)( (*ppstHead)->pData ) );
		}
		printf("\n");
	}
	return origin_tlist;
}

LPTLIST append_elem(LPTLIST *ppstHead, void *data)
{

	LPTLIST origin_tlist = *ppstHead;
	if (*ppstHead)
	{
		while ((*ppstHead)->pNext)
		{
			*ppstHead = (*ppstHead)->pNext;
		}
		LPTLIST new_node = new TList;
		new_node->pNext = NULL;
		new_node->pData = data;

		(*ppstHead)->pNext = new_node;
	}
	return origin_tlist;

}

int main()
{
	LPTLIST temp = NULL;
	print_tlist(&temp);

	LPTLIST test_tlist = new TList;
	test_tlist->pNext = NULL;
	test_tlist->pData = NULL;

	// LPTLIST origin_tlist = test_tlist;

	int elem1 = 1;
	int elem2 = 2;
	int elem3 = 7;

	test_tlist = append_elem(&test_tlist, &elem1);
	test_tlist = append_elem(&test_tlist, &elem2);
	test_tlist = append_elem(&test_tlist, &elem3);

	test_tlist = print_tlist(&test_tlist);

	ReverseList(&test_tlist);

	print_tlist(&test_tlist);

	return 0;
}
```


## 栈存储解法

``` c++

#include <stack>
#include <stdio.h>

using std::stack;

struct ListNode
{
	void *pData;
	ListNode *pNext;
};

void AppendListNode(ListNode *pHead, void *Data)
{
	if (!pHead)
	{
		return;
	}
	ListNode *pEndNode = NULL;
	ListNode *pTemp = pHead;
	while (pTemp)
	{
		pEndNode = pTemp;
		pTemp = pTemp->pNext;
	}
	ListNode *NewListNode = new ListNode;
	if (!NewListNode)
	{
		return;
	}
	NewListNode->pData = Data;
	NewListNode->pNext = NULL;
	pEndNode->pNext = NewListNode;
}

void PrintList(ListNode *pHead)
{
	if (!pHead)
	{
		return;
	}
	while (pHead = pHead->pNext)
	{
		printf( "%d ->", *(int *)(pHead->pData) );
	}
}

void PrintListReversingly_Iteratively(ListNode *pHead)
{
	if (!pHead)
	{
		return;
	}
	stack<ListNode> stackListNode;
	while (pHead = pHead->pNext)
	{
		stackListNode.push(*pHead);
	}

	while (!stackListNode.empty())
	{
		printf("%d -> ", *((int *)(stackListNode.top().pData)));
		stackListNode.pop();
	}
}

int main(int argc, char* argv[])
{
	ListNode* TestListNode = new ListNode;
	TestListNode->pData = NULL;
	TestListNode->pNext = NULL;

	int Data1 = 5;
	int Data2 = 6;
	int Data3 = 7;

	AppendListNode(TestListNode, &Data1);
	AppendListNode(TestListNode, &Data2);
	AppendListNode(TestListNode, &Data3);

	PrintListReversingly_Iteratively(TestListNode);

	return 0;
}

```