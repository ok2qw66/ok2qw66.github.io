---
title:  " 알고리즘 특강 정리 "
date:   2020-11-01 00:25:50 +0900
categories: 
  - 알고리즘
tags:
  - 특강
toc: true
toc_sticky: true
---

<br>

## 자료구조

### List

#### Linear List

```python
# 선형리스트 구현
katok = ['1','2','3']


def add_data(position,friend):
    katok.append(None) # 빈칸 추가
    kLen = len(katok)

    for i in range(kLen-1,position,-1):
        katok[i] = katok[i-1]
        katok[i-1] = None

    katok[position] = friend


def delete_data(postion):
    katok[postion] = None
    kLen = len(katok)
    for i in range(postion,kLen-1):
        katok[i] = katok[i+1]
    # del (katok[kLen-1])
    katok.pop()


# add_data(1,'a')
# add_data(2,'b')
# add_data(3,'c')
# print(katok)
# delete_data(3)
# print(katok)
```

<br>

```python
def printPoly(p_x):
    term = len(p_x)-1
    polyStr = "P(x)= "
    for i in range(len(p_x)):
        coef = p_x[i]

        if coef >=0:
            polyStr += '+'
        polyStr += str(coef) + 'x^'+str(term)+" "
        term -=1
    return polyStr


def calcPoly(xValue,p_x):
    pLen = len(p_x)-1
    answer = 0
    for x in p_x:
        answer += x * xValue ** pLen
        # for i in range(pLen):
        #     temp *=xValue
        #print(temp)
        #answer +=temp
        pLen -=1
    return answer


## 전역 변수부
px = [4,-3,0,2]

if __name__ == '__main__':
    # print(printPoly(px))
    # print(calcPoly(5,px))
```

<br>

### Node List

```python
## 함수 선언부
class Node() :
    def __init__ (self) :
        self.data = None
        self.link = None

def printNode(head) :
    current = head
    print(current.data, end = '  ')
    while current.link != None :
        current = current.link
        print(current.data, end='  ')
    print()

def insert_node(findData, insertData) :
    global memory, head, pre, current

    if findData == head.data : # 첫노드가 찾는 값일때
        node = Node()
        node.data = insertData
        node.link = head
        head = node
        return
    # 두번째 노드 이후일때....
    current = head
    while current.link != None :
        pre = current
        current = current.link
        if current.data == findData :
            node = Node()
            node.data = insertData
            node.link = current
            pre.link = node
            return
    # 마지막까지 못찾을 때.. (=마지막에 삽입)
    node = Node()
    node.data = insertData
    current.link = node

def delete_node(deleteData) :
    global memory, head, pre, current
    if deleteData == head.data:  # 첫노드가 찾는 값일때
        current = head
        head = head.link
        del(current)
        return
    # 두번째 노드 이후를 삭제
    current = head
    while current.link != None:
        pre = current
        current = current.link
        if current.data == deleteData:
            pre.link = current.link
            del(current)
            return

def find_node(findData) :
    pass # 찾으면 True, 못찾으면 False  리턴

## 전역 변수부
memory = []
head, pre, current = None, None, None
dataAry = ['다현', '정연', '쯔위', '사나', '지효']

## 메인 코드부
if __name__ == '__main__' :
    # 첫번째 노드
    node = Node()
    node.data = dataAry[0]
    head = node # 헤드 지정 (*중요)
    memory.append(node)

    # 두번째 노드 부터.....
    for data in dataAry[1:] :
        pre = node # 이전노드 기억(*중요)
        node = Node()
        node.data = data
        pre.link = node
        memory.append(node)

    printNode(head)

    insert_node('다현', '화사');    printNode(head)
    insert_node('사나', '솔라');    printNode(head)
    insert_node('마동석', '문별');    printNode(head)

    delete_node('화사'); printNode(head)
    delete_node('솔라'); printNode(head)
```

<br>

### Stack

- 프링글스 통 안에 과자
- LIFO (Last In First Out) : 마지막에 들어간 값이 처음에 나온다

### Queue

- 리스트와 유사
- 일반적인 줄서기
- FIFO (First In First Out) : 처음에 들어간 값이 처음에 나온다

