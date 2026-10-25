# Authoring walkthroughs

[Release instructions](README.md) explain asset loading, API results, teardown, fallback and host security. Use this release's own schemas and complete examples, not documentation from a moving default branch.

## Version selection

The default and only supported schemaVersion is **1.0.0**. A renderer version identifies an immutable asset/document set; a payload schema version selects an exact data contract. Do not put renderer selection metadata inside the payload.

| Schema version | Capability | Schema | Complete default example |
| --- | --- | --- | --- |
| 1.0.0 | All eighteen lenses, pins, CodeLens maxLines, sixteen-section capacity | [schema](schema/walkthrough.v1.schema.json) | [complete example](examples/v1.0.0.json) |

Pre-release labels and undeclared versions are rejected, never promoted or migrated. Every example uses the stable contract. Eighteen supported lens types do not increase the sixteen-section cap.

Complete synthetic examples by focus:

- [trace-diff](examples/trace-diff.json)
- [trace](examples/trace.json)
- [code-lens](examples/code-lens.json)
- [what-breaks](examples/what-breaks.json)
- [lifecycle](examples/lifecycle.json)
- [c4](examples/c4.json)
- [event-model](examples/event-model.json)
- [contracts](examples/contracts.json)
- [corpus](examples/corpus.json)
- [impact](examples/impact.json)
- [telemetry](examples/telemetry.json)
- [tradeoffs](examples/tradeoffs.json)
- [data-flow](examples/data-flow.json)
- [tour](examples/tour.json)
- [section-capacity](examples/section-capacity.json)
- [pins](examples/pins.json)
- [code-limit](examples/code-limit.json)
- [recovery](examples/recovery.json)
- [decision-table](examples/decision-table.json)
- [identity](examples/identity.json)

## Structures, bounds and source

Schemas define payload structures and bounds. This guide explains use; it does not introduce an alternate payload contract. Validate offline against the complete local schema closure using canonical `$id` identifiers. HTTPS identifiers are names, not permission to fetch a schema. Draft 2020-12 support comes from the validator. Runtime validation adds relationships; mounting additionally resolves sanitized rendered targets and may fail even after schema validation.

Every object rejects unknown properties. JSON is at most 256 KiB. Sections are 1–16. Input order is authoritative. Titles and lead are nonblank, up to 512 UTF-8 bytes; overview changes are 1–12 entries of at most 512 bytes. References are 0–32 records with labels up to 256 bytes and values up to 2 KiB. References are displayed literally, never fetched or verified.

The custom `maxUtf8Bytes` keyword counts `new TextEncoder().encode(value).length`, not UTF-16 code units. The custom `maxSourceLines` keyword counts `value.split(/\r\n|\r|\n/).length`: CRLF, CR and LF split lines, including a final empty row. Limits are inclusive. Generating types does not replace these runtime bounds; an offline validator must implement both keywords rather than silently ignore them.

Mermaid source is literal inert text: nonblank, at most 8192 UTF-8 bytes (8 KiB) and 256 source lines. Flowcharts start with `graph` or `flowchart`; sequences start with `sequenceDiagram`. Inline `+`/`-` activation notation is supported; standalone activation syntax and other families are not. No links, images, source fetching or code execution are enabled. Bounds do not promise a rendering deadline for arbitrary graphs.

Section IDs are unique and match `^[a-z][a-z0-9-]{0,63}$`; `overview` and `exact-references` are reserved. Node/endpoint IDs match `^[A-Za-z0-9_-]{1,64}$`. A diagram includes authored description, ordered steps, explicit node/edge changes and optional initial focus. Status values are `added`, `removed`, `moved`, `same`; unmarked elements use same presentation, not an inferred equality verdict.

## Exact addresses and pins

An architecture address is **section ID + level ID + pane role + FocusTarget**. Pane roles are `single`, `before`, `after`; the selected pane must exist. A FocusTarget is `{ kind: "node", node }` or `{ kind: "edge", edge: { from, to, label, occurrence } }`. Match the exact sanitized node ID or emitted edge label, including historical wrapping. Occurrence is zero-based in emitted order among equal from/to/label triples. Equal endpoints or IDs across levels/panes are not interchangeable. Empty edge labels are valid; labels allow 2 KiB and occurrences are integers 0–127.

Pins are optional: `{ section, architecture, level, pane, target }`, up to sixteen. Each non-Tour section has at most one pin; architecture names a different existing C4 section, level and present pane. Relationship errors fail validation; a target absent from the sanitized pane fails mounting. Pins navigate the existing reader; they do not discover architecture or make reference text executable.

## Eighteen lenses

### TraceDiff

`trace-diff` supplies `id`, `title`, `reason`, `before` and `after`. Both diagrams use the same kind, flowchart or sequence. Each supplies source, description, 1–64 steps, nodeChanges, edgeChanges and optional initialFocus. Markers are explicit, not a computed diff. Node changes allow at most 64 entries and edge changes 128; duplicates and unresolved targets fail. Initial focus is a persistent non-dimming pin, separate from pointer or keyboard focus. The before/after route map retains each pane's viewport and complete source.

### Trace


A Trace section has `id`, `type: "trace"`, `title`, `reason`, `diagram` and `paths`.
Its diagram has `kind`, `source` and `description`. Each path has `id`, `label`, `steps`,
`outcome` (`ok` or `reject`) and `outcomeDescription`, plus optional `notReached` on reject paths.
Each step has `title`, `narration` and `node`; every step after the first also has an exact
`incoming: { from, to, label, occurrence }` connecting the previous node to the current node.
The first step omits `incoming`. Sequence nodes are actor IDs and incoming edges are emitted messages.

- Paths per Trace section: 1–12; path IDs are unique within their section and follow the section-ID syntax.
- Steps per path: 1–64; repeated visits and exact repeated connections are allowed.
- Path labels and step titles: nonblank, at most 512 UTF-8 bytes each.
- Step narration and outcome description: nonblank, at most 2 KiB each.
- Not-reached lists: at most 64 nodes and 128 exact edges, with no duplicates or targets traversed by that path.
  Empty lists are valid; an untraversed parallel edge between visited nodes can be explicitly not reached.

Choose a path button, then a step in its ordered rail. Selection is manual, with no automatic playback.
Changing paths starts at its first step without resetting pan/zoom. The active instruction shows
`Step n / total`, title and narration. Only its final step shows the authored outcome explanation,
labelled `OK outcome` or `Reject outcome`; these are not execution results.
Only authored not-reached targets gain marks at the final reject step. No downstream execution is inferred.
All paths and targets must validate before any rich page mounts, including paths not initially selected.

Trace uses the shared Fit, 1:1, Zoom in/out and Full view controls, pointer drag, modified wheel zoom,
and keyboard `+`, `-`, `0`, `f` and arrow keys. Native Tab, Enter and Space operate path and rail buttons.
On a focused rail button, arrow keys select the previous or next step, and Home and End the first or last.
Hover and keyboard focus retain exact selected marks while dimming unrelated geometry.
Section navigation, themes and Full view retain each section's independent path, step and viewport;
a fresh mount starts at the first path and step. Presentation follows the shared Base16 theme; visual review remains pending.


### CodeLens


