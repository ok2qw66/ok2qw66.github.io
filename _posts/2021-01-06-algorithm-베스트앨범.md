---
title:  "베스트앨범 from 프로그래머스"
date:   2021-01-06 23:52:50 +0900
categories: 
  - 알고리즘
tags:
  - 프로그래머스
toc: true
toc_sticky: true
---

<br>

## 문제 출처

프로그래머스 > 해시 > 베스트앨범
[https://programmers.co.kr/learn/courses/30/lessons/42579](https://programmers.co.kr/learn/courses/30/lessons/42579)
<br>

## 문제

###### 문제 설명

스트리밍 사이트에서 장르 별로 가장 많이 재생된 노래를 두 개씩 모아 베스트 앨범을 출시하려 합니다. 노래는 고유 번호로 구분하며, 노래를 수록하는 기준은 다음과 같습니다.

1. 속한 노래가 많이 재생된 장르를 먼저 수록합니다.
2. 장르 내에서 많이 재생된 노래를 먼저 수록합니다.
3. 장르 내에서 재생 횟수가 같은 노래 중에서는 고유 번호가 낮은 노래를 먼저 수록합니다.

노래의 장르를 나타내는 문자열 배열 genres와 노래별 재생 횟수를 나타내는 정수 배열 plays가 주어질 때, 베스트 앨범에 들어갈 노래의 고유 번호를 순서대로 return 하도록 solution 함수를 완성하세요.

##### 제한사항

- genres[i]는 고유번호가 i인 노래의 장르입니다.
- plays[i]는 고유번호가 i인 노래가 재생된 횟수입니다.
- genres와 plays의 길이는 같으며, 이는 1 이상 10,000 이하입니다.
- 장르 종류는 100개 미만입니다.
- 장르에 속한 곡이 하나라면, 하나의 곡만 선택합니다.
- 모든 장르는 재생된 횟수가 다릅니다.

##### 입출력 예

| genres                                | plays                      | return       |
| ------------------------------------- | -------------------------- | ------------ |
| [classic, pop, classic, classic, pop] | [500, 600, 150, 800, 2500] | [4, 1, 3, 0] |

##### 입출력 예 설명

classic 장르는 1,450회 재생되었으며, classic 노래는 다음과 같습니다.

- 고유 번호 3: 800회 재생
- 고유 번호 0: 500회 재생
- 고유 번호 2: 150회 재생

pop 장르는 3,100회 재생되었으며, pop 노래는 다음과 같습니다.

- 고유 번호 4: 2,500회 재생
- 고유 번호 1: 600회 재생

따라서 pop 장르의 [4, 1]번 노래를 먼저, classic 장르의 [3, 0]번 노래를 그다음에 수록합니다.

※ 공지 - 2019년 2월 28일 테스트케이스가 추가되었습니다.

<br>

---

### 완성 코드

```python
def solution(genres, plays):
    answer = []
    genre = set(genres)
    totalPlay = {}
    
    for g in genre:
        totalPlay[g] = [0,[-1,-1]]
    
    
    for i in range(len(genres)):
        
        totalPlay[genres[i]][0] += plays[i]
        
        first = totalPlay[genres[i]][1][0]
        second = totalPlay[genres[i]][1][1]
        # first 값 비교
        if first == -1:
            totalPlay[genres[i]][1][0] = i
        elif plays[i] > plays[first] or (plays[i] == plays[first] and i < first):
            totalPlay[genres[i]][1][0] = i
            totalPlay[genres[i]][1][1] = first
        elif plays[i] == plays[first] and i > first:
            totalPlay[genres[i]][1][1] = i
        # second 값 비교        
        elif second == -1 or plays[i] > plays[second]:
            totalPlay[genres[i]][1][1] = i
        elif plays[i] == plays[second] and i < second:
            totalPlay[genres[i]][1][1] = second
            
    # play 총합 내림차순으로 정렬        
    sorting = sorted(totalPlay.values(),key=lambda x : -x[0])
    
    for x in sorting:
        a,b = x[1]
        answer.append(a)
        if b != -1:
            answer.append(b)
        
    return answer
```

첫번째는 for문을 3번 돌려서 코드를 만들었다.

![image-20210107001733155](https://user-images.githubusercontent.com/69428620/103785287-607f7800-507e-11eb-87f7-20244343be3c.png)

간단하게 과정을 설명하자면....<br>

1. genre 이름을 key로 가진 dict 생성 (play total값, 첫번째/두번째로 큰 play 가진 idx 넣을 모양 세팅)
2. 순서대로 genre별로 play total값, 첫번째/두번째로 큰 play 가진 idx 업데이트
3. play total값으로 내림차순 정렬한 후 최대 2개씩 값을 answer에 저장

<br>

##### 첫번째 for문

``for g in genre:
        totalPlay[g] = [0,[-1,-1]]``<br>

처음값을 세팅해주었다.  genre이름으로 dictionary를 만들어서

[play total값, [첫번째로 큰 play 가진 idx, 두번째로 큰 play 가진 idx]] 이렇게 넣도록 하였다.

 <br>

##### 두번째 for문

``for i in range(len(genres)):``<br>

우선 해당 genre에 total값을 증가시켜주고, 첫번째/두번째로 큰 play 값을 비교해서 idx값을 업데이트 시켜준다

(작성한 코드가 if연속으로 되어 있어서 가독성이 좋지 않다...)

<br>

##### 세번째 for문

``for x in sorting:
        a,b = x[1]
        answer.append(a)
        if b != -1:
            answer.append(b)``<br>

두번째 for문이 끝나고 play total 값 오름차순으로 정렬한 상태에서 첫번째/두번째로 큰 play 값 배열을 불러온다. 이때, -1로 되어있다면 값이 없는 것이기 때문에 pass 한다

<br>

---

### 두번째 완성 코드

```python
def solution(genres, plays):
    answer = []
    total_sort = []
    
    for i in range(len(genres)):
        for x in total_sort:
            if x[0] == genres[i]:
                x[1] += plays[i]
                x[2].append([plays[i],i])
                break
        else:
            total_sort.append([genres[i],plays[i],[[plays[i],i]]])
                
    total_sort = sorted(total_sort,key=lambda x: -x[1])
    
    for x in total_sort:
        sorting = sorted(x[2],key=lambda x : (-x[0],x[1]))
        print(sorting)
        answer.append(sorting[0][1])
    
        if len(sorting) >= 2:
            answer.append(sorting[1][1])
    
    return answer
```

두번째 작성한 코드는 이중 for문 + for문 으로 구성되어 있다.

이중 for문이다보니 첫번째 코드보단 조금 더 시간이 소요된다.

![image-20210107001613854](https://user-images.githubusercontent.com/69428620/103785292-62493b80-507e-11eb-8842-447423593933.png)

1. 순서대로 하나씩 원소를 가져와서 

   해당 genre가 존재하지 않는다면 새로 생성 / 기존에 있다면 업데이트

2. total_sort 값을 play 총합값으로 내림차순 정렬

3. paly 총합값으로 내림차순 정렬된 상태에서 play 큰순으로, idx 작은순으로 재정렬

   정렬된 값중 제일 앞 2개값을 answer에 추가 