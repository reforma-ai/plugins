# Icons8 marketplace plugin

## Question

How does a Reforma marketplace MCP plugin get its manifest, logo, and preview-plate color, and what does the official Icons8 MCP actually expose (endpoint, transport, auth, tools, local vs remote) that such a plugin would have to match?

## Scope

- Code: `/Users/bionic/WORK/Reforma/plugins` only. The reforma app repo was not opened. `README.md` points at `https://docs.reforma.ai/integrations`, not at that repo.
- External: Icons8 MCP primary pages, fetched 2026-11-18. Context7 has the Icons8 REST API (`/websites/developers_icons8`), not an MCP library. That query returned icon/search/photo HTTP APIs with an `Api-Key` header. It is not the MCP server.

## Summary

A catalog plugin is a `plugin.json` plus optional `mcp.json`, listed under a shelf in `marketplace.json`. The preview box background in this repo is a hex plate, `extensions.reforma.interface.brandColor`, not an image. Logos are files under `assets/` (`svg`, `png`, `webp`, `jpg`), published to the CDN only when `S3_URL` is set.

Icons8’s server is remote Streamable HTTP at `https://mcp.icons8.com/mcp/`. There is no local package. Auth is OAuth (scope `icons8`) or `Authorization: Bearer` with a personal API key. Thirteen tools cover icons, Ouch illustrations, and `account_info`. Free PNG needs a link back to Icons8. Paid SVG, Lottie, and most illustration downloads need a plan with MCP access and are capped at 100/hour and 1000/day.

The site header mark is `https://goodies.icons8.com/web/common/header/logo/logo-icons8.svg` (14×14, fill `#1fb141`). No brand-guidelines page with a stated hex was found.

## Topic pages

- [plugin-shape.md](plugin-shape.md) — manifest, category, MCP file, logos, `brandColor`, pack, tests
- [icons8-mcp.md](icons8-mcp.md) — official server, tools, auth, license, brand assets

## Important files and sources

- `README.md` — catalog layout and pack steps
- `marketplace.json` — shelf membership; `parseMarketplace` in `scripts/pack/shared.ts`
- `scripts/pack/index.ts` — `packPlugin` order
- `scripts/pack/logos.ts` — `normalizePluginLogos`, `applyCatalogOverlay`
- `scripts/pack/publish-logos.ts` — `publishCatalogLogosFromEnv`
- `https://docs.reforma.ai/integrations/manifest` and `/mcp` — plate color and MCP auth (fetched 2026-11-18)
- `https://developers.icons8.com/docs/mcp`
- `https://github.com/icons8/icons8-mcp` README
- `https://mcp.icons8.com/llms.txt` and `https://mcp.icons8.com/auth.md`
- `https://icons8.com/license`

## Open questions

- Exact paid plan names and prices. The README links `https://icons8.com/pricing` and says “a plan with MCP access.” That page was not read.
- Whether a paid Icons8 plan drops the attribution link. Only the free-PNG rule was stated.
- Tool JSON schemas (`inputSchema`). Names and behavior come from `llms.txt` and the README, not from a live `tools/list`.
- Whether Reforma’s OAuth client completes Icons8’s PKCE flow (Client ID Metadata Document or dynamic registration, scope `icons8`). The plugins repo does not implement that client.
- A full legal license URL beyond the summary at `https://icons8.com/license`.

## Assumptions and coverage limits

- Plugins has no `.codegraph/` index. This map is from direct reads of the files named above, not from CodeGraph.
- Traced end to end: `marketplace/search/exa`, `marketplace/media/lottiefiles`, `marketplace/media/recraft`. `marketplace/media/fal` was read only for the bearer-key shape. Other plugins were not.
- `bun run pack` and `bun test` were not run. The live MCP URL was not probed.
- No catalog-card component exists in this repo. The plate color is defined in the manifest docs and stamped by the packer.
- Secondary color pages (`brandcolor.dev`, IconCreator) were not used as sources. The hex below is the `fill` on Icons8’s own header SVG.
