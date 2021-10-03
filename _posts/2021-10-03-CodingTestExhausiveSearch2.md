---
title: 알고리즘 STEP 2 완전탐색/이분탐색 2 소수 찾기
tags: ENCORE Algorithm CodingTest  ExhuasiveSearch
---

* 링크: [코딩테스트 연습>완전탐색>소수 찾기](https://programmers.co.kr/learn/courses/30/lessons/42839)

>### 소수 찾기


>#### 문제 설명
>
>한자리 숫자가 적힌 종이 조각이 흩어져있습니다. 흩어진 종이 조각을 붙여 소수를 몇 개 만들 수 있는지 알아내려 합니다.
>
>각 종이 조각에 적힌 숫자가 적힌 문자열 numbers가 주어졌을 때, 종이 조각으로 만들 수 있는 소수가 몇 개인지 return 하도록 solution 함수를 완성해주세요.

>#### 제한사항
> * numbers는 길이 1 이상 7 이하인 문자열입니다.
> * numbers는 0~9까지 숫자만으로 이루어져 있습니다.
> * "013"은 0, 1, 3 숫자가 적힌 종이 조각이 흩어져있다는 의미입니다.

>#### 입출력 예

|numbers|	return|
|---------------|-------|
|"17"| 2|
|"011"|	2|

>예제 #1
>[1, 7]으로는 소수 [7, 17, 71]를 만들 수 있습니다.
>
>예제 #2
>[0, 1, 1]으로는 소수 [11, 101]를 만들 수 있습니다.
>
> * 11과 011은 같은 숫자로 취급합니다.


### 1차 시도 Code
```python
def solution(numStr):
    numbers = []
    for element in numStr:            # 문자형태로 받은 입력값을 한자리 숫자의 배열 형태로 바꿈.
        numbers.append(int(element))

    counter = 0
    nums_dict=dict()
    nums = []
    # 1. 조합 가능한 수의 배열 만들기   # 가진 숫자의 배열로 조합 가능한 모든 수의 list 를 만듬
    for i1 in range(len(numbers)):      # 가진 숫자의 배열로 조합 가능한 모든 수의 list 를 만듬
        tempNums1 = numbers.copy()      # 다음 for 문에 넘겨줄 것이므로 원본의 copy 배열을 하나 만듬.
        temp1 = tempNums1.pop(i1) * (10**(len(tempNums1))) # 가장 상단(최대 10^6 )의 값을 구성할 숫자를 배열해서 하나 뽑음
        if len(tempNums1) == 0:         # 혹시 해당 자리가 배열의 마지막 수(더이상 뺄 숫자가 배열에 남아 있지 않다면)
            nums_dict[temp1]=1          # 해당 숫자(만약 여기가 실행된다면 해당 숫자는 1자리 숫자)를 nums_dict 에 넣음
        else:                                 # 만약 배열에 남은 숫자가 있다면
            for i2 in range(len(tempNums1)):  # (앞의 과정의 반복)위에서 숫자를 하나 뺀 배열을 받아서 for 문을 실행
                tempNums2 = tempNums1.copy()  #  다음 for 에 넘겨줄 배열 copy 본을 만듬.
                temp2 = tempNums2.pop(i2) * (10**(len(tempNums2)))   # 두번째 상단(최대10^5) 의 값을 구성할 숫자를 배열해서 하나 뽑음
                if len(tempNums2) == 0:       #  혹시 해당 자리가 배열의 마지막 수(더이상 뺄 숫자가 배열에 남아 있지 않다면)
                    nums_dict[temp1 + temp2]=1#  (만약 여기가 실행된다면 해당 배열은 최대 2자리 숫자를 지니므로) 이를 nums_dict 에 넣고
                    nums_dict[temp2]=1        #  2개의 숫자중 하나만으로도 숫자 구성이 가능하므로 해당 값을 nums_dict 에 넣는다.
                else:                         # 아래는 위의 내용의 반복이므로 생략
                    for i3 in range(len(tempNums2)):
                        tempNums3 = tempNums2.copy()
                        temp3 = tempNums3.pop(i3) * (10**(len(tempNums3)))    # 최대 10^4
                        if len(tempNums3) == 0:
                            nums_dict[temp1 + temp2 + temp3]=1
                            nums_dict[temp2 + temp3]=1
                            nums_dict[temp3]=1
                        else:
                            for i4 in range(len(tempNums3)):
                                tempNums4 = tempNums3.copy()
                                temp4 = tempNums4.pop(i4) * (10**(len(tempNums4)))    # 최대 10^3
                                if len(tempNums4) == 0:
                                    nums_dict[temp1 + temp2 + temp3 + temp4]=1
                                    nums_dict[temp2 + temp3 + temp4]=1
                                    nums_dict[temp3 + temp4]=1
                                    nums_dict[temp4]=1     
                                else:
                                    for i5 in range(len(tempNums4)):
                                        tempNums5 = tempNums4.copy()
                                        temp5 = tempNums5.pop(i5) * (10**(len(tempNums5)))    # 최대 10^2
                                        if len(tempNums5) == 0:
                                            nums_dict[temp1 + temp2 + temp3 + temp4 + temp5]=1
                                            nums_dict[temp2 + temp3 + temp4 + temp5]=1
                                            nums_dict[temp3 + temp4 + temp5]=1
                                            nums_dict[temp4 + temp5]=1
                                            nums_dict[temp5]=1
                                        else:
                                            for i6 in range(len(tempNums5)):
                                                tempNums6 = tempNums5.copy()
                                                temp6 += tempNums6.pop(i6) * (10**(len(tempNums6)))    # 최대 10^1
                                                if len(tempNums6) == 0:
                                                    nums_dict[temp1 + temp2 + temp3 + temp4 + temp5 + temp6]=1
                                                    nums_dict[temp2 + temp3 + temp4 + temp5 + temp6] =1
                                                    nums_dict[temp3 + temp4 + temp5 + temp6] =1
                                                    nums_dict[temp4 + temp5 + temp6] =1
                                                    nums_dict[temp5 + temp6] =1
                                                    nums_dict[temp6] =1
                                                else:
                                                    for i7 in range(len(tempNums6)):
                                                        tempNums7 = tempNums6.copy()
                                                        temp7 += tempNums7.pop(i7) * (10**(len(tempNums7)))    # 10^0 (최대 7자리의 경우)
                                                        nums_dict[temp1 + temp2 + temp3 + temp4 + temp5 + temp6 + temp7] = 1
                                                        nums_dict[temp2 + temp3 + temp4 + temp5 + temp6 + temp7] = 1
                                                        nums_dict[temp3 + temp4 + temp5 + temp6 + temp7] = 1
                                                        nums_dict[temp4 + temp5 + temp6 + temp7] = 1
                                                        nums_dict[temp5 + temp6 + temp7] = 1
                                                        nums_dict[temp6 + temp7] = 1
                                                        nums_dict[temp7] = 1

    # 2. 수의 조합 배열에서 소수 구하기 # 모든 배열의 수를 각각 완전탐색으로 소수인지 찾는것은 비효율적일것으로 판단.
    num_max = max(nums_dict.keys())     # 배열에서 가장 큰수를 찾아 여기까지에 해당하는 소수의 배열을 만들어 비교 하고자 함.
    prime_nums=[2]                      # 소수의 배열

    for nu in range(3, num_max+1, 2):   # 짝수를 for문을 돌려 효율성을 높이고자 함. (결과적으로 실제 속도 개선은 거의 없었음.)
            for deviding in range(2, nu):# 3~ max 값 사이의 값들이 만약 2 ~ 자기 자신-1 의 수로
                if nu % deviding ==0:   # 나누어 진다면
                    break               # 해당수는 소수가 아니며
                if nu - deviding == 1:
                    prime_nums.append(nu)# 끝까지 나눠지지 않는 수만 소수 배열에 넣음.

    for num in prime_nums:               # 소수 배열과 주어진 숫자들의 조합으로 만든 배열 dictionary  key 값을 비교
        if num in nums_dict:             
            counter += 1                 # 일치하는 수가 나올때마다 counter 로 갯수를 셈

    return counter

```

### Error Comments (1st)

  * test 결과 : 12개 test 중 5개 통과/ 5개 런타임 에러/ 2 개 시간초과

  * 어렵지 않을 것이라고 예상한 1번째 수의 조합을 생성하는 부분에 예상보다 훨씬 길고 복잡한 code 가 생성됨.

  * 우선 첫번째 부분의 해결에 초점을 둠. 분명 library 를 이용하는 방법이 있을것으로 예상되어 인터넷 검색결과 다음과 같이 사용할수 있음을 찾음 .

```python
from itertools import permutations

items = ['A', 'B', 'C', 'D']
print(list(map(''.join, permutations(items)))) # items의 모든 원소를 가지고 순열을 만든다.
print(list(map(''.join, permutations(items, 2)))) # 2개의 원소를 가지고 순열을 만든다
```
```python
>>['ABCD', 'ABDC', 'ACBD', 'ACDB', 'ADBC', 'ADCB', 'BACD', 'BADC', 'BCAD', 'BCDA', 'BDAC', 'BDCA', 'CABD', 'CADB', 'CBAD', 'CBDA', 'CDAB', 'CDBA', 'DABC', 'DACB', 'DBAC', 'DBCA', 'DCAB', 'DCBA']

>> ['AB', 'AC', 'AD', 'BA', 'BC', 'BD', 'CA', 'CB', 'CD', 'DA', 'DB', 'DC']
```

  * permutations 를 사용하여 해당 부분을 개선함.

### 2차 시도 Code
```python
def solution11(numStr):
    from itertools import permutations

    counter = 0
    nums_dict=dict()
    nums = []
    num = 0
    numTemp=0
    for idx, element in enumerate(numStr):     # 입력값을 직접 받아 순열 조합의 숫자 배열을 생성
        nums += permutations(numStr, idx+1)    # 입력(배열)에서 (idx+1)갯수 만큼의 원소를 순열조합하여 nums list 에 추가함
                                               # nums = [('1',), ('7',), ('1', '7'), ('7', '1')]
    for num in nums:                           
        for i in range(len(num)):              # nums (문자들의 tuple 들을 원소로 가진 list) 를 숫자로 변환하기 위해
            numTemp += int(num[i]) * (10 ** i) # Tuple 내 문자들을 int 로 변환 후 위치에 맞는 10 진수를 부여함.
        nums_dict[numTemp] = 1                 # 이를 겹치는 값들은 제거되도록 dictionary 로 옮김
        numTemp=0

    # 이하 기존과 동일
    num_max = max(nums_dict.keys())
    prime_nums=[2]

    for nu in range(3, num_max+1, 2):  
            for deviding in range(2, nu):
                if nu % deviding ==0:
                    break
                if nu - deviding == 1:
                    prime_nums.append(nu)

    for num in prime_nums:
        if num in nums_dict:
            counter += 1
            #print(num)

    return counter
```
### Error Comments (2nd)

  * code 는 매우 간략해졌지만, 실제 Test 에서는 5개 통과  / 7개 시간 초과 의 결과를 얻음 (런타임 오류는 없어짐.)


  * 따라서 시간 단축을 이루기 위해서는 다른 부분에서의 정리가 필요해 보임.

### 3차 시도 Code
```python
def solution3(numStr):
    from itertools import permutations

    # 기존 dictionary 를 사용한 이유는 중복 data 를 제거 하기 위함이였으므로 해당 부분을 set 으로 바꿈.
    nums_set=set()
    nums = []
    num = 0
    numTemp=0
    for idx, element in enumerate(numStr):
        nums += permutations(numStr, idx+1)

    for num in nums:
        for i in range(len(num)):
            numTemp += int(num[i]) * (10 ** i)
        nums_set.add(numTemp)
        numTemp=0

    num_max = max(nums_set)
    prime_nums=set([2])

    for nu in range(3, num_max+1, 2):  
            for deviding in range(2, nu):
                if nu % deviding ==0:
                    break
                if nu - deviding == 1:
                    prime_nums.add(nu)

    return len(nums_set.intersection(prime_nums))

```
### Error Comments (3rd)

  * dictionary -> set 으로 변경하였으나 역시 속도에 큰 변화는 없음.


  * 결국 소수를 찾아 비교하는 부분에 대한 완전한 변경이 필요해 보임.

### 4차 시도 Code
```python
def solution(numStr):
    from itertools import permutations

    nums_set=set()
    nums = []
    num = 0
    numTemp=0
    for idx, element in enumerate(numStr):
        nums += permutations(numStr, idx+1)

    for num in nums:
        for i in range(len(num)):
            numTemp += int(num[i]) * (10 ** i)
        nums_set.add(numTemp)
        numTemp=0
                                                 # 위에서 찾은 수의 조합 list 를 각각 소수인지를 확인하고 아닌 수들을 제거하기로 함.  
    nums_list = list(nums_set.difference([0,1])) # for 문에 의한 직접 비교는 2 부터 이루어 지므로 우선 0,1 을 제거
    result = nums_list.copy()                    # 소수만 남기기 위한 별도의 배열 copy
    #print("1: ",result)   # 오류확인용
                                                 # list 의 모든 수를 비교하되, 반복횟수를 줄이고자
    for num in nums_list:                        # 각수의 제곱근+1 의 위치까지만 숫자를 올려가면서 해당 값으로 나눠봄
        for i in range(2, int(num**(1/2)+1) ):   # ex) 36 이면 2, 3, 4, 6, 9, 12, 24 의 약수가 존재. 해당 값들은 2*24, 3*12, 4*9, 6*6 의 짝으로 이루짐.
                                                 #     즉 해당 수의 제곱근(6) 근처까지만 확인해서 만약 그 앞에 약수가 나오지 않는다면 그수는 소수임을 알수 있음.
            if (num % i) == 0 and num != 2 and num !=3: # 위 수식에서 2, 3 검사 불가능 하며, 소수 이므로 pass
                result.remove(num)               # 제곱근+1 이하의 수에서 약수가 발견되면 최종배열에서 제거
                break

    #print("2: ",result) # 오류 확인용
    return len(result)
```
### Error Comments (4th)

  * 최종 Test 통과 최대 소요 시간 : 29.33ms


  * 해당 code 에서 별도의 result 배열을 만들지 않고 for 에서 사용중인 nums_list 를 그대로 사용하여 remove(num) 을 반영하여 결과 값을 얻고자 하였으나. 문제 발생
    - remove 되면 배열자체가 한칸 줄어들기 때문에 for 문이 정상적으로 작동하지 않으며, 결과값이 다르게 산출됨을 확인함.
