---
title: Issue body markers
description: "Buffy's database: named values stored between HTML comment markers in the GitHub issue body."
type: concept
tags:
  - wiki
  - concept
---
# Issue body markers

## Definition

Buffy stores review state directly in the GitHub issue body, inside HTML comment markers: a value named `editor` lives between `<!--editor-->` and `<!--end-editor-->`. The `Actions` module provides the vocabulary: `read_value_from_body`, `read_values_from_body`, `update_value`, `update_or_add_value`, `update_list`, and `issue_body_has?`. Conventional fields include `editor`, `reviewers-list`, `target-repository`, `branch`, `version`, and `archive`.

## Why it matters

This is Buffy's only persistence mechanism — there is no database, which keeps instances stateless and makes the issue page itself the human-readable source of truth (markers are invisible when rendered). Authorization by role (`authorized_roles_in_issue`), responder preconditions (`if: value_exists` / `role_assigned` / `value_matches`), template `locals`, and worker payloads all read from these markers.

## Where it lives

- Read/write helpers: [app/lib/actions.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/actions.rb)
- Example writers: [app/responders/assign_editor_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/assign_editor_responder.rb) (`editor`), [app/responders/set_value_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/set_value_responder.rb), [app/responders/list_of_values_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/list_of_values_responder.rb), [app/responders/initial_values_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/initial_values_responder.rb)
- Example readers: [app/responders/reminders_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/reminders_responder.rb) (`reviewers-list`), check responders (`target-repository`, `branch`)

## Related

- [GitHub integration](../modules/github-integration.md) — the `Actions` module
- [Teams and authorization](teams-and-authorization.md) — role-based checks read markers
- [New submission](../flows/new-submission.md) — markers get seeded when the issue opens