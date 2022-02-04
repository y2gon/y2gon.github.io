---
title: string literal(&str)
date: 2022-02-03
category: Rust from Scratch
tags : [rust, string literal, &str, String]
layout: post
---

Rust 를 배우면서 여러가지로 개념과 사용법을 배우는 것이 어려웠는데, 그중 하나는 문자열 처리 였다. 특히 파이썬과 비교하면...

이번 기회를 통해 이를 정리해 보고자 한다.

문자열에 대해서 rust 에는 두가지 자료형이 존재한다.

* Dynamically Sized Type (DST) : String
* Fixed Sized Type : string literal (&str)

mutable 로 선언된 문자열의 경우, 그 길이를 고정시킬 수 없으므로 Heap 영역에 저장해야 한다.  (DST in rust: vector, Slice, String, trait object)

그렇다면, 길이가 고정된 string literal 에 대해 몇가지 질문이 생긴다.

### 1.string literal 은 mutable 선언 및 사용이 불가능 한가?

다음과 같이, mutable 로 사용 가능하다.  

```rust
fn main() {
  let mut fixed:&str = "abc";
  println!("{}", fixed);
  fixed = "xyz123";
  println!("{}", fixed);
}
```
결과
```rust
abc   
xyz123
```
내용도, 길이도 변경이 가능하다. 이건 또다른 질문을 불러온다.

### 2.string literal 의 길이가 변경 가능 하다면, stack 에 저장되는가? heap 에 저장되는가?

찾아본 내용에 의하면, 둘 다 아니다.

