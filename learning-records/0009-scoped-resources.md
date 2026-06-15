# Scoped Resources & Cleanup Understood

User understands v4's scoped resource flow: `Layer.effect` + `Effect.acquireRelease` (no separate `Layer.scoped`). The Scope is automatically provided by the Layer build process; finalizers registered via `acquireRelease` run when `Effect.provide` closes the Scope after the program finishes.

Implications: Ready for the integration lesson — putting all concepts together with a real example.
