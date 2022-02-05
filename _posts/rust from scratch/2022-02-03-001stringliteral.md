---
title: string literal(&str)
date: 2022-02-03
category: Rust from Scratch
tags : [rust, string literal, &str, String]
layout: post
---

Rust 를 배우면서 여러가지로 개념과 사용법을 배우는 것이 어려웠는데, 그중 하나는 문자열 처리 였다.

이번 기회를 통해 이를 정리해 보고자 한다.

--------------------------------------------------------------------------------------
문자열 관련, rust 에는 두가지 자료형이 존재한다.

* Dynamically Sized Type (DST) : String
* Fixed Sized Type : string literal (&str)

String type 의 경우, 해당 Data 의 길이을 고정할 수 없는 경우(mutable)가 발생하므로 저장 공간을 유동적으로 확보할 수 있는 Heap 영역에 저장 한다.  그 밖의 모든 DST type (vector, Slice, String, trait object) 은 모두 Heap 영역을 사용한다.

그렇다면 string literal 은 stack 에 저장되는 건가?
문자열의 길이를 변경할 수 없는건가?
우선 rust 에서 문자열을 결합하는 예제를 살펴보자.

CDOE 1
```rust
fn add_to_String() {
    let mut hello = "hello ".to_string();
    let world = "world!";
    hello += world;
    println!("{}", hello); // 결과 : "hello world!"
    }
fn add_to_str() {
    let mut hello = "hello ";
    let world = "world!";
    hello += world;  
    // error! : binary assignment operation `+=` cannot be applied to type `&str`

    let hello_world = hello + world;
    // error! : cannot add `&str` to `&str`
    // `+` cannot be used to concatenate two `&str` strings
}
fn main() {
    add_to_String();
    add_to_str();
}
```
* String 형 변수 에 &str 형 data 를 추가하는 것은 문제 없이 실행된다.  

* 동일한 작업을 &str 형에 시도했을 때, 실행되지 않는다.

* &str 변수를 선언 하면서, 두 &str 형 변수를 더하는 것도 실행되지 않는다.



위 Code 에서 확인했을 때, string literal 형 변수는 문자열 변경이 불가능하므로, 문자열을 결합하기 위해서는 mutable String 형으로 변환해서 사용해야 한다.

그러다면 다음과 같이 생각해볼 수 있다.


-----------------------------------------------------------------------
### 1.string literal 은 mutable 선언 및 사용이 불가능 한가?

그렇제 않다. mutable 로 사용 가능하다.  

CODE 2
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
CODE 1 에서 mut &str 변수에 문자열을 추가할려고 할 때 에러가 발생했던 것과 달리, 내용도, 길이도 변경이 가능하다. 기존의 문자열에 추가 할 수는 없지만, 변경은 할 수 있다.

뭔가 더 복잡해진듯 하다. 이를 해결하기 위해 약간 원론적인 질문을 던져보자.  


-----------------------------------------------------------------------
### 2.string literal type 은 stack 에 저장되는가? heap 에 저장되는가?

찾아본 내용에 의하면, 둘 다 아니다.

