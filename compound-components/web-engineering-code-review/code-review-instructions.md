# Web Engineering Code Review Standard

**Version:** 1.0  
**Audience:** Web engineers, reviewers, tech leads, and AI code-review agents  
**Scope:** React, TypeScript, TanStack Query/Router, Node.js, GraphQL, BFFs, AWS integrations, web accessibility, security, performance, and production operations.

## 1. Purpose and principles

A review exists to improve correctness, security, reliability, and long-term maintainability—not to enforce reviewer preference.

1. **Correctness first.** Verify behavior against requirements, acceptance criteria, contracts, and surrounding code.
2. **Evidence over speculation.** Report a finding only when the changed code creates a credible, explainable failure scenario.
3. **Automate mechanical checks.** Formatting, lint, type checking, generated-code validation, and routine static analysis belong in tooling where practical.
4. **Review the change in context.** Read callers, callees, types, state transitions, error paths, tests, and relevant configuration.
5. **Keep feedback actionable and respectful.** Explain the impact and suggest a practical fix; critique code, not people.
6. **Prefer small, focused changes.** A PR/MR should have one coherent purpose and include the tests and documentation needed for that purpose.
7. **Do not invent requirements.** If intent is unclear and correctness depends on it, ask a focused question instead of asserting a defect.
8. **Human accountability.** AI-generated findings and tests must be validated; never claim a command ran unless it actually ran.

## 2. Review workflow

### Pass A — Understand intent and scope
- Read the issue, acceptance criteria, MR description, screenshots, API/schema changes, and rollout notes.
- Identify the user-visible behavior and important invariants.
- Check whether the diff contains unrelated changes, generated files, dependency changes, or accidental secrets.
- If context is missing, inspect surrounding files and call sites before commenting.

### Pass B — Find high-impact defects first
Review in this order:
1. Security, privacy, authorization, and data integrity
2. Functional correctness and contract compatibility
3. State management, async behavior, concurrency, and race conditions
4. Failure handling and recovery
5. Performance and scalability
6. Accessibility and user experience
7. Test quality and missing regression coverage
8. Maintainability and architecture

### Pass C — Verify evidence
For each candidate finding:
- Point to the smallest relevant changed line range.
- Describe a concrete trigger or input.
- Explain the actual impact.
- Confirm the issue is introduced by this change, not merely pre-existing.
- Check whether existing validation, types, tests, or upstream guards already prevent it.
- Suggest the smallest safe correction.

### Pass D — Summarize
- Group findings by severity.
- List checks actually run and their results.
- State important residual risks or unverified assumptions.
- If no material issues are found, say so without manufacturing comments.

## 3. Severity and disposition

Use these project-level labels; they are a convention, not an official Microsoft/Netflix/Spotify/Atlassian classification.

| Severity | Meaning | Typical disposition |
|---|---|---|
| **Blocker** | Critical security/privacy breach, severe data corruption, or widespread outage likely if merged | Must fix before merge |
| **Major** | Important feature broken, authorization bypass, serious data loss, contract break, or high-probability production incident | Must fix before merge unless an accountable owner explicitly accepts the risk |
| **Moderate** | Meaningful edge-case defect, resilience gap, accessibility barrier, or measurable performance regression with bounded impact | Fix before merge when in scope; otherwise record an explicit follow-up |
| **Minor** | Low-impact defect with a clear scenario | Fix when practical; do not block by default |
| **Nit** | Style or preference with no meaningful behavior or maintenance impact | Do not raise unless explicitly requested |

Do not inflate severity. Do not report style preferences as bugs. Security and privacy findings should state the affected boundary and exploit/failure scenario without exposing sensitive data.

## 4. Correctness and business logic

Check:
- Does the implementation satisfy all acceptance criteria and preserve existing behavior outside the intended scope?
- Are boundary values, empty results, missing fields, invalid input, duplicate actions, retries, and partial failures handled?
- Are conditions, boolean operators, comparisons, date/time boundaries, currency/decimal handling, and sorting/pagination correct?
- Are loading, success, empty, error, and partial-data states represented correctly?
- Are mutations idempotent where retry or repeated submission is possible?
- Can stale state or a late async response overwrite newer state?
- Are feature flags, rollout paths, and fallback behavior safe?
- Are user-visible labels, status, and actions consistent with the actual server state?

Report concrete bugs, not hypothetical improvements.

## 5. TypeScript and API contracts

