# Testing with Layers Understood

User understands six testing patterns: `Layer.mock` for partial mocks, `Layer.succeed`/`Layer.sync` for manual test layers, `it.effect` for per-test isolation (default), `it.layer` for suite-shared expensive resources, `Layer.fresh` for selective freshness in shared suites, and `it.live` for integration tests without TestEnv overrides.

Implications: All core Layer topics covered. Ready for `Layer.suspend`, `Layer.flatMap`, or a real project.
