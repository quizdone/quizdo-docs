# Tags

[Backend](../../backend/content/tags/README.md) · [Frontend](../../frontend/README.md)

## Overview

**Tags** are an optional, **non-hierarchical** way to label questions and quizzes, **alongside [categories](Category.md)**.

| | **Categories** | **Tags** |
|---|----------------|----------|
| Structure | Tree (subject → field → custom topics), `depth` + `parent_id` | Flat labels; **tag groups** (e.g. difficulty, school type, question type, etc.) |
| Typical use | Curriculum placement, browsing the library | Cross-cutting filters, metadata, exam query axes |

> Unlike categories, tags are not hierarchical. One question can use **multiple categories** and **multiple tags** (groups may define rules: single vs multiple selection, values, etc.).

## Purpose

- **Flexible labelling**: Filter and sort without forcing a single tree path.
- **Tag groups** (product): Groups can constrain selection (single vs multi), values, or aggregations (e.g. avg/min/max for numeric tags).
- **Exam queries**: Exams can select questions by **tags** and **tag group values** (see [Exam](../../backend/content/exam/README.md) query model).
- **Access control**: Grants are stored on the **tag group** (`tag_groups_permissions`); tags inherit effective rights through their group. API messages expose `permission` (`TagPermissionBit` / `TagGroupPermissionBit`). See [ADR 0008](../../ADR/0008-per-resource-permission-tables.md).