CodeLens B has `id`, `type: "code-lens"`, `title`, `reason`, `entries`, `coverage` and optional `initialHunk`.
Optional `maxLines` caps the visible diff rows; longer hunks scroll inside the diff. Omission uses 30 rows.
Each entry has `id`, `title`, `file`, `hunk`, `oldStart`, `newStart`, `why` and optional `references`.
Hunk IDs are unique per section and match `^[a-z][a-z0-9-]{0,63}$`.
A supplied `initialHunk` must name an entry; omission selects the first entry.
File identifiers group by exact equality in first-occurrence order, with no source access or path normalization.
References are inert identifiers under **Cited references**, not verified proof, links or passing-test claims.
Absent and empty reference lists both show `No cited references supplied`.

| CodeLens boundary | Inclusive maximum or required range |
| --- | --- |
| Entries per section | 1–32 |
| Hunk title and file identifier | Nonblank, at most 512 UTF-8 bytes each |
| Rationale `why` | Nonblank, at most 2 KiB |
| Hunk string | Nonempty, at most 8 KiB and 256 rows |
| References per hunk | 0–32; each nonblank, at most 256 bytes |
| Omitted categories | 0–32; labels unique, nonblank, at most 256 bytes |
| Coverage total and each omitted count | Integer, 1–4096 |
| Old/new starts | Integer, 0–1,000,000 |
| `maxLines` | Integer, 5–256; default 30 |
| Last emitted line number | At most 1,000,000 on each side |

Each hunk row begins with space (context), `+` (addition) or `-` (removal).
CRLF, CR and LF separate rows. The first marker is stripped; the body remains literal, preserving whitespace.
Marker-only rows are valid; unprefixed empty rows, empty trailing rows and context-only hunks are invalid.
Context advances both gutters; additions advance only new; removals advance only old.
Zero is allowed only on an unconsumed side for pure additions/deletions, never displayed as a line number.
No file headers, `@@` headers, patch parsing or code execution is provided.

Coverage is `{ total, omitted: [{ label, count }] }`, counting changed hunks in the author's declared scope.
`total` must equal `entries.length` plus the sum of omitted counts; no inferred remainder is added.
The ledger always shows `Showing N of T changed hunks`, the omitted count (including zero) and every category.
`Curated snippets, not complete source or test evidence` remains visible. Arithmetic validation does not verify completeness.
Every entry and relationship validates before any rich section mounts, including unselected hunks and mixed diagrams.


### WhatBreaks


WhatBreaks C has `id`, `type: "what-breaks"`, `title`, `reason`, `diagram`, `faults` and optional `initialFault`.
Its diagram has `kind: "flowchart"`, `source` and `description`; sequence faults are not supported.
Each fault has `id`, `node`, `title`, `severity`, `breaks`, `becomes` and `brokenRule`, plus optional `invariant` and `corpus`.
Fault IDs are unique per section and match `^[a-z][a-z0-9-]{0,63}$`.
Each fault names a distinct node matching `^[A-Za-z0-9_-]{1,64}$` in its sanitized diagram.
A supplied `initialFault` must name a fault; omission selects the first fault.
Every record, selection reference and target validates before any rich section mounts, including unselected faults.

| WhatBreaks boundary | Inclusive maximum or required range |
| --- | --- |
| Faults per section | 1–32 |
| Fault title | Nonblank, at most 512 UTF-8 bytes |
| Each consequence `breaks`, response delta `becomes` and `brokenRule` | Nonblank, at most 2 KiB |
| Each supplied `invariant` and `corpus` | Nonblank, at most 256 bytes |
| Diagram source | Nonblank, at most 8 KiB and 256 source lines |
| Diagram description | Nonblank, at most 2 KiB |

Severity is authored: `high` displays **High**, `med` displays **Watch**, and `low` displays **Low**.
The argument pairs the consequence and response delta with **The broken rule** beside them, above the flowchart.
Optional identifiers appear literally under **Cited invariant** and **Cited case**, never as links or verified test results.
Missing identifiers show `No invariant reference supplied` or `No case reference supplied`; blank supplied identifiers are invalid.
`Authored counterfactual — not executed fault injection or test results` stays visible.
No graph transformation, source access, response execution or consequence inference occurs.


### Lifecycle


Lifecycle B has `id`, `type: "lifecycle"`, `title`, `reason`, `states` and optional `readLabel`.
Each ordered state has `id`, `label`, `happened`, `readReturns`, optional `t` and optional `inWindow`.
State IDs are unique within their section and match `^[a-z][a-z0-9-]{0,63}$`; separate sections may reuse them.
The first supplied state starts selected. Array order is authoritative; timing hints never sort or schedule states.
Every state and relationship validates before any rich page mounts, including unselected states.

| Lifecycle boundary | Inclusive maximum or required range |
| --- | --- |
| States per section | 1–64 |
| State label and supplied read label | Nonblank, at most 512 UTF-8 bytes |
| Each completed-effect `happened` and `readReturns` | Nonblank, at most 2 KiB |
| Supplied display-only timing hint `t` | Nonblank, at most 64 bytes |

`readLabel` defaults to **A concurrent read right now returns** when omitted; blank supplied labels or hints are invalid.
`happened` states completed effects; `readReturns` separately states the concurrent-read result, including authored uncertainty.
A completed projection write does not guarantee every replica can read it.
**Authored completed effects and read results — not live responses or execution results** and
**Illustrative timings — not observed runtime measurements** stay visible, even without timing hints.

Only `inWindow: true` means inside; omitted and `false` mean outside.
A declared window forms one contiguous block followed by at least one outside state; it cannot reopen or end at the final state.
The range names its first and last inside states and the next outside state where it closes.
No true flags shows **No consistency window declared**, not an invented zero-duration window.
Outside-window membership never implies projection completion or a successful read.
The ledger retains all states, read results and explicit window statuses beside a text indication of the selected row.
Selected detail shows position/count, optional timing, window status, **What has happened** and the named read result.


### C4


C4 B has `id`, `type: "c4"`, `title`, `reason`, `mode`, `levels` and optional `initialLevel`.
Each level has `id`, `title`, `altitude` (`c1`, `c2`, `c3`), required `drills`, and mode-specific diagrams.
In `single`, every level supplies `diagram` and omits `before`/`after`.
In `diff`, every level omits `diagram` and supplies at least one of `before`/`after`; paired kinds must match.
Every diagram reuses the foundation's `DiagramData`, including narration, markers and optional initial pin.
Missing panes say **No prior diagram supplied** or **No resulting diagram supplied**, without invented wholesale changes.
Empty markers say **No change markers supplied**, not verified no-change.
**Authored architecture and change markers — not an inferred diff or verified source model** stays visible.

| C4 boundary | Inclusive maximum or required range |
| --- | --- |
| Levels per section | 1–16 |
| Drills per level | 0–32 |
| Level title | Nonblank, at most 512 UTF-8 bytes |
| Each diagram | Unchanged foundation source, narration, steps, marker and exact-edge bounds |

Level IDs and references match `^[a-z][a-z0-9-]{0,63}$`; IDs are unique within a section.
The first level is the only C1 root. A C1 drill targets C2; a C2 drill targets C3; C3 has no drills.
Every nonroot level has exactly one parent, though several drills from that parent may name the same child.
All levels must be reachable; missing children, cycles, self/root/skipped-altitude drills and duplicate pane/node drills are invalid.
A drill is `{ pane: "single" | "before" | "after", node, level }`, naming a present pane, exact node/sequence actor and child level.
Subgraph frames and sequence blocks are not drill targets. Omitted `initialLevel` selects the root; supplied values must exist.
All levels, diagrams, markers, pins and drills prepare before any rich page mounts, including hidden panes and unselected levels.