>String literals are not stored in the heap or the stack, they are stored directly in your program’s binary. Literally embedded in the binary, and the reference is a reference to the location in the binary. [(원문 링크)](https://users.rust-lang.org/t/str-string-literals/29635)

string literal 의 경우, program memory 에 저장되어 있으며, 변수는 해당 memory 의 주소를 가져오게 된다.

이 내용이 맞다면, 첫번째 질문과 CODE 1, 2 에서 확인한 모순적인 결과에 대해 다음과 같이 해석이 가능해 진다.

* string literal 의 값은 program memory 에 code 와 같이 저장되어 있기 때문에, 직접 수정은 불가능하며, 참조자로 접근만 가능하다.

* mutable string literal 변수가 CODE 2 에서 값이나 길이를 변경한 문자열 값을 가지게 된 것은, Value 자체를 변경한 것이 아니라, 참조자로써 수정된 문자열의 메모리 주소로 참조 위치를 변경한 것이다.



이 내용을 다시 한번 증명하기 위해 다음 code 를 확인해 보자. [(code 원문 링크)](https://users.rust-lang.org/t/why-am-i-able-to-mutate-a-string-literal/39778/16)

CODE 3
```rust
fn main()
{
    let mut x: &str = "hello";
    let     y: &str = "world";

    // (1) mutable string literal x 와 immutable staring literal y 의 값을
    //     주소값 형태로 변환하여 x_addr 과 y_addr 변수에 저장.
    let x_addr = x.as_ptr() as usize;
    let y_addr = y.as_ptr() as usize;

    // 주소가 정상적으로 출력되는지 확인
    // 두 주소값의 차이가 5 (hello 길이) 인것으로 보아, 동일 영역의 메모리를 연속적으로 사용했으며,
    // 각각 문자열의 첫번째 위치를 가지고 있다고 예상됨.
    println!( "Address of x ({}) = 0x{:2x}", x, x_addr ); // 결과 : Address of x (hello) = 0x7ff78941d370
    println!( "Address of y ({}) = 0x{:2x}", y, y_addr ); // 결과 : Address of y (world) = 0x7ff78941d375

    // (2) (1) 조건을 assert! 메크로를 사용해서도 확인 가능.
    //     다만 메모리를 연속적으로 사용하는 것을 완벽하게 보장할 수 없기 때문에(특히 playgroud 같은 web 기반에서는),
    //     예외 발생 시, panic 상태로 빠지게 만들 수 있다.
    //     (WARNING: the following is undefined behavior.)
    assert!( x_addr+5 == y_addr );    

    // (3) x 에 y 값 대입 후, 해당 값을 value 형식과 주소값 형식로 출력.
    x = y;

    // x 에 y가 가지고 있는 메모리 주소값 그대로 가져왔음을 다시 확인
    println!( "Address of x ({}) = {:p}", x, x );         // 결과 : Address of x (world) = 0x7ff78941d375

    // (4) x_addr 이 pointer 일 때, 가리키는 위치부터 일정 범위까지 값을 직접 가져올 수 있는
    //     std::slice::from_raw_part() 함수를 사용, byte 형태로 배열에 저장.
    //     그 후, std::str::from_utf8() 함수로 해당 배열 값을 문자로 다시 복원.
    let bytes: &[u8]        = unsafe { std::slice::from_raw_parts(x_addr as *const u8, 10 ) };
    let new  : &'static str = std::str::from_utf8( bytes ).expect( "valid UTF" );

    // 변수 x 자체는 이제 "world" 의 위치를 가리키고 있지만, x_addr 가 기존 메모리 주소를 저장하고 있으므로
    // 이를 통해 "helloworld" 를 모두 다시 가져올 수 있음.
    println!( "Address of new ({}) = {:p}", new, new );   // 결과 : Address of new (helloworld) = 0x7ff78941d370
}
```
결과
```rust
Address of x (hello) = 0x7ff78941d370  
Address of y (world) = 0x7ff78941d375       
Address of x (world) = 0x7ff78941d375       
Address of new (helloworld) = 0x7ff78941d370
```

#### 결론
 * string literal 변수는, 값을 가지고 있는 것이 아니라, pointer 이다. 따라서, mutable 로 사용 가능하다. (pointer 주소를 새로운 문자열이 저장된 위치로 변경만 해주면 되므로)

 * 자료형 &str 에서 str 이 string slice 의미하므로, 명시적으로 (어딘가에 저장되어 있는) 문자열 slice 의 참조자 인 것이다.


위 질문 이외 string literal 을 사용할 때, 고려해야할 사항을 추가로 알아보고자 한다.

-----------------------------------------------------------------------
### 3.string literal 의 lifetime

구조체 멤버로 문자열를 사용하고자 할때, 만약 &str type 을 사용하고자 한다면, 다음과 같이 에러가 발생한다.

CODE 4
```rust
struct  Person {
    name : &str,   // &str 사용불가
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
결과
```rust
missing lifetime specifier
expected named lifetime parameter
```
구조체 멤버로 참조형은 쓸 수 없다. struct 가 사용되는 기간동안 original data 의 lifetime 을 보장할 수 없기 때문이다.
stack 에 저장된 자료형(ex. i32, f64)의 경우는 값을 복사하여 가져올 수 있고, heap 영역에 저장된 자료형의 경우, ownership 을 가져오면 되므로 문제가 발생하지 않는다. 그러나 string literal 의 경우, 해당 자료형은 명시적으로(&) 참조형이며, heap 영역에 저장된 것도 아니므로, ownership 을 가져올 수도 없다.

이를 해결하는 방법은 크게 두가지로 생각 해볼 수 있다.

#### 3.1 'static lifetime parameter 사용

최초의 입력된 string literal 을 &'static str 로 선언해 주든지, 구조체에서 받을 때 &'static str 로 변환해서 받을 경우 해당 문제가 발생되지 않는다.
CODE 5
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
    let mate = Mate{person1 : mike, person2 : john};

    mate.show_mate();
}
```
결과
```rust
person1 : Mike
person2 : John
```


이와 관련하여, 공식 문서에 다음과 같이 언급되어 있다. [(원문 링크)](https://rinthel.github.io/rust-lang-book-ko/ch10-03-lifetime-syntax.html)

>#### 정적 라이프타임(Static lifetime)
우리가 논의할 필요가 있는 특별한 라이프타임이 딱 하나 있습니다: 바로 'static입니다. 'static 라이프타임은 프로그램의 전체 생애주기를 가리킵니다. 모든 스트링 리터럴은 'static 라이프타임을 가지고 있는데, 아래와 같이 명시하는 쪽을 선택할 수 있습니다:
```rust
let s: &'static str = "I have a static lifetime.";
```
>이 스트링의 텍스트는 여러분의 프로그램의 바이너리 내에 직접 저장되며 여러분 프로그램의 바이너리는 항상 이용이 가능하지요. 따라서, 모든 스트링 리터럴의 라이프타임은 'static입니다.

> 여러분은 어쩌면 에러 메시지 도움말에서 'static 라이프타임을 이용하라는 제안을 보셨을지도 모릅니다만, 참조자의 라이프타임으로서 'static으로 특정하기 전에, 여러분이 가지고 있는 참조자가 실제로 여러분 프로그램의 전체 라이프타임 동안 사는 것인지 대해 생각해보세요 (혹은 가능하다면 그렇게 오래 살게끔 하고 싶어 할지라도 말이죠). 대부분의 경우, 코드 내의 문제는 댕글링 참조자를 만드는 시도 혹은 사용 가능한 라이프타임들의 불일치이며, 해결책은 이 문제들을 해결하는 것이지 'static 라이프타임으로 특정하는 것이 아닙니다.


lifetime 을 보장할 수 없는 자료형이기 때문에 &str 자체로 구조체에서 사용할 수 없지만, program binary 로 존재하기 때문에 언제나 static lifetime 을 가진다는... 아이러니한 구조이다.   


-----------------------------------------------------------------------
### 4. &str 형을 String 형으로 변경

아래와 같이, 총 4가지 방법으로 형변환을 할 수 있음을 찾았다. 각각에 대해 간단하게 알아보자.

CODE 6
```rust
use std::any::type_name;

fn type_of<T> (_:T) -> &'static str {
    type_name::<T>()
}
fn main()
{
    let a = "Hello";

    let a_string_from = String::from(a);
    let a_to_string = a.to_string();
    let a_to_owned = a.to_owned();
    let a_into : String = a.into();

    println!("type of a            : {}", type_of(a));
    println!("type of a_string_from: {}", type_of(a_string_from));
    println!("type of a_to_string  : {}", type_of(a_to_string));
    println!("type of a_to_owned   : {}", type_of(a_to_owned));
    println!("type of a_into       : {}", type_of(a_into));
}
```
결과
```rust
type of a            : &str
type of a_string_from: alloc::string::String
type of a_to_string  : alloc::string::String
type of a_to_owned   : alloc::string::String
type of a_into       : alloc::string::String
```
참고로 Crate alloc 대해 간단히 확인해 보면, [(원문 링크)](https://doc.rust-lang.org/alloc/index.html)

>This library provides smart pointers and collections for managing heap-allocated values.

>This library, like libcore, normally doesn’t need to be used directly since its contents are re-exported in the std crate. Crates that use the #![no_std] attribute however will typically not depend on std, so they’d use this crate instead.

#### 4.1 to_string()

CDOE 7 (주석으로 각 출력값 표시)
```rust
fn main() {
    println!("{}", "a".to_string());    // a
    println!("{}", "안녕".to_string()); // ?덈뀞 (깨지는 것은 OS 또는 IDE 때문으로 판단됨)
    println!("{}", 'a'.to_string());    // 공백 출력
    println!("{}", 1.to_string());      // 1
    println!("{}", 1.1.to_string());    // 1.1
    println!("{}", true.to_string());   // true   
}
```
해당 method 는 Trait std::string::ToString 에 정의 되어 있으며, 이미 다양한 자료형에 대해 String 자료형으로 return 하도록 implementors 가 정의 되어 있기 때문에 숫자에 적용해도 String 형을 얻을 수 있다. [(원문 링크)](https://doc.rust-lang.org/std/string/trait.ToString.html)

#### 4.2 String::from()

CDOE 8 (주석으로 각 출력값 표시)
```rust
fn main() {
    println!("{}", String::from("a"));   // a
    println!("{}", String::from("안녕"));// ?덈뀞 (깨지는 것은 OS 또는 IDE 때문으로 판단됨)
    println!("{}", String::from('a'));   // 공백 출력
    println!("{}", String::from(1));     // 실행불가
    println!("{}", String::from(1.1));   // 실행불가
    println!("{}", String::from(true));  // 실행불가    
}
```
공식문서 [원문 링크](https://rinthel.github.io/rust-lang-book-ko/ch08-02-strings.html) 에 다음과 같이 언급되어 있다.

>String::from과 .to_string은 정확히 똑같은 일을 하며, 따라서 어떤 것을 사용하는가는 여러분의 스타일에 따라 달린 문제입니다.
>스트링이 UTF-8로 인코딩되었음을 기억하세요. 즉, 아래의 Listing 8-14에서 보는 것처럼 우리는 인코딩된 어떤 데이터라도 포함시킬 수 있습니다:

그러나 String::from() 을 사용한 경우, 문자열 타입일 때만 정상적으로 동작함을 볼수 있다.  이는 rust standard crate 에 대해 설명한 문서를 보면 차이점을 이해 할 수 있다.

to_string() 이 return 자료형을 String 형으로 변환하는데 특화된 method 라면, std::convert::From Trait 은 광의적으로 이미 작성된 implementors 에 명시된 모든 자료형을 변환할 수 있게 해주며, &str -> String (struct) 는 그 중 하나의 하나일 뿐이다.

(해당 내용은 아직 이해가 부족하여 추가 학습이 필요함.) [(원문 링크)](https://doc.rust-lang.org/std/convert/trait.From.html#tymethod.from)

#### 4.3 to_owned()
method 명이 원래 다른 목적으로 만들어졌음을 추측하게 만든다. 공식 문서를 우선 살펴 보면,

> #### Trait std::borrow::ToOwned [(원문 링크)](https://doc.rust-lang.org/std/borrow/trait.ToOwned.html)

> fn to_owned(&self) -> Self::Owned
> Creates owned data from borrowed data, usually by cloning.

CDOE 9
```rust
use std::any::type_name;
fn type_of<T> (_:T) -> &'static str {
    type_name::<T>()
}
fn main() {
    let s: &str = "a";
    let ss: String = s.to_owned();
    println!("type of s : {}    ->  ss : {}", type_of(s), type_of(ss));

    let v: &[i32] = &[1, 2];
    let vv: Vec<i32> = v.to_owned();
    println!("type of v : {}  ->  vv : {}", type_of(v), type_of(vv));
}
```
결과
```rust
type of s : &str    ->  ss : alloc::string::String
type of v : &[i32]  ->  vv : alloc::vec::Vec<i32>
```
to_owned() methods 는 참조로 가져온(borrowed) data 에 대해, 동일 데이터를 새로 생성해서 해당 변수에 ownership 을 생성해준다. 그래서 위와 같이, &str -> String 으로 &[i32] -> Vec<i32> 타입으로 변환된 새로운 data 를 갖도록 만들어 준다.

clone() 과 기능이 비슷하지만, clone() 은 동일 자료형 (ex. &str -> &str) 로 새 data 를 생성하는 것이기 때문에,  clone 을 사용한 string literal 형은 변환 작업을 한번 더 해줘야 한다.  


#### 4.4 into()

역시 공시문서를 확인하면 다음과 같이 명시되어 있다.

> #### Trait std::convert::Into [(원문 링크)](https://doc.rust-lang.org/std/convert/trait.Into.html)
> A value-to-value conversion that consumes the input value. The opposite of From.

앞에서 From 을 다음과 같이 사용하였다.

       let 변수:String = String::form("문자열"):
                  ㄴ> 자동 type 지정

그리고 이렇게 해서 형변환이 가능한 이유는, std::convert::From Trait 에 &str -> String implementor 가 이미 정의 되어 있기 때문이다.

From 에 impementing 된 자료형은 자동으로 Into 에도 implementor 가 생성된다고 명시되어 있다.
따라서 into() 를 사용하여 동일 작업을 할 수 있으며, 사용 방법은

       let 변수:String = "문자열".into();
                  ㄴ> 명시적으로 지정해줘야 함.


#### 4.5 결론
일반적으로 string literal 을 포함, 일반 데이터를 String 으로 형변환을 하려면, to_string() 을 사용하는 것이 가장 무난해 보인다.

From Trait 의 경우, 좀더 다양한 경우에 사용될 수 있다. (ex. i32 -> f64, char -> u32)
특히 rust 는 자동 형변환이 허용되지 않으므로, 이를 잘 사용해야 하며, 사용자가 임의로 작성한 struct 에 대해서도 implementor 를 추가하여 사용할 수 있을 듯 하다.

to_owned() 의 경우, borrowed type 의 ownership 을 해결할 수 있으므로, struct member 로 사용할 자료형이 DST 인 경우, 활용할 수 있을 듯 하다.

into() 의 경우, 아직 어떻게 활용할수 있을지, 잘 모르겠다. From 과 함께 추가로 학습이 필요해 보인다.
