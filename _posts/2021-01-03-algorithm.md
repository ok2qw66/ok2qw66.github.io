<br>

## 문제 출처

백준 5972 택배 배송
[https://www.acmicpc.net/problem/5972](https://www.acmicpc.net/problem/5972)
<br>

## 문제

Dijkstra 알고리즘 사용하는 문제인데... 내가 작성한 코드가 왜 틀렸는지 잘 모르겠다

어디서 오류가 났는지 한번 더 봐야함

<br>

## 작성 코드

```python
import sys
from heapq import heappush, heappop

def Dijkstra(start, graph, N):
    heap = []
    distance = [1e9] * (N +1)
    distance[start] = 0
    heappush(heap, (0, start))

    ret = [0 for _ in range(N+1)]
    while heap:
        weight, location = heappop(heap)
        print(weight, location)
        if distance[location] < weight:
            continue

        for l, w in graph[location]:
            w += weight
            if w < distance[l]:
                distance[l] = w
                heappush(heap, (w, l))

    print(distance)

def solution():
    N, M = map(int, sys.stdin.readline().split())

    graph = [[] for _ in range(N + 1)]

    for _ in range(M):
        u, v, w = map(int, sys.stdin.readline().split())
        graph[u].append([v, w])
        graph[v].append([u, w])

    Dijkstra(1, graph, N)

solution()
```



### 안 되는 코드

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
    if start < end:
        board[start].append([end,value])
    else:
        board[end].append([start,value])

answer = dijkstra(board,1,n)
print(answer)
```

