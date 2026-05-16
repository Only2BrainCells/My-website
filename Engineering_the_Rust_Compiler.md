

---

# Page 1

Engineering the Rust Compiler
A Comprehensive Guide from Foundations to
Production-Grade Implementation
Table of Contents
Preface
How to Use This Guide
Rust Standard Library Reference [Basic]
Part 0: Rust Fundamentals for Compiler Development [Basic]
Chapter 1: Why Rust for Compilers?
Chapter 2: Core Rust Concepts
Chapter 2.4: Ownership in Depth
Chapter 2.5: Lifetimes in Compilers
Chapter 2.6: Traits and the Visitor Pattern
Part 1: Computer Science Foundations [Basic]
Chapter 3: Data Structures
Chapter 3.5: Graphs and Compilers
Chapter 3.6: Big-O Notation and Performance
Chapter 3.7: Formal Grammars and the Chomsky Hierarchy [Advanced]
Part 2: Compiler Pipeline Basics [Intermediate]
Chapter 4: Lexical Analysis
Chapter 4.3: Error Recovery in Lexers
Chapter 4.4: Handling Whitespace and Comments


---

# Page 2

Chapter 4.5: DFA and NFA Construction [Advanced]
Chapter 5: Parsing
Chapter 5.3: Operator Precedence and Associativity
Chapter 5.4: Pratt Parsing
Chapter 5.5: Error Recovery in Parsers
Chapter 5.6: The Abstract Syntax Tree (AST) Concept
Chapter 5.7: LR and LALR Parsing [Advanced]
Chapter 6: Semantic Analysis
Chapter 6.3: Nested Scopes and Scope Stacks
Chapter 6.4: Type Inference
Part 3: Intermediate Representation [Advanced]
Chapter 7: Intermediate Representation (IR) Generation
Chapter 7.3: HIR, MIR, and LIR
Chapter 7.4: Static Single Assignment (SSA) Form
Chapter 7.5: Control Flow Graphs (CFGs)
Chapter 7.6: Dataflow Analysis [Advanced]
Chapter 7.7: SSA Construction (Dominance Frontiers) [Advanced]
Part 4: Professional Compiler Architecture [Advanced]
Chapter 8: Project Structure
Chapter 8.2: The Compilation Pipeline Data Flow
Chapter 8.3: Implementing the Visitor Pattern
Chapter 8.4: The Session Object
Chapter 8.5: Runtime Systems and Memory Management [Advanced]
Chapter 8.6: Just-In-Time (JIT) Compilation [Advanced]
Part 5: Performance & Optimization [Advanced]
Chapter 9: Vectorization and SIMD
Chapter 10: Using LLVM with Rust
Chapter 10.2: Common Optimization Passes
Chapter 10.2.1: Constant Folding [Advanced]


---

# Page 3

Chapter 10.2.2: Dead Code Elimination (DCE) [Advanced]
Chapter 10.2.3: Function Inlining [Advanced]
Chapter 10.2.4: Loop Unrolling [Advanced]
Chapter 10.2.5: Instruction Selection (Tree Tiling) [Advanced]
Chapter 10.2.6: Register Allocation (Graph Coloring) [Advanced]
Chapter 10.2.7: Calling Conventions and ABI [Advanced]
Part 6: Testing & Automation [Intermediate]
Chapter 11: Performance Monitoring
Chapter 12: Test Generation
Chapter 12.3: Snapshot Testing
Chapter 12.4: Fuzzing Compilers
Chapter 13: Build System Automation
Part 7: Python Automation [Intermediate]
Debugging & Troubleshooting
Appendices
Appendix A: Glossary of Compiler Terms
Appendix B: Rust Compiler Error Codes Reference
Appendix C: Advanced Rust Types in Compilers
Appendix D: Further Reading and Resources
References
Preface
Compiler engineering is one of the most intellectually rewarding and practically valuable
skills in computer science. Yet most resources assume significant background knowledge,
leaving beginners overwhelmed and professionals without practical guidance.
This guide bridges that gap. It takes you from zero computer science knowledge through
advanced professional compiler engineering, combining rigorous theory with practical


---

# Page 4

implementation in Rust and Python automation.
How to Use This Guide
For beginners: Start from the Rust Standard Library Reference, then Part 0, and read
sequentially.
For experienced programmers: Reference the Rust Standard Library Reference as
needed, then start from Part 1.
For professionals: Use as a reference. The cheatsheet and appendices provide quick
lookups.
Rust Standard Library Reference [Basic]
This section explains every Rust function, method, and type used in this guide. Refer back
here whenever you see unfamiliar syntax.
Understanding Rust Types
Option: A Value That Might Not Exist
What it is: Option<T> represents a value that might exist or might not. It’s either
Some(value) or None.
Why it matters: Instead of using null (which causes bugs), Rust forces you to explicitly
handle the “no value” case.
Visual representation:


---

# Page 5

Option<i32>
├─ Some(42)     ← The value exists
└─ None         ← The value doesn't exist
When to use: When a function might not return a value, or when looking up something
that might not exist.
Example:
let maybe_number: Option<i32> = Some(42);
let no_number: Option<i32> = None;
// You MUST handle both cases
match maybe_number {
    Some(n) => println!("Got: {}", n),
    None => println!("No value"),
}
Key methods:
Method What it does Example
is_some() Returns true if Some opt.is_some() → true/false
is_none() Returns true if None opt.is_none() → true/false
unwrap() Gets the value (panics if None) opt.unwrap() → 42 or crash
unwrap_or(default) Gets value or default opt.unwrap_or(0) → 42 or 0
map(f) Transform the value if Some opt.map(|x| x * 2)
and_then(f) Chain operations that return
Option
opt.and_then(|x| Some(x *
2))
filter(f) Keep only if condition is true opt.filter(|x| x > 10)
Real example from compilers:


---

# Page 6

// Looking up a variable in symbol table
let symbol_table: HashMap<String, Type> = ...;
// get() returns Option<&Type>
match symbol_table.get("x") {
    Some(ty) => println!("Variable x has type: {:?}", ty),
    None => println!("ERROR: Variable x not defined"),
}
// Or more concisely:
if let Some(ty) = symbol_table.get("x") {
    println!("Type: {:?}", ty);
}
Result: Success or Error
What it is: Result<T, E> represents either success (Ok(value)) or failure
(Err(error)).
Why it matters: Forces you to handle errors explicitly instead of ignoring them.
Visual representation:
Result<i32, String>
├─ Ok(42)              ← Success, contains value
└─ Err("not a number") ← Failure, contains error message
When to use: When an operation might fail (parsing, file I/O, etc.).
Example:


---

# Page 7

fn parse_number(s: &str) -> Result<i32, String> {
    // s.parse() returns Result<i32, ParseIntError>
    // We convert the error to a String
    s.parse::<i32>()
        .map_err(|_| format!("'{}' is not a valid number", s))
}
// Using it:
match parse_number("42") {
    Ok(n) => println!("Parsed: {}", n),
    Err(e) => println!("Error: {}", e),
}
Key methods:
Method What it does Example
is_ok() Returns true if Ok result.is_ok() → true/false
is_err() Returns true if Err result.is_err() → true/false
unwrap() Gets the value (panics if Err) result.unwrap() → value or crash
unwrap_or(default) Gets value or default result.unwrap_or(0)
map(f) Transform the value if Ok result.map(|x| x * 2)
map_err(f) Transform the error if Err result.map_err(|e|
e.to_string())
and_then(f) Chain operations that return
Result result.and_then(|x| Ok(x * 2))
? operator Propagate error up let x = parse_number(s)?;
The ? operator (error propagation):


---

# Page 8

// Without ?
fn parse_expression(s: &str) -> Result<i32, String> {
    let n = match parse_number(s) {
        Ok(val) => val,
        Err(e) => return Err(e),  // Return error immediately
    };
    Ok(n * 2)
}
// With ? (much cleaner)
fn parse_expression(s: &str) -> Result<i32, String> {
    let n = parse_number(s)?;  // If Err, return it; if Ok, get value
    Ok(n * 2)
}
Real example from compilers:
// Type checking function
fn check_type(expr: &Expr) -> Result<Type, String> {
    match expr {
        Expr::Number(_) => Ok(Type::Integer),
        Expr::Identifier(id) => {
            // lookup returns Option<Type>
            // We convert it to Result
            self.lookup(id)
                .ok_or_else(|| format!("Undefined variable: {}", id))
        }
        Expr::BinaryOp { left, op, right } => {
            // ? operator propagates errors
            let left_type = self.check_type(left)?;
            let right_type = self.check_type(right)?;
            
            if left_type != right_type {
                return Err(format!("Type mismatch"));
            }
            
            Ok(Type::Integer)
        }
    }
}


---

# Page 9

Understanding Collections
Vec: Dynamic Arrays
What it is: A growable array. Starts small and grows as you add elements.
Why it matters: You don’t need to know the size upfront. Perfect for storing tokens, AST
nodes, etc.
Creating vectors:
let mut v = Vec::new();           // Empty vector
let mut v = vec![1, 2, 3];        // With initial values
let mut v: Vec<i32> = Vec::new(); // With explicit type
Key methods:
Method What it does Example
push(value) Add to end v.push(42);
pop() Remove from end v.pop() → Some(42) or None
len() How many elements v.len() → 3
is_empty() Is it empty? v.is_empty() → true/false
get(index) Safe access v.get(0) → Some(&value) or None
[index] Direct access v[0] → value or panic
iter() Iterate (read-only) for x in v.iter() { }
iter_mut() Iterate (mutable) for x in v.iter_mut() { }
insert(index, value) Insert at position v.insert(1, 99);
remove(index) Remove at position v.remove(1);
Real example from compilers:


---

# Page 10

