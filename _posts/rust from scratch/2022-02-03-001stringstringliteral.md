---
title: (Rust) string literal(&str)
date: 2022-02-03
category: Rust from Scratch
tags : [rust, concatenating, string]
layout: post
---

Rust 를 배우면서 여러가지로 개념과 사용법을 배우는 것이 어려웠는데, 그중 하나는 문자열 처리 였다. 특히 파이썬과 비교하면...

이번 기회를 통해 이를 정리해 보고자 한다.

문자열에 대해서 rust 에는 두가지 자료형이 존재한다.

* Dynamically Sized Type (DST) : String
* Fixed Sized Type : string literal (&str)

mutable 로 선언된 문자열의 경우, 그 길이를 고정시킬 수 없으므로 Heap 영역에 저장해야 한다. (이외 DST : vector, Slice, trait object)

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

string literal 의 경우, program memory 에 bytes 형태로 저장되어 있으며, 이를 변수로 선언했을 때, 해당 변수는 해당 memory 의 주소를 가져오게 된다.

위 두 질문에 대해 아래 code 를 통해 확인해 보자. [(아래 code 에 대한 링크)](https://users.rust-lang.org/t/why-am-i-able-to-mutate-a-string-literal/39778/16)

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
  - 결과 1, 2 번째 줄에서 변수 x, y 값이 주소값 형태로 정상적으로 출력됨을 확인할 수 있다.
  - y_addr 값이 x_addr 의 +5 이다. 그 이유가 hello 의 문자 길이가 5 이기 때문에 동일 영역의 메모리를 연달아 사용했기 때문이라고 생각해볼 수 있다.
  => 아직 100% 확정 할 수 없지만, x, y 가 가지고 있던 것은, 문자열 자체가 아니라, 문자열이 저장되어 있는 어딘가를 가리키는 주소값일 것이라 가정해볼 수 있다.

(2) 1 번 조건을 assert! 메크로를 사용해서도 확인해볼 수 있다. 다만 +5 는 조건이 완벽하게 보장할 수 없기 때문에(특히 playgroud 같은 web 기반에서는), 이런 예외의 경우, assert! 를 사용하면 제대로 실행되지 않을 수도 있다.

(3) x 값을 y 로 변경한 후, x 가 가지고 있는 값을 value 형식과 주소값 형식로 출력해 보았다.
  - 결과 3번째 줄과 같이, 단순히 value 형식 결과는 문자열 값이지만, 이를 주소값 형식으로 출력 했을 때, 기존 y가 가지고 있는 메모리 주소값 그대로 가져왔음을 다시 확인 할 수 있다.

(4) 이번에는 x_addr 이 pointer 일 때, pointer 위치의 value 값을  일정 길이까지 가져올 수 있는 std::slice::from_raw_part() 함수를 사용하여, byte 형태로 배열에 저장한다. 그리고 이를 std::str::from_utf8() 함수로 문자 형태로 다시 복원한다.
 - 변수 x 자체는 이제 "world" 의 위치를 가리키고 있지만, 기존 주소값을 그대로 가지고 있는 x_addr 의 값을 통해서 "helloworld" 를 모두 다시 가져올 수 있었다.

 * 결론
 - 위 code 를 통해서 string literal 변수를 선언했을 때, 해당 변수는 해당 값을 그대로 가지고 있는 것이 아니라, pointer 로 작동하고 있음을 알 수 있다. 따라서, mutable 로 선언 후, 해당 문자열을 변경이 가능해진다.
 - 다만 위 과정을 통해서 string literal 자체가 어디에 저장되는지는 확인할 수 없다. 이를 간접적으로 확인해볼 수 있는 방법은 string literal 의 lifetime 을 살펴 보는 것이다.  

### 3.string literal 의 lifetime



```Rust
```





우선 다음의 간단한 예제를 보면,

```Rust
fn dynamic(b:String){
    println!("{}", b);
}

fn main() {
    let b :String = "hi !!".to_string();
    dynamic(b);
    println!("{}", b);
}
```
```
borrow of moved value: `b`
value borrowed here after move
```


```Rust
fn literal(a:&str){
    println!("{}", a);
}

fn dynamic(b:String){
    println!("{}", b);
}

fn main() {
    let a:&str = "hello";
    literal(a);
    println!("{}", a);

    let b :String = "hi !!".to_string();
    dynamic(b);
    println!("{}", b);
}
```



해당 내용은 책 [Rust Standard Library Cookbook: Over 75 recipes to leverage the power of Rust](https://www.amazon.com/Rust-Standard-Library-Cookbook-leverage-ebook/dp/B07B8RJ8VJ/ref=sr_1_1?keywords=rust+standard+library+cookbook&qid=1643872378&sprefix=rust+standard+cookbo%2Caps%2C295&sr=8-1) 예제를 통해 학습한 내용임.

```Rust
fn moving() {
    let hello = "hello";
    let world = "world";
    let hello_world = hello.to_string() + world;
    println!("{}", hello_world);   
}
fn cloning(){

}
fn mutating(){

}


fn main() {
    moving();
    cloning();
    mutating();
}

```

```
cannot find value `helloWorld` in this scope
a local variable with a similar name exists: `hello_world`
```
