# AGENTS.md — tunix/flatpak

Self-hosted Flatpak (OSTree) repository for tunix's GNOME apps, published to
GitHub Pages and rebuilt automatically when an app releases.

## What lives here

- `apps/<app-id>.json` — per-app flatpak-builder manifests (single-module,
  git-tag sources; no subdirectory manifests).
- `apps/<app-id>.flatpakref` — per-app install links pointing at this repo.
- `tunix.flatpakrepo` — remote metadata + GPG key reference (the file users add).
- `.github/workflows/update-repo.yml` — the only real machinery: listens for
  `repository_dispatch` (`app-released`), builds the named app, commits it into
  `repo/` (OSTree), regenerates the signed `summary`, deploys to `gh-pages`.
- `repo/` is generated output; never hand-edit it.

## Rules

- The GPG key lives ONLY in the `GPG_KEY` repo secret. Losing it invalidates
  every existing remote (users must re-add). Never regenerate it casually.
- Preserve prior builds in the OSTree repo — pull the existing `repo/` from
  gh-pages before building, pass `--force-commit`, and let
  `build-update-repo --prune` handle growth. Deleting old branches breaks
  installed clients' updates.
- Manifests must use toplevel files and bare `sdk-extensions` ids (e.g.
  `org.freedesktop.Sdk.Extension.rust-stable` — versioned refs fail with
  "extension not installed"). Same traps as the Flathub submission manifests.
- `generated-sources.json` is ALWAYS regenerated from the app's `Cargo.lock`
  on every build (checked-in copies drift behind renovate).
- One app per `matrix.app` entry in `update-repo.yml`; adding an app is a
  manifest + one matrix line, nothing else.
- Trigger contract: app repos send `repository_dispatch` with client_payload
  `{ "app_id": "<id>", "tag": "vX.Y.Z" }`. The workflow reads the tag from the
  payload, not from defaults.

## Conventions

- Commits: conventional style (`feat:`, `fix:`, `ci:`).
- Do not push to `gh-pages` manually except when bootstrapping the very first
  repo state; afterwards only the workflow writes it.
- Runtime choice: pick the OLDEST supported GNOME runtime (travels with the
  app; distro GNOME version is irrelevant).