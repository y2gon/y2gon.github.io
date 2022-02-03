---
title: 컴퓨터는 어떻게 (고차원) 프로그래밍 언어를 이해하는가? (2)
subtitle : 프로그래밍 언어론 - 원리와 실제
date: 2022-02-02
category: General Knowledge for Programming
tags : [프로그래밍 언어, 유도, derivation, 유도 트리, derivation tree, 파스 트리, parse tree, 구문 트리, syntax tree]
layout: post
---

해당 내용은 책 [프로그래밍 언어론 - 원리와 실제 (저자: 창병모)](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791185578729) 및 관련 검색으로 통해 학습한 내용임.

앞에서 컴퓨터가 프로그램을 이해하는 최소 단위에 대해 알아보았다. 그렇다면 좀더 긴 수식은 어떻게 확인하고, 우리에게 오류를 알려 줄 수 있을까?

유도 (Derivation)
-------------

String E 가 "3 + 4 * 5" 일 때, syntax 가 다음과 같이 정의된 프로그램 언어 환경에 입력했을 경우,

 Defined Syntax

      E -> E + E | E * E | (E) | N                     : 입력값에 대해 +, *, () 연산 및 이진수로 해석 가능이 정의됨
      D -> 0  | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 0  : 0 ~ 9 컴퓨터가 인식 가능한 숫자 표기
      N -> D | ND                                      : 이진수는 그자체 또는 이진수와 숫자(0, 1) 의 연속 표기 형태

위 정의를 사용하여 다음의 유도(derivation) 과정을 통해 여려개의 연산자가 연결된 수식을 처리할 수 있다.

      E => E + E => N + E => D + E => 3 + E => 3 + E * E => 3 + N * E  
      => 3 + D * E => 3 + 4 * E => 3 + 4 * N => 3 + 4 * D => 3 + 4 * 5

하나의 입력된 덩어리 E에 대해, 정의된 syntax 내에서 참인 조건들로 위의 과정을 유도하면, 최종적으로 "3 + 4 * 5" 라는 String '참' 이 조건, 즉 컴퓨터가 계산 가능한 표현식(expression) 임이 증명되었다.


유도 트리 (Derivation Tree)
-------------
이를 좀더 직관적으로 인식하기 위해 유도트리 (또는 파스 트리 (Parse Tree), 또는 구문 트리(Syntax Tree)) 로 표현하면 다음과 같다.

            E
          / | \   
         E  +  E
         |   / | \
         N   E * E
         |   |   |
        D(3) N   N
             |   |
            D(4) D(5)


모호성 문제
-------------
위 유도 및 파스트리를 이용한 방범은 사실 두가지로 해석 될수 있다.

해석 가능한 또다른 형태의 Parse Tree

              E
            / | \   
           E  *  E
         / | \   |
        E  +  E  N
        |     |  |
        N     N  D(5)
        |     |
       D(3)  D(4)

이 경우 다른 결과값 (3 + 4) * 5 = 35 가 계산된다.

#### 결론
*
