# Concurrency and Fiber Basics

User understands Effect's concurrency primitives (`Effect.fork`, `Effect.all`, `Promise`/`Deferred`, `Queue`) and has basic Fiber knowledge. This is relevant to Layer because Layer construction happens concurrently via `Layer.merge` and Layers interact with the fiber-scoped `MemoMap` for sharing.

Evidence: User stated "good understanding of concurrency in Effect" and "basic understanding of Fiber."