// Storing tokens
let mut tokens: Vec<Token> = Vec::new();
// Lexer adds tokens
tokens.push(Token::Let);
tokens.push(Token::Identifier("x".to_string()));
tokens.push(Token::Assign);
tokens.push(Token::Number(42));
// Parser reads tokens
for (i, token) in tokens.iter().enumerate() {
    println!("Token {}: {:?}", i, token);
}
// Safe access
if let Some(token) = tokens.get(0) {
    println!("First token: {:?}", token);
}
HashMap: Key-Value Storage
What it is: A map from keys to values. Fast lookups.
Why it matters: Symbol tables are HashMaps. You look up variable names and get their
types.
Creating hashmaps:
use std::collections::HashMap;
let mut map = HashMap::new();
map.insert("x", 42);
map.insert("y", 99);
Key methods:


---

# Page 11

Method What it does Example
insert(key, value) Add or update map.insert("x", 42);
get(key) Look up map.get("x") → Some(&42) or None
remove(key) Delete map.remove("x");
contains_key(key) Does key exist? map.contains_key("x") → true/false
len() How many entries map.len() → 2
is_empty() Is it empty? map.is_empty() → true/false
keys() All keys for k in map.keys() { }
values() All values for v in map.values() { }
iter() All key-value pairs for (k, v) in map.iter() { }
Real example from compilers:
// Symbol table: variable name → type
let mut symbol_table: HashMap<String, Type> = HashMap::new();
// Declare variables
symbol_table.insert("x".to_string(), Type::Integer);
symbol_table.insert("name".to_string(), Type::String);
// Look up variable type
match symbol_table.get("x") {
    Some(ty) => println!("Type: {:?}", ty),
    None => println!("ERROR: Variable not defined"),
}
// Check if variable exists
if symbol_table.contains_key("x") {
    println!("Variable x exists");
}


---

# Page 12

Understanding Iterators
Iterator Methods: map(), filter(), fold()
What iterators are: A way to process collections without writing explicit loops.
Why they matter: Cleaner code, often better optimized by the compiler.
map(): Transform Each Element
What it does: Apply a function to each element, creating a new collection.
Syntax: collection.iter().map(|element| transformation).collect()
Example:
let numbers = vec![1, 2, 3, 4];
// Double each number
let doubled: Vec<i32> = numbers
    .iter()           // Create iterator
    .map(|x| x * 2)   // Transform each element
    .collect();       // Collect into Vec
// doubled = [2, 4, 6, 8]
Real example from compilers:
// Convert tokens to their string representations
let tokens = vec![Token::Let, Token::Number(42)];
let token_strings: Vec<String> = tokens
    .iter()
    .map(|t| format!("{:?}", t))
    .collect();
// token_strings = ["Let", "Number(42)"]


---

# Page 13

filter(): Keep Matching Elements
What it does: Keep only elements that match a condition.
Syntax: collection.iter().filter(|element| condition).collect()
Example:
let numbers = vec![1, 2, 3, 4, 5, 6];
// Keep only even numbers
let evens: Vec<i32> = numbers
    .iter()
    .filter(|x| x % 2 == 0)  // Keep if condition is true
    .collect();
// evens = [2, 4, 6]
Real example from compilers:
// Keep only error tokens
let tokens = vec![Token::Number(1), Token::Error("bad"), Token::Number(2)];
let errors: Vec<_> = tokens
    .iter()
    .filter(|t| matches!(t, Token::Error(_)))
    .collect();
fold(): Combine Into One Value
What it does: Combine all elements into a single value.
Syntax: collection.iter().fold(initial_value, |accumulator, element|
new_accumulator)
Example:


---

# Page 14

let numbers = vec![1, 2, 3, 4];
// Sum all numbers
let sum = numbers
    .iter()
    .fold(0, |acc, x| acc + x);
// sum = 10
Step by step:
Start: acc = 0
Step 1: acc = 0 + 1 = 1
Step 2: acc = 1 + 2 = 3
Step 3: acc = 3 + 3 = 6
Step 4: acc = 6 + 4 = 10
Result: 10
Real example from compilers:
// Count total tokens
let tokens = vec![Token::Let, Token::Number(42), Token::Semicolon];
let count = tokens
    .iter()
    .fold(0, |acc, _| acc + 1);
// count = 3
Chaining Iterators
What it is: Combining multiple operations.
Example:


---

# Page 15

let numbers = vec![1, 2, 3, 4, 5, 6];
// Get even numbers, double them, sum them
let result = numbers
    .iter()
    .filter(|x| x % 2 == 0)      // [2, 4, 6]
    .map(|x| x * 2)              // [4, 8, 12]
    .fold(0, |acc, x| acc + x);  // 24
// result = 24
Real example from compilers:
// Get all identifiers, convert to uppercase, collect
let tokens = vec![
    Token::Identifier("x".to_string()),
    Token::Number(42),
    Token::Identifier("y".to_string()),
];
let identifiers: Vec<String> = tokens
    .iter()
    .filter_map(|t| match t {
        Token::Identifier(id) => Some(id.to_uppercase()),
        _ => None,
    })
    .collect();
// identifiers = ["X", "Y"]
Understanding String Methods
String Operations
String vs &str:


---

# Page 16

let owned = String::from("hello");     // Owned String
let borrowed: &str = &owned;           // Borrowed reference
let literal = "hello";                 // &str literal
Key methods:
Method What it does Example
push_str(s) Add string to end s.push_str(" world");
push(c) Add character to end s.push('!');
len() Length in bytes s.len() → 5
chars() Iterate characters for c in s.chars() { }
as_bytes() Get bytes s.as_bytes()[0]
to_string() Convert to String "hello".to_string()
to_uppercase() Convert to uppercase s.to_uppercase()
to_lowercase() Convert to lowercase s.to_lowercase()
trim() Remove whitespace s.trim()
split(sep) Split by separator s.split(' ')
contains(s) Does it contain? s.contains("ell") → true
starts_with(s) Starts with? s.starts_with("he") → true
ends_with(s) Ends with? s.ends_with("lo") → true
Real example from compilers:


---

# Page 17

// Reading identifier characters
let input = "hello_world123";
let mut id = String::new();
for ch in input.chars() {
    if ch.is_alphanumeric() || ch == '_' {
        id.push(ch);
    } else {
        break;
    }
}
// id = "hello_world123"
Understanding match and if let
match: Handle All Cases
What it is: Pattern matching. Handle different cases explicitly.
Why it matters: Compiler forces you to handle all cases.
Syntax:
match value {
    pattern1 => action1,
    pattern2 => action2,
    _ => default_action,  // _ matches anything
}
Example:


---

# Page 18

enum Token {
    Number(i32),
    Identifier(String),
    Operator(char),
}
let token = Token::Number(42);
match token {
    Token::Number(n) => println!("Number: {}", n),
    Token::Identifier(id) => println!("Identifier: {}", id),
    Token::Operator(op) => println!("Operator: {}", op),
}
if let: Handle One Case
What it is: Simpler syntax when you only care about one case.
Why it matters: Less verbose than match when you don’t need all cases.
Syntax:
if let pattern = value {
    // Handle this case
} else {
    // Handle other cases
}
Example:


---

# Page 19

let maybe_number: Option<i32> = Some(42);
// Instead of:
match maybe_number {
    Some(n) => println!("Got: {}", n),
    None => {},
}
// Use if let:
if let Some(n) = maybe_number {
    println!("Got: {}", n);
}
Understanding Closures
What they are: Anonymous functions you can pass around.
Syntax: |parameters| expression
Example:
// Simple closure
let add_one = |x| x + 1;
println!("{}", add_one(5));  // 6
// Closure with multiple parameters
let add = |x, y| x + y;
println!("{}", add(2, 3));  // 5
// Closure with block
let greet = |name| {
    println!("Hello, {}!", name);
    name.len()
};
Real example from compilers:


---

# Page 20

// Using closure with map
let numbers = vec![1, 2, 3];
let doubled = numbers.iter().map(|x| x * 2).collect();
// Using closure with filter
let evens = numbers.iter().filter(|x| x % 2 == 0).collect();
Part 0: Rust Fundamentals for Compiler
Development [Basic]
Chapter 1: Why Rust for Compilers?
Rust combines three critical properties essential for compiler development:
1. Performance: As fast as C. No garbage collection overhead.
2. Memory Safety: No segmentation faults or use-after-free bugs. Caught at compile
time.
3. Concurrency Safety: No data races. Prevents entire classes of bugs.
Chapter 2: Core Rust Concepts
2.1 Variables and Mutability
What it is: A variable is named storage. By default, variables are immutable.
What to know:


---

# Page 21

let x = 5;              // Immutable - can't change
// x = 10;             // ERROR!
let mut y = 5;          // Mutable - can change
y = 10;                 // OK
When to use: Always start immutable. Add mut only when needed.
2.2 Ownership
What it is: Every value has exactly one owner. When the owner goes out of scope, the
value is dropped.
What to know:
let s = String::from("hello");      // s owns the String
let s2 = s;                         // Ownership moves to s2
// println!("{}", s);               // ERROR! s no longer owns it
let x = 5;                          // x owns the integer
let y = x;                          // x is still valid! (copied)
println!("{}", x);                  // OK
2.3 Borrowing
What it is: Temporary access without taking ownership. Use & for immutable, &mut for
mutable.
What to know:


---

# Page 22

