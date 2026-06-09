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

## Phase II: Reproduce and Plan

To be completed after Phase I.

---

## Phase III: Implementation and Testing

To be completed later.

---

## Phase IV: Pull Request and Maintainer Feedback

To be completed later.
