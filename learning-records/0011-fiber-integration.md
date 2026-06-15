# Fiber Integration Understood

User understands the fiber as the execution environment: every fiber carries a Context holding services, Scope, and MemoMap. `Layer.build` reads Scope/MemoMap from the fiber via `core.withFiber`. Forked fibers inherit the parent's context, so layered services are available in children. In v4, `forkScoped`/`forkChild` auto-provide Scope for cleanup.

Implications: Full stack covered — Context → Layer → Scope → Fiber. Ready for testing, suspend, or a real project.
