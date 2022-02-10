---
title: 타입 검사 (Type Checking)
subtitle : 프로그래밍 언어론 - 원리와 실제
date: 2022-02-07
category: General Knowledge for Programming
tags : [프로그래밍 언어, type Checking, 타입검사, 타입 시스템, Type System, 타입 규칙, typing rule, 타입 환경, type environment, 안전한 타입 시스템, sound type system, 강한 타입 언어, strongly typed language, 약한 타입 언어, weakly typed language, 정적 타입 언어, statically typed language, 동적 타입 언어, Dynamically typed language, 자동 형변환, implicit type conversion, automatic type conversion]
layout: post
---

해당 내용은 책 [프로그래밍 언어론 - 원리와 실제 (저자: 창병모)](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791185578729) 을 학습한 내용을 바탕으로 작성되었습니다.

이전 포스트 [(컴퓨터가 수식과 구문구조를 이해하는 원리)]({{site.baseurl}}/rust from scratch/001stringliteral.html) 에서 학습한 구문법(syntax grammar) 은 1세대 기술로 1970 년대 활발히 연구 개발되었다고 한다.

타입 검사는 2세대 기술로 1990 년대 연구 개발 되어 Java, C#, Python 등에 적용했다고 한다.

어떻게 타입 검사를 하는지 간단하게 알아보고, 현재 언어에 적용된 상황을 살펴 보자.

------------------------------------------------------------------------------
### 1. 타입 시스템 (Type System)

#### 1.1 타입 규칙 (typing rule)

> 타입 규칙은 수식, 문장, 함수 등과 같은 프로그램 구성 요소의 올바른 타입 사용 및 그 결과 타입을 정하는 규칙 이다.

에를 들면, 정수 표현 E1 과 E2 를 더하면 결과값은 정수임을 알고 있다. 이를 논리적 추론 규칙(logical inference rule) 형태로 기술하면 다음과 같다.

<p align="center"><strong>If an expression E1 has type integer and an expression E2 has type integer, </strong></p>
<p align="center"><strong>then E1 + E2 has type integer </strong></p>


이 문장을 기호화된 타입 규칙을 표현하면,

<p align="center"><strong>(E1 : integer ^ E2 : integer) → E1 + E2 : integer </strong></p>


결합기호 ^ 에 대해서, 기존 syntax 에서 유도 과정에서 진행것과 동일하게, 재귀적으로 무한 integer 의 결합 (E1^E2^E3...^En) 에 대해서도 성립한다.

이렇게 일반환된 타입 규칙을,

<p align="center"><img src="003drawio01.png"></p>

으로 나타내며, 일반화된 결합조건을 기호 '┣ ' 와 함께 표현하면 다음과 같다.

<p align="center"><img src="003drawio02.png"></p>

위 정의를 트리 형태로 표현하면 다음과 같다.

<p align="center"><img src="003drawio03.png"></p>

#### 1.2 타입 환경(type environment)

위에서 입력 표현식을 수식을 계산한 결과값에 대한 타입 규칙은 지정되었으나, 만약 변수를 수식에 대입한다면 어떻게 될까?

이를 해결하기 위해서는 이전 포스트에서 정의한 식별자(identifier) Token 의 집합에서 Type Token 의 집합으로 가는 함수에 표현되어 있어야 한다. 이렇게

<p align="center"><strong>Γ : Identifier → Type</strong></p>

이렇게 type 집합에 대한 함수를 정의하여 계산 후의 상태 (state) 의 type 을 유지, 보장 할 수 있는 환경을 <strong>타입 환경(type environment)</strong> 이라고 하며 이는 다음 과 같이 표현할 수 있다.

<p align="center"><strong>Γ = { x ┣> integer, y ┣> integer }</strong></p>

#### 1.3 타입 안정성

위와 같이 언어의 수식이나 문장들의 타입 규칙을 설계, 운용되는 전체에 대해 타입 시스템(type system) 이라고 한다.

그리고 타입 시스템에 의해 어떤 식의 타입이 t라고 할 때, 실제 적용에서 예외 없이 항상 t 타입이 보장될 경우, 이를 안전한 타입 시스템(sound type system) 이라고 한다.

<p align="center"><img src="003drawio04.png"></p>

------------------------------------------------------------------------------

### 2. Type System 의 실제 적용

Type checking 이 Syntax analysis 보다 늦게 개발 되었으며, 각 프로그래밍 언어의 특성에 따라, 그 적용 정도가 다르다.

#### 2.1 강한 타입 언어 (strongly typed language)

 * 엄격한 타입 규칠을 적용하여 (가능한 모든) 타입 오류를 찾아 낼 수 있는 언어

 * Java, ML, C#, Pythone 등

