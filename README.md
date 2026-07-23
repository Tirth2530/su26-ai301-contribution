# CodePath AI301 Contribution README

## Student Information

**Name:** Tirth Thakkar
**Program:** CodePath AI301 | AI Open Source Capstone
**Role Goal:** AI/ML Engineer or Quant Researcher
**GitHub:** https://github.com/Tirth2530

---

## Project Overview

For my AI301 open source contribution, I worked on GEPA, an open-source framework for optimizing AI programs. My selected issue focused on reducing redundant reflective dataset context for DSPy/ReAct-style traces.

The main contribution was a small adapter-level fix in GEPA’s DSPy full program adapter. The fix changes how normal cumulative traces are selected for reflection: when there is no failed prediction, the reflective dataset now keeps only the final trace entry instead of including every cumulative trace entry.

---

# Phase I: Issue Selection

## Status

Phase I Complete

---

## Selected Issue

**Issue Title:** Reflective dataset contains potentially redundant data for ReAct agent
**Repository:** `gepa-ai/gepa`
**Issue Link:** https://github.com/gepa-ai/gepa/issues/97
**Fork:** https://github.com/Tirth2530/gepa
**Working Branch:** https://github.com/Tirth2530/gepa/tree/fix-gepa-97-reflective-dataset

---

## Problem Summary

GEPA creates reflective datasets that help a language model improve AI programs by analyzing previous execution traces. In issue #97, the concern was that reflective datasets for ReAct-style agents may contain redundant trace data.

For a multi-step ReAct agent, later trace entries can already include earlier reasoning, observations, and tool-use context. If GEPA includes every cumulative trace entry in the reflection dataset, the reflection model may receive repeated information. This can increase token usage and make reflection less efficient.

In simpler terms, the issue is that the reflection data can repeat the same earlier agent steps multiple times. The goal is to preserve useful trajectory information while avoiding unnecessary duplication.

---

## Why I Chose This Issue

I chose this issue because it connects directly to my goal of becoming an AI/ML engineer. The issue involves LLM agents, ReAct-style reasoning, DSPy traces, reflective datasets, and token-efficiency optimization. These are all relevant to modern applied AI systems.

I also chose this issue because it was a good balance between being approachable and technically meaningful. It was not just a documentation update. It required me to inspect actual library code, understand how traces are represented, reproduce the behavior, implement a fix, add a regression test, and communicate with a maintainer.

From a career perspective, this contribution gives me a strong project story: I improved the efficiency of an open-source AI optimization framework by reducing redundant context in an LLM reflection pipeline.

---

## Initial Understanding

From reading the issue, I understood that `make_reflective_dataset` may create multiple trace entries where later entries repeat information from earlier entries. For a multi-step ReAct-style agent, this can lead to cumulative context being passed repeatedly into the reflection process.

My initial hypothesis was that the redundant context was not caused by the ReAct agent alone, but by how GEPA’s DSPy adapter selected trace entries for the reflective dataset. I planned to verify this by inspecting the adapter code and creating a small reproduction.

---

## Issue Selection Quality Checks

I confirmed the issue was appropriate for this contribution cycle:

* The issue was open.
* No one was assigned to it.
* There was no linked pull request already solving it.
* The repository was active and maintained.
* The repository had a usable development setup.
* The issue matched my skills and learning goals.
* The issue was small enough to investigate and submit within the AI301 timeline.

---

## Phase I Checklist

* [x] Selected a live GitHub issue
* [x] Confirmed the issue is open
* [x] Confirmed no one is assigned
* [x] Confirmed there are no linked pull requests or branches solving it
* [x] Commented on the GitHub issue expressing interest
* [x] Commented on the CodePath issue sheet row
* [x] Forked the repository
* [x] Created this Contribution README
* [x] Updated Phase I section
* [x] Submitted Phase I check-in
* [x] Announced milestone in Slack

---

# Phase II: Reproduce and Plan

## Status

Phase II Complete

---

## Branch

Working branch in my fork:

https://github.com/Tirth2530/gepa/tree/fix-gepa-97-reflective-dataset

---

## Environment Setup

I set up GEPA locally and moved to the recommended `uv`-based development workflow.

Initial commands used:

```bash
git clone https://github.com/Tirth2530/gepa.git
cd gepa
git checkout -b fix-gepa-97-reflective-dataset
```

I first attempted a regular virtual environment setup, but later switched to the repository’s recommended setup using `uv` and Python 3.11:

```bash
uv sync --extra dev --python 3.11
uv run pre-commit install
uv run pytest tests/
```

The baseline test suite ran successfully before my implementation work.

