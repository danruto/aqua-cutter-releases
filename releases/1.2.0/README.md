# Renderer release

Use the assets, schemas, [authoring guide](authoring.md) and [complete default example](examples/v1.0.0.json) from this same immutable release. The manifest supplies the exact renderer version, source commit, default schema, supported roots and SHA-384 integrity values. Renderer versions and payload schema versions are independent.

## Loading the assets

1. Verify `manifest.json` and the final asset bytes before use.
2. Select an approved immutable base URL, such as `https://approved-host.example/releases/X.Y.Z/`. This is an illustrative address, not an approved public host.
3. Load `renderer.css` and `renderer.js` from that exact directory. Set each `integrity` attribute to its own manifest value; retain `crossorigin="anonymous"` and `referrerpolicy="no-referrer"` on both elements.
4. Wait for both loads to succeed before mounting. Do not retry with unhashed assets after failure.

The following is an HTML **snippet**, not a complete host. Replace the uppercase placeholders with the approved immutable URLs and manifest integrity strings:

```html
<link rel="stylesheet" href="VERSIONED_BASE/renderer.css" integrity="CSS_SHA384" crossorigin="anonymous" referrerpolicy="no-referrer">
<script defer src="VERSIONED_BASE/renderer.js" integrity="JS_SHA384" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
```

The JavaScript and CSS are a self-contained pair: no package imports, runtime chunks, remote fonts or React setup are required. Importing the script does not scan or mount the document. This release uses Pele, not a runtime engine switch or automatic backup. Replacing either asset requires a new renderer release, never an in-place correction.

## Browser API

`window.AquaCutterRender` has exactly these four members:

| Member | Result |
| --- | --- |
| `rendererVersion` | Actual package-derived build version as a string |
| `validateWalkthrough(json)` | `{ ok: true, data }` or `{ ok: false, issues }` |
| `mountWalkthrough(container, json, options?)` | Promise of `{ ok: true, handle }` or `{ ok: false, issues }` |
| `mountDiagram(container, { source, description }, { scheme })` | Promise of `{ ok: true, handle }` or `{ ok: false, issues }` for one diagram reader |

Supply a JSON string and an empty dedicated `HTMLElement`. Validation checks structure, bounds and relationships without DOM changes. Mounting repeats validation, renders and sanitizes every diagram, resolves all targets and only then installs the reader. Validation success is not mount success, including for hidden or unselected diagrams.

The following is a JavaScript **snippet** inside a trusted async bootstrap, after both verified assets load. `json`, `container`, `fallback`, `failureNotice` and `saveHandle` are host-owned values, not library exports:

```js
const mounted = await window.AquaCutterRender.mountWalkthrough(container, json, { mode: "embedded" });
if (mounted.ok) {
  fallback.hidden = true;
  saveHandle(mounted.handle);
} else {
  failureNotice.hidden = false;
}
```

Options are `{ mode: "embedded" }` (the default) or `{ mode: "page" }`. Embedded mode themes only the reader. Page mode also themes the document canvas and gutters; choose it only when the walkthrough owns the page. It restores its previous document attributes on teardown or failed mounting without rewriting host inline styles.

Keep the successful handle and call `handle.unmount()` before removing or replacing the reader. Teardown is idempotent and releases owned DOM, listeners, observers, dialogs, page theme and scroll locks without releasing host-owned locks. Source replacement means unmount and mount again; no update or public selection API exists. Only one active walkthrough per document is supported. An occupied container is rejected without replacing its contents.

Each issue has only `{ code, path }`. An empty path means a root failure; input values, raw engine errors and stacks are not returned.

| Code | Meaning |
| --- | --- |
| `invalid-json` | JSON could not be parsed |
| `unsupported-version` | Schema version is not declared by this release |
| `unsupported-lens` | Lens is not available in that exact version |
| `invalid-data` | Invalid shape, unknown field or relationship |
| `limit-exceeded` | Byte, line or collection bound exceeded |
| `invalid-diagram` | Mount-only unsupported header/family, parsing rejection or empty diagram |
| `invalid-target` | Mount-only ambiguous or unresolved sanitized target |
| `render-failed` | Mount-only occupied container or unexpected rendering failure |

Do not infer success from a partial page or reinterpret a programming defect as invalid authored data.

### Diagram mounts

A documentation host can mount one diagram reader per Mermaid block, with many readers on one page. The following is a JavaScript **snippet** inside a trusted async bootstrap, after both verified assets load. `container`, `source`, `description`, `fallback` and `saveHandle` are host-owned values:

```js
const mounted = await window.AquaCutterRender.mountDiagram(container, { source, description }, { scheme: "flexoki-light" });
if (mounted.ok) {
  fallback.hidden = true;
  saveHandle(mounted.handle);
}
```

- `container` must be an empty dedicated `HTMLElement`.
- `source` must start with a `flowchart` or `graph` header (`TD`, `TB`, `LR`, `BT` or `RL`) or with `sequenceDiagram`. It is limited to 8192 UTF-8 bytes and 256 lines.
- `description` is required, nonblank and limited to 2048 UTF-8 bytes. It is the diagram's accessible name.
- `scheme` is a required colour scheme slug. No other input or option fields are accepted.

