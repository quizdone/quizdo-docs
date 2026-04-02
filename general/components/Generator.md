# Generator component

## Overview

The **Generator** is the **Go library and CLI** ([pisemkomat/generator](https://gitlab.com/pisemkomat/generator)) that turns **parametric** question (and exam) definitions into concrete rendered content and answer sets.

In micros, a **dedicated gRPC service** is planned to wrap the library, **hash** outputs, and **cache** or **store** generations for reuse (answer checking, retries, evaluation). See [Generator (backend)](../../backend/generator/README.md).

## Purpose

- Validate and render templates (variables, math, answer expansion).
- Produce stable **hashes** for idempotent retrieval of a generated artefact.
- Integrate with Content and Evaluator without duplicating generation logic in every service.

## Open source

The upstream generator library is **GPL-3.0**; see the external repository for licence and CLI usage.
