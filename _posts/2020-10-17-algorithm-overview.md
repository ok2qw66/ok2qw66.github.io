---
title:  "알고리즘 정리"
date:   2020-10-17 23:48:50 +0900
categories: 
  - 알고리즘
tags:
  - 백준
toc: true
---

<br>

## 1. 입력

### 변수 입력받기

#### 1개 변수 입력받기

```python
n = int(input())
```

#### 여러개 변수 입력받기

```python
N,M = map(int,input().split())
```

<br>

### 리스트 입력받기

#### 1차원 리스트 입력받기

```python
commands = list(map(int,input().split()))
```

<br>

#### 2차원 리스트 (N x M 배열) 입력받기

```python
board = [list(map(int,input().split())) for _ in range(N)]
```

열의 개수 * 행의 개수로 표현된다

```python
board = [[0 for _ in range(M)] for _ in range(N)]
board = [[0]*M for _ in range(N)]
```

<br>

### dict 형태로 입력받기

```python
L = int(input())
snake_act = dict([list(map(str,input().split())) for _ in range(L)])
```

<br>

<br>

## 2. 선언

### 1차원 리스트 0으로 초기화 선언

```python
dice, temp = [0]*6, [0]*6
```

### 2차원 리스트 0으로 초기화 선언

```python
board = [[0 for _ in range(M)] for _ in range(N)]
```

### 변수 여러개 0으로 초기화 선언

```python
rx,ry,bx,by = [0]*4
```

<br>

<br>

## 3. 문제풀이 방식

### DFS : 깊이 우선 탐색 -> stack,재귀함수,visited

```python
def dfs(x, y):
    visited[x][y] = True
    
    #원하는 작업 수행
    
    # 조건에 맞는다면 dfs 재귀호출
    if visited[next_x][next_y]:
        dfs(next_x,next_y)
	else:
        return
        
        
visited= [[0]*n]*m
dfs((0,0))
```

<br>

### BFS : 너비 우선 탐색 -> queue,while,visited

```python
from collections import deque
visited= [[0]*n]*m
q=deque()
q.append((0,0,1))

while q:
        # FIFO : queue
        # list에서는 q.pop(0) 과 동일
        x,y,depth = q.popleft()
        # 원하는 작업 수행
        
        #방문했는지 체크
        if not visited[x][y]:
            visited[x][y] = True
            q.append((x,y,depth+1))
```

<br>

### Dynamic Programming -> 이전 계산값을 저장

**ex. 피보나치 수열**

```python
def fibo_dp(num):
    cache =  [ 0 for index in range(num + 1) ]
    cache[0] = 0
    cache[1] = 1
    
    for index in range(2, num + 1):
        cache[index] = cache[index - 1] + cache[index - 2]
    return cache[num]
```



<br>

### 방향 or 이동

```python
dx = (0,0,-1,1)
dy = (1,-1,0,0)
```

(0,1) , (0,-1) , (-1,0) , (1,0) 순으로 방향이동

<br>

<br>

## 3. 예외 처리

```python
try:
    # 실행하려는 구문
except Exception as e:
    continue
```

<br>

<br>

## 4. 문제 풀 때 기억해야 할 TIP!

### "값이 0이 아니면 True이다!"

### 리스트 전체 복사

```python
# 원래 보드의 상태를 b에 복사
b = [x[:] for x in board]
```

<br>

### 가장 큰값 찾기

```python
for i in range(n):
	answer = max(answer,max(board[i]))
```

- max(board[i]) => board[i] 리스트에서 가장큰 값

<br>

### 가장 큰 수 : inf

```python
# 파이썬이 제공하는 inf 를 사용해보세요. inf는 어떤 숫자와 비교해도 무조건 크다고 판정됩니다.

min_val = float('inf')
min_val > 10000000000

#inf에는 음수 기호를 붙이는 것도 가능합니다.
max_val = float('-inf')
```

<br>

### 몫과 나머지 순으로 출력

```python
print(*divmod(a, b))
```

<br>

### 2차원 리스트 뒤집기 - zip