let s = String::from("hello");
let s1 = &s;                        // Immutable borrow
let s2 = &s;                        // Multiple immutable borrows OK
println!("{}", s1);                 // OK
let mut s = String::from("hello");
let s_mut = &mut s;                 // Mutable borrow
s_mut.push_str("!");
// println!("{}", s);               // ERROR!
Golden rule: Either multiple readers OR one writer, never both.
2.4 Ownership in Depth [Basic]
Conceptual Explanation: Ownership is Rust’s unique way of managing memory without a
garbage collector. It works through a set of rules that the compiler checks at compile time.
The core idea is that every value in Rust has a single variable that is its “owner.” When the
owner’s scope ends, Rust automatically cleans up the memory. This prevents memory
leaks and “use-after-free” bugs, where a program tries to use memory that has already
been released.
Mental Model: Imagine a library book. There is only one person who has the book
checked out (the owner). If they give the book to someone else (a move), they no longer
have it. If they let someone look at the book while they still have it (a borrow), they are still
the owner. When the owner returns the book to the library (goes out of scope), the book is
gone from their possession. This ensures the book is always accounted for and never lost
or duplicated.
Compilers and Ownership: Compilers deal with massive, interconnected data structures
like ASTs and Symbol Tables. Ownership ensures that these structures are cleanly
managed. For example, when a function that generates an AST finishes, the ownership of
that AST is passed back to the caller, ensuring the memory stays valid as long as needed.
Common Borrow Checker Error:


---

# Page 23

fn main() {
    let mut list = vec![1, 2, 3];
    let first = &list[0]; // Think: "Immutable borrow of list"
    
    list.push(4); // Think: "Mutable borrow of list occurs here"
    
    // println!("{}", first); // ERROR: cannot borrow `list` as mutable 
because it is also borrowed as immutable
}
The Fix: Ensure the immutable borrow (first) is no longer needed before performing
the mutable operation (push), or clone the value if you need it independently.
2.5 Lifetimes in Compilers [Basic]
Conceptual Explanation: Lifetimes are a way for the Rust compiler to ensure that
references are always valid. A lifetime is the period of time during which a reference points
to a valid piece of memory. Most of the time, Rust infers lifetimes automatically, but
sometimes you need to explicitly label them, especially when storing references inside
structs.
Mental Model: Think of a lifetime as a “validity tag” on a reference. If you have a reference
to a name in a book, that reference is only valid as long as the book exists. If the book is
destroyed, the reference becomes a “dangling pointer.” Lifetimes prevent this by ensuring
the “book” (the data) lives at least as long as any “references” to it.
Example: Storing References in a Token Struct


---

# Page 24

// Think: "'a is a lifetime parameter"
// Think: "It says: 'This Token cannot outlive the source string it 
references'"
struct Token<'a> {
    kind: TokenKind,
    lexeme: &'a str, // Think: "A reference to a part of the source code"
}
fn main() {
    let source = String::from("let x = 5;");
    let token = Token {
        kind: TokenKind::Let,
        lexeme: &source[0..3], // Think: "Borrowing from source"
    };
    // If 'source' was dropped here, 'token' would be invalid.
    // Rust prevents this using lifetimes.
}
2.6 Traits and the Visitor Pattern [Basic]
Conceptual Explanation: Traits in Rust define shared behavior that different types can
implement. They are similar to interfaces in other languages. In compiler development, the
Visitor Pattern is a common design pattern used to traverse and perform operations on
complex structures like an Abstract Syntax Tree (AST). Instead of putting all the logic (like
type checking, optimization, code generation) inside the AST nodes themselves, you
define a Visitor trait.
Mental Model: Imagine a building (the AST) with many different rooms (the nodes). A
“Visitor” is a specialist (like an electrician, a plumber, or a painter) who walks through every
room. Each specialist does something different in each room, but they all follow the same
path through the building. The building doesn’t need to know how to fix pipes or wires; it
just needs to allow the specialist to enter.
Practical Example: AST Visitor


---

# Page 25

// Think: "Define the shared behavior for all visitors"
trait Visitor {
    fn visit_number(&mut self, n: i32);
    fn visit_binary_op(&mut self, left: &Expr, op: BinOp, right: &Expr);
}
// Think: "A specific visitor that prints the AST"
struct AstPrinter;
impl Visitor for AstPrinter {
    fn visit_number(&mut self, n: i32) {
        println!("Number: {}", n);
    }
    
    fn visit_binary_op(&mut self, left: &Expr, op: BinOp, right: &Expr) {
        println!("Op: {:?}", op);
        // Think: "Continue visiting children"
        left.accept(self);
        right.accept(self);
    }
}
// Think: "AST nodes must 'accept' a visitor"
impl Expr {
    fn accept(&self, visitor: &mut dyn Visitor) {
        match self {
            Expr::Number(n) => visitor.visit_number(*n),
            Expr::BinaryOp { left, op, right } => 
visitor.visit_binary_op(left, *op, right),
        }
    }
}


---

# Page 26

Part 1: Computer Science Foundations
[Basic]
Chapter 3: Data Structures
3.1 Arrays and Vectors
What it is: Ordered collection of elements stored contiguously.
Performance:
Operation Time
Access by index O(1)
Append O(1) amortized
Insert at position O(n)
3.2 Stacks
Mental model: Last-In-First-Out (LIFO).
3.3 Trees
Mental model: Hierarchical structure.
3.4 Hash Tables
Mental model: Key-value storage with fast lookups.
3.5 Graphs and Compilers [Basic]
Conceptual Explanation: A graph is a collection of “nodes” connected by “edges.” Unlike
trees, graphs can have cycles (loops) and multiple paths between nodes. In compilers,


---

# Page 27

graphs are everywhere. They represent the flow of a program, the dependencies between
variables, and the relationships between different modules.
Mental Model: Think of a graph as a map of a city. The intersections are nodes, and the
streets are edges. Some streets are one-way (directed graph), and you can travel in circles
around blocks (cycles). A compiler uses this “map” to understand how data travels through
your code and which parts of the code depend on others.
Key Compiler Graphs:
1. Control Flow Graph (CFG): Represents all paths that might be traversed through a
program during its execution. Nodes are “basic blocks” (sequences of instructions
with no jumps), and edges represent jumps or falls-through.
2. Dependency Graph: Shows which variables or modules depend on others. This is
used to determine the order of compilation or which parts of the code can be
optimized away.
3.6 Big-O Notation and Performance [Basic]
Conceptual Explanation: Big-O notation is a mathematical way to describe how the
performance of an algorithm changes as the size of the input grows. It focuses on the
“worst-case scenario.” In compiler engineering, choosing the right algorithm can mean the
difference between a compiler that takes seconds to run and one that takes hours.
Mental Model: Imagine you are looking for a name in a phone book. If you look page by
page, it takes longer as the book gets bigger (O(n)). If you open the book in the middle and
keep halving the search area, it’s much faster even for huge books (O(log n)). Big-O is like
a “speed limit” or “efficiency rating” for your code.
Common Complexities in Compilers:


---

# Page 28

Notation Name Meaning in Compilers Example
O(1) Constant Time stays the same regardless
of input size.
Accessing an element in a vector
by index.
O(log n) Logarithmic Time grows slowly as input size
increases.
Searching for a symbol in a
balanced binary tree.
O(n) Linear Time grows proportionally to
input size.
A single pass over the source code
(Lexing).
O(n log
n) Linearithmic Very efficient for large inputs. Sorting tokens or identifiers.
O(n2) Quadratic Time grows quickly; becomes
slow for large inputs.
Naive nested loops for name
resolution.
O(2n) Exponential Becomes unusable very
quickly.
Certain complex optimization or
constraint solving problems.
Part 2: Compiler Pipeline Basics
[Intermediate]
Chapter 4: Lexical Analysis
4.1 What is Lexical Analysis?
Conceptual Explanation: Lexical analysis, or lexing, is the first phase of a compiler. Its
primary job is to read the raw source code character by character and group them into
meaningful units called tokens. Think of it like breaking down a sentence into individual
words and punctuation marks.
Mental Model: Imagine a diligent librarian meticulously scanning a book. Instead of
understanding the story, the librarian’s job is to identify each word, number, and symbol,
categorize it (e.g., “noun,” “verb,” “number”), and note its position.


---

# Page 29

4.2 Building a Lexer
Token definition:
#[derive(Debug, Clone, PartialEq)]
pub enum Token {
    Let, If, Else,
    Number(i32),
    Identifier(String),
    Plus, Minus,
    LeftParen, RightParen,
    Eof,
    Error(String), // Think: "Represent invalid characters as error tokens"
}
4.3 Error Recovery in Lexers [Intermediate]
Conceptual Explanation: A robust lexer shouldn’t just crash when it sees a character it
doesn’t recognize (like a random symbol @ in a language that doesn’t use it). Instead, it
should perform error recovery. This means reporting the error, skipping the bad character,
and continuing to find more tokens. This allows the compiler to show the user multiple
errors at once instead of stopping at the first one.
Mental Model: Imagine the librarian finds a smudge on the page that doesn’t look like a
word. Instead of throwing the whole book away, they write down “I found a smudge at page
10,” skip it, and keep reading the next word.
Code Example: Lexer Error Recovery


---

# Page 30

fn next_token(&mut self) -> Token {
    self.skip_whitespace();
    
    match self.current_char() {
        None => Token::Eof,
        Some(ch) => {
            match ch {
                '+' => { self.advance(); Token::Plus }
                // ... other valid cases ...
                _ => {
                    // Think: "We don't recognize this character!"
                    let error_msg = format!("Unexpected character: '{}'", 
ch);
                    self.advance(); // Think: "Skip it and move on"
                    Token::Error(error_msg) // Think: "Return an error 
token"
                }
            }
        }
    }
}
4.4 Handling Whitespace and Comments [Intermediate]
Conceptual Explanation: Most programming languages ignore “whitespace” (spaces,
tabs, newlines) and “comments” because they don’t affect how the program runs. The
lexer is responsible for filtering these out so the parser only sees the important tokens.
Mental Model: Think of whitespace and comments as the “packaging” of a product. When
you buy a toy, you throw away the box and the plastic wrap (whitespace/comments) and
only keep the toy parts (tokens) to build it.
Code Example: Handling Comments


