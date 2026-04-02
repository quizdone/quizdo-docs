# Question component

## Overview

Questions are the **atomic content** the **[Generator](Generator.md)** turns into concrete items (variants, languages, scored answers). They support closed and open-ended styles, **parametric** fields, and **per-locale** text.

> **Categories** are hierarchical; questions are linked to **one or more** categories via a junction table. **[Tags](Tags.md)** add a second, non-hierarchical labelling axis (when implemented server-side).

## Purpose

- **Storage**: Question row + **localized** rows (title, description, content, variables, answers JSON, etc.).
- **Parametric generation**: Template variables and generator-compatible answer structures.
- **Multi-language**: `QuestionLocalized`-style data per locale.
- **Answers**: Multiple options, correctness, shuffle/final flags on the base question where applicable.
- **Organization**: **Categories** (tree) and **tags** (flat / groups) for filters and exams.
