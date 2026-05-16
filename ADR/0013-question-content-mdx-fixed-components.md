# ADR 0013: Question stem content as MDX with a fixed component set

## Status

Accepted

## Context

Question task bodies need richer formatting than plain text while staying **safe and predictable**. Letting authors paste arbitrary MDX but without their own `import` / `export` lines as it increases complexity, security risk, and support burden.

## Decision

- Treat **`QuestionLocalized.content`** (the task stem) as **MDX source**: Markdown plus JSX tags that map only to **application-defined** components.
- **Authors do not write import headers** (or export declarations). The runtime supplies a **closed `components` map** (same set everywhere: editor preview, app surfaces, any server render path).
- **Validation and persistence** must **reject or strip** author-supplied `import`/`export` and other disallowed constructs before save, so stored content stays within the supported subset.
- The **allowlisted component list** is versioned with the app; adding a component is a deliberate product/engineering change, not an author configuration.

## Relation to ADR 0012

[0012](0012-content-question-render-via-generator-grpc.md) keeps **HTTP access** to rendered questions via the Generator service. How MDX interacts with Generator (e.g. expand variables in MDX source before render, or evaluate MDX only on clients) is an **implementation detail** of the pipeline, but the **authoring contract** for the stem is MDX with fixed components as above.

## Consequences

- Editors and APIs document “MDX body, no imports” and enforce it.
- Security and UX improve: no arbitrary module loading from user text.
- New stem features ship by extending the shared component map and docs, not by authors importing modules.