Stable architecture addresses name **section ID + level ID + pane role + FocusTarget**.
A node target names its exact ID; an edge names `{ from, to, label, occurrence }` with the exact emitted label and zero-based occurrence.
Equal IDs or endpoint pairs never imply a shared target across levels, panes or repeated edges.
These bindings are exercised by markers, pins and drills. Impact provides internal bound inspection,
not a new hash grammar or public selection API. Earlier C4 envelopes remain unchanged.


### EventModel


EventModel A has `id`, `type: "event-model"`, `title`, `reason`, `flows`, `scenarios` and optional `initialFlow`.
Each flow has `id`, `title`, a lightweight `diagram: { kind: "flowchart", source, description }` and `blocks`.
Each block has `node`, `title`, `role: "trigger" | "command" | "event" | "view"` and optional `coverage`.
Each coverage has required `scenarioIds` and `requirements`, plus optional `title`, `architecture`, `seams` and `cases`.
Each catalogue scenario has `id`, optional `name` and optional literal `body`; each requirement has `id` and optional `text`.

Flow IDs use the section-ID syntax and are unique within their section; block nodes are unique within each flow.
`initialFlow` names a supplied flow; omission selects the first. Every declared block resolves to its own sanitized node before mount.
All flows prepare atomically, including unselected flows and blocks without coverage. Only flowcharts are accepted here.
Role declarations neither generate relationships nor infer execution; undeclared nodes retain ordinary inspection with no supplied role.

| EventModel boundary | Inclusive maximum or required range |
| --- | --- |
| Flows per section / blocks per flow | 1–16 / 1–64 |
| Scenario catalogue / scenario references per coverage | 0–128 / 0–32 |
| Requirements / each architecture, seam or case list | 0–32 |
| Flow/block title, supplied coverage title or scenario name | Nonblank, at most 512 UTF-8 bytes |
| Scenario/requirement ID or reference identifier | Nonblank, at most 256 bytes |
| Supplied requirement text | Nonblank, at most 2 KiB |
| Supplied scenario body | Nonblank, at most 8 KiB and 256 lines |
| Diagram source / description | Unchanged 8 KiB, 256 lines / 2 KiB |

Scenario IDs are unique within their section and resolve there by exact equality, in each coverage record's cited order.
Duplicate scenario references, requirement IDs or identifiers within one reference category are invalid.
Shared scenarios across blocks/flows and equal identifiers in different reference categories remain valid.
Unused catalogue scenarios do not create inferred coverage. Scenario bodies preserve literal whitespace; no grammar is parsed or executed.
Missing bodies show **No scenario body supplied**; missing requirement text shows **No requirement text supplied**.
Absent coverage says **No coverage supplied for this block**; empty coverage shows zero counts and explicit empty-category notices.
Empty scenarios/requirements say **No scenarios supplied** / **No requirements supplied**.
Absent or empty reference lists say **No architecture references supplied**, **No interface references supplied** or **No case references supplied**.
References remain literal under **Cited architecture**, **Referenced interfaces** and **Cited cases**, even when URL-like or unresolved.
**Authored coverage references — not executed scenarios or passing tests** remains visible below the flow.


### Contracts


Contracts C has `id`, `type: "contracts"`, `title`, `reason`, `contracts` and optional `initialContract`.
Each interface has required `id`, `title`, `spec` and optional `kind`, `summary`, `provenance`.
Contract IDs are nonblank inert strings, unique by exact equality within each section, not a mandatory document-family grammar.
`initialContract` names one local contract exactly; omission selects the first. Unknown choices are invalid, never silently defaulted.
Supplied order controls the selector order. Different sections may reuse IDs and provenance without sharing identity or state.
Every record validates before mount, including unselected bodies; mixed diagram preparation stays atomic.

| Contracts boundary | Inclusive maximum or required range |
| --- | --- |
| Contracts per section | 1–32 |
| Contract ID, supplied initial choice, kind and each provenance identifier | Nonblank, at most 256 UTF-8 bytes |
| Interface title | Nonblank, at most 512 bytes |
| Supplied summary | Nonblank, at most 2 KiB |
| Supplied provenance list | 0–32 identifiers; no exact duplicates within one contract |
| Required `spec` body | Nonblank, at most 8 KiB and 256 CRLF/CR/LF-separated lines |

`kind` retains authored spelling and case. Only allowlisted aliases select cosmetic syntax highlighting; no kind executes or validates the source.
Omitted kind/summary show **No interface kind supplied** and **No interface summary supplied**.
**Supplied provenance** retains supplied order; absent and empty lists show **No provenance supplied**.
Blank supplied metadata is invalid. URL-like identifiers stay plain text, not links or source lookups.
**What this interface does** explains the connection but never replaces, trims or rewrites `spec`.
**Exact source body** is complete literal text, optionally highlighted through safe syntax spans, without execution or grammar validation.
Leading/trailing whitespace, tabs, blank lines and mixed line endings remain supplied characters, not an upstream-file encoding guarantee.
**Supplied interface source — not fetched, executed or verified against a repository** stays visible in every state.
Displaying source implies no compatibility verdict, source freshness, passing test or mandatory contracts document family.


### Corpus


Corpus D has `id`, `type: "corpus"`, `title`, `reason`, `entries` and optional `initialCase`.
Each case requires `id`, `title`, `provenance`, `dependency`, `normalise`, `input` and `expected`; `maps` is optional.
Case IDs are nonblank inert strings, unique by exact equality within each section; no corpus document family is mandatory.
`initialCase` must name a local case exactly. Omission selects the first entry; an unknown choice is invalid.
Supplied order controls choices. Equal IDs across sections and shared scenario references do not share state.
All cases validate before mount, including unselected ones, and mixed diagram preparation remains atomic.

| Corpus boundary | Inclusive maximum or required range |
| --- | --- |
| Cases per section | 1–32 |
| Case ID, supplied initial choice and each scenario reference | Nonblank, at most 256 UTF-8 bytes |
| Case title and supplied dependency pin | Nonblank, at most 512 bytes |
| Provenance and normalization policy | Nonblank, at most 2 KiB each |
| Scenario references per case | 0–32; no exact duplicates within one case |
| Each required input/expected string | 0–8 KiB and at most 256 CRLF/CR/LF-separated lines |

**Supplied provenance**, **Supplied dependency pin** and **Declared normalization** are authored text, not verified evidence.
The renderer never applies `normalise`, resolves a pin, captures, replays, executes or compares a case.
**Cited scenarios** preserves `maps` spelling/order as inert text; absent and empty lists show **No scenario references supplied**.
Unresolved and URL-like references remain plain text, not links or cross-lens targets.
**Input** and **Expected** preserve complete decoded strings, whitespace, tabs, blank lines and mixed line endings without formatting or truncation.
Local syntax colouring makes a best-effort guess limited to JSON, YAML and HTML. It uses safe text spans, never HTML insertion, syntax validation, execution or a request. Other text and tokenizer failures retain the exact plain-text fallback.
This is not an original-file encoding or binary-byte guarantee. Expected text is an authored expectation, never actual output or a passing replay.
Zero-length strings show **Empty input (0 UTF-8 bytes)** or **Empty expected (0 UTF-8 bytes)** beside their empty regions.
Whitespace-only strings remain text; missing fields are invalid. No filenames or fixture roles are inferred from the text.
**Supplied case material — not captured, replayed or verified by this renderer** stays visible in every state.


### Impact


