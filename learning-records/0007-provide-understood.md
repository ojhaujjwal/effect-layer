# Layer.provide and Effect.provide Distinction Understood

User demonstrated understanding of both `provide` operations. Key insight: `Layer.provide` is layer-to-layer (filling dependency holes in the graph), `Effect.provide` is layer-to-effect (building the layer and running the program with it). The rule: compose all layers first with `Layer.provide`, then one `Effect.provide` at the entry point.

Implications: User is ready for horizontal composition (`Layer.merge` and `Layer.provideMerge`).
