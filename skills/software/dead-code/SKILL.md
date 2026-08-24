---
name: dead-code-removal
description: Prove and remove dead code conservatively. Use when removing unused imports, functions, classes, exports, files, legacy code, unreachable functionality, or obsolete implementation paths.
disable-model-invocation: true
---

# Dead Code Removal

Dead code removal is a **proof exercise**.

A candidate is not dead because grep found no callers. It is dead when its production reachability, indirect registrations, package/API boundary, and dynamic usage have been checked, and deleting it does not worsen the repository's validation signal.

**Only automatically remove high-confidence dead code. Surface ambiguity instead of guessing.**

## Phase 1: Establish the baseline

Before looking for dead code:

1. Read repository instructions and relevant architecture/domain documentation.
2. Inspect `git status --short`.
3. Preserve all unrelated existing changes. Never reset, checkout, clean, or overwrite them.
4. Determine the requested scope.
5. Identify package/workspace boundaries and production entry points.
6. Discover the repository's own validation commands from CI, manifests, Makefiles, package scripts, or documentation.

Run the narrowest useful validation before editing.

A repository that is already failing does not need to become green, but record existing failures so new failures can be distinguished from baseline failures.

Do not create recursive backup directories inside the repository. Prefer version control. If version control is unavailable, keep any backup outside the project root.

### Completion criterion

Before proceeding, you can state:

- the scope being analysed
- the production/package boundaries
- the validation commands
- any pre-existing failures
- any unrelated dirty files that must be preserved

## Phase 2: Map production reachability

Identify the roots from which production code can become reachable.

Inspect whichever apply to the repository:

- application, server, CLI and worker entry points
- package exports and public APIs
- routes and controllers
- dependency-injection or service registrations
- event handlers and subscribers
- scheduled jobs and queues
- framework hooks, decorators and annotations
- templates and string-based references
- reflection and dynamic loading
- configuration-driven registrations
- build configuration
- code generation
- plugin or extension registrations
- framework conventions that create implicit reachability

Do not assume conventional filenames are the real entry points. Derive entry points from the repository.

Use existing compiler, typechecker, linter, IDE and dead-code tooling where available.

Treat tool output as **candidate generation, not proof**.

## Phase 3: Generate candidates

Find possible dead code using the strongest available signals.

Useful signals include:

- compiler or typechecker unused-symbol diagnostics
- linter unused-code diagnostics
- language-aware reference search
- unused-export tools
- unreachable-code analysis
- dependency graph tooling
- AST or semantic analysis
- repository search
- unreferenced files
- obsolete feature or configuration paths identified by the task

Prefer semantic or symbol-aware tooling over raw text search.

Text search is still useful for:

- string references
- configuration
- templates
- framework registrations
- dynamic lookups
- shell scripts
- documentation that describes external consumers

Do not delete candidates during discovery.

## Phase 4: Prove each candidate

Maintain a candidate ledger, explicitly or mentally depending on scope.

For every candidate considered for deletion, establish the following.

### Origin

Why was it suspected?

Examples:

- compiler diagnostic
- unused-export tool
- unreachable branch
- unreferenced file
- repository search
- obsolete feature path
- dependent code already removed

### Direct references

Search symbol-aware references where possible, then textual references.

Check for:

- direct calls
- imports
- aliases
- inheritance
- composition
- callbacks
- re-exports
- type references
- static property access

### Indirect references

Check for usage through:

- string references
- configuration
- templates
- dependency injection
- routing
- events
- serializers/deserializers
- reflection
- runtime registries
- scheduled jobs
- queues
- plugin systems
- framework annotations/decorators
- naming conventions
- generated registration tables
- dynamic imports or module loading

Dynamic behaviour is a **candidate-level risk**, not a repository-wide veto.

The existence of `getattr`, reflection, dynamic imports, service containers, or similar mechanisms somewhere in the repository does not make all code undeletable. Determine whether the specific candidate can be reached through them.

### Boundary

Determine whether the candidate is:

- private/internal
- exported only within a closed workspace
- package-public but internally consumed
- externally consumable/public API

"No references in this repository" is not sufficient proof for an externally consumable API.

For public or externally consumable APIs, require stronger evidence such as:

