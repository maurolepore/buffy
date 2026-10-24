---
title: Configure a command
description: How to wire up or tune bot behavior in the settings YAML without writing code.
type: guide
tags:
  - wiki
  - guide
  - how-to
---
# Configure a command

## Goal

Change what a deployed Buffy bot does — enable responders, restrict them to teams, add label side effects, attach external calls — by editing `config/settings-<environment>.yml`. The full parameter reference is [docs/configuration.md](https://github.com/ropensci-org/buffy/blob/main/docs/configuration.md) and [docs/available_responders.md](https://github.com/ropensci-org/buffy/blob/main/docs/available_responders.md).

## Steps

1. **Pick the environment file** — `settings-production.yml` for the deployed bot (selected by `RACK_ENV`).
2. **Declare teams** under `teams:` if you will use `only:` restrictions; values can be numeric team IDs or `org/team-slug` strings (resolved at startup).
3. **Add the responder** under `responders:` by its keyname. Use the hash form for one instance with params, or the array form for multiple named instances of the same class (e.g. several `set_value` responders).
4. **Set common params**:
   - `only: <team>` or `only: [team1, team2]` — restrict to teams; `authorized_roles_in_issue: <field>` — restrict to users named in an issue-body field
   - `if:` — preconditions (`title`, `body`, `value_exists`, `role_assigned`, `value_matches`, `labels`) with optional `reject_msg`
   - `add_labels:` / `remove_labels:` — label side effects
   - `template_file:` — reply with a template from the reviews repo's templates path
   - `external_call:` — invoke an [external service](../concepts/external-service.md) alongside the behavior
   - `hidden: true`, `description:`, `example_invocation:` — control `@bot help` output
5. **Keep secrets in environment variables** via ERB interpolation (`<%= ENV['BUFFY_GH_ACCESS_TOKEN'] %>`), never as literals — see [Configuration](../modules/configuration.md).
6. **Restart the processes** (web and worker) to pick up changes; workers re-read the file per job.

## Relevant code

- Settings loading: [app/buffy.rb](https://github.com/ropensci-org/buffy/blob/main/app/buffy.rb) (`config_file`), defaults: [app/lib/defaults.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/defaults.rb)
- Instantiation semantics: `load_responders!` in [app/lib/responder_registry.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder_registry.rb)
- Condition evaluation: `meet_conditions?` in [app/lib/responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder.rb)
- Minimal working example: [config/settings-test.yml](https://github.com/ropensci-org/buffy/blob/main/config/settings-test.yml)

## Gotchas

- A responder key that doesn't match any class's `keyname` fails loudly at boot (`NoMethodError` on nil) — check spelling against the [catalog](../modules/responders-catalog.md).
- Team names are resolved through the GitHub API at startup; an unreachable org raises a configuration error before the app serves requests.
- `required_params` in the responder class raise on missing config values — the error names the class and the missing key.

## Related

- [Configuration module](../modules/configuration.md)
- [Teams and authorization](../concepts/teams-and-authorization.md)
- [Add a custom responder](add-a-responder.md)