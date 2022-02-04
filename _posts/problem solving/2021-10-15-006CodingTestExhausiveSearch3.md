---
title: (프로그래머스) 숫자카드2
date: 2021-10-15
category: Problem Solving
tags : [Algorithm, 프로그래머스, 이분탐색, 숫자카드2]
layout: post
---
* 링크: [코딩테스트 연습>이분탐색>숫자카드2](https://www.acmicpc.net/problem/10816)

### 문제
>숫자 카드는 정수 하나가 적혀져 있는 카드이다. 상근이는 숫자 카드 N개를 가지고 있다. 정수 M개가 주어졌을 때, 이 수가 적혀있는 숫자 카드를 상근이가 몇 개 가지고 있는지 구하는 프로그램을 작성하시오.

>입력
>첫째 줄에 상근이가 가지고 있는 숫자 카드의 개수 N(1 ≤ N ≤ 500,000)이 주어진다. 둘째 줄에는 숫자 카드에 적혀있는 정수가 주어진다. 숫자 카드에 적혀있는 수는 -10,000,000보다 크거나 같고, 10,000,000보다 작거나 같다.

>셋째 줄에는 M(1 ≤ M ≤ 500,000)이 주어진다. 넷째 줄에는 상근이가 몇 개 가지고 있는 숫자 카드인지 구해야 할 M개의 정수가 주어지며, 이 수는 공백으로 구분되어져 있다. 이 수도 -10,000,000보다 크거나 같고, 10,000,000보다 작거나 같다.

>출력
>첫째 줄에 입력으로 주어진 M개의 수에 대해서, 각 수가 적힌 숫자 카드를 상근이가 몇 개 가지고 있는지를 공백으로 구분해 출력한다.

>예제 입력 1
>10
>6 3 2 10 10 10 -10 -10 7 3
>8
>10 9 -5 2 3 4 5 -10

>예제 출력 1
>3 0 0 1 2 0 0 2


### 1차 시도 CODE (시간초과)
```python
from sys import stdin
ownedNum = stdin.readline()
ownedCards = sorted(map(int,stdin.readline().split()))
checkingNum = stdin.readline()
stdCardList = map(int,stdin.readline().split())

answer = str()

for x in stdCardList:
    answ = 0
    begin = 0
    length = len(ownedCards)
    middle = length//2
    end = length-1

    while end - begin > 1 and length > 0:
        if x == ownedCards[middle]:
            while middle < length and x == ownedCards[middle]:
                answ += 1
                ownedCards.pop(middle)
                length -= 1
            while middle >= 1 and x == ownedCards[middle-1]:
                answ += 1
                ownedCards.pop(middle-1)
                length -= 1
            answer += (str(answ) + " ")
            break

        elif x < ownedCards[middle]:
            end = middle
            middle = (end - begin)//2

        else:
            begin = middle
            middle =  begin + (end - begin)//2

        if end - begin == 1:
            if x == ownedCards[begin]:
                answ += 1
                ownedCards.pop(begin)
                length -= 1
            elif x == ownedCards[end]:
                answ += 1
                ownedCards.pop(end)
                length -= 1
            answer += (str(answ) + " ")

print(answer)
```

#### Error Comments

  * 이진 탐색을 사용하여 기본 알고리즘 구성


    - 내가 가지고 있는 숫자 카드를 정렬된 list 로 만들어서 이진 탐색으로 동일한 숫자가 위치한 곳을 찾아감.


    - 해당 장소에서 바로 앞, 또는 바로 뒤에 동일 숫자가 있는지를 확인해서 동일 숫자가 없을 때까지 주변지역을 계속 탐색 / pop() 을 실행

  * 문제점


    - 동일 숫자가 주변 영역에 있는지 탐색하는 기능을 구현하는데 code 의 간결성을 유지 하기 힘들며, 따라서 에러를 확인하기 어려워짐.


    - 결정적으로 Test 시 시간 초과가 발생. (해당 개념으로 더이상 처리 시간을 줄일 수 있는 방법을 찾지 못함.)


### 2차 CODE (성공)
```python
from sys import stdin
ownedNum = 10
ownedCards = map(int,"6 3 2 10 10 10 -10 -10 7 3".split())
checkingNum = 8
stdCardList = map(int, "10 9 -5 2 3 4 5 -10".split())

answer = str()
owned_dict = {}

for x in ownedCards:
    if x not in owned_dict:
        owned_dict[x] = 1
    else:
        owned_dict[x] += 1

for x in stdCardList:
    if x in owned_dict:
        answer += str(owned_dict[x])
    else:
        answer += str("0")
    answer += str(" ")

answer= answer[:-2]

print(answer)
```

#### Error Comments
  * 이진 탐색으로 적절한 방법을 찾을 수 없어서, dictionary 기능을 사용하여 기능을 구현함.


  * 내가 가진 카드 list 를 dictionary 로 만들어서 key 는 해당 카드의 숫자, value 는 동일 숫자카드의 갯수를 넣음.


  * 검색 번호 list 에서 동일 숫자가 나오면, 동일 번호에 대한 value 값을 string 에 넣고, 만약 해당 숫자가 없으면 0 을 넣음.
