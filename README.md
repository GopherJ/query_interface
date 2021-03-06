# query_interface
Dynamically query a type-erased object for any trait implementation

[Documentation](https://docs.rs/query_interface/)

Example:
```rust
#[macro_use]
extern crate query_interface;

use query_interface::{Object, ObjectClone};
use std::fmt::Debug;

#[derive(Copy, Clone, PartialEq, Eq, PartialOrd, Ord, Debug)]
struct Foo;

interfaces!(Foo: ObjectClone, Debug, Bar);

trait Bar {
    fn do_something(&self);
}
impl Bar for Foo {
    fn do_something(&self) {
        println!("I'm a Foo!");
    }
}

fn main() {
    let obj = Box::new(Foo) as Box<Object>;
    let obj2 = obj.clone();
    println!("{:?}", obj2);
   
    obj2.query_ref::<Bar>().unwrap().do_something();  // Prints: "I'm a Foo!"
}
```

In short, this allows you to pass around `Object`s and still have access to any of the (object-safe) traits
implemented for the underlying type. The library also provides some useful object-safe equivalents to standard
traits, including `ObjectClone`, `ObjectPartialEq`, `ObjectPartialOrd`, `ObjectHash`.

To improve usability, the non-object-safe versions of the traits are implemented directly on the `Object` trait
object, allowing you to easily clone `Object`s and store them in collections.

You can have your own `Object`-like traits to enforce some additional static requirements by using the `mopo!`
macro:
```rust
trait CustomObject : Object {
    ...
}
mopo!(CustomObject);

struct Foo;
interfaces!(Foo: CustomObject);

impl CustomObject for Foo {
    ...
}
```

Now you can use `CustomObject` in all the ways you could use `Object`.

With the "dynamic" feature, you can *at runtime* register additional traits to be queryable on a type. This
allows you to bypass the normal coherence rules:

```rust
trait Custom {}
impl Custom for String {}

fn main() {
    dynamic_interfaces! {
        String: Custom;
    }
}
```
