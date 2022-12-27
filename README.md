# Bitcoin Miniscript

This project is a JavaScript implementation of Bitcoin Miniscript, a high-level language for describing Bitcoin spending conditions.

It includes a transpilation of [Peter Wuille's C++ code](https://github.com/sipa/miniscript) for compiling spending policies into Miniscript and Bitcoin scripts, as well as a novel Miniscript Satisfier for generating explicit witness scripts that are decoupled from the tx signer.

## Features

- Compile Policies into Miniscript and Bitcoin scripts.
- A Miniscript Satisfier that discards malleable solutions.
- The Miniscript Satisfier is able to generate explicit witness scripts from Miniscripts using variables, such as `pk(key)`.

  For example, Miniscript `and_v(v:pk(key),after(10))` can be satisfied with `[{ witness: '<sig(key)>', nLockTime: 10 }]`.
- The ability to generate different satisfactions depending on the presence of `unknowns`.

  For example, Miniscript `c:and_v(or_c(pk(key1),v:ripemd160(H)),pk_k(key2))` can be satisfied with: `[{ witness: '<sig(key2)> <ripemd160_preimage(H)> 0' }]`.

  However, if `unknowns: ['<ripemd160_preimage(H)>']` is set, then the Miniscript can be satisfied with: `[{ witness: '<sig(key2)> <sig(key1)>' }]` because this solution can no longer be considered malleable, given then assumption that an attacker does not have access to the preimage.
- Thoroughly tested.

## Installation

To install the package, use npm:

```
npm install @bitcoinerlab/miniscript
```

## Usage

### Compiling Policies into Miniscript and Bitcoin script

To compile a Policy into a Miniscript and Bitcoin ASM, you can use the `compilePolicy` function:

```javascript
const { compilePolicy } = require('@bitcoinerlab/miniscript');

const policy = 'or(and(pk(A),older(8640)),pk(B))';

const { miniscript, asm, issane } = compilePolicy(policy);
```
`issane` is a boolean that indicates whether the Miniscript is valid and follows the consensus and standardness rules for Bitcoin scripts. A sane Miniscript should have non-malleable solutions, not mix different timelock units on a single branch of the script, and not contain duplicate keys. In other words, it should be a well-formed and standards-compliant script that can be safely used in transactions.

### Compiling Miniscript into Bitcoin script

To compile a Miniscript into Bitcoin ASM you can use the `compileMiniscript` function:

```javascript
const { compileMiniscript } = require('@bitcoinerlab/miniscript');

const miniscript = 'and_v(v:pk(key),or_b(l:after(100),al:after(200)))';

const { asm, issane } = compileMiniscript(miniscript);
```

### Generating explicit witness scripts

To generate an explicit witness script from a Miniscript, you can use the `satisfier` function:

```javascript
const { satisfier } = require('@bitcoinerlab/miniscript');

const miniscript =
  'c:or_i(andor(c:pk_h(key1),pk_h(key2),pk_h(key3)),pk_k(key4))';

const satisfactions = satisfier(miniscript);
```
`satisfier` makes sure that output `satisfactions` are non-malleable and that the `miniscript` is sane.

You can also set `unknowns`:

```javascript
const { satisfier } = require('@bitcoinerlab/miniscript');

const miniscript =
  'c:or_i(andor(c:pk_h(key1),pk_h(key2),pk_h(key3)),pk_k(key4))';
const unknowns = ['<sig(key1)>', '<sig(key2)>'];

const satisfactions = satisfier(miniscript, unknowns);
```

## Authors and Contributors

The project was initially developed and is currently maintained by [Jose-Luis Landabaso](https://github.com/landabaso). Contributions and help from other developers are welcome.

Here are some resources to help you get started with contributing:

### Building from source

To download the source code and build the project, follow these steps:

1. Clone the repository:

```
git clone https://github.com/bitcoinerlab/miniscript.git
```

2. Install the dependencies:

```
npm install
```

3. Make sure you have the [`em++` compiler](https://emscripten.org/) in your PATH.

4. Run the Makefile:

```
make
```

This will download and build Wuille's sources and generate the necessary Javascript files.

5. Build the project:

```
npm run build
```

This will build the project and generate the necessary files in the `dist` directory.

### Documentation

To generate the programmers's documentation, which describes the library's programming interface, use the following command:

```
npm run docs
```

This will generate the documentation in the `docs` directory.

### Testing

Before committing any code, make sure it passes all tests by running:

```
npm run tests
```

## License

This project is licensed under the MIT License.
