# Evaluator component

## Overview

The **Evaluator** lists live-test **results**, shows item-level detail, supports **grading**, and can produce **reports**. It consumes **Collector** output (file-based logs) and may use the DB for in-progress evaluation state.

**Grading is not fully automatic by default.** Modes include:

- **Manual** — Teacher feedback and marks.
- **Automatic** — Against **predefined** correct/incorrect answers where the question allows it.
- **AI-assisted** — May suggest scores or comments; can be **flagged for manual review** so nothing is blindly finalized.

> **Evaluator** is an HTTP service in the backend. See [Evaluator (backend)](../../backend/evaluator/README.md).

## Purpose

- List sessions / results eligible for review.
- Show prompts, learner answers, and reference solutions where available.
- Apply evaluation policy (manual / rules / AI) and persist outcomes.
- Export or display **reports**.
