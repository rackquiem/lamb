# lamb

Lamb lets you inspect how a small OCaml expression is typed, lowered and represented at runtime. It prints models of Typedtree, Lambda, Clambda, Cmm and x86-64 assembly, evaluates expressions, and shows tagged values and closure environments.

There are two implementations: a JavaScript CLI with no runtime dependencies, and an OCaml CLI that also has an optimiser and a tagged bytecode machine. Both use their own parser and type inference. The printed compiler stages are educational sketches; the assembly output is not an executable binary.

![Lamb running in a terminal](assets/demo.gif)

## Run with Node.js

Requires Node.js 20 or newer.

```sh
git clone https://github.com/rackquiem/lamb.git
cd lamb
node bin/lamb sample.ml
```

Use `npm link` if you want `lamb` on your path. Otherwise, run `node bin/lamb` from the checkout.

```sh
node bin/lamb sample.ml --stage clambda
node bin/lamb examples/list.ml --run
node bin/lamb examples/option.ml --repr
printf 'let x = 20 in x + 22\n' | node bin/lamb --stage cmm
```

## Build the OCaml implementation

Requires OCaml 5.1 or newer and Dune 3.12 or newer.

```sh
dune build
dune exec lambnative -- sample.ml --stage clambda
dune exec lambnative -- examples/list.ml --run
```

If your tools are installed through opam, run these commands in your switch environment, or prefix them with `opam exec --`.

The OCaml CLI also exposes the alpha-renamed Lambda IR, reduction passes and bytecode execution:

```sh
dune exec lambnative -- sample.ml --ir
dune exec lambnative -- sample.ml --optimize
dune exec lambnative -- examples/list.ml --bytecode
dune exec lambnative -- examples/list.ml --machine
```

The optimiser folds constants, simplifies branches, removes unused pure bindings, inlines small expressions and performs beta reduction. The bytecode machine executes the optimised IR after a stack verification pass. `--run` evaluates the source syntax; `--machine` executes the bytecode.

![Lambda optimisation in a terminal](assets/optimize.gif)

## Inspect a program

Both CLIs accept a file or standard input. With no mode selected, they print all six stages.

| Option | Output |
| --- | --- |
| `--stage source` | Source expression |
| `--stage typedtree` | Typed expression nodes |
| `--stage lambda` | Functional intermediate form |
| `--stage clambda` | Closures and captured environments |
| `--stage cmm` | Tagged arithmetic and allocation |
| `--stage assembly` | An x86-64 instruction sketch |
| `--run` | Inferred type and evaluated result |
| `--analyze` | Structural metrics and diagnostics |
| `--repr` | Result value, tagged word and allocated blocks |
| `--tokens` | Token stream and source offsets |
| `--word 42` | Tagged integer encoding |
| `--list` | Available compiler stages |

Add `--json` for structured output from the shared inspection modes. The OCaml-only IR, optimisation and bytecode modes print text. The Node CLI supports `--no-color` and `NO_COLOR` to disable ANSI colour; the OCaml CLI prints plain text.

```sh
node bin/lamb sample.ml --stage lambda --json
node bin/lamb --word -42 --json
```

Integer tagging uses `(n << 1) | 1`, so `42` becomes `85`. The Node word inspector uses `BigInt` and displays the low 64 bits in hexadecimal and binary. The OCaml implementation uses `Int64`.

![Tagged runtime layout in a terminal](assets/layout.gif)

## Supported expressions

The parsers cover literals, `let` and `let rec`, anonymous functions, application, conditionals, tuples, lists, option constructors and pattern matching. Type inference supports polymorphic let bindings. The examples directory contains small programs for each of the collection and pattern forms.

```ocaml
let rec length values =
  match values with
  | [] -> 0
  | value :: rest -> 1 + length rest
in
length [3; 5; 8; 13; 21]
```

This evaluates to `5`. Lamb accepts an expression subset, so it cannot compile arbitrary OCaml projects. Compilation also evaluates the expression to obtain its runtime value, even when you request only a printed stage.

## Checks

```sh
npm run check
npm run program
dune runtest
```

The JavaScript checks cover syntax, inference, evaluation, runtime representation, analysis and the CLI. The OCaml suite also checks optimisation, bytecode execution and stack verification. CI runs JavaScript checks on Node.js 20, 22 and 24, and the native suite on OCaml 5.2.

The implementation lives in `src/` and `ocaml/`; the CLI entry points are `src/cli/index.js` and `ocaml/main.ml`.
