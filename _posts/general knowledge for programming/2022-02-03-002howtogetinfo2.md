---
title: Token 의 정의 부터 AST 작성까지.
subtitle : 프로그래밍 언어론 - 원리와 실제
date: 2022-02-05
category: General Knowledge for Programming
tags : [프로그래밍 언어, 추상 구문 트리, AST, Abstract Syntax Tree, 어휘 분석, lexical analyzer, 파서, parser, parsing, compile]
layout: post
---

해당 내용은 책 [프로그래밍 언어론 - 원리와 실제 (저자: 창병모)](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791185578729) 및 관련 검색으로 통해 학습한 내용임.

이전 포스트 [(링크)]({{site.baseurl}}/rust from scratch/001stringliteral.html)에서 code 로 작성된 수식이나, 구문에 대해서 어떻게 컴퓨터가 읽을 수 있는지 알아보았다. 이번 Post 에서는 컴퓨터가 이를 해석 및 수행하는 과정까지 알아보자.

------------------------------------------------------------------------------

컴퓨터의 code 읽고, 적합성을 판단하고, 실제로 실행하기까지의 과정을 다이어그램으로 표현하면 다음과 같다.

<p align="center"><img src="002diagram01.png"></p>

------------------------------------------------------------------------------
### 1. 어휘 분석 (lexical analysis)

이전 Post 에서 수식과 구문에 대해 분석 했지만, code 에는 그 외 표현도 많다. 특히 non-termal 의 경우, 그 표현이 정해저 있지 않기 때문에 컴퓨터가 판단할 기준이 필요하다.

예를들면,  변수 hello 를 선언했다면, 이것이 변수명으로 사용된 것인지, 다른 의미로 사용된 것인지, 잘못 타이핑 된것인지 어떻게 알 수 있을까?

syntax 에 따라 수식을 계산할 수는 있겠지만, 어떤것을 숫자로 인식해서 계산할 수 있을까? 5.23e8 는 숫자인식해야 할까?

때문에 구문분석(parsing) 전, 어휘 분석부터 진행해야 하며, 어휘 분석을 하기 위해서는 모든 문법이 정의 되어 있어야 하며, 정의된 문법적 단위를 Token 이라 한다.  

#### 토큰 (Token)

syntax 을 정의하여 계산 가능 여부를 판단했던 것과 같이, code 에 사용될 모든 글자, 단어들에 대해서도 정의가 필요하다. 각각의 정의에 이름(Name Token)을 부여하고, 이에 대한 속성(Token Attrubute)를 쌍으로 정의해 놓는다면, 이에 따라 컴퓨터는 현재 작성자가 어떤 의미로 타이핑을 했는지, 가장 최소 단위에부터 인식할 수 있게 된다.

토큰 몇가지 명시해 보면 다음과 같다. (언어마다 다를 수 있음.)

&nbsp;
&nbsp;

* 예약어 (키워드) : 언어에서 미리 그 의미와 용법이 지정되어 사용되는 단어

  - bool, true, false, if, else, while, return 등

  &nbsp;
  &nbsp;

* 연산자 : 수식 연산의 적합성 판단 및 계산에 사용

  - =, ==, +, -, *, / 등

  &nbsp;
  &nbsp;

* String literal : "..." 의 형태로  큰 따옴표로 둘러싸인 임의의 문자열

  - 참고로 컴퓨터 공학에서 "literal" 이라는 의미는 "원시코드에 있는 어떤 고정된 값 (representing a fixed value in source code)" 을 의미한다고 한다. 이를 다시 풀어서 이야기하면, 어떠한 숫자 또는 기호로 다른 데이터를 가리키는 역할을 하지 않고, 그 자신이 바로 데이터로 사용되는 것을 말한다.

  &nbsp;
  &nbsp;

* 식별자 : 변수 또는 함수의 이름을 나타내는 token

  - 아래에서 식별자 Token 간략하게 정의해 보면,

```python
  식별자 이름 (Name Token) : ident 로 명명

  식별자 작성 규칙 : 첫번째 기호가 영문자 혹은 언더바(_)로 시작하고 두번째 기호부터는 영문자나 숫자, _ 를 허용

   - 이를 표현하기 위해서는 우선 영문자와 숫자에 대한 정의가 필요하며, 이를 BNF 로 표현하면,

     <digit>  = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
     <letter> = a | b | c |....| z | A | B | C | Z |

   - 변수명 식별자(ident) 의 BNF 는 다음과 같다.

     <ident> = (<letter> | _ ) (<letter> | <digit> | _ )
```

해당 내용을 token 으로 정리하면 다음과 같다.

       Name Token      : ident
       Token Attribute : (<letter> | _ ) (<letter> | <digit> | _ )
       Token Pattern   : 첫번째 기호가 영문자 혹은 언더바(_)로 시작하고 두번째 기호부터는 영문자나 숫자, _ 가 올수 있음.

그리고 일반적으로 Token 을 표현할 때, 그 이름과 속성의 쌍으로 표현하므로

       <ident, (<letter> | _ ) (<letter> | <digit> | _ )>

------------------------------------------------------------------------------
### 2. 파싱 (Parsing)

컴퓨터가 구문 분석(parsing) 을 할 때, 분석 중인 표현에 대해, 언어에서 미리 정의된 Token 을 어휘 분석기로부터 요청 (get Token()) 하여 해당 Token 을 가져온 후, code 를 분석하고, 분석된 결과를 추상 구문 트리(AST) 를 구성 하여 return 한다.

Parsing program 은 현재 학습의 목적의 범위를 벗어나는 것으로 판단되어 넘기도록 하겠다.

------------------------------------------------------------------------------
### 3. 추상 구문 트리 (Abstract Syntax Tree (AST))

AST 는 이전 Post 에서 언급한 Parsing Tree 를 추상화(간략화) 표현한 것이다. 다만 앞에서 주로 수식(expression) 과 구문구조(statement) 에 국한하여 작성했다면, 이제 lexical analyzer 에서 정의된 모든 Token 을 가지가 parsing 된 내용을 구조화 하는 것이다.

임의로 Token 이 정의되고, 이에 대한 parsing 이 완료되었다고 가정하고, 간단한 code 로 AST 를 구성하면 다음과 같다.

CODE
```c++
while (x > 0) {
  y = y * x;
  x = x - 1;
}
```
Abstract Syntax Tree
<p align="center"><img src="002diagram02.png"></p>


#### 4. 인터프리터(Interpreter)

프로그램 내의 각 문장의 AST 를 순회하면서 각 문자의 의미에 따라 해석하고 해당 내용을 실행

#### 5. 상태 (State)

인터프리터가 각 문장을 수행하면서 유효한 변수들의 상태를 저장하는 stack, heap 형태의 자료구조를 사용하여 data 를 저장, 수정, 삭제 등의 업무을 수행

#### 결론

간략하게나마 컴퓨터가 program 을 어떻게 인식하고, 이를 수행하는지를 이해할 수 있게 되었다.
