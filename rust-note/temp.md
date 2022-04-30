# Rust Note

## Type

The signed integers are: `i8`, `i16`, `i32`, `i64`, `i128`, and `isize`
The unsigned integers are: `u8`, `u16`, `u32`, `u64`, `u128`, and `usize`

- what is `isize` and `usize`?
  > This means the number of bits on your type of computer. (The number of bits on your computer is called the **architecture** of your computer.)

```rust
let num = 10; // num: i32 (default integer type
let num: u8 = 10;
let num = 10u8;
let num = 10_u8;

let num = 8.; // num: f64 (default float type
```

## Variables and Code block

Variables start and end inside a code block `{}`. In this example, `my_number` ends before we call `println!`, because it is inside its own code block.

```rust
{
    let num = 10;
} // num end here

println!("{}", num); // Error, num no available
```

## Stack, Head, Pointer

```rust
let num = 8; // num on the stack
let ref_num = &num; // ref_num is a pointer, make a reference by num

println!("{}", num);
println!("{}", ref_num); // error, because
```

## Const, Static

const and static need to out of main, and need to write them with ALL CAPITAL LETTERS

```rust
const NUMBER_OF_MONTHS: u32 = 12;
static SEASONS: [&str; 4] = ["Spring", "Summer", "Fall", "Winter"];
fn main() {}
```

## `&` and `*`

using `&` is called "referencing", using `*` is called "**de**referencing".

## mutable Reference and immutable

References are very useful for functions. The rule in Rust on values is: a value can only have one owner.

- **Rule 1**: If you have only immutable references, you can have as many as you want. 1 is fine, 3 is fine, 1000 is fine. No problem.
- **Rule 2**: If you have a mutable reference, you can only have one. Also, you can't have an immutable reference **and** a mutable reference together.

## Copy Type

can copy

```rust
fn prints_number(number: i32) { // There is no -> so it's not returning anything
                             // If number was not copy type, it would take it
                             // and we couldn't use it again
    println!("{}", number);
}

fn main() {
    let my_number = 8;
    prints_number(my_number); // Prints 8. prints_number gets a copy of my_number
    prints_number(my_number); // Prints 8 again.
                              // No problem, because my_number is copy type!
}
```

can’t copy

```rust
fn prints_country(country_name: String) {
    println!("{}", country_name);
}

fn main() {
    let country = String::from("Kiribati");
    prints_country(country);
    prints_country(country); // ⚠️
}
```

\*reference is faster than copy

```rust
fn get_copy_length(input: String) { // Takes ownership of a String
    println!("It's {} words long.", input.split_whitespace().count()); // splits to count the number of words
}

fn get_length(input: &String) {
    println!("It's {} words long.", input.split_whitespace().count());
}

fn main() {
    let mut my_string = String::new();
    for _ in 0..50 {
        my_string.push_str("Here are some more words "); // push the words on
        // get_copy_length(my_string.clone()); // gives it a clone every time
				get_length(&my_string); // faster than above
    }
}
```

## Collection

You can get a slice (a piece) of an array. First you need a &, because the compiler doesn't know the size. Then you can use `..` to show the range.

```rust
let array_of_ten = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let three_to_five = &array_of_ten[2..5];
```

## Array

## Vectors

Vectors slower than Array

```rust
let mut create_vec = Vec::new();
let mut another_way = vec![8, 10, 10];
let slice_vec = &another_way[1..2];
```

Using **capacity** make vectors faster without reallocation

```rust
let mut no_capacity = Vec::new();
// let mut with_capacity = Vec::with_capacity(8);
println!("{}", no_capacity.capacity()); // 0 elements: prints 0
no_capacity.push('a'); // add one character
println!("{}", no_capacity.capacity()); // 1 element: prints 4. Vecs with 1 item always start with capacity 4
no_capacity.push('a'); // add one more
no_capacity.push('a'); // add one more
no_capacity.push('a'); // add one more
println!("{}", no_capacity.capacity()); // 4 elements: still prints 4.
no_capacity.push('a'); // add one more
println!("{}", no_capacity.capacity()); // prints 8. We have 5 elements, but it doubled 4 to 8 to make space
```

## [Match](https://doc.rust-lang.org/rust-by-example/flow_control/match.html)

```rust
let num = 2
let num = match num {
	1 => 2,
	2 => 4,
	_ => 8,  // something else
};
```

