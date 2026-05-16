# Question component

## Overview

Questions are the **RAW data** that the **[Generator](Generator.md)** turns into questions for the exams (variants, languages, scored answers). The **task stem** (`content`) is **MDX** rendered with a **fixed set of components** supplied by the app; authors do not add `import` / `export` lines. They support closed and open-ended styles, **parametric** fields, and **per-locale** text.

> **Categories** are hierarchical; questions are linked to **one or more** categories via a junction table. **[Tags](Tags.md)** add a second, non-hierarchical labelling axis (when implemented server-side).

## Purpose

- **Storage**: Question row + **localized** rows (title, description, content as MDX, variables, answers JSON, etc.).
- **Parametric generation**: Template variables and generator-compatible answer structures (see [ADR 0013](../../ADR/0013-question-content-mdx-fixed-components.md) for MDX stem rules).
- **Multi-language**: `QuestionLocalized`-style data per locale.
- **Answers**: Multiple options, correctness, shuffle/final flags on the base question where applicable.
- **Organization**: **Categories** (tree) and **tags** (flat / groups) for filters and exams.
- **Visibility**: `public` questions are readable without a personal grant; others need ownership or a row in `questions_permissions`.
- **Collaboration**: Grants are VIEW, EDIT, or MANAGE per user in `questions_permissions`. Each response includes `permission` (`QuestionPermissionBit`); create/update/delete routes require EDIT or MANAGE. See [ADR 0008](../../ADR/0008-per-resource-permission-tables.md).
