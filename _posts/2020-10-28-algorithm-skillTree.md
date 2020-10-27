---
title:  "스킬 트리 from 프로그래머스 "
date:   2020-10-28 00:25:50 +0900
categories: 
  - 알고리즘
tags:
  - 프로그래머스
toc: true
toc_sticky: true
---

<br>
출처 : https://programmers.co.kr/learn/courses/30/lessons/49993
<br>
<br>

## 스킬 트리

선행 스킬이란 어떤 스킬을 배우기 전에 먼저 배워야 하는 스킬을 뜻합니다.

예를 들어 선행 스킬 순서가 `스파크 → 라이트닝 볼트 → 썬더`일때, 썬더를 배우려면 먼저 라이트닝 볼트를 배워야 하고, 라이트닝 볼트를 배우려면 먼저 스파크를 배워야 합니다.

위 순서에 없는 다른 스킬(힐링 등)은 순서에 상관없이 배울 수 있습니다. 따라서 `스파크 → 힐링 → 라이트닝 볼트 → 썬더`와 같은 스킬트리는 가능하지만, `썬더 → 스파크`나 `라이트닝 볼트 → 스파크 → 힐링 → 썬더`와 같은 스킬트리는 불가능합니다.

선행 스킬 순서 skill과 유저들이 만든 스킬트리[1](https://programmers.co.kr/learn/courses/30/lessons/49993#fn1)를 담은 배열 skill_trees가 매개변수로 주어질 때, 가능한 스킬트리 개수를 return 하는 solution 함수를 작성해주세요.

##### 제한 조건

- 스킬은 알파벳 대문자로 표기하며, 모든 문자열은 알파벳 대문자로만 이루어져 있습니다.
- 스킬 순서와 스킬트리는 문자열로 표기합니다.
  - 예를 들어, `C → B → D` 라면 CBD로 표기합니다
- 선행 스킬 순서 skill의 길이는 1 이상 26 이하이며, 스킬은 중복해 주어지지 않습니다.
- skill_trees는 길이 1 이상 20 이하인 배열입니다.
- skill_trees의 원소는 스킬을 나타내는 문자열입니다.
  - skill_trees의 원소는 길이가 2 이상 26 이하인 문자열이며, 스킬이 중복해 주어지지 않습니다.

##### 입출력 예

| skill   | skill_trees                         | return |
| ------- | ----------------------------------- | ------ |
| `"CBD"` | `["BACDE", "CBADF", "AECB", "BDA"]` | 2      |

##### 입출력 예 설명

- BACDE: B 스킬을 배우기 전에 C 스킬을 먼저 배워야 합니다. 불가능한 스킬트립니다.
- CBADF: 가능한 스킬트리입니다.
- AECB: 가능한 스킬트리입니다.
- BDA: B 스킬을 배우기 전에 C 스킬을 먼저 배워야 합니다. 불가능한 스킬트리입니다.

---

<br>

### 코드

```python
def solution(skill, skill_trees):
    answer = 0
	
    for x in skill_trees:
		# 처음 값을 미리 세팅
        temp = x.find(skill[0])
		# true 라면 처음부터 끝까지 다 확인한 상태
		# false라면 중간에 걸려서 중단된 상태
        complete = True

        for y in skill[1:]:
			# 앞에 스킬 미학습 상태라면 뒷 스킬도 미학습이어야 한다
			# 앞 스킬 & 뒷 스킬 모두 미학습상태이므로 패스
            if temp == -1 and x.find(y) == -1:
                continue
			# 앞 스킬 학습 & (뒷스킬 미학습 이거나 앞스킬배운후에 뒷스킬 학습)
            elif temp != -1 and (x.find(y) == -1 or temp <= x.find(y)):
                temp = x.find(y)
            # 위 조건을 만족하지 못하면 중단하고 다음 리스트로 넘어감
			else:
                complete = False
                break
				
		# complete를 완료한 경우만 답에 포함
        if complete:
            #print(x)
            answer += 1

    return answer
```

<br>

### 다른 사람의 간단한 풀이

```python
def solution(skill,skill_tree):
    answer=0
    for i in skill_tree:
        skill_list=''
        for z in i:
            # 만약에 스킬이 존재한다면 순서대로 skill_list 만들기
            if z in skill:
                skill_list+=z
        # skill_list 과 동일하다면 답 증가
        if skill_list==skill[0:len(skillist)]:
            answer+=1
    return answer
```