Impact B has `id`, `type: "impact"`, `title`, `reason`, `architecture` and `hits`.
Each hit has `id`, `title`, `ring: "changed" | "downstream"`, `level`, `pane`, `target` and optional `parent`, `why`.
`architecture` names an existing C4 section; level, present pane and node/exact-edge targets must resolve there, without fallback.
Hit IDs and architecture/level/parent references match `^[a-z][a-z0-9-]{0,63}$`. Hit IDs are unique locally.
Parents form an acyclic forest; forward references and repeated target addresses are valid. Roots and siblings retain supplied order in depth-first display.
The table shows **Node**, **Relation** and **Altitude**, complete parent ancestry and every exact address.
`changed` means **Directly changed**; `downstream` means **Downstream**, independently of parentage and C4 marker status.
Missing `why` shows **No rationale supplied**; supplied blank rationale is invalid.
**Authored impact dependencies — not discovered propagation or a risk score** stays visible.

| Impact boundary | Inclusive maximum or required range |
| --- | --- |
| Hits per section / maximum parent-chain depth | 1–64 / 64 |
| Hit title | Nonblank, at most 512 UTF-8 bytes |
| Supplied rationale | Nonblank, at most 2 KiB |
| Exact-edge label / occurrence | At most 2 KiB (empty valid) / integer 0–127 |

All relationships validate without the engine. Mount prepares every bound diagram and resolves every target before `createRoot`.
Invalid forest/section/level/pane relationships return `invalid-data`; missing rendered targets return mount-only `invalid-target`.
Identity resolution does not verify dependencies, source freshness or risk. Earlier exact envelopes remain unchanged.


### Telemetry


Telemetry C has `id`, `type: "telemetry"`, `title`, `reason`, `architecture`, `operationalChange` and `entries`.
Each entry requires `id`, `signal: "metric" | "span" | "log"`, `name`, `status: "added" | "changed" | "removed"`,
`level`, `pane: "single" | "before" | "after"` and `node`; `kind`, `powers` and `affects` are optional.
Entry IDs are unique locally; architecture, level and entry IDs match `^[a-z][a-z0-9-]{0,63}$`.
Node IDs retain `^[A-Za-z0-9_-]{1,64}$`. Equal names or emitters with different IDs remain distinct records.
`architecture` must name an existing C4 section; every level, present pane and exact sanitized node/sequence actor resolves there.
No edge, approximate label, absent-pane fallback or inferred emitter is accepted, even for Removed signals.
All relationships validate without the engine; mount resolves every emitter before `createRoot`, including hidden/unselected panes.
Missing rendered emitters return mount-only `invalid-target` at the entry's `/node`; bad relationships return `invalid-data`.

| Telemetry boundary | Inclusive maximum or required range |
| --- | --- |
| Entries per section | 1–64 |
| Signal name | Nonblank, at most 512 UTF-8 bytes |
| Operational change and each supplied use/consequence | Nonblank, at most 2 KiB each |
| Bound C4 diagrams and global envelope | Unchanged limits |

Groups appear **Metric**, **Span**, **Log field**, omitting empty groups and retaining relative input order within each.
Every entry shows its name, ID, type, Added/Changed/Removed status and complete bound address with derived altitude/ancestry.
Only metrics permit `kind: "counter" | "histogram" | "gauge" | "summary"`; omission shows **No metric kind supplied**.
Spans/log fields omit that notice. Optional `powers` appears under **Operational use**, or **No operational use supplied**.
Optional `affects` appears under **Dashboard or alert consequence**, or **No dashboard or alert consequence supplied**.
Blank supplied values are invalid; missing content never implies no impact, a healthy deployment or verified alert wiring.
Required **Operational change** prose sits beside the signal groups and stacks below on narrow screens.
**Authored signal definitions — no live measurements or telemetry collection** remains visible.
No live values, measurements, queries, execution, emission, uploads, requests, storage or analytics are added.


### Tradeoffs


Tradeoffs A has `id`, `type: "tradeoffs"`, `title`, `reason`, `architecture`, `decisions` and optional `initialDecision`.
Each decision requires `id`, `question`, `source`, `seam`, `level`, `pane`, exact `target`, `options` and `tradeoff`; `references` is optional.
Each option requires `label`, `verdict: "chosen" | "rejected"` and `why`. Exactly one option is chosen; supplied order is never ranked or sorted.
Decision IDs are unique locally; decision/architecture/level/initial IDs match `^[a-z][a-z0-9-]{0,63}$`.
`initialDecision` must name a decision; omission selects the first. Equal IDs across sections never share state.
Every decision binds to an existing C4 level and present single/before/after pane, with an exact node or edge FocusTarget.
All relationships validate before mount; all hidden/unselected targets resolve against prepared sanitized maps before `createRoot`.
Invalid relationships return `invalid-data`; missing targets return mount-only `invalid-target` at the decision's `/target/node` or `/target/edge`.
Earlier exact envelopes, schemas and fixtures remain unchanged.

| Tradeoffs boundary | Inclusive maximum or required range |
| --- | --- |
| Decisions per section / options per decision | 1–32 / 1–16 |
| Question, seam and option label | Nonblank, at most 512 UTF-8 bytes each |
| Source and each reference identifier | Nonblank, at most 256 bytes |
| Each option rationale and accepted cost | Nonblank, at most 2 KiB each |
| Optional references per decision | 0–32; no exact duplicates within a decision |
| Exact-edge label / occurrence | At most 2 KiB (empty valid) / integer 0–127 |

**Decision source** and **References** remain inert identifiers, even when URL-like; nothing is fetched or independently verified.
Each textual Chosen/Rejected option shows **Why chosen** or **Why not**. Required `tradeoff` appears as **The trade-off** in a separate warm-toned callout.
A chosen-only decision says **No rejected alternatives supplied**, not that alternatives do not exist.
Absent/empty references say **No cited references supplied**; blank supplied fields are invalid.
**Authored decision arguments — not independently verified alternatives or references** stays visible.
**Affected connection** shows the supplied `seam` label and a complete address derived from bound C4, including altitude and ancestry.
Local decision selection, Back and Next never drive architecture, change hash/history, open a modal or automatically scroll/focus.
Only **Inspect affected connection** lends the same production C4 through the existing inspection owner.


### DataFlow


DataFlow C has `id`, `type: "data-flow"`, `title`, `reason`, `architecture` and `crossings`.
Each crossing requires `id`, `level`, `pane`, exact `edge`, `data`, `classification`, `control`; `note` is optional.
`architecture` names an existing C4 section, even later in the page. Each level and present single/before/after pane must exist there.
`edge` reuses `{ from, to, label, occurrence }`; endpoints, exact emitted label and zero-based occurrence must all resolve.
Node-only targets, endpoint-only matching, absent-pane fallback and independently authored graphs are not accepted.
Crossing IDs are unique locally; crossing/architecture/level IDs match `^[a-z][a-z0-9-]{0,63}$`.
Repeated addresses and equal local IDs across sections remain separate authored records, not shared state.
All relationships validate first; every crossing resolves against prepared sanitized C4 maps before `createRoot`, including hidden panes.
Invalid identity/section/level/pane relationships return `invalid-data`; unresolved edges return mount-only `invalid-target` at `/sections/<index>/crossings/<index>/edge`.

| DataFlow boundary | Inclusive maximum or required range |
| --- | --- |
| Crossings per section | 1–64 |
| Carried data and named control | Nonblank, at most 512 UTF-8 bytes each |
| Supplied crossing note | Nonblank, at most 2 KiB |
| Exact-edge label / occurrence | At most 2 KiB (empty valid) / integer 0–127 |

