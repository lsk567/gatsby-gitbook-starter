
> :warning: **Important:** The Rust target is not functional yet. This is early WIP documentation to let you try it out if you're curious

In the Rust reactor target for Lingua Franca, reactions are written in Rust and the code generator generates a standalone Rust program that can be compiled and run on platforms supported by rustc. The program depends on a runtime library distributed as the crate [reactor_rt](https://github.com/icyphy/reactor-rust), and depends on the Rust standard library.

<!-- Note that C++ is not a safe language. There are many ways that a programmer can circumvent the semantics of Lingua Franca and introduce nondeterminism and illegal memory accesses. For example, it is easy for a programmer to mistakenly send a message that is a pointer to data on the stack. The destination reactors will very likely read invalid data. It is also easy to create memory leaks, where memory is allocated and never freed. Note, however, that the C++ reactor library is designed to prevent common errors and to encourage a safe modern C++ style. Here, we introduce the specifics of writing Reactor programs in C++ and present some guidelines for a style that will be safe. -->

## Setup

In order to compile the generated Rust source code, you need a recent version of [Cargo](https://doc.rust-lang.org/cargo/), the Rust package manager. See [How to Install Rust and Cargo](https://doc.rust-lang.org/cargo/getting-started/installation.html) if you don't have them on your system.

You can use a development version of the runtime library by setting the LFC option `--external-runtime-path` to the root directory of the runtime library crate sources. If this variable is mentioned, LFC will ask Cargo to fetch the runtime library from there.

For now the runtime crate is *not available on crates.io or publicly on Github*. Until then, if you don't have SSH access to the private repo, the `--external-runtime-path` option is the only way to use the Rust target.


## A Minimal Example

A "hello world" reactor for the Rust target looks like this:
```rust
target Rust;

main reactor Minimal {
    reaction(startup) {=
        println!("Hello, reactors!");
    =}
}
```

The `startup` action is a special [action](https://github.com/icyphy/lingua-franca/wiki/Language-Specification#Action-Declaration) that triggers at the start of the program execution causing the [reaction](https://github.com/icyphy/lingua-franca/wiki/Language-Specification#Reaction-Declaration) to execute. This program can be found in a file called [Minimal.lf](https://github.com/icyphy/lingua-franca/blob/master/test/Rust/src/Minimal.lf) in the [test directory](https://github.com/icyphy/lingua-franca/tree/master/test/Rust), where you can also find quite a few more interesting examples. If you compile this using the [`lfc` command-line compiler](downloading-and-building#Command-Line-Tools) or the [Eclipse-based IDE](downloading-and-building#Download-the-Integrated-Development-Environment), then generated source files will be put into a subdirectory called `src-gen/Minimal`. In addition, an executable binary will be compiled using Cargo. The resulting executable will be called `minimal` (note and be put in a subdirectory called `bin`. If you are in the Rust test directory, you can execute it in a shell as follows:
```
 bin/minimal
```

The resulting output should look something like this:
```
[INFO]  Starting the execution
Hello World!
[INFO]  Terminating the execution
```

## The Rust Target Specification

To have Lingua Franca generate Rust code, start your .lf file with the following target specification:
```rust
target Rust;
```

### Target properties summary

- `single-file-project: [true|false]`  - enables [single-file project layout](#single-file-layout)
- `timeout: <time value>` - timeout for the execution. The program will shutdown the specified amount of (logical) time after the start of its execution.
- `keepalive: [true|false]` - WIP, this doesn't work anymore

### The executable

The executable name is the name of the main reactor *transformed to snake_case*: `main reactor RustProgram` will generate `rust_program`.

Currently, any CLI arguments are ignored. Parsing arguments is future work.

#### Logging levels

The executable reacts to the environment variable `RUST_LOG`, which sets the logging level of the application. Possible values are
`off`, `error`, `warn`, `info`, `debug`, `trace`

Error logs are on by default. Enabling a level enables all greater levels (ie, `RUST_LOG=info` also enables `warn` and `error`, but not `trace` or `debug`).

### File layout

The Rust code generator *generates a Cargo project* with a classical layout: 
```
├── Cargo.lock
├── Cargo.toml
├── src
│   ├── main.rs
│   └── reactors
│       ├── mod.rs
|       ├── ...
|
└── target
    ├── ...
```

The module structure is as follows:
- the crate has a module `reactors`
- each LF reactor has its own submodule of `reactors`. For instance, `Minimal.lf` will generate `minimal.rs`. The name is transformed to snake_case.

This means that to refer to the contents of another reactor module, e.g. that of `Other.lf`, you have to write `super::other::Foo`. This is relevant to access `preamble` items.

#### Single-file layout

The Rust target supports an alternative file layout, where all reactors are generated into the `main.rs` file, making the project fit in a single file (excluding `Cargo.toml`). *The module structure is unchanged:* the file still contains a `mod reactors { ... }` within which each reactor has its `mod foo { ... }`. You can thus change the layout without having to update any LF code.

Set the target property `single-file-project: true` to use this layout.

Note: this alternative layout is provided for the purposes of making self-contained benchmark files. Generating actual runnable benchmarks from an LF file may be explored in the future.

### Generation scheme

Each reactor generates its own `struct` which contains state variables. For instance,

<table>
<thead>
<tr>
<th>LF</th>
<th>Generated Rust</th>
</tr>
</thead>
<tbody>
<tr>
<td>

```rust
reactor SomeReactor { 
  state field: u32(0)
}
``` 

</td>

<td>

```rust
struct SomeReactor { 
  field: u32
}
``` 

</td>

</tr>
</tbody>
</table>


Another corresponding struct is generated for constructor parameters.


<table>
<thead>
<tr>
<th>LF</th>
<th>Generated Rust</th>
</tr>
</thead>
<tbody>
<tr>
<td>

```rust
reactor SomeReactor(
  param1: i32(0),
  param2: {= &'static str =}) {

}
``` 

</td>

<td>

```rust
// this reactor has no state variables
struct SomeReactor { }
// the parameters are on this struct
struct SomeReactorParams { 
  param1: i32,
  param2: &'static str 
}
```

</td>

</tr>
</tbody>
</table>


In the following we refer to those as the *state struct* and the *param struct*, respectively.

#### Reactions

> :warning: Part of the following is a statement of intention and is not yet implemented, or uses different names.

Reactions are each generated in a separate method of the reactor struct whose name is unspecified and may be mangled to prevent explicit calling. The parameters of that method are
- `&mut self`: the state struct described above,
- `ctx: &mut ReactionCtx`: the context object for the reaction execution,
- <code>params: & <i>&lt;param struct&gt;</i></code>: a reference to the param struct,
- for each declared trigger on a port of type `T`: `name: &ValueSource<T>`,
- for each declared effect on a port of type `T`: `name: &mut ValueSink<T>`,
- for each declared trigger on an action of type `T`: `name: &Action<T>`,
- for each declared effect on an action of type `T`: `name: &mut Action<T>`.

Undeclared dependencies, and dependencies on timers and `startup` or `shutdown`, do not generate a parameter.

The `ReactionCtx` object is a mediator to manipulate all those dependency objects. It has methods to set ports, schedule actions, retrieve the current logical time, etc.

For instance:
```rust
reactor Source {
    output out: i32;
    reaction(startup) -> out {=
        ctx.set(out, 76600)
    =}
}
```
In this example, the context object `ctx` is used to set a port to a value. The port is in scope as `out`.


> :warning: TODO when the runtime crate is public link to the docs, they should be the most exhaustive documentation.

#### Actions

Within a reaction, actions may be scheduled using the [`schedule`](TODO_link) function:
```rust
// schedule without additional delay
ctx.schedule(act, Asap);
// schedule with an additional delay
ctx.schedule(act, After(Duration.of_millis(20)));
```

Actions may carry values if they mention a data type, for instance:
```rust
logical action act: u32;
```

Within a reaction, you can schedule that action with a value like so
```rust
ctx.schedule_with_v(act, Asap, 30);
```
you can get the value from another reaction like so:
```rust
if let Some(value) = ctx.get_action(act) {
  // a value is present at this tag
} else {
  // value was not specified
}
```

If an action does not mention a data type, the type is defaulted to `()`. 

#### Time

> :warning: todo




