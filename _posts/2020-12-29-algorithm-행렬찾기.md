---
title:  "행렬찾기 from SW Expert Academy "
date:   2020-12-29 13:59:50 +0900
categories: 
  - 알고리즘
tags:
  - swea
toc: true
toc_sticky: true
---

<br>

## 문제 출처

[강의형] SW 문제해결 응용 - 06 DP-1 
4차시 6일차 - 행렬찾기
[https://swexpertacademy.com/main/learn/course/subjectList.do?courseId=AVuPDYSqAAbw5UW6](https://swexpertacademy.com/main/learn/course/subjectList.do?courseId=AVuPDYSqAAbw5UW6)
<br>

## 문제

해당 링크 참고!

---

<br>

어제 백준에서 17135. 캐슬 디펜스 풀려다 못 풀어서 업로드를 하지 못했다ㅎㅎㅎ
우선 좀 더 쉬운 문제 하나 풀고 다시 풀어야겠다 생각해서 이 문제를 먼저 풀었다

<br>

## 작성 코드

```python
T = int(input())

for test_case in range(1, T + 1):
    n = int(input())
    board = [list(map(int, input().split())) for _ in range(n)]
    answer = []

    for i in range(n):
        for j in range(n):
            for x in answer:
                if x[0][0] <= i <= x[1][0] and x[0][1] <= j <= x[1][1]:
                    break
            else:
                if board[i][j]:
                    start = (i, j)
                    for ii in range(i + 1, n + 1):
                        if ii >= n:
                            end_i = n - 1
                        elif not board[ii][j]:
                            end_i = ii - 1
                            break
                    for jj in range(j + 1, n + 1):
                        if jj >= n:
                            end_j = n - 1
                        elif not board[i][jj]:
                            end_j = jj - 1
                            break
                    end = (end_i, end_j)
                    answer.append([start, end])

    # print(answer)
    answer_print = []
    for x in answer:
        row = x[1][0] - x[0][0] + 1
        col = x[1][1] - x[0][1] + 1
        answer_print.append([row,col,row*col])

    answer_print = sorted(answer_print, key=lambda x: (x[2],x[0]))
    # print(answer_print)
    print(f"#{test_case} {len(answer_print)}",end=" ")
    for x in answer_print:
        print(str(x[0])+' '+str(x[1]),end=' ')
    print()

```