The ordered notes show **Untrusted**, **Token**, **PII**, **Internal**, retaining input order within each class.
Each note preserves its ID, **From**, **To**, **Carried data**, **Named control**, note and complete C4 address/ancestry.
Omitted notes say **No crossing note supplied**; supplied blank notes are invalid.
**PII: personally identifiable information** and **Authored data classifications and controls — not verified security enforcement** remain visible.
The **Control** margin explains that neither classification nor a named control proves enforcement.
Classification is authored, not sensitive-data detection, a security verdict, risk score or exhaustive crossing inventory.
Control/data/note text, including URL-like and markup text, stays inert; nothing is executed, fetched or checked for enforcement.
Reading, hovering or focusing notes never drives C4; only **Inspect bound C4 edge** opens the existing same-reader inspection.


### Tour


The stable contract includes `tour` with `id`, `title`, `reason` and `chapters`.
Each chapter requires a local `id`, nonblank `title`, rationale `body` and `target`; `hint` is optional nonblank text.
IDs and targets match `^[a-z][a-z0-9-]{0,63}$`. Chapter IDs are unique within each Tour, not across Tours.
A target names any existing non-Tour section in the page, including later sections and repeated targets.
Reserved/missing IDs, self-targets and other Tours are invalid; duplicate IDs fail at the later chapter `/id`, bad targets at `/target`.
There are 1–32 chapters per Tour; title/hint allow 512 UTF-8 bytes each, body 2 KiB. Bounds are inclusive.
The sixteen-section and 256 KiB envelope limits apply. Every target prepares before rich mount, not just chapter one.
No authored selector, selection key, HTML, callback, targetless chapter, initial chapter or auto-start flag exists.
All narration stays literal, including markup and URL-like prose. Omitted hints render no empty chip.

**Start** alone opens chapter one; mount, navigation and hashes never start or advance it.
The Tour is not modal. A fixed bottom dock shows **Back**, `n / total · chapter title` with a progress bar, **Next** and **Exit**.
Each chapter collapses the other cards, opens its target card and scrolls to it. A chapter note with the title, optional hint and complete narration sits at the top of that card.
Back is disabled on the first chapter. On the last chapter Next becomes **Finish**, which exits like Exit; nothing declares review completion.
**Manual authored walkthrough — not execution, verification or review approval** remains visible on the launcher.
Readers stay in their cards, so selections, source disclosures, internal text scroll and viewer viewports are the page's own state.
Tour does not choose target-reader selections. Contracts/Corpus explicit selection changes retain their original current-body reset rules.

Existing Impact/Telemetry/Tradeoffs/DataFlow launchers open the bound C4 in the normal inspection dialog over the page and dock.
Closing the inspection returns to the same chapter and launcher focus.
Nested **Full view** closes first with Close/Escape, restoring the prior inline viewport while retaining the selected level, parent context and launcher focus.
Escape exits the Tour unless a dialog or Compare owns it first.
Left Arrow and Page Up go Back; Right Arrow and Page Down go Next, or Finish on the last chapter.
They act only while focus is on the page, a button, a link or a summary, so fields, selects,
scrollable panes and the diagram canvas keep their own keys. Exit hides the dock and returns focus to the Start launcher.
No local Tour action writes history, requests, storage, analytics or completion state. Unmount clears owned session/resources.


### Recovery

The stable contract includes diagram-free `recovery`. All eighteen lenses share the sixteen-section cap, optional pins and CodeLens maxLines. Undeclared versions return `unsupported-version`; unknown string lenses return `unsupported-lens`. Missing or non-string discriminators return `invalid-data`. No implicit migration is provided.

Every Recovery object rejects unknown fields. All fields below are required unless marked optional:

| Record | Fields |
| --- | --- |
| Section | `id`, `type: "recovery"`, `title`, `reason`, `operations`; optional `initialOperation` |
| Operation | `id`, `title`, `intent`, `identity`, `points` |
| Point | `id`, `title`, `attempt`, `observation`, `known`, `unknown`, `retained`, `alternatives`, `prohibited`; optional `budget` |
| Alternative | `id`, `action`, `when`, `owner`, `outcome`, `result` |

Local IDs and `initialOperation` match `^[a-z][a-z0-9-]{0,63}$`. Operation IDs are unique within their section, point IDs within their operation and alternative IDs within their point. Equal IDs under different owners or sections are independent. Repeated prose is valid. `initialOperation` names an operation in that section; omission selects the first. Local IDs are relationship keys, not DOM IDs, selectors, links or reference targets.

| Recovery boundary | Inclusive range or maximum |
| --- | --- |
| Operations per section | 1–16 |
| Points per operation | 1–32 |
| Alternatives per point | 1–16 |
| Each `known`, `unknown`, `retained`, `prohibited` list | 0–32 |
| Operation/point title, action and owner | Nonblank, at most 512 UTF-8 bytes each |
| Intent, identity, attempt, each list item, supplied budget, condition and result | Nonblank, at most 2048 UTF-8 bytes each |
| Global input / sections | 256 KiB / 1–16; maxima need not fit simultaneously |

`observation` is exactly `unknown`, `confirmed-success`, `confirmed-failure` or `not-attempted`. It describes supplied knowledge about the attempt, not the outcome of an alternative. A lost Complete response can remain `unknown`; the renderer never converts uncertainty into failure.

`intent` describes logical work; `attempt` describes an attempt or interruption within that work. `identity` describes what stays pinned, not verified uniqueness, provider idempotency or an external effect. `known` and `unknown` state the evidence boundary. `retained` names authored durable intent, identities, uploads or carried state; storage is not inspected.

`budget` is literal prose, for example **2 of 5 attempts used; original deadline retained**. No arithmetic, deadlines or policy are inferred. Omission shows **No retry budget supplied**; a supplied blank budget is invalid. Each empty list shows **None supplied**. Empty evidence does not prove absence, and empty prohibited effects do not imply permission.

`alternatives` are a semantic list of conditional authored possibilities, never recommendations or executable controls. Each shows its ID, action, condition `when`, accountable `owner`, `outcome` and `result` together. Outcomes are exactly:

- `continue`: result names the resumed work, not a link for the renderer to follow.
- `complete`: authored completion of the named work, not a verified execution result.
- `refuse`: refusal of the named action, not an inferred wider business failure.
- `skip`: skips only the named action without changing the wider business outcome.
- `park`: remains unresolved; the owner and result name who can act and what is pending. Name a manual-recovery park explicitly when applicable.

All new text is literal, including Markdown-like links, markup, closing-script strings and cited contract/scenario names. No RichText evaluation, fetched reference, cross-lens resolution or coverage claim applies. **Authored recovery model — not live status or an execution result** stays visible.

Operation tabs use native buttons in supplied order, labelled by ID and title with `aria-pressed`. Tab focuses without selecting; Enter, Space or a click changes only this card's operation. All its points remain in supplied order below intent and pinned identity. Evidence sits beside details at wide widths and stacks in reading order in narrow/comparison panes; nothing is truncated. Collapse/reopen, navigation, scheme changes, existing Compare and section-level Tour retain independent selections. Teardown is idempotent; remount starts from authored defaults. No point selector, retry button, timer, provider polling, scheduling, local-target Tour, new binding, callback or C4 peek is added.

The brief reports **N operations**, a neutral **Recovery** tag and Base16 hue `09`; it does not count recovered operations. Existing pins and Tours address only the section card. All shapes validate first, then local relationships in supplied order before page-wide Tour/pin checks. Later duplicate IDs fail at `/sections/<index>/operations/<index>/.../id`; an unknown default fails at the section's `/initialOperation`, always using input array indices, never authored IDs. Issues remain value-free `{ code, path }`. Invalid content rejects the whole rich page. Mixed diagrams and targets retain their existing mount-only checks.

