# metacognition-bench

**What two deterministic metacognition tools do to a small model's reasoning over a long, stateful chain.**

This repository is a benchmark instrument and the raw, verifiable data from running it. It does not measure whether a model gets answers *right*. It measures what happens to a model's *thinking* when, every single turn for a long run, it is forced to (a) diverge from the frame it is already assuming and (b) answer one question that challenges an assumption it did not state.

The two instruments are deliberately dumb: two single-file Python programs, no LLM, no network, no key, deterministic. Given the same input they always return the same output. That property is what makes the runs in here *provable*: the full execution transcript is included, and `verify/verify_calls.py` re-runs every tool call on its recorded input and confirms the output the model received is exactly what the tool produces now. A run in this repo cannot have been faked.

> **Status: prepared, not yet published as findings.** The data and verification are complete and reproducible. The observations are still being written ([`observations/`](observations/)). Nothing here claims a result yet. When the observation pass is done, the findings go in `observations/` and only then does this go out.

## The two instruments

| Tool | Move | Input | Output |
|---|---|---|---|
| [`tools/superposition.py`](tools/superposition.py) | **divergence** | `{"task","description","wants"}` | a two-pole tension: two readings of what you are doing and a question about which is real |
| [`tools/self_inspect.py`](tools/self_inspect.py) | **doubt** | a thought | one *metathought*: a short question that turns attention onto an unstated assumption |

Both are deterministic, keyless, stdlib-only. Source repos: [self-inspect](https://github.com/ejentum/self-inspect-mcp) · superposition. The copies in [`tools/`](tools/) are the exact files used by the runs here.

## The loop

Each turn, the model (holding the whole prior thread in one continuous context) does:

1. **Claim** its current best position on the turn's question, building on every earlier turn.
2. **Superposition** (mandatory): build a rich `{task,description,wants}` from the claim, run the tool, name the pole it was *already* assuming, then develop the **other** pole and reframe the claim from it. Taking the other pole is a gate, not a suggestion.
3. **Self-inspect** (recursive): run the doubt tool on the reframed claim, answer the metathought honestly.
4. **Insight + advance**: one crisp insight, then a genuinely new next question.

Full design, the exact orchestration prompt, and the known limitations are in [`methodology.md`](methodology.md).

## Runs

| Run | Model | Turns | State | Tool calls | Verified |
|---|---|---|---|---|---|
| [`runs/stateful-haiku-40turn`](runs/stateful-haiku-40turn/) | Claude Haiku 4.5 | 40 | continuous (one context) | 40 superposition + 40 self-inspect | yes, 80/80 re-run identical |

Each run directory holds: the raw execution `transcript.jsonl` (ground truth), the model's self-reported `journal.jsonl`, a reconstructed verbatim `full_loop.jsonl` (every turn's tool input/output plus the model's reasoning), a human-readable `full_loop.md`, and the exact `orchestration_prompt.txt`.

## Reproduce the proof

```sh
python verify/verify_calls.py runs/stateful-haiku-40turn
# self_inspect  re-run == logged: 40 ok, 0 mismatch
# superposition re-run == logged: 40 ok, 0 mismatch
# RESULT: ALL CALLS VERIFIED
```

## What this is honest about

- It measures **process**, not correctness. The claim is auditability and what structured doubt plus forced divergence *do* to a reasoning chain, never that the model's philosophy is true.
- The model's text is reproduced **verbatim** (including its own style and any transcript encoding artifacts). The `.jsonl` is faithful; the `.md` is lightly cleaned for reading only.
- No metric is reported until the observation pass in [`observations/`](observations/) substantiates it from this data.

## License

MIT. See [`LICENSE`](LICENSE).
