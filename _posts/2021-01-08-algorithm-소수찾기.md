---
title:  "소수찾기 from 프로그래머스"
date:   2021-01-08 02:40:50 +0900
categories: 
  - 알고리즘
tags:
  - 프로그래머스
toc: true
toc_sticky: true
---

<br>

## 문제 출처

프로그래머스 > 완전탐색 > 소수찾기
[https://programmers.co.kr/learn/courses/30/lessons/42839](https://programmers.co.kr/learn/courses/30/lessons/42839)
<br>

## 문제

- ###### 문제 설명

  한자리 숫자가 적힌 종이 조각이 흩어져있습니다. 흩어진 종이 조각을 붙여 소수를 몇 개 만들 수 있는지 알아내려 합니다.

  각 종이 조각에 적힌 숫자가 적힌 문자열 numbers가 주어졌을 때, 종이 조각으로 만들 수 있는 소수가 몇 개인지 return 하도록 solution 함수를 완성해주세요.

  ##### 제한사항

  - numbers는 길이 1 이상 7 이하인 문자열입니다.
  - numbers는 0~9까지 숫자만으로 이루어져 있습니다.
  - 013은 0, 1, 3 숫자가 적힌 종이 조각이 흩어져있다는 의미입니다.

  ##### 입출력 예

  | numbers | return |
  | ------- | ------ |
  | 17      | 3      |
  | 011     | 2      |

  ##### 입출력 예 설명

  예제 #1
  [1, 7]으로는 소수 [7, 17, 71]를 만들 수 있습니다.

  예제 #2
  [0, 1, 1]으로는 소수 [11, 101]를 만들 수 있습니다.

  - 11과 011은 같은 숫자로 취급합니다.

<br>

---

### 완성 코드

```python
def solution(numbers):
    answer = []
    real_answer = set()

    def findNum(numbers, num, start):
        if start == len(numbers):
            if len(num) != 0 and num !=['1'] and num !=['0']:
                answer.append(num)
        else:
            findNum(numbers, num, start + 1)
            num_copy = []
            for x in num:
                num_copy.append(numbers[start] + x)
                for l in range(1, len(x)):
                    num_copy.append(x[:l] + numbers[start] + x[l:])
                num_copy.append(x + numbers[start])
            else:
                num_copy.append(numbers[start])
            findNum(numbers, num_copy, start + 1)

            
    findNum(numbers, [], 0)

    
    for x in answer:
        for num in x:
            test = 3
            
            if int(num) ==2:
                real_answer.add(int(num))
                continue
            elif int(num)<2 or int(num)%2 == 0:
                continue
            else: 
                while test < int(num):
                    if int(num) % test == 0:
                        break
                    else:
                        test += 2
                if test == int(num):
                    real_answer.add(int(num))


    return len(real_answer)
```
