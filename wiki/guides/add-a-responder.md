---
title: Add a custom responder
description: How to implement a new bot command as a Responder subclass and enable it in configuration.
type: guide
tags:
  - wiki
  - guide
  - how-to
---
# Add a custom responder

## Goal

Add a new command (or event reaction) to a Buffy bot by writing a [Responder](../concepts/responder.md) subclass. The full tutorial with a worked example lives in the repo docs ([docs/custom_responder.md](https://github.com/ropensci-org/buffy/blob/main/docs/custom_responder.md)); this guide is the code-oriented summary.

## Steps

1. **Create the class file** in `app/responders/` (use a subfolder + module for organization-specific responders, e.g. `app/responders/myorg/clock_responder.rb` defining `Myorg::ClockResponder`). The file name maps to the class name via the registry's constantize convention.
2. **Subclass `Responder`** and require it: `require_relative '../lib/responder'`.
3. **Declare the keyname** — `keyname :clock`. This is the handle used in the settings file.
4. **Implement `define_listening`** — call `required_params` for any mandatory config keys, then set `@event_action` (e.g. `"issue_comment.created"` or `"issues.opened"`) and `@event_regex` (e.g. `/\A@#{bot_name} time\??\s*$/i`; `nil` means respond to every message for that event).
5. **Implement `process_message(message)`** — use the inherited helpers: `respond`, `update_value`, `label_issue`, `process_labeling`, `process_external_service`, and `@match_data` for regex captures.
6. **Add `default_description` and `default_example_invocation`** so `@bot help` documents the command; return `true` from `hidden?` to keep it out of help.
7. **Enable it** in `config/settings-<environment>.yml` under `responders:` with its keyname and params, then test (see [Run and test locally](run-and-test-locally.md)).

## Relevant code

- Base class contract: [app/lib/responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder.rb) (`define_listening`, `process_message`, `call`)
- Discovery/registration: [app/lib/responder_registry.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder_registry.rb) (`available_responders` requires every `app/responders/**/*.rb` file automatically)
- Minimal example: [app/responders/hello_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/hello_responder.rb); a param-driven example: [app/responders/basic_command_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/basic_command_responder.rb)
- Specs to imitate: [spec/responders/](https://github.com/ropensci-org/buffy/blob/main/spec/responders)

## Gotchas

- The class name must match the file path convention, or `available_responders` logs a NameError warning and the responder is unavailable.
- Command regexes must account for the bot mention (`@#{bot_name}`) and should anchor with `\A ... \z`-style boundaries to avoid partial matches.
- Anything slow (network, git) belongs in a [worker](../modules/workers.md), not in `process_message` — the webhook response should return immediately.
- If your responder needs extra gems, add them to [Gemfile_custom](https://github.com/ropensci-org/buffy/blob/main/Gemfile_custom), not the main Gemfile.

## Related

- [Responder framework](../architecture/responder-framework.md)
- [Configure a command](configure-a-command.md)
- [Responders catalog](../modules/responders-catalog.md)