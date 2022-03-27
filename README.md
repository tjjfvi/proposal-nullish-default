
# Nullish-Coalescing Default Assignment

A proposal for using `??=` in default assignment in function arguments and destructuring.

## Status

Champion(s): none yet

Author(s): [@tjjfvi](https://github.com/tjjfvi)

Stage: -1

## Motivation

Many recent ECMAScript features treat `null` and `undefined` equivalently (notably nullish coalescing and optional chaining), and today one can largely adopt a style of treating them interchangably via a combination of `== null`, `??`, and `?.`. However, there is one major place today where one cannot: default values. The `pattern = defaultValue` syntax in parameters and destructuring only replaces `undefined` values with the default – `null`s will pass through unscathed.

This proposal adds new syntax to remedy this – `pattern ??= defaultValue` – which is analogous to `pattern = defaultValue`, except that both `undefined` and `null` will be replaced by `defaultValue`.

This is perhaps demonstrated best with a simple example:

```ts
function log(message ??= "[empty]"){
  console.log(message)
}

log()          // logs "[empty]"
log(null)      // logs "[empty]"
log(undefined) // logs "[empty]"

log("abc")     // logs "abc"
```


> Note: Though one might prefer to only use `undefined`, some `null`s will inevitably occur, be it from standard library functions (like `RegExp.prototype.match`), external APIs (like GraphQL queries), or user code (when one is writing libraries). Thus, this approach of treating the two interchangably is still practical, even if one never writes `null` directly.

## Use cases

### Protecting against `null` in libraries

```ts
/* some library */
export function calculateAwesomeness(thing, multiplierA = 4, multiplierB = 2){
  // ...
}
```
```ts
/* someone using the library mistakenly passes null */

import { calculateAwesomeness } from "some-library"

// oops, this attempt to use the default will probably return NaN
calculateAwesomeness(thisProposal, null, 2)
```
```ts
/* new version of the library guards against this mistake */
export function calculateAwesomeness(thing, _multiplierA, _multiplierB){
  const multiplierA = _multiplierA ?? 4
  const multiplierB = _multiplierB ?? 2
  // ...
}

/* instead, with this proposal */
export function calculateAwesomeness(thing, multiplierA ??= 4, _multiplierB ??= 2){
  // ...
}
```

### Destructuring RegExp matches

```ts
function* search(strings){
  for(const [index, string] of strings.entries()){
    /* some complex logic */
    yield { string, match: regexp.exec(string) }
  }
}
```
```ts
// Currently:
for(const result of search(strings)){
  const { match } = result
  const [matchText] = match ?? [] // match might be null
}
```
```ts
// With this proposal:
for(const result of search(strings)){
  const { match: [matchText] ??= [] } = result
  // ...
}
```
```ts
// Or, with this proposal:
for(const { match: [matchText] ??= [] } of search(strings)){
  // ...
}
```

### Destructuring GraphQL queries

Since GraphQL queries return `null` for optional properties, destructuring them would be greatly more ergonomic with `??=`:

```ts
// Currently:
const {
  title,
  description: _description,
  author: _author,
} = await getBook()
const description = _description ?? "[no description in database]"
const {
  id: authorId,
  name: author,
} = _author ?? {}
```
```ts
// With this proposal:
const {
  title,
  description ??= "[no description in database]",
  author: {
    id: authorId,
    name: author,
  } ??= {},
} = await getBook()
```

## Implementations

[coming soon]