```python
mylist = [[1,2,3], [4,5,6], [7,8,9]]
# new_list = list(map(list, zip(mylist[0],mylist[1],mylist[2]))) 와 같은 의미
new_list = list(map(list, zip(*mylist)))   # [[1,4,7], [2,5,8], [3,6,9]]

animals = ['cat', 'dog', 'lion']
sounds = ['meow', 'woof', 'roar']
answer = dict(zip(animals, sounds)) # {'cat': 'meow', 'dog': 'woof', 'lion': 'roar'}
```

<br>

### 이진 탐색하기 - binary search

```python
import bisect
mylist = [1, 2, 3, 7, 9, 11, 33]
print(bisect.bisect(mylist, 3))


# 다른 언어로 작성시에 참고
def bisect(a, x, lo=0, hi=None):
    if lo < 0:
        raise ValueError('lo must be non-negative')
    if hi is None:
        hi = len(a)
    while lo < hi:
        mid = (lo + hi) // 2
        if a[mid] < x:
            lo = mid + 1
        else:
            hi = mid
    return lo
```



### <br><br>

# 문제 예시 from 백준 삼성 기출 문제

## 출처: https://jeongchul.tistory.com/ [Jeongchul]

<br>

## 1. 주사위 굴리기

#### 문제 링크 : [https://www.acmicpc.net/problem/14499](https://www.acmicpc.net/problem/14499)

<br>

```python
# board N x M 행렬, 주사위 위치(x,y) ,명령의 개수
N,M,x,y,K = map(int,input().split())
board = [list(map(int,input().split())) for _ in range(N)]
commands = list(map(int,input().split()))
# 순서대로 동/서/북/남
dx,dy = (0,0,-1,1),(1,-1,0,0)

#가장 처음 주사위는 0으로 초기화
dice, temp = [0]*6, [0]*6
#주사위의 현재 인덱스 정하기
direction = [
    (2, 0, 5, 3, 4, 1), # 동쪽 1
    (1, 5, 0, 3, 4, 2), # 서쪽 2
    (4, 1, 2, 0, 5, 3), # 북쪽 3
    (3, 1, 2, 5, 0, 4) # 남쪽 4
]

for command in commands:
    # 인덱스 일치시키기 위해
    # EX. 동쪽은 1번, 인덱스 0에 해당
    command -=1
    # 움직일 x,y 좌표 구한다
    x,y = x+dx[command], y+dy[command]
    # 범위를 벗어나면 x,y좌표 원상복구 시키고 진행
    if x<0 or x>=N or y<0 or y>=M:
        x,y = x-dx[command], y-dy[command]
        continue
    # 현재 주사위를 temp에 복사
    for idx in range(6):
        temp[idx] = dice[idx]
    # 주사위를 굴렸을 때 번호 매핑
    for idx in rnage(y)
     	# 원하는 방향으로 이동했을 때 direction[command]
        # 현재 인덱스에서 이동할 인덱스를 찾고, direction[command][idx]
        # 그 인덱스의 주사위 값을 주사위에 매핑
        dice[idx] = temp[direction[command][idx]]
    # 보드에 값이 존재한다면 바닥면에 값을 넣고 보드값은 0
    if board[x][y]:
        dice[5] = board[x][y]
        board[x][y] = 0
    # 값이 없다면 주사위에 값을 넣기
    else:
        board[x][y] = dice[5]
    # 맨위의 주사위 값 출력
    print(dice[0])
```



<br>

## 2. 2048 (Easy)

#### 문제 링크 : [https://www.acmicpc.net/problem/12100](https://www.acmicpc.net/problem/12100)

<br>

