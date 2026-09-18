# open-test-data

Synthetic people for use as test fixtures: names, short physical descriptions, and
AI-generated profile portraits. Every record is fabricated. Use it wherever you need
realistic-looking sample users without touching real personal data.

> **No real people.** Names are randomly generated. Faces are produced by an image
> model from a text description and do not depict any actual person. Any resemblance
> to a real individual is coincidental.

## Layout

```
manifest.json                       index of every person in the set
person/
  <name_hash>-<sequence>/
    person.json                     the record
    person_log.json                 image generation history
    profile/
      base.webp                     neutral reference portrait
      professional.webp             studio portrait derived from base
```

Each person lives in one directory named by its `key`. The key is a 16-hex-character
hash of the name followed by a sequence number, for example `00f3f6c55434db3e-1`.
The sequence lets two distinct people share a name.

## manifest.json

The entry point. Lists every person with the fields most consumers need.

```json
{
  "format_version": 1,
  "entries": [
    {
      "key": "00f3f6c55434db3e-1",
      "display_name": "André Vaillant",
      "images": {
        "base": "profile/base.webp",
        "professional": "profile/professional.webp"
      }
    }
  ]
}
```

Image paths are relative to the person's directory, so the full path is
`person/<key>/<path>`.

## person.json

| Field | Type | Meaning |
|---|---|---|
| `format_version` | int | Schema version, currently `1` |
| `key` | string | `<name_hash>-<sequence>`, matches the directory name |
| `name_hash` | string | 16 hex characters derived from the name |
| `sequence` | int | Disambiguator for people sharing a name hash |
| `first_name`, `last_name` | string | ASCII or romanized form |
| `display_name` | string | Preferred rendering, may use native script or accents |
| `names` | object | Per-locale spellings keyed by locale, e.g. `ja_JP`, `fr_FR`. Empty when the ASCII form is the only one |
| `gender` | string | `M` or `F` |
| `locale` | string | Locale the name was drawn from, e.g. `en_US`, `ja_JP` |
| `look` | string | Coarse appearance category used to steer image generation |
| `appearance` | string | One-sentence physical description used as the image prompt |
| `appearance_seed` | int | Seed that produced the appearance text |
| `images.<role>.path` | string | Relative path to the image |
| `images.<role>.image_seed` | int | Seed used to generate that image |
| `images.<role>.reference` | string | Role whose image was used as a likeness reference |

Example:

```json
{
  "format_version": 1,
  "key": "00f3f6c55434db3e-1",
  "name_hash": "00f3f6c55434db3e",
  "sequence": 1,
  "first_name": "Andre",
  "last_name": "Vaillant",
  "display_name": "André Vaillant",
  "names": { "fr_FR": { "first_name": "André", "last_name": "Vaillant" } },
  "gender": "M",
  "locale": "fr_FR",
  "look": "white",
  "appearance": "white man in his 50s, light skin, buzzed silver hair, grey eyes, no glasses, friendly grin, a trimmed goatee",
  "appearance_seed": 2528958472,
  "images": {
    "base": { "path": "profile/base.webp", "image_seed": 3019527744 },
    "professional": { "path": "profile/professional.webp", "image_seed": 312866366, "reference": "base" }
  }
}
```

## person_log.json

An append-only history of image generation for the person. Each entry records the
role, the action (`generate` or `regenerate`), the seed, the full prompt, the model,
a timestamp, and the reference role if one was used. The current image for a role is
the result of the latest entry for that role.

```json
{
  "format_version": 1,
  "key": "00f3f6c55434db3e-1",
  "entries": [
    {
      "ts": "2026-09-17T00:03:12Z",
      "role": "base",
      "action": "generate",
      "image_seed": 3019527744,
      "prompt": "Neutral reference portrait, head and shoulders, ...",
      "model": "gemini-2.5-flash-image",
      "model_version": "gemini-2.5-flash-image"
    }
  ]
}
```

## Images

Two portraits per person, both 384×384 WebP with no embedded metadata.

- **base**: head and shoulders, facing the camera, plain light grey background,
  even lighting, natural expression. This is the canonical likeness.
- **professional**: a studio portrait of the same person with a colored backdrop and
  business or smart-casual clothing, generated using `base` as a reference image.

## What's in the set

| | |
|---|---|
| People | 59 |
| Images | 118 |
| Locales | en_US, fr_FR, ja_JP, es_MX, en_IN, tr_TR, zh_CN |
| Gender | 37 M, 22 F |
| Image model | gemini-2.5-flash-image |

## Usage

```python
import json
from pathlib import Path

root = Path("open-test-data")
manifest = json.loads((root / "manifest.json").read_text())

for entry in manifest["entries"]:
    person_dir = root / "person" / entry["key"]
    person = json.loads((person_dir / "person.json").read_text())
    portrait = person_dir / entry["images"]["professional"]
    print(person["display_name"], person["locale"], portrait)
```

## Notes

- All content is generated. Names come from a locale-aware random name generator;
  descriptions and portraits come from seeded generation and can be reproduced from
  the recorded seeds and prompts.
- `look` and `appearance` exist to make the image set visually diverse and to keep
  each person's portraits consistent with each other. They are generation inputs,
  not demographic data about anyone.

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for the
full text and [NOTICE](NOTICE) for the attribution notice.

```
Copyright 2026 AppCove, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

This covers the whole repository, the JSON records and the images alike. If you
redistribute any part of it, including a single portrait, include a copy of the
license and retain the notice.
