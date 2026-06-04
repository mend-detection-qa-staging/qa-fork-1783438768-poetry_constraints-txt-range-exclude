# poetry_constraints-txt-range-exclude

**What this probe proves:** Mend honors a version-range constraint on a Poetry project, not just exact pins. A `pkg<2.0` constraint must cap the transitive at the highest 1.x release.

## Real-world scenario

Same `requests`/`urllib3` story as `poetry_constraints-txt-overlay`, but the user wrote the constraint as a range (`urllib3<2.0`) rather than an exact pin. This is more common in practice — users typically don't pin to one specific patch version.

## Files

- `pyproject.toml` — Poetry manifest pulling `requests = "2.32.0"`
- `poetry.lock` — **NOT YET GENERATED.** Run `poetry lock` on a real machine. Will reflect Poetry's unconstrained resolution (urllib3 at latest 2.x). Do NOT hand-craft.
- `constraints.txt` — `urllib3<2.0` (range, NOT exact pin)
- `whitesource.config` — `python.applyConstraints=true`

## Expected scan behavior (post SCA-5154)

Mend resolves `urllib3` to the highest 1.x release available at scan time. The exact version is non-deterministic across time (PyPI publishes new patches), so the assertion must be tolerant:

- `transitive_dependencies_names_list: contains` — `urllib3` is present (any version)
- `expected_dependency_pairs` — pin `urllib3@1.26.20` (the highest 1.x at probe-authoring time, mirrors `pip_constraints-txt-range-exclude`). **Acceptable drift risk** — the scenario is "Mend respected the range"; a SHA1/version drift is a probe-refresh signal, not a regression.

## Load-bearing assertion

The transitive is present at a 1.x version (NOT 2.x). Compare with the `poetry_constraints-txt-flag-off` sibling scan: there `urllib3` lands at 2.x. The divergence between `poetry.lock` (urllib3==2.7.0 — Poetry does NOT read constraints.txt natively) and Mend's scan output (urllib3==1.26.x) proves Mend applied the range constraint independently of the lockfile.

## Why this isn't redundant with poetry_constraints-txt-overlay

`poetry_constraints-txt-overlay` exercises exact-pin syntax (`urllib3==1.26.18`). This probe exercises range syntax (`urllib3<2.0`). Different constraint-parser code paths.