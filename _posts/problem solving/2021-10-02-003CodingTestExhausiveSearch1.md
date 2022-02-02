---
title: 알고리즘 STEP 2 완전탐색/이분탐색 1 모의 고사
date: 2021-10-02
category: Problem Solving
tags : [Algorithm, Stack, ExhuasiveSearch]
layout: post 
---

* 링크: [코딩테스트 연습>완전탐색>모의고사](https://programmers.co.kr/learn/courses/30/lessons/42840)

>### 모의고사


>#### 문제 설명
>
>수포자는 수학을 포기한 사람의 준말입니다. 수포자 삼인방은 모의고사에 수학 문제를 전부 찍으려 합니다. 수포자는 1번 문제부터 마지막 문제까지 다음과 같이 찍습니다.
>
>1번 수포자가 찍는 방식: 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, ...
>
>2번 수포자가 찍는 방식: 2, 1, 2, 3, 2, 4, 2, 5, 2, 1, 2, 3, 2, 4, 2, 5, ...
>
>3번 수포자가 찍는 방식: 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, ...

>1번 문제부터 마지막 문제까지의 정답이 순서대로 들은 배열 answers가 주어졌을 때, 가장 많은 문제를 맞힌 사람이 누구인지 배열에 담아 return 하도록 solution 함수를 작성해주세요.

>#### 제한사항
> * 시험은 최대 10,000 문제로 구성되어있습니다.
> * 문제의 정답은 1, 2, 3, 4, 5중 하나입니다.
> * 가장 높은 점수를 받은 사람이 여럿일 경우, return하는 값을 오름차순 정렬해주세요.

>#### 입출력 예

|answers|	return|
|---------------|-------|
|[1,2,3,4,5]| [1]|
|[1,3,2,4,2]|	[1,2,3]|


### Code
```python
def solution(answer):
    def p1(idx, answ):
        if (idx%5) +1 == answ:
            return True
        else:
            return False

    def p2(idx, answ):
        p2Pattern = [2,1,2,3,2,4,2,5]
        if p2Pattern[idx%8] == answ:
            return True
        else:
            return False

    def p3(idx, answ):
        p3Pattern = [3,3,1,1,2,2,4,4,5,5]
        if p3Pattern[idx%10] == answ:
            return True
        else:
            return False

    scores = [0, 0, 0]
    winner =[]

    for idx, answ in enumerate(answer):
        if p1(idx, answ):
            scores[0] += 1
        if p2(idx, answ):
            scores[1] += 1
        if p3(idx, answ):
            scores[2] += 1

    for idx, score in enumerate(scores):
        if max(scores) == score:
            winner.append(idx+1)

    return winner

```

### Error Comments

  * 일전에 파이선 수업 시간에 풀었던 문제였으며, 상대적으로 알고리즘 자체는 복잡하지 않아 문제를 푸는데 큰 어려움은 없었음.

  * 다만 최종 각 패턴 별 정답 갯수를 산출한 이후, 해당 값을 어떻게 효율적으로 정렬 및 출력할 것인가에 대해 기존에 불었던 내용을 참조하였음.
    - enumerate 를 사용하여 배열의 각 값에 대한 index 반영후, `max(scores)` 와 각 배열값을 비교하여 winner 를 결정함.
