# Icons8 MCP

## Summary

Icons8 publishes a remote MCP server, product name “Icons8 MCP” / “Icons8 MCP server”, registry id `com.icons8/icons8`. Endpoint `https://mcp.icons8.com/mcp/`. Transport is Streamable HTTP. Nothing is installed locally. A credential is required on every call: OAuth sign-in (scope `icons8`) or a personal API key in `Authorization: Bearer`. Thirteen tools: six icon, six Ouch illustration, plus `account_info`. Free PNG use needs a link to icons8.com. Paid downloads are 100 per hour and 1000 per day.

The header mark on icons8.com is an SVG filled `#1fb141`. Favicon PNGs are linked from the same homepage. No official brand-guidelines page stating a hex was found.

## Observed facts

Fetched 2026-11-18 unless noted.

### Where it is documented

- Product docs: `https://developers.icons8.com/docs/mcp`. Client snippets and the agent-skill list. This page does not list tools.
- Server README: `https://github.com/icons8/icons8-mcp` (`raw` `main/README.md`). Tools, free vs paid, rate limit, transport names.
- Agent card: `https://mcp.icons8.com/llms.txt`. Same tool set, with the quota and credential split spelled out.
- Auth recipe: `https://mcp.icons8.com/auth.md`.
- Skills index (linked, not fetched): `https://mcp.icons8.com/.well-known/agent-skills/index.json` and `https://github.com/icons8/agent-skills`.
- License summary: `https://icons8.com/license`.

Context7 library `/websites/developers_icons8` is the REST API at `developers.icons8.com` (search, icons, photos, music, illustrations, upscaler). Auth there is an `Api-Key` header or `token` query parameter. That is a different surface from the MCP server. Do not copy those REST paths into `mcp.json`.

The developers MCP page and the GitHub README disagree on the example server key (`icons8` vs `icons8mcp`). Both use the same URL. Reforma’s packer renames a lone server key to the plugin directory name, so the upstream key is not stable.

### Endpoint, package, transport

- URL: `https://mcp.icons8.com/mcp/`.
- No npm package and no stdio command. README: the client connects directly; there is no local process. FAQ: Node.js and `npx mcp-remote` are not required. An older `mcp-remote` config still works.
- Transport: MCP Streamable HTTP. Hosts may label it `http`, `streamable-http`, or, on older clients, `SSE`. Those labels refer to this same URL.
- Icons8’s own JSON examples use `"type": "http"`. Reforma’s `mcp.json` files and `https://docs.reforma.ai/integrations/mcp` use `"type": "streamable-http"` for remote servers (Exa, LottieFiles, Recraft).
- Data processing is at `mcp.icons8.com`. Icons8 terms and privacy apply. The README’s terms link is the homepage, not a separate privacy URL.
- Unauthenticated request: `401` with `WWW-Authenticate`.

Protected-resource metadata (from `auth.md`, not re-fetched as JSON):

- Resource `https://mcp.icons8.com/mcp`
- Authorization server `https://mcp.icons8.com/`
- Scope `icons8`
- Bearer method: header only
- Authorization `https://mcp.icons8.com/authorize`, token `https://mcp.icons8.com/token`, registration `https://mcp.icons8.com/register`
- PKCE `S256`, grant `authorization_code` plus refresh, and a WorkOS agent-claim grant
- Access token prefix shown as `mcp1_...`, `expires_in` 3600. The token resolves to the account’s personal API key.

### Auth

Two credentials. Icon tools accept either. Illustration paid formats and a full `account_info` need the OAuth sign-in. A pasted key authenticates the MCP call but is anonymous to the Ouch API, so those paid formats stay closed and illustration search previews come back watermarked.

OAuth, from `auth.md`:

1. `401` points at `https://mcp.icons8.com/.well-known/oauth-protected-resource/mcp` (also mirrored at the root well-known path).
2. Client ID Metadata Document, or `POST https://mcp.icons8.com/register`.
3. Browser sign-in at `/authorize` with `scope=icons8` and PKCE. Email/password on Icons8’s page. The README also says Google, Apple, Microsoft, or GitHub.
4. `POST /token`. Refresh and `POST /revoke`.
5. Call the MCP URL with `Authorization: Bearer <access_token>`.

A headless alternative (`service_auth` / verified-email claim) posts to `https://mcp.icons8.com/agent/identity` and polls the token endpoint with grant `urn:workos:agent-auth:grant-type:claim`. Anonymous is rejected (`anonymous_not_enabled`).

Personal key, no registration and no expiry:

```text
Authorization: Bearer <personal Icons8 API key>
```

The account’s MCP tab is where Icons8 says the snippet with the key already filled in lives. That page was not opened.

