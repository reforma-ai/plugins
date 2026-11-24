# Marketplace MCP plugin shape

## Summary

A plugin the catalog ships is a directory under `marketplace/<category>/<name>/` with a root `plugin.json`, listed by `name` and `source` in `marketplace.json`. MCP is a sibling `mcp.json`. The logo is an image file. The preview-box background is `extensions.reforma.interface.brandColor`, a hex plate behind that logo. This repo has no card renderer.

## How it connects

`marketplace.json` is the catalog index. `parseMarketplace` in `scripts/pack/shared.ts` walks `categories[].plugins`, rejects a top-level `plugins` array, and copies the parent category `id` onto each `MarketplaceListing`. `packPlugin` in `scripts/pack/index.ts` then:

1. Copies `listing.source` into `dist/catalog/<name>/` (or downloads a pinned GitHub tree).
2. `normalizePluginLayout` — vendor layouts into the canonical tree.
3. `unfoldReformaExtension` — hoists `extensions.reforma` to the top level so later steps can read `logo` and `interface`.
4. `normalizePluginContentPaths` — convention folders; `renameLoneMcpServerKey` renames a single MCP server key to the plugin directory name.
5. `normalizePluginHooks` — only if hooks exist.
6. `normalizePluginTools` — bundles `tools/*.ts` into `tools.mjs` when present.
7. `applyCatalogOverlay` — listing `displayName`, `description`, `brandColor`, `logo`, `logoSmall` from `marketplace.json` or `marketplace/_brand/<name>/`.
8. `normalizePluginLogos` — keeps or downloads the logo, drops a missing or oversized one.
9. `discoverMcpTools` — probes HTTP MCP for tool metadata. A non-OK response, including 401, is ignored. Stdio is not probed.
10. `stampPluginCategory` — writes `category` from the shelf id.
11. `stampPackedAgent` — mentions.

After every plugin, `publishCatalogLogosFromEnv` rewrites logo paths to CDN URLs when `S3_URL` and `S3_PUBLIC_URL` are set. `foldReformaExtension` then moves non-portable keys (`logo`, `interface`, `category`, `agent`, `variables`, …) back under `extensions.reforma`. Portable keys stay at the top: `$schema`, `name`, `version`, `description`, `author`, `homepage`, `repository`, `license`, `keywords`, `extensions` (`PORTABLE_MANIFEST_FIELDS` in `scripts/pack/extensions.ts`).

Source manifests are expected to already nest Reforma fields under `extensions.reforma`. Convention paths (`mcp.json`, `skills/<name>/SKILL.md`, `hooks/hooks.json`, `tools/*.ts`) do not have to be declared. Some plugins still set `extensions.reforma.mcpServers` to `./mcp.json` (Exa, LottieFiles, Recraft do).

Category is not authored on the source `plugin.json`. The shelf in `marketplace.json` is the category. `"disabled": true` keeps a listing out of the packed catalog.

## Traced plugins

### Exa — remote search, skill, two SVGs

Shelf `search` / “Search”. Listing is only name and source:

```162:164:marketplace.json
"name": "exa",
"source": "./marketplace/search/exa"
```

- `marketplace/search/exa/plugin.json` — name `exa`, author, homepage `https://exa.ai/docs/reference/exa-mcp`. Reforma block: logos, `interface.displayName` `Exa`, `brandColor` `#FFFFFF`, `agent.mentions`.
- `marketplace/search/exa/mcp.json` — one server, key `exa`, `type` `streamable-http`, URL `https://mcp.exa.ai/mcp?tools=web_search_exa,web_fetch_exa,web_search_advanced_exa,agent_run&login`. No `headers`, no `variables`. The `login` query is Exa’s own sign-in, not a Reforma API key.
- `marketplace/search/exa/skills/exa/SKILL.md` — skill `exa`. Tells the agent to use the connected account and names the tools. No rules, agents, tools, or hooks directory.
- `marketplace/search/exa/assets/logo.svg` — wordmark, `width="413"` `height="129"`.
- `marketplace/search/exa/assets/logo-small.svg` — mark only, `viewBox="0 0 107 129"`, fill `#0143D9`.

### LottieFiles — remote media, one SVG, no small logo

Shelf `media` / “Media”, source `./marketplace/media/lottiefiles`.

