---
title: Responder
description: "The unit of bot behavior: a class that listens for an event and message pattern and acts."
type: concept
tags:
  - wiki
  - concept
---
# Responder

## Definition

A responder is a Ruby class inheriting from `Responder` ([app/lib/responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder.rb)) that implements one bot behavior. It declares a `keyname` (its handle in the settings file), an `event_action` (which GitHub webhook action it listens to, e.g. `issue_comment.created`), an optional `event_regex` (the command syntax), and a `process_message` method (the behavior). Responders are instantiated from the `responders:` node of the [configuration](../modules/configuration.md) — one instance per entry, or several named instances via the array form.

## Why it matters

Responders are the entire extension model: a journal's bot is just a set of configured responders, and new behavior means writing a new subclass (or configuring `basic_command`). The base class supplies the dispatch pipeline (match → authorize → conditions → act), label side effects, template rendering, external-service calls, and responder chaining, so subclasses stay small.

## Where it lives

- Base class and pipeline: [app/lib/responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder.rb)
- Discovery and dispatch: [app/lib/responder_registry.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder_registry.rb)
- Built-in behaviors: [app/responders/](https://github.com/ropensci-org/buffy/blob/main/app/responders) — see the [Responders catalog](../modules/responders-catalog.md)
- Framework design: [Responder framework](../architecture/responder-framework.md); extension how-to: [Add a custom responder](../guides/add-a-responder.md)

## Related

- [Buffy context](buffy-context.md) — what a responder receives
- [Teams and authorization](teams-and-authorization.md)
- [Issue comment command](../flows/issue-comment-command.md)