---
name: exa
description: >-
  Searches the public web through the connected Exa account. Use when a fact,
  page, or source is not already in the thread and is not a library manual or
  a named GitHub repository.
---

# Exa

The connected Exa account is the search. Do not call the keyless `https://mcp.exa.ai/mcp` endpoint.

## Search

`web_search_exa` when the page is not already named: a current fact, a product, or something people published.

A URL the user gave is **Fetch**. `web_fetch_exa` only when that Fetch came back empty or unreadable.

Library and framework APIs stay on context7. A named GitHub repository stays on the GitHub plugin.

## Research

`agent_run` when the task is a list or more than one search. It spends the connected account's usage. A single lookup stays on `web_search_exa`.

If `agent_run` returns `status: running` and an id, call it again with that id as `runId`. Do not start a second research.
