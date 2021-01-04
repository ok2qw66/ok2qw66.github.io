---
title:  " 택배 배송 from 백준 "
date:   2021-01-04 10:34:50 +0900
categories: 
  - 알고리즘
tags:
  - 백준
toc: true
toc_sticky: true
---

<br>

## 문제 출처

백준 5972 택배 배송
[https://www.acmicpc.net/problem/5972](https://www.acmicpc.net/problem/5972)
<br>

## 문제

Dijkstra 알고리즘 사용하는 문제인데... 어제 현타가 온 문제다

왜 틀렸는지 오류를 찾지 못해 땅구덩이를 파다못해 드러누웠다ㅎㅎ

<br>근데 해결하고 나니 별거 아니었던 문제....ㅎㅎㅎㅎ

<br>

#### 오류 발생한 코드

```python
import heapq

def dijkstra(board,start,end):
    distances = {vertex: float('inf') for vertex in board.keys()}
    distances[start] = 0
    
    queue = []
    heapq.heappush(queue,[distances[start], start])

    while queue:
        current_distance, current_vertex = heapq.heappop(queue)
        
        if distances[current_vertex] < current_distance:
            continue

        for adjacent, weight in board[current_vertex]:
            distance = current_distance + weight
            if distance < distances[adjacent]:
                distances[adjacent] = distance
                heapq.heappush(queue,[distance,adjacent])

    return distances[end]


n, m = list(map(int,input().split()))
board = {i:[] for i in range(1,n+1)}
for _ in range(m):
    start, end, value = list(map(int, input().split()))
    if start < end:
        board[start].append([end,value])
    else:
        board[end].append([start,value])

answer = dijkstra(board,1,n)
print(answer)
```

- 오류 발생 원인 : 분기처리해서 board에 넣었다

  ```python
  if start < end:
          board[start].append([end,value])
      else:
          board[end].append([start,value])
  ```

  해당 문제에서 연결지점끼리 시작/끝이 정해져 있지 않기 때문에, 즉 방향이 없기 때문에

  이렇게 board를 세팅하면 end에서 start로 가지 못한다....

  그래서 양방향을 다 board에 넣어주어야 한다

  ``이 간단한 오류를 찾지 못해 개고생 했다니 너무 슬프다... 아직 난 멀었나 봄...정신차리자``

<br>

### 완성 코드

```python
import heapq

def dijkstra(board,start,end):
    distances = {vertex: float('inf') for vertex in board.keys()}
    distances[start] = 0
    # print(distances)
    queue = []
    heapq.heappush(queue,[distances[start], start])

    while queue:
        current_distance, current_vertex = heapq.heappop(queue)
        print(current_distance, current_vertex)
        if distances[current_vertex] < current_distance:
            continue

        for adjacent, weight in board[current_vertex]:
            distance = current_distance + weight
            if distance < distances[adjacent]:
                distances[adjacent] = distance
                heapq.heappush(queue,[distance,adjacent])

    print(distances)
    return distances[end]


n, m = list(map(int,input().split()))
board = {i:[] for i in range(1,n+1)}
# print(board)
for _ in range(m):
    start, end, value = list(map(int, input().split()))
    board[start].append([end,value])
    board[end].append([start,value])

answer = dijkstra(board,1,n)
print(answer)
```
