# merge and provideMerge Understood

User understands horizontal vs vertical composition: `merge` combines independent layers (both outputs preserved, built concurrently), `provide` hides the dependency, and `provideMerge` satisfies the dependency while keeping it visible — essential for test stacks where multiple services share a single dependency.

Implications: Ready for Layer memoization — how Effect shares layer instances and when to use `Layer.fresh`.
