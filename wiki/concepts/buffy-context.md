---
title: Buffy context
description: "The per-event data object every responder and worker receives: repo, issue, sender, action."
type: concept
tags:
  - wiki
  - concept
---
# Buffy context

## Definition

The context is the per-webhook-event data object passed to every responder. In the web process it is built by the webhook filter from the parsed payload (an object from the `issue` gem exposing `repo`, `issue_id`, `issue_body`, `issue_title`, `issue_author`, `issue_labels`, `sender`, `comment_body`, `comment_id`, and `event_action`). In workers it is rebuilt as an `OpenStruct` from the serialized job payload by `BuffyWorker#load_context_and_env`.

## Why it matters

The context is the seam between the GitHub world and the bot world: all GitHub API helpers (`GitHub` module) read `context.repo` and `context.issue_id`, and the responder pipeline branches on `context.event_action` and `context.sender`. Because workers rebuild the same shape from plain JSON, responder code and worker code share helpers without sharing memory.

## Where it lives

- Built in [app/sinatra_ext/github_webhook_filter.rb](https://github.com/ropensci-org/buffy/blob/main/app/sinatra_ext/github_webhook_filter.rb) (`@context = payload.context`)
- Rebuilt in [app/workers/buffy_worker.rb](https://github.com/ropensci-org/buffy/blob/main/app/workers/buffy_worker.rb)
- Consumed throughout [app/lib/github.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/github.rb) and [app/lib/responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder.rb) (`locals` exposes parts of it to templates)

## Related

- [Web app](../modules/web-app.md)
- [Responder](responder.md)
- [Background jobs](../architecture/background-jobs.md)