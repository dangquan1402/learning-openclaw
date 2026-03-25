# Repository Guidelines

## Project Structure & Module Organization
This repository is a Markdown knowledge base for learning OpenClaw. The root [README.md](/Users/quandang/Desktop/dquan/learning-openclaw/README.md) is the index and learning path. All content lives in [`notes/`](/Users/quandang/Desktop/dquan/learning-openclaw/notes), with numbered files that group topics by sequence and scope, for example `02-architecture.md`, `02a-integration-protocol.md`, and `08a-goclaw-channels.md`. Add new research notes to `notes/` and update the README tables so navigation stays accurate.

## Build, Test, and Development Commands
There is no project build pipeline in this repository. Use lightweight validation commands before opening a PR:

```bash
git status
rg ']\((?!http)' README.md notes
markdownlint README.md notes/*.md
```

`git status` confirms only intended files changed. The `rg` check helps catch broken relative Markdown links. `markdownlint` is recommended if installed locally, but it is not enforced by repository config.

## Coding Style & Naming Conventions
Write in clear, compact Markdown with sentence-case prose and descriptive headings. Prefer short sections, tables for comparisons, and fenced code blocks with `bash` for commands. Keep filenames numeric and ordered by topic: main sections use `NN-name.md`, while subtopics use `NNa-name.md`. Use hyphenated lowercase names only. When editing, preserve the existing table and checklist style used in the README and notes.

## Testing Guidelines
Quality checks are manual. Verify that new notes:

- render cleanly in Markdown preview,
- use working relative links,
- keep numbering consistent with neighboring files,
- do not duplicate existing topics without a clear reason.

If a note includes commands or version claims, sanity-check them against the cited source before merging.

## Commit & Pull Request Guidelines
Follow the repository’s existing commit style: imperative, descriptive subjects such as `Add GoClaw 7 messaging channels setup guide`. Keep commits focused on one topic area. Pull requests should include a short summary, list affected note files, mention any README index updates, and call out source or fact changes that need extra review. Include screenshots only if the PR changes rendered formatting in a significant way.