---

# Page 31

fn skip_whitespace_and_comments(&mut self) {
    loop {
        self.skip_whitespace(); // Think: "Skip spaces"
        
        // Think: "Check for start of a comment (e.g., //)"
        if self.current_char() == Some('/') && self.peek_char() == Some('/') 
{
            // Think: "Skip until end of line"
            while let Some(ch) = self.current_char() {
                if ch == '\n' { break; }
                self.advance();
            }
        } else {
            break; // Think: "No more whitespace or comments to skip"
        }
    }
}
Chapter 5: Parsing
5.1 What is Parsing?
Conceptual Explanation: Parsing is the second phase of a compiler. It takes the stream
of tokens and organizes them into a hierarchical structure called an Abstract Syntax Tree
(AST).
Mental Model: After the lexer identified words, the parser organizes them into sentences
and paragraphs, ensuring they follow the rules of grammar.
5.3 Operator Precedence and Associativity [Intermediate]
Conceptual Explanation: In mathematics and programming, certain operations happen
before others. For example, in 2 + 3 * 4, the multiplication happens first. This is
precedence. If operations have the same precedence, like 10 - 5 - 2, associativity
determines the order (usually left-to-right, so (10 - 5) - 2).
Mental Model: Think of precedence as a “priority queue” for math. Multiplication has a
higher priority than addition. Associativity is the “tie-breaker” when two people have the


---

# Page 32

same priority—whoever got there first (on the left) goes first.
5.4 Pratt Parsing [Advanced]
Conceptual Explanation: While simple “Recursive Descent” parsers work well for
statements, they can become messy for complex math expressions with many precedence
levels. Pratt Parsing (or Top-Down Operator Precedence) is an elegant alternative. It uses
a table of “binding powers” and small functions for each token type to handle expressions
cleanly.
Mental Model: Imagine each operator has a “magnetism” (binding power). Multiplication
has a stronger magnet than addition. When the parser sees 2 + 3 * 4, the * pulls the
3 towards the 4 more strongly than the + pulls the 3 towards the 2.
Parsing
Strategy Pros Cons
Recursive
Descent Easy to understand, good for statements. Can be verbose for expressions.
Pratt Parsing Extremely clean for expressions, easy to
add operators.
Slightly more abstract to
implement initially.
5.5 Error Recovery in Parsers [Intermediate]
Conceptual Explanation: Just like lexers, parsers need to recover from errors. If a user
forgets a semicolon or a closing parenthesis, the parser shouldn’t just give up. One
common technique is Panic Mode Recovery, where the parser “panics,” skips tokens until
it finds a “synchronization token” (like a semicolon or a keyword like fn), and then
resumes parsing the next statement.
Mental Model: If you’re reading a recipe and one step is missing a word, you don’t throw
the whole recipe away. You skip to the next step (the next “synchronization point”) and try
to finish the rest of the meal.
Code Example: Parser Synchronization


---

# Page 33

fn synchronize(&mut self) {
    self.advance(); // Think: "Skip the token that caused the error"
    while !self.is_at_end() {
        if self.previous().kind == TokenKind::Semicolon { return; }
        
        match self.current_token().kind {
            TokenKind::Class | TokenKind::Fn | TokenKind::Let | 
TokenKind::If => return,
            _ => self.advance(), // Think: "Keep skipping until we find a 
safe starting point"
        }
    }
}
5.6 The Abstract Syntax Tree (AST) Concept [Basic]
Conceptual Explanation: The AST is the “heart” of the compiler. It’s a tree-like
representation of your code that removes “syntax noise” (like parentheses and semicolons)
and keeps only the essential structure.
Mental Model: If source code is a raw sentence, the AST is a “sentence diagram.” It
shows that “3 * 4” is a single unit, and that unit is being added to “2”.
Visualizing “2 + 3 * 4”:
      Add (+)
     /       \
  Num(2)    Mul (*)
           /       \
        Num(3)    Num(4)
Chapter 6: Semantic Analysis
6.1 What is Semantic Analysis?
Conceptual Explanation: Semantic analysis checks for meaning and logical consistency.
It ensures that while the code is grammatically correct, it actually makes sense (e.g., no


---

# Page 34

adding strings to numbers).
6.3 Nested Scopes and Scope Stacks [Intermediate]
Conceptual Explanation: Most languages have “scopes”—regions where variables live. A
variable defined inside a function shouldn’t be accessible outside. Scopes can be “nested”
(a function inside a function). Compilers manage this using a Scope Stack.
Mental Model: Think of scopes as a stack of transparent boxes. You can see through the
boxes below you (outer scopes), but you can’t see into boxes that are inside others or
boxes that are next to you. When you enter a new block of code, you put a new box on the
stack. When you leave, you take it off.
Visualizing a Scope Stack:
[ Top: Local Scope (x, y) ]  <-- Compiler looks here first
[ Mid: Function Scope (a, b) ]
[ Bottom: Global Scope (PI, version) ]
6.4 Type Inference [Advanced]
Conceptual Explanation: Type inference is the compiler’s ability to figure out the type of a
variable without the programmer explicitly telling it. For example, in let x = 5;, the
compiler “infers” that x is an integer because 5 is an integer.
Mental Model: Think of it like a detective solving a mystery. If x is being added to 5, and
5 is a number, then x must also be a number. The compiler follows these “clues”
throughout your code to determine the types of everything.


---

# Page 35

Part 3: Intermediate Representation
[Advanced]
Chapter 7: Intermediate Representation (IR) Generation
7.1 What is Intermediate Representation (IR)?
Conceptual Explanation: IR is a bridge between the high-level AST and the low-level
machine code. It’s a simplified, machine-independent language.
Mental Model: Imagine translating a novel. Instead of going directly from English to
Japanese, you first create a simplified outline (the IR). From that outline, you can easily
translate into Japanese, French, or Spanish.
7.3 HIR, MIR, and LIR [Advanced]
Conceptual Explanation: Modern compilers often use multiple levels of IR to perform
different types of optimizations.
IR
Level Full Name Purpose
HIR High-level
IR Close to the AST; used for type checking and early optimizations.
MIR Mid-level
IR
Represents the flow of the program; used for borrow checking and flow-
based optimizations.
LIR Low-level
IR
Close to machine code; used for register allocation and instruction
selection.
7.4 Static Single Assignment (SSA) Form [Advanced]
Conceptual Explanation: SSA is a property of an IR where every variable is assigned
exactly once. If a variable is changed in the source code, the SSA form creates a new
version of it (e.g., x1, x2).


---

# Page 36

Mental Model: Think of it as “version control” for variables. Instead of overwriting x, you
create x_v1, x_v2. This makes it incredibly easy for the compiler to track where a value
came from and whether it’s still being used.
7.5 Control Flow Graphs (CFGs) [Advanced]
Conceptual Explanation: A CFG represents all possible paths through a program. It
consists of Basic Blocks (code with no jumps) connected by edges representing jumps.
Mental Model: Think of a CFG as a “choose your own adventure” map. Each page is a
basic block, and the instructions at the bottom tell you which page to turn to next based on
a condition.
Part 4: Professional Compiler
Architecture [Advanced]
Chapter 8: Project Structure
Real-world compilers are complex and are almost always split into multiple “crates”
(packages) within a Cargo Workspace. This improves build times, encourages modularity,
and allows for better testing.
8.1 Production-Grade Project Layout
my-compiler/
Cargo.toml: Workspace manifest; lists all crates.
Cargo.lock: Locked dependency versions for reproducible builds.
README.md: Project overview and build instructions.
crates/: Contains all the individual components of the compiler.
compiler-driver/: The top-level binary. It’s the “boss” that orchestrates the
entire pipeline from reading a file to emitting code.


---

# Page 37

compiler-session/: Centralized state. Holds global configuration, the
“Source Map” (mapping code back to files), and the “Diagnostic” system for
reporting errors.
compiler-lexer/: Turns characters into tokens.
compiler-parser/: Turns tokens into an AST.
compiler-ast/: Shared definitions of the AST nodes. This is a separate crate
so other crates (like the parser and resolver) can use it without depending on
each other.
compiler-resolve/: Name resolution. Connects every variable usage to its
definition.
compiler-typeck/: The type checker. Ensures everything is logically
consistent.
compiler-ir/: Definitions for HIR, MIR, and LIR.
compiler-opt/: Optimization passes (like Dead Code Elimination).
compiler-codegen/: The final stage. Translates IR into machine code or
LLVM IR.
compiler-common/: Shared utilities like “String Interning” (storing strings
efficiently) and “Arena Allocation”.
tests/: Integration tests that run the full compiler on real source files.
benches/: Performance benchmarks.
docs/: Technical specifications and architectural overviews.
8.2 The Compilation Pipeline Data Flow [Advanced]
Conceptual Explanation: In a professional compiler, data flows through the crates in a
strictly defined sequence. Each phase takes the output of the previous phase, transforms
it, and passes it along.
Mental Model: Think of an assembly line in a factory. The lexer crate provides the raw
materials (tokens), the parser builds the frame (AST), the resolver and typeck
inspect and validate the build, and finally, the codegen crate paints it and ships it out
(machine code).


---

# Page 38

