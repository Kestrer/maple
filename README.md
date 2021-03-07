# maple

A VDOM-less web library with fine-grained reactivity.

## Getting started

The recommended build tool is [Trunk](https://trunkrs.dev/).
Start by adding `maple-core` to your `Cargo.toml`:

```toml
maple-core = "0.1.0"
```

Add the following to your `src/main.rs` file:

```rust
use maple_core::prelude::*;

fn main() {
    let root = template! {
        p {
            # "Hello World!"
        }
    };

    render(root);
}
```

That's it! There's your hello world program using `maple`. To run the app, simply run `trunk serve --open` and see the result in your web browser.

## The `template!` macro

`maple` uses the `template!` macro as an ergonomic way to create complex user interfaces.

```rust
// You can create nested elements.
template! {
    div {
        p {
            span {
                # "Hello "
            }
            strong {
                # "World!"
            }
        }
    }
};

// Attributes (including classes and ids) can also be specified.
template! {
    p(class="my-class", id="my-paragraph")
};

template! {
    button(disabled="true") {
        # "My button"
    }
}

// Events are attached using the `on:*` directive.
template! {
    button(on:click=|| { /* do something */ }) {
        # "Click me"
    }
}
```

## Reactivity

Instead of relying on a Virtual DOM (VDOM), `maple` uses fine-grained reactivity to keep the DOM and state in sync.
In fact, the reactivity part of `maple` can be used on its own without the DOM rendering part.

Reactivity is based on reactive primitives. Here is an example:

```rust
use maple_core::prelude::*;
let (state, set_state) = create_signal(0); // create an atom with an initial value of 0
```

If you are familiar with [React](https://reactjs.org/) hooks, this will immediately seem familiar to you.

Now, to access the state, we call the first element in the tuple (`state`) like this:

```rust
println!("The state is: {}", state()); // prints "The state is: 0"
```

To update the state, we call the second element in the tuple (`set_state`):

```rust
set_state(1);
println!("The state is: {}", state()); // should now print "The state is: 0"
```

Why would this be useful? It's useful because it provides a way to easily be notified of any state changes.
For example, say we wanted to print out every state change. This can easily be accomplished like so:

```rust
let (state, set_state) = create_signal(0);
create_effect(move || {
    println!("The state changed. New value: {}", state());
});  // prints "The state changed. New value: 0" (note that the effect is always executed at least 1 regardless of state changes)

set_state(1); // prints "The state changed. New value: 1"
set_state(2); // prints "The state changed. New value: 2"
set_state(3); // prints "The state changed. New value: 3"
```

How does the `create_effect(...)` function know to execute the closure every time the state changes?
That's why `state` is a function rather than a simple variable.
Calling `create_effect` creates a new _"reactivity scope"_ and calling `state()` inside this scope adds itself as a _dependency_.
Now, when `set_state` is called, it automatically calls all its _dependents_, in this case, the closure passed to `create_effect`.

Now is when things get interesting. We can easily create a derived state using `create_memo(...)`:

```rust
let (state, set_state) = create_signal(0);
let double = create_memo(move || *state() * 2);

assert_eq!(*double(), 0);

set_state(1);
assert_eq!(*double(), 2);
```

`create_memo(...)` automatically recomputes the derived value when any of its dependencies change.

### Using reactivity with DOM updates

Reactivity is automatically built-in into the `template!` macro. Say we have the following code:

```rust
use maple_core::prelude::*;

let (state, set_state) = create_signal(0);

let root = template! {
    p {
        # state()
    }
}
```

This will expand to something approximately like:

```rust
use maple_core::prelude::*;
use maple_core::internal;

let (state, set_state) = create_signal(0);

let root = {
    let element = internal::element(p);
    let text = internal::text(String::new() /* placeholder */);
    create_effect(move || {
        // update text when state changes
        text.set_text_content(Some(&state()));
    });

    internal::append(&element, &text);

    element
}
```

If we call `set_state` somewhere else in our code, the text content will automatically be updated!

## Contributing

Issue reports and PRs are welcome!