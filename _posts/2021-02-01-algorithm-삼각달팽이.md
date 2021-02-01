---
title:  "삼각달팽이 from 프로그래머스 "
date:   2021-02-01 20:25:50 +0900
categories: 
  - 알고리즘
tags:
  - 프로그래머스
toc: true
toc_sticky: true
---



# 삼각달팽이

출처 : [https://programmers.co.kr/learn/courses/30/lessons/68645](https://programmers.co.kr/learn/courses/30/lessons/68645)

<br>

>문제 풀이 흐름
>
>1. direction 값을 가지고 어떤 방향으로 움직일지 확인한다
>
>2. 해당 값이 0일때, 즉 비어있다면 숫자를 채우고 +1 증가한다
>
>3. 해당 값이 out of index 이거나 값이 존재하는경우 
>
>   방향을 변경하고, 다음에 입력할 좌표로 이동시킨다
>
>4. 위의 과정을 반복하다 마지막 숫자까지 입력이 끝나면 while문을 빠져나간다.

=> 방향이 왼쪽아래로 이동/ 오른쪽으로 평행이동 / 오른쪽위로 이동 3가지이므로 경우를 나눴다.

<br>

### while문 안에 작성

```python
def solution(n):
	# 삼각형 리스트
    answer = [[0]*i for i in range(1, n + 1)]
	# 방향 : 0(왼쪽아래로), 1(바닥에서 오른쪽평행이동), 2(오른쪽위로)
    direction = [0, 1, 2]
    # 삼각형에 들어갈 숫자
    number = 1
    # x좌표 ,y좌표 , direction 순으로 상태표시 리스트
    status = [0] * 3
    # 실제로 리턴할 값
    return_answer = []
    
	# 삼각형에 들어갈 숫자 number는 n(n+1)/2까지이다.
    while number * 2 <= n * (n+1):
        # 왼쪽 아래로 내려가는 부분
        if status[2] % 3 == 0:
            # 비어있는 값이면 number 넣기 & number 1증가
            if answer[status[0]][status[1]] == 0:
                answer[status[0]][status[1]] = number
                number += 1
                # 제한범위 넘어가면 => 다음방향으로 변경 => y좌표 1증가 & 방향 변경
                if status[0] == len(answer) - 1:
                    status[1] += 1
                    status[2] += 1
                # 제한범위 내 => 그대로 진행 => x좌표 1증가
                else:
                    status[0] += 1
            # 이미 채워져있는 값이다 => 다음방향으로 변경 
						# 한단계 내려왔으므로 이전으로 돌리고 => y좌표 1증가 & 방향 변경
            else:
				# 이전 값으로 돌리기
                status[0] -= 1
				# 이전 값으로 돌리기 & y좌표 1증가
                status[1] += 1
				# 방향 변경
                status[2] += 1 
 
        # 오른쪽 옆으로 가는 부분
        elif status[2]%3 == 1:
            # 비어있는 값이면 number 넣기 & number 1증가
            if answer[status[0]][status[1]] == 0:
                answer[status[0]][status[1]] = number
                number += 1
                # 제한범위 넘어가면 => 다음방향으로 변경 => x,y좌표 -1증가 & 방향 변경
                if status[1] == len(answer[status[0]]) - 1:
                    status[0] -= 1
                    status[1] -= 1
                    status[2] += 1
                # 제한범위 내 => 그대로 진행 => y좌표 1증가
                else:
                    status[1] += 1
            # 이미 채워져있는 값이다 => 이전좌표로 돌리고 x,y좌표 -1증가 & 방향 변경
            else:
                status[0] -= 1
                status[1] -= 2
                status[2] += 1

        # 오른쪽 위로 가는 부분
        else:
            # 비어있는 값이면 number 넣기 & number 1증가
            if answer[status[0]][status[1]] == 0:
                answer[status[0]][status[1]] = number
                number += 1
                # 제한범위 넘어가면 => 다음방향으로 변경 => x 1증가 & 방향 변경
                if status[0] == 0:
                    status[0] += 1
                    status[2] += 1
                # 제한범위 내 => 그대로 진행 => x,y좌표 -1증가
                else:
                    status[0] -= 1
                    status[1] -= 1
            # 이미 채워져있는 값이다 => 이전좌표로 돌리고 x좌표 1증가 & 방향 변경
            else:
                status[0] += 2
                status[1] += 1
                status[2] += 1
    
    for i in answer:
        return_answer += i
    return return_answer
```

=> while문 안에 다 넣어서 계산하니 복잡해 보인다<br>

=> 가독성을 위해 함수를 사용해보겠다.

<br>

### 함수 사용

```python
def left_down(answer, status, number):
    # 비어있는 값이면 number 넣기 & number 1증가
    if answer[status[0]][status[1]] == 0:
        answer[status[0]][status[1]] = number
        number += 1
        # 제한범위 넘어가면 => 다음방향으로 변경 => y좌표 1증가 & 방향 변경
        if status[0] == len(answer) - 1:
            status[1] += 1
            status[2] += 1
        # 제한범위 내 => 그대로 진행 => x좌표 1증가
        else:
            status[0] += 1
    # 이미 채워져있는 값이다 => 다음방향으로 변경 
    # 한단계 내려왔으므로 이전으로 돌리고 => y좌표 1증가 & 방향 변경
    else:
        # 이전 값으로 돌리기
        status[0] -= 1
        # 이전 값으로 돌리기 & y좌표 1증가
        status[1] += 1
        # 방향 변경
        status[2] += 1
    return answer, status, number


def right_parallel(answer, status, number):
    # 비어있는 값이면 number 넣기 & number 1증가
    if answer[status[0]][status[1]] == 0:
        answer[status[0]][status[1]] = number
        number += 1
        # 제한범위 넘어가면 => 다음방향으로 변경 => x,y좌표 -1증가 & 방향 변경
        if status[1] == len(answer[status[0]]) - 1:
            status[0] -= 1
            status[1] -= 1
            status[2] += 1
        # 제한범위 내 => 그대로 진행 => y좌표 1증가
        else:
            status[1] += 1
    # 이미 채워져있는 값이다 => 이전좌표로 돌리고 x,y좌표 -1증가 & 방향 변경
    else:
        status[0] -= 1
        status[1] -= 2
        status[2] += 1
    return answer, status, number


def right_up(answer, status, number):
    # 비어있는 값이면 number 넣기 & number 1증가
    if answer[status[0]][status[1]] == 0:
        answer[status[0]][status[1]] = number
        number += 1
        # 제한범위 넘어가면 => 다음방향으로 변경 => x 1증가 & 방향 변경
        if status[0] == 0:
            status[0] += 1
            status[2] += 1
        # 제한범위 내 => 그대로 진행 => x,y좌표 -1증가
        else:
            status[0] -= 1
            status[1] -= 1
    # 이미 채워져있는 값이다 => 이전좌표로 돌리고 x좌표 1증가 & 방향 변경
    else:
        status[0] += 2
        status[1] += 1
        status[2] += 1
    return answer, status, number


def solution(n):
	# 삼각형 리스트
    answer = [[0]*i for i in range(1, n + 1)]
    # 방향 : 0(왼쪽아래로), 1(바닥에서 오른쪽평행이동), 2(오른쪽위로)
    direction = [0, 1, 2]
    # 삼각형에 들어갈 숫자
    number = 1
    # x좌표 ,y좌표 , direction 순으로 상태표시 리스트
    status = [0] * 3
    # 실제로 리턴할 값
    return_answer = []
    
    # 삼각형에 들어갈 숫자 number는 n(n+1)/2까지이다.
    while number * 2 <= n * (n+1):
        # 왼쪽 아래로 내려가는 부분
        if status[2] % 3 == 0:
             answer, status, number = left_down(answer, status, number)
        # 오른쪽 옆으로 가는 부분
        elif status[2]%3 == 1:
            answer, status, number = right_parallel(answer, status, number)
        # 오른쪽 위로 가는 부분
        else:
            answer, status, number = right_up(answer, status, number)
    
    for i in answer:
        return_answer += i
    return return_answer
```

함수 left_down , right_parallel, right_up 에서 겹치는 코드가 많다

=> 클래스를 사용해서 작성해보겠다.

<br>

### 클래스 사용해서 작성

```python
class Triangle:

    def __init__(self, n):
        # 삼각형 리스트
        self.answer = [[0]*i for i in range(1, n + 1)]
        # 삼각형에 들어갈 숫자
        self.number = 1

    def left_down(self, status):
        # 비어있는 값이면 number 넣기 & number 1증가
        if self.answer[status[0]][status[1]] == 0:
            self.answer[status[0]][status[1]] = self.number
            self.number += 1
            # 제한범위 넘어가면 => 다음방향으로 변경 => y좌표 1증가 & 방향 변경
            if status[0] == len(self.answer) - 1:
                status[1] += 1
                status[2] += 1
            # 제한범위 내 => 그대로 진행 => x좌표 1증가
            else:
                status[0] += 1
        # 이미 채워져있는 값이다 => 다음방향으로 변경 
        # 한단계 내려왔으므로 이전으로 돌리고 => y좌표 1증가 & 방향 변경
        else:
            # 이전 값으로 돌리기
            status[0] -= 1
            # 이전 값으로 돌리기 & y좌표 1증가
            status[1] += 1
            # 방향 변경
            status[2] += 1
        return status


    def right_pararell(self, status):
        # 비어있는 값이면 number 넣기 & number 1증가
        if self.answer[status[0]][status[1]] == 0:
            self.answer[status[0]][status[1]] = self.number
            self.number += 1
            # 제한범위 넘어가면 => 다음방향으로 변경 => x,y좌표 -1증가 & 방향 변경
            if status[1] == len(self.answer[status[0]]) - 1:
                status[0] -= 1
                status[1] -= 1
                status[2] += 1
            # 제한범위 내 => 그대로 진행 => y좌표 1증가
            else:
                status[1] += 1
        # 이미 채워져있는 값이다 => 이전좌표로 돌리고 x,y좌표 -1증가 & 방향 변경
        else:
            status[0] -= 1
            status[1] -= 2
            status[2] += 1
        return status


    def right_up(self, status):
        # 비어있는 값이면 number 넣기 & number 1증가
        if self.answer[status[0]][status[1]] == 0:
            self.answer[status[0]][status[1]] = self.number
            self.number += 1
            # 제한범위 넘어가면 => 다음방향으로 변경 => x 1증가 & 방향 변경
            if status[0] == 0:
                status[0] += 1
                status[2] += 1
            # 제한범위 내 => 그대로 진행 => x,y좌표 -1증가
            else:
                status[0] -= 1
                status[1] -= 1
        # 이미 채워져있는 값이다 => 이전좌표로 돌리고 x좌표 1증가 & 방향 변경
        else:
            status[0] += 2
            status[1] += 1
            status[2] += 1
        return status


def solution(n):
    # 방향 : 0(왼쪽아래로), 1(바닥에서 오른쪽평행이동), 2(오른쪽위로)
    direction = [0, 1, 2]
    # x좌표 ,y좌표 , direction 순으로 상태표시 리스트
    status = [0] * 3
    # 리턴할 값을 담을 리스트
    return_answer = []
    # 삼각형 관련 클래스 생성
    triange = Triangle(n)
    
    # 삼각형에 들어갈 숫자 number는 n(n+1)/2까지이다.
    while triange.number * 2 <= n * (n+1):
        # 왼쪽 아래로 내려가는 부분
        if status[2] % 3 == 0:
             status = triange.left_down(status)
        # 오른쪽 옆으로 가는 부분
        elif status[2]%3 == 1:
            status = triange.right_pararell(status)
        # 오른쪽 위로 가는 부분
        else:
            status = triange.right_up(status)
    
    for i in triange.answer:
        return_answer += i
    return return_answer
```

<br>

### 다른 사람 모범 답안

```python
import sys
sys.setrecursionlimit(1000000)

class Triangle():
    def __init__(self,n):
        self.board = [[0 for _ in range(n)] for _ in range(n)]
        self.cnt = 1
        self.answer = []
        self.ldown(0,0,n)
        for i in range(n):
            for j in range(i+1):
                self.answer.append(self.board[i][j])

    def ldown(self,x,y,n):
        if n < 1:
            return 0
        for i in range(x,x+n):
            self.board[i][y] = self.cnt
            self.cnt += 1
        self.right(i,y+1,n-1)

    def right(self,x,y,n):
        if n < 1:
            return 0
        for j in range(y,y+n):
            self.board[x][j] = self.cnt
            self.cnt += 1
        self.lup(x-1,j-1,n-1)

    def lup(self,x,y,n):
        if n < 1:
            return 0
        for i in range(x,x-n,-1):
            self.board[i][y] = self.cnt
            y -= 1
            self.cnt += 1
        self.ldown(i+1,y+1,n-1)

def solution(n):
    triangle = Triangle(n)
    return triangle.answer
```

=> 이 분은 아예 class안에서 실행을 돌리셨다.

그리고  방향이 바뀌면서 반복횟수가 1번씩 줄어든다는 점도 활용하셨다!