- Prefer precise types over `any`, unchecked casts, and broad suppression comments.
- Validate untrusted runtime data at system boundaries; TypeScript types do not validate network payloads.
- Model mutually exclusive states with discriminated unions when it prevents invalid combinations.
- Check nullability, optional properties, narrowing, exhaustive switches, and impossible states.
- Ensure generated frontend types match the authoritative BFF/OpenAPI/GraphQL contract.
- Do not manually patch generated files unless the repository explicitly requires it.
- Verify backward compatibility for request/response fields, enums, error shapes, and optionality.
- Avoid duplicated types that can drift from the source contract.
- Ensure errors and success results are distinguishable and handled exhaustively.

## 6. React and component design

- Components should have clear responsibilities and predictable props.
- Keep state as close as practical to the components that own it; avoid unnecessary global state.
- Check stable and correct React keys, controlled/uncontrolled input behavior, effect dependencies, cleanup, and stale closures.
- Do not use effects for derivable values or event-driven actions when render logic or event handlers are the correct model.
- Check that subscriptions, observers, timers, event listeners, and abortable work are cleaned up.
- Avoid unnecessary re-renders only when there is a demonstrable impact; do not demand `useMemo`/`useCallback` mechanically.
- Confirm forms handle validation, submission-in-progress, server errors, duplicate submission, and keyboard interaction.
- Keep business rules out of presentation components when they belong in domain/service layers.
- Ensure reusable components preserve semantic HTML and accessible names.

## 7. TanStack Query and Router

### Query/cache behavior
- Use stable, complete query keys that include every variable affecting the result, including tenant/account/customer context where relevant.
- Check stale data, cache lifetime, refetch policy, retries, cancellation, and error behavior.
- After mutations, invalidate or update every affected projection without unnecessarily clearing unrelated cache entries.
- Avoid cache keys that could mix data across users, customers, tenants, or authorization contexts.
- Ensure query functions propagate errors and support cancellation when appropriate.
- Avoid duplicate fetching caused by overlapping loaders, hooks, and parent layouts.
- Verify pagination, filters, sorting, and search state are reflected in the key when they affect server results.

### Routing
- Use route parameters and search parameters consistently and validate untrusted values.
- Ensure protected routes are protected by authoritative server-side authorization; client route guards are not a security boundary.
- Check direct navigation, refresh, back/forward, deep links, and missing/invalid IDs.
- Keep customer and non-customer layouts separate when their data dependencies, navigation, or security behavior materially differ; avoid duplicating routes solely for cosmetic differences.
- Avoid leaking sensitive identifiers or personal data into URLs unless explicitly approved by the product/security design.

## 8. Node.js, GraphQL, and BFF architecture

- Keep UI concerns, orchestration, domain logic, and upstream integration responsibilities clearly separated.
- Validate and normalize incoming data at the BFF boundary.
- Enforce authentication and authorization on the server for every protected operation and object; do not trust client-supplied customer IDs or roles.
- Check object-level authorization to prevent IDOR/BOLA vulnerabilities.
- Use explicit timeouts, bounded retries, cancellation, and sensible concurrency limits for upstream calls.
- Retry only safe/idempotent operations or operations protected by idempotency controls.
- Avoid N+1 upstream/database calls; use batching or DataLoader-like approaches where appropriate.
- Check GraphQL resolver authorization, pagination limits, depth/complexity controls, error redaction, and null propagation.
- Do not expose stack traces, internal service names, credentials, or upstream raw errors to clients.
- Ensure partial failures are represented consistently and do not silently convert failures into valid-looking empty data.
- Keep contract changes compatible or document and coordinate breaking changes.
- Avoid coupling frontend behavior to undocumented upstream response details.

## 9. Security and privacy

Treat security findings as high priority and validate the actual trust boundary.

- Authentication is not authorization; check permissions for each operation and resource.
- Never trust client-side role checks, hidden buttons, route guards, or customer identifiers as authorization.
- Prevent XSS: avoid unsafe HTML insertion; sanitize content when rich HTML is a real requirement; use safe rendering patterns.
- Prevent CSRF where cookie-based authentication and the application architecture require it.
- Validate and encode inputs appropriately; use parameterized queries and safe command/process APIs.
- Avoid open redirects, SSRF, path traversal, injection, insecure deserialization, and unsafe file handling.
- Do not log passwords, access tokens, session IDs, KBA answers, payment/account data, or unnecessary PII.
- Minimize PII in analytics events, URLs, browser storage, error reports, and telemetry.
- Ensure secrets are not committed, embedded in frontend bundles, exposed through source maps, or written to logs.
- Check CORS, cookie flags, CSP, token handling, session expiry, and redirect behavior where changed.
- Confirm sensitive data is cleared or refreshed appropriately on logout, user switch, tenant switch, and authorization changes.
- Avoid leaking protected data through caches, shared server rendering, browser history, or error messages.
- Prefer least privilege and explicit allowlists.

