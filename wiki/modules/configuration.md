---
title: Configuration
description: The settings YAML file — env, teams, and the responders node that defines a bot's behavior.
type: module
tags:
  - wiki
  - module
---
# Configuration

## Summary

A Buffy bot is defined almost entirely by one YAML file: `config/settings-<environment>.yml`, selected by `RACK_ENV` and loaded with `sinatra/config_file` (with ERB interpolation, so secrets can come from environment variables). The file declares the bot's environment settings, its GitHub teams, and which responders are active with what parameters. The operator-facing reference is [docs/configuration.md](https://github.com/ropensci-org/buffy/blob/main/docs/configuration.md); this page covers how the code consumes it.

## Responsibilities

- Provide `settings.buffy` to the app: the `env` node (bot username, access token, webhook secret, templates path), the `teams` node, and the `responders` node
- Resolve team names (`org/team-slug`) to GitHub team IDs at startup
- Instantiate one responder object per entry under `responders:`, including multiple named instances of the same responder (array form)
- Supply defaults from environment variables when keys are absent

## Public API / entry points

The settings shape (see [config/settings-test.yml](https://github.com/ropensci-org/buffy/blob/main/config/settings-test.yml) for a minimal example):

```yaml
buffy:
  env:
    bot_github_user: <%= ENV['BUFFY_BOT_GH_USER'] %>
    gh_access_token: <%= ENV['BUFFY_GH_ACCESS_TOKEN'] %>
    gh_secret_token: <%= ENV['BUFFY_GH_SECRET_TOKEN'] %>
  teams:
    editors: 1234567            # numeric id, or 'org/team-slug'
  responders:
    hello:                      # single instance, no params
    assign_editor:
      only: editors             # params hash form
    set_value:
      - version:                # array form: named instances
          only: editors
```

- `env` keys fall back to `Defaults#default_settings` ([app/lib/defaults.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/defaults.rb)): `BUFFY_BOT_GH_USER`, `BUFFY_GH_ACCESS_TOKEN`, `BUFFY_GH_SECRET_TOKEN`, and a default `templates_path` of `.buffy/templates`
- Per-responder params are an open hash consumed by each responder class (`command`, `only`, `if`, `add_labels`, `template_file`, `external_call`, ...); see [Configure a command](../guides/configure-a-command.md)

## Key files

- [config/settings-test.yml](https://github.com/ropensci-org/buffy/blob/main/config/settings-test.yml), [config/settings-development.yml](https://github.com/ropensci-org/buffy/blob/main/config/settings-development.yml), [config/settings-production.yml](https://github.com/ropensci-org/buffy/blob/main/config/settings-production.yml) — per-environment settings
- [app/buffy.rb](https://github.com/ropensci-org/buffy/blob/main/app/buffy.rb) — `config_file` wiring
- [app/lib/defaults.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/defaults.rb) — environment-variable defaults
- [app/lib/responder_registry.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder_registry.rb) — `load_responders!` and team-ID resolution via `Responder.get_team_ids`
- [Gemfile_custom](https://github.com/ropensci-org/buffy/blob/main/Gemfile_custom) — extra gems a deployment can add (e.g. for custom responders), evaluated from the main [Gemfile](https://github.com/ropensci-org/buffy/blob/main/Gemfile)

## Dependencies

- `sinatra-contrib`'s `config_file`, `yaml`, `erb`
- GitHub API access at boot time when teams are configured by name (to resolve IDs)

## Participates in

Configuration is read at the start of every flow; it is the input to [Responder framework](../architecture/responder-framework.md) instantiation and is re-read by [workers](workers.md) per job.

## Related

- [Teams and authorization](../concepts/teams-and-authorization.md)
- [Responder](../concepts/responder.md)
- [Configure a command](../guides/configure-a-command.md)
- [Add a custom responder](../guides/add-a-responder.md)