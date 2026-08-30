# listkomat-catalog

Public ticket catalog for the **Lístkomat** apps
([iOS](https://github.com/BugsBunny338/listkomat-ios),
[Android](https://github.com/BugsBunny338/listkomat-android)). Each app fetches
`tickets.json` on launch (cached locally, with a bundled fallback when offline), so
prices and SMS codes can be corrected here **without a store release**.

This file is now a **two-client contract** — both apps decode it, and older builds
of either keep reading it forever.

Verify every code/price against the operator's current ceník before editing.

## Editing rules

Every build ever shipped reads this file, not just the newest one — so changes must
be **additive**. Adding a key is safe (the app's decoder ignores keys it doesn't
know); renaming or removing one breaks older builds in the wild. Bump `version` on
every change: the app only adopts a fetched catalog whose `version` is `>=` the one
it already has.

## Localization (`i18n`)

Czech is the source language and lives in the plain fields (`name`, `note`). A city
or ticket may carry an optional `i18n` block with per-language overrides, keyed by
language code:

```jsonc
{
  "key": "praha", "name": "Praha",
  "i18n": { "en": { "name": "Prague" } },
  "tickets": [
    { "code": "DPO70", "duration": "70 min", "durationMinutes": 70, "priceKc": 38,
      "note": "o víkendu a svátcích 90 min",
      "i18n": { "en": { "note": "90 min on weekends and public holidays" } } }
  ]
}
```

The app looks up its own UI language and falls back to the Czech field when there is
no override, so a missing translation degrades to Czech rather than to nothing.

Only `name` (cities) and `note` (tickets) are translatable. Durations are **not**:
the app formats them from `durationMinutes` in the user's language, with correct
plurals. `duration` is a legacy display string kept only for builds shipped before
that change — keep it in sync with `durationMinutes`, but it is not what newer
builds show.

Ticket codes, SMS numbers, prices and city `key`s are never localized.
