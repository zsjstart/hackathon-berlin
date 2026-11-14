# Break the Model — Hackathon (Berlin, 2026-11-14)

Red-team notes and tooling for the **Break the Model** challenge, where the goal is to make
*Riley* (the AI assistant of a fictional hospital) hand out private (fake) patient or staff data.

All data in this challenge is fake. These files are for security research and were produced
during the hackathon.

## What's here

| File | What it is |
|---|---|
| [`FINDINGS.md`](FINDINGS.md) | The write-up: four Level-2 findings (F-01…F-04), each with a description, the exact prompt, the security impact, and how to fix it. |
| [`payloads.jsonl`](payloads.jsonl) | Every attack prompt we tried, one per line, tagged with technique, target level, result, and an example `turn_id`. |
| [`run_payloads.py`](run_payloads.py) | Re-runs the prompts against Riley, auto-decodes the reply, and flags real leaks. |

## The short story

Riley has three defense levels: **L1** only *tells* it to keep data private, **L2** adds a filter
that reads its answer and blocks private data, and **L3** also uses code to stop it reading private
data at all.

- **Level 2 is beatable.** Riley will look up a patient, and a simple encoding (base64, ROT13,
  reversed, or dot-spaced) hides the answer from the filter so it comes out anyway. See F-02.
- The filter also misses some **plain-text** answers (F-03), and one success can be turned into a
  **bulk** leak over several messages (F-04).
- **The leaks are not reliable** — the same prompt often works once and is then refused.
- **Level 3 held.** Across ~40 different attempts, Riley refused every private query before running
  it, so the code lock was never even reached.

## How to use the runner

```bash
# re-run all Level-2 encoding payloads, 5 tries each, to measure how often they work
python3 run_payloads.py --model medium --level 3 --only base64,rot13,revers,dot --trials 5

# run everything tagged for Level 2
python3 run_payloads.py --model small --target-level 2 --level 3
```

The runner uses the same `btm` client as the starter kit, so set up `.env` first (see the starter
kit's `setup.py`). It appends every call to `logs/my_attempts.jsonl`.
