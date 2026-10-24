---
title: External service
description: Declarative outbound HTTP calls from responders, executed asynchronously by ExternalServiceWorker.
type: concept
tags:
  - wiki
  - concept
---
# External service

## Definition

An external service is a configured HTTP call Buffy makes to a third-party system (for example, a journal's publishing pipeline). Responders declare it in their params — either as an `external_call` sub-config (fire-and-forget alongside other behavior) or as the whole behavior (`external_service` / `external_start_review` responders). The call itself is executed by `ExternalServiceWorker` in the background.

## Why it matters

This is how Buffy integrates with systems beyond GitHub without custom code: URL, method, headers, query parameters, and payload mapping from issue data are all YAML. On success the worker can post a configured message, render a template from the parsed JSON response, change labels, or close the issue; on failure it posts an error message (unless `silent`).

## Where it lives

- Enqueue helper: `Responder#process_external_service` in [app/lib/responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder.rb)
- Worker: [app/workers/external_service_worker.rb](https://github.com/ropensci-org/buffy/blob/main/app/workers/external_service_worker.rb) — method selection, `data_from_issue` / `mapping` parameter assembly, response handling
- Responders: [app/responders/external_service_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/external_service_responder.rb), [app/responders/external_start_review_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/external_start_review_responder.rb); also used by [welcome](../flows/new-submission.md) and `assign_editor` via `external_call`

## Related

- [Workers](../modules/workers.md)
- [Background jobs](../architecture/background-jobs.md)
- [Configure a command](../guides/configure-a-command.md)