# Layer Cheat Sheet — Effect v4

Type-safe dependency injection.

## Service Tags

**Function-style:**
```ts
const Foo = Context.Service<{
  readonly bar: Effect.Effect<number>
}>("Foo")
```

**Class-style (preferred):**
```ts
class Foo extends Context.Service<Foo, {
  readonly bar: Effect.Effect<number>
}>()("app/Foo") {}
```

> Identifiers must be unique. Use `"app/ServiceName"` prefix pattern.

## Layer Type Signature

```
Layer<ROut, E, RIn>
//     ↑     ↑   ↑
//   produces  may  requires
//   (output)  err  (input)
```

| Parameter | Variance | Meaning |
|-----------|----------|---------|
| `ROut` | Contravariant | Services this layer produces |
| `E` | Covariant | Errors during construction |
| `RIn` | Covariant | Services this layer needs as input |

> Mnemonic: **R**esources Output, **E**rrors, **I**nputs needed.

## Constructors

| Constructor | Signature | When |
|-------------|-----------|------|
| `Layer.succeed` | `(tag, implementation) → Layer<I>` | No dependencies, no errors, no async |
| `Layer.sync` | `(tag, () => impl) → Layer<I>` | Lazy construction, no deps |
| `Layer.effect` | `(tag, Effect<S, E, R>) → Layer<I, E, R>` | Has dependencies (`yield*` inside), may fail, may be async |
| `Layer.mock` | `(tag, partial) → Layer<I>` | Test mocks — only stub what you use |
| `Layer.empty` | `() → Layer<never>` | No-op placeholder |

```ts
// succeed — simplest
const ConfigLive = Layer.succeed(Config, {
  get: Effect.succeed({ level: "INFO" })
})
// Layer<Config, never, never>

// sync — lazy stateful mock
const CounterLive = Layer.sync(Counter, () => {
  let count = 0
  return {
    get: () => Effect.succeed(count),
    inc: () => Effect.sync(() => void count++)
  }
})

// effect — with dependencies
const LoggerLive = Layer.effect(Logger,
  Effect.gen(function* () {
    const config = yield* Config              // ← dependency
    return { log: (msg) => Effect.log(`[LOG] ${msg}`) }
  })
)
// Layer<Logger, never, Config>

// mock — partial test layer
const UserServiceTest = Layer.mock(UserService, {
  config: { apiUrl: "test" },                 // required (non-Effect)
  getUser: (id) => Effect.succeed({ id, name: "Test" })
  // deleteUser, updateUser omitted → throw if called
})
```

## Composition

| Operation | Direction | Output | Use for |
|-----------|-----------|--------|---------|
| `Layer.merge(A, B)` | Horizontal | `A \| B` | Combine independent leaf services |
| `A.pipe(Layer.provide(B))` | Vertical | `A` only | Hide B from output (encapsulation) |
| `A.pipe(Layer.provideMerge(B))` | Vertical + passthrough | `A \| B` | B shared by multiple consumers |

```ts
// provide — B satisfies A, B hidden from output
const LoggerWithConfig = LoggerLive.pipe(
  Layer.provide(ConfigLive)
)
// Logger: Layer<Logger, never, Config> + Config: Layer<Config, never, never>
// Result: Layer<Logger,     never,    never>

// provideMerge — B satisfies A, B kept in output
const LoggerWithConfig = LoggerLive.pipe(
  Layer.provideMerge(ConfigLive)
)
// Result: Layer<Logger | Config, never, never>

// merge — combine side by side, built concurrently
const AllServices = ConfigLive.pipe(
  Layer.merge(LoggerLive)
)
// Result: Layer<Config | Logger, never,  Config>
//                         Logger needs Config → hole remains!
```

> **Rule:** Use `provide` at the declaration site. Close dependencies where the service is defined, not where it's consumed.

## Wiring to Effects

| Operation | Signature | Level |
|-----------|-----------|-------|
| `Effect.provide` | `Effect<A,E,R> → Layer<ROut,...> → Effect<A,E, R-ROut+...>` | Entry point — layer meets program |
| `Effect.provideService` | `Effect<A,E,R> → Tag → impl → Effect<A,E, R-Tag>` | Quick single-service provision |

```ts
// Compose all layers, then provide once
const AppLayer = LoggerLive.pipe(Layer.provide(ConfigLive))

const program = Effect.gen(function* () {
  const logger = yield* Logger
  yield* logger.log("hello")
})

const runnable = program.pipe(Effect.provide(AppLayer))
// Effect<void, never, Logger> → Effect<void, never, never>

Effect.runPromise(runnable)
```

> **One `Effect.provide` at the top.** Compose layers first, then wire in once.

