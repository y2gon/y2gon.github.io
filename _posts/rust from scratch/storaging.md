---
title: Casting
date: 2022-02-03
category: Rust from Scratch
tags : [rust, Casting, type cast]
layout: post
---



Storage of structs in memory
https://users.rust-lang.org/t/storage-of-structs-in-memory/69916/2

Data Layouturl
Memory representations of common data types.
https://cheats.rs/#data-layout


```Rust
#![allow(non_snake_case)]
struct  Person {
    name : String,
    age : i32,
}
impl  Person {
    fn show_person (&self){
        println!("struct Name : {}  ({:p})", self.name, &self.name);
        println!("strauct Age  : {}  ({:p})", self.age, &self.age);
    }
}
fn main()
{
    let mike = "Mike".to_string();
    let mike_age = 30;

    println!("main Name : {} ({:p})", mike, &mike);
    println!("main Age  : {} ({:p})", mike_age, &mike_age);

    let person = Person{
        name : mike,
        age : mike_age,
    };

    person.show_person();

    // println!("main Name : {} ({:p})", mike, &mike);
    // println!("main Age  : {} ({:p})", mike_age, &mike_age);

}
```

```Rust
main Name : Mike (0x1b4cd7f990)
main Age  : 30 (0x1b4cd7f9ac)     
struct Name : Mike  (0x1b4cd7fa80)
strauct Age  : 30  (0x1b4cd7fa98)
```


```rust
#![allow(non_snake_case)]
struct  Mate {
    person1 : &'static str,
    person2 : &'static str,  // &str 로 선언된 변수를 'st
}
impl  Mate {
    fn show_mate (&self){
        println!("struct Name1 : {} ({:p})", self.person1, &self.person1);
        println!("struct Name2 : {} ({:p})", self.person2, &self.person2);
    }
}
fn main()
{
    let mike:&'static str = "Mike";
    let john = "John";

    println!("main Mike    : {} ({:p})", mike, &mike);
    println!("main John    : {} ({:p})", john, &john);

    let mate = Mate{
        person1 : mike,
        person2 : john,
    };

    mate.show_mate();

    // println!("main Name : {} ({:p})", mike, &mike);
    // println!("main Age  : {} ({:p})", mike_age, &mike_age);

}
```
```rust
main Mike    : Mike (0x51d3affa18)
main John    : John (0x51d3affa28)
struct Name1 : Mike (0x51d3affb08) // 왜 구조체로 가져가면 다 메모리 주소가 변하는지 모르겠음.
struct Name2 : John (0x51d3affb18) //
```
