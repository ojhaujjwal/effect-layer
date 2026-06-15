# Layer in Effect Resources

## Knowledge

- [Official docs: Managing Layers](https://effect.website/docs/requirements-management/layers/)
  The canonical reference. Covers all Layer constructors, composition (merge/provide), memoization, Effect.Service, error handling.
  Use for: everything — this is page one.

- [Official docs: Layer Memoization](https://effect.website/docs/requirements-management/layer-memoization/)
  Explains v3→v4 changes in memoization: shared MemoMap across `provide` calls, `local: true`, `Layer.fresh`.
  Use for: understanding resource sharing and test isolation.

- [Effect Solutions: Services & Layers](https://www.effect.solutions/services-and-layers)
  Practical, opinionated guide with real-world patterns: Context.Service, service-driven development, test layers.
  Use for: idiomatic patterns and production-grade examples.

- [DeepWiki: Layer and Dependency Injection](https://deepwiki.com/Effect-TS/effect/2.3-layer-and-dependency-injection)
  AI-generated deep dive into the Layer source code and internals. Covers opcodes, MemoMap internals, build pipeline.
  Use for: understanding the implementation when the docs don't go deep enough.

- [YouTube: Layers, Dependency Injection, Accessors, and Scopes (Office Hours 5)](https://www.youtube.com/watch?v=P_DdDIByzTM)
  Official Effect team office hours. Demonstrates idiomatic Layer wiring and testing patterns.
  Use for: seeing the thinking process behind Layer design decisions.

- [Kwicherbelliaken Studio: Effect Layers](https://kwicherbelliaken.studio/posts/effect-layers/)
  Walkthrough from simple tagged services to Layer-based DI, with a clear "here's the problem, here's how Layer solves it" narrative.
  Use for: first exposure — excellent "why" before "how".

- [Source: Layer.ts](https://github.com/Effect-TS/effect/blob/main/packages/effect/src/Layer.ts)
  The definitive type signatures. Every public API is documented here.
  Use for: checking exact parameter types, discovering lesser-known constructors.

- [Source: Layer.test.ts](https://github.com/Effect-TS/effect/blob/main/packages/effect/test/Layer.test.ts)
  Real-world usage examples via the official test suite.
  Use for: understanding edge cases, parallel acquisition, identity preservation.

## Wisdom (Communities)

- [Effect Discord](https://discord.gg/effect-ts)
  Active community with the core team present. Fastest way to get Layer questions answered.
  Use for: design review, "is this the right way?", debugging complex layer graphs.

## Gaps

- No dedicated Layer deep-dive video course exists yet. The Office Hours video is the closest.
- Limited third-party blog content covering advanced patterns (passthrough, suspend, circular dependencies).
