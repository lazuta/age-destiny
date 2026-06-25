# Directories — data format & publishing

"Directories" are the lists the **Age flight** mode matches an age against
(phone codes, capitals, regional lists, …). They are published from this repo
at [`data/directories.json`](../data/directories.json) and pulled by the app at
runtime.

## How delivery works

- The app ships a **bundled copy** of `directories.json` as an offline /
  first-launch fallback.
- On launch it shows the cached (or bundled) catalogue immediately, then
  fetches `https://lazuta.github.io/age-destiny/data/directories.json` in the
  background (8s timeout). The payload is validated; only a well-formed JSON
  array of valid directories is accepted.
- If the fetched content differs from the cache, it replaces the cache and the
  UI rebuilds this session. Otherwise nothing changes.
- The "airports" directory is **not** in this file — it is built inside the app
  from airport data and always appended.

So: **edit `data/directories.json`, commit, push.** Within a launch or two users
have the new directory. Invalid JSON is ignored (app keeps the last good copy),
so a bad push can't break the app — but it also won't take effect until fixed.

## File shape

A JSON **array** of directory objects:

```json
[
  {
    "id": "intl_phone_codes",
    "title": "International Phone Codes",
    "titleRu": "Телефонные коды стран",
    "description": "Travel the world through country calling codes.",
    "descriptionRu": "Путешествие по миру по телефонным кодам стран.",
    "iconEmoji": "📞",
    "inputType": "age",
    "codePrefix": "+",
    "items": [
      {
        "code": "1",
        "title": "USA / Canada",
        "titleRu": "США / Канада",
        "countryCode": "US",
        "coordinates": { "latitude": 38.8951, "longitude": -77.0364 }
      }
    ]
  }
]
```

### Directory fields

| Field          | Required | Notes |
|----------------|----------|-------|
| `id`           | yes | Stable unique slug, e.g. `world_capitals`. Don't reuse ids. |
| `title`        | yes | English title. |
| `titleRu`      | no  | Russian title (falls back to `title`). |
| `description`  | yes | English one-liner. |
| `descriptionRu`| no  | Russian description. |
| `iconEmoji`    | no  | Emoji shown next to the directory. |
| `inputType`    | yes | `"age"` for age-matched lists. `"initials"` is the special IATA-from-name mode (don't author new ones). |
| `codePrefix`   | no  | Prefix shown before each item code, e.g. `"+"`. |
| `countryScope` | no  | Array of ISO country codes. If set, the directory is only offered when the departure city is in one of those countries. Omit/empty = universal. |
| `items`        | yes | Array of item objects (below). |

### Item fields

| Field         | Required | Notes |
|---------------|----------|-------|
| `code`        | yes | The value matched against age (e.g. phone code, ordinal). |
| `title`       | yes | English destination name. |
| `titleRu`     | no  | Russian name. |
| `countryCode` | yes | ISO 3166-1 alpha-2 (used for the flag and continent stats). |
| `coordinates` | yes | `{ "latitude": <num>, "longitude": <num> }` — where the route lands. |

## Adding a new directory — checklist

1. Append a new object to the array in `data/directories.json`.
2. Give it a unique `id`, an `inputType` of `"age"`, and `items` with valid
   `coordinates` and `countryCode`.
3. Validate the JSON locally:
   ```bash
   python3 -c "import json; json.load(open('data/directories.json'))"
   ```
4. Commit and push to `main`. GitHub Pages serves the file within ~1 minute.
5. (Recommended) Keep the app's bundled copy in sync for the next release:
   copy this file to `HBA/assets/data/directories.json` so fresh installs and
   offline users get the new directory too.

## Notes & limits

- The app caps the download at 4 MB and ignores anything that isn't a valid
  directory array, so malformed pushes are safely ignored.
- Coordinates drive the animated route and the country flag/continent — wrong
  values just place the pin in the wrong spot; they can't crash the app.
- Cyrillic and emoji must be UTF-8 (the file already is).