```python
from collections import deque
n = int(input())
board = [list(map(int,input(),split())) for _ in range(n)]
answer,q = 0,deque()

# 값이 존재하면 q에 넣기
def get(i,j):
    # 0이 아니라면
    if board[i][j]:
        q.append(board[i][j])
        
# 합치기(row index, col index, y방향,x방향)
def merge(i,j,di,dj):
    while q:
        # FIFO : queue
        # list에서는 q.pop(0) 과 동일
        x = q.popleft()
        # 현재 board값이 0이라면 그대로 x값 넣기
        if not board[i][j]:
            board[i][j] = x
        # 값이 일치한다면 2배 증가
        elif board[i][j] == x:
            board[i][j] = x*2
            # 한번 합쳐지면 더이상 합쳐지지 않는다
            i,j = i+di,j+dj
        # 값이 다르면 위치 옮기고 값 넣기
        else:
            i,j = i+di,j+dj
            board[i][j] = x
            
            
# 방향에 따라 이동이 달라지는 이동함수
def move(k):
    # 위로 이동
    if k==0:
        for j in range(n):
            for i in range(n):
                # 열 별로 값 모으고 합치기
                get(i,j)
            # 위로 이동 => row index가 0 ~ n-1으로 증가
            merge(0,j,1,0)
    # 아래로 이동
    elif k==1:
        for j in range(n):
            # 위로 이동과 반대방향으로 진행
            for i in range(n-1,-1,-1):
                # 열 별로 값 모으고 합치기
                get(i,j)
            # 아래로 이동 => row index가 n-1 ~ 0으로 감소
            merge(n-1,j,-1,0)
    # 오른쪽으로 이동
    elif k==2:
        for i in range(n):
            for j in range(n-1,-1,-1):
                # 행 별로 값 모으고 합치기
                get(i,j)
            # 오른쪽으로 이동 => col index가 n-1 ~ 0으로 감소
            merge(i,n-1,0,-1)
    # 왼쪽으로 이동
    else:
        for i in range(n):
            # 오른쪽으로 이동과 반대방향으로 진행
            for j in range(n):
                # 행 별로 값 모으고 합치기
                get(i,j)
            # 왼쪽으로 이동 => col index가 0 ~ n-1으로 증가
            merge(i,0,0,1)
            

def solve(count):
    global board,answer
    # 5번 다 움직였다면 가장 큰 값이 answer
    if count == 5:
        for i in range(n):
            answer = max(answer,max(board[i]))
        return
    # 원래 보드의 상태를 b에 복사
    b = [x[:] for x in board]
    # 동서남북 방향으로 이동
    for k in range(4):
        # 해당 방향으로 이동 & 합치기
        move(k)
  		# 5번 움직일때까지 재귀호출
        solve(count+1)
        # board 원상복구
        board = [x[:] for x in b]
        
solve(0)
print(answer)
```



<br>

## 3. 구슬 탈출2

#### 문제 링크 : [https://www.acmicpc.net/problem/13460](https://www.acmicpc.net/problem/13460)

<br>

```python
from collections import deque

n,m = map(int,input().split())
board = [list(map(int,input().split())) for _ in range(n)]
visited = [[[[False]*m for _ in range(n)] for _ in range(m)] for _ in range(n)]

dx, dy = (-1, 0, 1, 0), (0, 1, 0, -1)
q = deque()


def init():
    rx,ry,bx,by = [0]*4
    for i in range(n):
        for j in range(m):
            # board에 빨간구슬 좌표 저장
            if board[i][j] == 'R':
                rx,ry = i,j
            # board에 파란구슬 좌표 저장
            elif board[i][j] == 'B':
                bx,by = i,j
    # 위치 정보와 depth
    q.append((rx,ry,bx,by,1))
    visited[rx][ry][bx][by] = True
    

#벽이나 구멍을 만날때까지 이동
def move(x,y,dx,dy):
    #이동한 칸 수
    count = 0
    #다음 이동이 벽/구멍이 아닐때까지 반복
    while board[x+dx][y+dy] != '#' and board[x][y] != 'O':
        x += dx
        y += dy
        count += 1
    return x,y,count



def bfs():
    init()
    while q:
        rx,ry,bx,by,depth = q.popleft()
        # 10 초과이면 패스!
        if depth > 10:
            break
        # 서동남북 순으로 이동
        for i in range(len(dx)):
            n_rx,n_ry,r_count = move(rx,ry,dx[i],dy[i])
            n_bx,n_by,b_count = move(bx,by,dx[i],dy[i])
            # 파란구슬이 구멍에 빠졌다면 실패!
            if board[n_bx][n_by] == '0':
                continue
            # 파란구슬은 안빠지고, 빨간구슬이 구멍에 빠졌다면 성공!
            if board[n_rx][n_ry] == '0':
                print(depth)
                return
            # 빨간구슬과 파란구슬이 이동할 위치가 같다면
            if n_rx == n_bx and n_ry == n_by:
                # 이동거리가 더 많은 구슬은 한칸뒤로
                if r_count > b_count:
                    n_rx -= dx[i]
                    n_ry -= dy[i]
				else:
                    n_bx -= dx[i]
                    n_by -= dy[i]
            # 이동할 위치를 방문하지 않았다면 방문처리하기
            if not visited[n_rx][n_ry][n_bx][n_by]:
                visited[n_rx][n_ry][n_bx][n_by] = True
                q.append((n_rx,n_ry,n_bx,n_by,depth+1))
    # 더이상 진행할 수 없다면 실패
    print(-1)
    
bfs()
```