Keep every inactive operation, point, field, alternative, prohibition, absence label and notice in the trusted fallback, alongside every mixed section and exact reference. Escape every field and embedded JSON; do not leave dead selection or execution controls. Retain the release's CSP, SRI, anonymous asset loads and no-referrer policy. The renderer does not fetch content, write storage or send analytics; trusted asset loading remains the host's responsibility.

The complete `1.0.0` example is synthetic, not execution evidence. Synthetic outcomes are still authored claims.

### Decision Table

The stable contract includes diagram-free `decision-table`. All eighteen lenses share the sixteen-section cap, optional pins and CodeLens maxLines. Undeclared versions return `unsupported-version`; unknown string lenses return `unsupported-lens`. Missing or non-string discriminators return `invalid-data`. No implicit migration is provided.

All records are closed: null records and unknown fields are invalid. Fields below are required except those marked optional:

| Record | Fields |
| --- | --- |
| DecisionTableSection | `id`, `type: "decision-table"`, `title`, `reason`, `tables`; optional `initialTable` |
| PolicyTable | `id`, `title`, `policy: "first-match"` or `"unique"`, `conditions`, `rules`, `unmatched`, `cases`; optional `initialCase` |
| PolicyCondition | `id`, `label` |
| PolicyRule | `id`, `title`, `cells`, `result` |
| PolicyResult | `outcome`, `reason`, `allowed`, `prohibited` |
| PolicyCase | `id`, `title`, `cells`, `matches`, `explanation`; optional `selectedRule`, `references` |
| PolicyCell | Exactly `{ kind: "literal", text: string }`, `{ kind: "any" }`, `{ kind: "absent" }`, `{ kind: "null" }` or `{ kind: "unknown" }` |

`any` is allowed only in rule cells and displays **Any (wildcard)**. The other tagged nonliteral cells display **Absent**, **Null** and **Unknown**. They carry no text and are neither empty strings nor wildcards. Literal text is exact display content, never an expression or typed value for the renderer to compare. Literal `*` and literal `null` remain **Literal**, not tagged kinds. Empty literal displays **Empty literal (0 UTF-8 bytes)**; whitespace displays **Whitespace literal**; the literal region preserves exact text in both cases.

| Decision Table boundary | Inclusive range or maximum |
| --- | --- |
| Tables per section | 1–16 |
| Conditions per table | 1–12 |
| Rules per table / cases per table | 1–32 each |
| Cells per rule / case | 1–12 each; exactly the condition count in condition order |
| Declared matches per case | 0–32; unique permits at most one |
| Each allowed / prohibited effect list | 0–32 narration entries |
| References per case | 0–32 title entries |
| Literal text | 0–2048 UTF-8 bytes, including empty and whitespace |
| Title, condition label, outcome and reference entry | Nonblank, at most 512 UTF-8 bytes |
| Section/result reason, explanation and each effect entry | Nonblank, at most 2048 UTF-8 bytes |
| Global input / sections | 256 KiB / 1–16; not all maxima need fit simultaneously |

Local IDs/defaults match `^[a-z][a-z0-9-]{0,63}$`. Table IDs are unique within the section; condition, rule and case IDs are unique within their respective table collections. Equal IDs across collections, tables or sections are valid. Repeated titles are valid. `initialTable` and each `initialCase` resolve inside their owner; omission selects the first record. IDs do not become RefTargets, DOM selectors, hashes or local Tour addresses.

`matches` lists the author's known rule IDs, once each, in supplied rule order. With first-match, the winner must be the first declared match. With unique, at most one match may be declared and that match must win. Any nonempty list requires `selectedRule`; zero matches forbids it and uses the table's explicit `unmatched` result. Unmatched is not an implicit last rule. These checks establish only declaration consistency, not actual matches, exclusivity, authorization or coverage. The renderer never evaluates predicates, compares literals, detects overlaps or infers missing cases.

Every rule's result includes its outcome, reason and both effect lists; the explicit unmatched result always stays readable. Empty effects show **None supplied**, never a permission or guarantee. Omitted/empty references show **No references supplied**. References are literal scenario/contract labels, not fetched sources, cross-lens bindings or passing-test claims. Every new string renders literally without HTML, Markdown, RichText linking or automatic URLs. **Authored matches and outcomes — not evaluated or exhaustive** remains above the matrix.

The semantic matrix has a caption, condition column headers and rule row headers, every rule in authored order and textual **Declared match**, **Declared winner** or **Not declared matched** labels. Below it, the selected case shows every input, explanation, matches, winner or **Unmatched**, complete result and references. Native table/case buttons show complete ID/title names and `aria-pressed`; focus alone does not select. Each table retains its latest case when switching away and back; sections remain independent across collapse/navigation, Compare, schemes and section-level Tour. Fresh remount uses authored defaults. The labelled keyboard-focusable matrix region alone scrolls horizontally at narrow/comparison widths; all prose wraps without ellipsis.

After schema checks, validate local IDs, defaults, cardinality and declarations in supplied order before existing page-wide Tour/pin checks. All issue paths use input array indices, starting `/sections/<index>/tables/<index>`, never authored values:

- Later duplicate table/condition/rule/case IDs: the later `/id`.
- Invalid default: `/initialTable` or the owning table's `/initialCase`.
- Wrong cell cardinality: the rule/case's `/cells`.
- Unknown, duplicate or out-of-order match: the offending `/cases/<index>/matches/<index>`.
- Second unique match: `/cases/<index>/matches/1`.
- Required missing or incorrect winner, or supplied winner without matches: `/cases/<index>/selectedRule`.

Failure issues remain value-free `{ code, path }`. Shape/lower-bound/relationship errors are `invalid-data`; exceeded collection/text/global maxima are `limit-exceeded`; malformed JSON is `invalid-json`. Unregistered strings retain `unsupported-version`/`unsupported-lens`. Decision Table skips diagram-only validation/preparation, including mixed documents. Mixed diagrams/targets retain mount-only `invalid-diagram`/`invalid-target`; occupied containers or unexpected defects return `render-failed`. Any failure rejects the whole rich page, leaving complete separate fallback.

Fallback must contain every inactive table, condition, ordered rule/cell/result/effect, case/input/explanation/match/winner/reference, explicit unmatched branch, absence label and authored notice, alongside all mixed content. Omit dead controls. Escape every field and embedded JSON; keep markup and closing-script strings inert. Preserve CSP, SRI, anonymous/no-referrer asset loads and existing failure behavior. No content request, storage, analytics, evaluation, credential check, callback, new viewer, Tour protocol or local binding is added. The brief counts **N tables**, with a neutral **Decision Table** tag and Base16 hue `0C`.

The `1.0.0` release example is synthetic, not audited service policy.

### Identity

The stable contract includes diagram-free `identity`. All eighteen lenses share the sixteen-section cap, optional pins and CodeLens maxLines. Undeclared versions return `unsupported-version`; unknown string lenses return `unsupported-lens`. Missing or non-string discriminators return `invalid-data`. No implicit migration is provided.

Every record is closed; null records and unknown properties are invalid. All fields below are required except those marked optional:

