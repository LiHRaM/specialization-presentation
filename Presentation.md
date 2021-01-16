---
title: "Specialization Course"
theme: white
highlightTheme: github
transition: none
---

### Is Rust Used Safely by Software Developers?

Ana Nora Evans, Bradford Campbell, and Mary Lou Soffa, 2020.
*42nd International Conference on Software Engineering (ICSE '20), May 23-29, 2020, Seoul, Republic of Korea.*

Presented by Hilmar Gústafsson

---

## The Problem
* Rust is a new language marketed as memory-safe.
* Rust's `unsafe` keyword bypasses guarantees for low-level programming.
* Is `unsafe` used safely in practice?

---

## Rust Language
* **Performance**: No garbage collection, very small runtime, "zero" cost abstractions.
* **Reliability**: Strong type system and Ownership model `==` Memory-safety and Thread-safety (?)
* **Productivity**: Well-documented, good error messages, great tooling, etc.

--

## Hello, world!
```rust
fn main() {
    println!("Hello, world!");
}
```

Running
<pre><font color="#4E9A06"><b>$</b></font> cargo run
<font color="#4E9A06"><b>   Compiling</b></font> hello v0.1.0 (/home/user/hello)
<font color="#4E9A06"><b>    Finished</b></font> dev [unoptimized + debuginfo] target(s) in 0.21s
<font color="#4E9A06"><b>     Running</b></font> `target/debug/hello`
Hello, world!</pre>

--

## Ownership

```rust
fn main() {
    let buffer = String::new();
    eat_buffer(buffer);
    let buffer_copy = buffer;
}

fn eat_buffer(buffer: String) {
    // ...
}   // <-- buffer dropped here
```

--

## Ownership

<pre><font color="#EF2929"><b>error[E0382]</b></font><b>: use of moved value: `buffer`</b>
 <font color="#729FCF"><b>--&gt; </b></font>src/main.rs:4:23
  <font color="#729FCF"><b>|</b></font>
<font color="#729FCF"><b>2</b></font> <font color="#729FCF"><b>| </b></font>    let buffer = String::new();
  <font color="#729FCF"><b>| </b></font>        <font color="#729FCF"><b>------</b></font> <font color="#729FCF"><b>move occurs because `buffer` has type 
    `String`, which does not implement the `Copy` trait</b></font>
<font color="#729FCF"><b>3</b></font> <font color="#729FCF"><b>| </b></font>    eat_buffer(buffer);
  <font color="#729FCF"><b>| </b></font>               <font color="#729FCF"><b>------</b></font> <font color="#729FCF"><b>value moved here</b></font>
<font color="#729FCF"><b>4</b></font> <font color="#729FCF"><b>| </b></font>    let buffer_copy = buffer;
  <font color="#729FCF"><b>| </b></font>                      <font color="#EF2929"><b>^^^^^^</b></font> <font color="#EF2929"><b>value used here after move</b></font>

<font color="#EF2929"><b>error</b></font><b>: aborting due to previous error</b></pre>

--

## Borrowing
```rust
fn main() {
    let buffer = String::new();
    borrow_buffer(&buffer);
    let buffer_copy = buffer;
}
 
fn borrow_buffer(buffer: &String) {
    // read buffer
}   // <-- reference dropped here
```
No problems. :)

--

## Borrowing

```rust
fn main() {
    let buffer = String::new();
    borrow_mut_buffer(&buffer);
    let buffer_copy = buffer;
}
 
fn borrow_mut_buffer(buffer: &mut String) {
    // change buffer
}   // <-- reference dropped here
```

--

## No Dangling Pointers
```rust
fn main() {
    let pointer: &i32 = get_pointer();
}

fn get_pointer() -> &i32 {
    let val = 0;
    &val
}
```

--

## No Dangling Pointers

<pre><font color="#EF2929"><b>error[E0106]</b></font><b>: missing lifetime specifier</b>
 <font color="#729FCF"><b>--&gt; </b></font>src/main.rs:5:21
  <font color="#729FCF"><b>|</b></font>
<font color="#729FCF"><b>5</b></font> <font color="#729FCF"><b>| </b></font>fn get_pointer() -&gt; &amp;i32 {
  <font color="#729FCF"><b>| </b></font>                    <font color="#EF2929"><b>^</b></font> <font color="#EF2929"><b>expected named lifetime parameter</b></font>
  <font color="#729FCF"><b>|</b></font>
  <font color="#729FCF"><b>= </b></font><b>help</b>: this function&apos;s return type contains a borrowed
    value, but there is no value for it to be borrowed from
<font color="#34E2E2"><b>help</b></font>: consider using the `&apos;static` lifetime
  <font color="#729FCF"><b>|</b></font>
<font color="#729FCF"><b>5</b></font> <font color="#729FCF"><b>| </b></font>fn get_pointer() -&gt; &amp;&apos;static i32 {
  <font color="#729FCF"><b>| </b></font>                    <font color="#34E2E2"><b>^^^^^^^^</b></font>

<font color="#EF2929"><b>error</b></font><b>: aborting due to previous error</b></pre>


---


## Function Taxonomy

Most use of unsafe is through function calls.

![Function taxonomy, safe, possibly unsafe, declared unsafe.](./func_taxonomy.svg)

--

## Function Taxonomy

* Declared Unsafe: `unsafe` in function declaration
* Possibly Unsafe: Declared unsafe, wrapping unsafe, or may call possibly unsafe function.
* Safe: Fully guaranteed to be memory safe at compile-time.

--


## Function Taxonomy -- Ex. 1

```rust
// Simple, safe function
fn add(a: usize, b: usize) -> usize {
    a + b
}
```

--

## Function Taxonomy -- Ex. 2

```rust
unsafe fn declared() {}
```

```rust
fn maybe() {
    unsafe { dangerous(); }
}
```

```rust
fn also_maybe() {
    if cond { /* safe operations */ }
    else { maybe_dangerous(); }
}
```

---


## Research Questions

1. How much do developers use *Unsafe Rust*?
1. How much of the Rust code is *Unsafe Rust*?
1. What *Unsafe Rust* operations are used in practice?
1. What abstract binary interfaces (programming languages) are used in the *declared unsafe* functions?
1. Does the use of *Unsafe Rust* change over time?
1. Why do Rust developers use unsafe?

---

## Approach

1. Parse crates to find instances of `unsafe`.
1. Generate two call graphs for each library.
    1. Optimistic: More functions are safe
    1. Conservative: More functions are unsafe
1. Propagate `unsafe` condition to functions.

Survey on Reddit about uses of `unsafe`.

--

## Call Graph Analysis

<pre>
<b>Data</b>: call graph
<b>Result</b>: list of possibly unsafe functions
<b>for</b> <i>all function definitions</i> <b>do</b>
    <b>if</b> <i>function has unsafe in body</i> <b>then</b>
        add function to possibly unsafe list;
        add function's call graph node to worklist;
    <b>end</b>
<b>end</b>
reverse the call graph;
<b>while</b> <i>worklist not empty</i> <b>do</b>
    current_func = pop the first element of the worklist 
    <b>for</b> <i>each neighbor of current_func</i> <b>do</b>
        <b>if</b> <i>neighbor not in possibly unsafe list</i> <b>then</b>
            add function to possibly unsafe list;
            add neighbor to worklist;
        <b>end</b>
    <b>end</b>
<b>end</b>
</pre>

---

## Conclusion

* Useful study of the propagation of unsafe in Rust.
* Does not answer the initial question.
* Should be compared to **How Do Programmers Use Unsafe Rust?**, which has different results using similar methods.