## Scoped Resources & Cleanup

```ts
const DatabaseLive = Layer.effect(Database,
  Effect.gen(function* () {
    const pool = yield* Effect.acquireRelease(
      Effect.tryPromise(() => createPool(url)),   // acquire
      (pool, exit) => Effect.promise(() =>        // release
        pool.end()
      )
    )
    return { query: (sql) => Effect.tryPromise(() => pool.query(sql)) }
  })
)

// acquireRelease auto-registers cleanup on the current Scope
// Effect.provide → scopedWith → Scope created → Scope closes after program → cleanup runs
```

> In v4, no separate `Layer.scoped` — just use `Effect.acquireRelease` inside `Layer.effect`.

## Memoization

| Mechanism | What | Use for |
|-----------|------|---------|
| Default | Same reference = built once | Normal operation |
| `Layer.fresh(layer)` | Always rebuild this layer | One service must never be shared |
| `provide(layer, { local: true })` | Entire subtree gets private MemoMap | Test isolation for a whole stack |

```ts
// Footgun: parameterized constructors
const bad = Layer.merge(
  UserRepo.pipe(Layer.provide(postgresLayer({ url }))),  // ← different reference
  OrderRepo.pipe(Layer.provide(postgresLayer({ url })))  // ← different reference
) // TWO database pools!

// Fix: bind to constant
const db = postgresLayer({ url })                        // ← ONE reference
const good = Layer.merge(
  UserRepo.pipe(Layer.provide(db)),
  OrderRepo.pipe(Layer.provide(db))
) // ONE pool, shared

// Fresh: force rebuild
program.pipe(
  Effect.provide(Layer.fresh(DatabaseLive))  // always fresh
)
```

## Testing

| API | What |
|-----|------|
| `it.effect(name, fn)` | Run Effect test with auto-Scope + TestEnv |
| `it.layer(layer)(name, fn)` | Share layer across suite (built once, cleanup in afterAll) |
| `it.live(name, fn)` | Run with real runtime (no TestClock/TestConsole overrides) |
| `Layer.mock(tag, partial)` | Partial mock — unimplemented methods throw if called |

```ts
// Per-test isolation (preferred)
it.effect("test", () =>
  Effect.gen(function* () {
    const db = yield* Database
    expect(yield* db.query("...")).toEqual([...])
  }).pipe(Effect.provide(DatabaseTest))
)

// Suite-shared layer (for expensive setup)
layer(DatabaseLive)("shared DB", (it) => {
  it.effect("test 1", () => ...)  // uses shared DatabaseLive
  it.effect("test 2", () => ...)  // same instance, shared state
})

// Selective freshness in shared suite
layer(DatabaseLive)("shared DB", (it) => {
  it.effect("stateful test", () =>
    Effect.gen(function* () { ... })
      .pipe(Effect.provide(Layer.fresh(CounterLive)))
  )
})
```

> **Golden rule:** prefer per-test isolation. Only share when construction is expensive.

## Naming Conventions (v4)

| Pattern | Meaning | Example |
|---------|---------|---------|
| `layer` | Primary implementation | `Logger.layer` |
| `layerTest` | Test implementation | `Logger.layerTest` |
| `layerX` | Variant | `Logger.layerSqlite` |

> v4 convention: generic `layer`, not v3's `Live`/`Default`.

## Common Patterns

```ts
// Pattern: close deps at declaration site
class Logger extends Context.Service<Logger, {
  readonly log: (msg: string) => Effect.Effect<void>
}>()("app/Logger") {
  static readonly layer = Layer.effect(this,
    Effect.gen(function* () {
      const config = yield* Config
      return { log: (msg) => Effect.log(`[LOG] ${msg}`) }
    })
  ).pipe(Layer.provide(Config.layer))  // ← closed here
}
// Logger.layer now has Layer<Logger, never, never>

// Pattern: multiple services sharing one dependency
const AppLayer = Layer.mergeAll(
  UserRepoLive.pipe(Layer.provideMerge(DatabaseLive)),  // keep DB visible
  OrderRepoLive.pipe(Layer.provideMerge(DatabaseLive)), // same DB instance
  ConfigLive                                             // leaf
)
// Layer<UserRepo | OrderRepo | Database | Config, ...>

// Pattern: build entire app from layers
const program = Effect.gen(function* () {
  yield* HttpServer.start
  yield* Effect.never  // keep alive
})

const main = program.pipe(Effect.provide(AppLayer))
Effect.runPromise(main)
```

---

Based on Effect v4 source (`packages/effect/src/Layer.ts`) and the [official docs](https://effect.website/docs/requirements-management/layers/).
