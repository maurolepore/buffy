---
title: Responder core
description: The Responder base class, registry, and loader — the framework every behavior builds on.
type: module
tags:
  - wiki
  - module
---
# Responder core

## Summary

Responder core is the framework module: the `Responder` base class every behavior inherits from, the `ResponderRegistry` that builds and dispatches to configured instances, and the `RespondersLoader` mixin that connects the registry to the Sinatra app. Supporting mixins provide authorization, defaults, and logging.

## Responsibilities

- Define the dispatch pipeline: event match → message match → authorization → conditions → `process_message`
- Discover responder classes in `app/responders/` and map their `keyname`s to config keys
- Instantiate responders from the `responders:` node of the settings file, including multiple named instances of the same class
- Provide shared helpers to all responders: issue actions, GitHub API access, templating, authorization checks
- Reply to unrecognized commands via `WrongCommandResponder`

## Public API / entry points

- `Responder#call(message, buffy_context)` — the pipeline entry point, invoked by the registry for each responder
- `Responder#define_listening` / `#process_message` — the two methods subclasses override
- `Responder.keyname(key)` — class-level declaration of the config key
- `Responder#required_params(*names)` — declares mandatory config params; raises on missing values and defines reader methods
- `ResponderRegistry#respond(message, context)` — fans a message out to all configured responders
- `ResponderRegistry.get_responder(config, key, name)` — builds a single configured responder, used by `process_other_responder`

## Key files

- [app/lib/responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder.rb) — base class: pipeline, conditions (`meet_conditions?`), label side effects (`process_labeling` / `process_reverse_labeling`), `locals` for templates, `process_external_service`, `process_other_responder`, `serializable`
- [app/lib/responder_registry.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder_registry.rb) — class discovery (`available_responders`), config instantiation (`load_responders!`), wrong-command fallback
- [app/lib/responders_loader.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responders_loader.rb) — memoized `responders` helper mixed into the Sinatra app
- [app/lib/authorizations.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/authorizations.rb) — resolves `only:` teams and `authorized_roles_in_issue:` into user/team checks
- [app/lib/defaults.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/defaults.rb) — default env settings read from `BUFFY_*` environment variables
- [app/lib/logging.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/logging.rb) — shared `Logger.new(STDOUT)` helper

## Dependencies

- [GitHub integration](github-integration.md) (`Actions`, `GitHub`, `Templating` modules are mixed into `Responder`)
- [Configuration](configuration.md) — the registry consumes `settings.buffy`
- `sinatra` (IndifferentHash), the `issue` gem's context objects

## Participates in

- [Issue comment command](../flows/issue-comment-command.md)
- [New submission](../flows/new-submission.md)

## Related

- [Responder framework architecture](../architecture/responder-framework.md) — the design story
- [Responder concept](../concepts/responder.md)
- [Responders catalog](responders-catalog.md)
- [Teams and authorization](../concepts/teams-and-authorization.md)