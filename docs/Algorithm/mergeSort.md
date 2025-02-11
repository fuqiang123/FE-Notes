#### 分治思想

> “分治”，分而治之。其思想就是将一个大问题分解为若干个子问题，针对子问题分别求解后，再将子问题的解整合为大问题的解

利用分治思想解决问题，我们一般分三步走：

- 分解子问题
- 求解每个子问题
- 合并子问题的解，得出大问题的解

#### 归并排序

> 把一系列排好序的子序列合并成一个大的完整有序序 列。从理论上讲，这个算法很容易实现。我们需要两个排好序的子数组，然后通过比较数 据大小，先从最小的数据开始插入，最后合并得到第三个数组

**思路分析**

归并排序是对分治思想的典型应用，它按照如下的思路对分治思想“三步走”的框架进行了填充：

- **分解子问题**：将需要被排序的数组从中间分割为两半，然后再将分割出来的每个子数组各分割为两半，重复以上操作，直到单个子数组只有一个元素为止。
- **求解每个子问题**：从粒度最小的子数组开始，两两合并、确保每次合并出来的数组都是有序的。（这里的“子问题”指的就是对每个子数组进行排序）。
- **合并子问题的解，得出大问题的解**：当数组被合并至原有的规模时，就得到了一个完全排序的数组

**编码实现**

通过上面的讲解，我们可以总结出归并排序中的两个主要动作：

- 分割
- 合并

这两个动作是紧密关联的，分割是将大数组反复分解为一个一个的原子项，合并是将原子项反复地组装回原有的大数组。整个过程符合两个特征：

1. 重复（令人想到递归或迭代）
2. 有去有回（令人想到回溯，进而明确递归这条路）

因此，归并排序在实现上依托的就是递归思想。
除此之外，这里还涉及到另一个小小的知识点——**两个有序数组的合并**。

```js
function mergeSort(arr) {
    const len = arr.length
    // 处理边界情况
    if(len <= 1) {
        return arr
    }   
    // 计算分割点
    const mid = Math.floor(len / 2)    
    // 递归分割左子数组，然后合并为有序数组
    const leftArr = mergeSort(arr.slice(0, mid)) 
    // 递归分割右子数组，然后合并为有序数组
    const rightArr = mergeSort(arr.slice(mid,len))  
    // 合并左右两个有序数组
    arr = mergeArr(leftArr, rightArr)  
    // 返回合并后的结果
    return arr
}
  
function mergeArr(arr1, arr2) {  
    // 初始化两个指针，分别指向 arr1 和 arr2
    let i = 0, j = 0   
    // 初始化结果数组
    const res = []    
    // 缓存arr1的长度
    const len1 = arr1.length  
    // 缓存arr2的长度
    const len2 = arr2.length  
    // 合并两个子数组
    while(i < len1 && j < len2) {
        if(arr1[i] < arr2[j]) {
            res.push(arr1[i])
            i++
        } else {
            res.push(arr2[j])
            j++
        }
    }
    // 若其中一个子数组首先被合并完全，则直接拼接另一个子数组的剩余部分
    if(i<len1) {
        return res.concat(arr1.slice(i))
    } else {
        return res.concat(arr2.slice(j))
    }
}
```

**复杂度分析**

> 归并的过程 每次进行拆分，当数组的个数为`n`时，需要拆分为`logn`（也就是递归树的层数）次。而每层的节点之间的合并需要遍历`n`个节点，所以最终的时间复杂度为`nlogn`

