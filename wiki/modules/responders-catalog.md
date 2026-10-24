---
title: Responders catalog
description: The built-in responders under app/responders/ — what each one does and listens for.
type: module
tags:
  - wiki
  - module
---
# Responders catalog

## Summary

This module is the set of built-in [Responder](../concepts/responder.md) subclasses in [app/responders/](https://github.com/ropensci-org/buffy/blob/main/app/responders). Each file defines one behavior, registered under its `keyname` and activated when listed in the settings file's `responders:` node. The authoritative user-facing descriptions live in the repo docs ([docs/available_responders.md](https://github.com/ropensci-org/buffy/blob/main/docs/available_responders.md)); this page maps the code.

## Responsibilities

Grouped by purpose (keyname in parentheses):

- **Conversation**: `HelloResponder` (`hello`) and `ThanksResponder` (`thanks`) answer greetings and thanks; `HelpResponder` (`help`) lists the currently active commands; `WrongCommandResponder` (`wrong_command`) replies to unrecognized commands.
- **Lifecycle messages**: `WelcomeResponder` (`welcome`) fires on `issues.opened`; `GoodbyeResponder` (`goodbye`) on `issues.closed`; `InitialValuesResponder` (`initial_values`) seeds issue-body values when an issue opens.
- **Editorial roles**: `AssignEditorResponder` (`assign_editor`) and `RemoveEditorResponder` (`remove_editor`) manage the `editor` value, collaborators, and assignees; `ReviewersListResponder` (`reviewers_list`) maintains the `reviewers-list`; `AddAndRemoveAssigneeResponder` (`add_remove_assignee`) manages issue assignees; `InviteResponder` (`invite`) sends repo-collaborator invitations.
- **Issue-body state**: `SetValueResponder` (`set_value`) writes a named value; `ListOfValuesResponder` (`list_of_values`) adds/removes entries in a named list; both edit [issue body markers](../concepts/issue-body-markers.md).
- **Configurable commands**: `BasicCommandResponder` (`basic_command`) replies to a configured command; `LabelCommandResponder` (`label_command`) only changes labels; `CloseIssueCommandResponder` (`close_issue_command`) closes the issue.
- **Checks and integrations**: `CheckReferencesResponder` (`check_references`) enqueues [DOI checks](paper-doi.md); `RepoChecksResponder` (`repo_checks`) enqueues repository checks; `ExternalServiceResponder` (`external_service`) and `ExternalStartReviewResponder` (`external_start_review`) call [external services](../concepts/external-service.md); `GithubActionResponder` (`github_action`) triggers a workflow dispatch.
- **Review workflow**: `RemindersResponder` (`reminders`) schedules reminders; `ReviewerChecklistCommentResponder` (`reviewer_checklist_comment`) updates a checklist comment; `UpdateCommentResponder` (`update_comment`) edits an existing comment; `ListTeamMembersResponder` (`list_team_members`) posts team rosters; `AddAndRemoveUserChecklistResponder` (`add_remove_checklist`) manages per-user checklists.

## Public API / entry points

Each subclass implements the framework contract from [Responder core](responder-core.md): `keyname`, `define_listening` (event + regex), `process_message`, plus `default_description` / `default_example_invocation` used by `help`. Comment commands take the form `@<bot-name> <verb> ...`, matched case-insensitively by each responder's `event_regex`.

## Key files

- [app/responders/basic_command_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/basic_command_responder.rb) — the simplest useful responder and a good reading starting point
- [app/responders/welcome_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/welcome_responder.rb) — the richest event responder: messages, templates, external services, DOI/repo checks, chained responders, optional issue close
- [app/responders/assign_editor_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/assign_editor_responder.rb), [app/responders/reviewers_list_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/reviewers_list_responder.rb) — role management and issue-body state edits
- [app/responders/help_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/help_responder.rb) — builds the help reply from the live registry
- [app/responders/reminders_responder.rb](https://github.com/ropensci-org/buffy/blob/main/app/responders/reminders_responder.rb) — schedules [workers](workers.md) with `chronic`-parsed times

## Dependencies

- [Responder core](responder-core.md) — the base class and registry
- [Workers](workers.md) — `WelcomeResponder`, `CheckReferencesResponder`, `RepoChecksResponder`, `RemindersResponder`, and the external-service responders enqueue Sidekiq jobs
- [GitHub integration](github-integration.md) and [Paper and DOI checks](paper-doi.md) (indirectly, via workers)

## Participates in

- [Issue comment command](../flows/issue-comment-command.md)
- [New submission](../flows/new-submission.md)
- [Asynchronous repository check](../flows/async-repo-check.md)
- [Scheduled reminder](../flows/reminder.md)

## Related

- [Configure a command](../guides/configure-a-command.md)
- [Add a custom responder](../guides/add-a-responder.md)