The reader shows the canvas, zoom controls, Fit, 1:1 and Full view, with no caption, source disclosure, Colour scheme picker or mode button. The host is the only scheme control. A diagram mount never themes the document root or body, and makes no network requests or storage writes.

A failure returns exactly one `{ code, path }` issue and changes nothing in the container, so the host keeps its own readable source fallback. `invalid-data` and `limit-exceeded` use path `/source`, `/description`, `/scheme` or `""`. `invalid-diagram` uses `/source`. `render-failed` uses `""` for an occupied container or an unexpected rendering failure.

Keep each successful handle. `handle.setScheme(slug)` switches only that reader and any open Full view, keeps its zoom and position, and returns `true`; an unknown slug or a call after teardown changes nothing and returns `false`. `handle.unmount()` is idempotent: it closes an open Full view, releases only this reader's scroll lock and leaves the container empty for a new mount. No source update API exists; unmount and mount again.

Diagram mounts and a mounted walkthrough in the same document are unsupported. Use `mountDiagram` on documentation pages and `mountWalkthrough` on separate walkthrough pages.

## Capabilities and limits

The [guide](authoring.md) lists all eighteen lenses in the sole stable schema `1.0.0`, including diagram-free Recovery, Decision Table, Identity and Rollout. Only flowcharts and sequence diagrams are supported; WhatBreaks and EventModel require flowcharts. The shared viewer supports pan, zoom, Fit, 1:1, Full view, pointer and keyboard inspection. C4 drills, architecture companions, pins and the manual Tour use exact authored addresses, not inferred source relationships.

Recovery presents operation tabs and an ordered ledger of supplied knowledge, retained intent, literal budgets and conditional alternatives. It never retries, polls or executes an action, treats unknown as failure, or turns a park into resolved work. All operation choices and fields must remain in the complete host fallback; see [Recovery authoring](authoring.md#recovery).

Decision Table presents ordered rule matrices, native table/case choices, authored first-match or unique declarations, complete outcomes/reasons/effects and an explicit unmatched result. It never evaluates predicates, derives matches, proves exclusivity/coverage or verifies a credential. Every inactive table, rule, case, reference and effect belongs in complete fallback; see [Decision Table authoring](authoring.md#decision-table).

Identity presents native identity/pair choices, raw/normalized ingredients, literal supplied schemes/encodings/keys, declared same/different/unknown relations and a complete lifetime ledger with optional scheme evolution. It never generates, hashes, normalizes, looks up or verifies identities or remote effects; equal keys do not infer sameness. Every inactive identity/pair/sample/value and all absence labels belong in complete fallback; see [Identity authoring](authoring.md#identity).

Rollout presents ordered authored snapshots, native rollout/phase choices, gates with separate supplied evidence/source, existing/new work, complete rollback obligations, every coexistence combination and release-together group. It never deploys, infers readiness or compatibility, compares versions or verifies claims. All inactive phases and all absence notices belong in complete fallback; see [Rollout authoring](authoring.md#rollout). The complete default example is synthetic, not a live release or operational approval.

All authored strings are inert text. Cosmetic source highlighting never executes, fetches or verifies them. References, coverage, expected results, dependency pins, classifications and narration are authored claims, not passing tests, observed telemetry, replay results or security guarantees. There is no arbitrary Mermaid-family support, source fetching, inferred diff, automatic migration, persistent theme storage or analytics.

## Host security and fallback duties

- Keep complete readable fallback visible until mounting succeeds. Preserve every section, source, reference and unselected item for disabled JavaScript, offline use, unavailable assets, integrity rejection and mount failure. A selected tab alone is not complete fallback.
- Escape fallback HTML. Escape embedded JSON, including `<`, so authored closing-script text cannot become executable markup. Never insert authored content as trusted HTML.
- Use Hypertext Transfer Protocol Secure (HTTPS) and Subresource Integrity (SRI) on both assets. Retain anonymous cross-origin loading, the host's required cross-origin resource-sharing (CORS) responses and no-referrer loading policy.
- Retain a restrictive Content Security Policy (CSP), including `connect-src 'none'`, blocked forms and base URLs, and only approved script/style resources. The viewer requires controlled style attributes. Authorize a trusted bootstrap explicitly; do not relax the policy to admit arbitrary inline scripts.
- Asset requests carry no credentials or private walkthrough data. A trusted renderer can read its page: integrity proves byte identity, not harmlessness.
- Discovery does not authorize a content delivery network (CDN), public assets on private pages, private-content publication or telemetry. Host approval, authentication and production security acceptance remain separate.

The renderer does not fetch release metadata at runtime. Discovery belongs to the consuming tool, outside the browser and payload. See the [consumer selection protocol](authoring.md#consumer-selection-protocol). Fixed synthetic examples demonstrate authored content; they are not execution evidence or an arbitrary-data private publisher.

Publication needs explicit approval and a future destination-specific deployment adapter. These documents and local verification do not implement remote publication, promotion or rollback.
