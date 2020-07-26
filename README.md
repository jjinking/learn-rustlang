# learn-rustlang
Notes and code that help me learn rust lang

```bash
# Create new project.
cargo new <project-name>

# Build binary and write it to <project-root>/target/debug/<project-name>
cargo build
# Compile with optimizations and write to <project-root>/target/release/<project-name>
# Rust will not include checks for integer overflow that cause panics - program exits with error.
# Instead, it will perform two's complement wrapping.
cargo build --release

# Shortcut for building binary and running it.
cargo run

# Check to see if code compiles, but no executable is outputted. Useful for quickly checking code.
cargo check

# Locally build and open documentation in browser for all project dependencies.
cargo doc --open
```

`String` is growable, UTF-8

*associated function* is analogous to *static member functions* in c++.

## Variables

variables are declared with `let`, while constants are declared with `const`.

*Constants* are similar to immutable variables, but different in that there is no way to make constants mutable.

Constants require type annotation.

Example:

```rust

const MAX_POINTS: u32 = 100_000;
```

### Tuples

- Different types in one tuple.
- Access positional elements with `.` operator followed by 0-based index.

```rust

fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
	
	// Destructuring/unpacking a tuple.
	let (x, y, z) = tup;
	
	let a = tup.0; // 500
	let b = tup.1; // 6.4
	let c = tup.2; // 1
}

```

### Array

- Elements must all be same type.
  - Fixed length, unlike vector, which can grow.
- Useful when you want to store data in the stack than the heap.

```rust

fn main() {
    let a = [1, 2, 3, 4, 5];
	
	// Annotating array type by `[type; size]`
	let b: [i32; 5] = [1, 2, 3, 4, 5];
	
	// Shorthand for initializing array with same value elements.
	let c = [3; 5];
	// equialent to:
	let c = [3, 3, 3, 3, 3];
	
	// Accesing elements.
	let first = a[0]; // 1
    let second = a[1]; // 2
	
	// Accessing out-of-bounds index will cause runtime error, not compile-time.
}

```

## Functions

- Use snake case naming: All letters are lowercase with underscores.
- Functions can call other functions even if they're defined below it.
- Must always define parameter types.
- Function bodies contain *statements* (no return value) and *expressions* (returns a value).
  - *Expressions* do not have semicolons at the end - adding semicolon makes it a *statement*.
- Return values do not need to be named, but type must be declared using `->`.
- Return early from a function using the `return` keyword. Otherwise, function returns last *expression*.

```rust

fn main() {

	// Call functions defined below with *argument* 5.
    another_function(5);
}

// Function takes *parameter* x of type i32.
fn another_function(x: i32) {
    println!("The value of x is: {}", x);
}

// Returns 5. No semicolon after 5.
fn five() -> i32 {
    5
}
```

## Control Flow

### Conditionals

- Conditional expressions must evaluate to bool, not 0 or nonzero like C++.

```rust

fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
	
	// Using `if else` as an expression to assign an i32 value to `number`.
	// In this case, the last expression in both `if` and `else` blocks must be the same type.
	let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {}", number);
}
```

### Loops

#### Infinite loop using `loop`

`break` can be used to break out of loop, with a return value.

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```

#### `while` loops

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number -= 1;
    }
}
```

### `for` loops

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }
	
	// Use Range syntax.
	for number in (1..4).rev() {
        println!("{}!", number);
    }
}
```

## Strings

```rust
// immutable string literal.
let s = "hello";

// mutable String type.
let s = String::from("hello");
s.push_str(" world!");
println!("{}", s);

```

## Ownership

When a variable goes out of scope (e.g. closing curly bracket), Rust calls a special function called `drop` which is like a destructor.

```rust
let s1 = String::from("hello");

// This copies just the  pointer, length, and capacity of the string which lives in the stack,
// not the actual string data which lives in the heap. This is similar to a shallow copy.
// It's also followed by invalidating s1, so this is actually called a "move" like in C++11.
let s2 = s1;
```	

To prevent *double free* error, s1 will no longer be valid so that it won't need to be freed when both s1 and s2 go out of scope.

By default, Rust never "deep" copies any heap data during variable assignment, but there is a `clone` method for explicitly invoking a deep copy.

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

// s1 is still valid.
println!("s1 = {}, s2 = {}", s1, s2);
```

Exception to the *move* assignment are scalar values which do not contain data in the heap, so it'll just do shallow copy and then it's done.

This actually compiles:

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

