---
title: 剑指Offer_4Extra_合并两个有序数组
date: 2017-03-14 21:51:15
tags: [剑指Offer]
---

**题目描述**

>有两个排序的数组A1和A2，内存在A1的末尾有足够多的空余空间容纳A2。请事先一个函数，把A2中的所有数字插入到A1zhong并且所有的数字是排序的。


**解题思路**

和前面的例题一样，很多人首先想到的办法是在A1中从头到尾复制数字，但是这样会出现多次复制一个数字的情况。
更好的办法是从尾到头比较A1和A2中的数字，并把较大的数字复制到A1的合适位置。
注意边界条件的控制，如果A1已经移动结束，但是A2还没有结束，需要将A2移动到A1的其实位置而A1的指针不动

<!--more-->

**代码实现**

```java
public class MergeSortedArray
{

    public static int[] merge(int[] a, int[] b)
    {
        if (a == null)
            return b;
        if (b == null)
            return a;

        int p1 = a.length - 1 - b.length;
        int p2 = b.length - 1;
        while (p2 >= 0)
        {
            if (p1 >= 0 && a[p1] > b[p2])
            {
                a[p1 + p2 + 1] = a[p1];
                p1--;
            } else
            {
                a[p1 + p2 + 1] = b[p2];
                p2--;
            }
        }

        return a;
    }

    public static void main(String[] args)
    {
        //int[] a = {1, 5, 7, 9, 15, 0, 0, 0};
        //int[] a = {9, 15, 0, 0, 0};
        //int[] a = {1,9, 15, 0, 0, 0, 0,0};
        int[] a = {1,3,5,7,8,9,11,12,13,14,15,0,0,0,0,0};

        int[] b = {2, 4, 6, 10,16};
        merge(a, b);
        for (int i : a)
        {
            System.out.print(i + " ");
        }
        System.out.println("");
    }
}

```