<br>

## 4. **테트로미노** 

#### 문제 링크 : [https://www.acmicpc.net/problem/14500](https://www.acmicpc.net/problem/14500)

<br>

```python
n, m = map(int, input().split())
board = [list(map(int, input().split())) for _ in range(n)]
tetromino = [
    [(0,0), (0,1), (1,0), (1,1)], # ㅁ
    [(0,0), (0,1), (0,2), (0,3)], # ㅡ
    [(0,0), (1,0), (2,0), (3,0)], # ㅣ
    [(0,0), (0,1), (0,2), (1,0)], 
    [(1,0), (1,1), (1,2), (0,2)],
    [(0,0), (1,0), (1,1), (1,2)], # ㄴ
    [(0,0), (0,1), (0,2), (1,2)], # ㄱ
    [(0,0), (1,0), (2,0), (2,1)],
    [(2,0), (2,1), (1,1), (0,1)],
    [(0,0), (0,1), (1,0), (2,0)], 
    [(0,0), (0,1), (1,1), (2,1)],
    [(0,0), (0,1), (0,2), (1,1)], # ㅜ
    [(1,0), (1,1), (1,2), (0,1)], # ㅗ
    [(0,0), (1,0), (2,0), (1,1)], # ㅏ
    [(1,0), (0,1), (1,1), (2,1)], # ㅓ
    [(1,0), (2,0), (0,1), (1,1)],
    [(0,0), (1,0), (1,1), (2,1)],
    [(1,0), (0,1), (1,1), (0,2)],
    [(0,0), (0,1), (1,1), (1,2)]
]


def find(x, y):
    global answer
    for i in range(19):
        result = 0
        for j in range(4):
            try:
                next_x = x+tetromino[i][j][0] # x 좌표
                next_y = y+tetromino[i][j][1] # y 좌표
                result += board[next_x][next_y]
            except IndexError:
                continue
        answer = max(answer, result)
        
def solve():
    for i in range (n):
        for j in range(m):
            find(i, j)
            
answer = 0
solve()
print(answer)
```



<br>

## 5. 퇴사

#### 문제 링크 : [https://www.acmicpc.net/problem/14501](https://www.acmicpc.net/problem/14501)

<br>

```python
n = int(input())

t, p = [0]*n, [0]*n

for i in range(n):
    t[i], p[i] = map(int, input().split())
        
dp = [0]*25

for i in range(n):
    if dp[i] > dp[i+1]: # 현재가 다음날보다 보상이 높다면
        dp[i+1] = dp[i] # 다음날 보상은 현재로
    if dp[i+t[i]] < dp[i] + p[i]: # T일 후에 받게될 금액이 현재의 보상보다 높다면
        dp[i+t[i]] = dp[i] + p[i] # T일후에 보상을 넣는다.
        
print(dp[n])
```



<br>

## 6. 연구소

#### 문제 링크 : [https://www.acmicpc.net/problem/14502](https://www.acmicpc.net/problem/14502)

<br>

