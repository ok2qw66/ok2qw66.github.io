---
title:  "두개 뽑아서 더하기 from programmers "
date:   2020-12-31 23:50:50 +0900
categories: 
  - 알고리즘
tags:
  - 프로그래머스
toc: true
toc_sticky: true
---

<br>

## 문제 출처

프로그래머스 월간 코드 챌린지 시즌 1 > 두 개 뽑아서 더하기
[https://programmers.co.kr/learn/courses/30/lessons/68644](https://programmers.co.kr/learn/courses/30/lessons/68644)
<br>

## 문제

###### 문제 설명

정수 배열 numbers가 주어집니다. numbers에서 서로 다른 인덱스에 있는 두 개의 수를 뽑아 더해서 만들 수 있는 모든 수를 배열에 오름차순으로 담아 return 하도록 solution 함수를 완성해주세요.

------

##### 제한사항

- numbers의 길이는 2 이상 100 이하입니다.
  - numbers의 모든 수는 0 이상 100 이하입니다.

------

##### 입출력 예

| numbers       | result          |
| ------------- | --------------- |
| `[2,1,3,4,1]` | `[2,3,4,5,6,7]` |
| `[5,0,2,7]`   | `[2,5,7,9,12]`  |

------

##### 입출력 예 설명

입출력 예 #1

- 2 = 1 + 1 입니다. (1이 numbers에 두 개 있습니다.)
- 3 = 2 + 1 입니다.
- 4 = 1 + 3 입니다.
- 5 = 1 + 4 = 2 + 3 입니다.
- 6 = 2 + 4 입니다.
- 7 = 3 + 4 입니다.
- 따라서 `[2,3,4,5,6,7]` 을 return 해야 합니다.

입출력 예 #2

- 2 = 0 + 2 입니다.
- 5 = 5 + 0 입니다.
- 7 = 0 + 7 = 5 + 2 입니다.
- 9 = 2 + 7 입니다.
- 12 = 5 + 7 입니다.
- 따라서 `[2,5,7,9,12]` 를 return 해야 합니다.

---

<br>

오늘은 2020년 마지막날이다!!<br>

마지막 날이라 가벼운 문제를 풀었다 내일부턴 열심히 해야지..ㅎㅎㅎ<br>

2021년은 다들 좋은 일만 가득하고 코로나가 얼른 끝나서 다들 행복했으면 좋겠다!!

<br>

<br>

---

<br>

## 작성 코드

```python
def solution(numbers):
    answer = []
    
    for x in range(len(numbers)-1):
        for y in range(x+1,len(numbers)):
            answer.append(numbers[x]+numbers[y])  

    return sorted(list(set(answer)))
```
