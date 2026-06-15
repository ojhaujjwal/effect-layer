# Error Handling in Effect

User knows expected vs unexpected errors, `Effect.fail`, `Effect.catchAll`, `Effect.catchTag`, `Either`, `Cause`, and error channel operations. This matters for Layer because Layer construction itself can fail — `Layer<ROut, E, RIn>` includes an error type `E`.

Evidence: User stated "good understanding of error handling in Effect."
