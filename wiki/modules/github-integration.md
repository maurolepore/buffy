---
title: GitHub integration
description: The Octokit wrapper, issue-body actions, and message templating shared by responders and workers.
type: module
tags:
  - wiki
  - module
---
# GitHub integration

## Summary

GitHub integration is the set of mixins that give responders and workers their powers: `GitHub` wraps the Octokit API client, `Actions` provides higher-level operations on issues (including reading and writing [issue body markers](../concepts/issue-body-markers.md)), and `Templating` renders reply messages from local ERB files or templates stored in the reviews repository.

## Responsibilities

- Maintain one memoized, authenticated `Octokit::Client` per responder/worker instance
- Expose one Ruby method per GitHub operation used by the bot: comment, update comment, react, label/unlabel, update/close issue, manage assignees and collaborators, team membership and invitations, workflow dispatch
- Read and write named values embedded in the issue body between `<!--name-->` / `<!--end-name-->` markers
- Render replies from bundled ERB templates or from template files fetched from the target repository

## Public API / entry points

- `GitHub`: `github_client`, `issue`, `bg_respond`, `label_issue` / `unlabel_issue` / `issue_labels`, `update_comment`, `react_to_comment`, `update_issue` / `close_issue`, `add_collaborator` / `remove_collaborator` / `is_collaborator?`, `add_assignee` / `remove_assignee`, `api_team_id` / `api_team_members` / `invite_user_to_team`, `trigger_workflow`, `user_in_authorized_teams?`, plus URL helpers and the `get_team_ids` class method used at config load
- `Actions`: `respond` (post a comment), `update_body` / `append_to_body` / `prepend_to_body` / `new_body` / `delete_from_body`, `read_value_from_body` / `read_values_from_body` / `update_value` / `update_or_add_value` / `update_list`, `invite_user`, `replace_assignee`
- `Templating`: `respond_template` (bundled `.erb` from [app/responses/](https://github.com/ropensci-org/buffy/blob/main/app/responses)), `respond_external_template` / `render_external_template` (repo-hosted templates with `{{placeholder}}` substitution)

## Key files

- [app/lib/github.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/github.rb) — the Octokit wrapper; also uses Faraday directly for org invitations and workflow dispatches
- [app/lib/actions.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/actions.rb) — issue-body editing and value helpers
- [app/lib/templating.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/templating.rb) — ERB and external-template rendering
- [app/responses/help.erb](https://github.com/ropensci-org/buffy/blob/main/app/responses/help.erb), [app/responses/doi_checks.erb](https://github.com/ropensci-org/buffy/blob/main/app/responses/doi_checks.erb), [app/responses/list_team_members.erb](https://github.com/ropensci-org/buffy/blob/main/app/responses/list_team_members.erb) — the bundled reply templates

## Dependencies

- Gems: `octokit`, `faraday`, `erb`
- [Configuration](configuration.md) — the access token comes from `env.gh_access_token` (default: the `BUFFY_GH_ACCESS_TOKEN` environment variable)

## Participates in

- Every flow: [Issue comment command](../flows/issue-comment-command.md), [New submission](../flows/new-submission.md), [Asynchronous repository check](../flows/async-repo-check.md), [Scheduled reminder](../flows/reminder.md)

## Related

- [Issue body markers](../concepts/issue-body-markers.md)
- [External service](../concepts/external-service.md)
- [Responder core](responder-core.md) — where these mixins are included