One setup challenge was related to DSPy. Some DSPy adapter tests use `pytest.importorskip("dspy")`, so those tests are skipped unless DSPy is installed. To run the relevant adapter tests, I installed DSPy and then reinstalled GEPA in editable mode to make sure Python used my local branch:

```bash
uv pip install dspy
uv pip install -e .
```

I verified that GEPA was importing from my local repository:

```bash
uv run python -c "import gepa; print(gepa.__file__)"
```

The output pointed to my local repo path:

```text
/Users/tirth2530/Desktop/gepa/src/gepa/__init__.py
```

This confirmed that my tests were running against my local implementation, not only against an installed package version.

---

## Files and Functions Investigated

I searched for `make_reflective_dataset` and found the relevant implementation in:

```text
src/gepa/adapters/dspy_full_program_adapter/full_program_adapter.py
```

The main function involved in the issue was:

```text
DspyAdapter.make_reflective_dataset
```

I also inspected the nearby DSPy adapter tests in:

```text
tests/test_dspy_full_program_adapter.py
```

This helped me understand the project’s existing test patterns and how to add a focused regression test without introducing a completely separate testing style.

---

## Reproduction Steps

I created a lightweight local reproduction script during investigation:

```text
reproduce_gepa_97.py
```

This script built a fake `eval_batch.trajectories` object shaped like what `DspyAdapter.make_reflective_dataset` expects. It did not call an actual LLM and did not require an API key. The goal was to isolate reflective dataset construction behavior.

Reproduction process:

1. Create a mock trajectory with multiple trace entries.
2. Make each later trace entry represent a cumulative step that carries forward earlier context.
3. Pass the trajectory into `DspyAdapter.make_reflective_dataset`.
4. Inspect the resulting `"Program Trace"` field in the reflective dataset.
5. Confirm whether all trace entries were preserved.

The script was useful for investigation, but it was later removed from the final PR after maintainer feedback because it was a standalone helper file rather than a permanent test.

---

## Actual Behavior

Before the fix, normal traces without `FailedPrediction` were passed into the reflective dataset with all trace entries included.

For cumulative ReAct-style traces, this means earlier context can appear repeatedly. For example:

* Trace entry 1 may contain the first reasoning/tool step.
* Trace entry 2 may include step 2 plus earlier context.
* Trace entry 3 may include the final state plus earlier context again.

Because `make_reflective_dataset` preserved every trace entry, the reflection data could contain repeated or overlapping trajectory information.

---

## Expected Behavior

For normal traces with no `FailedPrediction`, the reflective dataset should keep only the final trace entry.

The final trace entry is the most useful for cumulative traces because it represents the complete final state of the trajectory. Keeping only that final entry reduces redundant context while preserving the important information needed for reflection.

For traces with a `FailedPrediction`, the existing behavior should remain unchanged. The failed trace entry is important because it points directly to where the prediction failed and can help GEPA generate useful feedback.

---

## Root Cause

The root cause was in the trace selection logic inside `DspyAdapter.make_reflective_dataset`.

The method already had special handling for `FailedPrediction`:

* If a `FailedPrediction` was found, the method selected that failed trace entry.
* If no `FailedPrediction` was found, the method left `trace_instances` unchanged.

That meant all trace entries were included in the reflective dataset for normal successful traces. For cumulative traces, this caused redundant context.

The issue was not that ReAct itself was wrong. ReAct made the redundancy especially visible because ReAct traces often carry forward earlier reasoning and observations. The adapter-level behavior could affect any DSPy program that produces cumulative trace entries.

---

## UMPIR Plan

### Understand

Issue #97 reported redundant reflective dataset context for ReAct-style traces. The concern was that later entries already include earlier context, but GEPA was including every entry in the reflection data.

### Match

The relevant code path was `DspyAdapter.make_reflective_dataset` in:

```text
src/gepa/adapters/dspy_full_program_adapter/full_program_adapter.py
```

The relevant behavior was the selection of `trace_instances` before the program trace was converted into reflection data.

### Plan

Make the smallest possible change:

* Preserve the existing `FailedPrediction` behavior.
* If no failed prediction exists, keep only the final trace entry.
* Avoid changing unrelated adapter behavior.
* Add a regression test to prevent the issue from returning.

### Implement

Modify the non-failure path so that:

```python
trace_instances = trace_instances[-1:]
```

is used when no `FailedPrediction` is selected.

### Review

Review the diff carefully to make sure:

* The failure path is unchanged.
* The normal non-failure path now keeps only the final trace entry.
* No unrelated files or formatting-only changes remain in the final PR.

### Evaluate

Run:

```bash
uv run pytest tests/test_dspy_full_program_adapter.py
uv run pytest tests/
```

Also run pre-commit checks and confirm the PR checks pass on GitHub.