## 10. Accessibility and UX

Target **WCAG 2.2 AA** unless the product has a stricter documented standard.

- Use semantic HTML and accessible names, roles, states, and relationships.
- Ensure full keyboard operation, logical focus order, visible focus, and no keyboard traps.
- Check dialog/popover focus entry, containment, escape behavior, and focus restoration.
- Ensure dynamic errors, status updates, and asynchronous results are announced appropriately to assistive technology.
- Do not communicate meaning by color alone; check text and non-text contrast.
- Ensure form labels, instructions, validation messages, and error summaries are programmatically associated.
- Check zoom/reflow, responsive layouts, target size, reduced motion, and screen-reader behavior for changed interactions.
- Charts, tables, custom controls, and visual status indicators need meaningful accessible alternatives.
- Avoid `tabIndex` hacks and clickable non-interactive elements where native controls work.
- Verify loading, empty, error, disabled, and confirmation states are understandable and accessible.

## 11. Performance and resilience

- Check unnecessary network calls, waterfall requests, repeated renders with material cost, large payloads, and unbounded lists.
- Avoid importing large dependencies or loading expensive code on the critical path without justification.
- Use pagination, virtualization, batching, or server-side filtering when data volume requires it.
- Check image dimensions, responsive images, caching, bundle impact, and lazy-loading behavior when relevant.
- For BFF/upstream calls, check latency budgets, timeout propagation, concurrency, backpressure, and circuit-breaker/fallback behavior where needed.
- Ensure retries use bounded exponential backoff/jitter where appropriate and do not amplify outages.
- Check cache invalidation and consistency before recommending more caching.
- Avoid premature micro-optimization; prefer measurable bottlenecks and performance budgets.
- Ensure loading indicators and cancellation prevent duplicate or confusing interactions.

## 12. Testing and evidence

Tests should verify behavior and failure modes, not merely execute lines.

- Add or update tests in the same MR for changed behavior and bug fixes.
- Cover the main success path, meaningful boundary cases, and relevant failure paths.
- Include regression tests for reported defects.
- Test authorization failures and cross-customer/tenant isolation when relevant.
- Test async races, cancellation, duplicate submissions, retries, stale cache, and partial failures where affected.
- Prefer user-observable assertions over implementation-detail assertions.
- Avoid brittle snapshots that obscure meaningful behavioral changes.
- Ensure mocks match the real API contract and cannot silently diverge from production response shapes.
- Do not change tests merely to make incorrect behavior pass.
- Verify that tests fail for the bug they claim to prevent.
- State clearly which tests were run and which were not; never imply execution without evidence.

## 13. Observability and operations

- Use structured, appropriately leveled logs with correlation/request IDs where available.
- Never log secrets or unnecessary PII.
- Ensure errors preserve useful context while redacting sensitive information.
- Add or update metrics/traces for new critical workflows where operationally necessary.
- Check dashboards, alerts, runbooks, feature flags, rollback, and migration plans for material operational changes.
- Ensure telemetry failures do not break the primary user journey.
- Confirm user-facing errors are safe and actionable without exposing internals.

## 14. Maintainability and architecture

- Follow existing repository conventions unless there is a clear reason to change them.
- Keep modules cohesive and dependencies flowing in the intended direction.
- Avoid duplicated business logic, speculative abstractions, unnecessary frameworks, and oversized components/services.
- Keep names explicit and domain-appropriate.
- Comments should explain intent, constraints, or non-obvious decisions—not narrate obvious code.
- Update documentation when behavior, contracts, setup, operational procedures, or public interfaces change.
- Avoid unrelated refactors in feature/bug-fix MRs unless needed for correctness.
- Flag a design concern only when it creates a concrete cost, risk, or barrier to the requested change.

## 15. Mocking and contract fidelity

