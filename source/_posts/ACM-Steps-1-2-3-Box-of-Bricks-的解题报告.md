---
title: ACM Steps 1.2.3 Box of Bricks 的解题报告
date: 2016-11-30 00:24:17
tags:
---

## 描述信息
Problem Description
Little Bob likes playing with his box of bricks. He puts the bricks one upon another and builds stacks of different height. “Look, I\'ve built a wall!”, he tells his older sister Alice. “Nah, you should make all stacks the same height. Then you would have a real wall.”, she retorts. After a little consideration, Bob sees that she is right. So he sets out to rearrange the bricks, one by one, such that all stacks are the same height afterwards. But since Bob is lazy he wants to do this with the minimum number of bricks moved. Can you help?

 

Input
The input consists of several data sets. Each set begins with a line containing the number n of stacks Bob has built. The next line contains n numbers, the heights hi of the n stacks. You may assume 1≤n≤50 and 1≤hi≤100.

The total number of bricks will be divisible by the number of stacks. Thus, it is always possible to rearrange the bricks such that all stacks have the same height.

The input is terminated by a set starting with n = 0. This set should not be processed.

Output
For each set, print the minimum number of bricks that have to be moved in order to make all the stacks the same height.
Output a blank line between each set.

Sample Input
6
5 2 4 1 7 5
0

Sample Output
Set #1
The minimum number of moves is 5.

Author
qianneng

Source
冬练三九之二

Recommend
lcy
 


## 解决方案

### 解题思路
假设n个stack，每个stack的高度为(h0, h1, ... hn)，那么求解问题的算法步骤如下：
1. 每个stack的高度相同，那么stack的高度就是havg = (h0 + h1 + ... + hn) / n；
2. 依次计算每个stack上需要搬出的brick个数，然后计算所有的stack搬出的brick的总和，即为题目所求；
3. 计算第i个stack上需要搬出的brick的个数，如果hi <= havg，则不需要搬出brick，如果hi > havg，则需要搬出brick的个数为hi - havg。


### 代码
```
#include <stdio.h>

#define N 51

int stack[N];

int main(int argc, char *argv[])
{
    int i = 0, n = 0;
    int avg = 0;
    int sum = 0;
    int moved = 0;
    int set = 0;  

    while((scanf("%d", &n) != EOF) && (n != 0)) {
        sum = 0;
        moved = 0;
        for (i = 0; i < n; i++) {
            scanf("%d", &stack[i]);
            sum += stack[i];
        }

        avg = sum / n;
        for (i = 0; i < n; i++) {
            if (stack[i] > avg) {
                moved += stack[i] - avg;
            }
        }
        set++;
        printf("Set #%d\nThe minimum number of moves is %d.\n\n", set, moved);
    }
    return 0;
}
```
