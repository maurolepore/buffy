---
title: Teams and authorization
description: "Who may invoke what: GitHub teams via `only`, and issue roles via `authorized_roles_in_issue`."
type: concept
tags:
  - wiki
  - concept
---
# Teams and authorization

## Definition

Buffy authorizes commands on two axes, both declared per responder in the settings file. `only:` restricts a command to members of named GitHub teams (the `teams:` node maps names to team IDs or `org/team-slug` strings, resolved to IDs at startup). `authorized_roles_in_issue:` restricts it to users whose handles appear in given [issue body marker](issue-body-markers.md) fields — e.g. only the assigned editor. With neither key, a command is open to everyone.

## Why it matters

Editorial workflows have strict roles: editors-in-chief invite, editors assign reviewers, reviewers check off their own checklist items. The two-axis model lets configuration express both standing permissions (team membership) and per-submission permissions (the people named on this issue), and the denial reply (`authorized_teams_and_roles_sentence`) tells the sender exactly who is allowed.

## Where it lives

- Authorization logic: [app/lib/authorizations.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/authorizations.rb) and `Responder#authorized?` in [app/lib/responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/responder.rb)
- Team ID resolution at boot: `Responder.get_team_ids` in [app/lib/github.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/github.rb), called by `ResponderRegistry#load_responders!`
- Team config shape: [Configuration](../modules/configuration.md); a minimal example is [config/settings-test.yml](https://github.com/ropensci-org/buffy/blob/main/config/settings-test.yml)

## Related

- [Responder framework](../architecture/responder-framework.md) — where `authorized?` sits in the pipeline
- [Responder](responder.md)
- [Issue comment command](../flows/issue-comment-command.md)