8.3 Implementing the Visitor Pattern [Advanced]
Conceptual Explanation: As discussed in Part 0, the Visitor pattern is essential for
keeping your compiler code clean. It separates the structure of the AST from the
operations you perform on it.
Rust Code Example: Full Visitor Implementation
// In compiler-ast/src/visitor.rs
pub trait AstVisitor<'ast> {
    fn visit_expr(&mut self, expr: &'ast Expr) {
        // Think: "Default behavior: just walk the children"
        walk_expr(self, expr);
    }
    fn visit_stmt(&mut self, stmt: &'ast Stmt) {
        walk_stmt(self, stmt);
    }
}
pub fn walk_expr<'ast, V: AstVisitor<'ast> + ?Sized>(visitor: &mut V, expr: 
&'ast Expr) {
    match expr {
        Expr::Binary { left, right, .. } => {
            visitor.visit_expr(left);
            visitor.visit_expr(right);
        }
        Expr::Literal(_) => {} // Think: "Leaf node, nothing to visit"
    }
}
8.4 The Session Object [Advanced]
Conceptual Explanation: The Session object is a shared context passed through every
phase of the compiler. It centralizes error reporting (diagnostics), compiler flags (like --
release), and the source map.
Mental Model: Think of the Session as a “Project Folder” that every worker on the
assembly line carries around. It contains the blueprints, the error log, and the instructions
for the final product.


---

# Page 39

Part 5: Performance & Optimization
[Advanced]
Chapter 9: Vectorization and SIMD
9.1 When to Use SIMD
Decision Tree for SIMD:
1. Is it a loop over a large array? -> Yes
2. Are iterations independent? -> Yes
3. Is the logic simple (no complex branches)? -> Yes
4. RESULT: USE SIMD
Chapter 10: Using LLVM with Rust
10.2 Common Optimization Passes [Advanced]
Conceptual Explanation: Optimization passes transform the code to make it faster or
smaller without changing what it does.
1. Constant Folding: Evaluating math at compile time.
Before: x = 2 + 2
After: x = 4
2. Dead Code Elimination (DCE): Removing code that can never run.
Before: if false { do_thing(); }
After: (Empty)
3. Inlining: Replacing a function call with the function’s body.
Before: y = square(x)
After: y = x * x


---

# Page 40

4. Loop Unrolling: Reducing loop overhead by repeating the body.
Before: for i in 0..3 { a[i] = 0; }
After: a[0] = 0; a[1] = 0; a[2] = 0;
Part 6: Testing & Automation
[Intermediate]
Chapter 12: Test Generation
12.3 Snapshot Testing [Intermediate]
Conceptual Explanation: Snapshot testing involves saving the “correct” output of a
compiler phase (like the AST or IR) to a file. Every time you run tests, the compiler
compares the new output to the saved “snapshot.” If they differ, the test fails.
Mental Model: It’s like taking a “before” and “after” photo. If you change the engine of a
car, you check the photo to make sure the car still looks the same on the outside.
12.4 Fuzzing Compilers [Advanced]
Conceptual Explanation: Fuzzing is a technique where you feed the compiler millions of
random, slightly malformed inputs to see if it crashes. This is vital for finding rare bugs and
security vulnerabilities.
Mental Model: Imagine a “stress test” where you throw random objects at a machine to
see if it breaks. You might throw a rock, a feather, or a bucket of water. If the machine
survives everything, it’s “fuzz-tested.”


---

# Page 41

Debugging & Troubleshooting Guide
Debugging the Parser [Intermediate]
1. Trace Printing: Add println!("Parsing expression at token: {:?}",
self.current()) to see where the parser is.
2. The dbg! Macro: Use dbg!(node) to print the value and location of a variable in
one go.
3. Pretty-Printing: Implement a function that prints the AST with indentation to
visualize the structure.
4. Common Bug: Infinite Recursion.
Symptom: Stack Overflow.
Cause: A rule like Expr -> Expr + Term where the parser calls itself
immediately.
Fix: Use a loop or change the grammar to Expr -> Term (+ Term)*.
Appendices
Appendix A: Glossary
1. Abstract Syntax Tree (AST): A tree representation of source code structure.
2. Basic Block: A sequence of instructions with one entry and one exit.
3. Control Flow Graph (CFG): A graph of all paths through a program.
4. Dead Code Elimination: Removing code that has no effect.
5. Dominance: A node A dominates B if every path to B goes through A.
6. Function Inlining: Replacing a call with the function body.
7. Grammar: The formal rules of a language.
8. Lexeme: A sequence of characters matching a token.


---

# Page 42

9. Lifetime: The scope for which a reference is valid.
10. Monomorphization: Generating specific code for generic types.
11. Operator Precedence: The priority of math operations.
12. Parse Tree: A detailed tree showing every grammar rule used.
13. Pratt Parser: A top-down operator precedence parser.
14. Recursive Descent: A parser built from mutually recursive functions.
15. Register Allocation: Assigning variables to CPU registers.
16. SSA Form: Every variable is assigned exactly once.
17. Scope: The region where a name is valid.
18. Symbol Table: A map from names to their properties.
19. Token: A categorized chunk of text (e.g., “Keyword”).
20. Type Inference: Automatically determining types.
Appendix B: Rust Quick Reference
Common Compiler Errors
E0382 (Use of moved value): You tried to use a variable after its ownership was
moved.
E0502 (Mutable/Immutable borrow conflict): You tried to change data while
someone else was reading it.
E0106 (Missing lifetime specifier): You have a reference in a struct but didn’t tell
Rust how long it should live.
Useful Types
BTreeMap: An ordered map (good for deterministic output).
IndexMap: A map that preserves insertion order.
Rc / Arc: Reference counting for shared ownership.
Cell / RefCell: “Interior mutability”—changing data even through an immutable
reference.


---

# Page 43

Appendix C: Performance Tips
1. Arena Allocation: Allocate all AST nodes in one big chunk of memory for speed.
2. String Interning: Store only one copy of every unique string (like variable names).
3. Lazy Evaluation: Don’t calculate things until you absolutely need them.
Appendix D: Advanced Topics
Constraint Solving: Used in advanced type systems (like Rust’s) to find valid types.
Loop Optimization: Techniques like “Loop Unswitching” to move checks outside of
loops.
Garbage Collection Implementation: How to build a system that manages memory
for the user.
Formal Verification: Using math to prove a compiler is 100% bug-free.
Conclusion
You’ve now learned the complete compiler engineering journey.
Total Pages: ~210
Recommended Paper: 20 lb bond, white
Binding: Spiral or comb binding recommended
Chapter 3.7: Formal Grammars and the Chomsky
Hierarchy [Advanced]
Conceptual Explanation: Formal grammars are mathematical systems for describing
languages. They consist of a set of rules that dictate how symbols can be combined to
form valid strings in a language. In compiler design, these grammars precisely define the
syntax of a programming language, allowing the parser to determine if a given program is
syntactically correct. The Chomsky Hierarchy classifies these grammars into four types


---

# Page 44

based on their expressive power and the complexity of the automata required to recognize
the languages they generate.
Mental Model: Imagine a recipe book for building sentences. A formal grammar is that
recipe book, specifying exactly how words (symbols) can be combined to form valid
sentences (programs). The Chomsky Hierarchy is like categorizing these recipe books
based on how complex their rules are. A simple recipe might just list ingredients, while a
complex one might have conditional steps and sub-recipes.
The Chomsky Hierarchy in Compiler Design:
Type Grammar
Name Automaton Language Application in Compilers
Type
0 Unrestricted Turing Machine Recursively
Enumerable
Theoretical; not directly used for
programming language syntax.
Type
1
Context-
Sensitive
Linear-
Bounded
Automaton
Context-
Sensitive
Rarely used for syntax due to
complexity; sometimes for semantic
checks.
Type
2 Context-Free Pushdown
Automaton Context-Free
Most common for programming
language syntax (e.g., C, Java,
Rust). Used by parsers to build
ASTs.
Type
3 Regular Finite
Automaton Regular
Used for lexical analysis (lexers).
Describes tokens (identifiers,
keywords, numbers).
Why it matters for compilers:
Lexical Analysis (Type 3 - Regular Grammars): Regular expressions, which
describe regular languages, are perfect for defining tokens. Finite Automata
(deterministic or non-deterministic) are used to implement lexers efficiently.
Syntax Analysis (Type 2 - Context-Free Grammars): The syntax of most
programming languages can be described by context-free grammars. Parsers (like
LR or LL parsers) use these grammars to construct the Abstract Syntax Tree (AST).
Formal Definition of a Context-Free Grammar (CFG):
A CFG is a 4-tuple (V, T, P, S) where:


---

# Page 45

V: A finite set of non-terminal symbols (variables), representing syntactic
categories (e.g., Expr, Stmt).
T: A finite set of terminal symbols (tokens), which are the actual words/symbols
from the input (e.g., +, if, identifier). V and T are disjoint.
P: A finite set of production rules, each of the form A → β, where A is a non-
terminal and β is a string of symbols from (V ∪  T)*.
S: A distinguished start symbol from V, representing the entire program or the
highest-level syntactic construct.
Example CFG for Simple Arithmetic Expressions:
Expr -> Expr + Term
Expr -> Term
Term -> Term * Factor
Term -> Factor
Factor -> ( Expr )
Factor -> Number
Here:
V = {Expr, Term, Factor} (Non-terminals)
T = {+, *, (, ), Number} (Terminals)
S = Expr (Start symbol)
P is the set of production rules listed above.
This grammar defines how numbers, parentheses, addition, and multiplication can be
combined to form valid expressions. The parser’s job is to take a sequence of tokens and
determine if it can be derived from the start symbol Expr using these rules.
4.5 DFA and NFA Construction [Advanced]
Conceptual Explanation: Lexers are typically implemented using Finite Automata. There
are two main types: Non-deterministic Finite Automata (NFAs) and Deterministic
Finite Automata (DFAs). NFAs are easier to construct from regular expressions but can
be less efficient to execute. DFAs are more complex to build but are highly efficient for
recognizing tokens.


---

# Page 46

