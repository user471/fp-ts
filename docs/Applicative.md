---
id: Applicative
title: Module Applicative
---

[← Index](.)

[Source](https://github.com/gcanti/fp-ts/blob/master/src/Applicative.ts)

# Applicative

**Signature** (type class) [Source](https://github.com/gcanti/fp-ts/blob/master/src/Applicative.ts#L37-L39)

```ts
export interface Applicative<F> extends Apply<F> {
  readonly of: <A>(a: A) => HKT<F, A>
}
```

The `Applicative` type class extends the [Apply](./Apply.md) type class with a `of` function, which can be used to create values
of type `f a` from values of type `a`.

Where [Apply](./Apply.md) provides the ability to lift functions of two or more arguments to functions whose arguments are
wrapped using `f`, and [Functor](./Functor.md) provides the ability to lift functions of one argument, `pure` can be seen as the
function which lifts functions of _zero_ arguments. That is, `Applicative` functors support a lifting operation for
any number of function arguments.

Instances must satisfy the following laws in addition to the [Apply](./Apply.md) laws:

1. Identity: `A.ap(A.of(a => a), fa) = fa`
2. Homomorphism: `A.ap(A.of(ab), A.of(a)) = A.of(ab(a))`
3. Interchange: A.ap(fab, A.of(a)) = A.ap(A.of(ab => ab(a)), fab)

Note. [Functor](./Functor.md)'s `map` can be derived: `A.map(x, f) = A.ap(A.of(f), x)`

Added in v1.0.0

## getApplicativeComposition

Like `Functor`, `Applicative`s compose. If `F` and `G` have `Applicative` instances, then so does `F<G<_>>`

**Signature** (function) [Source](https://github.com/gcanti/fp-ts/blob/master/src/Applicative.ts#L228-L235)

```ts
export function getApplicativeComposition<F, G>(F: Applicative<F>, G: Applicative<G>): ApplicativeComposition<F, G>  { ... }
```

**Example**

```ts
import { getApplicativeComposition } from 'fp-ts/lib/Applicative'
import { option, Option, some } from 'fp-ts/lib/Option'
import { task, Task } from 'fp-ts/lib/Task'

const x: Task<Option<number>> = task.of(some(1))
const y: Task<Option<number>> = task.of(some(2))

const A = getApplicativeComposition(task, option)

const sum = (a: number) => (b: number): number => a + b
A.ap(A.map(x, sum), y)
  .run()
  .then(result => assert.deepStrictEqual(result, some(3)))
```

Added in v1.0.0

## getMonoid

If `F` is a `Applicative` and `M` is a `Monoid` over `A` then `HKT<F, A>` is a `Monoid` over `A` as well.
Adapted from http://hackage.haskell.org/package/monoids-0.2.0.2/docs/Data-Monoid-Applicative.html

**Signature** (function) [Source](https://github.com/gcanti/fp-ts/blob/master/src/Applicative.ts#L266-L272)

```ts
export function getMonoid<F, A>(F: Applicative<F>, M: Monoid<A>): () => Monoid<HKT<F, A>>  { ... }
```

**Example**

```ts
import { getMonoid } from 'fp-ts/lib/Applicative'
import { option, some, none } from 'fp-ts/lib/Option'
import { monoidSum } from 'fp-ts/lib/Monoid'

const M = getMonoid(option, monoidSum)()
assert.deepStrictEqual(M.concat(none, none), none)
assert.deepStrictEqual(M.concat(some(1), none), none)
assert.deepStrictEqual(M.concat(none, some(2)), none)
assert.deepStrictEqual(M.concat(some(1), some(2)), some(3))
```

Added in v1.4.0

## when

Perform a applicative action when a condition is true

**Signature** (function) [Source](https://github.com/gcanti/fp-ts/blob/master/src/Applicative.ts#L163-L165)

```ts
export function when<F>(F: Applicative<F>): (condition: boolean, fu: HKT<F, void>) => HKT<F, void>  { ... }
```

**Example**

```ts
import { IO, io } from 'fp-ts/lib/IO'
import { when } from 'fp-ts/lib/Applicative'

const log: Array<string> = []
const action = new IO(() => {
  log.push('action called')
})
when(io)(false, action).run()
assert.deepStrictEqual(log, [])
when(io)(true, action).run()
assert.deepStrictEqual(log, ['action called'])
```

Added in v1.0.0