| Record | Fields |
| --- | --- |
| IdentitySection | `id`, `type: "identity"`, `title`, `reason`, `identities`; optional `initialIdentity` |
| IdentityModel | `id`, `title`, `role`, `scope`, `encoding`, `normalization`, `ingredients`, `pairs`, `lifetime`; optional `namespace`, `initialPair`, `evolution` |
| IdentityIngredient | `id`, `label`, `meaning` |
| IdentityPair | `id`, `title`, `left`, `right`, `relation: "same"`, `"different"` or `"unknown"`, `reason` |
| IdentitySample | `label`, `values`; optional `scheme`, `encoded`, `key` |
| IdentitySampleValue | `raw`; optional `normalized` |
| IdentityValue | Exactly `{ kind: "literal", text: string }`, `{ kind: "missing" }` or `{ kind: "null" }` |
| IdentityLifetime | `event`, `action: "reuse"`, `"replace"`, `"retire"` or `"unknown"`, `reason` |
| IdentityEvolution | `before`, `after`, `existingRecords` |

`role` names an authored purpose, not a closed domain taxonomy. `scope` states where declared equivalence applies; equal keys in different roles, namespaces or lifetimes never imply the same work or effect. Omitted namespace shows **No namespace supplied**, not an invented default. `encoding` describes the supplied format; `normalization` describes the policy, including an explicit no-normalization statement where relevant. Neither is implemented by the renderer.

| Identity boundary | Inclusive range or maximum |
| --- | --- |
| Identities per section | 1–16 |
| Ingredients per identity / pairs per identity / lifetime entries per identity | 1–32 each |
| Values per sample | 1–32; exactly the ingredient count, in ingredient order |
| Literal value text | 0–2048 UTF-8 bytes |
| Supplied encoded form / key | 0–8192 UTF-8 bytes each |
| Title, role, ingredient label, sample label, supplied scheme label and lifetime event | Nonblank, at most 512 UTF-8 bytes each |
| Section/pair/lifetime reason, scope, namespace, encoding, normalization, meaning and evolution prose | Nonblank, at most 2048 UTF-8 bytes each |
| Global input / sections | 256 KiB / 1–16; local maxima need not fit simultaneously |

Local keys/defaults match `^[a-z][a-z0-9-]{0,63}$`. Identity IDs are unique within their section; ingredient IDs and pair IDs within their respective identity collections. Equal local IDs across owners/sections, duplicate titles and repeated supplied encoded forms/keys are valid. No uniqueness constraint applies to the displayed identity itself. `initialIdentity` resolves inside its section; `initialPair` inside its identity. Omission selects the first record. These IDs never become RefTargets, hashes, selectors or local Tour addresses.

Every sample supplies a value slot for every ingredient, even when `raw` is `{ kind: "missing" }`. Raw and normalized values are independently labelled. Omitted normalized values show **No normalized value supplied**, never a copy of raw. Tagged **Missing** and **Null** carry no text. Literal `null`, `undefined`, numbers as text, markup, empty strings and whitespace retain every supplied character. Empty text shows **Empty literal (0 UTF-8 bytes)**; whitespace is labelled **Whitespace literal** without being trimmed. Whitespace-preserving wrapping regions remain present even when empty.

Scheme labels are literal, not version selectors. Omission shows **No scheme label supplied**; a supplied label cannot be blank. `encoded` and `key` are optional exact strings, not computed output. Omission shows **No encoded form supplied** / **No key supplied**. Empty supplied strings remain valid and show **Empty encoded form (0 UTF-8 bytes)** / **Empty key (0 UTF-8 bytes)** beside the literal regions. Missing, tagged null, literal null, empty and omitted optional text are not interchangeable.

Relations display **Declared same identity**, **Declared different identities** and **Identity relation unknown**, with the author's reason. The declaration concerns only that role within its stated scope; it is not general entity equality, verified key equality, collision freedom or remote-effect deduplication. Equal supplied keys never upgrade unknown to same. The renderer does not generate, hash, normalize, look up, execute, compare for equality or verify uniqueness. **Authored identity relationships — no keys generated or uniqueness verified** stays visible.

The lifetime ledger preserves every event in supplied order. Each entry shows action and reason together: reuse, replacement, retirement or unknown handling. Replacement names fresh work, never automatic reuse of historical receipts. Retry, restart and continuation policies remain authored prose, not a state machine, deadline calculation or execution assertion. Evolution compares **Before**, **After** and **Existing records** consequences; omission shows **No scheme change supplied**. It supplies no operational rollout, rollback or compatibility verdict. All new text is literal, not RichText, Markdown, HTML, executable links or fetched references.

Native identity/pair Button/TabRow selectors use full ID/title names and `aria-pressed`. Focus/hover alone does not select. Each identity retains its latest pair independently when switching away/back; sections remain independent through collapse, navigation, schemes, existing Compare, pin and section-level Tour. Fresh remount restores authored defaults; cleanup remains idempotent. Role/scope/namespace/encoding/normalization/evolution precede the selected pair. Relation and reason precede labelled left/right samples with every ingredient ID/label/meaning, raw and normalized value, scheme, encoded form and key. Samples sit side by side at wide widths and stack left then right at narrow/comparison widths with ingredient associations repeated. Full literal text wraps without clipping or ellipsis. The complete lifetime ledger follows without another selector.

All shapes validate before local relationships in supplied order, then existing page-wide pin/Tour checks. Paths use input array indices under `/sections/<index>/identities/<index>/...`, never supplied IDs, with standard JSON Pointer escaping:

- Later duplicate identity/ingredient/pair ID: the later `/id`.
- Unknown default: its `/initialIdentity` or owning `/initialPair`.
- Wrong sample cardinality: `/pairs/<index>/left/values` or `/right/values`, even when every value shape is valid.

Shapes, nulls, bad keys/enums, blank prose, empty required arrays and bad relationships return value-free `invalid-data`. Exceeded text/collection/global maxima return `limit-exceeded`; malformed JSON returns `invalid-json`. Distinct diagram-free validation/preparation/render/fallback arms precede historical before/after fallthrough. Mixed invalid diagrams/targets retain mount-only `invalid-diagram`/`invalid-target`; occupied containers and unexpected defects return `render-failed`. Every failure refuses the entire rich page without replacing the host fallback. No new callback, option, local binding, target protocol or viewer behavior is added. Briefs count **N identities**, with neutral **Identity** tag and Base16 hue `0B`.

Complete fallback must enumerate every inactive identity, ingredient, pair, sample, raw/normalized value, supplied/omitted/empty scheme/encoded/key distinction, lifetime/evolution field, absence label and authored notice, alongside all mixed sections and references. Omitted namespace, scheme and evolution have notices; empty supplied namespace/scheme is invalid, not an empty-value state. Escape every field and embedded JSON, including markup and closing-script text, and omit dead controls. Preserve CSP, SRI, anonymous/no-referrer asset loads and all release safety policies. The renderer performs no content requests, storage writes or analytics; host asset loading stays separate.

The complete `1.0.0` release example is synthetic, not verified identity execution.

### Rollout

The stable contract includes diagram-free `rollout`. All eighteen lenses share the sixteen-section cap, optional pins and CodeLens maxLines. Undeclared versions return `unsupported-version`; unknown string lenses return `unsupported-lens`. Missing or non-string discriminators return `invalid-data`. No implicit migration is provided.

All records reject unknown properties. Every field is required unless marked optional. `Key` is `^[a-z][a-z0-9-]{0,63}$`; `Title` and `Narration` are nonblank literal strings of at most 512 and 2048 UTF-8 bytes respectively.

