---
title: Run and test locally
description: How to boot Buffy locally and run the RSpec suite.
type: guide
tags:
  - wiki
  - guide
  - how-to
---
# Run and test locally

## Goal

Run the Buffy web app on a development machine and execute the test suite. Installation details (Ruby version, Redis, environment variables) are covered by [docs/installation.md](https://github.com/ropensci-org/buffy/blob/main/docs/installation.md); this guide orients you in the code involved.

## Steps

1. **Install dependencies**: `bundle install` (Ruby per [.ruby-version](https://github.com/ropensci-org/buffy/blob/main/.ruby-version)). A local Redis is required for the worker side.
2. **Set environment variables**: `BUFFY_BOT_GH_USER`, `BUFFY_GH_ACCESS_TOKEN`, `BUFFY_GH_SECRET_TOKEN` — or edit `config/settings-development.yml`. Never commit real tokens.
3. **Boot the web process**: `bundle exec puma -C ./puma-config.rb` (or `bundle exec rackup`); the app serves on the configured port with `GET /status` as a liveness check.
4. **Boot the worker process** (optional, for background jobs): `bundle exec sidekiq -r ./app/lib/workers.rb`.
5. **Run the tests**: `bundle exec rspec spec` — the same command CI runs on multiple Ruby versions ([.github/workflows/tests.yml](https://github.com/ropensci-org/buffy/blob/main/.github/workflows/tests.yml)).

## Relevant code

- Test harness: [spec/spec_helper.rb](https://github.com/ropensci-org/buffy/blob/main/spec/spec_helper.rb) — Rack::Test for HTTP, WebMock with all network access disabled, `Sidekiq.testing!(:fake)` so jobs are enqueued but not executed, `RACK_ENV=test` (loading [config/settings-test.yml](https://github.com/ropensci-org/buffy/blob/main/config/settings-test.yml))
- Fixtures and helpers: [spec/support/](https://github.com/ropensci-org/buffy/blob/main/spec/support) — webhook payload fixtures (`json_fixture`) used to simulate GitHub events
- Suite layout: top-level specs per lib module plus [spec/responders/](https://github.com/ropensci-org/buffy/blob/main/spec/responders) and [spec/workers/](https://github.com/ropensci-org/buffy/blob/main/spec/workers)
- Process definitions: [Procfile](https://github.com/ropensci-org/buffy/blob/main/Procfile), [puma-config.rb](https://github.com/ropensci-org/buffy/blob/main/puma-config.rb), [config.ru](https://github.com/ropensci-org/buffy/blob/main/config.ru)

## Gotchas

- Tests stub all HTTP: if a spec needs a GitHub API response, add a WebMock stub — real network calls fail by design.
- `Sidekiq.testing!(:fake)` means worker `perform` bodies don't run unless the spec invokes them directly; check the existing worker specs for the pattern.
- The test settings file uses placeholder credentials; specs that hit authorization logic stub the team-membership API calls.

## Related

- [Web app](../modules/web-app.md)
- [Workers](../modules/workers.md)
- [Add a custom responder](add-a-responder.md)