---
name: exa
description: >-
  Finds pages, reads them, and builds lists through the connected Exa account,
  including Similarweb traffic, Fiber companies and people, Baselayer business
  records, and Jinko fares. Use when a fact, page, source, or researched list
  is not already in the thread and is not a library manual or a named GitHub
  repository.
---

# Exa

The connected Exa account is the retrieval. Do not call the keyless `https://mcp.exa.ai/mcp` endpoint.

## Search

`web_search_exa` when the page is not already named: a current fact, a product, or something people published.

`web_search_advanced_exa` when the result has to stay inside a domain, a date range, a category, or a place. Ordinary lookups stay on `web_search_exa`.

A URL the user gave is **Fetch**. `web_fetch_exa` only when that Fetch came back empty or unreadable.

Library and framework APIs stay on context7. A named GitHub repository stays on the GitHub plugin.

## Research

`agent_run` when the task is a list, an enrichment of rows they already have, or more than one search. Pass `outputSchema` when the answer has to come back as fields. Pass `input.data` for rows they already named. It spends the connected account's usage. A single lookup stays on `web_search_exa`.

`dataSources` is Exa Connect: a partner database beside the web, at most five. Use one when the task needs that database, and name the field in the query and the schema. Self-serve providers include `fiber` (companies, people, jobs), `similarweb` (traffic), `financial_datasets` (US ticker news), `baselayer` (US business records), `affiliate` (product catalogs), `particle` (podcast transcripts), and `jinko` (travel fares).

If `agent_run` returns `status: running` and an id, call it again with that id as `runId`. Do not start a second research.
