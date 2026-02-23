# Collector Component

## Overview

Collector is responsible for collecting results from live testing. It gathers answers and outcome data when users take Quizzes, Campaigns, or individual Questions in a live or testing session. The collected data is used for evaluation and reporting.

> The **Live testing service** (backend) uses **file-based test logging**: it produces **text files** with test results and watched behaviors. These files are consumed by the Evaluator for listing results and generating reports. The Collector (e.g. HTTP/WebSocket or gRPC client) feeds data into the livetesting service, which persists it to disk.

## Purpose

- Capture answers and scores from live Quiz, Campaign, and Question sessions
- Associate results with specific sessions and users
- Persist raw results via file-based test logging (text files produced by the livetesting service) for later evaluation and analysis
- Allow Evaluator to show, evaluate, and report the results