- `marketplace/media/lottiefiles/plugin.json` — `brandColor` `#00DDB3`, `logo` `assets/logo.svg` on both `extensions.reforma` and `interface`. No `logoSmall`. Mentions `lottiefiles.com`.
- `marketplace/media/lottiefiles/mcp.json` — `type` `streamable-http`, `url` `https://mcp.lottiefiles.com/mcp`. No headers.
- `marketplace/media/lottiefiles/assets/logo.svg` — `viewBox="0 0 24 24"`, `fill="#ffffff"`. The mark is white; the plate `#00DDB3` is what makes it visible.
- No `skills/`, `hooks/`, or `tools/`.

### Recraft — remote media, PNG pair

Shelf `media`, source `./marketplace/media/recraft`.

- `marketplace/media/recraft/plugin.json` — `brandColor` `#000000`, `logo` `assets/logo.png`, `logoSmall` `assets/logo-small.png`, same paths on `interface`.
- `marketplace/media/recraft/mcp.json` — `type` `streamable-http`, `url` `https://mcp.recraft.ai/mcp`. No headers.
- Both PNGs are 180×180, 858 bytes (`sips`). The packer does not check pixel size. Docs call `logoSmall` a 12–14px glyph; that size is not enforced.
- No skills or hooks.

`marketplace/media/fal` is the bearer-key shape, not part of the three traces: `extensions.reforma.variables.properties.API_KEY` (required string) and `mcp.json` header `Authorization: Bearer ${API_KEY}` against `https://mcp.fal.ai/mcp`. Reforma docs say `${API_KEY}` substitution is a Reforma extension, and string variables are vault secrets by default.

Design shelf (`id` `design`) contains only Figma. Media contains LottieFiles, Recraft, SVGator, and other generation/asset servers. Which shelf an Icons8 listing would use is not fixed by code. Observed neighbors for icon and illustration retrieval sit under `media`.

## Logo files

Observed from `scripts/pack/logos.ts` and the manifest docs (fetched 2026-11-18 from `https://docs.reforma.ai/integrations/manifest`).

| Piece | Where it lives | Rule |
| --- | --- | --- |
| Main logo | `extensions.reforma.logo` and `interface.logo` | Relative path, usually `assets/logo.svg` or `assets/logo.png`. `readLogoFields` prefers the outer `logo`, then `interface.logo`. |
| Small logo | `logoSmall` and `interface.logoSmall` | Optional. Omitted, or deleted at pack time, when it equals the main logo. Docs: falls back to `logo`. |
| Allowed types | `LOGO_EXT` | `svg`, `png`, `webp`, `jpg`, `jpeg` (jpeg stored as `.jpg`). Anything else throws `unsupported logo`. |
| Size cap | `LOGO_MAX_BYTES` | 512×1024 bytes. Larger: warning, local file deleted, logo fields removed. Pack continues. |
| Missing file | `materializeLogo` | Warning `skip missing logo`, fields removed. Does not throw. |
| HTTP logo | listing `logo` or a manifest URL | `applyCatalogOverlay` copies the URL onto the manifest; `materializeLogo` downloads it. Failed download throws. |
| External pin artwork | `marketplace/_brand/<name>/logo.*` and `logo-small.*` | Used when the listing has no logo path and the upstream tree has none. Local plugins keep artwork in their own `assets/`. |
| CDN | `publishCatalogLogos` | Key `plugins/<sha256>.<ext>` in bucket `reforma-public`. Stamps `https` URLs back onto `logo` / `interface.logo`. Local `bun run pack` skips this and warns when `S3_URL` or `S3_PUBLIC_URL` is unset. CI (`GITHUB_ACTIONS=true`) throws if they are unset. |

No test reads image bytes. A placeholder file with an allowed extension under 512KB packs. The “real icon” requirement is not enforced here.

`scripts/pack/publish-logos.ts` does not touch `brandColor`.

## Preview-box background

There is no preview-box component, CSS module, or card renderer in this repo. Nothing named preview background, plate image, or cover image is a manifest field.

The field that fills that role is `extensions.reforma.interface.brandColor`. The manifest doc calls it “Hex background plate color behind the logo.” `interface.brandColorDark` is documented as the optional dark-mode plate. A search of `*.json`, `*.ts`, and `*.md` in this repo found zero `brandColorDark` values.

`stampInterface` in `scripts/pack/logos.ts` writes `brandColor` only when the `marketplace.json` listing sets it (remote overlays such as Linear `#5E6AD2`, Supabase `#3ECF8E`, Higgsfield `#D1FD17`). Local plugins set the hex themselves. `parseMarketplace` requires a listing `brandColor` to be a non-empty string. It does not check that the string is hex.

Examples from source `plugin.json`:

- Exa — `#FFFFFF` — white plate, blue/black wordmark SVG
- LottieFiles — `#00DDB3` — green plate, white 24×24 SVG
- Recraft — `#000000` — black plate, 180×180 PNG

The file that would have contained a renderer is not in this repository. The definition sits on `https://docs.reforma.ai/integrations/manifest`. `README.md` sends authoring semantics there and does not name a front-end file.

## Hooks

None of the three traced plugins ship `hooks/hooks.json`. `HOOK_EVENT_NAMES` in `scripts/pack/hooks.ts` includes `Install` and `Uninstall`, so an install hook is a recognized event when a plugin has one. Adding a plugin does not require one.

## Tests that fail if a plugin is added wrong

`package.json` script `pack` is `bun run scripts/pack.ts`. Tests are `bun:test` files under `scripts/pack/`. They were not executed.

Would throw or fail:

- `scripts/pack/marketplace-sources.test.ts` — every `marketplace/**/plugin.json` top-level key must be in `PORTABLE_MANIFEST_FIELDS`. A source file with top-level `logo`, `brandColor`, or `mcpServers` fails.
- `parseMarketplace` (`scripts/pack/shared.ts`, covered by `shared.test.ts`) — missing `name` or `source`, duplicate plugin name, duplicate category id, `disabled` other than `true`, overlay field that is not a non-empty string, a root `plugins` array.
- `packPlugin` — `Plugin source missing` when the local directory does not exist; `cannot stamp category (no plugin.json)`.
- `normalizePluginLogos` / `applyCatalogOverlay` — unsupported extension; overlay file path that does not exist (`overlay logo missing`).
- `publishCatalogLogosFromEnv` — in CI, missing `S3_URL` or `S3_PUBLIC_URL`.

Would not fail:

- No logo and no `brandColor`. `normalizePluginLogos` returns immediately when `readLogoFields` finds no logo.
- Missing logo file (warning, fields stripped).
- MCP server that returns 401. `mcpRpc` in `scripts/pack/mcp-tools.ts` returns `undefined` when `!res.ok`, so tool metadata is simply absent.
- `logoSmall` at the wrong pixel size.
- Listing `name` that does not equal `plugin.json` `name`. That match is stated in the manifest doc and was not found as a test. `renameLoneMcpServerKey` uses the destination directory name, which comes from `listing.name`.

## Relevant files

- `README.md` — repo layout, pack pipeline, links to docs
- `marketplace.json` — shelves and listings
- `scripts/pack/index.ts` — `packPlugin`
- `scripts/pack/shared.ts` — `MarketplaceListing`, `parseMarketplace`
- `scripts/pack/extensions.ts` — `PORTABLE_MANIFEST_FIELDS`, `unfoldReformaExtension`, `foldReformaExtension`
- `scripts/pack/logos.ts` — `LOGO_EXT`, `LOGO_MAX_BYTES`, `materializeLogo`, `applyCatalogOverlay`
- `scripts/pack/publish-logos.ts` — `publishCatalogLogosFromEnv`
- `scripts/pack/content.ts` — `stampPluginCategory`, `renameLoneMcpServerKey`
- `scripts/pack/mcp-tools.ts` — `discoverMcpTools`
- `scripts/pack/hooks.ts` — `HOOK_EVENT_NAMES`
- `scripts/pack/marketplace-sources.test.ts` — source manifest key allowlist
- `scripts/pack/logos.test.ts` — overlay stamping, not pixel checks

## Constraints

- New Reforma-owned plugins use the canonical layout. `.mcp.json` and `.cursor-plugin` are compatibility inputs.
- One MCP server should use the plugin name as its key. A lone other key is renamed to the directory name.
- OAuth is not configured in `plugin.json`. Docs: connect, and on `401` or `403` with `insufficient_scope`, the product shows Log in and later sends `Authorization: Bearer <token>`.
- API keys go in `extensions.reforma.variables` and are referenced as `${NAME}` in `mcp.json` headers.
- Pack does not store MCP resource bodies.

## Open questions

- Where the catalog card reads `brandColor` and whether it uses `logo` or `logoSmall`. That UI is outside this repo and was not opened.
- Whether a missing logo is rejected later by the app, or only stripped here.

## Assumptions and limits

- Docs pages were fetched 2026-11-18. They can drift from the packer. `brandColorDark` is docs-only in this tree.
- PNG dimensions are from `sips` on the two Recraft files. SVG sizes are from the file headers. Pixels were not inspected.
- `marketplace/_brand/` was listed, not read file by file. It is for external pins.
- Hooks, tools bundling, and agent mention folding were read only far enough to see they are optional for an MCP plugin.