---

## Phase II Checklist

* [x] Set up the local development environment
* [x] Created and pushed a working branch
* [x] Located the relevant function
* [x] Created a lightweight reproduction
* [x] Identified expected vs. actual behavior
* [x] Identified the root cause
* [x] Wrote an implementation plan using UMPIR
* [x] Submitted Phase II check-in

---

# Phase III: Build

## Status

Phase III Complete

---

## Implementation Notes

I implemented the fix in:

```text
src/gepa/adapters/dspy_full_program_adapter/full_program_adapter.py
```

The original behavior selected a failed trace entry if a `FailedPrediction` was present. If there was no failed prediction, it kept all trace entries.

I changed the non-failure path so that when no `FailedPrediction` exists, GEPA keeps only the final trace entry:

```python
if selected is not None:
    trace_instances = [selected]
else:
    trace_instances = trace_instances[-1:]
```

This keeps the implementation small and focused. It preserves the existing failed-prediction behavior while reducing redundant cumulative context for normal traces.

---

## Code Changes

Branch:

```text
https://github.com/Tirth2530/gepa/tree/fix-gepa-97-reflective-dataset
```

Pull Request:

```text
https://github.com/gepa-ai/gepa/pull/383
```

Key commits:

```text
5df3b52 - Reduce redundant reflective trace context
143033b - Add regression test for reflective trace selection
2bff64f - Remove standalone reproduction script
```

Files changed in the final PR:

```text
src/gepa/adapters/dspy_full_program_adapter/full_program_adapter.py
tests/test_dspy_full_program_adapter.py
```

The standalone reproduction helper file was removed from the final PR after maintainer feedback.

---

## Regression Test

I added a regression test in:

```text
tests/test_dspy_full_program_adapter.py
```

The test creates a multi-entry cumulative trace and confirms that the reflective dataset keeps only the final entry when there is no failure.

The test specifically checks that:

* A trace can contain multiple entries.
* No `FailedPrediction` is present.
* `make_reflective_dataset` returns a program trace with length 1.
* The remaining trace entry corresponds to the final generated output.

This directly exercises the fixed code path.

---

## Testing Strategy

Automated tests:

```bash
uv run pytest tests/test_dspy_full_program_adapter.py
```

Result:

```text
8 passed
```

Full test suite:

```bash
uv run pytest tests/
```

Result:

```text
484 passed, 3 skipped
```

Pre-commit checks also passed for the changed files.

---

## Manual Testing and Review

In addition to automated tests, I manually reviewed the changed logic to confirm:

* The `FailedPrediction` path still selects the failed trace entry.
* The non-failure path keeps only the final trace entry.
* The final PR diff only includes the adapter change and the regression test.
* The local GEPA import path pointed to my local repository.
* The temporary reproduction script was removed from the final PR after reviewer feedback.

---

## Challenges Faced

One challenge was setting up the environment correctly. The repository uses `uv`, and some DSPy adapter tests are skipped unless DSPy is installed. I had to install DSPy and then reinstall GEPA in editable mode so tests would run against my local branch.

Another challenge was deciding how broad the fix should be. I wanted to avoid changing unrelated trace behavior, so I kept the implementation narrowly scoped to the adapter’s trace selection logic.

A third challenge was understanding how to handle the reproduction script. It was helpful during investigation, but the maintainer correctly pointed out that it should not be included in the final PR. I removed it in a follow-up commit.

---

## Phase III Checklist

* [x] Implemented the source-code fix
* [x] Added a regression test
* [x] Ran focused adapter tests
* [x] Ran the full test suite
* [x] Ran pre-commit checks
* [x] Documented files changed and key commits
* [x] Documented testing strategy
* [x] Documented challenges faced
* [x] Submitted Phase III check-in

---

# Phase IV: Submit and Iterate

## Status

Phase IV Complete / Iterating

---

## Pull Request

**PR Link:** https://github.com/gepa-ai/gepa/pull/383
**Current Status:** Iterating

---

## PR Description

This PR fixes GEPA issue #97 by reducing redundant cumulative trace context in `DspyAdapter.make_reflective_dataset`.

When no `FailedPrediction` is present, the reflective dataset now keeps only the final trace entry instead of including every cumulative trace entry. This prevents earlier steps from being repeated in the reflection context.

The existing behavior for failures is preserved: when a `FailedPrediction` exists, that failed trace entry is still selected.

---

## Issue Reference

```text
Fixes #97
```

---

## Acceptance Criteria

