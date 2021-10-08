---
title: 알고리즘 STEP 1 Stack/Queue 2 기능개발
tags: ENCORE Algorithm CodingTest Queue
---

* 링크: [코딩테스트 연습>스택/큐>기능개발](https://programmers.co.kr/learn/courses/30/lessons/42586)

>### 기능개발


>#### 문제 설명
>
>프로그래머스 팀에서는 기능 개선 작업을 수행 중입니다. 각 기능은 진도가 100%일 때 서비스에 반영할 수 있습니다.
>
>또, 각 기능의 개발속도는 모두 다르기 때문에 뒤에 있는 기능이 앞에 있는 기능보다 먼저 개발될 수 있고, 이때 뒤에 있는 기능은 앞에 있는 기능이 배포될 때 함께 배포됩니다.
>
>먼저 배포되어야 하는 순서대로 작업의 진도가 적힌 정수 배열 progresses와 각 작업의 개발 속도가 적힌 정수 배열 speeds가 주어질 때 각 배포마다 몇 개의 기능이 배포되는지를 return 하도록 solution 함수를 완성하세요.


>#### 제한사항
>
>작업의 개수(progresses, speeds배열의 길이)는 100개 이하입니다.
>
>작업 진도는 100 미만의 자연수입니다.
>
>작업 속도는 100 이하의 자연수입니다.
>
>배포는 하루에 한 번만 할 수 있으며, 하루의 끝에 이루어진다고 가정합니다. 예를 들어 진도율이 95%인 작업의 개발 속도가 하루에 4%라면 배포는 2일 뒤에 이루어집니다.
>

>#### 입출력 예
>
>       progresses	            speeds	        return
>     [93, 30, 55]	          [1, 30, 5]	        [2, 1]
>

>#### 입출력 예 설명
>
>입출력 예 #1
>
>첫 번째 기능은 93% 완료되어 있고 하루에 1%씩 작업이 가능하므로 7일간 작업 후 배포가 가능합니다.
>
>두 번째 기능은 30%가 완료되어 있고 하루에 30%씩 작업이 가능하므로 3일간 작업 후 배포가 가능합니다. 하지만 이전 첫 번째 기능이 아직 완성된 상태가 아니기 때문에 첫
번째 기능이 배포되는 7일째 배포됩니다.
>
>세 번째 기능은 55%가 완료되어 있고 하루에 5%씩 작업이 가능하므로 9일간 작업 후 배포가 가능합니다.
>
>따라서 7일째에 2개의 기능, 9일째에 1개의 기능이 배포됩니다.

### Code
```python
def solution(progresses, speeds):
    answer = []
                                                # 주어진 배열값을 queue 형식으로 계속 pop 할 예정이므로
    while len(progresses) > 0:                  # 그 길이가 0 이 될때까지 실행 (한번 실행시 하루가 소요됨을 가정)

        for order, speed in enumerate(speeds):  # 위치값을 표시하기 위해 enumerate 로 반복문 실행
            if progresses[order] < 100:         # progress 가 100 미만일때만 progress 가산 실행
                progresses[order] += speed

        count = 0
        while len(progresses) > 0 and progresses[0] >=100: # 앞에서 부터 progress 가 100 이상인 값을 꺼냄
            progresses.pop(0)                              # 한편 자체 while 실해 중 배열이 완전히 비워질수 있으므로
            speeds.pop(0)                                  # len(progresses) > 0 를 먼저 검사 후 실행
            count += 1                                     # pop 이 실행된 횟수를 기록하기 위해 count 증가값 추가

        if count > 0:                           # count 값이 발생시에 대해서만 return 될 배열값으로 추가
            answer.append(count)                           

    return answer                               
```
해당 함수 실행 시,

```python
solution([95, 90, 99, 99, 80, 99],[1, 1, 1, 1, 1, 1])
```
`[1, 3, 2]`

### Error Comments

* 초반에는 이전 문제와 같이 각각의 주어진 배열을 dictionary 넣고, progress 가 100 이상이 된 값들을 stack 에 저장하고자 하였으나, 이 방법으로는 답을 찾을 수 없었음.


* progress 100 이상의 경우에 대해  progress 가산을 멈추고 기다렸다가, 앞의 값들이 해당 조건을 충족시키면 같는 회차에 빠져 나가도록 coding 함. (queue 활용)

* 배열 앞에서부터 pop 을 실행하는 과정에서 자체 while 문에서 에러가 발생됨을 확인함.

  - len(progresses) > 0 조건을 추가하여 해당 error 해결

### 2차 풀이
```python
def solution(progresses, speeds):
    days = 1
    answer = []
    while len(progresses) > 0:
        counter = 0
        while len(progresses) > 0 and progresses[0] + speeds[0]* days >=100:
            progresses.pop(0)
            speeds.pop(0)
            counter += 1
        if counter > 0:
            answer.append(counter)
        days += 1

    return answer
```

### Error Comments

*  다시 문제를 보니 매일 progresses 값들에 굳이 값을 올려줄 필요가 없었다.


*  날짜수를 반복문을 통해 오려주면서, progresses[0] 값만 변경해서 100 을 넘는지 확인하고, True 일 경우 , pop 으로 꺼내서 다시 progresses[0] 의 값을 확인하는 반복문을 통해 이전보다 간단하게 풀이가 가능했음.

### 다른 사람 풀이 보기

```python
def solution(progresses, speeds):
Q=[]
for p, s in zip(progresses, speeds):
    if len(Q)==0 or Q[-1][0]<-((p-100)//s):
        Q.append([-((p-100)//s),1])
        # print(Q, "if")                 # code 이해를 위한 출력 1        
    else:
        Q[-1][1]+=1
        # print(Q, "else")               # code 이해를 위한 출력 2
return [q[1] for q in Q]                     
```

```python
solution([93, 30, 55], [1, 30, 5])
```
* 결과값

  - [[7, 1]] if
  - [[7, 2]] else
  - [[7, 2], [9, 1]] if
  - [2, 1]          (최종 정답)


* zip 함수를 통해 첫번째 progress 와 speed 값 (93, 1) 로 for 시작


* len(Q) == 0 이므로 if 문 진행
  - 남은 progress//speed 로 작업에 걸리는 일수- 1 값 (7)을 구함
  - 최초 진행 시 이미 하루가 소요되었음을 감안하면 -1 은 생략 가능함.
  - Q[0] =[7, 1]


* 두번째 값 (30, 30) 을 진행 시 else 진행
  - Q[0] = [7, **2**]


* 세번째 값 (55, 5) 을 진행 시  Q[-1][0]<-((p-100)//s) 조건을 충족 if 문 실행
  - Q = [[7, 2], **[9, 1]**]


* Q 에서 리스트의 각 첫번째 값으로 새로운 list 생성하여 return.
