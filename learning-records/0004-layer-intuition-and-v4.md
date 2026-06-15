# Layer Intuition and v4 Context

User knows Layer provides type-safe dependency injection — unlike traditional DI containers (Symfony, NestJS) where wiring isn't type-checked. User finds Layer "tricky" — the mental model hasn't fully clicked. Committed to Effect v4 specifically; v4 source is available at `~/.local/share/effect-solutions/effect-smol/`.

Key v4 changes: `Context.Service` replaces `Context.Tag`, `Scope.extend` → `Scope.provide`, shared MemoMap across `provide` calls, convention is `layer` not `Default`/`Live`.

Evidence: User stated explicit preference for v4 and described Layer as "tricky."
