---
title: ACM Steps 1.2.5 find your present (2) 的解题报告
date: 2016-11-30 23:34:22
tags:
---

## 描述信息
Problem Description
In the new year party, everybody will get a "special present".Now it's your turn to get your special present, a lot of presents now putting on the desk, and only one of them will be yours.Each present has a card number on it, and your present's card number will be the one that different from all the others, and you can assume that only one number appear odd times.For example, there are 5 present, and their card numbers are 1, 2, 3, 2, 1.so your present will be the one with the card number of 3, because 3 is the number that different from all the others.

Input
The input file will consist of several cases. 
Each case will be presented by an integer n (1<=n<1000000, and n is odd) at first. Following that, n positive integers will be given in a line, all integers will smaller than 2^31. These numbers indicate the card numbers of the presents.n = 0 ends the input.

Output
For each case, output an integer in a line, which is the card number of your present. 

Sample Input
5
1 1 3 2 2
3
1 2 1
0

Sample Output
3
2

Hint
Hint
 
use scanf to avoid Time Limit Exceeded

Author
8600

Source
HDU 2007-Spring Programming Contest - Warm Up （1）

Recommend
8600


## 解决方案

### 解题思路
从题目可知，只有一个数出现的次数是奇数次，其他的数出现的次数是偶数，而我们就要找出那个出现了奇数次的数。我们可以利用位异或来求解：
1. a ^ a = 0;
2. 0 ^ a = a;
3. a ^ b = b ^ a;

可以得知：
如果一个数进行奇数次异或之后其值为其本身，一个数进行偶数次异或之后其值为0。

### 代码
```
#include <stdio.h>

int main(int argc, char *argv[])
{
    int i, n;
    int num;
    int res;

    while((scanf("%d", &n) != EOF) && (n != 0)) {
        res = 0;
        for (i = 0; i < n; i++) {
            scanf("%d", &num);
            res ^= num;
        }

        printf("%d\n", res);
    }

    return 0;
}
```