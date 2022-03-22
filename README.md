# reborrow
Emulate reborrowing for user types.

Given a `&'a` [mutable] reference of a `&'b` view over some owned object,
reborrowing it means getting an active `&'a` view over the owned object,
which renders the original reference inactive until it's dropped, at which point
the original reference becomes active again.

# Examples:
This fails to compile since we can't use a non-`Copy` value after it's moved.
```rust
fn takes_mut_option(o: Option<&mut i32>) {}
                                                                                                
let mut x = 0;
let o = Some(&mut x);
takes_mut_option(o); // `o` is moved here,
takes_mut_option(o); // so it can't be used here.
```
                                                                                                
This can be worked around by unwrapping the option, reborrowing it, and then wrapping it again.
```rust
fn takes_mut_option(o: Option<&mut i32>) {}
                                                                                                
let mut x = 0;
let mut o = Some(&mut x);
takes_mut_option(o.as_mut().map(|r| &mut **r)); // "Reborrowing" the `Option`
takes_mut_option(o.as_mut().map(|r| &mut **r)); // allows us to use it later on.
drop(o); // can still be used here
```
                                                                                                
Using this crate, this can be shortened to
```rust
use reborrow::ReborrowMut;
                                                                                                
fn takes_mut_option(o: Option<&mut i32>) {}
                                                                                                
let mut x = 0;
let mut o = Some(&mut x);
takes_mut_option(o.rb_mut()); // "Reborrowing" the `Option`
takes_mut_option(o.rb_mut()); // allows us to use it later on.
drop(o); // can still be used here
```