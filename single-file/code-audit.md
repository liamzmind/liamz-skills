---
name: code-audit
description: Deep, evidence-based, read-only audit of a codebase or a part of it. Finds bugs, fragile logic, security and data risks, duplication, needless complexity, architectural weaknesses, performance problems, test gaps and technical debt, then reports findings ranked by severity and leverage with file and line evidence and a recommended direction. Never modifies code. Use when the user asks for an audit, an app-wide or module-wide quality review, a health check, "what is wrong with this code", or invokes /code-audit.
---

# Code audit

Audit only. Do not edit, format, fix, install, commit or delete anything. The product is a ranked, evidence-backed report the user can act on, not a pile of observations.

## 1. Fix the scope

Before reading code, settle what is being audited: the whole repo, one module, one feature, or a diff. If the user did not say, audit the whole repo but state that scope in the report. Ask a question only if the answer would change what you look at (for example, "security only" vs "everything").

Note the goal if the user gave one (ship soon, inherit a codebase, reduce bugs). It sets which findings rank highest.

## 2. Map before judging

Build a picture of the system first, cheaply:

- Languages, frameworks, entry points, build and test setup (manifests, configs, CI files).
- Directory layout and the main modules, with what each is responsible for.
- Data flow: where input comes in, where state lives, where it goes out.
- Existing docs, decision records and conventions (README, AGENTS.md, CLAUDE.md, ADRs). Judge the code against its own stated standards, and do not flag a deliberate, documented choice as a defect.
- Churn and size hints when git is available (`git log --stat`, largest and most-changed files) to find the hot spots.

For a large codebase, split by area and delegate independent areas to subagents when available, each with the same finding format below. Consolidate and de-duplicate before ranking. Say which areas were sampled instead of fully read.

## 3. Apply the lenses

Go through each lens and skip those that do not apply. Look for real instances, not generic advice.

**Correctness**: logic errors, off-by-one and boundary cases, null and empty handling, wrong assumptions about input, time zones and dates, number precision, string encoding, state that can become inconsistent, unreachable or dead branches.

**Error handling**: swallowed exceptions, errors converted to success, missing timeouts and retries, partial failure that leaves data half written, unclear failure behavior, cleanup that is skipped on error.

**Concurrency and state**: races, shared mutable state, non-idempotent operations that may run twice, missing transactions, ordering assumptions, resource leaks (handles, listeners, connections, memory).

**Security and privacy**: injection (SQL, shell, template, path), missing authentication or authorization on any path, broken access checks (IDOR), secrets in code or logs, unsafe deserialization, weak crypto or randomness, unvalidated redirects and uploads, over-broad permissions, personal data logged or exposed, vulnerable or unpinned dependencies.

**Data and persistence**: schema vs code mismatch, missing constraints or indexes, destructive or non-reversible migrations, N+1 queries, unbounded queries, data that can be lost or duplicated.

**Duplication and reuse**: repeated logic that should be one function, component or module, near-identical files, parallel implementations of the same concept, copy-paste with drifting differences.

**Complexity and "bandage" code**: layered workarounds, flags and special cases piled on a weak design, deep nesting, god files and functions, speculative abstraction, wrappers that add nothing, config that nothing reads, code that fights its framework.

**Architecture**: unclear boundaries, circular or hidden dependencies, business logic in the wrong layer, tight coupling to a vendor or framework, shared globals, inconsistent patterns for the same job, things that will not scale with the obvious next growth.

**Performance**: work in hot paths or loops that could be hoisted, repeated I/O, missing caching or pagination, large payloads, blocking calls, unnecessary re-renders or recomputation. Report only where you can name the path and why it matters; do not guess at micro-optimizations.

**Tests and verification**: untested critical paths, tests that assert nothing meaningful, mocks that hide real behavior, flaky or order-dependent tests, no tests where a bug would be costly, missing CI checks (types, lint, security scan).

**Maintainability and operations**: unclear naming, stale or misleading comments, dead code, undocumented setup, hard-coded environment values, missing logging or monitoring on important failure points, inconsistent conventions.

**UI and accessibility** (only if there is a UI): keyboard access, focus handling, labels and contrast, loading, empty and error states, layout breaking on small screens.

## 4. Verify each finding

A finding only counts if you can point to it. For every candidate:

1. Read the actual code around it, and follow the call path far enough to confirm the problem is reachable and not already handled elsewhere (a guard, a framework default, a test, a documented decision).
2. Record the exact location as `path:line`.
3. Assign a confidence level:
   - **Confirmed**: you traced it and can describe the failing input or condition.
   - **Likely**: strong evidence, but depends on something you could not see (runtime config, data, a service).
   - **Suspected**: worth a look, not established. Keep these few and put them in their own section.
4. Drop anything that is a style preference, hypothetical without a path, or already addressed.

You may run read-only commands that do not change the project (listing, searching, `git log`, `git blame`, dependency listing, running existing tests or a linter if they only read and write nothing tracked). Ask first before anything that writes files, installs packages, hits the network with project data, or runs a build that leaves artifacts. Never claim a test or tool result you did not actually run, and say plainly what you did not check.

## 5. Rank by impact and leverage

Severity:

- **Critical**: exploitable security hole, data loss or corruption, outage on a main path, or a defect that makes results wrong without anyone noticing.
- **High**: a likely bug or failure in normal use, a serious weakness in a core area, or debt that blocks safe change.
- **Medium**: real but contained problems, notable duplication or complexity, meaningful test gaps.
- **Low**: minor inconsistencies and small cleanups. Mention briefly and group them.

Within a level, order by leverage: structural issues that cause many others, and fixes that are cheap but remove a lot of risk, come before one-off nits. Prefer a few well-evidenced structural findings over a long list of small ones. When several findings share one root cause, report the cause once and list the symptoms under it.

## 6. Write the report

Lead with the answer, then the detail.

```
# Code audit: <scope>

**Verdict**: 2-4 sentences: overall health, the biggest risks, and where to start.
**Scope and method**: what was covered, what was sampled, what was not looked at, which tools were run.

## Top priorities
The 3-7 findings to act on first, one line each, linking to the detail below.

## Findings
### [Severity] Short title — Confidence
- **Where**: path:line (and other places)
- **Problem**: what is wrong, concretely
- **Why it matters**: the consequence, with a failing scenario if there is one
- **Direction**: what to do, without writing the patch; note the cheapest safe option and any larger redesign separately
- **Effort**: S / M / L

(repeat, grouped by category, ranked within each)

## Suspected (needs confirmation)
Short list, each with what to check and how.

## Cross-cutting themes
Patterns seen in several places (for example "no consistent error handling strategy") and the one change that would address them.

## Strengths
A few things that are done well and should be kept, so they are not broken by cleanup.

## Not verified
Areas, behaviors and tools you could not check, and why.
```

Deliver the report in the conversation. Save it to a file only if the user asks. Keep it as short as the findings allow. Do not pad with generic advice, restate the code, or praise. If the code is in good shape, say so and report the few things that matter instead of manufacturing issues.

## Rules

- Report findings only. Suggest a direction, never apply a change, and offer to fix as a separate step the user must ask for.
- Question existing implementations where a simpler or more direct design exists, but respect documented decisions and say when you are challenging one.
- Separate fact from inference: what you saw, what you concluded, and what you assume.
- Treat secrets and personal data you come across as sensitive: report that they exist and where, never print the values.
- Prioritize structural and risk findings over formatting and naming.
