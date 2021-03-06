# typify [![Build Status](https://secure.travis-ci.org/phadej/typify.png?branch=master)](http://travis-ci.org/phadej/typify)

Runtime type-checking.

## Getting Started
Install the module with: `npm install typify`

## Synopsis

```javascript
var typify = require("typify").typify;

/*
 * `sum` function takes either two numbers or two strings as a parameters,
 * and returns a number or a string respectively.
 */
var add = typify("sum :: a : number|string => a -> a -> a", function (a, b) {
    return a + b;
});

/*
 * `toArray` function takes either array of numbers or single number,
 * and returns a array of numbers.
 *
 * We could write more general, polymorphic function with type signature
 * `toArray :: a : *, (array a)|a -> array a`, where `*` means _any type_.
 *
 * Unfortunately any type `*` is seriosly any. 
 * Types as *typify* understandes them, are more like Java's interfaces or Haskell's typeclasses.
 * Of course, we can iterate throught them all, but we cannot deduce most principal type (because it doesn't exist).
 * So eg. function signature `id :: a : *, a -> a` behaves similarly as `id :: * -> *`, which isn't strict enough.
 */
var toNumberArray = typify("toNumberArray :: (array number)|number -> array number", function (a) {
    return Array.isArray(a) ? a : [a];
});

/*
 * `myParseInt` takes string and an optional number (radix) and returns number.
 */
var myParseInt = typify("myParseInt :: string -> number? -> number", function (n, radix) {
    return parseInt(n, radix || 10)
});

/*
 * `foo` takes at least one number parameter and returns a number.
 */
var foo = typify("foo :: number -> number.. -> number", function (a) {
    return a + arguments.length;
});
```

## Documentation

### Checkable type

*Checkable* means, that given an object you can check whether an object is or isn't of the particular type.
For example `number` is checkable type, given any object you can tell if it's a number.

```javascript
typify.check('number', 1); // => true
typify.check('number', 'foobar'); // => false
```

There are few predefined checkable types:
- `number`
- `string`
- `boolean`
- `null`
- `date`
- `regexp`
- `function`
- `array` _a_
- `map` _a_

#### Formal syntax of checkable type declaration:

- *checkable type* σ ::= σ_alt
    - σ_alt ::= σ_poly (`|` σ_poly)*
    - σ_poly ::= *typename* σ_opt+ | σ_opt
    - σ_opt = σ_term | σ_term `?`
    - σ_term ::= `*` | α | *typename* | `(` σ_alt `)`
- *type variable* α ::= *identifier*
- *identifier*, *typename* ::= /[a-zA-Z]+/

### Function type

Function types are difficult to check. Given a function object, only you can tell, it's a function object.
To be more precise, you can decorate your function with *function type* signature to verify parameters' and result's types, but the check will occur only when function is executed ie. run-time.

```javascript
var add = typify("add :: number -> number -> number", function (a, b) {
    return a + b;
});

console.log(add(1, 2)); // ok
console.log(add("foo", "bar")); // throws TypeError
```

### New types

New types can be added with `typify.type` method:

```javascript
typify.type("char", function(n) {
    return typeof n === "string" && n.length === 1;
});
```

Also you can define *record* types with `typify.record`:

```javascript
typify.record("person", {
    name: "string",
    age: "number",
});
```

#### Formal syntax of function type declaration:

- *function type* λ ::= ν μ | ν Γ (σ `->`)* ρ σ
- *action* μ ::= `->` τ
- *context* Γ ::= α `:` Σ (`,` α `:` Σ)* `=>` | ε
- *typeset* Σ ::= σ_opt (`|` σ_opt)*
- *rest parameters* ρ ::= σ `...` `->` | `...` `->` | ε
- *function name* ν ::= *identifier* `::` | ε

## Contributing
In lieu of a formal styleguide, take care to maintain the existing coding style. Add unit tests for any new or changed functionality. Lint and test your code using [Grunt](http://gruntjs.com/).

## Release History

- 0.1.0 Initial release

## License
Copyright (c) 2013 Oleg Grenrus. Licensed under the MIT license.
