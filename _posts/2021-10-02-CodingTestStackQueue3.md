---
title: 알고리즘 STEP 1 Stack/Queue 3 다리를 지나는 트럭
tags: ENCORE Algorithm CodingTest Deque Collections
---

* 링크: [코딩테스트 연습>스택/큐>다리를 지나는 트럭](https://programmers.co.kr/learn/courses/30/lessons/42583)

>### 다리를 지나는 트럭


>#### 문제 설명
>
>트럭 여러 대가 강을 가로지르는 일차선 다리를 정해진 순으로 건너려 합니다. 모든 트럭이 다리를 건너려면 최소 몇 초가 걸리는지 알아내야 합니다.
>다리에는 트럭이 최대 bridge_length대 올라갈 수 있으며, 다리는 weight 이하까지의 무게를 견딜 수 있습니다.
>단, 다리에 완전히 오르지 않은 트럭의 무게는 무시합니다.
>예를 들어, 트럭 2대가 올라갈 수 있고 무게를 10kg까지 견디는 다리가 있습니다.
>무게가 [7, 4, 5, 6]kg인 트럭이 순서대로 최단 시간 안에 다리를 건너려면 다음과 같이 건너야 합니다.

|경과 시간|다리를 지난 트럭|다리를 건너는 트럭|대기 트럭|
|--------|---------------|-----------------|--------|
|0	|[]|	[]|	[7,4,5,6]|
|1~2|	[]|	[7]|	[4,5,6]|
|3|	[7]|	[4]|	[5,6]|
|4|	[7]|	[4,5]|	[6]|
|5|	[7,4]|	[5]|	[6]|
|6~7|	[7,4,5]|	[6]|	[]|
|8|	[7,4,5,6]|	[]|	[]|

>따라서, 모든 트럭이 다리를 지나려면 최소 8초가 걸립니다.
>solution 함수의 매개변수로 다리에 올라갈 수 있는 트럭 bridge_length, 다리가 견딜 수 있는 무게 weight, 트럭 별 무게 truck_weights가 주어집니다.
>이때 모든 트럭이 다리를 건너려면 최소 몇 초가 걸리는지 return 하도록 solution 함수를 완성하세요.

>#### 제한사항
> * bridge_length는 1 이상 10,000 이하입니다.
> * weight는 1 이상 10,000 이하입니다.
> * truck_weights의 길이는 1 이상 10,000 이하입니다.
> * 모든 트럭의 무게는 1 이상 weight 이하입니다.

>#### 입출력 예

|bridge_length	|weight	|truck_weights|	return|
|---------------|-------|-------------  |------|
|2	|10	|[7,4,5,6]|	8|
|100|	100|	[10]|	101|
|100|	100|	[10,10,10,10,10,10,10,10,10,10]|	110|


### Code1 (최장 9781.72ms 소요, 그러나 재시도 시, 시간초과로 실패 )
```python
def solution(bridge_length, weight, truck_weights):
    counter=0
    loading_weights = []

    while len(loading_weights) < bridge_length-1:
            loading_weights.append(0)

    for truck_weight in truck_weights:

        if len(loading_weights) == bridge_length:
            loading_weights.pop(0)

        if (sum(loading_weights) + truck_weight) <= weight:
            loading_weights.append(truck_weight)
            counter += 1

        else:
            while (sum(loading_weights) + truck_weight) > weight:
                loading_weights.append(0)
                counter += 1
                if len(loading_weights) == bridge_length:
                    loading_weights.pop(0)

            loading_weights.append(truck_weight)
            counter += 1

    counter += bridge_length       

    return counter                            
```


### Code2 (최장소요시간:  0.87ms)
```python
def solution(bridge_length, weight, truck_weights):
    import collections

    trucks = collections.deque(t for t in truck_weights)  
    loaded = collections.deque()                             # 다리위에 올라간 차랑의 queue
    counter = collections.deque()                            # 다리위 각 차량끼리의 간격
    tTime = 0                                                # ex) 첫번째 차량은 무조건 처음 1의 counter 를 가짐 (case 1.1 실행에 의해서)                                                      #
                                                             #     두번째 차랑이 첫번째 차량에 바로 이여서 출발할 경우 1
    while len(trucks) > 0:                                

        if sum(counter) < bridge_length:                     # case 1 : 전체 차량이 늘어선 길이가 다리길이보다 짧다면
            if sum(loaded) + trucks[0] <= weight:            #      1.1: 그중 (다리위 차량 무게 + 대기열 첫번째 차량 무게)가 허용무게보다 적다면
                loaded.append(trucks.popleft())              #           -> 다리위에 차량을 추가하고
                counter.append(1)                            #           -> 해당차량의 간격은 1
                # print("1 : ", tTime, " : " , loaded, "  ,  " ,counter) # 오류 확인용

            else:                                             #   1.2 : 허용무게보다 이상이  다리위 + 대기열 첫번째에 있다면
                counter[-1] += (bridge_length - sum(counter)) #        -> 더 올리지 않고 해당 차량들을 다리끝까지 보냄

        else:                                                 #  case2
            if sum(loaded) - loaded[0] + trucks[0] <= weight: #    2.1 : 다리 길이를 차량들이 꽉채웠으나, 아직 허용 무게를 넘지않은경우
                loaded.popleft()                              #          -> 맨 앞의 차량은 빼내고
                loaded.append(trucks.popleft())               #          -> 뒤에 차량 다리위로 올리고

                tTime += counter.popleft()                    #          -> 빠진 차량의 counter 를 tTime 에 더하고
                counter.append(1)                             #          -> 새로 올라온 차량의 counter  1을 붙임
                # print("2 : ", tTime, " : " , loaded, "  ,  " ,counter) # 오류 확인용

            else:                                                   #    2.2 : 무게 때문에 차량을 더 올릴 수 없다면
                while sum(loaded) - loaded[0] + trucks[0] > weight: #    -> 다리위 무게가 대기열 차량의 무게를 받을 수 있을때까지
                    loaded.popleft()                                #    -> 기존 차량을 앞에서 한대씩 보내고
                    tTime += counter.popleft()                      #    ->  보낸 차량의 counter 를 더함.
                    # print("3 : ", tTime, " : " , loaded, "  ,  " ,counter) # 오류 확인용

    counter[-1] += bridge_length                              # 대기열 마지막차랑가지 다리에 오르면, 해당 차량들은 그대로 전진하여
                                                              # 다리를 건너므로 마지막 차량의 counter = 다리길이 임.
    while len(counter) >0:                                    # 다리위에 남아있는 차량들에 대한 counter 를 모두 tTime 에 더함.
        loaded.popleft()
        tTime += counter.popleft()
        # print("4 : ", tTime, " : " , loaded, "  ,  " ,counter)   # 오류 확인용   

    return tTime
```

### Error Comments

#### code1 문제점 고찰
  * code1 의 전체적 구성은 list 형태로 다리위에 순서대로 차량을 올리고 만약 허용 중량을 넘기는 것이 예상될때는 해당 list 에 0을 채워서 가면, 해당 process 가 진행될 때마다 counter 가 1씩 증가하여 소요되는 시간을 산출하는 방식임.



  * 문제점
    1. 뒤에 0을 채워야 하는 경우에 대해 남은 다리 길이 만큼 while 문을 통해 0을 채워나가야 함.
      - 따라서 다리 길이가 길 경우, 이를 처리하는 시간이 매우 길어지게 됨.


    2. 시간을 줄이고자 다양한 경우에 대해 시간 단축을 시킬 수 있는 code 를 추가해봄.
      - ex) 매우 무거운 차량이 올라온 경우, 그 뒤에는 바로 다른 차량이 따라올 수 없고, 그대로 다리 끝까지 가야 하므로, 한번에 다리끝까지 갈 수 있도록 다수의 0 을 뒤에 채우고, 해당 숫자만큼 counter 를 더함.
        -> 해당 code 를 추가할 경우, 시간은 크게 단축 시킬수 있었으나, 일부의 경우에 대해 error 가 발생 (해당 배열에 0 과 차량 중량값이 복잡하게 섞여있는 경우, 전체 배열을 한번에 처리했을 경우 문제가 발생)
      - 전체적으로  예외 사항 발생에 대한 추가 code, 추가 code 에 의한 error 발생을 없애고자 조건문이 추가되어 점점 code 는 복잡해지고, 세분화된 조건에 대한 추가 error 가 발생하게 됨.



  * 해결 방법
    1. 해당 code 의 수정을 포기하고, 새로운 방법을 고안하여 문제 해결
      - 전체 다리길이 만큼의 배열에 차량을 추가하고 빈공간을 0 을 추가하는 기존의 방식으로는 시간을 단축 시키기 어려울 것으로 판단
      - 다리위 차량의 queue 와  해당 차량이 다리를 건너는데 소요되는 시간을 측정하는 counter queue 를 각가 만들고, 각각의 조건에서 해당 값들을 조절함. (세부 사항은 code 내 주석 참조. )


    2. 결과적으로 code processing 시간은  1/10000 이하로 단축되었으며 (9781.72ms -> 0.87 ms), 예외 사항 발생에 대한 고려가 추가되지 않아, code 의 가독성도 높아짐.


  * collections     ([Python Documentation link](https://docs.python.org/ko/3/library/collections.html))
    1. 문제 해결에 어려움이 있어, 그 과정 중 다른 사람의 코드를 참조함. (참조한 코드 자체는 제대로 이해할 수 없었으며, 위의 코드는 참조한 코드와는 거의 무관함.)
      - 해당 code 에서 사용된 collections 를 사용해봄.
      - 모듈 정의


    > collections — 컨테이너 데이터형
    >
    > 파이썬의 범용 내장 컨테이너 dict, list, set 및 tuple에 대한 대안을 제공하는 특수 컨테이너 데이터형을 구현
    >
    > namedtuple(): 이름 붙은 필드를 갖는 튜플 서브 클래스를 만들기 위한 팩토리 함수
    >
    >deque:  양쪽 끝에서 빠르게 추가와 삭제를 할 수 있는 리스트류 컨테이너
    >
    >ChainMap: 여러 매핑의 단일 뷰를 만드는 딕셔너리류 클래스
    >
    >Counter:  해시 가능한 객체를 세는 데 사용하는 딕셔너리 서브 클래스
    ?
    >OrderedDict:  항목이 추가된 순서를 기억하는 딕셔너리 서브 클래스
    >
    >defaultdict:  누락된 값을 제공하기 위해 팩토리 함수를 호출하는 딕셔너리 서브 클래스
    >
    >UserDict:   더 쉬운 딕셔너리 서브 클래싱을 위해 딕셔너리 객체를 감싸는 래퍼
    >
    >UserList: 더 쉬운 리스트 서브 클래싱을 위해 리스트 객체를 감싸는 래퍼
    >
    >UserString: 더 쉬운 문자열 서브 클래싱을 위해 문자열 객체를 감싸는 래퍼

    2. deque 객체


    > 데크는 스택과 큐를 일반화 한 것입니다 (이름은 《deck》이라고 발음하며 《double-ended queue》의 약자입니다). 데크는 스레드 안전하고 메모리 효율적인 데크의 양쪽 끝에서의 추가(append)와 팝(pop)을 양쪽에서 거의 같은 O(1) 성능으로 지원합니다.
    >
    >list 객체는 유사한 연산을 지원하지만, 빠른 고정 길이 연산에 최적화되어 있으며, 하부 데이터 표현의 크기와 위치를 모두 변경하는 pop(0)과 insert(0, v) 연산에 대해 O(n) 메모리 이동 비용이 발생합니다.

    3. deque 의 활용
      - 초기 풀이 방법을 수정하고자 할 때, 다리 길이를 가진 배열의 초기화에 `bridge = collections.deque([0]*bridge_length)` 로 활용하였으나, 최종 code 에서는 필요없어짐.
      - pop / popleft , append / appendleft 와 배열의 양쪽에서 접근하여 값을 추가/인출 할 수 있음. (기존의 list 를 활용한 방법과 어떤차이가 있는지는 모르겠음. )
