# Age Destiny — site & data repo

This repository serves two purposes for the **Age Destiny** iOS app:

1. **Marketing / support site** published via GitHub Pages at
   <https://lazuta.github.io/age-destiny/> (`index.html`, `support.html`,
   `privacy.html`, `site.css`, `logo.png`).
2. **Live directories data** consumed by the app at runtime from
   [`data/directories.json`](data/directories.json). Pushing changes here
   updates the directories users see — no App Store release required.

## Repository layout

```
index.html        Landing page
support.html      Support page (App Store "Support URL")
privacy.html      Privacy policy (App Store "Privacy Policy URL")
site.css          Shared styles
logo.png          App icon / brand mark
data/
  directories.json   Published "Age flight" directories (pulled by the app)
docs/
  ARCHITECTURE.md    App architecture overview
  DIRECTORIES.md     Directories data format + how to publish a new one
```

## Documentation

- **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** — how the Flutter app is
  structured (layers, state, map rendering, on-device data, privacy).
- **[docs/DIRECTORIES.md](docs/DIRECTORIES.md)** — the directories JSON schema
  and the step-by-step workflow for adding a new directory by pushing here.

## How the app pulls data

On launch the app loads the cached/bundled directories instantly, then fetches
`https://lazuta.github.io/age-destiny/data/directories.json` in the background.
If it changed, the catalogue is updated for this session and cached for the
next launch. A copy is also bundled in the app as an offline / first-launch
fallback, so the app always works even with no network. See
[docs/DIRECTORIES.md](docs/DIRECTORIES.md) for details.
