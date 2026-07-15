# CodePath AI301 Contribution README

## Student Information

**Name:** Tirth Thakkar
**Program:** CodePath AI301 | AI Open Source Capstone
**Role Goal:** AI/ML Engineer or Quant Researcher

---

## Phase I: Issue Selection

### Status

Phase I Complete

---

### Selected Issue

**Issue Title:** Reflective dataset contains potentially redundant data for ReAct agent
**Repository:** gepa-ai/gepa
**Issue Link:** https://github.com/gepa-ai/gepa/issues/97

---

### Problem Summary

This issue focuses on redundant trajectory data being included in GEPA’s reflective dataset for a ReAct agent. In the current behavior, multiple trace entries appear to repeat previous context, observations, and intermediate steps, which can cause the reflection language model to process more tokens than necessary.

This matters because repeated trajectory data can make LLM-based reflection more expensive and less scalable, especially when the agent has a large initial context. A better reflective dataset structure could help preserve the important trajectory information while reducing unnecessary token usage.

---

### Why I Chose This Issue

I chose this issue because it connects directly to my goal of becoming an AI/ML engineer. The issue involves LLM agents, ReAct-style reasoning, reflective datasets, and token-efficiency optimization, which are all highly relevant to modern applied AI systems.

I also chose it because the issue is labeled as a good first issue, has a clear problem description, and includes enough context to begin investigating. Compared to a documentation-only contribution, this issue gives me a chance to work on real AI infrastructure and understand how agent trajectories are represented and optimized.

From a career perspective, this contribution also gives me a strong project story: improving the efficiency of an open-source AI optimization framework by investigating redundant data in an LLM reflection pipeline.

---

### Initial Understanding

From reading the issue, I understand that `make_reflective_dataset` may currently create multiple trace entries where later entries repeat information from earlier entries. For a multi-step ReAct agent, this can lead to cumulative context being passed repeatedly into the reflection process.

A potential solution may involve changing how reflective examples are constructed so the reflection model receives the most useful trajectory information without unnecessary repetition. Before implementing anything, I will reproduce the issue locally and inspect the relevant adapter logic.

---

### Phase I Checklist

* [x] Selected a live GitHub issue
* [x] Confirmed the issue is open
* [x] Confirmed no one is assigned
* [x] Confirmed there are no linked pull requests or branches
* [x] Commented on the GitHub issue expressing interest
* [x] Commented on the CodePath issue sheet row
* [x] Forked the repository
* [x] Created this Contribution README
* [x] Updated Phase I section
* [x] Submitted Phase I check-in
* [x] Announced milestone in Slack

---

## Phase II: Reproduce & Plan

### Branch

Working branch in my fork:
https://github.com/Tirth2530/gepa/tree/fix-gepa-97-reflective-dataset

### Local Setup

I cloned my fork of the GEPA repository, created a new working branch, and set up a local Python virtual environment.

Commands used:

```bash
git clone https://github.com/Tirth2530/gepa.git
cd gepa
git checkout -b fix-gepa-97-reflective-dataset
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -e ".[dev]"
pip install dspy
pip install -e ".[dev]"
```

After installing DSPy, I reinstalled GEPA in editable mode to make sure Python was using my local repository version instead of the package installed from PyPI.

Verification:

```bash
python -c "import gepa; print(gepa.__file__)"
```

This pointed to my local repo path:

```text
/Users/tirth2530/Desktop/gepa/src/gepa/__init__.py
```

### Files Investigated

I searched for `make_reflective_dataset` and found the relevant implementation in:

```text
src/gepa/adapters/dspy_full_program_adapter/full_program_adapter.py
```

The important function is:

```text
DspyAdapter.make_reflective_dataset
```

### Reproduction Steps

I created a lightweight reproduction script:

```text
reproduce_gepa_97.py
```

The script builds a fake `eval_batch.trajectories` object shaped like what `DspyAdapter.make_reflective_dataset` expects. It does not call an actual LLM or require an API key. The goal was to isolate the reflective dataset construction behavior.

To run the reproduction:

```bash
python reproduce_gepa_97.py
```

### Reproduction Evidence

The script produced this key output:

```text
GEPA #97 reproduction
============================================================
Number of trace entries in reflective dataset: 3
```

It showed that `make_reflective_dataset` keeps all three trace entries inside `"Program Trace"`:

1. The first trace entry contains the original question and first tool call.
2. The second trace entry repeats earlier context through `previous_step`.
3. The third trace entry includes `full_trajectory_so_far`, which again carries forward information from earlier steps.

This supports the issue’s concern that for cumulative ReAct-style traces, later entries may already contain earlier information. Keeping every trace entry can send repeated context to the reflection LM.

### Current Understanding

In `make_reflective_dataset`, the code loops through `trace_instances` and appends each processed trace item into `trace_d`:

```python
trace_d.append(d)
```

Because of this, if the trace is cumulative, the reflective dataset may include repeated or overlapping information. For ReAct-style agents, each later step can already include previous reasoning/tool context, so including all trace entries may create redundant reflection data.

### Implementation Plan

My plan is to keep the change small and focused:

1. Add a test or reproduction-style case that uses a cumulative ReAct-like trace.
2. Confirm that the current behavior includes multiple repeated trace entries.
3. Investigate whether the adapter should:

   * keep only the final/full trajectory for reflection, or
   * deduplicate repeated cumulative context, or
   * add an optional setting so existing behavior stays backward-compatible.
4. Prefer a minimal fix that reduces redundant trace data without changing unrelated adapter behavior.
5. Run the reproduction script and relevant tests before opening a pull request.

### Phase II Status

Phase II reproduction and planning are complete. I have:

* Set up the local development environment.
* Created and pushed a working branch.
* Located the relevant function.
* Created a reproduction script.
* Confirmed that multiple cumulative trace entries are kept in the reflective dataset.
* Written an initial implementation plan.


---

## Phase III: Build

### Implementation Notes

* Set up the GEPA development environment using `uv` with Python 3.11.
* Reviewed `CONTRIBUTING.md`, installed pre-commit hooks, and ran the baseline test suite.
* Updated `src/gepa/adapters/dspy_full_program_adapter/full_program_adapter.py`.

### What I Changed

For issue #97, I changed `make_reflective_dataset` so that when a normal cumulative trace has no `FailedPrediction`, GEPA keeps only the final trace entry instead of sending every cumulative entry into the reflection dataset.

This avoids repeated earlier ReAct context while preserving the existing behavior for failed predictions.

### Code Changes

* Branch: https://github.com/Tirth2530/gepa/tree/fix-gepa-97-reflective-dataset
* Draft PR: https://github.com/gepa-ai/gepa/pull/383
* Implementation commit: `5df3b52` — Reduce redundant reflective trace context
* Test commit: `143033b` — Add regression test for reflective trace selection

### Testing Strategy

* Added a regression test confirming that only the final trace entry is included when no failure exists.
* Focused DSPy adapter tests: `8 passed`
* Full test suite: `484 passed, 3 skipped`
* Pre-commit checks passed for the changed files.


---

## Phase IV: Submit & Iterate

### Pull Request

* PR Link: https://github.com/gepa-ai/gepa/pull/383
* Status: Iterating

### Contribution Summary

I fixed GEPA issue #97 by reducing redundant cumulative trace context in `DspyAdapter.make_reflective_dataset`. When there is no `FailedPrediction`, the reflective dataset now keeps only the final trace entry rather than repeating every cumulative step.

I also added a regression test confirming that only the final trace entry is retained for normal cumulative traces.

### Validation

* Focused DSPy adapter tests: 8 passed
* Full GEPA test suite: 484 passed, 3 skipped
* Pre-commit checks passed

### Maintainer Feedback

* Lakshya A. Agrawal asked me to remove the standalone `reproduce_gepa_97.py` helper script. I removed it in commit `2bff64f`.
* Lakshya asked whether the behavior was limited to ReAct. I explained that the issue can affect DSPy programs with cumulative trace entries, while ReAct makes the duplication especially visible.
* Current next step: awaiting additional maintainer feedback.

### Week 5 Status: Phase IV / Iterating
PR #383 is open. Initial maintainer feedback has been addressed. Currently waiting for further review.

## Week 6 Check-In

### Current Status

I am continuing Phase IV / Iterating for GEPA PR #383.

The PR is open and under review. I previously addressed initial maintainer feedback by removing the standalone `reproduce_gepa_97.py` helper script and responded to the reviewer’s question about whether the issue is specific to ReAct or applies more broadly to DSPy programs with cumulative traces.

### Next Steps

I am waiting for additional maintainer feedback. If another review comment is posted, I will respond and make any requested changes.

