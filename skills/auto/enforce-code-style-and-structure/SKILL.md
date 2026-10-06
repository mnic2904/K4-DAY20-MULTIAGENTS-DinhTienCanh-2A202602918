---
name: enforce-code-style-and-structure
description: Use this skill when preparing or modifying code files to ensure compliance with style, typing, and project structure rules.
---
- Do not modify existing test files; create new test files for new tests or regressions.
- Add type annotations to all public functions (functions whose names do not start with '_'), including all parameters and return types.
- Add or update a regression test file with at least one test function per fixed bug.
- Record each fix in the CHANGELOG.md under the '## Unreleased' heading with a short bullet describing the fix.
- Avoid parallel edits to the same file in one operation to prevent conflicts.
- Use consistent rounding modes and numeric handling as specified (e.g., Decimal rounding with ROUND_HALF_UP).
- Sort and filter data outputs as per specification (e.g., sorting names ignoring case).
- Always run tests after changes to verify compliance.
