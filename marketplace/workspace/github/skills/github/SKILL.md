---
name: github
description: >-
  Reads a GitHub repository, code search, issues, and pull requests through
  the connected GitHub MCP. Use when the user shares a github.com URL, names
  a repository, or asks to look up code, issues, or pull requests on GitHub.
---

# GitHub

The connected GitHub MCP is the account. Do not Fetch `github.com` HTML to learn a repo. Do not shell out to `gh`.

## Read

A `github.com/owner/repo` link, a named repo, or a question about what a project does:

- `search_repositories` when the repo is not named yet.
- `get_file_contents` for the README and the files the task needs. A directory path lists that directory. Pass `ref` or `sha` when the link names a branch, tag, or commit.
- `search_code` with `repo:owner/repo` when the file is unknown. Add `path:`, `language:`, or `filename:` to narrow it. Code search needs the connected account.

Library and framework docs stay on context7. This skill is the repository.

## Write

An issue, pull request, fork, or push only when the user asked for that action.

- Open or update an issue with `issue_write`.
- Open a pull request with `create_pull_request`.
- Fork with `fork_repository`.
- Change files in a repo they own with `push_files`, on the branch they named. Do not push to the default branch unless they said to.

Do not open a pull request on someone else's repo. Do not call `create_pull_request_with_copilot` unless they asked.
