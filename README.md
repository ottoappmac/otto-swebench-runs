# Otto on SWE-bench Verified: run results

Results of running Otto's `swebench-coding-agent` on all 500 instances of
[SWE-bench Verified](https://www.swebench.com/). The harness that produced them is
`scripts/swebench` in the [otto](https://github.com/ottoappmac/otto) repo (PR #59); this repo
holds only the outputs, so a run can be inspected without re-running it.

## Result

**327 of 500 resolved (65.4%).**

| Setting | Value |
|---|---|
| Run | `202610271735`, Oct 27 to Nov 8 2026 |
| Model | `mlx-community/Qwen3.6-35B-A3B-8bit` through oMLX, local on Apple Silicon |
| Agent | `swebench-coding-agent`: no subagents, no memory, narrow toolbox |
| Limits | 5400 s and 200 tool calls per instance, 4 agent workers |
| Environment | Docker, LogicStar prebuilt images, no network |
| Scoring | `score.py --method local` with the LogicStar bundle images |

| Outcome | Instances |
|---|---|
| Resolved | 327 |
| Unresolved (patch applied, hidden tests failed) | 169 |
| Empty patch (no source edit) | 3 |
| Scoring error (tests ran past the 1800 s harness limit) | 1 |

| Repo | Resolved / scored |
|---|---|
| scikit-learn | 28 / 32 (88%) |
| pydata (xarray) | 16 / 22 (73%) |
| django | 161 / 231 (70%) |
| sympy | 51 / 75 (68%) |
| pytest-dev | 13 / 19 (68%) |
| astropy | 12 / 22 (55%) |
| matplotlib | 18 / 34 (53%) |
| sphinx-doc | 23 / 44 (52%) |
| pylint-dev | 2 / 10 (20%) |
| psf (requests) | 1 / 8 (13%) |
| mwaskom (seaborn) | 1 / 2 |
| pallets (flask) | 1 / 1 |

## Read this before quoting the number

- **Not a leaderboard number.** Scoring used LogicStar's compressed images, which were compared
  with the official ones on only 23 instances. Re-score with the official images
  (`--image-source official`) before citing it.
- **Empty patches were re-run once.** The first pass resolved 319 (63.8%) with 19 empty patches;
  re-running those 19 added 8 resolved. 63.8% is the single-attempt figure.
- **Unscored instances count as unresolved.** `django__django-15128` produced a patch whose test
  run exceeded the harness limit (it probably loops forever); the 3 empty patches are
  `django__django-11734`, `django__django-13513` and `sphinx-doc__sphinx-9229`.
- Scores depend on the model, the limits above and the prompt at the time of the run.

## Layout

```
out/202610271735/
  preds.jsonl            one prediction per instance: instance_id, model_name_or_path, model_patch
  results.json           score summary: resolved / unresolved ids, empty patches, errors
  score_ledger.jsonl     append-only scoring history, one line per scoring event
  otto.*.json            per-batch reports from the SWE-bench harness
  trajectories/<id>.json         run record: status, timings, full message and tool-call log
  trajectories/<id>.events.jsonl streamed events with timestamps
```

`preds.jsonl` is in the format the SWE-bench harness takes, so it can be re-scored directly:

```bash
python -m swebench.harness.run_evaluation \
  --dataset_name princeton-nlp/SWE-bench_Verified \
  --predictions_path out/202610271735/preds.jsonl \
  --max_workers 4 --run_id rescore
```

## Reproducing

From the otto repo (see `scripts/swebench/README.md` there):

```bash
scripts/swebench/bench.sh warm      # one time: dataset and image bundle
scripts/swebench/bench.sh run --workers 4 --offline --yes --out ../otto-swebench-runs/out/<run>
```

The harness writes `work/`, `home/`, `dataset/`, `http-cache/` and `backend.log` next to `out/`;
they are git-ignored here because they are large or machine-specific. Trajectories are about
200 MB, so the repo is large; use Git LFS or drop `trajectories/` if that matters.
