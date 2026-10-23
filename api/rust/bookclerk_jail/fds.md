**bookclerk_jail > fds**

# Module: fds

## Contents

**Functions**

- [`close_inherited`](#close_inherited) - Close every descriptor above stdio except those the host wired deliberately.

---

## bookclerk_jail::fds::close_inherited

*Function*

Close every descriptor above stdio except those the host wired deliberately.

A sweep that could not be performed is not a sweep that can be assumed, so
this fails closed like the rest of the launcher.

```rust
fn close_inherited(preserve: &[i32]) -> Result<(), String>
```



