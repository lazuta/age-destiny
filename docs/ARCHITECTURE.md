# Age Destiny — app architecture

Age Destiny is a Flutter (Dart) iOS app. It is a **personal travel tracker**
first — a map of visited countries with stats — plus a playful **Age flight**
mode that turns an age into a destination.

> This doc lives in the public site/data repo for reference. The app source
> lives in the separate `HBA` project.

## Tech stack

| Concern            | Choice |
|--------------------|--------|
| UI                 | Flutter, Material 3 |
| State management   | Riverpod (`flutter_riverpod`) |
| Navigation         | `go_router` |
| Maps               | `flutter_map` (raster tiles) + a custom SVG/equirectangular projection for the dotted style |
| Local storage      | `shared_preferences` (settings/cache), `flutter_secure_storage` + `encrypt` (AES) for travel data |
| Networking         | `http` (directories catalogue only) |
| Localization       | Hand-written `AppStrings` (EN/RU), no codegen |

## Layered structure (`lib/`)

```
core/        Cross-cutting: prefs keys & stores, map projection/geometry,
             i18n (AppStrings), security helpers, small utils.
data/        Models, datasources (asset/remote), repository implementations.
domain/      Entities, repository interfaces, pure services
             (age matching, IATA matching, route building, visit overlap).
presentation/
  providers/ Riverpod providers (state + wiring).
  screens/   Travel map (home), Age flight, Age-flight setup, onboarding, settings.
  widgets/   Map widgets, overlays, stats bar, coach marks, backdrop, etc.
  sheets/    Modal sheets (country visit, country search).
app/         App root, theme/palette, router, route observer.
```

The dependency direction is `presentation → domain → data`/`core`. Domain
services are pure and unit-tested; UI talks to them through Riverpod providers.

## Navigation & app flow

`go_router` with a redirect gate:

- First launch → **intro** (onboarding) → on finish, lands on `/` (travel map).
- `/` — **travel map** (home): visited countries, stats bar, year timeline.
- `/fun` — **Age flight** mode. First entry routes through `/fun-setup`
  (departure city, date of birth, initials → personal IATA code); afterwards it
  opens directly.
- `/settings`, `/cities`, `/directories` — supporting screens.

A one-time interactive coach-mark tour highlights the key actions on the travel
map after onboarding.

## State management

Riverpod providers expose data and derived state, e.g.:

- `countryVisitsProvider` — the user's visits (source of truth, persisted
  encrypted).
- `travelStatsProvider` — derived stats (countries, continents, % of world,
  trips, year span).
- `travelTimelineYearsProvider` — only the years that actually contain trips.
- `directoriesProvider` — the Age flight catalogue (see data flow below).
- Settings: locale, theme mode, accent color, map style.

## Maps

Two styles share one projection:

- **Tile style** — `flutter_map` raster tiles (CARTO basemaps) with custom
  overlays for country markers and the journey path.
- **Dotted style** — an in-app SVG world silhouette positioned with an
  equirectangular projection (`core/map/world_map_projection.dart`); markers and
  routes are projected to the same space.

The travel **journey path** (arcs connecting visited countries in order) draws
instantly on screen open and only animates when the route changes at runtime
(toggling it, or adding/editing a country).

## Data flow

- **Countries / cities / airports** — bundled JSON assets, loaded once.
- **Directories** — "remote with fallback": cached/bundled copy shown
  instantly, then revalidated from GitHub Pages in the background. See
  [DIRECTORIES.md](DIRECTORIES.md).

## Privacy & on-device data

- No account, no analytics, no personal data leaves the device.
- Travel visits are stored **AES-encrypted**; the key lives in the iOS Keychain.
- The only network egress is map tiles and the directories catalogue (both
  static, public HTTPS) plus optional user-initiated tourism links.
- `ITSAppUsesNonExemptEncryption=false` is correct (encryption only protects the
  user's own local data → export-exempt). An App Privacy manifest ships in the
  iOS bundle.

## Testing

Pure domain services and datasources are covered by unit tests
(`test/`), including age matching, visit overlap, IATA name matching, journey
path building, and the directories remote/cache/fallback logic.
