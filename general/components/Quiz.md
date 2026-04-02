# Quiz component

In the product language, a **Quiz** is an **exam**: a defined set of questions (or **queries** that select questions), layout/rendering options, and rules for live or static delivery.

The **backend** and **proto** use the name **Exam**; the **frontend** “Exams” area matches the **Questions** pattern (category finder + table).

## Overview

- **Exam** resource: list of questions and/or **query blocks** that describe how to pull questions (by question ids, **category**, **tags**, counts, filters). See [Exam (Content API)](../../backend/content/exam/README.md).
- **Generation**: Instances are produced with the **Generator** pipeline where parametric content applies.

## Purpose

- Aggregate questions from categories/tags and other filters.
- Support previews and **live testing** (Collector → Evaluator flow).
- Reuse templates to generate multiple instances (randomization, counts), subject to product rules.
