---
title:  "프렌즈4블록 from 프로그래머스 "
date:   2020-10-25 02:25:50 +0900
categories: 
  - 알고리즘
tags:
  - 프로그래머스
toc: true
toc_sticky: true

---

<br>
출처 : https://programmers.co.kr/learn/courses/30/lessons/17679
<br>
<br>

## 프렌즈4블록

<br>

블라인드 공채를 통과한 신입 사원 라이언은 신규 게임 개발 업무를 맡게 되었다. 이번에 출시할 게임 제목은 프렌즈4블록.
같은 모양의 카카오프렌즈 블록이 2×2 형태로 4개가 붙어있을 경우 사라지면서 점수를 얻는 게임이다.

![board map](http://t1.kakaocdn.net/welcome2018/pang1.png)
만약 판이 위와 같이 주어질 경우, 라이언이 2×2로 배치된 7개 블록과 콘이 2×2로 배치된 4개 블록이 지워진다. 같은 블록은 여러 2×2에 포함될 수 있으며, 지워지는 조건에 만족하는 2×2 모양이 여러 개 있다면 한꺼번에 지워진다.

![board map](http://t1.kakaocdn.net/welcome2018/pang2.png)

블록이 지워진 후에 위에 있는 블록이 아래로 떨어져 빈 공간을 채우게 된다.

![board map](http://t1.kakaocdn.net/welcome2018/pang3.png)

만약 빈 공간을 채운 후에 다시 2×2 형태로 같은 모양의 블록이 모이면 다시 지워지고 떨어지고를 반복하게 된다.
![board map](http://t1.kakaocdn.net/welcome2018/pang4.png)

위 초기 배치를 문자로 표시하면 아래와 같다.

```
TTTANT
RRFACC
RRRFCC
TRRRAA
TTMMMF
TMMTTJ
```

각 문자는 라이언(R), 무지(M), 어피치(A), 프로도(F), 네오(N), 튜브(T), 제이지(J), 콘(C)을 의미한다

입력으로 블록의 첫 배치가 주어졌을 때, 지워지는 블록은 모두 몇 개인지 판단하는 프로그램을 제작하라.

### 입력 형식

- 입력으로 판의 높이 `m`, 폭 `n`과 판의 배치 정보 `board`가 들어온다.
- 2 ≦ `n`, `m` ≦ 30
- `board`는 길이 `n`인 문자열 `m`개의 배열로 주어진다. 블록을 나타내는 문자는 대문자 A에서 Z가 사용된다.

### 출력 형식

입력으로 주어진 판 정보를 가지고 몇 개의 블록이 지워질지 출력하라.

### 입출력 예제

| m    | n    | board                                            | answer |
| ---- | ---- | ------------------------------------------------ | ------ |
| 4    | 5    | [CCBDE, AAADE, AAABF, CCBBF]                     | 14     |
| 6    | 6    | [TTTANT, RRFACC, RRRFCC, TRRRAA, TTMMMF, TMMTTJ] | 15     |

### 예제에 대한 설명

- 입출력 예제 1의 경우, 첫 번째에는 A 블록 6개가 지워지고, 두 번째에는 B 블록 4개와 C 블록 4개가 지워져, 모두 14개의 블록이 지워진다.
- 입출력 예제 2는 본문 설명에 있는 그림을 옮긴 것이다. 11개와 4개의 블록이 차례로 지워지며, 모두 15개의 블록이 지워진다.

<br>

## 코드

```python
# board 정리
def re_arrange(board):
    row_len = len(board)
    col_len = len(board[0])
    
    # col 별로 밑에서부터 값있는 칸으로 채워주기
    for y in range(col_len):
        # i : 이동하는 인덱스 (값이 있는 칸을 찾아서 j에 넣어주는 역할)
        # j : 기준 인덱스 (값이 없는 칸에 위치)
        i,j = row_len-1, row_len-1
        while True:
            if i <0:
                break
            # 현재 기준 j 위치 칸에 값이 있다면 그위로 올라가기
            if board[j][y] != -1:
                j -=1
            # 현재 기준 j 위치 칸에 값이 없고 가장 근접한 i에 값이 있다면 넣어주기
            elif board[i][y] != -1:
                board[j][y],board[i][y] = board[i][y],board[j][y]
                # 한칸위로 이동
                j = j-1
            i -=1
    return board


# 2*2 블록 삭제
def delete_sqr(board,delete_list):
    count = 0
    # 제거할 칸을 -1 값으로 만들고 count 추가
    for x,y in delete_list:
        # -1 이 아니다 => 처음 지우는 경우다
        # count 추가 & 해당 칸 -1로 만들기
        if board[x][y] != -1:
            count += 1
            board[x][y] = -1
    # board 중간에 -1이 없도록 정리하기
    new_board = re_arrange(board)
    return new_board,count


# 2*2 블록 찾기
def find_sqr(board):
    delete_list = list()
    for x in range(len(board)-1):
        for y in range(len(board[0])-1):
            # 사각형 만들 때 기준이 되는 칸
            standard = board[x][y]
            # 해당 기준 칸이 비어있다면 패스
            if standard == -1:
                continue
            # 해당 기준 칸을 기준으로 옆,아래,대각선의 값이 모두 동일하다면 삭제할 배열에 추가
            if standard == board[x][y+1] and standard == board[x+1][y] and standard == board[x+1][y+1]:
                delete_list.append([x,y])
                delete_list.append([x+1,y])
                delete_list.append([x,y+1])
                delete_list.append([x+1,y+1])
    # 블록 삭제함수(+보드 재정리) 계산값을 리턴
    return delete_sqr(board,delete_list)


def solution(m, n, board):
    answer = 0
    
    nb = [[]*n]*m
    # 문자열로 구성된 row를 list 형태로 만들기
    for row in range(m):
        nb[row] = list(board[row])
                
    while True:
        nb,delete_num = find_sqr(nb)
        # 동일 사각형 블록 삭제/정리 후에 삭제한 칸 개수를 answer에 넣기
        if delete_num:
            answer += delete_num
        # 더이상 삭제한 칸이 없다 => 계산 끝
        else:
            break
    return answer
```

