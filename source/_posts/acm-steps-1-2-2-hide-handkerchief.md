---
title: ACM Steps 1.2.2 hide handkerchief 的解题报告
date: 2016-11-28 08:24:12
tags:
---
<!-- more -->
## 描述信息
Problem Description
The Children’s Day has passed for some days .Has you remembered something happened at your childhood? I remembered I often played a game called hide handkerchief with my friends.
Now I introduce the game to you. Suppose there are N people played the game ,who sit on the ground forming a circle ,everyone owns a box behind them .Also there is a beautiful handkerchief hid in a box which is one of the boxes .
Then Haha(a friend of mine) is called to find the handkerchief. But he has a strange habit. Each time he will search the next box which is separated by M-1 boxes from the current box. For example, there are three boxes named A,B,C, and now Haha is at place of A. now he decide the M if equal to 2, so he will search A first, then he will search the C box, for C is separated by 2-1 = 1 box B from the current box A . Then he will search the box B ,then he will search the box A.
So after three times he establishes that he can find the beautiful handkerchief. Now I will give you N and M, can you tell me that Haha is able to find the handkerchief or not. If he can, you should tell me "YES", else tell me "POOR Haha".



Input
There will be several test cases; each case input contains two integers N and M, which satisfy the relationship: 1<=M<=100000000 and 3<=N<=100000000. When N=-1 and M=-1 means the end of input case, and you should not process the data.
 

Output
For each input case, you should only the result that Haha can find the handkerchief or not.

Sample Input
3 2
-1 -1
 

Sample Output
YES
 

Source
HDU 2007-6 Programming Contest
 


## 解决方案

### 解题思路
这道题可能理解有点困难，其实就是N个盒子摆成一圈，Haha每次跳过M-1个盒子，去检查第M个盒子，看是否能将所有的盒子都找一遍。如果能够找到的话，就输出YES，否则的话，就输出POOR Haha。
这道题其实考察的是N和M是否为互质的问题，如果互质的话，因为可以无限循环，那么肯定可以将所有的盒子都找一遍。如果两个数不互质的话，总有几个盒子会遍历不到。
使用辗转相除法求N和M的最大公约数，如果最大公约数为1的话，说明两个数互质。

### 代码
```
#include <stdio.h>

int gcd(int a, int b)
{
    int r;

    if (a < b) { /*求a和b的最大公约数时，保证a>b*/
        r = a;
        a = b;
        b = r;
    }

    /*辗转相除*/
    while (b != 0) {
        r = a % b;
        a = b;
        b = r;
    }

    return a;
}


int main(int argc, char *argv[])
{
    int n, m;

    while ((scanf("%d %d", &n, &m) != EOF) && (m != -1 && n != -1)) {
        if (gcd(n, m) == 1) {
            printf("YES\n");
        } else {
            printf("POOR Haha\n");
        }
    }
}
```