```python
n, m = map(int, input().split())
board = [list(map(int, input().split())) for _ in range(n)]
visited = [[False]*m for _ in range(n)]

virus = []
num_virus = 9999
safe = -3 # 벽을 3개 꼭 세워야함.

def dfs(x, y): # virus를 퍼뜨린다.
    res = 1
    visited[x][y] = True
    for dx, dy in (-1,0), (1,0), (0,-1), (0,1): # 위, 아래, 좌, 우
        nx, ny = x+dx, y+dy
        if nx < 0 or nx >= n or ny < 0 or ny >= m: # 맵의 경계보다 커진다면
            continue # 넘어간다.
        if not (visited[nx][ny] or board[nx][ny]): # 방문하지 않았거나, 빈칸이 아니라면
            res += dfs(nx, ny)
    return res

def solve(start, wall): # 벽을 세운다
    global n, m, num_virus, visited
    if wall == 3: # 벽이 3개라면
        count = 0
        visited = [[False]*m for _ in range(n)]
        for x, y in virus:
            count += dfs(x, y)
        num_virus = min(num_virus, count)
        return
    for i in range(start, n*m): # 2차원 배열에서 x, y의 조합을 뽑습니다. 
        x = i // m
        y = int(i % m)
        if board[x][y] == 0: # 빈칸(0) 이라면
            board[x][y] = 1 # 벽(1)을 세운다
            solve(i+1, wall+1)
            board[x][y] = 0 # 위에서 세운 벽을 다시 빈칸(0)으로 만든다. (초기화)  

for i in range(n):
    for j in range(m):
        if board[i][j] != 1: # 벽이 아니라면,
            safe += 1
        if board[i][j] == 2: # virus라면
            virus.append((i, j))
            
solve(0, 0)
print(safe - num_virus)
```



<br>

## 7. 로봇청소기

#### 문제 링크 : [https://www.acmicpc.net/problem/14503](https://www.acmicpc.net/problem/14503)

<br>

```python
n, m = map(int, input().split())
x, y, d = map(int, input().split())
board = [list(map(int, input().split())) for _ in range(n)]

dx, dy = (-1, 0, 1, 0), (0, 1, 0, -1) # 북쪽(0), 동쪽(1), 남쪽(2), 서쪽(3)
board[x][y] = 2 # 빈칸(0), 벽(1), 청소(2)

def solve(x, y, d, answer):
    while True:
        check = False
        for k in range(4): # 4방향을 탐색한다.
            nd = (d + 3) % 4 # 회전
            next_x, next_y = x + dx[nd], y + dy[nd]
            d = nd
            if not board[next_x][next_y]: # 빈칸이라면
                board[next_x][next_y] = 2 # 청소한다.
                answer += 1 # 청소 횟수 증가
                x, y = next_x, next_y
                check = True
                break
        if not check: # 4방향으로 이동이 불가능하고,
            if board[x - dx[d]][y - dy[d]] == 1: # 후진시 벽이라면
                return answer # 작동 종료
            else:
                x, y = x - dx[d], y - dy[d] # 후진한다. 

print(solve(x, y, d, 1))
```



<br>

## 8. 스타트와 링크

#### 문제 링크 : [https://www.acmicpc.net/problem/14889](https://www.acmicpc.net/problem/14889)

<br>

```python
n= int(input())
team = [list(map(int,input().split())) for _ in range(n)]

# True: 스타트팀 / False: 링크팀
myteam = [False] *n
answer = 1e9


def solve(count,index):
    global answer
    
    # 전체 n개 확인했다면 종료
    inf index == n:
        return
    # 절반체크 했다면
    if count == n//2:
        s_team, link_team = 0,0
        for i in range(n):
            for j in range(n):
                # i,j번째 팀이 모두 True(스타트팀)이라면
                if myteam[i] and myteam[j]:
                    s_team += team[i][j]
                # i,j번째 팀이 모두 False(링크팀)이라면
                if not myteam[i] and not myteam[j]:
                    link_team += team[i][j]
        # 두 팀의 능력 차이가 최소인 갑을 answer에 넣기
        answer = min(answer,abs(s_team - link_team))
        return
    
    # 현재 인덱스 팀이 스타트팀일때 재귀함수 호출
    myteam[index] = True
    solve(count+1, index+1)
    # 현재 인덱스 팀이 링크팀일때 재귀함수 호출
    myteam[index] = False
    solve(count, index+1)

solve(0,0)
print(answer)
```



<br>