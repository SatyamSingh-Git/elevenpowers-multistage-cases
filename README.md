# ElevenPowers multi-stage cases

Twelve small Python projects built to test one question about AI coding
assistants: **when a project evolves through several requests in one
conversation, do earlier work and the docs survive the later changes?**

These are the frozen test projects for an experiment in
[ElevenPowers](https://github.com/SatyamSingh-Git/ElevenPowers). That experiment
compares Claude Code running the [superpowers](https://github.com/obra/superpowers)
plugin alone against superpowers plus ElevenPowers. It tests two things that
developers using ElevenPowers on a real project reported: docs were kept
current, and a fix stopped breaking code that depended on what it changed.

> **Status: these twelve (v2) were piloted on 2026-10-28 and found too easy.**
> In four pilot conversations nothing broke in either setup: the assistant
> found every dependent with one search. The experiment moved to
> **[v3](v3/)**: twelve bigger projects built so that the code at risk is hard
> to find. These v2 projects stay here unchanged, as frozen.

## The projects

| project | source lines | what it is |
|---|---|---|
| [`access_policy`](access_policy) | 789 | role and attribute access policies |
| [`cronish`](cronish) | 615 | schedule expressions |
| [`eventlog`](eventlog) | 759 | append-only event log on disk |
| [`invoice_calc`](invoice_calc) | 639 | invoices, tax and rounding |
| [`layered_config`](layered_config) | 688 | layered configuration loader |
| [`ledgerbook`](ledgerbook) | 697 | double-entry ledger with a saved journal |
| [`marklite`](marklite) | 842 | markdown-subset renderer |
| [`stockroom`](stockroom) | 668 | inventory with a JSON store |
| [`tabulate_report`](tabulate_report) | 652 | CSV reports |
| [`throttle`](throttle) | 644 | rate limiting |
| [`tinyroute`](tinyroute) | 736 | URL router |
| [`unitconv`](unitconv) | 652 | unit conversion |

Each project is a working standard-library Python package in `start/`. It
comes with:
- its own pytest suite;
- a `docs/reference.md` table of documented facts;
- a `CHANGELOG.md`;
- one protected file;
- a `CLAUDE.md` with four rules:
  1. keep the docs current;
  2. add a changelog entry for every change;
  3. every bug fix carries a test;
  4. never edit the protected file.

## What an agent is asked to do

`case.json` holds six requests, written the way a user would ask. The agent
receives them one at a time, in one conversation.
- **At least four are cross-cutting.** They change a shared helper, a data
  format, a default or a signature that other code already depends on, and
  they don't name that code. An engineer who searches the codebase finds and
  updates it; one who only changes the obvious place breaks something that
  used to work.
- **At least one is self-contained.** Nothing depends on what it changes,
  which makes it a control.
- **At least one is a bug report.**
- **Every request changes a documented fact.** None of them mentions docs,
  tests or the changelog: the rules in `CLAUDE.md` have to do that work.

## How an attempt is graded

Each project has these parts:

| part | what it holds |
|---|---|
| `oracle/groups/` | hidden pytest groups that check behaviour through the public interface, cumulatively: two base groups plus two per stage |
| `oracle/facts.json` | the documented facts expected after each stage |
| `stages/<n>/reference.diff` | a correct implementation of stage n |
| `stages/<n>/dependent-missed.diff` | for cross-cutting stages: the mistake of changing the obvious place and leaving a dependent behind |

After every stage, three things are measured:
- **Earlier behaviour lost:** groups that passed after the previous stage and
  fail now.
- **Stale docs:** documented facts that are now wrong or missing.
- **Rules kept:** a changelog entry was added, the protected file is
  untouched, and a bug fix carries a test that fails when the fix is reverted.

The hidden groups and the reference answers are public here on purpose, so
anyone can check the grading. During the experiment the agents work in a copy
of `start/` only, with network tools blocked.

## How the projects were made and checked

**Authoring.** Twelve separate agents each wrote one project from
[`AUTHORING.md`](AUTHORING.md). None was told anything about ElevenPowers, so
none could shape a project to suit it.

**No-model rehearsal.** For every stage of every project
([`rehearsal.json`](rehearsal.json)):
- the reference passes all its groups, documents every fact and keeps the rules;
- leaving a dependent behind fails exactly the earlier groups it should;
- keeping the previous stage's docs leaves exactly that stage's facts stale;
- doing nothing fails the stage's new groups.

**Two independent review rounds** ([`review.json`](review.json)). Round one
found six places where a correct but different implementation would have been
graded as failing. It also found, in ten of the twelve projects, documented
facts worded in ways no implementer could guess. Every project was revised
under the fairness rules now in `AUTHORING.md`:
- a graded fact's key either already exists in the docs or is named in the
  request;
- its value is stated in the request;
- a hidden group requires only what the request says.

Round two cleared all twelve.

**Frozen.** [`manifest.json`](manifest.json) records the SHA-256 of every file
here, as committed in ElevenPowers, and the analysis fixed before any run. It
also names the two pilot projects, chosen by seeded hash before any outcome
existed: `access_policy` and `layered_config`.

## Rehearse a project yourself

The tools that build and grade these projects live in ElevenPowers:

```
git clone https://github.com/SatyamSingh-Git/ElevenPowers
git clone https://github.com/SatyamSingh-Git/elevenpowers-multistage-cases
cd ElevenPowers
python -m eval.multistage_rehearse --cases ../elevenpowers-multistage-cases --case throttle --out rehearsal.json
```

You need Python 3.11 or newer, pytest and git. Files here are stored byte for
byte (`.gitattributes`), because the stage diffs must apply exactly.

## More

- **Design and analysis:** [the experiment design](https://github.com/SatyamSingh-Git/ElevenPowers/blob/main/docs/superpowers/specs/2026-10-23-multistage-field-claims-design.md)
- **Build record:** [results/multistage](https://github.com/SatyamSingh-Git/ElevenPowers/tree/main/results/multistage)
- **The developers' account behind the two claims:** [field report](https://github.com/SatyamSingh-Git/ElevenPowers/blob/main/docs/field/2026-10-real-project.md)
