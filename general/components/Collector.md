# Collector component

## Overview

The **Collector** captures **live session** outcomes when users run **Questions**, **Quizzes/Exams**, or **Campaigns** in test or live mode. Raw data feeds **Evaluator** for grading and reports.

> The live-testing path persists **file-based logs** (JSON lines) consumed by the Evaluator service. See [Collector (backend)](../../backend/collector/README.md).

## Purpose

- Record answers, timings, and behaviour signals per session.
- Tie results to users/sessions for auditing.
- Decouple high-volume capture from evaluation UIs.