Reforma’s MCP doc (same fetch day) matches the OAuth case without a plugin field: do not put OAuth config in `plugin.json`; a `401` (or `403` plus `insufficient_scope`) surfaces Log in; the stored access token is sent as `Authorization: Bearer`. The API-key case matches `marketplace/media/fal`: a required string variable and `"Authorization": "Bearer ${API_KEY}"`. That `${}` substitution is Reforma-specific. A key-only plugin would not reach paid illustration tools, per `auth.md`.

`get_icon_svg` stays off `tools/list` until the connection has a credential.

### Tools

Counts and names agree between the GitHub README and `llms.txt`: 13 tools. Parameter schemas were not captured from a live `tools/list`. Behavior below is those two pages.

Icons (buttons, menus, tables):

| Tool | What it does | Plan |
| --- | --- | --- |
| `search_icons` | Keyword search. `animated=True` keeps only animated icons. Those hits include permanent `animation.gif` and `animation.apng` links that need no further credential. | Free once connected |
| `list_categories` | Icon categories, paginated. | Free once connected |
| `list_platforms` | Styles (iOS, Fluent, Color, …), paginated. | Free once connected |
| `get_icon_png_url` | PNG preview URL, 24×24 by default. | Free |
| `get_icon_svg` | Icon as SVG. Hidden from `tools/list` until a credential is present. | Plan with MCP access |
| `get_icon_animation` | Animated icon only (`isAnimated`). `gif` and `apng` are permanent URLs at the style’s own size; a size the icon is not drawn at is refused. `format="lottie"` returns Lottie/Bodymovin JSON. No animated SVG. | GIF/APNG free; Lottie needs MCP access |

Illustrations (Ouch; heroes, empty states, onboarding):

| Tool | What it does | Plan |
| --- | --- | --- |
| `search_illustrations` | Keyword search. Every hit has a permanent preview link. | Connected; previews watermarked if the credential is only a pasted key |
| `list_illustrations_categories` | Paginated categories. | Connected |
| `list_illustrations_styles` | Styles. Docs say keep one style per project. | Connected |
| `get_illustration_svg` | Presigned SVG URL, not the file body. Expires in one hour. | Sign-in whose plan covers illustrations and MCP, unless the hit has `free_distribution: true` |
| `get_illustration_png_url` | Presigned PNG in three sizes. `size="hd"` is paid. | Same; `gif-low` is called out as free on the animation tool, and HD PNG is the paid PNG size |
| `get_illustration_animation` | Presigned URL: WebM, HEVC MP4, ProRes, GIF, Lottie, or After Effects. Search hits often already include permanent WebM and HEVC. | Any format except `gif-low` needs the illustration+MCP plan, unless `free_distribution: true` |

Account:

| Tool | What it does |
| --- | --- |
| `account_info` | Plan, SVG/Lottie/illustration downloads left, renewal, server paid-download limit. Takes no download. After sign-in, full detail. A plan without MCP access gets the plan and the free formats. A pasted key shows the account only if that account also signed in; otherwise it says to sign in. |

Paid-download cap, icons and illustrations together: 100 per hour and 1000 per day. Applies to `get_icon_svg`, `get_icon_animation` with `format="lottie"`, `get_illustration_svg`, `get_illustration_png_url` with `size="hd"`, and `get_illustration_animation` except `gif-low`. Refusal includes `retry_after_seconds` and names a free format that still works.

A second cap is the plan’s own SVG and Lottie allowance. When it is spent, `code` is `limit_exhausted`. `get_icon_svg` then returns a free 24px PNG in `url`. Lottie refusal returns gif and apng links. Allowances are per tool. `get_icon_png_url` still works. `get_icon_svg` can still serve an icon that account already downloaded.

The Illustrations plan alone has no MCP access. Dollar prices and plan names were not read. The README links `https://icons8.com/pricing?utm_source=github&utm_campaign=mcp`.

Catalog sizes in the README: 420,000+ icons in 130+ styles; 110,000+ Ouch illustrations in 340+ styles. The developers homepage snippet in search said “368,000 icons in 100+ styles.” The README figures are the ones quoted here. They were not rechecked against the live catalog.

### License and attribution

`https://icons8.com/license` (summary page, 2026-11-18): free graphics in exchange for a link to icons8.com. Websites and apps: link on pages that use the content, or the footer if that is most pages. Desktop and mobile: About or Settings, plus the store description for mobile. Other media have their own credit lines (tag, credits, caption). Open-source projects can ask Icons8 to open-source needed assets. Educational licenses exist via support.

The MCP README narrows the free tier: “Free PNG use requires attribution with a link back to Icons8,” linking that license page. It does not say a paid plan removes the link. That was not inferred.

### Brand assets

