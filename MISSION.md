# Mission: Layer in Effect V4

## Why
I understand Effect's core primitives (Effect, pipe, generators), error handling, concurrency, and Fiber basics — but Layer is the piece I keep avoiding. I know it provides type-safe dependency injection, unlike traditional DI containers (Symfony, NestJS) where wiring is not type-checked. I want to fill this gap so Effect becomes a complete toolset, not one where I sidestep dependency management.

## Success looks like
- Wire up a non-trivial Effect application from scratch, designing its full dependency graph with Layer
- Read and debug any Layer-based Effect code fluently, understanding the wiring and failure modes at a glance
- Teach Layer's design, tradeoffs, and patterns clearly to another developer

## Constraints
- Session length is variable (short or long, no fixed pattern)

## Out of scope
- Nothing explicitly ruled out — covering Layer + its runtime integration (Scope, Runtime, Fiber interaction) is in scope
