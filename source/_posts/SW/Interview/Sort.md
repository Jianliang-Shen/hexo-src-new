---
title: 排序算法
date: 2024-01-19 16:22:43
index_img: /img/sort/quick-sort.gif
tags: 
    - C/C++
    - Algorithm
    - Data Structure
categories: 
    - Interview
---
排序是很重要的一个算法，对于排序的定义，排序是按照关键字进行非递减或者非递增的顺序对原序列进行有序排列，排序分为`稳定排序`和`非稳定排序`，其中稳定排序是指在排序前两个关键词相同的相对顺序在排序后未发生改变。

<!-- more -->
按照内存的消耗区分，排序还分为内排序和外排序，内排的内存都在原序列内存中解决，外排序还需要额外的内存。排序算法这里介绍七种，包括：

- [冒泡排序](#冒泡排序)
- [简单选择排序](#简单选择排序)
- [直接插入排序](#直接插入排序)
- [希尔排序](#希尔排序)
- [堆排序](#堆排序)
  - [数据结构--堆](#数据结构--堆)
  - [堆排序原理](#堆排序原理)
  - [堆排序代码](#堆排序代码)
  - [堆排序测试](#堆排序测试)
- [归并排序](#归并排序)
- [快速排序](#快速排序)
  
<!-- more -->
## 冒泡排序

![](/img/sort/bubble-sort.gif)

两两比较相邻结果，按要求的顺序互换位置。每次循环都使得极值移动到最后

```txt
9  1  5  8  3  7  4  6👈2
9  1  5  8  3  7  4👈2  6
9  1  5  8  3  7👈2  4  6
9  1  5  8  3👈2  7  4  6
9  1  5  8👈2  3  7  4  6
9  1  5👈2  8  3  7  4  6
9  1  2  5  8  3  7  4  6  //1小于2因此不交换
9👈1  2  5  8  3  7  4  6
1  9  2  5  8  3  7  4  6
```

可以增加一个flag作为有序的标志，避免无意义的比较。复杂度O(N^2)。

## 简单选择排序

![](/img/sort/selection-sort.gif)

通过n-i次比较，找到m-i+1中的关键字最小的记录，并和第i个记录交换。复杂度O(N^2)。

```txt
9  1  5  8  3  7  4  6  2
|  |
↑←↓
1  9  5  8  3  7  4  6  2
   |                    |
   ↑←←←←←←←←←←←↓
1  2  5  8  3  7  4  6  9
```

```cpp
    vector<int> vt = {9, 1, 5, 8, 3, 7, 4, 6, 2};
    int min_pos = 0;
    for (int i = 0; i < vt.size(); i++) {
        min_pos = i;
        for (int j = i + 1; j < vt.size(); j++) {
            if (vt[j] < vt[min_pos]) {
                min_pos = j;  //更新位置
            }
        }
        if (min_pos != i) {  //交换位置
            int tmp = vt[i];
            vt[i] = vt[min_pos];
            vt[min_pos] = tmp;
        }
    }
```

## 直接插入排序

![](/img/sort/insertion-sort.gif)

将一个记录插入到一个已经排好序的有序表中，从而得到一个新的、记录数增1的有序表。复杂度O(N^2)。

```cpp
 vector<int> vt = {13, 5, 3, 4, 6, 2, 8, 22, 1, 7};
 int len = vt.size();
 int j;
 for (int i = 1; i < len; i++)
 {
  if (vt[i] < vt[i - 1])
  {
   int tmp = vt[i];              //临时保存vt[i]的值
   for (j = i - 1; vt[j] > tmp; j--)
   {
    vt[j + 1] = vt[j];    //将比vt[i]大的数全部向右移动1位
   }
   vt[j + 1] = tmp;
  }
 }
```

## 希尔排序

将复杂度提高到O(NlogN)。
基本有序：小的元素基本在左边，大的元素基本在右边
希尔排序原理：将无序的数组折中排为基本有序数组，再次折中进一步排为更“细致”的基本有序数组。

```cpp
 vector<int> vt = {0, 13, 7, 3, 4, 6, 2, 8, 22};   //有部分bug，vt[0]不参与排序
 int len = vt.size();
 int i, j;
 int increment = len;
 do
 {
  increment = increment / 3 + 1;     //具体的增量序列仍然没有较好的公式
  for (i = increment + 1; i < len; i++)
  {
   if (vt[i] < vt[i - increment])
   {
    int tmp = vt[i];
    vt[i] = vt[i - increment];
    vt[i - increment] = tmp;
   }
  }
 } while (increment > 1);
```

## 堆排序

### 数据结构--堆

`堆（Heap）`是每个结点的值都`大于等于`（或者小于等于）其左右孩子的结点的值的`完全二叉树`，前者称之为`大顶堆`，后者称之为`小顶堆`。
`完全二叉树`的定义：对一棵具有n个结点的二叉树按`层序遍历`，如果编号为i的结点与`同样深度的满二叉树`中编号为i的结点位置完全相同，则为完全二叉树。

```
这是一个合理的堆：
       90
     /    \
    70    80
   / \    / \
  60 10  40 50
  /\  
30  20
层序遍历[90 ,70 ,80 ,60 ,10 ,40 ,50 ,30 ,20]
编号     1   2   3   4   5   6   7   8   9
更具层序遍历结果，大顶堆满足：
k[i] >= k[2i]
k[i] >= k[2i+1]
1 <= i <= n/2
完全二叉树的当前结点编号为i，左孩为2i，右孩为2i+1
```

### 堆排序原理

![](/img/sort/heap-sort.gif)

堆排序要解决两个问题：

- 如何将无序数组转变为堆
  - 递归建立
- 如何处理大顶堆
  - 依次交换
  
### 堆排序代码

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <time.h>

using namespace std; //堆排序的核心是建堆,传入参数为数组，根节点位置，数组长度
// #define MAXSIZE 50000
#define MAXSIZE 2000000
void Heap_build(int a[], int root, int length)
{
 int lchild = root * 2 + 1; //根节点的左子结点下标
 if (lchild < length)    //左子结点下标不能超出数组的长度
 {
  int flag = lchild;   //flag保存左右节点中最大值的下标
  int rchild = lchild + 1; //根节点的右子结点下标
  if (rchild < length)  //右子结点下标不能超出数组的长度(如果有的话)
  {
   if (a[rchild] > a[flag]) //找出左右子结点中的最大值
   {
    flag = rchild;
   }
  }
  if (a[root] < a[flag])
  {
   //交换父结点和比父结点大的最大子节点
   swap(a[root], a[flag]);
   //从此次最大子节点的那个位置开始递归建堆
   Heap_build(a, flag, length);
  }
 }
}

void Heap_sort(int a[], int len)
{
 for (int i = len / 2; i >= 0; --i) //从最后一个非叶子节点的父结点开始建堆
 {
  Heap_build(a, i, len);
 }

 for (int j = len - 1; j > 0; --j) //j表示数组此时的长度，因为len长度已经建过了，从len-1开始
 {
  swap(a[0], a[j]);  //交换首尾元素,将最大值交换到数组的最后位置保存
  Heap_build(a, 0, j); //去除最后位置的元素重新建堆，此处j表示数组的长度，最后一个位置下标变为len-2
 }
}

void insertSort(int vt[], int len)
{
 //vector<int> vt = {13, 5, 3, 4, 6, 2, 8, 22, 1, 7};
 //int len = vt.size();
 int j;
 for (int i = 1; i < len; i++)
 {
  if (vt[i] < vt[i - 1])
  {
   int tmp = vt[i]; //临时保存vt[i]的值
   for (j = i - 1; vt[j] > tmp; j--)
   {
    vt[j + 1] = vt[j]; //将比vt[i]大的数全部向右移动1位
   }
   vt[j + 1] = tmp;
  }
 }
}
int main(int argc, char **argv)
{
 int a[MAXSIZE];
 for (int i = 0; i < MAXSIZE; i++)
 {
  a[i] = rand() % MAXSIZE;
 }
#if 1
 clock_t Start_time = clock();
 Heap_sort(a, MAXSIZE);
 clock_t End_time = clock();
 cout << "Total Heap_sort running time is: " << static_cast<double>(End_time - Start_time) / CLOCKS_PER_SEC * 1000 << " ms" << endl;
#endif
#if 0
 clock_t Start_time = clock();
 insertSort(a, MAXSIZE);
 clock_t End_time = clock();
 cout << "Total insertSort running time is: " << static_cast<double>(End_time - Start_time) / CLOCKS_PER_SEC * 1000 << " ms" << endl;
#endif
 for (size_t i = 0; i < 100; ++i)
 {
  cout << a[i] << " ";
 }
 cout << endl;
 return 0;
}
```

### 堆排序测试

```c
#include <iostream>
#include <vector>
#include <algorithm>
#include <time.h>

using namespace std; //堆排序的核心是建堆,传入参数为数组，根节点位置，数组长度
// #define MAXSIZE 50000
#define MAXSIZE 1000000
void Heap_build(int a[], int root, int length)
{
    int lchild = root * 2 + 1; //根节点的左子结点下标
    if (lchild < length)       //左子结点下标不能超出数组的长度
    {
        int flag = lchild;       //flag保存左右节点中最大值的下标
        int rchild = lchild + 1; //根节点的右子结点下标
        if (rchild < length)     //右子结点下标不能超出数组的长度(如果有的话)
        {
            if (a[rchild] > a[flag]) //找出左右子结点中的最大值
            {
                flag = rchild;
            }
        }
        if (a[root] < a[flag])
        {
            //交换父结点和比父结点大的最大子节点
            swap(a[root], a[flag]);
            //从此次最大子节点的那个位置开始递归建堆
            Heap_build(a, flag, length);
        }
    }
}

void Heap_sort(int a[], int len)
{
    for (int i = len / 2; i >= 0; --i) //从最后一个非叶子节点的父结点开始建堆
    {
        Heap_build(a, i, len);
    }

    for (int j = len - 1; j > 0; --j) //j表示数组此时的长度，因为len长度已经建过了，从len-1开始
    {
        swap(a[0], a[j]);    //交换首尾元素,将最大值交换到数组的最后位置保存
        Heap_build(a, 0, j); //去除最后位置的元素重新建堆，此处j表示数组的长度，最后一个位置下标变为len-2
    }
}

void insertSort(int vt[], int len)
{
    //vector<int> vt = {13, 5, 3, 4, 6, 2, 8, 22, 1, 7};
    //int len = vt.size();
    int j;
    for (int i = 1; i < len; i++)
    {
        if (vt[i] < vt[i - 1])
        {
            int tmp = vt[i]; //临时保存vt[i]的值
            for (j = i - 1; vt[j] > tmp; j--)
            {
                vt[j + 1] = vt[j]; //将比vt[i]大的数全部向右移动1位
            }
            vt[j + 1] = tmp;
        }
    }
}

void Merge(int arr[], int left, int mid, int right, int temp[])
{
    int i = left;    //左序列指针
    int j = mid + 1; //右序列指针
    int t = 0;       //临时数组指针
    while (i <= mid && j <= right)
    {
        if (arr[i] <= arr[j])
        {
            temp[t++] = arr[i++];
        }
        else
        {
            temp[t++] = arr[j++];
        }
    }
    while (i <= mid)
    { //将左边剩余元素填充进temp中
        temp[t++] = arr[i++];
    }
    while (j <= right)
    { //将右序列剩余元素填充进temp中
        temp[t++] = arr[j++];
    }
    t = 0;
    //将temp中的元素全部拷贝到原数组中
    while (left <= right)
    {
        arr[left++] = temp[t++];
    }
}

void MergeSort(int arr[], int left, int right, int temp[])
{
    if (left < right)
    {
        int mid = (left + right) / 2;
        MergeSort(arr, left, mid, temp);      //左边归并排序，使得左子序列有序
        MergeSort(arr, mid + 1, right, temp); //右边归并排序，使得右子序列有序
        Merge(arr, left, mid, right, temp);   //将两个有序子数组合并操作
    }
}

void quickSort(int a[], int l, int r)
{
    if (l < r)
    {
        int i, j, x;

        i = l;
        j = r;
        x = a[i];
        while (i < j)
        {
            while (i < j && a[j] > x)
                j--; // 从右向左找第一个小于x的数
            if (i < j)
                a[i++] = a[j];
            while (i < j && a[i] < x)
                i++; // 从左向右找第一个大于x的数
            if (i < j)
                a[j--] = a[i];
        }
        a[i] = x;
        quickSort(a, l, i - 1); /* 递归调用 */
        quickSort(a, i + 1, r); /* 递归调用 */
    }
}

int main(int argc, char **argv)
{
    int a[MAXSIZE];
    for (int i = 0; i < MAXSIZE; i++)
    {
        a[i] = rand() % MAXSIZE;
    }
#if 0
 clock_t Start_time = clock();
 Heap_sort(a, MAXSIZE);
 clock_t End_time = clock();
 cout << "Total Heap_sort running time is: " << static_cast<double>(End_time - Start_time) / CLOCKS_PER_SEC * 1000 << " ms" << endl;
#endif

#if 1
    clock_t Start_time = clock();
    insertSort(a, MAXSIZE);
    clock_t End_time = clock();
    cout << "Total insertSort running time is: " << static_cast<double>(End_time - Start_time) / CLOCKS_PER_SEC * 1000 << " ms" << endl;
#endif

#if 0
    clock_t Start_time = clock();
    int temp[MAXSIZE];
    MergeSort(a, 0, MAXSIZE - 1, temp);
    clock_t End_time = clock();
    cout << "Total insertSort running time is: " << static_cast<double>(End_time - Start_time) / CLOCKS_PER_SEC * 1000 << " ms" << endl;
#endif

#if 0
    clock_t Start_time = clock();
    quickSort(a, 0, MAXSIZE - 1);
    clock_t End_time = clock();
    cout << "Total insertSort running time is: " << static_cast<double>(End_time - Start_time) / CLOCKS_PER_SEC * 1000 << " ms" << endl;
#endif
    for (size_t i = 0; i < 100; ++i)
    {
        cout << a[i] << " ";
    }
    cout << endl;
    return 0;
}
```

堆排序时间复杂度测试：(相同时间种子)  
  
| 数组长度 50000   | 插入排序   | 堆排序    |
| ----------------- | ---------- | --------- |
| WSL运行时间       | 1187.5 ms  | 15.625 ms |
| ARM linux运行时间 | 11837.9 ms | 92.139 ms |

当数据长度提高时，显然插入排序已经无法计算，对比堆排序的运行结果：  

| 数组长度 2000000   | 堆排序    |
| ----------------- | --------- |
| WSL运行时间       | 875 ms |
| ARM linux运行时间 | 5894.98 ms |  

## 归并排序

归并排序是一种稳定排序。

## 快速排序

![](/img/sort/quick-sort.gif)

```cpp
#include <iostream>
#include <vector>

using namespace std;

void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

int partition(vector<int>& vec, int low, int high){
    int p = vec[high];
    int idx = low - 1;
    int tmp;

    for (int j = low; j < high; j++){
        if (vec[j] < p){
            idx ++;
            swap(vec[idx], vec[j]);
        }
    }

    swap(vec[idx + 1], vec[high]);

    return idx + 1;
}

/*
 * 快排
 */
void quickSort(vector<int>& vec, int low, int high){
    if(low < high){
        int pi  = partition(vec, low, high);

        quickSort(vec, low, pi - 1);
        quickSort(vec, pi + 1, high);
    }
}

/*
 * 冒泡
 */
void bubbleSort(vector<int>& vec){
    int n = vec.size();
    for (int i = 0; i < n - 1; i ++){
        for (int j = 0; j < n - 1 - i; j++){
            if (vec[j] > vec[j+1]){
                swap(vec[j], vec[j+1]);
            }
        }
    }
}

int main(){
    int arr[] = {12, 3, 5, 6, 8, 2, 9, 2};
    int n = sizeof(arr) / sizeof(arr[0]);
    vector<int> vec(arr, arr + n);

    quickSort(vec, 0, n - 1);

    for(int i = 0; i < n; i ++){
        cout << vec[i] << " ";
    }
    cout << endl;
    return 0;
}
```
