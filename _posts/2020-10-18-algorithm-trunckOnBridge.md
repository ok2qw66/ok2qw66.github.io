---
title:  "다리를 지나는 트럭 from 프로그래머스"
date:   2020-10-18 23:25:50 +0900
categories: 
  - 알고리즘
tags:
  - 프로그래머스
toc: true
toc_sticky: true
---

출처 : https://programmers.co.kr/learn/courses/30/lessons/42583

<br>

## 문제 설명

트럭 여러 대가 강을 가로지르는 일 차선 다리를 정해진 순으로 건너려 합니다. 모든 트럭이 다리를 건너려면 최소 몇 초가 걸리는지 알아내야 합니다. 트럭은 1초에 1만큼 움직이며, 다리 길이는 bridge_length이고 다리는 무게 weight까지 견딥니다.
※ 트럭이 다리에 완전히 오르지 않은 경우, 이 트럭의 무게는 고려하지 않습니다.

예를 들어, 길이가 2이고 10kg 무게를 견디는 다리가 있습니다. 무게가 [7, 4, 5, 6]kg인 트럭이 순서대로 최단 시간 안에 다리를 건너려면 다음과 같이 건너야 합니다.

| 경과 시간 | 다리를 지난 트럭 | 다리를 건너는 트럭 | 대기 트럭 |
| --------- | ---------------- | ------------------ | --------- |
| 0         | []               | []                 | [7,4,5,6] |
| 1~2       | []               | [7]                | [4,5,6]   |
| 3         | [7]              | [4]                | [5,6]     |
| 4         | [7]              | [4,5]              | [6]       |
| 5         | [7,4]            | [5]                | [6]       |
| 6~7       | [7,4,5]          | [6]                | []        |
| 8         | [7,4,5,6]        | []                 | []        |

따라서, 모든 트럭이 다리를 지나려면 최소 8초가 걸립니다.

solution 함수의 매개변수로 다리 길이 bridge_length, 다리가 견딜 수 있는 무게 weight, 트럭별 무게 truck_weights가 주어집니다. 이때 모든 트럭이 다리를 건너려면 최소 몇 초가 걸리는지 return 하도록 solution 함수를 완성하세요.

### 제한 조건

- bridge_length는 1 이상 10,000 이하입니다.
- weight는 1 이상 10,000 이하입니다.
- truck_weights의 길이는 1 이상 10,000 이하입니다.
- 모든 트럭의 무게는 1 이상 weight 이하입니다.

### 입출력 예

| bridge_length | weight | truck_weights                   | return |
| ------------- | ------ | ------------------------------- | ------ |
| 2             | 10     | [7,4,5,6]                       | 8      |
| 100           | 100    | [10]                            | 101    |
| 100           | 100    | [10,10,10,10,10,10,10,10,10,10] | 110    |





## 코드

```python
def solution(bridge_length, weight, truck_weights):
    answer = 1
    # 현재 다리를 건너는 [트럭무게,지나고있는 다리길이] 넣는 리스트
    queue = []
    # answer 1초에 [첫번째 트럭무게,다리1번째지나고 있음]을 의미
    queue.append([int(truck_weights.pop(0)),1])
    
    # 다리 지나고 있는 트럭이 있다면 진행
    while queue:
        #print(queue)
        # 1초 증가
        answer += 1
        possible_weight = weight
        # 다리를 다 지난 트럭 인덱스 담는 리스트
        delete_queue_idx= []
        # 현재 다리에 올라와 있는 트럭이 다리를 다 지난경우가 있는지 확인
        for idx,x in enumerate(queue):
            # 아직 다 지나지 않았다면
            if x[1] < bridge_length:
                # 지나고 있는 다리 길이 1 이동
                x[1] += 1
                # 현재 다리위에 있기 때문에 가능한 무게에서 트럭무게 빼주기
                possible_weight -= x[0]
            # 다리를 다 지났다면 제거할 리스트에 추가
            else:
                delete_queue_idx.append(idx)
        # 다리 다 건넌 트럭 없애기
        while delete_queue_idx:
            queue.pop(delete_queue_idx.pop())
        #print(">>>",queue)
        # 이후 대기 트럭에 남은 트럭이 있다 & 트럭무게가 다리에 올라와도 괜찮다면 queue에 추가
        if len(truck_weights) != 0 and int(truck_weights[0]) <= possible_weight:
            new_truck = truck_weights.pop(0)
            queue.append([new_truck,1])

    return answer
```

