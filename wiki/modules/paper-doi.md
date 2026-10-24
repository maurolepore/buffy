---
title: Paper and DOI checks
description: BibTeX parsing, DOI validation against doi.org and Crossref, and local repository utilities.
type: module
tags:
  - wiki
  - module
---
# Paper and DOI checks

## Summary

This module supports Buffy's paper-review checks: finding and parsing a submission's `paper.md`/`paper.tex` and its BibTeX bibliography, validating each DOI, and suggesting candidate DOIs from Crossref when entries lack one. It also holds the shell utilities for cloning repositories safely. Everything here is used by the [workers](workers.md), not by the web process directly.

## Responsibilities

- Locate the paper file in a cloned repository and resolve its bibliography from the paper's metadata
- Parse BibTeX (preserving raw DOI fields alongside LaTeX-filtered parsing) and surface parse errors
- Validate DOIs: reject malformed or `doi.org`-prefixed strings, verify resolvability with an HTTP HEAD to doi.org, special-case the ACM `10.5555` handle prefix
- For entries without a DOI, query Crossref (via `serrano`) and suggest a candidate when titles match closely (Levenshtein distance < 3)
- Clone git repositories and switch branches with input validation

## Public API / entry points

- `PaperFile.find(search_path)` — locate `paper.md`/`paper.tex`; `#bibtex_entries`, `#bibtex_error`, `#text`
- `DOIChecker.new(entries).check_dois` — returns a `{ok:, skip:, missing:, invalid:}` summary consumed by the [doi_checks.erb](https://github.com/ropensci-org/buffy/blob/main/app/responses/doi_checks.erb) template
- `Utilities`: `clone_repo(url, path)`, `change_branch(branch, path)`, `run_cloc(path)`

## Key files

- [app/lib/paper_file.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/paper_file.rb) — `PaperFile`: paper discovery, bibliography resolution, BibTeX parsing
- [app/lib/doi_checker.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/doi_checker.rb) — `DOIChecker`: validation, Crossref lookup, string similarity
- [app/lib/utilities.rb](https://github.com/ropensci-org/buffy/blob/main/app/lib/utilities.rb) — git clone/branch helpers with URL and branch-name validation (`SAFE_BRANCH_NAME`) and a `cloc` wrapper

## Dependencies

- Gems: `bibtex-ruby`, `serrano` (Crossref API), `faraday`
- External tools: `git`, `cloc` (used by `RepoChecksWorker`)
- Called from `DOIWorker` and `RepoChecksWorker` in [Workers](workers.md)

## Participates in

- [Asynchronous repository check](../flows/async-repo-check.md)
- [New submission](../flows/new-submission.md) (when the welcome responder enables reference/repo checks)

## Related

- [Workers](workers.md)
- [Background jobs](../architecture/background-jobs.md)