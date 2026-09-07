---
name: maintainable-code
description: Write, modify, refactor, or review hand-authored code in any programming language, including tests, scripts, stylesheets, and markup, with readable source layout, restrained abstractions, regression awareness, and behavior-based verification. Not for prose-only tasks or generated/vendor output.
---

# Maintainable Code

Deliver the requested behavior in code the next maintainer can read without effort. Readability, correctness, and proportionate verification are all required; short code or green tests alone prove nothing.

The rules below are requirements, not preferences. Loading or naming this skill is not compliance; the diff is.

A review request produces findings, not edits.

## 1. Source layout rules (apply to the first draft of every edit)

Do not rely on a later formatting pass. The text inside each edit tool call is the delivered code; the repository formatter is a check on it, not a repair for it. Before sending an edit, read the text you are about to write and fix every violation below.

Follow the repository's formatter config and conventions when they exist; otherwise follow the language's established idioms and the defaults below. The readability standard is the same in every language. Where a rule names a construct the language lacks, apply the nearest equivalent; do not treat the language difference as a reason to skip the rule.

- One executable statement per line; do not pack independent operations onto one line. Normal loop headers are not separate statement sequences for this rule.
- Maximum line width 100 characters. Wrap arguments, conditions, object literals, and JSX attributes rather than exceeding it.
- Put control-flow and function bodies on their own indented lines, using the language's native block syntax and the repository's brace placement where applicable. Retain idiomatic short guard clauses when consistent with the surrounding code.
- No nested ternaries. Use a ternary only for a short either/or that fits on one line; otherwise use `if`/`else`, an early return, or a lookup object.
- Collection and record literals with more than three entries, or nested values: one entry per line. Use trailing commas only where the language and construct support them and conventions favor them.
- Method chains with three or more links: one `.method()` per line, wrapped in parentheses where the language needs them to continue the line.
- Compound conditions with more than two clauses: break one clause per line using valid continuation syntax, or give the condition a meaningful local name.
- CSS: one declaration per line, opening brace on the selector line, closing brace on its own line, blank line between rules. Never `a { b: c; d: e; }` on one line.
- JSX/HTML: once an element has more than two attributes or any callback prop, put each attribute on its own line. Indent children one level. When a conditional render branch hides the parent layout, move it to a named `const` above the `return` or a small component in the same file; leave short branches inline.
- Blank lines separate phases: imports / constants / state / handlers / render; or setup / action / result. Not zero blank lines, and not one after every statement.

### Example

Shown in TypeScript, CSS, and JSX because those are where compression most often appears; the same standard applies in every language.

Not acceptable:

```ts
export function summarize(items: Item[], opts: Opts = {}) { const { currency = "USD", includeTax = true } = opts; const total = items.filter(i => i.active && (i.qty > 0 || i.backordered) && !i.hidden).reduce((s, i) => s + i.price * i.qty, 0); return { total: includeTax ? total * (1 + (opts.taxRate ?? 0)) : total, currency, count: items.length, label: total > 1000 ? "large" : total > 100 ? "medium" : "small" }; }
```

```css
.card{display:flex;gap:8px;padding:12px 16px;border:1px solid var(--line);border-radius:8px}.card:hover{border-color:var(--accent)}
```

```tsx
return <div className="row">{items.length === 0 ? <Empty /> : items.map(i => <Row key={i.id} item={i} onSelect={() => select(i.id)} selected={i.id === selectedId} disabled={!i.active} />)}</div>;
```

Acceptable (same behavior, including the label computed from the pre-tax subtotal):

```ts
export function summarize(items: Item[], opts: Opts = {}) {
  const { currency = "USD", includeTax = true, taxRate = 0 } = opts;

  const billable = items.filter((item) => {
    const hasQuantity = item.qty > 0 || item.backordered;
    return item.active && hasQuantity && !item.hidden;
  });

  const subtotal = billable.reduce((sum, item) => sum + item.price * item.qty, 0);
  const total = includeTax ? subtotal * (1 + taxRate) : subtotal;

  return {
    total,
    currency,
    count: items.length,
    label: sizeLabel(subtotal),
  };
}

function sizeLabel(amount: number): "large" | "medium" | "small" {
  if (amount > 1000) return "large";
  if (amount > 100) return "medium";
  return "small";
}
```

```css
.card {
  display: flex;
  gap: 8px;
  padding: 12px 16px;
  border: 1px solid var(--line);
  border-radius: 8px;
}

.card:hover {
  border-color: var(--accent);
}
```

```tsx
const rows = items.map((item) => (
  <Row
    key={item.id}
    item={item}
    selected={item.id === selectedId}
    disabled={!item.active}
    onSelect={() => select(item.id)}
  />
));

return <div className="row">{items.length === 0 ? <Empty /> : rows}</div>;
```

### Editing mechanics

- Use the patch/edit tool for hand-authored source. The intended code must be visible and readable in the edit itself.
- Never compress source to make an edit call shorter. Split a large change into several coherent edits instead.
- Do not use shell find-and-replace, regex substitution, or scripts as a formatter. Scripted transformations are only for genuinely repetitive structural work; scope them, confirm the match count, and inspect the resulting diff.
- Do not restyle generated or vendor files. Change their source or generator if that is in scope.

