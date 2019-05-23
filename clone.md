---
title: Rebuffing the Attack of the Clones
description: How to use, and not to use, `clone`
layout: post
date: 2019-05-21T00:00:00Z
---

I left [a comment](https://news.ycombinator.com/item?id=19920747) on HN about how I teach new users
not to clone, and someone asked me if I'd written more, so here we go!

When new users are writing Rust code, it's reasonably common to "fight with the borrow checker". I
personally find this to be quite a misnomer, as it is not a fight so much as a misunderstanding of
the nature of Rust's semantics. In an attempt to make the errors go away, users will understandably
take the path of least resistance, according to the docs they've got. They dutifully read up, and
eventually land on the `clone` method. This makes the errors go away! Great! Except if you were my newbie, in the code review I'd tell you **`clone` is banned unless you can tell me why you need it**.

## A simple example

Let's consider a simple example. I'm going to write some C, and then write the same thing in Rust,
and we'll discuss how they differ.

```c
#include <string.h>
#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>

/* A thing struct that holds a pointer to some data */
typedef struct {
    char* name;
} AThing;

/* A method to operate on that data */
size_t thinglen(const AThing *const thing) {
    return strlen(thing->name);
}

int main() {
    char* name = (char*)malloc(11);
    if (!name) return -1;
    strcpy("rico suave", name);
    AThing my_thing = { .name = name };
    printf("%s is %lu chars long\n", name, thinglen(&my_thing));
    free(name);

return 0;
}
```

Okay, so that's ... not the best C ever written. But it gets the job done and it's intended to be a
simple illustration. Let's try writing a version in naive Rust:

```rust
/// Our thing
struct AThing {
    /// Some data associated with our thing
    pub name: String,
}

impl AThing {
    /// Do some operation upon this thing
    pub fn thing_len(&self) -> usize {
        self.name.len()
    }
}

fn main() {
    let name = String::from("don juan");
    let my_thing = AThing { name: name };
    println!("{} is {} chars long", name, my_thing.thing_len());
}
```

Great! You know that you used a pointer in the C code, but trying to make the type of `AThing::name`
a `&str` made the compiler mad and you fell down a rabbit hole reading about lifetimes. Rust has
`String`, so we'll use that. You probably predicted this, but the compiler will further complain
about the above example:

```
error[E0382]: borrow of moved value: `name`
  --> src/main.rs:12:37
   |
10 |     let name = String::from("don juan");
   |         ---- move occurs because `name` has type `std::string::String`, which does not implement the `Copy` trait
11 |     let my_thing = AThing { name: name };
   |                                   ---- value moved here
12 |     println!("{} is {} chars long", name, thinglen(my_thing));
   |                                     ^^^^ value borrowed here after move
```

Oh bother. So you go and you read up on what the `Copy` trait is, and the docs say you can derive
it. Easy! So you try, and it fails. So you do some more reading, and you fight with the borrow
checker a bit, and maybe eventually you land on

```
    let my_thing = AThing { name: name.clone() };
```

and the code compiles and runs like you expect.  This is admittedly a contrived example, and only really illustrates the kind of error that every tutorial covers in better depth than this post is intended to do.

The important part is this: cloning is almost always the wrong solution, but it's an alluringly easy
way to make the compiler stop bugging you and let you get on with your work. So how can you, as a
novice Rust programmer, know when it's the right call and when it's not?

## What Does `clone` Do?

I'm not going to regurgitate [the
docs](https://doc.rust-lang.org/std/marker/trait.Copy.html#whats-the-difference-between-copy-and-clone)
here. They're exceptionally well-written. Give them a read after we have our chat.

At its simplest, cloning makes a copy of the data, in a way that makes sure any references and
pointers internal to the data are valid (usually by copying the data behind the pointer). Most of
the time, you're only going to care about library types like `String` and `Vec` when you're deciding
whether to `clone`. This means the stdlib has taken care of making sure "any references and pointers
internal to the data are valid". If you're implementing `Clone` yourself, it's almost certainly
going to consist of you calling `clone` on some private objects that themselves implement `Clone`.
It's clones all the way down. Either way, you end up with a new copy of the data.

This is the first big clue: do you really need a whole new copy of the data? Irrespective of the demands
of the compiler, does it make sense _from a design perspective_ to have two copies of the data? Take a
look at the above code examples. The C code has the `AThing` struct keep a pointer to the data. That
means you can pass the pointer around. This makes sense, right? You only really need the string to
exist in one place at a time in this simple example. The Rust code clones the string. Is that
sensical? Do you need multiple copies of "don juan" in memory? Maybe you do! But probably you don't.

It works similarly for vectors. If you're cloning the vector, you're saying that you need two entire
copies of everything in the vector. Do you, really? Likewise for HashMaps, or any other collection.

## What Does a Bad Clone Look Like?

So now I've told you all this, but the compiler is still refusing to build your code, or I am still
refusing to approve your pull request. You're stuck! I've encountered a few anti-patterns that Rust
newbies encounter that can be instructive.

### Copying a `String` when a `&str` Will Do

The first pattern is calling `clone` on a `String`. This is occasionally a totally reasonable thing
to do, but very often you don't actually need it. Let's consider two examples:

```rust
fn make_lipographic(banned: char, line: String) -> String {
    line.as_str().chars().filter(|&c| c != banned).collect()
}

fn main() {
    let passage = String::from("If Youth, throughout all history, had had a champion to stand up for it");
    assert_eq!(make_lipographic('e', passage.clone()), passage);
}
```

The problem with this code is pretty straightforward: as in C, the `make_lipographic` function
doesn't need to _own_ the `line`. You wouldn't pass it by value in C (ignoring, for a moment, the
difficulty in doing such a thing), so you shouldn't move it in Rust. Instead, have it take a
reference (in C, a `const * const`):

```rust
fn make_lipographic(banned: char, line: &str) -> String {
    line.as_str().chars().filter(|&c| c != banned).collect()
}
```

Okay, but what about more complicted situations? Let's say you want to store a value in a `HashMap`,
and then retrieve it later.

```rust
use std::collections::HashMap;

fn main() {
    // this String e.g. is read from a file or from stdin.
    let quote = String::from("Up to about its primary school days a child thinks, naturally, only of play.");

    // A map to store example quotes and the character it avoids.
    let mut lipogram_map: HashMap<String, char> = HashMap::new();

    // store the quote
    lipogram_map.insert(quote, 'e');

    // get the quote back out... oops!
    println!("quote [{}] is missing character '{}'", quote, lipogram_map.get(&quote).unwrap());
}
```

This example is, of course, a bit contrived, but compiling it will fail:

```
error[E0382]: borrow of moved value: `quote`
  --> src/main.rs:10:54
   |
6  |     let quote = String::from("Up to about its primary school days a child thinks, naturally, only of play.");
   |         ----- move occurs because `quote` has type `std::string::String`, which does not implement the `Copy` trait
7  |
8  |     lipogram_map.insert(quote, 'e');
   |                         ----- value moved here
9  |
10 |     println!("quote [{}] is missing character '{}'", quote, lipogram_map.get(&quote).unwrap());
   |                                                      ^^^^^ value borrowed here after move
```

This error message probably looks a bit similar, since it also complains about not implementing
`Copy`. In this case, the `HashMap` _owns_ the `String`s it uses as keys. That is, in order to
insert a value, you must _move_ the key into it... but once you do that, you don't have it any more!

Cloning the data is, in this case, possibly a good choice. If you have confidence that the key
(which, remember, the `HashMap` owns) can be recreated, or doesn't need to be unique in memory, then
you may find cloning to be a better option than trying to use some type of smart pointer to
eliminate duplication of data in memory.

### Cloning a `Vec` when iterating

Another common pattern is when a user will clone a collection when trying to iterate over it. Here's
an example of an attempt at iteration that will fail to compile:

```rust
fn main() {
    let mut bad_letters = vec!['e', 't', 'o', 'i'];
    for l in bad_letters {
        // do something here
    }
    bad_letters.push('s');
}
```

This results in the following error:

```
error[E0382]: borrow of moved value: `bad_letters`
 --> src/main.rs:6:5
  |
2 |     let mut bad_letters = vec!['e', 't', 'o', 'i'];
  |         --------------- move occurs because `bad_letters` has type `std::vec::Vec<char>`, which does not implement the `Copy` trait
3 |     for l in bad_letters {
  |              ----------- value moved here
...
6 |     bad_letters.push('s');
  |     ^^^^^^^^^^^ value borrowed here after move
```

Once again, the compiler appears to be telling us that we need to clone the `Vec` before we can
iterate over it:


```rust
fn main() {
    let mut bad_letters = vec!['e', 't', 'o', 'i'];
    for l in bad_letters.clone() {
        // do something here
    }
    bad_letters.push('s');
}
```

...but that doesn't make sense from a design perspective. Why would we need to copy it if we're not doing anything to, or even with, it? The reason for this is that the `for _ in expr` syntax is _syntactic sugar_ that uses [`IntoIterator::into_iter`](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html#tymethod.into_iter).

That is to say, a `for` loop is the equivalent to:

```rust
fn main() {
    let mut bad_letters = vec!['e', 't', 'o', 'i'];
    let mut iter = bad_letters.into_iter();
    loop {
        match iter.next() {
            Some(c) => {
                // do something here
            },
            None => break,
        }
    }
    bad_letters.push('n')
}
```

The relevant signature is:

```rust
    fn into_iter(mut self) -> IntoIter<T>;
```

Note that _this function moves `self`_. When we add a `clone` into the mix, what we've written is:

```rust
    let iter = bad_letters.clone().into_iter();
    loop {
        match iter.next() { // ...
```

This way, the iterator is created over its own copy (an "owned copy") of `bad_letters` that
`into_iter` can consume. Put another way, we move a new owned copy of `bad_letters` into
`into_iter`. It makes the compiler error go away, but it changes the meaning of our program
considerably.

The need, therefore, is to find a way to construct the loop so that `bad_letters` is not moved. The
solution is easy: move a reference. References in Rust are free and can be created and destroyed
freely! We can make a reference, and move it into `into_iter`, and still own the original vector!

```rust
    for l in &bad_letters { // that's it! the code now compiles and does what you expect
        // ...
    }
```

Or, the desugared version:

```rust
    let mut iter = &bad_letters.into_iter();
    // ...
```

I had a whole write up about this, but then I found [this blog
post](http://xion.io/post/code/rust-for-loop.html) by Karol Kuczmarski, which is _excellent_. I
highly recommend reading and understanding it. It gets pretty deep into the weeds, but is a good
case study in how traits and types compose in the Rust standard library.  Understanding this (deep,
but straightfoward) relationship will help to frame your thinking about Rust in the future. As a
reader's guide, I recommend you pay close attention to the different function signatures for
`into_iter` when implemented for `Vec<T>` and `&Vec<T>`.

### Cloning when Handling an `Option` or a `Result`

Another tempting place to clone is when dealing with `Option` and `Result`. These are specific cases
of any enumeration type that carries internal data, but as a new Rust programmer, you're likely to
encounter them quickly.

As before, let's contrive an example. Here, we've got a collection of `Strings` that should not
contain a particular `char`. Because we may not yet have some text, we will keep it inside
an `Option`.

```rust
pub struct LipogramCorpora {
    selections: Vec<(char, Option<String>)>,
}

impl LipogramCorpora {
    pub fn validate_all(&mut self) -> Result<(), char> {
        for selection in &self.selections {
            if selection.1.is_some() {
                if selection.1.unwrap().contains(selection.0) {
                    return Err(selection.0);
                }
            }
        }
        Ok(())
    }
}
```

Excellent! We've got a way to store example bodies of text, alongside the character they shouldn't
contain. And because C has trained us to check for null pointers, and `Option::None` is kinda-sorta
like a null pointer, we even check to make sure our text is there before we unwrap it.

Except, our old friend "cannot move out of borrowed content" shows up again:

```
error[E0507]: cannot move out of borrowed content
  --> src/lib.rs:10:20
   |
10 |                 if selection.1.unwrap().contains(selection.0) {
   |                    ^^^^^^^^^^^ cannot move out of borrowed content

error: aborting due to previous error
```

But you were so careful! No worries, you know what to do here. You'll just `clone`!

```rust
            if selection.1.is_some() {
                if selection.1.clone().unwrap().contains(selection.0) {
                    return Err(selection.0);
```

And all is well. Until I get your code review and remind you that we'll be running this method on
billions of texts that could well be in the tens of GiBs and copying that much data would murder our
performance. So how do we accomplish this with no `clone`? Let's consider two options, one more
idiomatic than the other.

First we consider what `unwrap` actually _does_:

```rust
pub fn unwrap(self) -> T;
```

This function _moves_ self, and returns an _owned_ copy of `T`. Remember, that means it _consumes_
`self` and gives you the actual data inside the `Option`. When you clone it, you're consuming the
clone, and getting another copy of that "actual data".

In Rust, we really like references, right? Happily, `Option` provides a method called
[`as_ref`](https://doc.rust-lang.org/std/option/enum.Option.html#method.as_ref), with the signature:

```rust
pub fn as_ref(&self) -> Option<&T>;
```

This _borrows_ self and returns a _reference to_ the data inside the `Option`. Huzzah! Now our code
can look like this:

```rust
            if selection.1.is_some() {
                if selection.1.as_ref().unwrap().contains(selection.0) {
                    return Err(selection.0);
```

This makes `unwrap` move the reference out of the `Option` that `as_ref` returns. That is, it consumes the _reference_.
And remember, references are cheap! And then we've got a reference to our `String` and all is well.

A more idiomatic way is to dispense with checking that the `Option` is not `None` and stop using
`unwrap` entirely. We do this using `if let`:

```rust
            if let Some(text) = selection.1 {
                if text.contains(selection.0) {
                    return Err(selection.0);
```

Much better.

## Now Go Forth and Clone

Hopefully showing you some counterexamples will help you think about _why_ you're doing what you do
in Rust, which will help reduce the amount of time you spend writing ill-formed code. If you have
any more examples that you've seen in the wild, please send them along. I'll add some more as I come
up with them too.