`Copy` trait vs `Drop` trait (see [book](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#stack-only-data-copy))

Passing values to a function has the same `move` or `copy` semantics as assignment.

Return values also have same semantis. (see [book](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#return-values-and-scope))

## References and Borrowing

Book [section](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#references-and-borrowing)

Reference does not take ownership.

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

References are immutable by default.

Marking function arguments as references with `&` is done both in function definition as well as when passing in variables to the function, unlike C++ where it's only defined in function definition. Same with `mut`.

There can be many immutable references to a value at the same time.

There can be only one mutable reference at a time.
For sequential code, use curly brackers to create a new scope to allow multiple mutable references, since they will not be modified simultaneously.

```rust
fn main() {
    let mut s = String::from("hello");

    {
        let r1 = &mut s;
    } // r1 goes out of scope here, so we can make a new reference with no problems.

    let r2 = &mut s;
}
```

A mutable reference cannot coexist with any other references to the same value, even if the others are immutable.

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!("{}, {}, and {}", r1, r2, r3);
}
```

This will compile because the compiler knows the last usage of the immutable ref is before the mutable ref is created.

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
println!("{} and {}", r1, r2);
// r1 and r2 are no longer used after this point

let r3 = &mut s; // no problem
println!("{}", r3);
```

Rust compiler prevents dangling references (see [book](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#dangling-references))
Example, a function that returns a reference to a `String` that is created inside a function.

Rule: There can either be one mutable ref or many immutable refs at the same time.

## Slice Type

String literals are slices that reference const strings persisting in the compiled binary.

```rust
fn main() {
    let my_string = String::from("hello world");

    // first_word works on slices of `String`s
    let word = first_word(&my_string[..]);

	// Type is `&str`
    let my_string_literal = "hello world";

    // first_word works on slices of string literals
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

## Structs

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

let mut user1 = User {
	email: String::from("someone@example.com"),
	username: String::from("someusername123"),
	active: true,
	sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");

// *Field init shorthand* syntax provides a way to avoid writing field names twice.
fn build_user(email: String, username: String) -> User {
    User {
        email, // avoid email: email
        username,
        active: true,
        sign_in_count: 1,
    }
}

// *Struct update syntax* enables copying field values from another struct.
let user2 = User {
	email: String::from("another@example.com"),
	username: String::from("anotherusername567"),
	..user1 // copy the rest of the field values from user1
};
```

`Tuple Structs` are similar to structs, without the named fields. So they're basically like tuples, but give you the ability to name the type differently.

```rust
fn main() {
    struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);
	
	// `black` and `origin` are different types even though they're made of the same `i32` triplets.
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
	
	let (x, y, z) = origin;
	let r = black.0;
	let g = black.1;
	let b = black.2;
}

```

To use references in struct field values, you have to provide lifetime parameter.


## Methods

Methods are functions defined within the context of a struct, enum or trait.

First parameter is always `self`.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
	// Methods can borrow self mutably or immutably.
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

*Automatic referencing and dereferencing* is a feature where calling methods that define various types of borrows can be done by an object without actually applying these operators on the object itself. For example, if a method takes `&mut self` as an argument, the compiler will automatically apply the operators on the calling object.


## Associated Functions

Defined in `impl` blocks without `self` param.

Usually for constructors.

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
}
```

Structs can have multiple `impl` blocks.

## Enums

Enumerations are like algebraic data types in Haskell.

```rust

fn main() {
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
}
```

```rust

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

// Similar implementation using structs.
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
```

Using structs would not allow methods that take all the struct types. Probably since no inheritance.

```rust
fn main() {
    enum Message {
        Quit,
        Move { x: i32, y: i32 },
        Write(String),
        ChangeColor(i32, i32, i32),
    }

    impl Message {
        fn call(&self) {
            // method body would be defined here
        }
    }

    let m = Message::Write(String::from("hello"));
    m.call();
}
```

No *null* values in Rust. Use the `Option<T>` enum.

```rust
fn main() {
    let some_number = Some(5);
    let some_string = Some("a string");

	// Must tell compiler the T is i32.
    let absent_number: Option<i32> = None;
}
```

## Match Expressions

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}

fn main() {
    value_in_cents(Coin::Quarter(UsState::Alaska));
}
```

Matches must be exhausive. There must be an arm for every possible enum value.


```rust
fn main() {
    let some_u8_value = 0u8;
    match some_u8_value {
        1 => println!("one"),
        3 => println!("three"),
        5 => println!("five"),
        7 => println!("seven"),
        _ => (), // For the default case, do nothing and return the "unit value".
    }
}
```

## `if let`

Syntactic sugar for a match on only one pattern and ignoring all others.

```rust
fn main() {
    let some_u8_value = Some(0u8);
    match some_u8_value {
        Some(3) => println!("three"),
        _ => (),
    }

	// Does the same thing but shorter.
    if let Some(3) = some_u8_value {
        println!("three");
    }
}
```
 
`if let` also has an `else` which is the same as using the `_` arm in a match block, but this appears to be more verbose, so the match block seems to be more concise?