Mental Model: Imagine a maze. An NFA is like having multiple paths you can take at any
junction, sometimes even without consuming input. A DFA is like a maze where at every
junction, there’s only one clear path to follow based on the next step. For a computer, a
DFA is much easier to navigate quickly.
NFA Construction (Thompson’s Construction):
Thompson’s Construction is a method to convert a regular expression into an NFA. It
builds NFAs for basic regular expression components (single characters, epsilon) and then
combines them using rules for concatenation, alternation, and Kleene star.
Example: NFA for a|b
Start
a_start b_start
a
a_end
b
b_end
End
DFA Construction (Subset Construction):
The Subset Construction Algorithm converts an NFA into an equivalent DFA. The key
idea is that each state in the resulting DFA corresponds to a set of states in the original
NFA. This process can sometimes lead to a DFA with a larger number of states than the
NFA, but the DFA will always be deterministic.
Algorithm Steps (Simplified):


---

# Page 47

1. Start with an initial DFA state that is the epsilon-closure of the NFA’s start state.
2. For each current DFA state and each input symbol: a. Find all NFA states reachable
from the current NFA states by that input symbol. b. Compute the epsilon-closure of
this new set of NFA states. c. If this new set of NFA states doesn’t correspond to an
existing DFA state, create a new one.
3. Repeat until no new DFA states can be created.
Why convert NFA to DFA?
Efficiency: DFAs are generally faster to simulate than NFAs because they never
have to backtrack or explore multiple paths simultaneously. For each input character,
a DFA transitions to exactly one next state.
Simplicity of Implementation: A DFA can be implemented as a simple state
machine with a transition table, making the lexer code straightforward and highly
optimized.
Trade-offs:
Feature NFA DFA
Construction Easier from regex More complex (subset
construction)
States Fewer states possible Can have many more states
Execution Can be slower (backtracking/multiple
paths) Faster (single path)
Implementation More complex simulation Simpler transition table
5.7 LR and LALR Parsing [Advanced]
Conceptual Explanation: While Recursive Descent and Pratt parsing are top-down
parsing methods, LR parsers are a family of powerful bottom-up parsers. They read the
input from left-to-right and construct a rightmost derivation in reverse. LR parsers are
widely used because they can parse a large class of context-free grammars, detect errors
early, and are efficient.
Mental Model: Imagine building a house from the foundation up. A bottom-up parser starts
with the smallest components (tokens) and combines them into larger structures


---

# Page 48

(expressions, statements) until it forms the complete house (the AST). At each step, it
decides whether to “shift” the next token onto a stack or “reduce” a sequence of symbols
on the stack into a non-terminal, following the grammar rules.
Types of LR Parsers:
Type Description Power Table
Size
Error
Detection
LR(0) Simplest, no lookahead. Limited grammar
support. Least Smallest Early
SLR(1) Simple LR, uses 1-token lookahead. More
powerful than LR(0). Medium Small Early
LR(1) Canonical LR, uses 1-token lookahead. Most
powerful, handles most grammars. Most Largest Earliest
LALR(1) Look-Ahead LR, a compromise between
SLR(1) and LR(1). High Medium Early
Why LALR(1) is popular:
Power: LALR(1) parsers can handle nearly all grammars used for programming
languages.
Efficiency: They are efficient to implement and execute, often using a parsing table
generated by tools like yacc or bison.
Table Size: The parsing tables for LALR(1) grammars are significantly smaller than
those for LR(1) grammars, making them practical for real-world compilers.
LR Parsing Algorithm (High-Level):
An LR parser uses a stack, an input buffer, and a parsing table (composed of ACTION and
GOTO tables).
1. Initialize: Push $ (bottom-of-stack marker) and the initial state onto the stack.
2. Loop: Until the input is accepted or an error occurs: a. Consult ACTION table: Look
at the current state on top of the stack and the current lookahead token from the
input. b. Perform Action: * Shift s: Push the current lookahead token and state s
onto the stack. Advance the input. * Reduce A -> β: Pop |β| symbols from the
stack. Look at the new state on top of the stack and consult the GOTO table with


---

# Page 49

non-terminal A. Push A and the new state onto the stack. * Accept: The input is
successfully parsed. * Error: Report a syntax error.
Example: Shift-Reduce Process
Consider the grammar E -> E + E | id and input id + id.
Stack Input Action
$0 id + id $ Shift 5 (id)
$0 id 5 + id $ Reduce E -> id
$0 E 3 + id $ Shift 6 (+)
$0 E 3 + 6 id $ Shift 5 (id)
$0 E 3 + 6 id 5 $ Reduce E -> id
$0 E 3 + 6 E 4 $ Reduce E -> E + E
$0 E 2 $ Accept
This simplified example illustrates how the parser shifts tokens onto the stack and reduces
them according to grammar rules until the entire input is reduced to the start symbol.
7.6 Dataflow Analysis [Advanced]
Conceptual Explanation: Dataflow analysis is a technique used by compilers to gather
information about the possible set of values computed at various points in a program. This
information is crucial for performing many compiler optimizations. It involves analyzing the
flow of data through the Control Flow Graph (CFG) to determine properties of variables
and expressions.
Mental Model: Imagine you’re tracking a package through a complex delivery network.
Dataflow analysis is like knowing at any point in the network where the package might
have come from (reaching definitions) or where it might be going (live variables). This
knowledge helps you optimize the delivery route or decide if a package is still needed.
Key Dataflow Analyses:
1. Reaching Definitions Analysis:


---

# Page 50

Purpose: Determines, for each program point, which definitions (assignments
to variables) might have reached that point without being overwritten. This is a
forward dataflow analysis.
Application: Useful for constant propagation, common subexpression
elimination, and dead code elimination.
Example: If x = 5 is a reaching definition for a later use of x, the compiler
might replace x with 5.
2. Live Variables Analysis:
Purpose: Determines, for each program point, which variables might be used
in the future before being redefined. This is a backward dataflow analysis.
Application: Crucial for register allocation (only live variables need to be kept
in registers) and dead code elimination (if a variable is not live, its definition
might be dead).
Example: If variable y is not live after y = x + 1, then y does not need to
be stored in a register or memory after this instruction if it’s not used again.
Dataflow Equations (General Form):
Dataflow analyses are often solved iteratively using sets of equations over the CFG. For a
basic block B:
IN[B]: The set of dataflow facts true at the entry of block B.
OUT[B]: The set of dataflow facts true at the exit of block B.
GEN[B]: The set of facts generated within block B.
KILL[B]: The set of facts killed (made invalid) within block B.
Forward Analysis (e.g., Reaching Definitions): OUT[B] = GEN[B] ∪  (IN[B] -
KILL[B]) IN[B] = ∪  (OUT[P]) for all predecessors P of B
Backward Analysis (e.g., Live Variables): IN[B] = USE[B] ∪  (OUT[B] - DEF[B])
OUT[B] = ∪  (IN[S]) for all successors S of B
These equations are solved iteratively until a fixed point is reached, meaning the sets IN
and OUT no longer change. The initial values for IN and OUT are typically empty sets or
universal sets, depending on the analysis type.


---

# Page 51

7.7 SSA Construction (Dominance Frontiers) [Advanced]
Conceptual Explanation: Static Single Assignment (SSA) form is an intermediate
representation property where every variable is assigned a value exactly once. This
simplifies many optimizations because it makes data dependencies explicit. When a
variable is assigned multiple times in the original code, SSA introduces “phi functions” (Φ-
functions) at merge points in the Control Flow Graph (CFG) to select the correct version of
the variable.
Mental Model: Imagine a river with multiple tributaries merging. A phi function is like a
gate at the confluence that decides which tributary’s water (variable value) flows into the
main river. SSA ensures that each time a variable’s value changes, it’s treated as a new,
distinct variable, making it easier to track its lineage.
Dominance Frontiers:
To efficiently place phi functions, compilers use the concept of Dominance Frontiers. A
node Y is in the dominance frontier of a node X if X dominates Y’s immediate
predecessor, but X does not strictly dominate Y. In simpler terms, Y is the first node on
some path from the start node that X does not strictly dominate, but X dominates the
node immediately preceding Y on that path.
Why Dominance Frontiers are important for SSA:
Phi functions are inserted at precisely the nodes in the dominance frontier of any node X
that contains a definition of a variable V. This ensures that at any point where control flow
merges, and multiple definitions of V could reach that point, a phi function is present to
merge these definitions into a single new SSA-form definition.
Algorithm for SSA Construction (High-Level):
1. Build the Control Flow Graph (CFG): Represent the program as a graph of basic
blocks.
2. Compute Dominators: For each node N in the CFG, find all nodes that must be
executed before N (its dominators).
3. Compute Dominance Frontiers: For each node N, compute its dominance frontier
DF(N).
4. Insert Phi Functions: For every variable V and every basic block B that defines
V:


---

# Page 52

For each block Y in DF(B):
Insert a phi function for V at the beginning of Y.
5. Rename Variables: Systematically rename all variable uses and definitions to
ensure each variable has a unique assignment. This involves assigning subscripts
(e.g., x1, x2) and ensuring phi functions correctly merge these versions.
Example (Simplified):
Consider the following code snippet and its transformation to SSA form:
// Original Code
if (cond) {
    x = 1;
} else {
    x = 2;
}
y = x + 3;
// SSA Form (Conceptual)
if (cond) {
    x1 = 1;
} else {
    x2 = 2;
}
x3 = phi(x1, x2); // Phi function merges x1 and x2
y1 = x3 + 3;
In this example, x is defined in two different branches. At the merge point after the if-
else statement, a phi function phi(x1, x2) is introduced to create a new variable x3,
which represents the value of x that reaches that point. This x3 is then used in the
subsequent calculation y1 = x3 + 3.
Chapter 10.2: Common Optimization Passes [Advanced]
Conceptual Explanation: Compiler optimizations are transformations applied to the
intermediate representation (IR) of a program to improve its performance (e.g., faster
execution, less memory usage) or reduce its size, without changing its observable
behavior. These passes often rely on information gathered during dataflow analysis.


---

