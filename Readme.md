# Prompt 12 Generate Work-Order References

This README explains, step by step, what was done to implement the prompt:

> Create `apps/backend/src/domain/reference.ts` with `nextWorkOrderReference(existingReferences, year)`.

## Goal

Generate the next Work Order reference for a given year, in the format `WO-YYYY-NNNN`, where:

- Only references belonging to the requested year count toward the sequence.
- The sequence starts at `0001` when no reference exists for that year.
- Gaps are allowed — the next number is always `max + 1` of the existing sequence numbers for that year.
- The function must not crash or miscount when given unsorted input or unrelated/malformed strings.

## Steps followed

1. **Explored the existing conventions** in `apps/backend` before writing any code:
   - Read [workOrderLifecycle.ts](apps/backend/src/domain/workOrderLifecycle.ts) to see how domain logic is structured (plain functions, typed inputs/outputs, no classes).
   - Read [workOrderLifecycle.test.ts](apps/backend/src/domain/workOrderLifecycle.test.ts) to match the existing test style: `vitest`, `describe`/`it`, relative imports with a `.js` extension (required by the project's ESM/TypeScript setup).

2. **Implemented `nextWorkOrderReference`** in [apps/backend/src/domain/reference.ts](apps/backend/src/domain/reference.ts):
   - Used a strict regular expression (`^WO-(\d{4})-(\d{4})$`) to parse each existing reference and safely ignore anything that doesn't match the exact format.
   - Filtered the parsed references down to the ones matching the requested `year`.
   - Computed the next sequence number as `Math.max(...sequences) + 1`, or `1` if there were no matches for that year — this naturally supports gaps and unsorted input, since `Math.max` doesn't care about order.
   - Formatted the result back into `WO-YYYY-NNNN`, zero-padding the sequence to 4 digits.

3. **Wrote tests** in [apps/backend/src/domain/reference.test.ts](apps/backend/src/domain/reference.test.ts) covering the required cases:
   - No reference exists for the requested year → returns `WO-YYYY-0001`.
   - References from older years are ignored.
   - Input references are unsorted, and the function still finds the correct next number.
   - Gaps in the sequence are allowed (`max + 1`, not "next after the count").
   - Unrelated/malformed strings (empty string, wrong format, extra suffix) are ignored safely instead of throwing.

4. **Ran the test suite** with `npx vitest run src/domain/reference.test.ts` from `apps/backend` to confirm all 5 tests pass.

## How to run the tests yourself

```bash
cd apps/backend
npx vitest run src/domain/reference.test.ts
```

## Running the app (Windows terminal)

There is currently **no runnable app** in this repository yet — only domain logic and its tests exist so far (`apps/backend/src/domain`). There is no Vite dev server, no Fastify server entry point, and no frontend `src` folder yet. The root `package.json` confirms this: its `dev` script is just a placeholder.

From `powershell.exe` in the project root:

```powershell
npm run dev
```

Running it today will only print:

```
no dev server configured yet
```

Once the frontend (Vite) and backend (Fastify) apps are actually wired up in a later prompt, this README will be updated with the real commands to start them (e.g. `npm run dev --workspace=@equipment-hub/frontend` / `--workspace=@equipment-hub/backend`, or a combined `npm run dev` from the root). For now, use the test command below to verify the code that does exist.

## Key takeaway for students

- Keep domain logic pure and framework-free (no I/O, no side effects) — easy to test and reason about.
- Look at neighboring files before writing new code, to match the project's conventions (naming, import style, test structure) instead of inventing new ones.
- When a spec lists explicit test cases ("no reference exists", "gaps are allowed", etc.), turn each one into its own `it(...)` block — it makes the requirements traceable directly from the test file.
