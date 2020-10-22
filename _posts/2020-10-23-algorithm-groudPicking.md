---
title:  "땅따먹기 from 프로그래머스 "
date:   2020-10-23 02:25:50 +0900
categories: 
  - 알고리즘
tags:
  - 프로그래머스
toc: true
toc_sticky: true
---

<br>
출처 : https://programmers.co.kr/learn/courses/30/lessons/12913
<br>
<br>

## 땅따먹기

###### 문제 설명

땅따먹기 게임을 하려고 합니다. 땅따먹기 게임의 땅(land)은 총 N행 4열로 이루어져 있고, 모든 칸에는 점수가 쓰여 있습니다. 1행부터 땅을 밟으며 한 행씩 내려올 때, 각 행의 4칸 중 한 칸만 밟으면서 내려와야 합니다. **단, 땅따먹기 게임에는 한 행씩 내려올 때, 같은 열을 연속해서 밟을 수 없는 특수 규칙이 있습니다.**

예를 들면,

| 1 | 2 | 3 | 5 |

| 5 | 6 | 7 | 8 |

| 4 | 3 | 2 | 1 |

로 땅이 주어졌다면, 1행에서 네번째 칸 (5)를 밟았으면, 2행의 네번째 칸 (8)은 밟을 수 없습니다.

마지막 행까지 모두 내려왔을 때, 얻을 수 있는 점수의 최대값을 return하는 solution 함수를 완성해 주세요. 위 예의 경우, 1행의 네번째 칸 (5), 2행의 세번째 칸 (7), 3행의 첫번째 칸 (4) 땅을 밟아 16점이 최고점이 되므로 16을 return 하면 됩니다.

##### 제한사항

- 행의 개수 N : 100,000 이하의 자연수
- 열의 개수는 4개이고, 땅(land)은 2차원 배열로 주어집니다.
- 점수 : 100 이하의 자연수

##### 입출력 예

| land                            | answer |
| ------------------------------- | ------ |
| [[1,2,3,5],[5,6,7,8],[4,3,2,1]] | 16     |

<br>

## 코드

<br>

```python
def solution(land):
    answer = 0

    # depth : 행의 인덱스
    for depth in range(1,len(land)):
        # n_idx = 현재값의 인덱스
        for n_idx in range(4):
            # 합의 최댓값 넣을 변수
            temp = 0
            # p_idx = 이전값의 인덱스
            for p_idx in range(4):
                # 현재와 이전 인덱스가 다른 경우에서 최댓값 찾기
                if p_idx != n_idx:
                    temp = max(temp,land[depth-1][p_idx])
            # 해당 최댓값 추가해서 누적시키기
            land[depth][n_idx] += temp
            #print(land[depth],depth)
	
    #마지막 행에서의 최댓값 출력
    return max(land[len(land)-1])
```

<br>

```python
def solution(land):
	# depth: 행의 인덱스
    for depth in range(1, len(land)):
        # idx: 현재 값의 인덱스
        for idx in range(len(land[0])):
            # 현재 인덱스와 동일한 인덱스를 제외한 배열에서 최대값
            # max(land[depth -1][: idx] + land[depth - 1][idx + 1:])
            # 현재 인덱스의 값을 더해 누적시킨다
            land[depth][idx] = max(land[depth -1][: idx] + land[depth - 1][idx + 1:]) + land[depth][idx]

    # 마지막 행에서의 최댓값 출력
    return max(land[-1])
```

