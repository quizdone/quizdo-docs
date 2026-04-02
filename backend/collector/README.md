# Collector service

> [**Collector**](../../general/components/Collector.md)

Backend service that runs live testing sessions (exams, campaigns), collects answers and watched behaviors, and **persists them as text files** (file-based test logging). No database — the [Evaluator service](../evaluator/README.md) consumes these files for listing, evaluation, and reports. Session metadata (user id, exam id, campaign id, etc.) is fetched from the Evaluator over HTTP when needed.

## Role

- HTTP API for planning and running live testing sessions.
- Plans and runs live testing sessions.
- Accept WebSocket connections for live sessions (links from Content API: [Exam livetest](../content/exam/README.md), [Campaign livetest](../content/campaign/README.md)), checks for session data in the Evaluator service database and starts the live session.
- Collect test results and behaviors; write to **per-session JSON lines files** in storage shared with Evaluator.
- Notify Evaluator when a session is finished (e.g. send result file path).
- **Logger design**: Single logger goroutine; handlers push parsed events into a channel (no file I/O in handlers). Logger routes to a per-connection/attempt file (deterministic name), keeps an open-handle cache with **idle timeout (LRU/TTL)** to avoid FD explosion, uses **bufio.Writer** and flushes periodically/on close.

## Related

- [**Content**](../content/README.md) — Provides livetest links for exam/campaign.
- [**Evaluator**](../evaluator/README.md) — Consumes log files; stores session metadata; lists results and reports.
- [**Collector** (app component)](../../general/components/Collector.md) — Conceptual overview.