| Record | Fields |
| --- | --- |
| RolloutSection | Foundation `id`, `type: "rollout"`, `title: Title`, `reason: Narration`, `rollouts: RolloutModel[]`; optional `initialRollout: Key` |
| RolloutModel | `id: Key`, `title: Title`, `phases: RolloutPhase[]`, `combinations: RolloutCombination[]`, `releaseTogether: RolloutReleaseGroup[]`; optional `initialPhase: Key` |
| RolloutPhase | `id: Key`, `title: Title`, `intent: Narration`, `components: RolloutComponent[]`, `gates: RolloutGate[]`, `existingWork: Narration`, `newWork: Narration`, `verification: RolloutVerification[]`, `rollback: RolloutRollback` |
| RolloutComponent | `component: Title`, `version: Title`, `settings: Narration` |
| RolloutGate | `condition: Narration`, `assessment: "met"`, `"unmet"` or `"unknown"`; optional `evidence: RolloutEvidence` |
| RolloutVerification | `check: Narration`; optional `evidence: RolloutEvidence` |
| RolloutEvidence | `description: Narration`, `source: Narration` |
| RolloutRollback | `mode: "switch-back"`, `"repair-required"`, `"unavailable"` or `"unknown"`, `action: Narration`, `existingWork: Narration`, `newWork: Narration`, `boundary: Narration` |
| RolloutCombination | `id: Key`, `title: Title`, `components: RolloutComponent[]`, `compatibility: "compatible"`, `"incompatible"` or `"unknown"`, `reason: Narration` |
| RolloutReleaseGroup | `members: Title[]`, `reason: Narration` |

| Rollout collection | Inclusive range |
| --- | --- |
| Rollouts per section | 1–16 |
| Phases per rollout | 1–32 |
| Components per phase or combination | 1–16 |
| Gates / verification checks per phase | 0–32 each |
| Combinations per rollout | 1–32 |
| Release groups per rollout | 0–16 |
| Members per release group | 2–16 |

The global 256 KiB UTF-8 cap applies before parsing; maxima need not fit simultaneously. Data is never truncated. Rollout IDs are unique within their section; phase and combination IDs are unique within their respective rollout collections. `initialRollout` and each `initialPhase` resolve locally; omission selects the first record. A repeated release-group member is invalid by exact string equality, without trimming, case-folding or alias resolution. Repeated names across groups, component lists or sections are independent descriptions, not registry references.

Validate shapes before local relationships in supplied order, then preserve page-wide pin/Tour checks. Later duplicate rollout/phase/combination IDs fail at their later `/id`; unknown defaults fail at their fields; repeated members at the later `/releaseTogether/<index>/members/<index>`. Paths use input indices under `/sections/<index>/rollouts/<index>`, never supplied values, with standard JSON Pointer escaping. Invalid shapes, keys, enums, blank prose, minima and relationships return closed value-free `invalid-data`; exceeded byte/collection/global maxima return `limit-exceeded`. Malformed JSON returns `invalid-json`. Distinct diagram-free validation/preparation/page/fallback dispatch avoids historical TraceDiff fallthrough. Mixed diagram/target failures remain atomic `invalid-diagram`/`invalid-target`; occupied containers or unexpected defects return `render-failed` and leave complete host fallback intact.

Phases are ordered authored snapshots, not observed progress, safe deployment steps or a branching planner. Ordinals show supplied reading order only. Initial selection is not deployed state. Native Button/TabRow controls display full rollout/phase ID and title with `aria-pressed`; every phase remains selectable with unmet or unknown gates. Focus/hover alone changes nothing. Each rollout retains its own phase when switching away/back; normal cards preserve choices through collapse, navigation, schemes, pin/Compare and section-level Tour. Fresh remount restores defaults; cleanup remains idempotent. Local IDs never become RefTargets, local hashes or local-target Tour bindings.

The selected snapshot shows intent, every component/version/settings description, gates, **Existing work**, **New work**, **Planned verification** and **Rollback**. Gates display **Author says met**, **Author says unmet** or **Unknown**. Evidence sits beneath its gate/check with separate **Supplied evidence** and **Supplied source** labels; omission displays **No evidence supplied**. A check is a planned check, not a renderer-verified result. A supplied claim beside an unmet gate never changes that assessment or authorizes progression. Empty lists display **No gates supplied** / **No verification checks supplied**, not readiness or completed verification. Sources remain literal descriptions: never validate, link, fetch or execute them.

Rollback keeps mode, action, existing work, new work and boundary together, even for unknown/unavailable reversal. Modes display **Switch-back described**, **Repair required**, **Rollback unavailable** or **Rollback unknown**. Existing obligations are not canceled or migrated just because new admission switches back. No rollback button or executable command is created.

Every combination and release group for the selected rollout remains readable below the snapshot. Combination components are independent descriptions, not cross-references to phase components. Labels are **Authored compatible**, **Authored incompatible** or **Compatibility unknown**, with full reasons. Finite supplied cases are not an exhaustive matrix; unknown means neither safe nor unsafe. No component merging, setting comparison or semantic-version comparison occurs. Groups describe authored release-together constraints, not development order or phase dependencies. Empty groups display **No release-together groups supplied**, not permission to ship independently.

**Authored rollout plan — not deployed state, verified evidence or permission to proceed** stays visible. All text renders literally, without RichText, Markdown interpretation, authored HTML or automatic links. Full settings, names, versions, sources and rollback descriptions wrap without clipping or ellipsis; desktop detail columns stack in reading order at 390px and comparison widths. Existing Base16 tokens and visible native keyboard focus are retained. Briefs count **N rollouts**, with neutral **Rollout** tag and hue `0D`. No deployment, activation, environment probe, readiness computation, content request, storage, analytics, callback, option or shared viewer change is added.

Complete fallback includes every inactive rollout, phase, component, gate, supplied evidence/source, work class, check, rollback mode/action/work/boundary, combination, group/member, all absence notices and the authored-plan notice, alongside every mixed section. Escape every field and safely embed JSON, including closing-script/markup text; omit dead phase/execution controls. Preserve CSP, SRI and anonymous/no-referrer asset loading. Asset loads remain separate from renderer interactions. Missing, blocked or integrity-failed assets and disabled JavaScript must leave this content readable.

The complete `1.0.0` example is synthetic, not verified execution or a live deployment.

## Claims and private content

Examples use fixed synthetic content. Narration, role labels, coverage, source provenance, expected output, normalization policies, dependency pins, signal definitions, classifications and controls are authored claims, not observed execution, replayed cases, passing tests or security approval. The renderer never applies normalization, verifies a repository, captures a fixture or publishes the text. Keep authored markup and closing-script text inert in both rich content and host fallback.

Keep private content local unless its host and asset-loading policy are explicitly approved. Release discovery alone authorizes neither public CDN use nor publication of private walkthroughs. Complete readable fallback and restrictive host security are mandatory; see [host duties](README.md#host-security-and-fallback-duties).

## Consumer selection protocol

This is a protocol for future consuming-tool integration, not an implemented discovery client or a durable sidecar format.

1. Read `current.json` once when starting a new walkthrough.
2. Resolve and verify that pointer's manifest, containment and renderer-version agreement.
3. Read that release's default schema and matching authoring guide.
4. Generate the exact declared schemaVersion and validate offline with that release.
5. Mount using that release and retain complete readable fallback on failure.
6. Save the exact renderer/schema selection outside the payload in the consuming tool's future durable sidecar.
7. Write immutable versioned asset URLs and the manifest's integrity values for both assets into generated HTML.

Generated HTML never contains current/latest asset URLs. Existing pins never consult the pointer again. An upgrade requires an explicit new selection. Missing releases and unsupported schemas are errors, never automatic migration permission. The browser renderer performs no metadata discovery.
