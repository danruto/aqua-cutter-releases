# Aqua Cutter releases

Distribution repository for approved Aqua Cutter renderer artifacts. It contains built releases only, with no source history. No `current.json` pointer has been promoted yet.

## Loading a release

Files are served through jsDelivr from this repository. Pin the full commit that added the release, never a branch:

```text
https://cdn.jsdelivr.net/gh/danruto/aqua-cutter-releases@<commit>/releases/<version>/renderer.js
https://cdn.jsdelivr.net/gh/danruto/aqua-cutter-releases@<commit>/releases/<version>/renderer.css
```

Load both with the `integrity` hashes from that release's `manifest.json`, `crossorigin="anonymous"` and `referrerpolicy="no-referrer"`.

## Contents

```text
releases/<renderer-version>/
  manifest.json
  renderer.js
  renderer.css
  README.md
  authoring.md
  schema/
  examples/
  licenses/
current.json                 # created only after approved publication and promotion
```

Renderer versions identify immutable assets. Walkthrough schema versions identify exact payload contracts. The initial prepared versions are both `1.0.0`, but they evolve independently. The browser API remains `window.AquaCutterRender`.

## Safety rules

1. Publish only an explicitly approved stable version from its clean, matching tagged source. A successful build is not release approval.
2. Install each complete `releases/<version>/` directory create-only. Never replace, repair, merge into or force-push a published version, even with identical bytes.
3. Include the manifest, schemas, complete public synthetic examples, reader guides and third-party licence notices with the assets.
4. Read back every published file and compare its bytes with the verified local artifact. Prove real browser loading, integrity, anonymous CORS loading, no-referrer and the intended content security policy from the selected host.
5. Promote `current.json` only after approval and published checks. Reject stale concurrent selection, preserve the prior pointer on failure, and reconcile uncertain outcomes by reading the pointer.
6. Roll back by selecting an older approved, still-verifiable artifact. Never rewrite old releases or consumer pins.

A normal non-force Git push rejects a stale branch update, but does not by itself prove host-level atomic publication or selection. Hosting, publication and promotion need a separately verified destination-specific procedure. There is no automatic publisher here.

## Source and privacy

All code and internal documentation remain in the private Aqua Cutter studio for now. No public renderer source split has occurred. A manifest's `sourceCommit` identifies its exact private source commit; this does not make the source publicly inspectable or establish public reproducibility.

Never copy the studio Git history, source checkout, private walkthroughs, internal demos/designs, credentials or machine-specific paths into this repository. Only reviewed generic assets and public synthetic content belong here. Do not grant a public asset host access to private walkthrough data.

## Preparing a release

1. In the studio's locked Nix/Bun environment, run `bun install --frozen-lockfile` and `bun run check`.
2. After separate release/tag approval, assemble from clean tagged source using `bun run release:assemble --tag vX.Y.Z --output /absolute/path/to/aqua-cutter-releases`.
3. Verify with `bun run release:verify --directory /absolute/path/to/aqua-cutter-releases/releases/X.Y.Z`.
4. Follow the approved destination-specific publishing procedure. Do not create a pointer or publish merely because local assembly passed.

Add each release as a new commit on `main`. Never force-push, rewrite history or delete `main`; a repository ruleset enforces this.
