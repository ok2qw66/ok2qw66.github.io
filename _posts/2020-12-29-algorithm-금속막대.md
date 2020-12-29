---
title:  "금속막대 from SW Expert Academy "
date:   2020-12-29 17:59:50 +0900
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
5차시 6일차 - 금속막대
[https://swexpertacademy.com/main/learn/course/subjectList.do?courseId=AVuPDYSqAAbw5UW6](https://swexpertacademy.com/main/learn/course/subjectList.do?courseId=AVuPDYSqAAbw5UW6)
<br>

## 문제

해당 링크 참고!

---

<br>

앞에 올린 행렬찾기 뒤에 이어진 문제이다.<br>

앞에 문제에 비해 난이도는 낮은 편인거 같다. (행렬찾기는 정답률 60퍼이고, 이 문제는 80퍼 이상이다)

<br>

## 작성 코드

```python
def find(stock_list, n):
    answer = [stock_list[0],stock_list[1]]
    bool_list = [0]* n
    bool_list[0] = 1

    # bool_list 모든 값이 1 일때까지 ==> 모든 값이 다 answer에 들어갈 때까지
    while sum(bool_list)!= n:
        for x in range(len(bool_list)):
            # 아직 answer에 들어가지 않음 && answer의 마지막 값과 특정 막대 처음이 같을 때
            # answer 뒤에 붙여주기
            if not bool_list[x] and answer[-1] == stock_list[2 * x]:
                bool_list[x] = 1
                answer.append(stock_list[2 * x])
                answer.append(stock_list[2 * x + 1])
            # 아직 answer에 들어가지 않음 && answer의 처음값과 특정 막대 마지막이 같을 때
            # answer 앞에 붙여주기    
            elif not bool_list[x] and answer[0] == stock_list[2 * x + 1]:
                bool_list[x] = 1
                answer.insert(0, stock_list[2 * x + 1])
                answer.insert(0, stock_list[2 * x])
	# 출력을 위해 ' '로 구분된 string으로 변환
    return ' '.join(answer)


if __name__ == '__main__':
    T = int(input())
    # 여러개의 테스트 케이스가 주어지므로, 각각을 처리합니다.
    for test_case in range(1, T + 1):
        n = int(input())
        stock_list = list(map(str, input().split()))
        answer = find(stock_list,n)
        print(f'#{test_case} {answer}')
```