# Page 53

Mental Model: Think of an optimization pass as a specialized engineer who takes a
blueprint (IR) and refines it. One engineer might remove unnecessary parts (dead code
elimination), another might pre-assemble simple components (constant folding), while a
third might streamline repetitive tasks (loop unrolling).
10.2.1 Constant Folding [Advanced]
Conceptual Explanation: Constant folding is a compiler optimization that evaluates
constant expressions at compile time and replaces them with their computed values. This
reduces the amount of computation needed at runtime.
Mental Model: If you know 2 + 2 is 4 before you even start building, why wait to
calculate it? Just write down 4 directly. This saves the computer from doing the addition
later.
Algorithm (High-Level):
1. Traverse the Abstract Syntax Tree (AST) or Intermediate Representation (IR).
2. When an expression node is encountered (e.g., ADD, MULTIPLY): a. Recursively
evaluate its operands. b. If all operands are constants, perform the operation and
replace the expression node with a constant node containing the result.
Example:
Before:
let x = 10 * 5 + (20 / 4);
After (after constant folding):
let x = 50 + 5;
let x = 55;


---

# Page 54

10.2.2 Dead Code Elimination (DCE) [Advanced]
Conceptual Explanation: Dead code elimination is an optimization that removes code
that is either unreachable (never executed) or whose results are never used. This reduces
program size and can improve execution speed by removing unnecessary instructions.
Mental Model: If you have a machine part that is never connected to anything or a path in
a factory that no one ever walks down, you can simply remove it. It doesn’t affect the final
product or process.
Algorithm (High-Level, often based on Live Variables Analysis):
1. Perform Live Variables Analysis to determine which variables are live at each
program point.
2. Iterate through the IR instructions.
3. An instruction I is considered dead if: a. It assigns a value to a variable V, and V
is not live after I. b. It is unreachable (e.g., due to a previous optimization like
constant folding making a conditional branch always false).
4. Remove all identified dead instructions.
Example:
Before:
let x = 10; // Definition of x
let y = 20; // Definition of y
println!("Value of x: {}", x);
// y is never used after this point
After (after dead code elimination):
let x = 10;
println!("Value of x: {}", x);
// The definition of y has been removed as it's dead code.


---

# Page 55

10.2.3 Function Inlining [Advanced]
Conceptual Explanation: Function inlining is an optimization where a function call is
replaced by the actual body of the called function. This eliminates the overhead of a
function call (stack frame setup, parameter passing, return address management) and can
expose further optimization opportunities for the compiler.
Mental Model: Instead of sending a messenger to get a specific tool from another room,
you just bring the tool directly to where you need it. This saves the trip and allows you to
use the tool immediately.
Algorithm (High-Level):
1. Identify small, frequently called functions (heuristics are often used).
2. For each call site of such a function: a. Replace the function call instruction with the
IR instructions from the function’s body. b. Adjust variable names and control flow to
integrate the inlined code correctly.
Example:
Before:
fn add_one(x: i32) -> i32 {
    x + 1
}
let a = 5;
let b = add_one(a);
After (after inlining add_one):
let a = 5;
let b = a + 1; // Function call replaced by its body


---

# Page 56

10.2.4 Loop Unrolling [Advanced]
Conceptual Explanation: Loop unrolling is an optimization that transforms a loop by
replicating its body multiple times and adjusting the loop control code. This reduces the
number of loop iterations and the overhead associated with loop control (e.g., loop
condition checks, incrementing loop counters).
Mental Model: Instead of repeatedly telling a robot to do a small task (like picking up one
item), you tell it to pick up three items at once, reducing the number of times you have to
give instructions.
Algorithm (High-Level):
1. Identify loops that are good candidates for unrolling (e.g., loops with a small, fixed
number of iterations or loops where the number of iterations can be determined at
compile time).
2. Choose an unroll factor k (how many times to replicate the loop body).
3. Replicate the loop body k times.
4. Adjust the loop bounds and increment step to account for the unrolling. A
10.2.5 Instruction Selection (Tree Tiling) [Advanced]
Conceptual Explanation: Instruction selection is the process of mapping the operations in
the Intermediate Representation (IR) to the specific instructions available on the target
machine (e.g., x86, ARM). This is a critical step in code generation, as the choice of
instructions can significantly impact the performance and size of the generated machine
code.
Mental Model: Imagine you have a complex design (the IR) and a toolbox full of
specialized tools (machine instructions). Instruction selection is like choosing the best
combination of tools to build each part of the design efficiently. Some tools might be
general-purpose, while others are highly specialized for certain tasks.
Tree Tiling:
One common approach to instruction selection, especially when the IR is represented as a
tree (like an AST or a tree-based IR), is Tree Tiling (also known as Tree Pattern
Matching). In this method:


---

# Page 57

1. Define Tiles: Each machine instruction is represented as a small tree pattern, or
“tile.” These tiles represent the operations that a single machine instruction can
perform.
2. Match and Cover: The instruction selector traverses the IR tree and attempts to
“cover” it with these instruction tiles. The goal is to find a set of tiles that completely
cover the IR tree with the minimum cost (e.g., fewest instructions, fastest execution
time).
3. Dynamic Programming: Often, dynamic programming techniques are used to find
the optimal tiling. Each node in the IR tree is annotated with the minimum cost to
generate code for the subtree rooted at that node, considering all possible instruction
patterns that could cover it.
Example: Tree Tiling for a = b + c
Consider an IR tree for a = b + c:
      Assign
     /      \
  Var(a)   Add
          /   \
       Var(b) Var(c)
Possible machine instruction tiles might be:
ADD R1, R2, R3 (cost 1): Covers Add(Var, Var)
LOAD R1, Mem (cost 1): Covers Var
STORE Mem, R1 (cost 1): Covers Assign(Var, Reg)
The instruction selector would try to match these patterns to generate code like:
LOAD R1, b
LOAD R2, c
ADD R3, R1, R2
STORE a, R3
Advantages of Tree Tiling:


---

# Page 58

Retargetability: To target a new machine, you only need to define a new set of
instruction tiles for that architecture.
Optimality: With dynamic programming, it can find optimal (or near-optimal)
instruction sequences for a given cost model.
Modularity: The instruction selection logic is separated from the IR representation
and the target machine details.
10.2.6 Register Allocation (Graph Coloring) [Advanced]
Conceptual Explanation: Register allocation is the process of assigning program
variables to the limited number of CPU registers available on the target machine. Registers
are the fastest memory locations, so efficient register allocation is crucial for generating
high-performance code. When there are more variables that are “live” (their values might
be used later) than available registers, some variables must be “spilled” to main memory.
Mental Model: Imagine you have a small workbench (CPU registers) and many tools
(variables) you need to use. Register allocation is like deciding which tools to keep on the
workbench for quick access and which ones to put back in the toolbox (memory) when
they’re not immediately needed. If you need a tool that’s in the toolbox, you have to go get
it, which takes time.
Graph Coloring Algorithm (Chaitin’s Algorithm):
One of the most widely used and effective algorithms for global register allocation is based
on graph coloring. The steps are:
1. Build the Interference Graph:
Create a node for each variable that needs to be allocated to a register.
Draw an edge between two nodes (variables) if they are “live” at the same time
(i.e., their lifetimes overlap). This means they “interfere” with each other and
cannot be assigned to the same physical register.
2. Simplify the Graph:
Repeatedly remove nodes with a degree (number of edges) less than K,
where K is the number of available registers. These nodes can always be
colored. Push the removed nodes onto a stack.
3. Spill if Necessary:


---

# Page 59

If, at any point, all remaining nodes have a degree greater than or equal to K,
then a variable must be “spilled” to memory. Choose a node (variable) to spill
(often based on heuristics like usage frequency or loop depth), remove it from
the graph, and continue simplifying.
4. Select Colors (Assign Registers):
Pop nodes from the stack. For each node, assign it a color (register) that is not
used by any of its neighbors (interfering variables).
If a node was chosen to be spilled, it is not assigned a register; instead, code is
inserted to load and store its value from memory.
Example (Simplified):
Consider variables a, b, c, d and 3 available registers (R1, R2, R3).
Interference Graph:
a interferes with b, c
b interferes with a, d
c interferes with a, d
d interferes with b, c
Coloring:
a -> R1
b -> R2
c -> R3
d -> R1 (since d does not interfere with a)
Advantages of Graph Coloring:
Optimal (for K-colorable graphs): If the interference graph is K-colorable, the
algorithm finds an optimal register assignment.
Global Scope: Considers variable liveness across the entire function, leading to
better register utilization than local allocation schemes.
Challenges:


---

# Page 60

NP-Completeness: Graph coloring is an NP-complete problem in general, so
heuristics are used for practical compilers. This means the solution might not always
be perfectly optimal, but it’s usually very good.
Spilling: Deciding which variables to spill when registers run out is a complex
heuristic problem that can significantly impact performance.
10.2.7 Calling Conventions and ABI [Advanced]
Conceptual Explanation: A Calling Convention is a low-level agreement between a
caller function and a callee function on how to pass arguments, receive return values, and
manage the call stack. It dictates things like which registers are used for arguments,
whether arguments are pushed onto the stack in left-to-right or right-to-left order, and who
is responsible for cleaning up the stack after a function call. The Application Binary
Interface (ABI) is a broader concept that encompasses calling conventions, data type
layouts, object file formats, and other low-level details that allow different pieces of
compiled code (e.g., libraries compiled by different compilers) to interoperate.
Mental Model: Imagine two people speaking different languages trying to communicate. A
calling convention is like a set of agreed-upon rules for their conversation: “You speak first,
then I speak. I’ll write down my main points, and you’ll write down yours.” The ABI is the
entire dictionary, grammar, and cultural context that allows them to understand each other
at a fundamental level, even if they use different dialects.
Key Aspects of Calling Conventions:
1. Parameter Passing:
Registers: Arguments are passed in a specific set of registers (e.g., rdi,
rsi, rdx, rcx, r8, r9 for the first six arguments on x86-64 System V ABI).
This is the fastest method.
Stack: If there are more arguments than available registers, the remaining
arguments are pushed onto the stack. The order (left-to-right or right-to-left)
varies by convention.
2. Return Values:
Small return values (e.g., integers, pointers) are typically returned in a specific
register (e.g., rax on x86-64).