```python
# 큐의 초기화
queue = [None, None, None]
front = rear = -1

rear += 1
queue[rear] = 'A'

rear += 1
queue[rear] = 'B'

front += 1
data = queue[front]
queue[front] = None
print(data)

rear += 1
queue[rear] = 'C'

front += 1
data = queue[front]
queue[front] = None
print(data)

front += 1
data = queue[front]
queue[front] = None
print(data)
```

<br>

#### Circular Queue

```python
##함수 선언부
def isQueueFull() : # T / F
    global SIZE, queue, front, rear
    if ( (rear + 1) % SIZE == front ) :
        return  True
    else :
        return  False

def enQueue(data) :
    global SIZE, queue, front, rear
    if (isQueueFull()) :
        print('큐가 꽉참')
        return
    rear = (rear + 1) % SIZE
    queue[rear] = data

def isQueueEmpty() :
    global SIZE, queue, front, rear
    if (front == rear) :
        return  True
    else :
        return  False

def deQueue() :
    global SIZE, queue, front, rear
    if (isQueueEmpty()) :
        print('큐가 비었음')
        return
    front = (front + 1) % SIZE
    data = queue[front]
    queue[front] = None
    return data

## 전역 변수부
SIZE = 5
queue = [ None for _ in range(SIZE)]
front=rear = 0

## 메인 코드부
enQueue('a');enQueue('b');enQueue('c');enQueue('d')
enQueue('e')
print(queue)
```

<br>

---

<br>

## 정렬

```python
import random
## 함수 선언부
def findMinIdx(ary) :
    retIdx = 0
    for i in range(len(ary)) :
        if (ary[retIdx] > ary[i]) :
            retIdx = i
    return retIdx

## 전역 변수부
before = [ random.randint(10, 99) for _ in range(20) ]
after = []
## 메인 코드부
print('정렬전-->',  before)
for i in range(len(before)) :
    minIdx = findMinIdx(before)
    after.append(before[minIdx])
    del(before[minIdx])
print('정렬후-->',  after)
```

<br>

```python
import random
## 함수 선언부


def insertionSort(ary) :
    def findMinIdx(ary):
        retIdx = 0
        for i in range(len(ary)):
            if (ary[retIdx] < ary[i]):
                retIdx = i
        return retIdx
    newAry = []
    for i in range(len(ary)) :
        minIdx = findMinIdx(ary)
        newAry.append(ary[minIdx])
        del(ary[minIdx])
    return  newAry

## 전역 변수부
before = [ random.randint(10, 99) for _ in range(20) ]
after = []
## 메인 코드부
print('정렬전-->',  before)
after = insertionSort(before) # 내림차순
print('정렬후-->',  after)
```

<br>

---

<br>

## 검색

### 순차검색

```python
import random

# 중복된 데이터가 없다고 가정
# (1) 1개만 찾는 값이 있다
# (2) 1개만 찾으면 된다.
def seqSearch(ary, data) :
    pos = -1
    for i in range(len(ary)) :
        if (ary[i] == data) :
            pos = i
            break
    return  pos

def seqSearchMulti(ary, data) :
    pAry = []
    for i in range(len(ary)) :
        if (ary[i] == data) :
            pAry.append(i)
    return  pAry

dataAry = [ random.randint(11,50)  for _ in range(50)]
print(dataAry)

position = seqSearch(dataAry, 50)
print(position)

posAry = seqSearchMulti(dataAry, 50)
print(posAry)
```

<br>

### 이진검색

```python
import random

def binSearch(ary, data) :
    global count
    pos = -1
    start = 0
    end = len(ary) -1
    while (start <= end) :
        count += 1
        mid = (start + end ) // 2
        if (ary[mid] == data) :
            return mid
        elif (ary[mid] < data) :
            start = mid + 1
        else :
            end = mid -1
    return pos

count = 0
dataAry = [ random.randint(100000, 999999) for _ in range(1000000)]
dataAry.sort()

position = binSearch(dataAry, 111111)
print(position, dataAry[position], ' 횟수-->', count)

#print(dataAry[0:10], dataAry[-10:-1])
```

