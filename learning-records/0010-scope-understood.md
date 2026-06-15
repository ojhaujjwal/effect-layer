# Scope Understood

User understands Scope as a lifecycle boundary tracking finalizers run in LIFO when closed. Three key APIs: `acquireRelease` (registers cleanup on current Scope), `Effect.scoped` (auto-creates/closes Scope around an effect), and `scopedWith` (the engine inside `Effect.provide` that creates Scope → builds layer → runs program → closes Scope via `onExit`). Key insight: `acquireRelease` gets its Scope from fiber context; `scopedWith` creates a fresh one so inner acquireRelease calls register on the right boundary.

Implications: Core Layer + Scope foundation is solid. Ready for testing patterns, `Layer.suspend`, or a real project.
