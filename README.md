# Pay-by Clock (prototype)

Small prototype for the MSME 45-day payment rule: **Income-tax Act 1961 s.43B(h)**
(from tax year 2026-27 the same rule sits in **s.37(2)(g) of the Income-tax Act 2025**) and
**MSMED Act 2006 s.15**. Jurisdiction: India.

The idea: an AI "reader" turns messy text (a payment clause, a goods-receipt note) into a tiny
form with a word-for-word quote. Plain Python then works out the due date and the tax effect.
The reader never decides a date, a status or an amount.

## Files
| File | What it is |
|---|---|
| `rules.py` | The rule pack (about 100 readable lines). Dates, coverage, statuses. |
| `extract.py` | Three readers (gold / keyword baseline / LLM) and the guardrails between reader and rules. |
| `cases.json` | 16 synthetic cases (built by `make_cases.py`). Every vendor, amount and date is made up. |
| `run_tests.py` | Runs all cases in one mode and writes `results_<mode>.json`. |
| `test_guardrails.py` | Shows that invented quotes, low confidence and unclear clauses go to a human. |
| `SOURCES.md` | Where each rule comes from. |

## Run it
```
python3 make_cases.py        # optional: rebuilds cases.json
python3 run_tests.py gold    # rules engine alone, fed the correct reading
python3 run_tests.py baseline  # keyword reader + same rules engine (non-AI control)
python3 test_guardrails.py
# with a model (free tier on OpenRouter):
export OPENROUTER_API_KEY=...        # optional: export OPENROUTER_MODEL=<a free model id>
python3 run_tests.py llm
```
Only Python 3 standard library is needed.

## What I measured (honest version)
| Mode | Result (status AND due date both right) |
|---|---|
| gold | 16/16. Checks the code, not the AI. My hand-worked due dates agree with the engine. |
| baseline | 11/16. Misses: C07 (ignores the objection), C09 (no period, guesses 15 days), C11 (complaint still open), C13 ("thirty" written as a word), C16 (two instalments). All five are silent: a wrong date or a confident verdict where a person should look. |
| llm | **Not run yet.** The code is written but I could not call OpenRouter from where I built this. Run it, then add your score and model name here. |

Limits to keep in mind:
* I wrote the 16 cases myself, knowing the weak spots, so they are friendlier to the design than real files would be.
* A better regex would fix C13. It would not fix C07, C09, C11 or C16.
* `llm` mode's network call is untested. The validator it feeds was tested with fake outputs.
* An empty goods-receipt note is read as "no objection". In real life silence may just mean "nobody wrote it down".
* Year of deduction when a bill is not yet due at year end but is paid late is a contested point. The engine flags it for the CA.
* The synthetic vendors are not real. No real company or personal data is used.

## Simplifications
* Due date = acceptance date + N days, last day to pay inclusive. Practitioners may count the "appointed day" one day differently. The CA should confirm once.
* The tax year is taken from the delivery date.
* "Trader" and "mixed" vendors are set once by a human on the vendor card. Sources disagree on traders; see `SOURCES.md`.
