# Context Understood

User demonstrated understanding of Context: a typed heterogeneous map from service tags to service values. The `R` parameter in `Effect<A, E, R>` is compile-time proof of which services are needed; `yield*` retrieves from context at runtime. Compared Effect Context with LoopBack Context and identified the key difference: Effect Context is type-safe (missing service = type error) while LoopBack Context is a dynamic runtime bag.

Implications: User is ready for Layer — the mechanism that constructs services when they have their own dependencies.
