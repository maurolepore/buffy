---
title: Web app
description: The Sinatra application and GitHub webhook filter — Buffy's HTTP boundary.
type: module
tags:
  - wiki
  - module
---
# Web app

## Summary

The web app is the thinnest possible HTTP layer: a [Sinatra](https://github.com/ropensci-org/buffy/blob/main/app/buffy.rb) application that receives GitHub webhooks, verifies and parses them, and hands the resulting message and context to the [responder framework](../architecture/responder-framework.md). It knows nothing about individual bot commands.

## Responsibilities

- Serve `POST /dispatch`, the single webhook entry point
- Verify webhook signatures and filter events before any bot logic runs
- Load the per-environment settings file (`config/settings-<environment>.yml`)
- Expose `/status` (reports the bot user and environment) and `/` (a health check) for monitoring

## Public API / entry points

- `POST /dispatch` — receives `issues` and `issue_comment` webhook events; calls `responders.respond(@message, @context)` and returns `200 Message processed`
- `GET /status` — plain-text liveness with the configured bot user
- `GET /` — plain-text health check
- Boot: [config.ru](https://github.com/ropensci-org/buffy/blob/main/config.ru) requires `app/buffy.rb` and runs the `Buffy` class under Puma

## Key files

- [app/buffy.rb](https://github.com/ropensci-org/buffy/blob/main/app/buffy.rb) — the `Buffy < Sinatra::Base` class; registers `Sinatra::ConfigFile` and `GitHubWebhookFilter`, includes `RespondersLoader`
- [app/sinatra_ext/github_webhook_filter.rb](https://github.com/ropensci-org/buffy/blob/main/app/sinatra_ext/github_webhook_filter.rb) — a Sinatra extension with a `before '/dispatch'` filter that uses the `issue` gem's `Issue::Webhook` to verify the secret token, discard events sent by the bot itself (for `issue_comment`), accept only `issues`/`issue_comment` events, and set `@context` and `@message` (comment body, falling back to issue body)
- [config.ru](https://github.com/ropensci-org/buffy/blob/main/config.ru), [puma-config.rb](https://github.com/ropensci-org/buffy/blob/main/puma-config.rb), [Procfile](https://github.com/ropensci-org/buffy/blob/main/Procfile) — rackup, Puma threads/port config, and the `web` process definition

## Dependencies

- Gems: `sinatra` + `sinatra-contrib` (config_file), `issue` (webhook parsing/verification), `puma`
- [Responder core](responder-core.md) via `RespondersLoader`
- [Configuration](configuration.md) for `settings.buffy[:env]` values used by the filter

## Participates in

- [Issue comment command](../flows/issue-comment-command.md)
- [New submission](../flows/new-submission.md)

## Related

- [System overview](../architecture/system-overview.md)
- [Buffy context](../concepts/buffy-context.md) — what the filter builds