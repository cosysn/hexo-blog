---
title: ACM Steps 1.2.4 Nasty Hacks 的解题报告
date: 2016-11-30 22:21:26
tags:
---

## 描述信息
Problem Description
You are the CEO of Nasty Hacks Inc., a company that creates small pieces of malicious software which teenagers may use
to fool their friends. The company has just finished their first product and it is time to sell it. You want to make as much money as possible and consider advertising in order to increase sales. You get an analyst to predict the expected revenue, both with and without advertising. You now want to make a decision as to whether you should advertise or not, given the expected revenues.

Input
The input consists of n cases, and the first line consists of one positive integer giving n. The next n lines each contain 3 integers, r, e and c. The first, r, is the expected revenue if you do not advertise, the second, e, is the expected revenue if you do advertise, and the third, c, is the cost of advertising. You can assume that the input will follow these restrictions: -106 ≤ r, e ≤ 106 and 0 ≤ c ≤ 106.

Output
Output one line for each test case: “advertise”, “do not advertise” or “does not matter”, presenting whether it is most profitable to advertise or not, or whether it does not make any difference.

Sample Input
3
0 100 70
100 130 30
-100 -70 40

Sample Output
advertise
does not matter
do not advertise

Source
Nordic 2006

Recommend
zty

## 解决方案

### 解题思路
计算打广告和不打广告的收益差x = e-c-r，并做如下判断：
1. x > 0， 需要打广告
2. x = 0,  打不打广告都可以
3. x < 0, 不需要打广告

### 代码
```
#include <stdio.h>

int main(int argc, char *argv[])
{
    int i;
    int n, r, e, c;
    int revenues;

    scanf("%d", &n);
    for (i = 0; i < n; i++) {
        scanf("%d %d %d", &r, &e, &c);
        revenues = e-c-r;

        if (revenues > 0) {
            printf("advertise\n");
        } else if (revenues == 0) {
            printf("does not matter\n");
        } else {
            printf("do not advertise\n");
        }
    }
}
```
