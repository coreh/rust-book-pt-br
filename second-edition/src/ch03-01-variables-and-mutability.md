## Variáveis e Mutabilidade

Como mencionamos no capítulo 2, por padrão variáveis em Rust são *imutáveis*.
Este é um de muitos “empurrõezinhos” que te encorajam a escrever seu código de
forma a se beneficiar da segurança e concorrência oferecidas pela linguagem.
Apesar disso, você ainda tem a opção de tornar as suas variáveis mutáveis.
Vamos explorar como e por que Rust lhe encoraja a favorecer imutabilidade e sob
quais circunstâncias você pode querer abrir mão dessa garantia.

Quando uma variável é imutável, assim que um valor passa a estar vinculado ao seu
nome é impossível alterá-lo. Para ilustrar, vamos gerar um novo projeto chamado
*variaveis* [sic] no seu diretório *projetos*, usando o comando `cargo new --bin variaveis`.

Em seguida, no diretório *variaveis* que acabou de ser criado, abra *src/main.rs* e
substitua seu código com o seguinte:

<span class="filename">Nome de Arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let x = 5;
    println!("O valor de x é: {}", x);
    x = 6;
    println!("O valor de x é: {}", x);
}
```

Salve e execute o programa usando `cargo run`. Você deve receber uma mensagem
de erro, conforme destacado nessa saída (em inglês):

```text
error[E0384]: re-assignment of immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         - first assignment to `x`
3 |     println!("The value of x is: {}", x);
4 |     x = 6;
  |     ^^^^^ re-assignment of immutable variable
```

*erro[E0384]: Re-atribuição de variável imutável*

Este exemplo revela como o compilador lhe ajuda a encontrar erros em seus programas.
Apesar dos erros do compilador serem frustrantes, eles significam apenas que seu programa
não está ainda fazendo com segurança o que você quer que ele faça; eles *não* significam que você
não é um bom programador! Rustáceos experientes ainda encontram erros do compilador. O
erro indica que a causa do problema é a *re-atribuição de variável imutável*,
já que tentamos atribuir um segundo valor a variável imutável `x`.

It’s important that we get compile-time errors when we attempt to change a
value that we previously designated as immutable because this very situation
can lead to bugs. If one part of our code operates on the assumption that a
value will never change and another part of our code changes that value, it’s
possible that the first part of the code won’t do what it was designed to do.
This cause of bugs can be difficult to track down after the fact, especially
when the second piece of code changes the value only *sometimes*.

In Rust the compiler guarantees that when we state that a value won’t change,
it really won’t change. That means that when you’re reading and writing code,
you don’t have to keep track of how and where a value might change, which can
make code easier to reason about.

But mutability can be very useful. Variables are immutable only by default; we
can make them mutable by adding `mut` in front of the variable name. In
addition to allowing this value to change, it conveys intent to future readers
of the code by indicating that other parts of the code will be changing this
variable value.

For example, change *src/main.rs* to the following:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

When we run this program, we get the following:

```text
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
     Running `target/debug/variables`
The value of x is: 5
The value of x is: 6
```

Using `mut`, we’re allowed to change the value that `x` binds to from `5` to
`6`. In some cases, you’ll want to make a variable mutable because it makes the
code more convenient to write than an implementation that only uses immutable
variables.

There are multiple trade-offs to consider, in addition to the prevention of
bugs. For example, in cases where you’re using large data structures, mutating
an instance in place may be faster than copying and returning newly allocated
instances. With smaller data structures, creating new instances and writing in
a more functional programming style may be easier to reason about, so the lower
performance might be a worthwhile penalty for gaining that clarity.

### Differences Between Variables and Constants

Being unable to change the value of a variable might have reminded you of
another programming concept that most other languages have: *constants*. Like
immutable variables, constants are also values  that are bound to a name and
are not allowed to change, but there are a few differences between constants
and variables.

First, we aren’t allowed to use `mut` with constants: constants aren’t only
immutable by default, they’re always immutable.

We declare constants using the `const` keyword instead of the `let` keyword,
and the type of the value *must* be annotated. We’re about to cover types and
type annotations in the next section, “Data Types,” so don’t worry about the
details right now, just know that we must always annotate the type.

Constants can be declared in any scope, including the global scope, which makes
them useful for values that many parts of code need to know about.

The last difference is that constants may only be set to a constant expression,
not the result of a function call or any other value that could only be
computed at runtime.

Here’s an example of a constant declaration where the constant’s name is
`MAX_POINTS` and its value is set to 100,000. (Rust constant naming convention
is to use all upper case with underscores between words):

```rust
const MAX_POINTS: u32 = 100_000;
```

Constants are valid for the entire time a program runs, within the scope they
were declared in, making them a useful choice for values in your application
domain that multiple parts of the program might need to know about, such as the
maximum number of points any player of a game is allowed to earn or the speed
of light.

Naming hardcoded values used throughout your program as constants is useful in
conveying the meaning of that value to future maintainers of the code. It also
helps to have only one place in your code you would need to change if the
hardcoded value needed to be updated in the future.

### Shadowing

As we saw in the guessing game tutorial in Chapter 2, we can declare a new
variable with the same name as a previous variable, and the new variable
*shadows* the previous variable. Rustaceans say that the first variable is
*shadowed* by the second, which means that the second variable’s value is what
we’ll see when we use the variable. We can shadow a variable by using the same
variable’s name and repeating the use of the `let` keyword as follows:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    let x = x * 2;

    println!("The value of x is: {}", x);
}
```

This program first binds `x` to a value of `5`. Then it shadows `x` by
repeating `let x =`, taking the original value and adding `1` so the value of
`x` is then `6`. The third `let` statement also shadows `x`, taking the
previous value and multiplying it by `2` to give `x` a final value of `12`.
When you run this program, it will output the following:

```text
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
     Running `target/debug/variables`
The value of x is: 12
```

This is different than marking a variable as `mut`, because unless we use the
`let` keyword again, we’ll get a compile-time error if we accidentally try to
reassign to this variable. We can perform a few transformations on a value but
have the variable be immutable after those transformations have been completed.

The other difference between `mut` and shadowing is that because we’re
effectively creating a new variable when we use the `let` keyword again, we can
change the type of the value, but reuse the same name. For example, say our
program asks a user to show how many spaces they want between some text by
inputting space characters, but we really want to store that input as a number:

```rust
let spaces = "   ";
let spaces = spaces.len();
```

This construct is allowed because the first `spaces` variable is a string type,
and the second `spaces` variable, which is a brand-new variable that happens to
have the same name as the first one, is a number type. Shadowing thus spares us
from having to come up with different names, like `spaces_str` and
`spaces_num`; instead, we can reuse the simpler `spaces` name. However, if we
try to use `mut` for this, as shown here:

```rust,ignore
let mut spaces = "   ";
spaces = spaces.len();
```

we’ll get a compile-time error because we’re not allowed to mutate a variable’s
type:

```text
error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected &str, found usize
  |
  = note: expected type `&str`
             found type `usize`
```

Now that we’ve explored how variables work, let’s look at more data types they
can have.