# Personal Ledger Manager (P12)

**Team ID:** LSH26-T029
**Problem ID:** P12 — Personal Ledger Manager
**Live URL:** https://github.com/rahulchakraborty6991-cmd/lsh26-t029-p12

A single React component (`personal-ledger.jsx`). A salaried person in
Dhaka sets their monthly salary, records spending with as little typing as
possible, sees where the money went, gets a forecast of the rest of the
month, and gets a real date on every savings goal.

## Proof each requirement is met

| # | Requirement | Where / how |
|---|---|---|
| 1 | Set a monthly salary; add expenses including by photo of a bill; show what was read; allow correction before saving | Salary is an editable field in the header and feeds every downstream number. "Add expense" tab has two paths: a photo upload (`handlePhoto()`) that reads amount, date and shop from the image and returns a per-field confidence flag, and a manual form. The review panel shows every extracted field with a "looks right" / "check this" badge; `saveOcrExpense()` is blocked until amount, date and shop are all filled |
| 2 | Monthly dashboard: total against salary, breakdown by category, largest expenses, change on last month | Four stat tiles (spent so far, still to come, change on last month, left/short at month end), a category bar list with each category's top shop, the five largest expenses of the month, and a signed change figure with percentage against the previous month; all computed in the `dash` memo from the live expense list |
| 3 | Forecast and written insights from the actual numbers: rest-of-month spending, money left or short at month end, at least three insights naming specific categories and amounts | The `forecast` memo produces rest-of-month spending as its own tile and the month-end position as another. The `insights` memo emits up to six lines, every one built from computed values — biggest category increase with both months' figures, largest cost centre with its share and top shop, the category set to take the most over the remaining days, the recurring-charge total, the month-end position, and the pocket funding position |
| 4 | Savings pockets with name, target, item details and monthly contribution; expected completion date from the forecast; DPS return over that time | The "Savings pockets" tab lists each pocket with an editable target and contribution, and a form to create new ones. `simulate()` runs the forecast surplus month by month and returns the month each pocket completes, which is rendered as a date via `addMonths(today, months)`, alongside total paid in, DPS interest earned and final payout |

**Constraints followed:**

- **Unsure fields are never filled in.** The receipt reader is instructed to
  return `null` with low confidence rather than guess a number it cannot
  read. A null arrives as an empty field flagged "check this", and the save
  button stays disabled until the user supplies it.
- **Insights are computed, never fixed.** Every insight string interpolates
  values from the current state. Editing the salary, adding an expense or
  moving the what-if slider rewrites them.
- **Completion dates come from the forecast.** `simulate()` funds pockets
  from the projected monthly surplus, scaled by the share of the requested
  contributions that surplus can actually cover — not `target ÷ contribution`.
  Where the forecast leaves nothing to save, the card says so and gives the
  monthly reduction needed instead of printing a date.
- **DPS rate and method are stated on screen.** The rate comes from each
  case's `dps_annual_rate_percent`. The Pockets tab states it and the rule:
  contribution is added to the balance first, then interest of
  balance × rate ÷ 12 ÷ 100 is rounded half up to the paisa and added to the
  balance, so later months earn on it.

**Bonus features implemented (all three):**

- Contribution sliders on every pocket; moving one re-runs the whole
  simulation and every completion date moves immediately.
- Automatic recurring detection — same shop, amount within 15%, present in
  both months. Each prior charge can anchor only one match, so two visits
  this month don't both bind to a single charge last month.
- A what-if control that cuts one category by a percentage and shows the
  effect on every pocket date.

## Major decisions

- **Paid recurring charges are not projected forward.** The naive forecast
  extrapolates every category at its daily rate, which charges rent again
  for each remaining day when rent was paid on the 3rd. On PUB-09 that
  projected ৳119,870 of spending against a ৳90,000 salary for a person who
  was not overspending. Charges already matched as recurring are carried at
  face value and only variable spending is extrapolated. Across the 25
  published cases this takes the number where no pocket can be given a date
  from 8 down to 2 — and those two, PUB-06 and PUB-23, are genuinely
  spending past their salary.
- **Money is held in integer paisa.** Balances, deposits and interest are
  computed in paisa with explicit half-up rounding, so the DPS rule is
  applied exactly as written rather than accumulating floating-point drift
  over hundreds of simulated months.
- **The DPS rate is read, not chosen.** The brief allows stating a rate, but
  every published case supplies one, so the app uses the case's value.
- **Sample data sourced from the official fixture.** All 25 cases are the
  published P12 set (schema 2.1), normalised from the published field names
  into numeric fields with values unchanged. The app also accepts the raw
  published schema at runtime through "Load case file", so an unmodified
  fixture can be dropped in without touching code.
- **No backend.** All state is in-memory in the component. This is a
  single-screen personal tool and the brief describes no login; see
  limitations.

## Known limitations

- **The surplus is assumed to repeat.** The forecast produces one month's
  projected surplus and `simulate()` applies it to every future month. The
  brief does not define a recurring baseline, so a month with an unusual
  one-off will pull every completion date with it. A longer-running version
  would build the baseline from several months of history rather than one.
- **No persistence.** Expenses added, pockets created and salary changes
  live for the session only — a refresh returns to the loaded case.
- **Single user, no authentication.** Acceptable for a one-person ledger,
  not multi-device safe.
- **Receipt reading needs a network call** and is only as good as the photo;
  a blurred total produces an empty flagged field rather than a wrong
  number, which is the intended failure mode but still costs the user typing.
- **Two months of history.** Recurring detection and the change-on-last-month
  figure both need a previous month, so both are unavailable for a first-ever
  month of use.

## Approach and contributions

**Approach:** the team read the problem statement and the published fixture
schema together first, so the DPS rule and the case shape were settled
before any UI was built. Working from the organisers' own sample data
rather than invented numbers surfaced the forecast problem early — a third
of the published cases could not produce a completion date under a naive
daily-rate projection — which shaped the central design decision. The build
then moved to the dashboard, the forecast and insights, and the pocket
simulation, finishing with a requirement-by-requirement and
constraint-by-constraint check against the running app across all 25 cases.

**Contributions:**

- **Rahul — Team Leader:** set the overall approach and architecture (data
  model, how the forecast feeds the pocket simulation, and the UI
  direction), and drove the build.
- **Hemayet:** problem-solving and analytical logic — the recurring-charge
  rule, the forecast correction and its validation across the published
  cases, the paisa-exact DPS simulation, and the requirement-by-requirement
  proof in this README.
- **Manik:** the dashboard's data-visualization implementation — the stat
  tiles, the category bar breakdown, the pocket cards and the what-if
  control.
