---
layout:     post
title:      哈希表
subtitle:   由分析求两数之和的算法引申
date:       2019-07-07
author:     Aro
header-img: img/post-bg-hash.jpg
catalog: true
tags:
    - 哈希表
    - 数据结构
    - 数组
    - LeetCode
---

# 题目描述

>给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

**例:**

给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9 所以返回 [0, 1]


## 暴力解法

<a href="https://leetcode-cn.com/submissions/detail/25406117/" target="_blank">点击查看提交记录</a>

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
    for(let i in nums){
        for(let j in nums){
            if( i != j && nums[i] + nums[j] == target){
                return [i, j];
            }
        }
    }
};
```

**复杂度分析:**
- 时间复杂度：O(n^2)
- 空间复杂度：O(1)


## 哈希表映射

为了对运行时间复杂度进行优化，我们需要一种更有效的方法来检查数组中是否存在目标元素。如果存在，我们需要找出它的索引。保持数组中的每个元素与其索引相互对应的最好方法是什么？哈希表。

通过以空间换取速度的方式，我们可以将查找时间从 O(n) 降低到 O(1)。哈希表正是为此目的而构建的，它支持以 近似 恒定的时间进行快速查找。我用“近似”来描述，是因为一旦出现冲突，查找用时可能会退化到 O(n)。但只要你仔细地挑选哈希函数，在哈希表中进行查找的用时应当被摊销为 O(1)。


#### 什么是哈希表(Hash Table)？

哈希表（Hash table，也叫散列表），是根据关键码值(Key value)而直接进行访问的数据结构。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。

散列法存储的基本思想：建立记录关键码字与其存储位置的对应关系，或者说，由关键码的值决定数据的存储地址。

这样，不经过比较，一次存取就能得到所查元素的查找方法

优点：查找速度极快（O(1)）,查找效率与元素个数n无关！

**例:**

有数据元素序列(14，23，39，9，25，11)，若规定每个元素k的存储地址H（k）＝k ， H(k)称为散列函数,画出存储结构图。

![hash-example](/img/hash-example.png)

根据散列函数H（k）＝k ，可知元素14应当存入地址为14的单元，元素23应当存入地址为23的单元，...，


#### 如何进行散列查找？

根据存储时用到的散列函数H(k)表达式，迅即可查到结果！例如，查找key=9,则访问H(9)=9号地址，若内容为9则成功；若查不到，应当设法返回一个特殊值，例如空指针或空记录。很显然这种搜索方式空间效率过低。

哈希函数可写成：addr(ai)=H(ki)

选取某个函数，依该函数按关键字计算元素的存储位置并按此存放；查找时也由同一个函数对给定值k计算地址，将k与地址中内容进行比较，确定查找是否成功。哈希方法中使用的转换函数称为哈希函数(杂凑函数).在记录的关键码与记录的存储地址之间建立的一种对应关系。


#### 可能导致的冲突

通常关键码的集合比哈希地址集合大得多，因而经过哈希函数变换后，可能将不同的关键码映射到同一个哈希地址上，这种现象称为冲突。

**例:**

有6个元素的关键码分别为：（14，23，39，9，25，11）。选取关键码与元素位置间的函数为H(k)=k mod 7

![hash-example02](/img/hash-example02.png)

根据哈希函数算出来发现同一个地址放了多个关键码，也就是冲突了。在哈希查找方法中，冲突是不可能避免的，只能尽可能减少。所以，哈希方法必须解决以下两个问题：

1.构造好的哈希函数
    - 所选函数尽可能简单，以便提高转换速度；
    - 所选函数对关键码计算出的地址，应在哈希地址内集中并大致均匀分布，以减少空间浪费。
2.制定一个好的解决冲突的方案
    - 查找时，如果从哈希函数计算出的地址中查不到关键码，则应当依据解决冲突的规则，有规律地查询其它相关单元。

#### 结论

哈希函数只是一种映象，所以哈希函数的设定很灵活，只要使任何关键码的哈希函数值都落在表长允许的范围之内即可
冲突：key1≠key2，但H(key1)=H(key2)
同义词：具有相同函数值的两个关键码
哈希函数冲突不可避免，只能尽量减少。所以，哈希方法解决两个问题：
构造好的哈希函数；

制定解决冲突基本要求：
要求一：n个数据原仅占用n个地址，虽然散列查找是以空间换时间，但仍希望散列的地址空间尽量小。
要求二：无论用什么方法存储，目的都是尽量均匀地存放元素，以避免冲突。


#### 下面来解决具体问题，如何用哈希表的方法解决求两数之和

<a href="https://leetcode-cn.com/submissions/detail/25428067/" target="_blank">点击查看提交记录</a>

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
    let map = new Map();
    let array = new Array();
    for(let i in nums){
        map.set(nums[i], i);
    }
    for(let i in nums){
        let final = target - nums[i];
        if(map.has(final)&&map.get(final)!=i){
            array.push(i, map.get(final))
            return array;
        }
    }
};
```

**复杂度分析:**
- 时间复杂度：O(n)
- 空间复杂度：O(n)


#### 再优化

>在进行迭代并将元素插入到表中的同时，我们还会回过头来检查表中是否已经存在当前元素所对应的目标元素。如果它存在，那我们已经找到了对应解，并立即将其返回。

<a href="https://leetcode-cn.com/submissions/detail/25434406/" target="_blank">点击查看提交记录</a>

```js
var twoSum = function (nums, target) {
    let map = new Map();
    let array = new Array();
    for(let i in nums){
        let final = target - nums[i];
        if(map.has(final)&&map.get(final)!=i){
            array.push(i, map.get(final))
            return array;
        }
        map.set(nums[i], i);
    }
};
```

**复杂度分析:**
- 时间复杂度：O(n)
- 空间复杂度：O(n)


# 参考链接

- <a href="https://www.jianshu.com/p/7495fad83877" target="_blank">https://www.jianshu.com/p/7495fad83877</a>
- <a href="https://leetcode-cn.com/problems/two-sum/" target="_blank">https://leetcode-cn.com/problems/two-sum/</a>