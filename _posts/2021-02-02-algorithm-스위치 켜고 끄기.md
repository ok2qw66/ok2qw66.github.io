---
title:  "스위치 켜고 끄기 from baekjoon "
date:   2021-02-02 20:25:50 +0900
categories: 
  - 알고리즘
tags:
  - 백준
toc: true
toc_sticky: true
---



# 삼각달팽이

출처 : [https://www.acmicpc.net/problem/1244](https://www.acmicpc.net/problem/1244)

<br>

### 작성한 코드

```python
# switch 개수
s_num = int(input())
# switch 리스트 받아오기
switch = list(map(int, input().split()))
# 학생의 수
stu_num = int(input())
# 출력할 답 담을 string
answer = ''

# 학생 수 만큼 for문
for _ in range(stu_num):
    # 성별, 숫자를 입력받기
    gender, number = list(map(int,input().split()))
    # 남자라면
    if gender == 1:
        # number 배수부터 끝까지 switch의 숫자 변경
        # index는 0부터 시작, 숫자는 1부터 시작이므로 number-1 로 인덱스 계산 
        for i in range(number, len(switch) + 1, number):
            switch[i-1] = 0 if switch[i-1] else 1
    # 여자라면
    else:
        # 자기자신 값은 무조건 반대로 적용되니 미리 적용하기
        switch[number-1] = 0  if switch[number-1] else 1
        # 1차이(number 바로 옆)부터 number차이(인덱스0까지) 확인
        for i in range(1, number):
            # 리스트 크기 밖에 넘어가지 않으면서 양옆 두값이 같다면
            if 0 <= number-1-i and number-1+i < len(switch) and switch[number-1-i] == switch[number-1+i]:
                # 먼저 현재 존재값과 반대되는 값을 찾고
                update_val = 0 if switch[number-1-i] else 1
                # 양옆 switch 값에 넣어주기
                switch[number-1-i] = update_val
                switch[number-1+i] = update_val
            # 조건에 맞지 않는다면 for문 나가기
            else:
                break

# idx 1부터 시작하고 switch에서 하나씩 가져오기                
for idx, s in enumerate(switch, 1):
    # ' '으로 연결하기
    answer += str(s) + ' '
    # 만약 20개가 꽉찼다면 마지막 ' ' 없애고 라인변경하기
    if idx % 20 == 0:
        answer = answer.rstrip()+'\n'
# 마지막 ' ' 없애고 답 출력
print(answer.strip())
```