>String literals are not stored in the heap or the stack, they are stored directly in your program’s binary. Literally embedded in the binary, and the reference is a reference to the location in the binary. [(링크)](https://users.rust-lang.org/t/str-string-literals/29635)

string literal 의 경우, program memory 에 저장되어 있으며, 변수는 해당 memory 의 주소를 가져오게 된다.

위 두 질문에 대해 아래 code 를 통해 확인해 보자. [(아래 code 원문)](https://users.rust-lang.org/t/why-am-i-able-to-mutate-a-string-literal/39778/16)

```rust
fn main()
{
    let mut x: &str = "hello";
    let     y: &str = "world";

    let x_addr = x.as_ptr() as usize;
    let y_addr = y.as_ptr() as usize;

    // (1)
    println!( "Address of x ({}) = 0x{:2x}", x, x_addr );
    println!( "Address of y ({}) = 0x{:2x}", y, y_addr );

    // (2)
    assert!( x_addr+5 == y_addr ); // WARNING: the following is undefined behavior.   

    // (3)
    x = y;
    println!( "Address of x ({}) = {:p}", x, x );

    // (4)
    let bytes: &[u8]        = unsafe { std::slice::from_raw_parts(x_addr as *const u8, 10 ) };
    let new  : &'static str = std::str::from_utf8( bytes ).expect( "valid UTF" );

    println!( "Address of new ({}) = {:p}", new, new );
}
```
결과
```rust
Address of x (hello) = 0x7ff78941d370      
Address of y (world) = 0x7ff78941d375       
Address of x (world) = 0x7ff78941d375       
Address of new (helloworld) = 0x7ff78941d370
```
  (1) mutable string literal x 와 immutable staring literal y 가 가지고 있는 값을 주소값 형태로 변환하여 x_addr 과 y_addr 변수에 저장 하였다.
      * 결과 1, 2 번째 줄에서 변수 x, y 값이 주소값 형태로 정상적으로 출력됨을 확인할 수 있다.
      * y_addr 값이 x_addr 의 +5 이다. 그 이유가 hello 의 문자 길이가 5 이기 때문에 동일 영역의 메모리를 연달아 사용했기 때문이라고 생각해볼 수 있다.
      => 아직 100% 확정 할 수 없지만, x, y 가 가지고 있던 것은, 문자열 자체가 아니라, 문자열이 저장되어 있는 어딘가를 가리키는 주소값일 것이라 가정해볼 수 있다.

  (2) 1 번 조건을 assert! 메크로를 사용해서도 확인해볼 수 있다. 다만 +5 는 조건이 완벽하게 보장할 수 없기 때문에(특히 playgroud 같은 web 기반에서는), 이런 예외의 경우, assert! 를 사용하면 제대로 실행되지 않을 수도 있다.

  (3) x 값을 y 로 변경한 후, x 가 가지고 있는 값을 value 형식과 주소값 형식로 출력해 보았다.
      * 결과 3번째 줄과 같이, 단순히 value 형식 결과는 문자열 값이지만, 이를 주소값 형식으로 출력 했을 때, 기존 y가 가지고 있는 메모리 주소값 그대로 가져왔음을 다시 확인 할 수 있다.

  (4) 이번에는 x_addr 이 pointer 일 때, pointer 위치의 value 값을  일정 길이까지 가져올 수 있는 std::slice::from_raw_part() 함수를 사용하여, byte 형태로 배열에 저장한다. 그리고 이를 std::str::from_utf8() 함수로 문자 형태로 다시 복원한다.
      * 변수 x 자체는 이제 "world" 의 위치를 가리키고 있지만, 기존 주소값을 그대로 가지고 있는 x_addr 의 값을 통해서 "helloworld" 를 모두 다시 가져올 수 있었다.

#### 결론
 * 위 code 를 통해서 string literal 변수를 선언했을 때, 해당 변수는 해당 값을 그대로 가지고 있는 것이 아니라, pointer 로 작동하고 있음을 알 수 있다. 따라서, mutable 로 선언 후, 해당 문자열을 변경이 가능해진다.
 * 자료형 &str 에서 str 이 string slice 의미하므로, 명시적으로 (어딘가에 저장되어 있는) 문자열 slice 의 참조자 인 것이다.  
 * 다만 위 과정을 통해서 string literal 자체가 어디에 저장되는지는 확인할 수 없다.

-----------------------------------------------------------------------
위 질문 이외 string literal 을 사용할 때, 고려해야할 사항을 추가로 알아보고자 한다.

### 3.string literal 의 lifetime

구조체 멤버로 문자열 형식을 사용하고자 할때, 만약 String 이 아닌 &str 을 사용하고자 한다면, 다음과 같이 에러가 발생한다.

```rust
struct  Person {
    name : &str,
}
impl  Person {
    fn show_person (&self){
        println!("{}", self.name);
    }
}
fn main()
{
    let person = Person{
        name : "Mike".to_string(),
    };

    person.show_person();
}
```
```rust
missing lifetime specifier
expected named lifetime parameter
```
구조체 멤버로 사용될 자료형은 참조형은 허용되지 않는다. struct 가 사용되는 기간동안 original data 의 lifetime 을 보장할 수 없기 때문이다.
stack 에 저장된 자료형의 경우는 값을 복사하여 가져올 수 있고, heap 영역에 저장된 자료형의 경우, ownership 을 가져오면 되므로 해당 문제가 발생하지 않는다. 그러나 string literal 의 경우, 해당 자료형은 명시적으로(&) 참조형이며, heap 영역에 저장된 것도 아니므로, ownership 을 가져올 수도 없다.

이를 해결하는 방법은 크게 두가지로 생각 해볼 수 있다.

#### 1. 'static lifetime parameter 사용

최초의 입력된 string literal 을 &'static str 로 선언해 주던지, 구조체에서 받을 때 &'static str 로 변환해서 받을 경우 해당 문제가 발생되지 않는다.
```rust
struct  Mate {
    person1 : &'static str,  // 기존 &'static str 로 선언된 변수를 받음.
    person2 : &'static str,  // &str 로 선언된 변수를 static lifetime 조건으로 받음
}
impl  Mate {
    fn show_mate (&self){
        println!("person1 : {}", self.person1);
        println!("person2 : {}", self.person2);
    }
}
fn main()
{
    let mike:&'static str = "Mike";
    let john = "John";

    let mate = Mate{
        person1 : mike,
        person2 : john,
    };

    mate.show_mate();
}
```
```rust
person1 : Mike
person2 : John
```


이와 관련하여, 공식 문서에 다음과 같이 언급되어 있다. [(링크)](https://rinthel.github.io/rust-lang-book-ko/ch10-03-lifetime-syntax.html)

>정적 라이프타임(Static lifetime)
우리가 논의할 필요가 있는 특별한 라이프타임이 딱 하나 있습니다: 바로 'static입니다. 'static 라이프타임은 프로그램의 전체 생애주기를 가리킵니다. 모든 스트링 리터럴은 'static 라이프타임을 가지고 있는데, 아래와 같이 명시하는 쪽을 선택할 수 있습니다:

```rust
let s: &'static str = "I have a static lifetime.";
```
>이 스트링의 텍스트는 여러분의 프로그램의 바이너리 내에 직접 저장되며 여러분 프로그램의 바이너리는 항상 이용이 가능하지요. 따라서, 모든 스트링 리터럴의 라이프타임은 'static입니다.

> 여러분은 어쩌면 에러 메시지 도움말에서 'static 라이프타임을 이용하라는 제안을 보셨을지도 모릅니다만, 참조자의 라이프타임으로서 'static으로 특정하기 전에, 여러분이 가지고 있는 참조자가 실제로 여러분 프로그램의 전체 라이프타임 동안 사는 것인지 대해 생각해보세요 (혹은 가능하다면 그렇게 오래 살게끔 하고 싶어 할지라도 말이죠). 대부분의 경우, 코드 내의 문제는 댕글링 참조자를 만드는 시도 혹은 사용 가능한 라이프타임들의 불일치이며, 해결책은 이 문제들을 해결하는 것이지 'static 라이프타임으로 특정하는 것이 아닙니다.



lifetime 을 보장할 수 없는 자료형이기 때문에 &str 자체로 구조체에서 사용할 수 없지만, program binary 로 존재하는 string literal 이기 때문에 언제나 static lifetime 을 가진다는... 아이러니한 설명이다.
