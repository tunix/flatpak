# tunix/flatpak

Self-hosted [Flatpak](https://flatpak.org) (OSTree) repository for the GNOME
applications I publish. Published to GitHub Pages at
**https://tunix.github.io/flatpak/** and updated automatically on every app
release.

## Installing apps

```bash
flatpak remote-add --if-not-exists tunix https://tunix.github.io/flatpak/tunix.flatpakrepo
flatpak install tunix <app-id>
```

Example:

```bash
flatpak install tunix io.github.tunix.valhalla
```

Updates are delivered through the normal channel — `flatpak update` picks up
new releases from this repo automatically.

## Hosted apps

| App | Description |
| --- | --- |
| [io.github.tunix.valhalla](https://github.com/tunix/valhalla) | Theme-aware wallpaper switcher (Rust + GTK4/libadwaita) |

## How it works

- Each app release in its own repository sends a `repository_dispatch` event
  (`app-released`) to this repo.
- The [`update-repo`](.github/workflows/update-repo.yml) workflow rebuilds the
  changed app with `flatpak-builder`, commits it into the OSTree repo under
  `repo/`, and republishes the `summary` (signed with a GPG key held in
  repository secrets).
- Previous app builds are preserved in the repo, so existing installs keep
  working and deltas (`--generate-static-deltas`) keep upgrades fast.
- `tunix.flatpakrepo` (remote metadata + GPG key) and the per-app
  `.flatpakref` files live at the repo toplevel and are what users add.

## Repository layout

```
apps/<app-id>.json            per-app flatpak-builder manifest
apps/<app-id>.flatpakref      per-app install link
tunix.flatpakrepo             remote metadata (add this one)
repo/                         OSTree repository (published to gh-pages)
```

## Adding an app

1. Drop the app's manifest into `apps/<app-id>.json`.
2. Add the app id to the `matrix.app` list in `update-repo.yml`.
3. In the app's own repo, add a job that dispatches `app-released` with
   `{ "app_id": "...", "tag": "vX.Y.Z" }` on release.

## Notes

- The GPG signing key is generated once and stored in the `GPG_KEY` secret;
  losing it means every user has to re-add the remote, so it is kept long-term.
- GitHub Pages (1 GB limit) is the publishing target; `build-update-repo
  --prune` keeps the OSTree repo from growing without bound.