#### 2.2 약한 타입 언어 (weakly typed language)

 * 느슨한 타입 규칙을 적용하여, 타입 검사를 하더라도 이 검사를 통과한 프로그림애 실행 중에 타입 오류를 발생 시킬 수 있다.

 * C/C++, PHP, Perl, Java Script 등


#### 2.3 정적 타입 언어 (statically typed language)

 * 변수의 타입이 컴파일 시간에 결정되어 고정되므로 보통 타입 검사도 컴파일 시간에 한다.

 * 한번 선언된 변수는 선언된 타입으로 고정되어 프로그램 중간에 다른 타입의 데이터를 갖도록 할 수 없다.

 * Java, C, C++, FORTRAN, Pascal, Scala 등

 * Java 예시
```
String name;    // 변수를 선언하면서 해당 type 을 명시해야 함.
name = "John";
name = 34;      // error! : String type 변수에 다른 type 의 data 를 대입할 수 없다.
```

#### 2.4 동적 타입 언어 (Dynamically typed language)

 * 변수의 타입이 고정되지 않고, 실행 중에 변경이 가능한 언어.

 * Perl, python, Sheme, JavaScript 등

 * JavaScript 예시
```
var name;       // 변수 선언시, type 에 대한 명시가 없다.
name = "John";  // string type 값을 대입
name = 34;      // integer type 으로 새로 대입해도 오류가 발생하지 않는다.
```

언어별 type 에 대한 성향을 그래프로 표현하면 아래와 같다.

<p align="center"><img src="003typespectrum05.png"></p>

------------------------------------------------------------------------------

### 3. 자동 형변환 (implicit type conversion / automatic type conversion)

항상 동일한 type 간의 계산을 진행하는 것이 이상적이지만, 현실에서 이를 항상 지키는 것은 불가능하다. 심지어 같은 정수 타입에도, 8비트 부터 64비트 다른 type 이 존재하며, 음수를 포함한 정수형, 0 과 양수만을 표현가능한 정수형 등 세분화 되어 있다.

따라서, 한번 선언된 변수의 type 에 대해 별도의 명령이 없으면, 끝까지 해당 type 에 맞게 code 를 작성해야 하는 언어가 있고, 사용자 편의를 고려하여, 일부 자동으로 type 변경 (implicit type conversion / automatic type conversion)을 해주는 언어가 있다. (ex. C/C++, Java 등)

자동 형변환이 적용된 언어들은 이항 연산에서 두 피연사자의 자료형이 서로 다를 경우, 표현 범위가 큰쪽으로 자동으로 자료형을 변환한 후 연산한다. 이를 상향 변환(promotion) 또는 확장 변환(widening conversion) 하며, 자동 형변환은 대부분 확장 변환이다.

 Java 를 예로 들어, 정수형 / 실수형 확장 변환 순서를 적어보면 다음과 같다. (왼쪽에서 오른쪽 방향으로 변환 진행)

 ```
 byte(1) < short(2) < int(4) < long(8)
 float(4) < double(8)
 ```

------------------------------------------------------------------------------

#### 4. 결론

* 현재 공부하고 있는 rust 의 경우, 매우 안전한 타입 시스템을 가지며, 가장 정적이고 강한 타입의 언어 중 하나일 것이다. 상대적으로 low level language 인 C/C++ 의 단점(weakly typed language)을 보안하고, 이를 대체(compiling language 이므로 statically typed language) 위한 목적으로 설계된 언어이기 때문에 해당 특성을 가질 것으로 예상된다.  

*  다만 자동 형변환도 거의 지원되지 않기 때문에 해당 부분이 초보자 입장에서 많이 조금 어렵게 느껴진다. 아마, 강한 정적 타입을 갖추면서 자동 형변환을 지원하는 것이  어려울 것으로 보인다.

*  마지막으로 책에 예제로 나온 C 언어 (weakly and strongly typed language) 치명적 단점을 보고 마무리 하자.

  ```
  int main() {
    int x = 0;
    float y = 3;
    char z = 'A';

    x = y;              // (1) int 형에 float 값 대입
    printf("%d\n", x);  // 출력 : 3

    y= z;              // (2) float 형에 char 값 ('A' 의 ASCII 값) 대입
    printf("%f\n", y);  // 출력 : 65.000000

    z = -1;              // (3) char 형에서 ASCII 값으로도 정의가 없는 -1 을 대입
    printf("%c\n", z);   // 오류가 발생하지 않고 알수 없는 '�' 이 출력됨
 }
 ```
* (1) compiling language 특성 상, x 를 float type 으로 변경할 수 없으므로, 기존 정수형에 해당 값을 그대로 받으므로써, 소수점 이하의 값을 소실하였다.

* (2) 'A' 의 ASCII 값 (65) 를 실수로 전환하여 값을 저장함.

* (3) char 에서 받을 수 없는 음수를 형변환 없이 그대로 받으므로써, 의미를 알수 없는 값을 가지게 됨.
