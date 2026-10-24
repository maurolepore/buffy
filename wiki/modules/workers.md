---
title: Workers
description: The Sidekiq background jobs under app/workers/ — async checks, external calls, and reminders.
type: module
tags:
  - wiki
  - module
---
# Workers

## Summary

The workers module ([app/workers/](https://github.com/ropensci-org/buffy/blob/main/app/workers)) contains the Sidekiq jobs that run outside the web process. All inherit from `BuffyWorker`, which provides context/env loading, repository cloning, and the same GitHub/templating helpers responders use. Jobs are enqueued by responders with serialized payloads and report results back as issue comments. See [Background jobs](../architecture/background-jobs.md) for the design.

## Responsibilities

- **`BuffyWorker`** — base class: `load_context_and_env` (rebuild context from the payload, re-read settings YAML), `setup_local_repo` (clone + checkout branch into `tmp/<job-id>`), `cleanup`, retries disabled
- **`DOIWorker`** — clone the reviewed repo, locate `paper.md`/`paper.tex`, parse its BibTeX bibliography, validate DOIs, and post the summary (template: [app/responses/doi_checks.erb](https://github.com/ropensci-org/buffy/blob/main/app/responses/doi_checks.erb))
- **`RepoChecksWorker`** — clone the repo and run the configured checks: `cloc` summary, top-3 languages as labels (via github-linguist), paper wordcount, license detection (via licensee), statement-of-need presence
- **`ExternalServiceWorker`** — perform the configured HTTP call (GET/POST, headers, query params, issue-data mapping) and act on the response: success/error messages, external templates, labels, optional issue close
- **`ReviewReminderWorker`** — post a scheduled reminder to a reviewer or author, unless the issue closed meanwhile
- **`AsyncMessageWorker`** — post a scheduled plain message, unless the issue closed meanwhile

## Public API / entry points

Each worker implements `perform(...)` and is enqueued from responders via `perform_async` or `perform_at`. The worker process boots with `bundle exec sidekiq -r ./app/lib/workers.rb` (see the [Procfile](https://github.com/ropensci-org/buffy/blob/main/Procfile)).

## Key files

- [app/workers/buffy_worker.rb](https://github.com/ropensci-org/buffy/blob/main/app/workers/buffy_worker.rb) — the shared base class
- [app/workers/doi_worker.rb](https://github.com/ropensci-org/buffy/blob/main/app/workers/doi_worker.rb), [app/workers/repo_checks_worker.rb](https://github.com/ropensci-org/buffy/blob/main/app/workers/repo_checks_worker.rb) — the repository-inspection jobs
- [app/workers/external_service_worker.rb](https://github.com/ropensci-org/buffy/blob/main/app/workers/external_service_worker.rb) — generic outbound HTTP
- [app/workers/review_reminder_worker.rb](https://github.com/ropensci-org/buffy/blob/main/app/workers/review_reminder_worker.rb), [app/workers/async_message_worker.rb](https://github.com/ropensci-org/buffy/blob/main/app/workers/async_message_worker.rb) — scheduled messages
- [app/lib/workers.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/workers.rb) — requires all worker files for the Sidekiq process

## Dependencies

- Gems: `sidekiq`, `faraday` + `faraday-retry`, `rugged`, `github-linguist`, `licensee`
- [Paper and DOI checks](paper-doi.md) — `PaperFile`, `DOIChecker`, and the git/shell `Utilities`
- [GitHub integration](github-integration.md) — comment/label helpers via mixed-in modules
- A Redis instance for the Sidekiq queue

## Participates in

- [Asynchronous repository check](../flows/async-repo-check.md)
- [Scheduled reminder](../flows/reminder.md)
- [New submission](../flows/new-submission.md) (welcome-triggered checks)

## Related

- [Background jobs architecture](../architecture/background-jobs.md)
- [External service concept](../concepts/external-service.md)