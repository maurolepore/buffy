---
title: Wiki Log
description: Append-only audit trail of wiki generation and refresh runs.
---

# Wiki Log

Append-only audit trail. Add one dated entry per generation or refresh run, recording the profile, the `source_commit` it was anchored to, and the coverage. The codebase-wiki skill describes the entry shape.

## 2026-10-24: generate

- Profile: public/standard
- source_commit: f357243 (initial generation; no previous stamp)
- Coverage: full codebase survey — Sinatra web app + webhook filter, responder framework (base class, registry, loader), built-in responders, Sidekiq workers, GitHub/Octokit integration + issue-body templating, paper/BibTeX/DOI checks, and the settings-YAML configuration model. All source references are GitHub blob URLs to the upstream repo.
- Pages: [Overview](./OVERVIEW.md), [System overview](./architecture/system-overview.md), [Responder framework](./architecture/responder-framework.md), [Background jobs](./architecture/background-jobs.md), [Web app](./modules/web-app.md), [Responder core](./modules/responder-core.md), [Responders catalog](./modules/responders-catalog.md), [Workers](./modules/workers.md), [GitHub integration](./modules/github-integration.md), [Paper and DOI checks](./modules/paper-doi.md), [Configuration](./modules/configuration.md), [Issue comment command](./flows/issue-comment-command.md), [New submission](./flows/new-submission.md), [Asynchronous repository check](./flows/async-repo-check.md), [Scheduled reminder](./flows/reminder.md), [Responder](./concepts/responder.md), [Buffy context](./concepts/buffy-context.md), [Issue body markers](./concepts/issue-body-markers.md), [External service](./concepts/external-service.md), [Teams and authorization](./concepts/teams-and-authorization.md), [Add a custom responder](./guides/add-a-responder.md), [Configure a command](./guides/configure-a-command.md), [Run and test locally](./guides/run-and-test-locally.md)