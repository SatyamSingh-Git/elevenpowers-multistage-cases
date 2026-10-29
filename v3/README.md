# v3: projects where the code at risk is hard to find

The twelve v2 projects in the folder above were too easy. In their pilot, the
AI assistant found every piece of code that depended on a change with one
search, and nothing broke in either setup. These twelve v3 projects are built
so that can go wrong fairly.

- **Bigger:** 2,500–2,750 lines each, over 40–50 modules in several
  subpackages.
- **Longer:** eight change requests in one conversation.
- **A quiet dependent in every request that touches shared code.** That is
  code in another module that must also change, which a search for the
  changed names does not find and the project's own tests do not catch.

  They come from ordinary design choices:
  - a registry looked up by a string;
  - a file one module writes and another reads;
  - a second copy of a rule;
  - code that relies on an ordering or rounding.

  A careful engineer can always find them by tracing what the code does.

Each case's `case.json` lists its quiet dependents, with the kind and one
sentence on how to find each.

| project | source lines | what it is |
|---|---|---|
| [`clinic_booking`](clinic_booking) | 2,573 | appointment booking for a small clinic |
| [`fleet_service`](fleet_service) | 2,618 | vehicle fleet maintenance |
| [`gradebook`](gradebook) | 2,609 | a school gradebook |
| [`greenhouse_monitor`](greenhouse_monitor) | 2,513 | greenhouse sensors and alerts |
| [`hotel_desk`](hotel_desk) | 2,513 | a small hotel's front desk |
| [`household_budget`](household_budget) | 2,568 | household budgeting |
| [`library_loans`](library_loans) | 2,635 | a public library's lending |
| [`parcelpost`](parcelpost) | 2,641 | parcel shipping |
| [`payroll_lite`](payroll_lite) | 2,509 | small-business payroll (fictional rules) |
| [`recipe_box`](recipe_box) | 2,743 | recipes and meal planning |
| [`survey_tally`](survey_tally) | 2,524 | survey responses and tables |
| [`ticket_office`](ticket_office) | 2,549 | theatre box office |

## How they were checked

- **No-model rehearsal ([`rehearsal.json`](rehearsal.json)).** Every case passes
  from a fresh Windows checkout. For every stage, the reference solution passes
  all hidden checks, documents every fact and keeps the project rules. Each
  quiet dependent, if left behind, does three things:
  - it fails exactly the earlier checks declared for it;
  - it keeps the project's own suite green;
  - it does not appear in a search for anything the stage changed.
- **Independent review ([`review.json`](review.json)).** Each case had one fresh
  reviewer, who hunted for two problems: correct work graded as wrong, and
  requests that hint at the code at risk. Ten blocking findings across the set
  went back to the authors and were fixed before the freeze.
- **What is disclosed, not fixed.** 55 of the 68 quiet dependents can be
  reached by searching for words the request itself uses, through ordinary
  comments and docs. Removing that prose would mean hiding things, so it stays.
- **Frozen ([`manifest.json`](manifest.json)).** Every file's hash is recorded at
  ElevenPowers commit `8c3d7b1`.

## The pilot

`gradebook` and `parcelpost` were written first and run as a pilot: four
conversations, Sonnet 5.5 at medium effort, with superpowers in both setups.
With superpowers alone, the assistant broke earlier working behaviour on both.
On gradebook, a precision change left the grades export at one decimal place.
On parcelpost, reordering the manifest split each customer's invoice in two. A
blind classifier, who was not told the setup, judged both genuine
([`pilot-classified.json`](pilot-classified.json)). No documentation went stale
in either setup. Pilot runs are excluded from the comparison by design.

The full comparison has not run yet. Its size comes from
[`power.json`](power.json), and it waits for the project owner's go. Results
will be added in ElevenPowers' [`results/multistage-v3`](https://github.com/SatyamSingh-Git/ElevenPowers/tree/main/results/multistage-v3).
