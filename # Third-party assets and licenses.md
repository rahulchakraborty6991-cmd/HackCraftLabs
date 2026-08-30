# Third-party assets and licenses

Event start code: **LSH26-8490-C900**

This project is a single React component (`personal-ledger.jsx`). No CSS
framework, starter kit, project template, or component library was used —
all styling is hand-authored inline style objects. Third-party assets are
listed below.

## Libraries

| Asset | Source | License | Use |
|---|---|---|---|
| React | react.dev | MIT | UI runtime |
| lucide-react | lucide.dev | ISC | Interface icons (upload, alert, trend arrows, piggy bank, etc.) |

Icons are used as imported components from `lucide-react`; none were
redrawn or vendored into this repository.

## Fonts

| Asset | Source | License | Use |
|---|---|---|---|
| Source Serif 4 | Google Fonts (`fonts.googleapis.com`) | SIL Open Font License 1.1 | Display/heading typeface |
| Inter | Google Fonts (`fonts.googleapis.com`) | SIL Open Font License 1.1 | Body/UI typeface |
| IBM Plex Mono | Google Fonts (`fonts.googleapis.com`) | SIL Open Font License 1.1 | Taka amounts, dates, tabular figures |

Fonts are loaded at runtime via a standard `@import` from Google Fonts —
no font files are vendored in this repository.

## External services

Receipt reading calls the Anthropic Messages API (`api.anthropic.com`,
model `claude-sonnet-4-6`) with the uploaded image. The model returns the
amount, date and shop name plus a per-field confidence flag; low-confidence
fields are left blank and marked for the user to fill in. No image or
expense data is sent anywhere else, and nothing is persisted off-device.

## Code

All application code was written for this submission. No boilerplate,
starter template, or scaffold from a third party was used as a base.

## Sample data

The 25 cases embedded in `personal-ledger.jsx` are the official P12
published fixture set (`P12_personal_ledger_public.json`, schema 2.1),
normalised from the published field names (`amount_bdt` strings and
similar) into this project's numeric fields. Values are unchanged. The app
also accepts the raw published schema at runtime via the "Load case file"
control, so an unmodified fixture file can be used directly.

The data is synthetic. It contains no real personal or financial records.

## DPS calculation

The deposit-pension-scheme rate and compounding rule are taken from each
case's `dps_annual_rate_percent` and `dps_rule` fields rather than chosen
by this project: each month the contribution is added to the balance, then
interest of balance x rate / 12 / 100 is rounded half up to the paisa and
added to the balance.