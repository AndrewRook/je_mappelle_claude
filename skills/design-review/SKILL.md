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
