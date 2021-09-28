---
title: 알고리즘 STEP 1 Stack/Queue 1 주식 가격
tags: encore algorithm
---

* [코딩테스트 연습>스택/큐>주식가격](https://programmers.co.kr/learn/courses/30/lessons/42584)

>### 주식가격


>#### 문제 설명
>
>초 단위로 기록된 주식가격이 담긴 배열 prices가 매개변수로 주어질 때, 가격이 떨어지지 않은 기간은 몇 초인지를 return 하도록 solution 함수를 완성하세요.

>#### 제한사항
>
>prices의 각 가격은 1 이상 10,000 이하인 자연수입니다.
>
>prices의 길이는 2 이상 100,000 이하입니다.
>

>#### 입출력 예
>
>prices	return
>
>[1, 2, 3, 2, 3]	[4, 3, 1, 1, 0]
>
>#### 입출력 예 설명
>
>1초 시점의 ₩1은 끝까지 가격이 떨어지지 않았습니다.
>
>2초 시점의 ₩2은 끝까지 가격이 떨어지지 않았습니다.
>
>3초 시점의 ₩3은 1초뒤에 가격이 떨어집니다. 따라서 1초간 가격이 떨어지지 않은 것으로 봅니다.
>
>4초 시점의 ₩2은 1초간 가격이 떨어지지 않았습니다.
>
>5초 시점의 ₩3은 0초간 가격이 떨어지지 않았습니다.
>
>※ 공지 - 2019년 2월 28일 지문이 리뉴얼되었습니다.

```python
def solution(prices):
    answer = []
    priceStack = [(0, prices[0])] # 입력된 배열값의 첫 값을 대입
                                  # 2번째 값을 입력하면서 첫번째 값과의 비교 및 stack 으로 옮기는 작업 진행

    answerStack = dict()          # 원 배열의 위치값을 key 값으로, pop 되는 순간, 비교값의 위치와의 차이(시간차) 를 value 로
    curPrice = prices[0]

    for idx, nextPrice in enumerate(prices[1:]):    # 두번째 값부터 비교를 위한 반복문 진행 (idx 는 해당 값의 위치를 표시)
        nextIdx = idx + 1                           # idx 값이 0 부터 시작하므로 이를 맞추기 위해 1을 더함

                                                    # 기준 : stack 의 peek 값이 current value, 반복문에서 주어진 값이 next value
        if curPrice <= nextPrice:                   # 현재 stack 상단의 값보다 next 값이 크다면
            priceStack.append((nextIdx, nextPrice)) # next 값을 stack 에 쌓음
            curPrice = nextPrice                    

        else:                                       # 현재 stack 상단의 값보다 next 값이 작다면
            curIdx, curPrice=priceStack.pop()       # top 값을 꺼냄
            answerStack[curIdx] = nextIdx - curIdx  # 꺼낸값과 next 값과의 시간차를 계산하여 dictionary value 로 입력

            while len(priceStack) > 0 and priceStack[-1][1] > nextPrice: # 혹시 stack 이 빈상태가 아닌지, 새로운 top 값을 꺼내기전, 다시 비교 후
                curIdx, curPrice=priceStack.pop()   # 위 결과가 모두 참이라면 반복하여 top 값을 꺼내고
                answerStack[curIdx] = nextIdx - curIdx # 꺼낸값과 next 값과의 시간차를 계산하여 dictionary value 로 입력

            priceStack.append((nextIdx, nextPrice)) # 더이상 꺼낼 값이 없다면 현재 next value 를 stack 에 쌓음.
            curPrice = nextPrice

    while len(priceStack) > 0:                      # 반복문은 완료하였으나, 아직 stack 에 남아 있는 값들이 있을수 있으므로
        curIdx, curPrice=priceStack.pop()           # 남은 값들을 해당 stack 의 길이가 0 이 될때까지 꺼내서 dictionary 에 저장
        answerStack[curIdx] = nextIdx - curIdx

    for idx, count in sorted(answerStack.items()):  # 모든 값이 저장된 dictionary 를 key 값(기존 배열의 순서) 로 정렬하여
        answer.append(count)                        # value 를 list 에 순서대로 저장

    return answer                                   # list 를 return 값으로 보냄
```
해당 함수 실행 시,

```python
solution([50, 10, 20, 30, 20, 30])
```
[1, 4, 3, 1, 1, 0]

### Error Comment

* 논리구조를 파악하기 가장 어려웠던 부분은 else 문(기준값이 비교값보다 커져서 stack 에서 값을 꺼내야 할 경우) 였음.


  - 해당 부분을 우선 피해서 그다음 부분들을 완성하기 위하여 계속적으로 값이 증가하는 배열 [1,2,3,4,5] 를 입력해서 완성시킨 이후, else 부분을 다듬고자 함.


  - 그러나 else 내부 논리구조가 계속 정리되지 않아 아래 방법으로 다시 code 를 작성함.


* 배열의 첫 값을 for 문 이전에 미리 넣은 이유:


  - 첫 값부터 바로 for 문에 넣어도 문제가 없을 듯 하지만, 초반에 그렇게 code 를 작성했을 때, 기준값/비교값의 개념을 정리하는데 개인적으로 혼동이 많이되어 논리 구조를 파악하기 어려웠음. 그래서 기준값을 stack 의 최상단 값, 비교값을 그 다음 값으로 개념을 잡다 보니 이와 같이 구성되었음.


* 기본 예시 배열 [1, 2, 3, 2, 3] 은 성공하였으나, 사이트에서 test 시 대부분 실패로 뜸.


  - 원인을 파악하고자 다른 형태의 배열 값을 넣어봄.


  - 운좋게 일찍 그 원인을 찾을 수 있었음.


  - [5, 1, 2...] 와 같이 첫 값이 큰 경우, else 문으로 진행되면서 첫 값 이전 값을 비교 및 pop 을 진행해야 하는데 첫값 이전값은 없으므로 stack 이 비어 있는지를 확인하는 과정이 필요함을 확인함. 이에 대해 while 문에 논리판단를 추가.


* error 는 모든 막았으나. [5, 1, 2, 3, 2, 3] 을 넣으면 [1, **1** , 3, 1, 1, 0] 로 두번째 값 다르게 나옴.


  - else 문 가장 마지막에 `curPrice = nextPrice` 을 빠트린것을 찾음.  


* dictionary 를 사용한 이유


  - 최종 stack 에서 꺼낸값을 정렬할 때 반복문을 쓰지 않고자 dictionary 를 사용했으나, 정렬된 dictionary 값이 tuple 로 나오는데 이를 list 로 변환하는 최적의 방법을 찾지 못해 결국 반복문을 다시 썼음.
