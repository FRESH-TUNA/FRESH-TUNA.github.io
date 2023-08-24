---
layout: post
title: "두 배열의 합"
tags: []
comments: true
---

## 1. 개요
- [https://www.acmicpc.net/problem/2342](https://www.acmicpc.net/problem/2342)

## 2. DP 풀이
```python
import sys

def solution():
    READ = lambda: sys.stdin.readline().rstrip()
    READINTLIST = lambda: list(map(int, READ().split()))

    CMDS = list(READINTLIST())
    N, C, L, R, D = len(CMDS), 0, 1, 2, 5
    MAX_D = 4*N 
    
    DB = [[[MAX_D]*D for _ in range(D)] for _ in range(N)]

    DB[0][0][0] = 0

    def cost(d, nextD):
        if d==0:
            return 2
        elif d==nextD:
            return 1
        elif abs(nextD-d)==1 or abs(nextD-d)==3:
            return 3
        else:
            return 4

    def driver():
        for i in range(N-1):
            last, cur = i, i+1
            nextD = CMDS[i]
        
            # left move case
            for l in range(D):
                for r in range(D):
                    times = DB[last][l][r]+cost(l, nextD)
                    DB[cur][nextD][r] = min(DB[cur][nextD][r], times)

            # right move case
            for l in range(D):
                for r in range(D):
                    times = DB[last][l][r]+cost(r, nextD)
                    DB[cur][l][nextD] = min(DB[cur][l][nextD], times)

        answer = MAX_D
        
        for l in range(D):
            for r in range(D):
                answer = min(answer, DB[-1][l][r])
                
        return answer

    return driver()

print(solution())
```
