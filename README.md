# SWE-Xplorer Experiments

Raw trajectories and evaluation results from SWE-bench experiments comparing
**SWE-Xplorer** (a tree-search SWE agent) against baseline agents, across
multiple benchmarks and models.

## Layout

```
<Benchmark>/<Agent>/<Model>/<experiment>/<instance_id>/...
```

- **Benchmark**: `SWE-Bench-Verified`, `SWE-Bench-Multilingual`
- **Agent**: `SWE-Xplorer`, `mini-SWE-agent`, `SWE-Search`, `Claude-Code`, `OpenCode`, `OpenHands`
- **Model**: e.g. `gpt-5-mini`, `deepseek-v4-flash`, `qwen-2.5-7b`
- **Experiment**:
  - `main` — the headline run for that agent/model/benchmark combination
  - `ablations/<name>` (SWE-Xplorer only) — a component removed from the full method, e.g. `wo_reconcile`, `wo_pathlevel`
  - `scaling/<sN>` (SWE-Xplorer, SWE-Search) — the same run re-scored at a reduced search budget of `N` node expansions, for scaling-curve analysis. These contain only `predictions.json` + `report.json`/`report_ts.json` — no per-instance trajectories, since the candidate patches are reconstructed from the `main` run's already-completed search trees rather than from a fresh run.

Not every experiment/model combination exists for every agent — only the runs
that were actually performed are present.

## Per-instance file formats

Each agent produces different artifacts per SWE-bench instance:

**mini-SWE-agent** (linear agent, no search) — `<instance_id>/<instance_id>.traj.json`:
the full conversation (`messages`), plus `info.exit_status`, `info.submission`
(final patch), `info.model_stats`, `info.config`. Experiment-level files:
`preds.json` (SWE-bench predictions file), `minisweagent.log`, `exit_statuses_*.yaml`.

**SWE-Xplorer** (tree search) — per instance:
- `<instance_id>.traj.json` — the trajectory of the path ultimately submitted
- `<instance_id>.tree.json` — the full search tree (all explored nodes)
- `<instance_id>.rtv.json` — reward/value model output
- `experiment.log` — per-instance run log
- for some instances (mainly in SWE-Bench-Multilingual): a `.reproduction.*`
  counterpart of each file above, from a separate bug-reproduction sub-trajectory
  run before the main fix trajectory

Experiment-level files: `preds.json`, `minisweagent.log`, `run_statuses_*.yaml`.

**SWE-Search** (official Moatless Tree Search baseline) — per instance:
`trajectory.json` + `eval_result.json` only.

**Claude-Code / OpenCode** (run via the [Pier](https://github.com/mahirlabibdihan/pier)
CLI-agent wrapper) — per instance: `config.json`, `result.json`, `trial.log`,
`agent/` (`trajectory.json`, `model.patch`, raw CLI session transcripts under
`sessions/`), `verifier/` (test verification), `artifacts/`, `agent-build-context/`
(the Dockerfile used to build that instance's environment).

**OpenHands** — per instance: `<instance_id>.json` (full trajectory: `history`,
`test_result`, `metadata`, `report`), `infer.log`, and for instances that reached
the verification stage, an `eval/` folder (`report.json`, `eval.sh`,
`test_output.txt`, `patch.diff`, `run_instance.log`).

## Evaluation reports

- **`report.json`** — standard SWE-bench harness evaluation report
  (`total_instances`, `resolved_instances`, `resolved_ids`, `unresolved_ids`,
  `empty_patch_ids`, `error_ids`, ...), scored against the actual submitted
  patch per instance.
- **`report_ts.json`** (SWE-Xplorer only) — since a search tree can produce
  multiple complete candidate patches per instance, this evaluates *every*
  terminating candidate, not just the one submitted. Structure:
  ```json
  {
    "<instance_id>": [
      {"node_index": 0, "resolved": true,  "patch_exists": true, "patch_applied": true, "patch_is_none": false},
      {"node_index": 1, "resolved": false, "patch_exists": true, "patch_applied": true, "patch_is_none": false},
      ...
    ]
  }
  ```
  Candidates are ranked by the value model, so `node_index: 0` is the
  top-ranked candidate for that instance (closest to, but not always
  identical to, what was actually submitted). Not every instance has an
  entry at every node index — most trees produce only a few complete
  candidates per instance.

Some experiments don't yet have an evaluation report at all (evaluation is
still pending or was never run) — a missing `report.json`/`report_ts.json`
means the trajectories exist but haven't been scored.

## Large files

This repo uses [Git LFS](https://git-lfs.github.com) for individual files
over GitHub's 100MB limit.