## 2. Structure the logic for a reader

- Name things by domain meaning. Introduce a named intermediate value when it makes a complex expression readable; not for every trivial step.
- Prefer flat control flow: early returns over deep nesting, `if`/`else` over clever expressions, explicit steps over hidden side effects.
- Extract a helper when it has one coherent responsibility and removes real duplication or makes the caller readable. Do not shred a readable function into many tiny helpers, and do not impose line-count targets.
- In UI components, a maintainer should find the main layout, each meaningful state, and each user action without reconstructing them from scattered conditions. Keep data preparation, state decisions, and rendering visually distinct. Extract coherent interface units when a render branch obscures the parent layout; keep them in the same file unless a separate file is clearly better.
- Comment non-obvious business rules, ordering constraints, and workarounds. Do not comment obvious syntax, and do not use a comment to excuse a bad name.
- Tests and diagnostic helpers get the same layout and structure standards as production code.

## 3. Before editing: understand the change and its blast radius

- Read the relevant instructions, the code being changed, its callers, its tests, and formatter/CI config. Keep this proportional: a local fix does not need a repository audit.
- Write down, at least to yourself, the behavior to change and the adjacent behavior that must stay the same.
- Before changing a signature, return shape, CSS class, or exported name, search for every usage and account for each one.
- When the change touches behavior and the touched area has tests, run them before editing so you know which failures are pre-existing. Skip this for comment, copy, or pure styling changes.
- Preserve the user's uncommitted changes. Edit only what the request needs plus what is required to make it correct and readable.
- Ask only when a missing decision materially blocks correctness. Otherwise proceed and state the assumption.

## 4. Keep the solution proportionate

- Prefer the direct implementation and existing utilities. Add an abstraction, dependency, config option, or fallback path only for a requirement that exists now or a maintenance benefit you can name.
- Do not build for hypothetical future needs. Three similar lines are better than a premature framework.
- While simplifying, preserve error handling, validation, resource cleanup, accessibility, and performance characteristics. Do not hide failures behind broad exception handlers or defaults that look like success.

## 5. Verify behavior, not the implementation

- Decide the expected result before writing the code, from the request, a documented contract, or a hand-worked example. Never derive an expected value by running the function under test and pasting its output.
- Reuse existing test files and fixtures. Create a new test file only when conventions require it or nothing suitable exists.
- For a bug fix, reproduce the failure first. Show the check failing before and passing after when practical; otherwise state what evidence you used and where verification stops.
- Cover a plausible boundary or adjacent regression when the change creates that risk. Do not add tests to raise a count.
- A change to an assertion, snapshot, mock, skip, or tolerance is a separate decision from the implementation change. Never weaken an assertion, delete failing coverage, or refresh a snapshot to get green. Change an expectation only when the requested behavior or an authoritative contract justifies it, and say why. A mock must not bypass the behavior the test claims to establish.
- Verify at the level that establishes the behavior: unit/API tests for contracts, browser or device checks for interactions, visual inspection for layout. A success toast does not prove the imported records are correct; check the data and state transitions the feature promises.
- Match verification to the change. A behavior change gets the project's compile and test checks where they exist, and those passing is not by itself sufficient: for a user-visible workflow, exercise the workflow. A comment or styling change needs only a look at the result. In an existing project that lacks test tooling, say so rather than installing it or treating the gap as a pass. In a new project, choose test tooling appropriate to the language and scope and use it; a missing test runner is not a reason to ship logic with no automated checks.
- Run the repository's formatter and lint on changed files only. Do not reformat unrelated files or install tooling just to satisfy this skill. Formatter success establishes style consistency, not readable structure or correctness.
- For important logic or when an assertion looks weak, consider a targeted mutation check in a temporary copy: introduce a plausible mistake such as a flipped boundary and confirm the test catches it. Keep mutations out of delivered code. This is selective, not a required stage.

## 6. When a fix does not work

If the same failure persists after two attempted fixes, stop stacking fixes. Re-read the error, reduce the reproduction, list the assumptions the fixes shared, and test a different hypothesis. This is a reasoning reset, not a stop or a permission request.

## 7. Before reporting done

1. Read the full diff (for example `git diff`), not just the last edit.
2. Check the diff against every item in section 1. Any violation is fixed now, in this task.
3. Check for accidental deletions, unrelated changes, leftover debugging code, and abstractions the request did not need.
4. For any function that handles several modes or commands, check whether a reader can tell which mode each condition belongs to without tracking earlier returns. If not, dispatch per mode. Where a later statement is only safe because an earlier check must have returned, say so in a comment.
5. Confirm the pre-existing behavior identified in section 3 still works, by running the relevant checks.
6. Report: what behavior changed, which specific checks ran and what they establish, and what remains unverified. Do not write "followed the maintainable-code skill"; name the concrete checks instead. Do not claim bug-free results or treat an unavailable check as a pass.