[Guards](https://doc.rust-lang.org/rust-by-example/flow_control/match/guard.html)

```rust
fn main() {
    let number: u8 = 4;

    match number {
        i if i == 0 => println!("Zero"),
        i if i > 0 => println!("Greater than zero"),
        _ => println!("Fell through"), // This should not be possible to reach
    }
}
```

[Binding](https://doc.rust-lang.org/rust-by-example/flow_control/match/binding.html)

```rust
let num = [1,2,3,4,5,6,7,8,9];
match &num[0] {
		name @ 1 => println!("{}", name),
		name @ _ => println!("{}", name),
}
```

## Loop

escape from loop with name

```rust
'first_loop: loop {
		'second_loop:loop {
				break 'first_loop;
		}
}
```

## implement

Struct implement

```rust
struct Animal {
		age: u8,
		animal_type: AnimalType,
}

enum AnimalType {
    Cat,
    Dog,
}

impl Animal {
		fn new() -> Self {
				Self {
						age: 12,
						animal_type: AnimalType::Cat,
				}
		}
		fn check_type(&self) {
        match self.animal_type {
            AnimalType::Dog => println!("The animal is a dog"),
            AnimalType::Cat => println!("The animal is a cat"),
        }
    }
}

fn main() {
		let animal = Animal::new();
		animal.check();  // The animal is a cat
}
```

## Destructuring

```rust
struct Person {
		age: u8,
		name: String,
}

let jack = Person {
		age: 12,
		name: "Jack".to_string(),
};

let Person {
		age,
		name: b,
} = jack;
println!("name: {}, age: {}", age, b);
```

## Reference and dot operator

when you use the `.` operator, you don't need to worry about `*`.

## Generic

```rust
// fn fff<T: Display, U: Display + PartialOrd>(statement: T, num_1: U, num_2: U)
fn fff<T, U>(statement: T, num_1: U, num_2: U)  // easy to read?
where
    T: Display,
    U: Display + PartialOrd,
{
    println!("{}! Is {} greater than {}? {}", statement, num_1, num_2, num_1 > num_2);
}
```

## HashMap

using `.get()` is safer than `[]`

```rust
use std::collections::HashMap;

// ...
let mut fruits = HashMap::new();
fruilts.insert("apple", 10);
fruilts.insert("banana", 20);
fruilts.insert("mongo", 30);

println!("{}", fruilts["apple"])        // 10
println!("{}", fruilts.get("apple");    // Some(10)
println!("{}", fruilts.get("orange");   // None
```

## [BinaryHeap](https://dhghomon.github.io/easy_rust/Chapter_32.html#binaryheap)

it keeps largest item in the front.

## [VecDeque](https://dhghomon.github.io/easy_rust/Chapter_32.html#vecdeque)

VecDeque is good at popping item without moveing position.

## Iterator

we can make our struct into an iterator with impl Iterator for our struct.

```rust
#[derive(Clone)]
struct Library {
    books: Vec<String>,
}

impl Library {
    fn new() -> Self {
        Self {
            books: Vec::new(),
        }
    }

    fn add_book(&mut self, book: &str) {
        self.books.push(book.to_string());
    }
}

impl Iterator for Library {
    type Item = String;

    fn next(&mut self) -> Option<String> {
        match self.books.pop() {
            Some(book) => Some(book + "is found."),
            None => None,
        }
    }
}

fn main() {
    let mut my_library = Library::new();
    my_library.add_book("first");
    my_library.add_book("second");
    my_library.add_book("third");

    for item in my_library.clone() {
        println!("{}", item);
    }
}
```

## Closures | Lambdas

```rust
let print = || println!("print it");
print();
```

## [lifetime](https://dhghomon.github.io/easy_rust/Chapter_40.html)

```rust
struct Adventurer<'a> {
    name: &'a str,
    hit_points: u32,
}

impl Adventurer<'_> {
    fn take_damage(&mut self) {
        self.hit_points -= 20;
        println!("{} has {} hit points left!", self.name, self.hit_points);
    }
}

impl std::fmt::Display for Adventurer<'_> {

        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
            write!(f, "{} has {} hit points.", self.name, self.hit_points)
        }
}

fn main() {
    let mut billy = Adventurer {
        name: "Billy",
        hit_points: 100_000,
    };
    println!("{}", billy);
    billy.take_damage();
}
```

## Interior mutability

`Cell` COPY value
`RefCell` reference
`Mutex`
`RwLock` it can write like `Mutex` and read like `RefCell`

### [RefCell](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#enforcing-borrowing-rules-at-runtime-with-refcellt)

With references and `Box<T>`, the borrowing rules’ invariants are enforced at **_compile time_**. With `RefCell<T>`, these invariants are enforced at **_runtime_**.
Similar to `Rc<T>`, `RefCell<T>` is only for use in single-threaded scenarios and will give you a compile-time error if you try using it in a multithreaded context.

### [Arc](https://doc.rust-lang.org/rust-by-example/std/arc.html)

`Arc` can be shared ownership between threads
