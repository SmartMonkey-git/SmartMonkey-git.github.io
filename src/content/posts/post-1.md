---
title: "Schrödinger's Error"
description: "meta description"
date: 2022-04-04T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["programming"]
authors: ["Rouven Reuter"]
tags: ["rust", "python"]
draft: false
---
#### Disclaimer
This post is still under construction, but since I have not posted anything, yet I used it as a test.

If Schrödinger designed an error handling system, it would be similar to Rust's. Where boxing return values, and only knowing if your application is dead or alive when checking them, is common practice. It's Schrödinger's thought experiment cast in software. What sounds like a devastating business risk for every mail-order company is, in fact, clever error propagation.
In this post, we will mimic the error handling of the Rust programming language with Python. This article is useful for Pythonists getting into Rust.
#### How Rust handles errors
Before diving into the code, let's understand how Rust manages errors. Rust has two mechanisms to raise problems - panic! and Result. 
Using the former will force your application to exit with no ifs, no buts, no try, no except. Once called, it can not be caught. Technologically speaking, a panicking thread will unwind its stack ==and in the context of a multithreaded program only the affected thread will do so.== For simplicity, we will deal with single thread examples. While there is no keyword equivalent in Python, the behaviour can be imitated with sys.exit(msg).

Now, you might be saying: "This does not align with your Schrödinger analogy". You are right, but Rust's Result enum does. Result, while not the familiar try-catch mechanism, lets you handle errors with grace. It comes in two variants of Ok and Err. Ok marks the return of your function as success, while Err indicates the opposite. Both wrap the return value and appear as a Result object to the receiving function, that needs to ascertain the content. 
Which brings us back to Schrödinger, your errors are packaged, and you got to check whether the cat is dead or not. Result offers numerous ==ways== to make that check, let me unbox the ones relevant for this post. The unwrap() function is by far the one most unlikely to make it into production code, when called it will either return a value if Ok or panic and unwind the thread. Unwrapping gracefully to avoiding panics is possible with the ?-operator. Applied to an Ok it acts the same as unwrap, however Err's will lead to an early return, propagating them. Lastly, is_ok() and is_err() let you see inside the Result before opening it, returning a corresponding boolean.
So much for the error propagation basics in Rust. 
#### Result in Python

```python
def panic(msg: str):
    sys.exit(msg)
    
class Result(ABC):  
    def __init__(self, result: Union[Any, Exception]):  
        self.result = result  
  
    def unwrap(self) -> Any:  
        if self.is_ok():  
            return self.result  
        else:  
            panic(str(self.result))  
  
    def question_mark(self) -> Any:  
        if self.is_ok():  
            return self.result  
        else:  
            raise self.result  
  
    def is_ok(self) -> bool:  
        return isinstance(self, Ok)  
  
    def is_err(self) -> bool:  
        return isinstance(self, Err)

class Ok(Result):  
	pass  

class Err(Result):  
	pass
```



- Mention Panic and say that you should not use it and that there is not equivalent to it in python.
- Explain how python handles errors
- Explain how Rust handles errors
- Compare both
- Emulate Result class in python

Nice thing about results, is that you see the possible error in the interface of a function.


```python
import random

class Cat:  
    def __init__(self, name: str):  
        self.name = name  

class CatDeadError(Exception):  
    pass  
  
class SchrodingersExperiment:  
    def __init__(self, cat: Cat):  
        self.cat = cat  
  
    def run(self) -> Result:  
        if random.random() > 0.5:  
            return Ok(self.cat)  
        else:  
            return Err(CatDeadError(f"{self.cat.name} died miserably."))  
  
def test_result():  
    cat_names = [  
        "Whiskers",  
        "Socks",  
        "Bubbles",  
        "Shadow",  
        "Ginger",  
        "Oreo",  
        "Tiger",  
        "Bella",  
        "Luna",  
        "Simba",  
    ]  
    cat = Cat(random.choice(cat_names))  
    SchrodingersExperiment(cat).run().unwrap()
```

```rust
struct Cat {  
    name: String,  
}

#[derive(Error, Debug, PartialEq)]  
pub enum CatDeadError {  
    #[error("{0} died miserably.")]  
    Dead(String),  
}

struct SchrodingersExperiment {  
    cat: Cat,  
}  
  
impl SchrodingersExperiment {  
    fn run(&self, cat: Cat) -> Result<Cat, CatDeadError> {  
        let mut rng = rand::thread_rng();  
        let rand_float: f64 = rng.gen();  
        if rand_float > 0.5 {  
            Ok(cat)  
        } else {  
            Err(CatDeadError::Dead(cat.name))  
        }  
    }  
}
```
-----
Sources
https://doc.rust-lang.org/rust-by-example/std/panic.html
https://doc.rust-lang.org/std/result/
https://doc.rust-lang.org/rust-by-example/std/result/question_mark.html
