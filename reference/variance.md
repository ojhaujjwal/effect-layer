# Covariance & Contravariance in TypeScript

## Definitions

| Direction | Keyword | If `A extends B`... | Example |
|-----------|---------|---------------------|---------|
| **Covariant** (`out`) | — | `Box<A>` extends `Box<B>` | `string[]` is a `(string \| number)[]` |
| **Contravariant** (`in`) | — | `Box<B>` extends `Box<A>` | `(x: string \| number) => void` is a `(x: string) => void` |

The question is: *when a type wraps another type, does wrapping preserve or flip the subtype relation?*

## Why it matters for Layer

```ts
// From the v4 source:
interface Variance<in ROut, out E, out RIn> {
  readonly [TypeId]: {
    readonly _ROut: Types.Contravariant<ROut>
    readonly _E:     Types.Covariant<E>
    readonly _RIn:   Types.Covariant<RIn>
  }
}
```

### `ROut` is contravariant (`in`)

A layer that produces **more** services is assignable to one that produces **fewer**:

```ts
declare const big:  Layer<Config | Logger, never, never>
declare const want: Layer<Config, never, never>

const works: Layer<Config, never, never> = big
// big produces Config | Logger → fits where just Config is needed
// Contravariant: the wider union is a subtype of the narrower one
```

### `E` is covariant (`out`)

A layer with **no errors** can go where errors are expected:

```ts
declare const safe:  Layer<Config, never, never>
declare const risky: Layer<Config, SomeError, never>

const works: Layer<Config, SomeError, never> = safe
// `never` extends `SomeError` → covariant preserves the direction
```

### `RIn` is covariant (`out`)

A layer that **needs fewer** things can fit where more are expected:

```ts
declare const lean:   Layer<Logger, never, Config>
declare const hungry: Layer<Logger, never, Config | Database>

const works: Layer<Logger, never, Config | Database> = lean
// lean needs only Config → fits where Config | Database is needed
// Config extends (Config | Database) → covariant preserves it
```

## In practice

You don't write `in`/`out` annotations—TypeScript infers variance. But the Layer types *encode* variance so that:

- Merging layers widens outputs (`A | B`)
- `provide` narrows requirements (`Exclude<RIn, ROut>`)
- Error types compose safely through union

The variance markers make these type-level operations sound. Without them, you could construct a `Layer<Config | Logger>` and pass it where `Layer<Config>` is needed, then try to use the `Logger` method—TypeScript catches this at compile time.
