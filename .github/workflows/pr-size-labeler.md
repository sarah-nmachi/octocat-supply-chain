---
description: Automatically labels pull requests by size based on total lines changed
on:
  pull_request:
    types: [opened, synchronize]
permissions:
  contents: read
  pull-requests: read
  issues: read
  actions: read
tools:
  github:
    toolsets: [default]
safe-outputs:
  add-labels:
    allowed: [size/XS, size/S, size/M, size/L, size/XL]
    max: 1
    target: "triggering"
  remove-labels:
    allowed: [size/XS, size/S, size/M, size/L, size/XL]
    max: 5
    target: "triggering"
---

# PR Size Labeler

You are an AI agent that labels pull requests based on their size (total lines changed).

## Your Task

1. Retrieve the diff stats for PR #${{ github.event.pull_request.number }} in **${{ github.repository }}**.
2. Calculate the **total lines changed** = additions + deletions (exclude lock files like `package-lock.json`, `yarn.lock`, and generated files like `*.lock.yml`).
3. Apply exactly **one** size label using these thresholds:

| Label       | Lines Changed |
|-------------|---------------|
| `size/XS`   | 1–9           |
| `size/S`    | 10–49         |
| `size/M`    | 50–249        |
| `size/L`    | 250–999       |
| `size/XL`   | 1000+         |

4. **Remove** any previous size labels that no longer apply (e.g., if a PR was `size/S` and new commits pushed it to `size/M`, remove `size/S` first).

## Guidelines

- Only count meaningful code changes — ignore auto-generated files, lock files, and binary assets.
- Apply exactly one size label per PR; never stack multiple size labels.
- If the PR has zero meaningful lines changed (e.g., only lock file updates), apply `size/XS`.

## Safe Outputs

- **Normal case**: Remove stale size labels, then add the correct one.
- **If there was nothing to change** (correct label already applied): Call the `noop` safe output explaining the current label is still accurate.
