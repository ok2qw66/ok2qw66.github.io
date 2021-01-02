---
title:  " 2 x n 타일링 from programmers "
date:   2021-01-02 22:50:50 +0900
categories: 
  - 알고리즘
tags:
  - 프로그래머스
toc: true
toc_sticky: true
---

<br>

## 문제 출처

프로그래머스 연습문제 > 2 x n 타일링
[https://programmers.co.kr/learn/courses/30/lessons/12900](https://programmers.co.kr/learn/courses/30/lessons/12900)
<br>

## 문제

- ###### 문제 설명

  가로 길이가 2이고 세로의 길이가 1인 직사각형모양의 타일이 있습니다. 이 직사각형 타일을 이용하여 세로의 길이가 2이고 가로의 길이가 n인 바닥을 가득 채우려고 합니다. 타일을 채울 때는 다음과 같이 2가지 방법이 있습니다.

  - 타일을 가로로 배치 하는 경우
  - 타일을 세로로 배치 하는 경우

  예를들어서 n이 7인 직사각형은 다음과 같이 채울 수 있습니다.

  ![Imgur](https://i.imgur.com/29ANX0f.png)

  직사각형의 가로의 길이 n이 매개변수로 주어질 때, 이 직사각형을 채우는 방법의 수를 return 하는 solution 함수를 완성해주세요.

  ##### 제한사항

  - 가로의 길이 n은 60,000이하의 자연수 입니다.
  - 경우의 수가 많아 질 수 있으므로, 경우의 수를 1,000,000,007으로 나눈 나머지를 return해주세요.

  ------

  ##### 입출력 예

  | n    | result |
  | ---- | ------ |
  | 4    | 5      |

  ##### 입출력 예 설명

  입출력 예 #1
  다음과 같이 5가지 방법이 있다.

  ![Imgur](https://i.imgur.com/keiKrD3.png)

  ![Imgur](https://i.imgur.com/O9GdTE0.png)

  ![Imgur](https://i.imgur.com/IZBmc6M.png)

  ![Imgur](https://i.imgur.com/29LWVzK.png)

  ![Imgur](https://i.imgur.com/z64JbNf.png)

---

<br>

다시 자신감이 떨어지고 있어서 조금 쉬운문제로 자신감을 올려보려 했지만ㅎㅎㅎ<br>계속 런타임에러가 떠서 다시 자신감이 하락해버렸당ㅎㅎㅎㅎ헤헤헿

<br>

## 작성 코드

```python
def solution(n):
    answer = 0
    a = 1
    b = 2
    c = 0

    if n < 3:
        return n

    for i in range(3,n+1):
        c = b
        b = (a + b) % 1000000007
        a = c

    return b
```

이 문제는 규칙이 있다! 피보나치 수열의 규칙이!

그래서 피보나치 수열처럼 코드를 작성했다<br>

그리고 배열을 사용해서도 공간사용 때문인지 런타임에러 떠서 변수를 사용했다...<br>

정말 런타임에러가 싫어지는 문제였다.