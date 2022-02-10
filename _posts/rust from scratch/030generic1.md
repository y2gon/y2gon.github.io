---
title: Generic (1)
date: 2022-02-09
category: Rust from Scratch
tags : [rust, generic, type parameter,]
layout: post
---

Rust - Generic 관련, 공식문서 [(링크)](https://rinthel.github.io/rust-lang-book-ko/ch10-01-syntax.html) 만으로 이해가 어려워 [(The Ultimate Guide to Rust Programming)](https://www.educative.io/courses/ultimate-guide-to-rust-programming) 에서 학습한 내용을 바탕으로 이해, 정리했습니다.

-------------------------------------------------------------------------

### 1.Generic - Structs & Implementations

Rust 의 경우, 한글로 작성된 자료를 찾기가 어려워, 다른 언어에 비슷한 개념이 있는지 찾아보았다.

Java 관련 Post [자바 [JAVA] - 제네릭(Generic)의 이해](https://st-lab.tistory.com/153)  에서 유사한 개념을 발견했다. 그리고 해당 Post 에서 Java - Generic 장점을 다음과 같이 정리하였다.

>#### Generic(제네릭)의 장점 (Java)
>
> 1. 제네릭을 사용하면 잘못된 타입이 들어올 수 있는 것을 컴파일 단계에서 방지할 수 있다.
> 2. 클래스 외부에서 타입을 지정해주기 때문에 따로 타입을 체크하고 변환해줄 필요가 없다. 즉, 관리하기가 편하다.
> 3. 비슷한 기능을 지원하는 경우 코드의 재사용성이 높아진다.

Rust 에 class 는 없지만, 작업의 효율성을 위해 해당 역할을 하는 무언가가 분명히 필요할 것이다.

#### 1.1 Generic - Structs

만약 과일 가격을 구조체에 담기 위해 정의한다면, 다음과 같이 작성할 수 있다.

```rust
struct Fruit {
    apple: i32,
    banana: i32,
}
```
만약, 달러로도 과일 가격을 표시하고 싶다면, 각 member 의 자료형은 실수형인 것이 더 적절할 것이다. 이런 경우, 구조체는 한번만 선언하고, 실수형과 정수형 모두 사용할 수 있도록, 구조체를 일반화 할 수는 없을까?

이 때, Generic 를 사용하여 구조체를 정의 하면 다음과 같다.

```rust
struct Fruit<T> {
    apple: T,
    banana: T,
}
```
<strong>Generic data type <span style="color : red">T</span></strong> 를 사용하여, 모든 type 을 해당 구조체에서 사용할 수 있도록 명시해 주었다.

실제로 가능한지 아래의 code 를 통해 확인해 보자.

CODE 1
```rust
struct Fruit<T> {
    apple: T,
    banana: T,
}

fn main() {
    let fruit_won: Fruit<i32> = Fruit{
        apple : 2000_i32,
        banana : 1000_i32,
    };
    println!("an apple : {} won,\t a banana :{} won", fruit_won.apple, fruit_won.banana);

    let fruit_dallar: Fruit<f64> = Fruit{
        apple : 1.80_f64,
        banana : 0.75_f64,
    };
    println!("an apple : {} $,\t a banana :{} $", fruit_dallar.apple, fruit_dallar.banana);

    let fruit_QR: Fruit<String> = Fruit{
        apple : "N/A".to_string(),
        banana : "???".to_string(),
    };
    println!("an apple : {} QR,\t a banana :{} QR", fruit_QR.apple, fruit_QR.banana);
}
```
결과
```rust
an apple : 2000 won,     a banana :1000 won
an apple : 1.8 $,        a banana :0.75 $
an apple : N/A QR,       a banana :??? QR
```
각 인스턴트에서 지정한 data type 에 맞게 작동함을 볼 수 있다.

Generic 은 첫문자는 대문자로 표기하며, 만약 구조체 member 가 각각 다른 data type 으로 가져야 한다면, 이를 다르게 표기 해야 한다.

CODE 2
```rust
struct Fruit<Apple, Banana> {
    apple: Apple,
    banana: Banana,
}
fn main() {
    let fruit1: Fruit<i32, f64> = Fruit{
        apple : 2000_i32,
        banana : 0.75_f64,


```
결과
```rust
an apple : 2000 won,             a banana :0.75 $
an apple : on sale(true),        a banana :Sold out
```
#### 1.2 Generic - Implementations

위 code 에서 화면 출력하는 기능을 함수로 따로 작성하여, 필요할 때마다 호출하는 방법으로 하면 더 효율적일 것이다. 그런데, 일반 함수 표현으로는 실행이 되지 않는다. 구조체에서 정의된 generic 에 대해 함수에는 어떻게 적용해야 할지 관계가 성립되지 않았기 때문이다.

rust 문법에 따라 관계를 표시하여 실행한 code 는 다음과 같다.

CODE 3
```rust
struct Person<Name, Age>{
    name: Name,
    age: Age,
}
impl<T> Person<String, T> {                 // (1)
    fn greet(&self){                        // (2)
        println!("Hello, {}", self.name);   
    }
}
fn main() {
    let alice: Person<String, u32> = Person {
        name: "Alice".to_string(),
        age: 30_u32,
    };
    alice.greet();                          // (3)

    let bob: Person<String, u64> = Person {
        name : "Bob".to_string(),
        age : 35_u64,
    };
    bob.greet();
}
```
결과
```rust
Hello, Alice
Hello, Bob
```
(2) 표시의 함수가 (1) 표시의 <strong>impl(implementation) block</strong>에 의해 묶여 있다.

함수에도 generic 알리고, 각 data type 에 맞게 함수가 동작할 수 있도록 하는 조치가 필요하다. 따라서 하나의 함수는 generic 에서 사용할 data type 에 맞게 세분화된 정의를 필요로 한다. [(function - method - implementation 정리 링크)]({{site.baseurl}}/rust%20from%20scratch/0280methods.html)

잠시 여기에서 용어 정리가 필요해 보인다.

해당 Post 에서 지금까지 함수 실제적으로 사용하지 않았지만, 앞으로 사용할 함수, 특히




CODE 1
```rust
```
결과
```rust
```

CODE 1
```rust
```
결과
```rust
```


-------------------------------------------------------------------------


Monomorphization(https://en.wikipedia.org/wiki/Monomorphization)

In programming languages, monomorphization is a compile-time process where polymorphic functions are replaced by many monomorphic functions for each unique instantiation.