- Mock responses must satisfy the same contract as real BFF responses.
- Prefer shared schemas, generated types, contract fixtures, or validated factories over hand-written divergent mocks.
- Include representative optional fields, empty results, error responses, and discriminated-union variants.
- Avoid mock-only behavior that changes production component behavior.
- Test the component against the contract variants it actually supports.
- Keep mock enablement explicit and predictable; it must not silently alter response shapes or dropdown/domain values.
- Add contract checks to CI where practical.

## 16. Review comment format

Every actionable finding must use this format:

**[Severity] Short description**

- **Location:** `path/to/file.ts:line`
- **Scenario:** The specific input, timing, or user action that triggers it.
- **Impact:** What breaks, leaks, degrades, or becomes inconsistent.
- **Why:** The relevant invariant, requirement, or contract.
- **Suggested fix:** The smallest practical correction.

Example:

**[Major] A late search response can replace newer results**

- **Location:** `src/features/search/useSearch.ts:42`
- **Scenario:** A user types two queries quickly and the first request resolves after the second.
- **Impact:** The result list displays data for an older query.
- **Why:** The response is applied without checking that it belongs to the latest request.
- **Suggested fix:** Use query-keyed requests or cancellation and ensure only the current query result is rendered.

Use a question when intent is genuinely unclear. Use “Nit:” only for explicitly requested polish; do not disguise preferences as defects. Avoid multiple comments for the same root cause.

## 17. AI reviewer operating rules

The AI reviewer must:

1. Review only the requested diff and directly relevant surrounding code.
2. Read requirements, call sites, schemas, tests, and repository conventions before reaching conclusions.
3. Prioritize Blocker/Major findings; do not pad the review with speculative Moderate/Minor comments.
4. Report only findings that are introduced or materially worsened by the change.
5. Cite exact changed lines when possible and keep line ranges minimal.
6. Explain a realistic trigger and impact for every finding.
7. Check whether existing guards or tests already mitigate the concern.
8. Avoid generic advice such as “add error handling” without identifying the missing case.
9. Avoid style, naming, formatting, or refactoring feedback unless it creates a concrete problem.
10. Never claim to have run tests, builds, scanners, or benchmarks unless tool output confirms it.
11. Separate verified findings from questions and residual risks.
12. Do not rewrite the entire change unless asked; propose focused fixes.
13. Use professional, specific, constructive, mentoring language.
14. If no actionable defect is found, say so plainly.

## 18. Final review output template

### Summary
One or two sentences describing the change and overall risk.

### Findings
List findings by severity in descending order. For each finding, use the format in Section 16.

If no findings:
`No actionable defects identified in the reviewed diff. This does not guarantee the change is defect-free.`

### Validation
- **Inspected:** relevant areas and diff scope.
- **Executed:** commands actually run and their observed result.
- **Not executed:** relevant checks not run, if known.
- **Residual risks/questions:** only material unresolved concerns.

## 19. Merge-readiness checklist

- [ ] Change meets acceptance criteria and is focused.
- [ ] No known Blocker or Major findings remain.
- [ ] API/schema/type changes are compatible or coordinated.
- [ ] Security, authorization, and privacy boundaries are preserved.
- [ ] Loading, empty, error, retry, and partial-failure states are correct.
- [ ] Tests cover changed behavior and important regressions.
- [ ] Accessibility and performance impact are considered.
- [ ] Logs and telemetry avoid secrets and unnecessary PII.
- [ ] Documentation, flags, rollout, and rollback are updated when needed.
- [ ] Automated checks pass, or exceptions are explicitly documented and approved.

## 20. Source basis and limitations

This standard is an independent synthesis of publicly available engineering guidance, including Microsoft's Code With Engineering Playbook reviewer and PR guidance, Atlassian's public code-review guidance, and public Netflix project contribution guidance. It is not an official standard endorsed by Microsoft, Netflix, Spotify, or Atlassian, and it does not claim access to their private internal review practices.

Useful references:
- Microsoft reviewer guidance: https://github.com/microsoft/code-with-engineering-playbook/blob/main/docs/code-reviews/process-guidance/reviewer-guidance.md
- Microsoft PR guidance: https://github.com/microsoft/code-with-engineering-playbook/blob/main/docs/code-reviews/pull-requests.md
- Atlassian code review guidance: https://www.atlassian.com/agile/software-development/code-reviews
- Netflix Metaflow contribution guidance: https://github.com/Netflix/metaflow/blob/master/CONTRIBUTING.md
