---
title:  "평균은 넘겠지 from 백준"
date:   2021-01-10 23:40:50 +0900
categories: 
  - 알고리즘
tags:
  - 백준
toc: true
toc_sticky: true
---

<br>

## 문제 출처

백준 > 1차원 배열 > 평균은 넘겠지
[https://www.acmicpc.net/problem/4344](https://www.acmicpc.net/problem/4344)
<br>

## 문제

대학생 새내기들의 90%는 자신이 반에서 평균은 넘는다고 생각한다. 당신은 그들에게 슬픈 진실을 알려줘야 한다.

## 입력

첫째 줄에는 테스트 케이스의 개수 C가 주어진다.

둘째 줄부터 각 테스트 케이스마다 학생의 수 N(1 ≤ N ≤ 1000, N은 정수)이 첫 수로 주어지고, 이어서 N명의 점수가 주어진다. 점수는 0보다 크거나 같고, 100보다 작거나 같은 정수이다.

## 출력

각 케이스마다 한 줄씩 평균을 넘는 학생들의 비율을 반올림하여 소수점 셋째 자리까지 출력한다.

<br>

---

### 완성 코드

```python
c = int(input())

for _ in range(c):
    array = list(map(int,input().split()))
    num = array[0]
    score = array[1:]
    avg = sum(score)/num
    count = 0
    for x in score:
        if x > avg:
            count += 1
    print(format(count/num * 100,".3f")+'%')
```
