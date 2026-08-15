## 题目元信息

- 来源：`self_built`
- 处理流程：`Codex Gold 修复`
- Bug 分类：`error`
- 复现稳定性：`stable`
- 难度：`advanced`
- 目标 Bug 数量：`1`
- 上游 Issue：无（0-1 自建项目）

# BUG_TASK

Source: self_built

## Title

Idempotent retry of a duplicate event returns HTTP 500 instead of success

## Category

error / wrong_result

## User-visible symptom

When a client reports the same job lifecycle event a second time (for example,
a network retry of an already-accepted report), the service responds with
`500 Internal Server Error` instead of an idempotent success. The duplicate is
still correctly deduplicated — no duplicate data is written — but the HTTP
status code is wrong, which triggers client-side alerts and continued retries.

## Expected behaviour

A duplicate (network-retried) event report must be recognised as an idempotent
success and return `202 Accepted`, never `500`. A genuine internal error must
still return a 5xx status, and the deduplication / persisted counters must stay
consistent with the data actually written.

## Reproduction command

```sh
go test -run TestIdempotentRetryIsNotServerError -count=1 ./internal/httpapi/
```

## Acceptance criteria

- A first report of an event is accepted (`202`) and the event is persisted.
- A second report of the same event returns `202` (idempotent success), never
  `500`, and does not write a duplicate record.
- Real internal errors still return a 5xx status.
- Deduplication and persisted counters remain consistent (a duplicate never
  increments the persisted count).
