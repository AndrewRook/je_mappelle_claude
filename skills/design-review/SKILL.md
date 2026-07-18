---
name: design-review
description: Personal code design review guidelines. Use when the user asks to review code, review a diff, design new code, plan new features,
or invokes /design-review.
---

These are general design guidelines to help structure and organize code, as well as make it more reliable, flexible,
and easier to maintain. They operate at a higher level than the core code review concepts in the built-in /code-review
skill.

## Guidelines

- **Silent overrides are risky.** When one layer of config/options can be overridden
  by another (merged dicts, cascading defaults, later assignments), check whether the
  override can clobber something the surrounding code relies on, not just whether the
  override mechanism itself works.

- **Prefer erroring over silently assuming.** When a value can't be determined and
  the code falls back to a default, check whether that default is standing in for
  genuine uncertainty (vs. expected/acceptable imprecision). If downstream logic
  treats the fallback as ground truth, an unrecoverable "unknown" case should fail
  loudly rather than guess quietly.

- **Don't infer by elimination.** When code decides what something is by ruling out
  other possibilities rather than checking for an explicit, positive signal, check
  whether that fallback would mask a genuine error (bad input, typo, wrong type) as
  if it were just another valid case.

- **Strive for 100% test coverage, but compromise when needed.** There is no reason why
  a coverage check shouldn't return a 100% test coverage over all lines of a function.
  If the function can be tested in a way that's valuable, then it should be tested. If
  it cannot be cleanly tested (e.g. it would require a ton of mocks, or hacking internal
  attributes of an object), then it should be explicitly ignored in coverage checks using a
  nocover pragma. There should be a comment directly before the pragma that explains why
  the function is not covered (usually because it deals with I/O or a database or something
  not easily testable). Additionally, don't make 1-2 line functions _just_ so those lines
  of code can be tested, unless they're really critical and complex. Those should be inlined
  to reduce boilerplate and enhance readability.

- **Sanity-check implementation complexity against the naive approach.** When we choose to 
  do a more complex/advanced approach to solve a problem, don't just focus on the value created by
  the more complex code but balance that with the operational costs of the complexity. For example,
  How many more lines of code will need to be maintained? Are we using advanced language features that
  future maintainers may not know about?
  If we decide that the complexity is truly worth those costs, document the justification.
  Otherwise it's over-engineering.

- **Treat `nonlocal` and `global` variables as smells.** 
  Mutating an enclosing variable via `nonlocal` or `global` hides data flow and can lead to
  very subtle bugs; these langauge features are also not newcomer-friendly. If a function or
  module would require these in order to work as intended, strongly consider other options (up
  to and including complete restructuring of the relevant parts of the codebase).

- **Avoid function-in-function definitions where possible.** While there are some uses for defining
  a function inside of another function's definition (e.g., decorators or simple lambdas), in general
  this pattern is unnecessarily complex and makes writing tests more challenging. Be extra cautious
  of approving code with functions defined this way.

- **Don't carry a redundant identifier for a resource you already hold a handle to.** If a
  job/message needs to reference an object (e.g. a volume, a file) the caller already resolved,
  check whether it's re-deriving that object from a separately-threaded name/id it could avoid
  by passing the handle through instead — or the reverse: passing a bigger identifier (e.g. a
  fully-prefixed path) than the receiver needs when it already has the context to use the
  narrower piece.

- **Prefer the framework's supported mechanism over an environment hack.** `sys.path`
  manipulation, monkeypatching, or other manual workarounds to make imports/wiring work are a
  signal to check the framework's own docs/conventions (e.g. Modal's image/mount system) for a
  built-in way to do the same thing before reaching for the hack.

- **Check new code against the project's own module-boundary rules.** If the project documents
  a split between shared code and consumer-specific code (e.g. a general `src/` vs. a
  script-specific source directory), verify new code whose only real caller is one consumer
  actually lives in that consumer's location rather than the shared one.