---

# Page 61

Larger return values (e.g., structs) might be returned via memory, often by
passing a pointer to the return value’s location as a hidden first argument.
3. Stack Management:
Caller-save vs. Callee-save Registers: Some registers are designated as
“caller-save” (the caller must save their values before a call if it needs them
after) and others as “callee-save” (the callee must save and restore their values
if it modifies them).
Stack Frame: Each function call creates a new stack frame, which holds local
variables, saved registers, and return addresses.
4. Stack Cleanup:
Caller Cleans: The caller is responsible for popping arguments off the stack
after the function returns.
Callee Cleans: The callee is responsible for popping arguments off the stack
before returning.
Example (x86-64 System V ABI - Simplified):
; Caller side
mov rdi, arg1_value   ; First argument in RDI
mov rsi, arg2_value   ; Second argument in RSI
call my_function      ; Call the function
; Result is in RAX
; Callee side (my_function)
push rbp              ; Save base pointer
mov rbp, rsp          ; Set new base pointer
; ... function body using rdi, rsi for args ...
mov rax, return_value ; Set return value in RAX
pop rbp               ; Restore base pointer
ret                   ; Return
Why ABIs are crucial:
Interoperability: ABIs ensure that code compiled by different compilers, or even
different versions of the same compiler, can link and run together. This is
fundamental for using libraries and operating system services.


---

# Page 62

Language Bindings: When a program written in one language (e.g., Rust) calls a
function written in another (e.g., C), they must adhere to a common ABI.
Debugging: Understanding the ABI is essential for debugging at the assembly level,
as it explains how data is laid out and passed around.
Different architectures (x86, ARM, RISC-V) and operating systems (Linux, Windows,
macOS) often have their own specific ABIs. Compilers must generate code that strictly
adheres to the target ABI.
Chapter 8.5: Runtime Systems and Memory
Management [Advanced]
Conceptual Explanation: While Rust is a systems programming language that
emphasizes manual memory management and ownership, many higher-level languages
(like Java, Python, Go, JavaScript) rely on Garbage Collection (GC) for automatic
memory management. If you were building a compiler for such a language, implementing a
garbage collector would be a crucial part of your runtime system. A garbage collector
automatically reclaims memory that is no longer reachable or used by the program,
preventing memory leaks and simplifying programming.
Mental Model: Imagine a busy office where employees (your program) create many
documents (objects in memory). Instead of each employee manually shredding their old
documents, a dedicated cleaning crew (the garbage collector) periodically comes through,
identifies which documents are no longer in use, and disposes of them. This frees up
space and allows employees to focus on their work.
Common Garbage Collection Algorithms:
1. Reference Counting:
Mechanism: Each object keeps a count of how many references point to it.
When the count drops to zero, the object is immediately deallocated.
Pros: Simple to implement, memory is reclaimed immediately, good locality.
Cons: Cannot collect cyclic data structures (e.g., A refers to B, B refers to A,
but neither is reachable from the root), overhead on every reference
assignment/deletion.


---

# Page 63

Example (Rust’s Rc and Arc): Rust uses Rc (Reference Counted) and
Arc (Atomic Reference Counted) for shared ownership, which is a form of
reference counting, but it’s explicit and not a full-fledged garbage collector.
2. Mark-and-Sweep:
Mechanism: A two-phase algorithm. The “mark” phase starts from a set of root
objects (e.g., global variables, stack variables) and recursively traverses all
reachable objects, marking them as “live.” The “sweep” phase then iterates
through the entire heap, deallocating all unmarked (dead) objects.
Pros: Can collect cyclic data structures, relatively simple to understand.
Cons: “Stop-the-world” pauses (the program must halt during GC), can lead to
memory fragmentation.
3. Generational Garbage Collection:
Mechanism: Based on the observation that most objects die young. The heap
is divided into “generations” (e.g., young generation, old generation). New
objects are allocated in the young generation. GC runs more frequently on the
young generation, and objects that survive multiple young generation
collections are promoted to the old generation.
Pros: Reduces GC pause times by focusing on smaller, more volatile parts of
the heap, highly efficient for typical object allocation patterns.
Cons: More complex to implement than simple mark-and-sweep.
Implementing a Garbage Collector (High-Level):
Heap Management: You need a custom memory allocator to manage the program’s
heap, tracking allocated objects.
Root Set Identification: The GC needs to know where to start its traversal (e.g.,
registers, global variables, stack frames).
Object Metadata: Each object needs metadata to indicate its type and where its
internal pointers are, so the GC can correctly traverse the object graph.
Integration with Compiler: The compiler needs to generate code that cooperates
with the GC, such as writing barriers (to track changes to pointers) or read barriers
(to track reads of pointers) for concurrent or generational collectors.


---

# Page 64

While Rust itself doesn’t have a runtime GC, understanding these algorithms is crucial for
building compilers for languages that do, or for implementing specialized memory
management within a Rust-based runtime system.
Chapter 8.6: Just-In-Time (JIT) Compilation [Advanced]
Conceptual Explanation: Just-In-Time (JIT) compilation is a technique used in many
modern language runtimes (like Java Virtual Machine, JavaScript engines, .NET Common
Language Runtime) where code is compiled into machine code during program execution,
rather than before execution (Ahead-Of-Time, AOT compilation). JIT compilers can
achieve higher performance than interpreters and sometimes even AOT compilers
because they can perform optimizations based on runtime information (e.g., profiling data,
actual data types).
Mental Model: Imagine a chef who doesn’t prepare all meals in advance. Instead, when
an order comes in, they quickly cook it, but as they cook, they learn which ingredients are
most popular or which steps are repeated often. They then optimize their cooking process
for those popular dishes or steps, making subsequent orders faster. An interpreter is like a
chef who always follows the recipe step-by-step without learning. An AOT compiler is like a
chef who prepares all meals in advance without knowing what will be popular.
JIT Compilation Process (High-Level):
1. Interpretation: Initially, code (often bytecode or an intermediate representation) is
interpreted for quick startup.
2. Profiling: The runtime system monitors the executing code to identify “hot spots”—
sections of code that are executed frequently.
3. Compilation: When a hot spot is identified, the JIT compiler compiles that specific
section of code into optimized machine code.
4. Execution: Subsequent executions of the hot spot directly use the compiled
machine code, bypassing the interpreter.
5. Deoptimization (Optional): If runtime assumptions made during optimization prove
false, the JIT can deoptimize the code and revert to interpretation or less optimized
compilation.
JIT vs. AOT Compilation:


---

# Page 65

Feature Just-In-Time (JIT) Compilation Ahead-Of-Time (AOT)
Compilation
When Compiled During runtime, as needed Before runtime, during build
Optimization
Scope
Can use runtime profiling data for dynamic
optimizations.
Limited to static analysis; no
runtime data.
Startup Time Slower initial startup (due to interpretation
and compilation overhead).
Faster startup (code is already
compiled).
Peak
Performance
Potentially higher peak performance due
to runtime-specific optimizations.
Consistent performance, but
may not reach JIT peak.
Memory Usage Higher (stores both IR/bytecode and
compiled machine code).
Lower (only stores compiled
machine code).
Portability More portable (bytecode can run on any
platform with a JIT).
Less portable (machine code is
platform-specific).
Challenges in JIT Compilation:
Compilation Overhead: The time spent compiling code must be less than the time
saved by executing optimized code. This requires fast compilation algorithms.
Memory Footprint: Storing both the intermediate representation and the generated
machine code can increase memory usage.
Complexity: JIT compilers are inherently more complex than AOT compilers due to
the need for runtime profiling, dynamic optimization, and potential deoptimization.
Despite these challenges, JIT compilation is a cornerstone of performance in many
modern dynamic languages and managed runtimes, enabling them to achieve
performance levels competitive with statically compiled languages.
References
[1] Aho, A. V., Lam, M. S., Sethi, R., & Ullman, J. D. (2006). Compilers: Principles,
Techniques, and Tools (2nd Edition). Pearson Education. (Commonly known as the
“Dragon Book”).


---

# Page 66

[2] Cooper, K. D., & Torczon, L. (2011). Engineering a Compiler (2nd Edition). Morgan
Kaufmann.
[3] Appel, A. W., & Palsberg, J. (2002). Modern Compiler Implementation in Java (2nd
Edition). Cambridge University Press.
[4] Muchnick, S. S. (1997). Advanced Compiler Design and Implementation. Morgan
Kaufmann.
[5] Cytron, R., Ferrante, J., Rosen, B. K., Wegman, M. N., & Zadeck, F. K. (1991).
Efficiently computing static single assignment form and the control dependence graph.
ACM Transactions on Programming Languages and Systems (TOPLAS), 13(4), 451-490.
[6] Chaitin, G. J. (1982). Register allocation & spilling via graph coloring. ACM SIGPLAN
Notices, 17(6), 98-101.
[7] Nystrom, R. (2021). Crafting Interpreters. Genever Benning.
[8] The Rust Project Developers. The Rust Programming Language. https://doc.rust-
lang.org/book/
[9] LLVM Project. LLVM Language Reference Manual. https://llvm.org/docs/LangRef.html
[10] Wikipedia contributors. Just-in-time compilation. Wikipedia, The Free Encyclopedia.
https://en.wikipedia.org/wiki/Just-in-time_compilation