- explicit task scope to remove the API
- package/versioning policy allowing removal
- known consumer inventory
- deprecation/removal plan
- repository architecture proving the package is closed-world

### Test relationship

Tests are evidence about behaviour, not proof that production code is live.

If production code is otherwise proven dead and tests exist solely for that dead functionality, remove those tests with the functionality.

Do not preserve production code merely because an isolated test calls it.

### Generated and migration code

Do not directly remove or edit generated files when the source generator should be changed instead.

Treat database migrations, schema history, protocol definitions and compatibility artefacts conservatively. Historical artefacts may be required even when nothing calls them directly.

### Confidence

Classify every deletion candidate.

#### HIGH

All of the following are true:

- internal/private or inside a proven closed boundary
- no production reachability found
- no unresolved indirect or dynamic path
- no public API obligation
- no generated/migration/history constraint
- deletion can be validated

Only HIGH-confidence candidates may be automatically removed.

#### MEDIUM

Probably dead, but uncertainty remains because it is:

- exported
- dynamically reachable
- configuration-driven
- framework-managed
- convention-driven
- crossing an uncertain package boundary
- potentially consumed externally
- difficult to validate conclusively

Preserve it and report the uncertainty unless the user explicitly authorises a riskier removal.

#### LIVE / LOW

Evidence indicates the candidate is reachable or required.

Preserve it.

A candidate cannot become HIGH confidence from textual search alone when semantic, dynamic, framework, or package-boundary uncertainty remains.

### Completion criterion

Every candidate selected for deletion has a short, concrete proof explaining why it is unreachable from production.

## Phase 5: Delete a coherent slice

Remove HIGH-confidence candidates only.

Delete the smallest coherent unit that leaves the repository internally consistent.

This may include:

- implementation
- imports
- re-exports
- tests that exist solely for the dead functionality
- configuration that solely registers it
- now-unused private helpers
- now-empty files
- package references that became unused because of the deletion

Do not mix dead-code removal with unrelated refactoring.

Do not rename, redesign, optimise, modernise, or reformat surrounding code unless required to complete the removal safely.

Deprecated does not mean dead.

Do not remove deprecated functionality solely because it is deprecated.

## Phase 6: Validate the deletion

After each coherent batch, run the narrowest relevant validation available.

Prefer repository-native commands discovered during baseline analysis.

Validation may include:

1. syntax/parsing
2. formatting checks when relevant
3. linting
4. typechecking
5. nearest affected tests
6. package/workspace tests
7. build or compile
8. repository-wide validation

Use the repository's actual commands rather than inventing generic ones when they already exist.

If validation regresses:

1. identify which deletion caused the regression
2. restore only that deletion
3. downgrade its confidence
4. record the newly discovered reachability or constraint
5. continue with unrelated proven candidates

Do not weaken tests, suppress diagnostics, alter behaviour, or broaden unrelated changes merely to make a deletion pass.

At the end, run the normal full validation appropriate to the requested scope.

If full validation cannot reasonably be run, state exactly what was and was not validated.

## Phase 7: Audit the final diff

Inspect the final diff before finishing.

Confirm:

- every changed line belongs to the dead-code removal
- unrelated existing changes remain untouched
- no behavioural replacement was introduced
- no new validation failures exist
- removed exports or re-exports leave no dangling references
- removed configuration leaves no dangling registrations
- deleted files are no longer referenced by build/package configuration
- temporary analysis files or instrumentation are gone
- generated files were handled through their source where appropriate

Run `git status --short` again and verify the working tree contains only expected changes plus any pre-existing unrelated changes.

## Final report

Report concisely:

- what was removed
- why each removal was considered dead
- ambiguous candidates deliberately preserved
- validation commands run and their results
- any limitations preventing stronger proof

Do not report line-count, byte-count, bundle-size, or performance improvements unless they were actually measured.

## Helper scripts

Helper scripts may be used to generate candidates, but they do not authorise deletion.

For example, an AST-based unused-import detector can identify possible unused imports, but each result must still pass the proof and validation process above.

When adding helper scripts:

- prefer machine-readable output
- include file and symbol locations
- avoid modifying files during detection
- make false positives easy to inspect
- document known blind spots
- treat results as evidence, never as proof