* [x] Pull request opened against upstream `gepa-ai/gepa`
* [x] PR is no longer a draft
* [x] PR targets the upstream default branch
* [x] PR description explains the problem and the solution
* [x] PR description references the issue with `Fixes #97`
* [x] Regression test added
* [x] Focused adapter tests pass
* [x] Full GEPA test suite passes locally
* [x] Pre-commit checks passed
* [x] Initial maintainer feedback addressed
* [x] Current status documented in README

---

## Validation

```text
uv run pytest tests/test_dspy_full_program_adapter.py → 8 passed
uv run pytest tests/ → 484 passed, 3 skipped
Pre-commit checks passed
```

---

## Maintainer Feedback Log

### Feedback 1

Reviewer asked me to remove the standalone `reproduce_gepa_97.py` helper script from the PR.

Response:

I removed the file in commit:

```text
2bff64f - Remove standalone reproduction script
```

Reason:

The reproduction file was useful during local investigation, but it was not part of the permanent source-code fix or regression test suite.

---

### Feedback 2

Reviewer asked whether the issue was specific to ReAct or applied more broadly.

Response:

I explained that ReAct makes the redundancy especially visible, but the adapter-level behavior can affect any DSPy program that produces cumulative trace entries. The fix is therefore implemented at the adapter level while still preserving the existing failed-prediction behavior.

---

## Current Next Steps

The PR is open, checks are passing, and initial maintainer feedback has been addressed. I am waiting for additional maintainer feedback.

If a reviewer requests more changes, I will respond professionally, make the requested updates in my branch, push a follow-up commit, and document the change here.

---

# Weekly Check-Ins

## Week 5 Check-In

### Current Status

I am continuing Phase IV / Iterating for GEPA PR #383.

The PR is open and under review. Initial maintainer feedback has been addressed by removing the standalone reproduction script and responding to the reviewer’s question about whether the issue is specific to ReAct or applies more broadly to DSPy cumulative traces.

### Next Steps

I am waiting for additional maintainer feedback.

---

## Week 6 Check-In

### Current Status

I am continuing Phase IV / Iterating for GEPA PR #383.

The PR is open and under review. I previously addressed initial maintainer feedback by removing the standalone `reproduce_gepa_97.py` helper script and responded to the reviewer’s question about whether the issue is specific to ReAct or applies more broadly to DSPy programs with cumulative traces.

### Next Steps

I am waiting for additional maintainer feedback. If another review comment is posted, I will respond and make any requested changes.

---

## Week 7 Check-In

### Current Status

I am continuing Phase IV / Iterating for GEPA PR #383.

The PR is still open and under review. The initial maintainer feedback has already been addressed by removing the standalone `reproduce_gepa_97.py` helper script. I also responded to the reviewer’s question about whether the issue is specific to ReAct or applies more broadly to DSPy programs with cumulative traces.

### Next Steps

I am continuing to monitor the pull request for additional maintainer feedback. If another review comment is posted, I will respond professionally and make any requested updates.

---

## Week 8 Check-In

### Current Status

I am continuing Phase IV / Iterating for GEPA PR #383.

The PR remains open and review-ready. The implementation change and regression test are still in place, all GitHub checks are passing, and the initial maintainer feedback has already been addressed.

At this stage, I am not making additional code changes unless the reviewer requests them. This keeps the PR focused and avoids unnecessary changes while it is under review.

### Next Steps

I will continue monitoring the PR for maintainer feedback. If a reviewer asks for another change, I will update the branch, push a follow-up commit, and document the response in this README.

---

# Learnings and Reflections

This contribution helped me understand the full open-source workflow from issue selection to pull request review. I learned that solving an issue is not only about writing code. It also requires reproducing the behavior, understanding the project’s existing design, writing a focused fix, adding tests, and communicating clearly with maintainers.

Technically, I learned more about how GEPA builds reflective datasets and how DSPy program traces can be represented. I also learned why cumulative traces can create redundant context for reflection and why reducing repeated context can matter for token efficiency.

The hardest part was identifying the correct level for the fix. I did not want to make a broad change that could affect unrelated behavior. I chose a narrow adapter-level change that preserves the failed-prediction path while changing the normal non-failure trace selection behavior.

I also learned that reproduction scripts are useful during investigation, but they should not always be included in the final pull request. A maintainer asked me to remove the standalone reproduction script, and I handled that feedback by removing it while keeping the actual implementation and regression test.

If I did this again, I would document my setup, expected-vs-actual behavior, and testing notes more clearly from the beginning. I would also check the grading rubric earlier so my README matched the expected structure more closely throughout the process.

---

# Final Status

* Phase I: Complete
* Phase II: Complete
* Phase III: Complete
* Phase IV: Complete / Iterating
* Current PR: https://github.com/gepa-ai/gepa/pull/383
* Current PR status: Open and awaiting additional maintainer feedback
