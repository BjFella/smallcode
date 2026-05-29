---
name: tdd
trigger: match
keywords: [test, tdd, implement, feature, bugfix, requirements]
---

# Test-Driven Development

Strict Red → Green → Refactor loop for small models that over-implement. Can drive a single test cycle or a full requirements loop.

## Quick start — requirements loop

```
tdd_loop(requirements=[
  "add() returns the sum of two numbers",
  "add() raises TypeError on non-numeric input",
  "add() handles negative numbers"
])
```

The agent will work through every requirement in order using the cycle below. The loop only exits when all requirements have passing tests **and** a clean full-suite run confirms no regressions.

## Cycle (per requirement)

1. **Write the failing test.** One test, one behaviour.
2. **Call `tdd_begin_cycle(test_name)`** to enter the RED phase.
3. **Call `run_tests(test_filter=<name>)`** to confirm the test fails. The state machine auto-confirms RED and gates implementation until this step is done.
4. **Write minimum implementation** — only what the test requires.
5. **Call `run_tests(test_filter=<name>)`** again. The machine auto-advances to GREEN when the target passes.
6. **Call `tdd_advance`** to enter REFACTOR (or `skip_refactor=true` to skip straight to the next requirement).
7. In REFACTOR: make structural improvements. Call `run_tests` (full suite). The machine completes the cycle and auto-starts the next requirement.

## State machine tools

| Tool | When to call |
|------|-------------|
| `tdd_loop(requirements)` | Entry point for requirement-driven work |
| `tdd_begin_cycle(test_name)` | After writing the failing test |
| `run_tests(test_filter?)` | At every phase boundary — machine auto-transitions |
| `tdd_status` | Check current phase, requirements checklist, what's next |
| `tdd_advance` | Move GREEN → REFACTOR → next requirement |
| `tdd_reset` | Abandon the current loop/cycle |

## Phase gates (enforced automatically)

- **RED (unconfirmed):** writing implementation files is blocked until `run_tests` confirms the target test fails.
- **GREEN:** modifying test files is blocked.
- **Loop completion:** calling anything "done" is blocked until `run_tests` confirms the full suite is clean after all requirements are green.

## Loop completion condition

The loop is **only complete** when:
1. Every requirement's test is passing (all GREEN).
2. A full-suite `run_tests` (no filter) confirms no regressions.

The model cannot declare success before both conditions are met.

## Rules

- One test per requirement. Commit each green state.
- Never batch test + implementation in one commit.
- Bug fixes: write a regression test that fails on the old behaviour, then fix.
- Use `run_tests` (not bare `bash`) — the structured output drives the state machine.

## SmallCode tips

- `run_tests(test_filter=<name>)` narrows to a single test — faster red-phase confirmation.
- After loop completion, `memory_remember` type `workflow` if you found a project-specific test quirk.