No press-kit file or brand-guidelines page with a published hex was found. `https://icons8.com/contact` tells journalists to “check out our press kit” and does not give the kit URL. `https://icons8.com/l/media-kit/` is an advertising rate card, not a logo pack.

Primary assets that are actually linked from `https://icons8.com/` (HTML fetched 2026-11-18):

- Header mark: `https://goodies.icons8.com/web/common/header/logo/logo-icons8.svg`
- Favicon: `https://icons8.com/favicon.ico`
- 32×32 PNG: `https://maxst.icons8.com/vue-static/landings/primary-landings/favs/favicon-32x32.png`
- 96×96 PNG: `https://maxst.icons8.com/vue-static/landings/primary-landings/favs/favicon-96x96.png`
- Apple touch icon, `sizes="120x120"`: `https://maxst.icons8.com/vue-static/landings/primary-landings/favs/apple-touch-icon.png`

The SVG was read in memory and not saved. It is 216 bytes, `viewBox="0 0 14 14"`, one fill `#1fb141` (a square plus two circles). That is the only hex taken from an Icons8-hosted file. The PNGs were not decoded, so their colors are not claimed. Hashed `_nuxt` landing SVGs on `maxst.icons8.com` are build assets, not a stable logo URL.

`brandcolor.dev` and IconCreator also say `#1FB141`. They are not Icons8. They were not used.

For a marketplace plate, the field is `interface.brandColor` (see `plugin-shape.md`). `#1fb141` is the fill of this mark, not a color Icons8 published as “use this behind our logo.” On a green plate the green mark disappears. LottieFiles puts a white glyph on `#00DDB3`. Exa puts a colored wordmark on `#FFFFFF`. That contrast is an inference from those plugins plus this SVG, not an Icons8 rule.

## What lines up with a marketplace plugin

Observed Reforma facts that match this server, not a build sequence:

- Remote `mcp.json` with `type` `streamable-http` and `url` `https://mcp.icons8.com/mcp/`. Same URL the official docs use with `type` `http`.
- OAuth: no auth fields in the manifest. Icons8’s `401` plus `WWW-Authenticate` is the case the Reforma MCP doc describes. Scope and PKCE details live on Icons8, not in `plugin.json`.
- API key instead: `Authorization: Bearer` header, same header `auth.md` requires, same pattern as Fal’s `${API_KEY}`. A key does not unlock paid illustration tools.
- No stdio `command`. Pack will not probe a server that answers 401, so a packed catalog may ship without cached tool metadata. That does not fail `pack`.
- Logo file under `assets/`, referenced by `extensions.reforma.logo` and `interface.logo`. The SVG above is a real mark (14×14 viewBox, single path fill), not a placeholder. Favicon PNGs are the raster alternatives. Neither was copied into the repo.
- Plate: `interface.brandColor`. No background-image field exists.
- Skills are optional. Exa ships one; LottieFiles and Recraft ship none. Icons8’s skills (`icons8`, `ouch`, and the design checks named on the developers MCP page) live in `icons8/agent-skills`, separate from the MCP endpoint.
- No install hook is required.

## Sources

- `https://developers.icons8.com/docs/mcp` — 2026-11-18
- `https://raw.githubusercontent.com/icons8/icons8-mcp/main/README.md` — 2026-11-18
- `https://mcp.icons8.com/llms.txt` — 2026-11-18
- `https://mcp.icons8.com/auth.md` — 2026-11-18
- `https://icons8.com/license` — 2026-11-18
- `https://icons8.com/` HTML, logo SVG body — 2026-11-18
- `https://docs.reforma.ai/integrations/mcp` and `/manifest` — 2026-11-18
- Context7 `/websites/developers_icons8` — 2026-11-18, REST only

## Open questions

- Plan names and prices on `https://icons8.com/pricing`.
- Whether paid use still requires the icons8.com link.
- Live `tools/list` input schemas, and whether an unauthenticated probe is 401 before or after `initialize`.
- Whether Reforma’s OAuth client can complete this server’s registration and PKCE (scope `icons8`, bearer header only).
- Stable wordmark (text “Icons8”) as opposed to the 14×14 mark. The header asset found is the mark only.
- Full license document behind “Read full license.” The fetched page is the summary.

## Assumptions and limits

- Tool behavior is from README and `llms.txt`, not a call to `https://mcp.icons8.com/mcp/`.
- OAuth metadata JSON in `auth.md` was not re-fetched from the well-known URLs.
- `icons8/agent-skills` file tree was not read. Skill names are from the developers MCP page and `llms.txt`.
- Glama’s `search_icons` schema was not used. It is a third-party mirror.
- The `$15` figure that appeared in a search snippet of the GitHub README is not in the README fetched from `main`. It is omitted.
- Logo SVG was fetched to read `fill` and `viewBox` and was not written into the repo.
