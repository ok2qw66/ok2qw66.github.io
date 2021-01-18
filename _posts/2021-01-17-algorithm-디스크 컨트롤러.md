---
title: "디스크 컨트롤러 from 프로그래머스"
date : 2021-01-17 15:33:50 +0900
categories:
- 알고리즘
tags:
- 프로그래머스
toc: true
toc_sticky: true
---

<br>

## 문제 출처

프로그래머스 > 힙 > 디스크 컨트롤러

[https://programmers.co.kr/learn/courses/30/lessons/42627](https://programmers.co.kr/learn/courses/30/lessons/42627)

<br>

###### 문제 설명

하드디스크는 한 번에 하나의 작업만 수행할 수 있습니다. 디스크 컨트롤러를 구현하는 방법은 여러 가지가 있습니다. 가장 일반적인 방법은 요청이 들어온 순서대로 처리하는 것입니다.

예를들어

```
- 0ms 시점에 3ms가 소요되는 A작업 요청
- 1ms 시점에 9ms가 소요되는 B작업 요청
- 2ms 시점에 6ms가 소요되는 C작업 요청
```

와 같은 요청이 들어왔습니다. 이를 그림으로 표현하면 아래와 같습니다.
![test1](https://grepp-programmers.s3.amazonaws.com/files/production/b68eb5cec6/38dc6a53-2d21-4c72-90ac-f059729c51d5.png)

한 번에 하나의 요청만을 수행할 수 있기 때문에 각각의 작업을 요청받은 순서대로 처리하면 다음과 같이 처리 됩니다.
![test2](https://grepp-programmers.s3.amazonaws.com/files/production/5e677b4646/90b91fde-cac4-42c1-98b8-8f8431c52dcf.png)

```
- A: 3ms 시점에 작업 완료 (요청에서 종료까지 : 3ms)
- B: 1ms부터 대기하다가, 3ms 시점에 작업을 시작해서 12ms 시점에 작업 완료(요청에서 종료까지 : 11ms)
- C: 2ms부터 대기하다가, 12ms 시점에 작업을 시작해서 18ms 시점에 작업 완료(요청에서 종료까지 : 16ms)
```

이 때 각 작업의 요청부터 종료까지 걸린 시간의 평균은 10ms(= (3 + 11 + 16) / 3)가 됩니다.

하지만 A → C → B 순서대로 처리하면
![test3](https://grepp-programmers.s3.amazonaws.com/files/production/9eb7c5a6f1/a6cff04d-86bb-4b5b-98bf-6359158940ac.png)

```
- A: 3ms 시점에 작업 완료(요청에서 종료까지 : 3ms)
- C: 2ms부터 대기하다가, 3ms 시점에 작업을 시작해서 9ms 시점에 작업 완료(요청에서 종료까지 : 7ms)
- B: 1ms부터 대기하다가, 9ms 시점에 작업을 시작해서 18ms 시점에 작업 완료(요청에서 종료까지 : 17ms)
```

이렇게 A → C → B의 순서로 처리하면 각 작업의 요청부터 종료까지 걸린 시간의 평균은 9ms(= (3 + 7 + 17) / 3)가 됩니다.

각 작업에 대해 [작업이 요청되는 시점, 작업의 소요시간]을 담은 2차원 배열 jobs가 매개변수로 주어질 때, 작업의 요청부터 종료까지 걸린 시간의 평균을 가장 줄이는 방법으로 처리하면 평균이 얼마가 되는지 return 하도록 solution 함수를 작성해주세요. (단, 소수점 이하의 수는 버립니다)

##### 제한 사항

- jobs의 길이는 1 이상 500 이하입니다.
- jobs의 각 행은 하나의 작업에 대한 [작업이 요청되는 시점, 작업의 소요시간] 입니다.
- 각 작업에 대해 작업이 요청되는 시간은 0 이상 1,000 이하입니다.
- 각 작업에 대해 작업의 소요시간은 1 이상 1,000 이하입니다.
- 하드디스크가 작업을 수행하고 있지 않을 때에는 먼저 요청이 들어온 작업부터 처리합니다.

##### 입출력 예

| jobs                     | return |
| ------------------------ | ------ |
| [[0, 3], [1, 9], [2, 6]] | 9      |

##### 입출력 예 설명

문제에 주어진 예와 같습니다.

- 0ms 시점에 3ms 걸리는 작업 요청이 들어옵니다.
- 1ms 시점에 9ms 걸리는 작업 요청이 들어옵니다.
- 2ms 시점에 6ms 걸리는 작업 요청이 들어옵니다.

<br>

---

### 완성 코드

```python
import heapq

def solution(jobs):
    # 총 대기시간의 합
    total = 0
    # jobs의 길이
    length = len(jobs)

    # jobs를 heapq 로 만든다 ==> 요청시간 순으로 정렬된 상태
    heapq.heapify(jobs)
    temp = heapq.heappop(jobs)

		# heapq에 [처리시간, 요청시간] 형태로 넣는다
    h = [[temp[1],temp[0]]]
    # 시작시간은 맨처음 작업의 요청시간으로 세팅
    # 맨처음 작업은 대기시간 0 이기 때문
    count = temp[0]
    
    
    while h or jobs:
        # 힙큐 h에서 제일 처리시간이 짧은 것을 가져와서 처리한다
        last, start = heapq.heappop(h)
        # 대기시간(count -start) + 처리시간(last)의 합을 total에 넣는다
        total += count - start + last
        # 시작시간은 처리(last)후의 시간으로 수정
        count += last
        
        # 아직 처리할 일들이 남아있다면...
        while jobs:
            unknown = heapq.heappop(jobs)
            # 현재 시작시간(count) 보다 작업요청시간이 빠르면 h에 넣는다
            # 즉, 이미 작업요청이 들어온 상태
            if unknown[0] <= count:
                heapq.heappush(h,[unknown[1],unknown[0]])
            # 현재 시작시간보다 뒤에 요청이 들어오지만 h에 아무것도 없다
            # 즉, 지금 할 일이 없다는 의미이므로
            # 현재 시작시간(count)를 unknown의 요청시간과 맞추고 값을 h에 넣는다
            # break를 하지 않는 이유??? 
            # jobs에 아직 unknown과 요청시간이 같은 게 있을수 있기 때문
            elif not h:
                heapq.heappush(h,[unknown[1],unknown[0]])
                count = unknown[0]
            # 그외라면 아직 시작요청이 들어오지 않은 상태이므로 다시 jobs에 넣는다
            else:
                heapq.heappush(jobs,unknown)
                break

    return int(total/length)
```

