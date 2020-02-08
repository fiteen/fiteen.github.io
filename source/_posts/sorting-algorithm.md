---
title: 动画详解十大经典排序算法（C语言版）
date: 2019-12-27 22:58:49
tag: 
     - [数据结构]
     - [C]
categories: 算法
thumbnail: sort.png
---

排序算法是程序员必备的基础知识，弄明白它们的原理和实现很有必要。本文中将通过非常细节的动画展示出算法的原理，配合代码更容易理解。

<!--more-->

## 概述

由于待排序的元素数量不同，使得排序过程中涉及的存储器不同，可将排序方法分为两类：一类是**内部排序**，指的是待排序列存放在计算机随机存储器中进行的排序过程；另一类是**外部排序**，指的是待排序的元素的数量很大，以致内存一次不能容纳全部记录，在排序过程中尚需对外存进行访问的排序过程。

我们可以将常见的**内部排序算法**可以分成两类：

![](sort-category.png)

**比较类排序**：通过比较来决定元素间的相对次序，时间复杂度为 O(nlogn)～O(n²)。属于比较类的有：

|         排序算法          | 时间复杂度 | 最差情况 | 最好情况 | 空间复杂度 | 排序方式 | 稳定性 |
| :-----------------------: | :--------: | :------: | :------: | :--------: | :----: | :-------: |
|  [冒泡排序](#bubble-sort)   |   O(n²)    |  O(n²)   |   O(n)   |    O(1)​    |  In-place  | ✔ |
|   [快速排序](#quick-sort)   |  O(nlogn)​  |  O(n²)   | O(nlogn)​ |  O(logn)​   | In-place | ✘ |
| [插入排序](#insertion-sort) |   O(n²)    |  O(n²)   |   O(n)​   |    O(1)​    |  In-place  | ✔ |
|   [希尔排序](#shell-sort)   |  O(nlog²n)​  |  O(n²)   |   O(n)​   |    O(1)​    | In-place | ✘ |
| [选择排序](#selection-sort) |   O(n²)    |  O(n²)   |  O(n²)   |    O(1)​    | In-place | ✘ |
|    [堆排序](#heap-sort)     |  O(nlogn)​  | O(nlogn) | O(nlogn)​ |    O(1)​    | In-place | ✘ |
|   [归并排序](#merge-sort)   |  O(nlogn)​  | O(nlogn) | O(nlogn)​ |    O(n)​    |  Out-place  | ✔ |

**非比较类排序**：不通过比较来决定元素间的相对次序，其时间复杂度可以突破 O(nlogn)，以线性时间运行。属于非比较类的有：

|         排序算法         | 时间复杂度 | 最差情况  | 最好情况 | 空间复杂度 | 排序方式 | 稳定性 |
| :----------------------: | :--------: | :-------: | :------: | :--------: | :----: | :-------: |
|   [桶排序](#bucket-sort)   |   O(n+nlog(n/r))​   |   O(n²)   |   O(n)​   |   O(n+r)​   |  Out-place  | ✔ |
| [计数排序](#counting-sort) |   O(n+r)​   |  O(n+r)​   |  O(n+r)​  |   O(n+r)​   |  Out-place  | ✔ |
|  [基数排序](#radix-sort)   |  O(d(n+r))​  | O(d(n+r)) |  O(d(n+r))  |   O(n+r)​   |  Out-place  | ✔ |

**名次解释**：

**[时间/空间复杂度](asymptotic-time-complexity-and-space-complexity)**：描述一个算法执行时间/占用空间与数据规模的增长关系

**n**：待排序列的个数

**r**：“桶”的个数（上面的三种非比较类排序都是基于“桶”的思想实现的）

**d**：待排序列的最高位数

**In-place**：原地算法，指的是占用常用内存，不占用额外内存。空间复杂度为 O(1) 的都可以认为是原地算法

**Out-place**：非原地算法，占用额外内存

**稳定性**：假设待排序列中两元素相等，排序前后这两个相等元素的相对位置不变，则认为是稳定的。


## <span id="bubble-sort">冒泡排序</span>

冒泡排序（Bubble Sort），顾名思义，就是指越小的元素会经由交换慢慢“浮”到数列的顶端。

### 算法原理

1. 从左到右，依次比较相邻的元素大小，更大的元素交换到右边；
2. 从第一组相邻元素比较到最后一组相邻元素，这一步结束最后一个元素必然是参与比较的元素中最大的元素；
3. 按照大的居右原则，重新从左到后比较，前一轮中得到的最后一个元素不参与比较，得出新一轮的最大元素；
4. 按照上述规则，每一轮结束会减少一个元素参与比较，直到没有任何一组元素需要比较。

### 动图演示

![](bubble-sort.gif)

### 代码实现

```c
void bubble_sort(int arr[], int n) {
    int i, j;
    for (i = 0; i < n - 1; i++) {
        for (j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr, j, j+1);
            }
        }
    }
}
```
### 算法分析

冒泡排序属于**交换排序**，是**稳定排序**，平均时间复杂度为 O(n²)，空间复杂度为 O(1)。

但是我们常看到冒泡排序的**最优时间复杂度是 O(n)**，那要如何优化呢？

我们可以用一个 flag 参数记录新一轮的排序中元素是否做过交换，如果没有，说明前面参与比较过的元素已经是正序，那就没必要再从头比较了。代码实现如下：

```c
void bubble_sort_quicker(int arr[], int n) {
    int i, j, flag;
    for (i = 0; i < n - 1; i++) {
        flag = 0;
        for (j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr, j, j+1);
                flag = 1;
            }
        }
        if (!flag) return;
    }
}
```


## <span id="quick-sort">快速排序</span>

快速排序（Quick Sort），是冒泡排序的改进版，之所以“快速”，是因为使用了**分治法**。它也属于**交换排序**，通过元素之间的位置交换来达到排序的目的。

### 基本思想

在序列中随机挑选一个元素作基准，将小于基准的元素放在基准之前，大于基准的元素放在基准之后，再分别对小数区与大数区进行排序。

**一趟快速排序**的具体做法是：

1. 设两个指针 i 和 j，分别指向序列的头部和尾部；
2. 先从 j 所指的位置向前搜索，找到第一个比基准小的值，把它与基准交换位置；
3. 再从 i 所指的位置向后搜索，找到第一个比基准大的值，把它与基准交换位置；
4. 重复 2、3 两步，直到 i = j。

仔细研究一下上述算法我们会发现，在排序过程中，对基准的移动其实是多余的，因为只有一趟排序结束时，也就是 i = j 的位置才是基准的最终位置。

由此可以**优化**一下算法：

1. 设两个指针 i 和 j，分别指向序列的头部和尾部；
2. 先从 j 所指的位置向前搜索，找到第一个比基准小的数值后停下来，再从 i 所指的位置向后搜索，找到第一个比基准大的数值后停下来，把 i 和 j 指向的两个值交换位置；
3. 重复步骤2，直到 i = j，最后将相遇点指向的值与基准交换位置。

### 动图演示

![](quick-sort.gif)

### 代码实现

这里取序列的第一个元素为基准。

```c
/* 选取序列的第一个元素作为基准 */
int select_pivot(int arr[], int low) {
    return arr[low];
}

void quick_sort(int arr[], int low, int high) {
    int i, j, pivot;
    if (low >= high) return;
    pivot = select_pivot(arr, low);
    i = low;
    j = high;
    while (i != j) {
        while (arr[j] >= pivot && i < j) j--;
        while (arr[i] <= pivot && i < j) i++;
        if (i < j) swap(arr, i, j);
    }
    arr[low] = arr[i];
    arr[i] = pivot;
    quick_sort(arr, low, i - 1);
    quick_sort(arr, i + 1, high);
}
```
### 算法分析

快速排序是**不稳定排序**，它的平均时间复杂度为 O(nlogn)，平均空间复杂度为 O(logn)。

快速排序中，基准的选取非常重要，它将影响排序的效率。举个例子，假如序列本身顺序随机，快速排序是所有同数量级时间复杂度的排序算法中平均性能最好的，但如果序列本身已经有序或基本有序，直接**选取固定位置，例如第一个元素**作为基准，会使快速排序就会沦为冒泡排序，时间复杂度为 O(n^2)。为了避免发生这种情况，引入下面两种获取基准的方法：

**随机选取**

就是选取序列中的任意一个数为基准的值。

```c
/* 随机选择基准的位置，区间在 low 和 high 之间 */
int select_pivot_random(int arr[], int low, int high) {
    srand((unsigned)time(NULL));
    int pivot = rand()%(high - low) + low;
    swap(arr, pivot, low);
    
    return arr[low];
}
```

**三者取中**

就是取起始位置、中间位置、末尾位置指向的元素，对这三个元素排序后取中间数作为基准。

```c
/* 取起始位置、中间位置、末尾位置指向的元素三者的中间值作为基准 */
int select_pivot_median_of_three(int arr[], int low, int high) {
    // 计算数组中间的元素的下标
    int mid = low + ((high - low) >> 1);
    // 排序，使 arr[mid] <= arr[low] <= arr[high]
    if (arr[mid] > arr[high]) swap(arr, mid, high);
    if (arr[low] > arr[high]) swap(arr, low, high);
    if (arr[mid] > arr[low]) swap(arr, low, mid);
    // 使用 low 位置的元素作为基准
    return arr[low];
}
```

经验证明，三者取中的规则可以大大改善快速排序在最坏情况下的性能。


## <span id="insertion-sort">插入排序</span>

直接插入排序（Straight Insertion Sort），是一种简单直观的排序算法，它的基本操作是不断地将尚未排好序的数插入到已经排好序的部分，好比打扑克牌时一张张抓牌的动作。在冒泡排序中，经过每一轮的排序处理后，序列后端的数是排好序的；而对于插入排序来说，经过每一轮的排序处理后，序列前端的数都是排好序的。

### 基本思想

先将第一个元素视为一个有序子序列，然后从第二个元素起逐个进行插入，直至整个序列变成元素非递减有序序列为止。如果待插入的元素与有序序列中的某个元素相等，则将待插入元素插入大相等元素的后面。整个排序过程进行 n-1 趟插入。

### 动图演示

![](insertion-sort.gif)

### 代码实现

```c
void insertion_sort(int arr[], int n) {
    int i, j, temp;
    for (i = 1; i < n; i++) {
        temp = arr[i];
        for (j = i; j > 0 && arr[j - 1] > temp; j--)
            arr[j] = arr[j - 1];
        arr[j] = temp;
    }
}
```

### 算法分析

插入排序是**稳定排序**，平均时间复杂度为 O(n²)，空间复杂度为 O(1)。


## <span id="shell-sort">希尔排序</span>

希尔排序（Shell's Sort）是第一个突破 O(n²) 的排序算法，是直接插入排序的改进版，又称“**缩小增量排序**”（Diminishing Increment Sort）。它与直接插入排序不同之处在于，它会优先比较距离较远的元素。

### 基本思想

先将整个待排序列分割成若干个字序列分别进行直接插入排序，待整个序列中的记录“基本有序”时，再对全体记录进行一次直接插入排序。

子序列的构成不是简单地“逐段分割”，将相隔某个增量的记录组成一个子序列，让增量逐趟缩短，直到增量为 1 为止。

### 动图演示

![](shell-sort.gif)

### 代码实现

增量序列可以有各种取法，例如上面动图所示，增量序列满足 [n / 2, n / 2 / 2, ..., 1]，n 是序列本身的长度，这也是一种比较流行的增量序列定义方式。这时希尔排序的算法可以通过下面的代码实现：

```c
void shell_sort_split_half(int arr[], int n) {
    int i, j, dk, temp;
    for (dk = n >> 1; dk > 0; dk = dk >> 1) {
        for (i = dk; i < n; i++) {
            temp = arr[i];
            for (j = i - dk; j >= 0 && arr[j] > temp; j -= dk)
                arr[j + dk] = arr[j];
            arr[j + dk] = temp;
        }
    }
}
```

增量序列也可以有其它的定义方式，那么希尔排序的实现可以归纳成这样：

```c
void shell_insert(int arr[], int n, int dk) {
    int i, j, temp;
    for (i = dk; i < n; i += dk) {
        temp = arr[i];
        j = i - dk;
        while (j >= 0 && temp < arr[j]) {
            arr[j + dk] = arr[j];
            j -= dk;
        }
        arr[j + dk] = temp;
    }
}

void shell_sort(int arr[], int n, int dlta[], int t) {
    int k;
    for (k = 0; k < t; ++k) {
        // 一趟增量为 dlta[k] 的插入排序
        shell_insert(arr, n, dlta[k]);
    }
}
```

### 算法分析

希尔排序是**不稳定排序**，它的分析是一个复杂的问题，因为它的运行时间依赖于增量序列的选择，它的平均时间复杂度为O(n^1.3)，最好情况是 O(n)，最差情况是 O(n²)。空间复杂度为 O(1)。


## <span id="selection-sort">选择排序</span>

选择排序（Selection Sort）是一种简单直观的排序算法。它的基本思想就是，每一趟 n-i+1(i=1,2,...,n-1)个记录中选取关键字最小的记录作为有序序列的第 i 个记录。

### 算法步骤

**简单选择排序**：

1. 在未排序序列中找到最小（大）元素，存放到排序序列的起始位置;
2. 在剩余未排序元素中继续寻找最小（大）元素，放到已排序序列的末尾;
3. 重复步骤2，直到所有元素排序完毕。

### 动图演示

![](selection-sort.gif)

### 代码实现

```c
void selection_sort(int arr[], int n) {
    int i, j;
    for (i = 0; i < n - 1; i++) {
        int min = i;
        for (j = i + 1; j < n; j++) {
            if (arr[j] < arr[min])
                min = j;
        }
        swap(arr, min, i);
    }
}
```

### 算法分析

选择排序是**不稳定排序**，时间复杂度固定为 O(n²)，因此它不适用于数据规模较大的序列。不过它也有优点，就是不占用额外的内存空间。


## <span id="heap-sort">堆排序</span>

堆排序（Heap Sort）是指利用堆这种数据结构所设计的一种排序算法。堆的特点：

- 一颗完全二叉树（也就是会所生成节点的顺序是：从上往下、从左往右）
- 每一个节点必须满足父节点的值不大于/不小于子节点的值

### 基本思想

实现堆排序需要解决两个问题：

- 如何将一个无序序列构建成堆？

- 如何在输出堆顶元素后，调整剩余元素成为一个新的堆？

以升序为例，算法实现的思路为：

1. 建立一个 build_heap 函数，将数组 tree[0,...n-1] 建立成堆，n 表示数组长度。函数里需要维护的是所有节点的父节点，最后一个子节点下标为 n-1，那么它对应的父节点下标就是(n-1-1)/2。
2. 构建完一次堆后，最大元素就会被存放在根节点 tree[0]。将 tree[0] 与最后一个元素交换，每一轮通过这种不断将最大元素后移的方式，来实现排序。
3. 而交换后新的根节点可能不满足堆的特点了，因此需要一个调整函数 heapify 来对剩余的数组元素进行最大堆性质的维护。如果 tree[i] 表示其中的某个节点，那么 tree[2\*i+1] 是左孩子，tree[2\*i+2] 是右孩子，选出三者中的最大元素的下标，存放于 max 值中，若 max 不等于 i，则将最大元素交换到 i 下标的位置。但是，此时以 tree[max] 为根节点的子树可能不满足堆的性质，需要递归调用自身。

### 动图演示

![](heap-sort.gif)

### 代码实现

```c
void heapify(int tree[], int n, int i) {
    // n 表示序列长度，i 表示父节点下标
    if (i >= n) return;
    // 左侧子节点下标
    int left = 2 * i + 1;
    // 右侧子节点下标
    int right = 2 * i + 2;
    int max = i;
    if (left < n && tree[left] > tree[max]) max = left;
    if (right < n && tree[right] > tree[max]) max = right;
    if (max != i) {
        swap(tree, max, i);
        heapify(tree, n, max);
    }
}

void build_heap(int tree[], int n) {
    // 树最后一个节点的下标
    int last_node = n - 1;
    // 最后一个节点对应的父节点下标
    int parent = (last_node - 1) / 2;
    int i;
    for (i = parent; i >= 0; i--) {
        heapify(tree, n, i);
    }
}

void heap_sort(int tree[], int n) {
    build_heap(tree, n);
    int i;
    for (i = n - 1; i >= 0; i--) {
        // 将堆顶元素与最后一个元素交换
        swap(tree, i, 0);
        // 调整成大顶堆
        heapify(tree, i, 0);
    }
}
```

### 算法分析

堆排序是**不稳定排序**，适合数据量较大的序列，它的平均时间复杂度为 Ο(nlogn)，空间复杂度为 O(1)。
此外，堆排序仅需一个记录大小供交换用的辅助存储空间。


## <span id="merge-sort">归并排序</span>

归并排序（Merge Sort）是建立在**归并**操作上的一种排序算法。它和快速排序一样，采用了**分治法**。

### 基本思想

归并的含义是将两个或两个以上的有序表组合成一个新的有序表。也就是说，从几个数据段中逐个选出最小的元素移入新数据段的末尾，使之有序。

那么归并排序的算法我们可以这样理解：

假如初始序列含有 n 个记录，则可以看成是 n 个有序的子序列，每个子序列的长度为 1。然后两两归并，得到 n/2 个长度为2或1的有序子序列；再两两归并，……，如此重复，直到得到一个长度为 n 的有序序列为止，这种排序方法称为 **二路归并排序**，下文介绍的也是这种排序方式。

### 动图演示

![](merge-sort.gif)

### 代码实现

```c
/* 将 arr[L..M] 和 arr[M+1..R] 归并 */
void merge(int arr[], int L, int M, int R) {
    int LEFT_SIZE = M - L + 1;
    int RIGHT_SIZE = R - M;
    int left[LEFT_SIZE];
    int right[RIGHT_SIZE];
    int i, j, k;
    // 以 M 为分割线，把原数组分成左右子数组
    for (i = L; i <= M; i++) left[i - L] = arr[i];
    for (i = M + 1; i <= R; i++) right[i - M - 1] = arr[i];
    // 再合并成一个有序数组（从两个序列中选出最小值依次插入）
    i = 0; j = 0; k = L;
    while (i < LEFT_SIZE && j < RIGHT_SIZE) arr[k++] = left[i] < right[j] ? left[i++] : right[j++];
    while (i < LEFT_SIZE) arr[k++] = left[i++];
    while (j < RIGHT_SIZE) arr[k++] = right[j++];
}

void merge_sort(int arr[], int L, int R) {
    if (L == R) return;
    // 将 arr[L..R] 平分为 arr[L..M] 和 arr[M+1..R]
    int M = (L + R) / 2;
    // 分别递归地将子序列排序为有序数列
    merge_sort(arr, L, M);
    merge_sort(arr, M + 1, R);
    // 将两个排序后的子序列再归并到 arr
    merge(arr, L, M, R);
}
```

### 算法分析

归并排序是**稳定排序**，它和选择排序一样，性能不受输入数据的影响，但表现比选择排序更好，它的时间复杂度始终为 O(nlogn)，但它需要额外的内存空间，空间复杂度为 O(n)。


## <span id="bucket-sort">桶排序</span>

桶排序（Bucket sort）是[计数排序](#counting-sort)的升级版。它利用了函数的映射关系，高效与否的关键就在于这个映射函数的确定。

桶排序的工作的原理：假设输入数据服从均匀分布，将数据分到有限数量的桶里，每个桶再分别排序（也有可能是使用别的排序算法或是以递归方式继续用桶排序进行排序）。

### 算法步骤

1. 设置固定数量的空桶；
2. 把数据放在对应的桶内；
3. 分别对每个非空桶内数据进行排序；
4. 拼接非空的桶内数据，得到最终的结果。

### 动图演示

![](bucket-sort.gif)

### 代码实现

```c
void bucket_sort(int arr[], int n, int r) {
    if (arr == NULL || r < 1) return;

    // 根据最大/最小元素和桶数量，计算出每个桶对应的元素范围
    int max = arr[0], min = arr[0];
    int i, j;
    for (i = 1; i < n; i++) {
        if (max < arr[i]) max = arr[i];
        if (min > arr[i]) min = arr[i];
    }
    int range = (max - min + 1) / r + 1;

    // 建立桶对应的二维数组，一个桶里最多可能出现 n 个元素
    int buckets[r][n];
    memset(buckets, 0, sizeof(buckets));
    int counts[r];
    memset(counts, 0, sizeof(counts));
    for (i = 0; i < n; i++) {
        int k = (arr[i] - min) / range;
        buckets[k][counts[k]++] = arr[i];
    }

    int index = 0;
    for (i = 0; i < r; i++) {
        // 分别对每个非空桶内数据进行排序，比如计数排序
        if (counts[i] == 0) continue;
        counting_sort(buckets[i], counts[i]);
        // 拼接非空的桶内数据，得到最终的结果
        for (j = 0; j < counts[i]; j++) {
            arr[index++] = buckets[i][j];
        }
    }
}
```
### 算法分析

桶排序是**稳定排序**，但仅限于桶排序本身，假如桶内排序采用了快速排序之类的非稳定排序，那么就是不稳定的。

#### 时间复杂度

桶排序的时间复杂度可以这样看：
- n 次循环，每个数据装入桶
- r 次循环，每个桶中的数据进行排序（每个桶中平均有 n/r 个数据）

假如桶内排序用的是选择排序这类时间复杂度较高的排序，整个桶排序的时间复杂度就是 O(n)+O(n²)，视作 O(n²)，这是最差的情况；
假如桶内排序用的是比较先进的排序算法，时间复杂度为 O(nlogn)，那么整个桶排序的时间复杂度为 O(n)+O(r\*(n/r)\*log(n/r))=O(n+nlog(n/r))。k=nlog(n/r)，桶排序的平均时间复杂度为 O(n+k)。当 r 接近于 n 时，k 趋近于 0，这时桶排序的时间复杂度是最优的，就可以认为是 O(n)。也就是说如果数据被分配到同一个桶中，排序效率最低；但如果数据可以均匀分配到每一个桶中，时间效率最高，可以线性时间运行。但同样地，桶越多，空间就越大。

#### 空间复杂度

占用额外内存，需要创建 r 个桶的额外空间，以及 n 个元素的额外空间，所以桶排序的空间复杂度为 O(n+r)。


## <span id="counting-sort">计数排序</span>

计数排序（Counting Sort）是一种**非比较性质**的排序算法，利用了**桶**的思想。它的核心在于将**输入的数据值转化为键存储在额外开辟的辅助空间中**，也就是说这个辅助空间的长度取决于待排序列中的数据范围。

如何转化成桶思想来理解呢？我们设立 r 个桶，桶的键值分别对应从序列最小值升序到最大值的所有数值。接着，按照键值，依次把元素放进对应的桶中，然后统计出每个桶中分别有多少元素，再通过对桶内数据的计算，即可确定每一个元素最终的位置。

### 算法步骤

1. 找出待排序列中最大值 max 和最小值 min，算出序列的数据范围 r = max - min + 1，申请辅助空间 C[r]；
2. 遍历待排序列，统计序列中每个值为 i 的元素出现的次数，记录在辅助空间的第 i 位；
3. 对辅助空间内的数据进行计算（从空间中的第一个元素开始，每一项和前一项相加），以确定值为 i 的元素在数组中出现的位置；
4. 反向填充目标数组：将每个元素 i 放在目标数组的第 C[i] 位，每放一个元素就将 C[i] 减1，直到 C 中所有值都是 0

### 动图演示

![](counting-sort.gif)

### 代码实现

```c
void counting_sort(int arr[], int n) {
    if (arr == NULL) return;
    // 定义辅助空间并初始化
    int max = arr[0], min = arr[0];
    int i;
    for (i = 1; i < n; i++) {
        if (max < arr[i]) max = arr[i];
        if (min > arr[i]) min = arr[i];
    }
    int r = max - min + 1;
    int C[r];
    memset(C, 0, sizeof(C));
    // 定义目标数组
    int R[n];
    // 统计每个元素出现的次数
    for (i = 0; i < n; i++) C[arr[i] - min]++;
    // 对辅助空间内数据进行计算
    for (i = 1; i < r; i++) C[i] += C[i - 1];
    // 反向填充目标数组
    for (i = n - 1; i >= 0; i--) R[--C[arr[i] - min]] = arr[i];
    // 目标数组里的结果重新赋值给 arr
    for (i = 0; i < n; i++) arr[i] = R[i];
}
```
### 算法分析

计数排序属于**非交换排序**，是**稳定排序**，适合数据范围不显著大于数据数量的序列。

#### 时间复杂度

它的时间复杂度是线性的，为 O(n+r)，r 表示待排序列中的数据范围，也就是桶的个数。可以这样理解：将 n 个数据依次放进对应的桶中，再从 r 个桶中把数据按顺序取出来。

#### 空间复杂度

占用额外内存，还需要 r 个桶，因此空间复杂度是 O(n+r)，计数排序快于任何比较排序算法，但这是通过牺牲空间换取时间来实现的。


## <span id="radix-sort">基数排序</span>

基数排序（Radix Sort）是**非比较型**排序算法，它和[计数排序](#counting-sort)、[桶排序](#bucket-sort)一样，利用了“**桶**”的概念。基数排序不需要进行记录关键字间的比较，是一种**借助多关键字排序的思想对单逻辑关键字进行排序**的方法。比如数字100，它的个位、十位、百位就是不同的关键字。

那么，对于一组乱序的数字，基数排序的实现原理就是将整数按位数（关键字）切割成不同的数字，然后按每个位数分别比较。对于关键字的选择，有最高位优先法（MSD法）和最低位优先法（LSD法）两种方式。MSD 必须将序列先逐层分割成若干子序列，然后再对各子序列进行排序；而 LSD 进行排序时，不必分成子序列，对每个关键字都是整个序列参加排序。

### 算法步骤

以 LSD 法为例：
1. 将所有待比较数值（非负整数）统一为同样的数位长度，数位不足的数值前面补零
2. 从最低位（个位）开始，依次进行一次排序
3. 从最低位排序一直到最高位排序完成以后, 数列就变成一个有序序列

如果要支持负数参加排序，可以将序列中所有的值加上一个常数，使这些值都成为非负数，排好序后，所有的值再减去这个常数。

### 动图演示

![](radix-sort.gif)

### 代码实现

```c
// 基数，范围0~9
#define RADIX 10

void radix_sort(int arr[], int n) {
    // 获取最大值和最小值
    int max = arr[0], min = arr[0];
    int i, j, l;
    for (i = 1; i < n; i++) {
        if (max < arr[i]) max = arr[i];
        if (min > arr[i]) min = arr[i];
    }
    // 假如序列中有负数，所有数加上一个常数，使序列中所有值变成正数
    if (min < 0) {
        for (i = 0; i < n; i++) arr[i] -= min;
        max -= min;
    }
    // 获取最大值位数
    int d = 0;
    while (max > 0) {
        max /= RADIX;
        d ++;
    }
    int queue[RADIX][n];
    memset(queue, 0, sizeof(queue));
    int count[RADIX] = {0};
    for (i = 0; i < d; i++) {
        // 分配数据
        for (j = 0; j < n; j++) {
            int key = arr[j] % (int)pow(RADIX, i + 1) / (int)pow(RADIX, i);
            queue[key][count[key]++] = arr[j];
        }
        // 收集数据
        int c = 0;
        for (j = 0; j < RADIX; j++) {
            for (l = 0; l < count[j]; l++) {
                arr[c++] = queue[j][l];
                queue[j][l] = 0;
            }
            count[j] = 0;
        }
    }
    // 假如序列中有负数，收集排序结果时再减去前面加上的常数
    if (min < 0) {
        for (i = 0; i < n; i++) arr[i] += min;
    }
}
```

### 算法分析

基数排序是**稳定排序**，适用于关键字取值范围固定的排序。

#### 时间复杂度

基数排序可以看作是若干次“分配”和“收集”的过程。假设给定 n 个数，它的最高位数是 d，基数（也就是桶的个数）为 r，那么可以这样理解：共进行 d 趟排序，每趟排序都要对 n 个数据进行分配，再从 r 个桶中收集回来。所以算法的时间复杂度为 O(d(n+r))，在整数的排序中，r = 10，因此可以简化成 O(dn)，是**线性阶**的排序。

#### 空间复杂度

占用额外内存，需要创建 r 个桶的额外空间，以及 n 个元素的额外空间，所以基数排序的空间复杂度为 O(n+r)。

### 计数排序 & 桶排序 & 基数排序

这三种排序算法都利用了桶的概念，但对桶的使用方法上有明显差异：

- 桶排序：每个桶存储一定范围的数值，适用于元素尽可能分布均匀的排序；
- 计数排序：每个桶只存储单一键值，适用于最大值和最小值尽可能接近的排序；
- 基数排序：根据键值的每位数字来分配桶，适用于非负整数间的排序，且最大值和最小值尽可能接近。

---

本文关联[项目地址](https://github.com/fiteen/Sorting-Algorithm)