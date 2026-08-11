# agentseclab_x

Focused Module-First Development Specification

P15 GuardBreaker · P08 CrossJailbreak-1M · P05 MCPDrift

Detailed implementation plan for M0, M2, M4, M11, and M12

Prepared for Md Jahangir Alam | Scope: build the first reusable substrate and finish RQ1-RQ2 evidence for P15, P08, and P05 in seven days. This is a separate write-up expanding Section 16.3 of the module-first Top-15 plan.

1. Operating Goal and Scope

For now, develop only P15, P08, and P05 using the substrate modules M0, M2, M4, M11, and M12. The plan treats these five modules as the paper factory: experiment identity, event logging, episode execution, artifact packaging, and reporting. Paper-specific analysis code is allowed only as thin plugins under papers/P15, papers/P08, and papers/P05; once stable, those plugins can later be promoted into M6, M8, M9, or M10.

Paper

Core claim

Mandatory 7-day RQs

Finish line with only M0/M2/M4/M11/M12

P15 GuardBreaker

Composed jailbreak defenses can look robust in static matrices but fail when attacks adapt to the deployed pipeline.

RQ1 adaptive-vs-static ASR; RQ2 defense-composition monotonicity; RQ3 feedback-signal leverage if budget allows.

Day 3: attack/eval configs, adaptive-vs-static table, composition gain/loss table, artifact README.

P08 CrossJailbreak-1M

Organic jailbreak attempts form transfer clusters and model vulnerability partial orders different from synthetic benchmark rankings.

RQ1 strategy clusters; RQ2 asymmetric cross-model transfer; RQ3 replay validation if data/model access is ready.

Day 3: cluster assignments, transfer matrix, safety-clique graph/table, artifact README.

P05 MCPDrift

Semantic, schema, permission, and destination drift predicts security-relevant MCP updates better than hash/version changes.

RQ1 drift detection; RQ2 changed-component localization; RQ3 early-warning/scanner comparison if labels are ready.

Day 4: 50-150 labeled release/update pairs, drift feature table, localization table, artifact README.



2. Critical Path: Build These Modules First

Order

Module

Concrete output

Which paper finishes because of it

Day 1 AM

M0 Reproducibility Core

paper manifests, IDs, config schemas, registries, cache, ledger, run folders

All three papers can register experiments and create reproducible run folders.

Day 1 PM

M2 Event and Provenance Schema

validated JSONL event schema with P15/P08/P05-specific event types

P15/P08/P05 can log comparable evidence without paper-specific logging.

Day 2 AM

M4 Benchmark Harness

task interface, adapter interface, evaluator interface, matrix runner, dummy task pack

P15/P08 can run RQ1-RQ2 pilots; P05 can run static diff tasks as M4 task packs.

Day 2 PM

M11 Paper/Artifact Factory

paper folders, data cards, README templates, artifact bundles

All three have submission-style artifact shells from first result.

Day 2 PM-3 AM

M12 Reporting Layer

result aggregator, RQ dashboard, paper-specific table templates, failure-case exporter

P15/P08 get Day-3 tables; P05 gets Day-4 tables after labels land.



2.1 Minimal Repository Tree

agentseclab_x/

  core/

    ids.py                 # M0: stable IDs and run hashes

    config.py              # M0: typed config loaders and validators

    registry.py            # M0: model/task/attack/defense/paper registries

    cache.py               # M0: response/tool-output cache

    events.py              # M2: event dataclasses/Pydantic models

    event_writer.py         # M2: JSONL append/validate/write helpers

    runner.py              # M4: run_episode()

    matrix_runner.py        # M4: run_matrix()

  benchmarks/

    base_task.py            # M4: TaskPack interface

    jailbreak/              # P15/P08 task packs

    mcpdrift/               # P05 release/update task pack

  evaluators/

    base.py                 # M4: evaluator interface

  reporting/

    aggregate.py            # M12: event -> result rows

    rq_dashboard.py         # M12: RQ status tracker

    make_tables.py          # M12: paper tables

    make_figures.py         # M12: paper figures

    export_cases.py         # M12: representative cases

  artifacts/

    make_artifact.py        # M11: bundle builder

    templates/              # M11: README, data card, Open Science appendix

  papers/

    P15_GuardBreaker/

    P08_CrossJailbreak1M/

    P05_MCPDrift/

  configs/

    models.yaml

    papers.yaml

    datasets.yaml

    attacks.yaml

    defenses.yaml

    experiments/

  schemas/

    event_schema.json

    result_schema.json

  tests/

3. Existing Codebases to Reuse: Why and How

Codebase / source

Use for

Why this codebase

Concrete reuse instruction

JailbreakBench / jailbreakbench

P15 and P08; M0/M4/M11/M12

Standardized jailbreak behaviors, artifacts, scoring conventions, and reproducibility patterns.

Mirror behavior IDs and artifact folders; use its behavior/scoring separation as your configs/datasets.yaml and reporting templates.

HarmBench

P15 and P08; evaluator utilities

HarmBench provides harmful-behavior taxonomy and classifier/evaluation culture for automated red-teaming.

Use behaviors as seed categories; wrap classifier/judge output into M2 JudgeScoreEvent; reuse behavior-level tables.

StrongREJECT

P08 and P15 output scoring

Richer refusal/compliance scoring than binary keyword refusal; useful for organic compliance labels.

Call evaluator in a paper-specific scoring plugin; store score, refusal flag, and explanation hash as M2 JudgeScoreEvent.

EasyJailbreak package / framework

P15 attack integration

Selector/Mutator/Constraint/Evaluator abstraction maps cleanly to adaptive attack loops.

Implement an AttackAdapter wrapper that exposes generate(), adapt(), reset(), cost(); do not modify upstream package.

PAIR / JailbreakingLLMs

P15 adaptive black-box baseline

PAIR is a known iterative black-box attack engine and is already listed in the source plans.

Wrap PAIR as attack_backend=pair in configs/attacks.yaml; log every candidate as AttackCandidateEvent.

GPTFuzz

P15 mutation search baseline

Mutation/fuzzing operators support adaptive objective search and feedback-channel ablations.

Expose mutators as P15 plugins; each mutation receives previous prompt, guard result, and target result.

AutoDAN

P15 stealth-oriented optimization baseline

Optimization baseline for stealthy jailbreak prompts; useful as a stronger but slower stretch backend.

Keep optional; run on local/open model only if Day-3 P15 table is already complete.

llm-attacks / GCG

P15 white-box suffix baseline

White-box baseline for local models; useful for contrast with black-box adaptation.

Use only when local model gradients are available; otherwise mark as deferred.

LMSYS-Chat-1M dataset

P08 organic data source

Provides real-world conversations, model names, language tags, and moderation flags.

Load streaming Parquet; create P08 jailbreak_corpus.parquet with conv_id, model, language, prompt, response, moderation flag, compliance label.

modelcontextprotocol/servers

P05 benign corpus and schema examples

Reference/community MCP server corpus with manifests and schemas.

Clone selected repos with commit hashes; extract tool names, parameter schemas, manifests, release tags.

modelcontextprotocol/python-sdk

P05 replay-compatible schema understanding

Official SDK for local MCP server/client behavior when you later add replay; not required for static RQ1-RQ2.

Use schema examples and optional local replay; record server metadata as ReleaseSnapshotEvent.

Snyk agent-scan

P05 scanner baseline

Security scanner for AI agents, MCP servers, and skills.

Run scanner on selected snapshots; record findings as ScannerFindingEvent; use as baseline against drift score.

GitPython / git CLI

P05 release history collection

Fast, deterministic collection of commits, tags, diffs, and changed files.

Use git clone --bare or GitPython to get release pairs; write snapshot manifests and diff records.

Gitleaks / TruffleHog / Semgrep

P05 optional static baselines

Secret/static scans provide simple baseline signals for risky updates.

Run only on controlled public/local snapshots; save JSON output and normalize into M2 ScannerFindingEvent.



4. M0 — Reproducibility Core

Priority: Day 1 morning

Purpose: Every run, paper, RQ, dataset, attack, defense, model, seed, and artifact has an immutable identity. M0 prevents the three papers from becoming three unrelated codebases.

4.x.1 Codebases to reuse and why

Codebase/source

Part to reuse

How to use

JailbreakBench

Behavior/artifact and scoring conventions

Use for paper manifests, behavior IDs, and reproducibility culture for P15/P08.

HarmBench

Behavior taxonomy and evaluator style

Use for P15/P08 behavior categories and metric naming.

PurpleLlama CybersecurityBenchmarks

Runner/model adapter/result organization

Use organization patterns for configs/results even though these three papers are not cyber-range heavy.



4.x.2 Files to create with input/output contracts

File

What goes inside / input

Output / consumed by

agentseclab_x/core/ids.py

Inputs: config dict, git commit, seed, timestamp policy. Output: stable run_id, scenario_id, config_hash, artifact_id. Contains make_run_id(), make_config_hash(), short_id().

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/config.py

Inputs: YAML files from configs/. Output: validated ExperimentConfig, PaperManifest, DatasetManifest, ModelConfig, AttackConfig, DefenseConfig objects. Contains schema validation and defaults.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/registry.py

Inputs: configs/models.yaml, datasets.yaml, attacks.yaml, defenses.yaml, papers.yaml. Output: registry lookup objects used by M4. Contains get_model(), get_taskpack(), get_attack(), get_defense(), get_paper().

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/cache.py

Inputs: request hash, provider/model, prompt/tool args. Output: cached response JSON or cache miss. Contains read_cache(), write_cache(), cache_key().

Consumed by M4/M11/M12 and paper-specific plugins.

configs/papers.yaml

Inputs: manually written paper entries. Output: paper metadata for P15/P08/P05 including RQs, primary metric, module list, output folder, owner.

Consumed by M4/M11/M12 and paper-specific plugins.

configs/experiments/*.yaml

Inputs: per-RQ experiment definitions. Output: executable experiment matrix for M4.

Consumed by M4/M11/M12 and paper-specific plugins.

artifacts/experiment_ledger.csv

Inputs: append-only run metadata. Output: daily ledger consumed by M12 and M11.

Consumed by M4/M11/M12 and paper-specific plugins.

scripts/freeze_manifest.py

Inputs: experiment folder. Output: manifest.json containing configs, code commit, dependency lock hash, dataset version, run IDs.

Consumed by M4/M11/M12 and paper-specific plugins.



4.x.3 Detailed development steps

Initialize repository and configs/ directory; create P15, P08, P05 paper manifests first.

Define ID hierarchy: paper_id -> rq_id -> dataset_id -> scenario_id -> run_id. Use stable hashes rather than manual names.

Create default experiment templates: p15_rq1_adaptive_vs_static.yaml, p15_rq2_composition.yaml, p08_rq1_clusters.yaml, p08_rq2_transfer.yaml, p05_rq1_drift_detection.yaml, p05_rq2_localization.yaml.

Add a config validator that fails fast when a required field is absent.

Add run-folder constructor that creates runs/{paper_id}/{rq_id}/{run_id}/ with config.yaml, manifest.json, events.jsonl, metrics.csv, logs/, tables/, figures/.

Add cache and cost ledger hooks even if you start with dummy/local outputs; do not retrofit logging after experiments.

4.x.4 Acceptance test and paper unblocking effect

Acceptance test: `python scripts/init_experiment.py --paper P15 --rq RQ1 --config configs/experiments/p15_rq1_adaptive_vs_static.yaml` creates a run folder with config.yaml, manifest.json, events.jsonl, metrics.csv, and README_stub.md. Repeat for P08 RQ1 and P05 RQ1.

Paper

Unlocked after M0

P15

Experiment configs for adaptive/static runs and defense-composition matrix.

P08

Corpus mining and cluster/transfer run manifests.

P05

Release-pair manifest and label ledger.



4. M2 — Unified Event and Provenance Schema

Priority: Day 1 afternoon

Purpose: All evidence becomes one validated JSONL format. M2 is what allows M12 to generate tables without paper-specific parsers.

4.x.1 Codebases to reuse and why

Codebase/source

Part to reuse

How to use

AgentDojo

Task/injection/defense/security result separation

Use the separation principle, then generalize into a unified event log.

CyberSecEval-MCP plan

Tool, argument, observation, trust label, policy boundary fields

Use field vocabulary for P05/P15 and future compatibility.

LMSYS-Chat-1M metadata

conversation_id, model, language, moderation fields

Normalize P08 records into M2 DatasetRecord and ModelResponseEvent.



4.x.2 Files to create with input/output contracts

File

What goes inside / input

Output / consumed by

schemas/event_schema.json

Inputs: event JSON objects. Output: validation pass/fail. Defines required fields and paper-specific extensions.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/events.py

Inputs: Python event constructor calls. Output: typed event dictionaries. Classes: RunStart, DatasetRecord, AttackCandidateEvent, ModelResponseEvent, GuardDecisionEvent, JudgeScoreEvent, ClusterEvent, TransferEdgeEvent, ReleaseSnapshotEvent, DiffFeatureEvent, ScannerFindingEvent, FinalMetricEvent.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/provenance.py

Inputs: parent_event_ids and source metadata. Output: provenance chains and content hashes. Contains hash_content(), make_parent_chain(), trust_summary().

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/event_writer.py

Inputs: Event objects and run directory. Output: append-only events.jsonl with schema validation. Contains write_event(), load_events(), validate_event_file().

Consumed by M4/M11/M12 and paper-specific plugins.

tests/test_event_schema.py

Inputs: hand-authored traces. Output: pytest pass/fail for clean run, successful attack, blocked attack, failed scanner, transfer edge, release drift.

Consumed by M4/M11/M12 and paper-specific plugins.

docs/EVENT_SCHEMA.md

Inputs: schema. Output: human-readable documentation and examples for students.

Consumed by M4/M11/M12 and paper-specific plugins.



4.x.3 Detailed development steps

Create a base event with universal fields: event_id, run_id, paper_id, rq_id, scenario_id, step_id, event_type, timestamp, actor, source_channel, trust_level, content_hash, parent_event_ids, outcome.

Add P15 fields: attack_name, attack_family, defense_pipeline, feedback_signal, candidate_text_hash, guard_labels, judge_scores, query_count, adaptive_round.

Add P08 fields: conv_id, source_model, target_model, cluster_id, compliance_label, language, harm_category, transfer_source, transfer_target, support_n, ci_low, ci_high.

Add P05 fields: repo_id, server_name, version_from, version_to, changed_component, diff_type, permission_delta, schema_delta, destination_delta, scanner_name, scanner_finding, gold_label.

Ensure every event can be anonymized: text fields stored as hash by default; safe text snippets go in cases/ only after manual review.

Write hand-authored example traces before writing adapters; this makes M12 implementation deterministic.

4.x.4 Acceptance test and paper unblocking effect

Acceptance test: `pytest tests/test_event_schema.py` validates at least six traces: P15 adaptive candidate, P15 defense-composition run, P08 cluster event, P08 transfer edge, P05 diff feature, P05 scanner finding. M12 must compute at least one dummy metric from each.

Paper

M2 event types needed

P15

AttackCandidateEvent, GuardDecisionEvent, JudgeScoreEvent, FeedbackSignalEvent, FinalMetricEvent.

P08

DatasetRecord, ConversationEvent, ComplianceLabelEvent, ClusterEvent, TransferEdgeEvent, ReplayEvent.

P05

ReleaseSnapshotEvent, DiffFeatureEvent, DriftLabelEvent, ScannerFindingEvent, FinalMetricEvent.



4. M4 — Benchmark Harness

Priority: Day 2 morning

Purpose: M4 executes experiments defined by M0 and logs them through M2. For these three papers, M4 is mostly offline/semioffline: jailbreak runs, organic-data analysis runs, and release-diff tasks.

4.x.1 Codebases to reuse and why

Codebase/source

Part to reuse

How to use

JailbreakBench

behavior/scoring/run separation

Use for P15/P08 taskpack structure.

HarmBench

red-team evaluation utilities

Use as evaluator baseline where applicable.

modelcontextprotocol/servers + git

release/schema corpus

Use for P05 release drift taskpack.



4.x.2 Files to create with input/output contracts

File

What goes inside / input

Output / consumed by

agentseclab_x/benchmarks/base_task.py

Defines TaskPack interface. Input: config and data path. Output: iterable scenarios with success_oracle and violation_oracle.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/evaluators/base.py

Defines Evaluator interface. Input: M2 event file or model outputs. Output: metric events and result rows.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/runner.py

Input: one ExperimentConfig. Output: one run folder with events.jsonl and metrics.csv. Contains run_episode() and run_offline_analysis().

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/matrix_runner.py

Input: experiment matrix YAML. Output: multiple run folders and aggregate run index. Supports checkpoint/resume.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/benchmarks/jailbreak/guardbreaker_task.py

P15. Input: behaviors, attack backends, defense pipelines. Output: attack attempts and guard/judge events.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/benchmarks/jailbreak/organic_transfer_task.py

P08. Input: jailbreak_corpus.parquet or sample CSV. Output: cluster events, transfer edges, replay events.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/benchmarks/mcpdrift/release_drift_task.py

P05. Input: snapshot manifest pairs. Output: release snapshot events, diff feature events, scanner finding events, drift labels.

Consumed by M4/M11/M12 and paper-specific plugins.

configs/experiments/*.yaml

Input to matrix_runner. Each config has paper_id, rq_id, dataset, taskpack, model/evaluator, baseline, seeds, output_path.

Consumed by M4/M11/M12 and paper-specific plugins.



4.x.3 Detailed development steps

Implement BaseTaskPack and BaseEvaluator before paper-specific task packs.

Create a dummy model/guard/evaluator so the harness can be tested without API keys.

Implement P15 GuardBreakerTaskPack: read behaviors, generate static attacks, run one adaptive backend, log guard labels, target responses, judge scores, query counts.

Implement P08 OrganicTransferTaskPack: load corpus, filter compliant/refusal labels, embed/cluster or load precomputed clusters, compute directed transfer matrix and support thresholds.

Implement P05 ReleaseDriftTaskPack: read repo/version pairs, compute basic diffs, run scanner JSON if available, emit drift features and gold labels.

Add resume logic: if a run folder already has finished.ok, skip unless --force is set.

Add exception handling as M2 ErrorEvent; failed runs are evidence, not silent missing cells.

4.x.4 Acceptance test and paper unblocking effect

Acceptance test: `python -m agentseclab_x.core.matrix_runner --config configs/experiments/smoke_p15_p08_p05.yaml` runs one tiny scenario per paper and writes events.jsonl, metrics.csv, and run_index.csv.

Paper

First runnable taskpack

P15

GuardBreakerTaskPack: behaviors x attack backend x defense pipeline x seed.

P08

OrganicTransferTaskPack: corpus -> clusters -> transfer edges -> matrices.

P05

ReleaseDriftTaskPack: release pairs -> diff features -> drift labels -> scanner baselines.



4. M11 — Paper and Artifact Factory

Priority: Day 2 afternoon

Purpose: M11 ensures each paper has a reproducible artifact package from the first successful run. It prevents final-week scrambling.

4.x.1 Codebases to reuse and why

Codebase/source

Part to reuse

How to use

JailbreakBench

artifact release conventions

Use README/data structure patterns for P15/P08.

HarmBench

behavior documentation style

Use behavior/category documentation patterns.

USENIX/NDSS artifact expectations

README, smoke test, Open Science appendix

Generate artifact shell from day one.



4.x.2 Files to create with input/output contracts

File

What goes inside / input

Output / consumed by

agentseclab_x/artifacts/make_artifact.py

Inputs: paper_id and run IDs. Output: artifact zip/folder with README, configs, sample traces, tables, figures, data card, smoke command.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/artifacts/templates/README.md

Template. Contains install, smoke run, expected outputs, safety constraints, reproduction commands.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/artifacts/templates/DATA_CARD.md

Template. Contains dataset source, inclusion criteria, labels, splits, safety filtering, limitations.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/artifacts/templates/OPEN_SCIENCE.md

Template. Contains artifact availability, reproducibility, compute cost, excluded materials, ethics/safety.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/papers/P15_GuardBreaker/

Contains paper_manifest.yaml, RQ_STATUS.md, threat_model.md, tables/, figures/, cases/, appendix/, artifact_notes.md.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/papers/P08_CrossJailbreak1M/

Same structure plus data_access.md and privacy_filtering.md.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/papers/P05_MCPDrift/

Same structure plus release_label_policy.md and scanner_baselines.md.

Consumed by M4/M11/M12 and paper-specific plugins.

scripts/anonymize_artifact.py

Inputs: artifact folder. Output: anonymized artifact folder. Removes usernames, absolute paths, API logs, credentials, and unsafe payload text.

Consumed by M4/M11/M12 and paper-specific plugins.



4.x.3 Detailed development steps

Create paper folders on Day 1; do not wait for final results.

Generate README and data-card drafts from M0 manifests and M2 schema automatically.

Add a smoke config for each paper: p15_smoke.yaml, p08_smoke.yaml, p05_smoke.yaml. These should run in under five minutes on a laptop using dummy/local data.

Export only safe artifacts: configs, hashes, synthetic canaries, sampled sanitized traces, aggregate tables, and documentation. Do not export raw harmful prompts unless separately approved and sanitized.

Add artifact status to RQ_STATUS.md: smoke_passed, result_regenerated, table_regenerated, anonymized, ready_for_internal_review.

4.x.4 Acceptance test and paper unblocking effect

Acceptance test: `python -m agentseclab_x.artifacts.make_artifact --paper P15 --runs runs/P15/...` creates a zip containing README, configs, sample events, metrics, tables, figures, and data card. Repeat for P08 and P05.

Paper

Artifact folder must contain

P15

attack configs, defense-pipeline configs, sample sanitized events, adaptive/static table, composition table.

P08

data-card, privacy filtering note, cluster assignment sample, transfer matrix sample, replay config.

P05

release label policy, snapshot manifests, diff features, scanner outputs, drift tables.



4. M12 — Reporting Layer

Priority: Day 2 evening to Day 3 morning

Purpose: M12 turns M2 event logs into RQ dashboards, tables, figures, and failure cases. It includes a lightweight metric registry for the three selected papers.

4.x.1 Codebases to reuse and why

Codebase/source

Part to reuse

How to use

Jailbreak security plan

one-row-per-attempt reporting and heatmaps

Use for P15 adaptive attack tables and P08 transfer matrices.

HarmBench/JailbreakBench

behavior-level result tables

Use for P15/P08 reporting.

Pandas/Matplotlib/NetworkX

lightweight analysis and figure generation

Use to avoid manual spreadsheets and regenerate figures from logs.



4.x.2 Files to create with input/output contracts

File

What goes inside / input

Output / consumed by

agentseclab_x/reporting/result_schema.py

Defines universal result row. Input: M2 events. Output: normalized rows with paper_id, rq_id, metric values, confidence fields.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/aggregate.py

Inputs: run folders or events.jsonl. Output: results.parquet/csv and aggregate metrics.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/rq_dashboard.py

Inputs: results + paper manifests. Output: RQ_STATUS.md and dashboard.csv with NOT_STARTED/DATA_READY/RUNNING/RESULT_READY/TABLE_READY/DRAFT_READY.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/make_tables.py

Inputs: results.parquet and paper_id. Output: paper tables in CSV, Markdown, and LaTeX/DOCX-ready text.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/make_figures.py

Inputs: results.parquet. Output: PNG/PDF heatmaps, line plots, transfer graphs, drift timelines.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/export_cases.py

Inputs: events + metrics. Output: sanitized true positives, false negatives, false positives, representative case studies.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/templates/*.yaml

Per-paper table definitions for P15/P08/P05.

Consumed by M4/M11/M12 and paper-specific plugins.



4.x.3 Detailed development steps

Define universal result row: run_id, paper_id, rq_id, scenario_id, dataset_id, attack_id, model, defense, seed, primary_metric, TSR, ASR, utility, cost, latency, ci_low, ci_high, status.

Implement P15 metrics: static_ASR, adaptive_ASR, adaptive_gain, guard_evasion_rate, composition_gain_loss, feedback_ablation_effect, queries_to_bypass.

Implement P08 metrics: n_clusters, cluster_purity, transfer_T_ij, support_n, bootstrap_CI_width, clique_count, vulnerability_partial_order_edges, replay_success_rate.

Implement P05 metrics: drift_precision, drift_recall, feature_auc, changed_component_F1, first_risky_version_accuracy, scanner_agreement, early_warning_lead.

Generate paper-specific tables: P15 adaptive-vs-static table and composition matrix; P08 transfer heatmap and cluster table; P05 drift-feature ablation and localization table.

Export sanitized case studies: no raw harmful prompt by default; show hashes, categories, and safe summaries.

4.x.4 Acceptance test and paper unblocking effect

Acceptance test: `python -m agentseclab_x.reporting.rq_dashboard --papers P15 P08 P05 --runs runs/` writes dashboards and at least one table per paper using smoke results; with real results it marks RQ1-RQ2 as TABLE_READY.

Paper

Main M12 outputs

P15

Table 1 adaptive vs static ASR; Table 2 defense-composition gain/loss; Figure 1 feedback ablation.

P08

Figure transfer heatmap; Table strategy clusters; Graph safety cliques/partial order.

P05

Table drift-feature ablation; Table changed-component localization; Figure release timeline.



5. Paper-Specific Implementation Plans Using Only the Five Core Modules

The following plans intentionally use M0, M2, M4, M11, and M12 only. Where the full Top-15 plan would use M5/M6/M8/M9/M10, this document implements a minimal paper-local plugin and logs it through M2. After the first seven days, promote the stable plugin into the full shared module.

5.1 P15 GuardBreaker

RQ

Question

Core-module-only implementation

RQ1

How much does adaptive PAIR/TAP/GPTFuzz/AutoDAN/GCG-style search increase ASR over non-adaptive attacks against single defenses?

Use P15 AttackAdapter plugins under M4. Run static template baseline and one adaptive backend with matched query budgets. M12 computes adaptive_gain = adaptive_ASR - static_ASR.

RQ2

Does composing multiple guards monotonically improve security, or can composition create new bypass surfaces?

Define defense_pipeline in configs/defenses.yaml: none, pre_guard, post_guard, pre+post, pre+target-refusal+post. M4 runs the same attacks across pipelines; M12 computes composition_gain_loss.

RQ3

Which feedback signals provide the most leverage to an adaptive attacker?

Stretch with same modules: configure feedback_signal = accept/reject, score, label, latency, target_response. M12 outputs feedback ablation table.



P15 files to create

File

Input

Output / content

papers/P15_GuardBreaker/paper_manifest.yaml

Manual paper metadata

RQs, primary metrics, module list, target venues, owner, dataset/behavior list.

configs/experiments/p15_rq1_adaptive_vs_static.yaml

behavior set, attack backends, defenses, target models, budget

M4 matrix over static vs adaptive runs.

configs/experiments/p15_rq2_composition.yaml

same behavior set, defense pipeline list

M4 matrix over defense compositions.

agentseclab_x/benchmarks/jailbreak/guardbreaker_task.py

M0 config + behavior records + attack adapter

M2 events: AttackCandidateEvent, GuardDecisionEvent, JudgeScoreEvent.

papers/P15_GuardBreaker/plugins/attack_adapters.py

EasyJailbreak/PAIR/GPTFuzz/AutoDAN wrappers

common generate/adapt/reset/cost interface.

papers/P15_GuardBreaker/plugins/guard_pipelines.py

defense names and order

serial pipeline object that emits guard decisions.

papers/P15_GuardBreaker/analysis/p15_metrics.yaml

M12 metric definitions

adaptive_gain, guard_evasion_rate, queries_to_bypass, composition_gain_loss.



P15 7-day steps

Day 1: Create paper manifest, behavior subset, target model list, and defense pipeline list. Use 50-100 safe benchmark behaviors or sanitized IDs; avoid raw dangerous text in release artifacts.

Day 2: Implement GuardBreakerTaskPack and dummy attack backend; confirm M2 logs attack candidates, guard labels, judge scores, and costs.

Day 3: Integrate one real adaptive backend first: PAIR or GPTFuzz. Run RQ1 static vs adaptive on 3 attacks x 3 guards x 2 target models x small budget. Generate P15 Table 1.

Day 3 evening: Run RQ2 composition matrix: no defense, pre-guard, post-guard, pre+post. Generate composition gain/loss table. This is the paper finish line for RQ1-RQ2.

Day 4-5: Add feedback-channel ablation for RQ3 if RQ1/RQ2 are stable: accept/reject only, guard label, guard score, target response.

Day 6: Add sanity checks: same query budget, same seed set, same behavior set, benign utility controls, failure cases.

Day 7: Freeze artifact, create sanitized representative cases, write RQ_STATUS.md, update threat_model.md and limitations.

5.2 P08 CrossJailbreak-1M

RQ

Question

Core-module-only implementation

RQ1

What jailbreak strategy clusters emerge from large-scale organic conversations?

M4 runs an offline corpus pipeline. It loads conversations, filters candidate jailbreak/unsafe records, embeds or uses precomputed embeddings, clusters, emits ClusterEvent rows.

RQ2

Do successful jailbreaks transfer asymmetrically across models, forming safety cliques or vulnerability partial orders?

M4 computes cluster-level directed transfer T[i,j]; M12 produces heatmap, support table, clique table, and partial-order edge table.

RQ3

Can top organic clusters be replay-validated on current models under controlled budgets?

Stretch with same M4 harness: use top 10 clusters and evaluate current target models with sanitized prompts or internal approval-only samples.



P08 files to create

File

Input

Output / content

papers/P08_CrossJailbreak1M/paper_manifest.yaml

paper metadata

RQs, data access assumptions, privacy filtering, artifact policy.

configs/experiments/p08_rq1_clusters.yaml

corpus path, embedding model, clustering params

M4 clustering pipeline config.

configs/experiments/p08_rq2_transfer.yaml

cluster file, compliance labels, model list

M4 transfer matrix pipeline config.

agentseclab_x/benchmarks/jailbreak/organic_transfer_task.py

jailbreak_corpus.parquet or sample CSV

ClusterEvents, TransferEdgeEvents, matrix outputs.

papers/P08_CrossJailbreak1M/plugins/lmsys_loader.py

LMSYS Parquet or local sample

normalized records: conv_id, model, language, prompt_hash, response_hash, moderation flag.

papers/P08_CrossJailbreak1M/plugins/compliance_labeler.py

responses + scorer

compliance_label, confidence, evaluator version.

papers/P08_CrossJailbreak1M/plugins/cluster_transfer.py

embeddings/clusters/compliance labels

cluster assignment table, T matrix, support table, bootstrap CIs.

papers/P08_CrossJailbreak1M/privacy_filtering.md

manual policy

how raw prompts/responses are filtered, hashed, summarized, and excluded from public artifacts.



P08 7-day steps

Day 1: Confirm data path: full LMSYS-Chat-1M, local preprocessed sample, or internal sanitized subset. Create data_access.md documenting what is available.

Day 2: Build lmsys_loader.py and produce a small normalized corpus with conv_id, model, language, harm/moderation flag, prompt_hash, response_hash, and safe summary.

Day 3: Run RQ1 clustering on 10k-50k candidates or the maximum available local sample. Output cluster_assignments.parquet, cluster_summary.csv, and M2 ClusterEvents.

Day 3 evening: Run RQ2 transfer on cluster-level compliance labels. Output transfer_matrix.csv, support_matrix.csv, and bootstrap_CI.csv. This is the RQ1-RQ2 finish line.

Day 4-5: Run replay validation for RQ3 on top 10 clusters if you have model access and approval to use sanitized cluster representatives.

Day 6: Add robustness checks: minimum support thresholds, alternate clustering method, high-confidence-only labels.

Day 7: Package sanitized atlas, matrix, cluster summaries, and artifact README; exclude raw harmful text unless separately approved.

5.3 P05 MCPDrift

RQ

Question

Core-module-only implementation

RQ1

Can semantic, permission, schema, and destination drift predict security-relevant MCP updates better than hash/version changes?

Use a paper-local P05 diff plugin under M4 instead of full M6. It loads snapshot pairs, computes feature rows, compares against labels, and M12 reports precision/recall/AUC.

RQ2

Which drift features best localize the changed component that introduced risk?

Each DiffFeatureEvent includes changed_component. M12 computes localization accuracy/F1 by feature group.

RQ3

Can drift scoring provide early warning before runtime attack success or incident-like behavior?

Stretch: add time/release index and scanner finding time. M12 computes lead time against labeled first-risky version.



P05 files to create

File

Input

Output / content

papers/P05_MCPDrift/paper_manifest.yaml

paper metadata

RQs, repo list, label policy, drift features, baselines.

configs/experiments/p05_rq1_drift_detection.yaml

repo list, version pairs, feature flags

M4 release-drift task config.

configs/experiments/p05_rq2_localization.yaml

diff feature rows, labels

changed-component localization experiment.

agentseclab_x/benchmarks/mcpdrift/release_drift_task.py

snapshot manifest pairs

M2 ReleaseSnapshotEvents, DiffFeatureEvents, DriftLabelEvents.

papers/P05_MCPDrift/plugins/repo_collector.py

GitHub URLs or local repos

snapshot_manifest.jsonl with commit/tag/version IDs.

papers/P05_MCPDrift/plugins/diff_features.py

two snapshots

hash_delta, semantic_delta, schema_delta, permission_delta, endpoint_delta, dependency_delta.

papers/P05_MCPDrift/plugins/scanner_runner.py

snapshot path + scanner config

scanner findings normalized into M2 ScannerFindingEvent.

papers/P05_MCPDrift/release_label_policy.md

manual label guidelines

definition of security_relevant, changed_component, first_risky_version, evidence notes.



P05 7-day steps

Day 1: Select 30-50 MCP servers/repos first. Prioritize repos with visible tags/releases/commit history and readable schemas/manifests.

Day 2: Implement repo_collector.py and create snapshot manifests. If releases are sparse, use commit windows or curated v0/v1 pairs; record this in limitations.

Day 3: Implement diff_features.py with four feature groups: semantic description change, schema broadening, permission/scope change, new external destination. Output diff_features.parquet and M2 DiffFeatureEvents.

Day 3-4: Label 50-150 release/update pairs using release_label_policy.md. Minimum viable P05 needs 50 high-quality labels; strong result needs 150+.

Day 4: Run RQ1 drift detection and RQ2 localization. Generate feature ablation and localization tables. This is the RQ1-RQ2 finish line.

Day 5: Add scanner baseline with Snyk agent-scan and optional Gitleaks/TruffleHog/Semgrep. Compute scanner agreement and early-warning pilot.

Day 6: Add robustness check: benign-looking description/schema evasions and temporal holdout.

Day 7: Package snapshot manifests, label policy, feature tables, scanner outputs, and sanitized examples.

6. Seven-Day Execution Schedule and Finish-Line Mapping

Day

Module focus

P15 finish line

P08 finish line

P05 finish line

Day 1 AM

M0 configs, run IDs, paper manifests

P15 config and run folders initialized.

P08 data access and corpus manifest initialized.

P05 repo list and label ledger initialized.

Day 1 PM

M2 event schema and validators

P15 attack/guard/judge events validate.

P08 dataset/cluster/transfer events validate.

P05 snapshot/diff/scanner events validate.

Day 2 AM

M4 runner and taskpack API

P15 smoke run with dummy attack/guard.

P08 smoke run on small sample.

P05 smoke run on one release pair.

Day 2 PM

M11 artifact factory + M12 reporting skeleton

P15 RQ dashboard exists.

P08 RQ dashboard exists.

P05 RQ dashboard exists.

Day 3

P15/P08 experiments

RQ1-RQ2 table-ready: adaptive vs static and composition.

RQ1-RQ2 table-ready: clusters and transfer matrix.

Feature extraction starts; labels in progress.

Day 4

P05 experiments

RQ3 feedback pilot if stable.

RQ3 replay pilot if data/model access ready.

RQ1-RQ2 table-ready: drift detection and localization.

Day 5

Ablations and baselines

Feedback-channel ablation.

Support threshold/clustering robustness.

Scanner baseline and early-warning pilot.

Day 6

Failure cases and robustness

Representative failure cases and benign utility.

Replay-validation cases and confidence intervals.

Evasive update and temporal holdout.

Day 7

M11/M12 freeze

Artifact + draft sections: threat model, method, results.

Artifact + draft sections: data, method, transfer results.

Artifact + draft sections: data, diff features, results.



7. Concrete Commands to Implement First

# 1. Create the repo skeleton and paper folders

python scripts/init_repo.py --papers P15 P08 P05 --modules M0 M2 M4 M11 M12



# 2. Validate configs

python scripts/validate_configs.py --papers P15 P08 P05



# 3. Run smoke matrix after M4 exists

python -m agentseclab_x.core.matrix_runner --config configs/experiments/smoke_p15_p08_p05.yaml



# 4. Generate RQ dashboard after M12 exists

python -m agentseclab_x.reporting.rq_dashboard --papers P15 P08 P05 --runs runs/



# 5. Generate paper-specific tables

python -m agentseclab_x.reporting.make_tables --paper P15 --runs runs/P15

python -m agentseclab_x.reporting.make_tables --paper P08 --runs runs/P08

python -m agentseclab_x.reporting.make_tables --paper P05 --runs runs/P05



# 6. Package artifacts

python -m agentseclab_x.artifacts.make_artifact --paper P15 --runs runs/P15 --out artifacts/P15.zip

python -m agentseclab_x.artifacts.make_artifact --paper P08 --runs runs/P08 --out artifacts/P08.zip

python -m agentseclab_x.artifacts.make_artifact --paper P05 --runs runs/P05 --out artifacts/P05.zip

8. Appendix: Minimal Schemas

8.1 Experiment config skeleton

paper_id: P15

rq_id: RQ1

experiment_id: p15_rq1_adaptive_vs_static_v0

seed: 1

dataset:

  id: harmbench_subset_v0

  path: data/p15/behaviors_subset.csv

model:

  target: local_or_api_model_name

attacks:

  static: [template_baseline]

  adaptive: [pair_or_gptfuzz]

defenses:

  pipelines: [none, pre_guard, post_guard, pre_post]

budget:

  max_queries: 20

  max_tokens: 12000

outputs:

  run_dir: runs/P15/RQ1/

8.2 Minimal M2 event

{

  "event_id": "evt_000001",

  "run_id": "P15_RQ1_...",

  "paper_id": "P15",

  "rq_id": "RQ1",

  "scenario_id": "behavior_0001",

  "step_id": 3,

  "event_type": "GuardDecisionEvent",

  "timestamp": "2026-08-11T12:00:00-06:00",

  "actor": "guard",

  "source_channel": "model_output",

  "trust_level": 0.5,

  "content_hash": "sha256:...",

  "parent_event_ids": ["evt_000000"],

  "outcome": "blocked_or_allowed",

  "payload": {

    "guard_name": "wildguard_or_llamaguard",

    "label": "safe_or_unsafe",

    "score": 0.84,

    "defense_pipeline": "pre_post"

  }

}

8.3 Universal result row

run_id,paper_id,rq_id,scenario_id,dataset_id,model,attack,defense,seed,primary_metric,TSR,ASR,utility,cost,latency,ci_low,ci_high,status

P15_...,P15,RQ1,behavior_0001,harmbench_subset_v0,target_model,pair,pre_post,1,adaptive_gain,NA,0.42,0.95,1.23,18.4,0.31,0.52,TABLE_READY

9. Final Go/No-Go Criteria

Paper

Green by Day 3/4

Yellow fallback

Red condition

P15

RQ1 and RQ2 tables have real adaptive/static and composition values.

Use fewer attacks/guards but keep matched budgets.

No adaptive backend runs at all.

P08

Cluster table and transfer matrix generated from real or approved internal sample.

Use smaller sample and label as pilot; skip replay.

No data access and no internal sample.

P05

50+ high-quality labeled update pairs and drift/localization tables.

Use 20-50 hand-labeled cases as measurement-protocol pilot.

No release pairs or no labels.



Implementation principle: keep all raw harmful text, canary values, and potentially sensitive repository content out of public artifacts by default. The publishable artifact should contain manifests, hashes, sanitized summaries, code, configs, representative safe traces, and aggregate metrics.Focused Module-First Development Specification

P15 GuardBreaker · P08 CrossJailbreak-1M · P05 MCPDrift

Detailed implementation plan for M0, M2, M4, M11, and M12

Prepared for Md Jahangir Alam | Scope: build the first reusable substrate and finish RQ1-RQ2 evidence for P15, P08, and P05 in seven days. This is a separate write-up expanding Section 16.3 of the module-first Top-15 plan.

1. Operating Goal and Scope

For now, develop only P15, P08, and P05 using the substrate modules M0, M2, M4, M11, and M12. The plan treats these five modules as the paper factory: experiment identity, event logging, episode execution, artifact packaging, and reporting. Paper-specific analysis code is allowed only as thin plugins under papers/P15, papers/P08, and papers/P05; once stable, those plugins can later be promoted into M6, M8, M9, or M10.

Paper

Core claim

Mandatory 7-day RQs

Finish line with only M0/M2/M4/M11/M12

P15 GuardBreaker

Composed jailbreak defenses can look robust in static matrices but fail when attacks adapt to the deployed pipeline.

RQ1 adaptive-vs-static ASR; RQ2 defense-composition monotonicity; RQ3 feedback-signal leverage if budget allows.

Day 3: attack/eval configs, adaptive-vs-static table, composition gain/loss table, artifact README.

P08 CrossJailbreak-1M

Organic jailbreak attempts form transfer clusters and model vulnerability partial orders different from synthetic benchmark rankings.

RQ1 strategy clusters; RQ2 asymmetric cross-model transfer; RQ3 replay validation if data/model access is ready.

Day 3: cluster assignments, transfer matrix, safety-clique graph/table, artifact README.

P05 MCPDrift

Semantic, schema, permission, and destination drift predicts security-relevant MCP updates better than hash/version changes.

RQ1 drift detection; RQ2 changed-component localization; RQ3 early-warning/scanner comparison if labels are ready.

Day 4: 50-150 labeled release/update pairs, drift feature table, localization table, artifact README.





2. Critical Path: Build These Modules First

Order

Module

Concrete output

Which paper finishes because of it

Day 1 AM

M0 Reproducibility Core

paper manifests, IDs, config schemas, registries, cache, ledger, run folders

All three papers can register experiments and create reproducible run folders.

Day 1 PM

M2 Event and Provenance Schema

validated JSONL event schema with P15/P08/P05-specific event types

P15/P08/P05 can log comparable evidence without paper-specific logging.

Day 2 AM

M4 Benchmark Harness

task interface, adapter interface, evaluator interface, matrix runner, dummy task pack

P15/P08 can run RQ1-RQ2 pilots; P05 can run static diff tasks as M4 task packs.

Day 2 PM

M11 Paper/Artifact Factory

paper folders, data cards, README templates, artifact bundles

All three have submission-style artifact shells from first result.

Day 2 PM-3 AM

M12 Reporting Layer

result aggregator, RQ dashboard, paper-specific table templates, failure-case exporter

P15/P08 get Day-3 tables; P05 gets Day-4 tables after labels land.





2.1 Minimal Repository Tree

agentseclab_x/

  core/

    ids.py                 # M0: stable IDs and run hashes

    config.py              # M0: typed config loaders and validators

    registry.py            # M0: model/task/attack/defense/paper registries

    cache.py               # M0: response/tool-output cache

    events.py              # M2: event dataclasses/Pydantic models

    event_writer.py         # M2: JSONL append/validate/write helpers

    runner.py              # M4: run_episode()

    matrix_runner.py        # M4: run_matrix()

  benchmarks/

    base_task.py            # M4: TaskPack interface

    jailbreak/              # P15/P08 task packs

    mcpdrift/               # P05 release/update task pack

  evaluators/

    base.py                 # M4: evaluator interface

  reporting/

    aggregate.py            # M12: event -> result rows

    rq_dashboard.py         # M12: RQ status tracker

    make_tables.py          # M12: paper tables

    make_figures.py         # M12: paper figures

    export_cases.py         # M12: representative cases

  artifacts/

    make_artifact.py        # M11: bundle builder

    templates/              # M11: README, data card, Open Science appendix

  papers/

    P15_GuardBreaker/

    P08_CrossJailbreak1M/

    P05_MCPDrift/

  configs/

    models.yaml

    papers.yaml

    datasets.yaml

    attacks.yaml

    defenses.yaml

    experiments/

  schemas/

    event_schema.json

    result_schema.json

  tests/

3. Existing Codebases to Reuse: Why and How

Codebase / source

Use for

Why this codebase

Concrete reuse instruction

JailbreakBench / jailbreakbench

P15 and P08; M0/M4/M11/M12

Standardized jailbreak behaviors, artifacts, scoring conventions, and reproducibility patterns.

Mirror behavior IDs and artifact folders; use its behavior/scoring separation as your configs/datasets.yaml and reporting templates.

HarmBench

P15 and P08; evaluator utilities

HarmBench provides harmful-behavior taxonomy and classifier/evaluation culture for automated red-teaming.

Use behaviors as seed categories; wrap classifier/judge output into M2 JudgeScoreEvent; reuse behavior-level tables.

StrongREJECT

P08 and P15 output scoring

Richer refusal/compliance scoring than binary keyword refusal; useful for organic compliance labels.

Call evaluator in a paper-specific scoring plugin; store score, refusal flag, and explanation hash as M2 JudgeScoreEvent.

EasyJailbreak package / framework

P15 attack integration

Selector/Mutator/Constraint/Evaluator abstraction maps cleanly to adaptive attack loops.

Implement an AttackAdapter wrapper that exposes generate(), adapt(), reset(), cost(); do not modify upstream package.

PAIR / JailbreakingLLMs

P15 adaptive black-box baseline

PAIR is a known iterative black-box attack engine and is already listed in the source plans.

Wrap PAIR as attack_backend=pair in configs/attacks.yaml; log every candidate as AttackCandidateEvent.

GPTFuzz

P15 mutation search baseline

Mutation/fuzzing operators support adaptive objective search and feedback-channel ablations.

Expose mutators as P15 plugins; each mutation receives previous prompt, guard result, and target result.

AutoDAN

P15 stealth-oriented optimization baseline

Optimization baseline for stealthy jailbreak prompts; useful as a stronger but slower stretch backend.

Keep optional; run on local/open model only if Day-3 P15 table is already complete.

llm-attacks / GCG

P15 white-box suffix baseline

White-box baseline for local models; useful for contrast with black-box adaptation.

Use only when local model gradients are available; otherwise mark as deferred.

LMSYS-Chat-1M dataset

P08 organic data source

Provides real-world conversations, model names, language tags, and moderation flags.

Load streaming Parquet; create P08 jailbreak_corpus.parquet with conv_id, model, language, prompt, response, moderation flag, compliance label.

modelcontextprotocol/servers

P05 benign corpus and schema examples

Reference/community MCP server corpus with manifests and schemas.

Clone selected repos with commit hashes; extract tool names, parameter schemas, manifests, release tags.

modelcontextprotocol/python-sdk

P05 replay-compatible schema understanding

Official SDK for local MCP server/client behavior when you later add replay; not required for static RQ1-RQ2.

Use schema examples and optional local replay; record server metadata as ReleaseSnapshotEvent.

Snyk agent-scan

P05 scanner baseline

Security scanner for AI agents, MCP servers, and skills.

Run scanner on selected snapshots; record findings as ScannerFindingEvent; use as baseline against drift score.

GitPython / git CLI

P05 release history collection

Fast, deterministic collection of commits, tags, diffs, and changed files.

Use git clone --bare or GitPython to get release pairs; write snapshot manifests and diff records.

Gitleaks / TruffleHog / Semgrep

P05 optional static baselines

Secret/static scans provide simple baseline signals for risky updates.

Run only on controlled public/local snapshots; save JSON output and normalize into M2 ScannerFindingEvent.





4. M0 — Reproducibility Core

Priority: Day 1 morning

Purpose: Every run, paper, RQ, dataset, attack, defense, model, seed, and artifact has an immutable identity. M0 prevents the three papers from becoming three unrelated codebases.

4.x.1 Codebases to reuse and why

Codebase/source

Part to reuse

How to use

JailbreakBench

Behavior/artifact and scoring conventions

Use for paper manifests, behavior IDs, and reproducibility culture for P15/P08.

HarmBench

Behavior taxonomy and evaluator style

Use for P15/P08 behavior categories and metric naming.

PurpleLlama CybersecurityBenchmarks

Runner/model adapter/result organization

Use organization patterns for configs/results even though these three papers are not cyber-range heavy.





4.x.2 Files to create with input/output contracts

File

What goes inside / input

Output / consumed by

agentseclab_x/core/ids.py

Inputs: config dict, git commit, seed, timestamp policy. Output: stable run_id, scenario_id, config_hash, artifact_id. Contains make_run_id(), make_config_hash(), short_id().

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/config.py

Inputs: YAML files from configs/. Output: validated ExperimentConfig, PaperManifest, DatasetManifest, ModelConfig, AttackConfig, DefenseConfig objects. Contains schema validation and defaults.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/registry.py

Inputs: configs/models.yaml, datasets.yaml, attacks.yaml, defenses.yaml, papers.yaml. Output: registry lookup objects used by M4. Contains get_model(), get_taskpack(), get_attack(), get_defense(), get_paper().

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/cache.py

Inputs: request hash, provider/model, prompt/tool args. Output: cached response JSON or cache miss. Contains read_cache(), write_cache(), cache_key().

Consumed by M4/M11/M12 and paper-specific plugins.

configs/papers.yaml

Inputs: manually written paper entries. Output: paper metadata for P15/P08/P05 including RQs, primary metric, module list, output folder, owner.

Consumed by M4/M11/M12 and paper-specific plugins.

configs/experiments/*.yaml

Inputs: per-RQ experiment definitions. Output: executable experiment matrix for M4.

Consumed by M4/M11/M12 and paper-specific plugins.

artifacts/experiment_ledger.csv

Inputs: append-only run metadata. Output: daily ledger consumed by M12 and M11.

Consumed by M4/M11/M12 and paper-specific plugins.

scripts/freeze_manifest.py

Inputs: experiment folder. Output: manifest.json containing configs, code commit, dependency lock hash, dataset version, run IDs.

Consumed by M4/M11/M12 and paper-specific plugins.





4.x.3 Detailed development steps

Initialize repository and configs/ directory; create P15, P08, P05 paper manifests first.

Define ID hierarchy: paper_id -> rq_id -> dataset_id -> scenario_id -> run_id. Use stable hashes rather than manual names.

Create default experiment templates: p15_rq1_adaptive_vs_static.yaml, p15_rq2_composition.yaml, p08_rq1_clusters.yaml, p08_rq2_transfer.yaml, p05_rq1_drift_detection.yaml, p05_rq2_localization.yaml.

Add a config validator that fails fast when a required field is absent.

Add run-folder constructor that creates runs/{paper_id}/{rq_id}/{run_id}/ with config.yaml, manifest.json, events.jsonl, metrics.csv, logs/, tables/, figures/.

Add cache and cost ledger hooks even if you start with dummy/local outputs; do not retrofit logging after experiments.

4.x.4 Acceptance test and paper unblocking effect

Acceptance test: `python scripts/init_experiment.py --paper P15 --rq RQ1 --config configs/experiments/p15_rq1_adaptive_vs_static.yaml` creates a run folder with config.yaml, manifest.json, events.jsonl, metrics.csv, and README_stub.md. Repeat for P08 RQ1 and P05 RQ1.

Paper

Unlocked after M0

P15

Experiment configs for adaptive/static runs and defense-composition matrix.

P08

Corpus mining and cluster/transfer run manifests.

P05

Release-pair manifest and label ledger.





4. M2 — Unified Event and Provenance Schema

Priority: Day 1 afternoon

Purpose: All evidence becomes one validated JSONL format. M2 is what allows M12 to generate tables without paper-specific parsers.

4.x.1 Codebases to reuse and why

Codebase/source

Part to reuse

How to use

AgentDojo

Task/injection/defense/security result separation

Use the separation principle, then generalize into a unified event log.

CyberSecEval-MCP plan

Tool, argument, observation, trust label, policy boundary fields

Use field vocabulary for P05/P15 and future compatibility.

LMSYS-Chat-1M metadata

conversation_id, model, language, moderation fields

Normalize P08 records into M2 DatasetRecord and ModelResponseEvent.





4.x.2 Files to create with input/output contracts

File

What goes inside / input

Output / consumed by

schemas/event_schema.json

Inputs: event JSON objects. Output: validation pass/fail. Defines required fields and paper-specific extensions.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/events.py

Inputs: Python event constructor calls. Output: typed event dictionaries. Classes: RunStart, DatasetRecord, AttackCandidateEvent, ModelResponseEvent, GuardDecisionEvent, JudgeScoreEvent, ClusterEvent, TransferEdgeEvent, ReleaseSnapshotEvent, DiffFeatureEvent, ScannerFindingEvent, FinalMetricEvent.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/provenance.py

Inputs: parent_event_ids and source metadata. Output: provenance chains and content hashes. Contains hash_content(), make_parent_chain(), trust_summary().

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/event_writer.py

Inputs: Event objects and run directory. Output: append-only events.jsonl with schema validation. Contains write_event(), load_events(), validate_event_file().

Consumed by M4/M11/M12 and paper-specific plugins.

tests/test_event_schema.py

Inputs: hand-authored traces. Output: pytest pass/fail for clean run, successful attack, blocked attack, failed scanner, transfer edge, release drift.

Consumed by M4/M11/M12 and paper-specific plugins.

docs/EVENT_SCHEMA.md

Inputs: schema. Output: human-readable documentation and examples for students.

Consumed by M4/M11/M12 and paper-specific plugins.





4.x.3 Detailed development steps

Create a base event with universal fields: event_id, run_id, paper_id, rq_id, scenario_id, step_id, event_type, timestamp, actor, source_channel, trust_level, content_hash, parent_event_ids, outcome.

Add P15 fields: attack_name, attack_family, defense_pipeline, feedback_signal, candidate_text_hash, guard_labels, judge_scores, query_count, adaptive_round.

Add P08 fields: conv_id, source_model, target_model, cluster_id, compliance_label, language, harm_category, transfer_source, transfer_target, support_n, ci_low, ci_high.

Add P05 fields: repo_id, server_name, version_from, version_to, changed_component, diff_type, permission_delta, schema_delta, destination_delta, scanner_name, scanner_finding, gold_label.

Ensure every event can be anonymized: text fields stored as hash by default; safe text snippets go in cases/ only after manual review.

Write hand-authored example traces before writing adapters; this makes M12 implementation deterministic.

4.x.4 Acceptance test and paper unblocking effect

Acceptance test: `pytest tests/test_event_schema.py` validates at least six traces: P15 adaptive candidate, P15 defense-composition run, P08 cluster event, P08 transfer edge, P05 diff feature, P05 scanner finding. M12 must compute at least one dummy metric from each.

Paper

M2 event types needed

P15

AttackCandidateEvent, GuardDecisionEvent, JudgeScoreEvent, FeedbackSignalEvent, FinalMetricEvent.

P08

DatasetRecord, ConversationEvent, ComplianceLabelEvent, ClusterEvent, TransferEdgeEvent, ReplayEvent.

P05

ReleaseSnapshotEvent, DiffFeatureEvent, DriftLabelEvent, ScannerFindingEvent, FinalMetricEvent.





4. M4 — Benchmark Harness

Priority: Day 2 morning

Purpose: M4 executes experiments defined by M0 and logs them through M2. For these three papers, M4 is mostly offline/semioffline: jailbreak runs, organic-data analysis runs, and release-diff tasks.

4.x.1 Codebases to reuse and why

Codebase/source

Part to reuse

How to use

JailbreakBench

behavior/scoring/run separation

Use for P15/P08 taskpack structure.

HarmBench

red-team evaluation utilities

Use as evaluator baseline where applicable.

modelcontextprotocol/servers + git

release/schema corpus

Use for P05 release drift taskpack.





4.x.2 Files to create with input/output contracts

File

What goes inside / input

Output / consumed by

agentseclab_x/benchmarks/base_task.py

Defines TaskPack interface. Input: config and data path. Output: iterable scenarios with success_oracle and violation_oracle.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/evaluators/base.py

Defines Evaluator interface. Input: M2 event file or model outputs. Output: metric events and result rows.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/runner.py

Input: one ExperimentConfig. Output: one run folder with events.jsonl and metrics.csv. Contains run_episode() and run_offline_analysis().

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/core/matrix_runner.py

Input: experiment matrix YAML. Output: multiple run folders and aggregate run index. Supports checkpoint/resume.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/benchmarks/jailbreak/guardbreaker_task.py

P15. Input: behaviors, attack backends, defense pipelines. Output: attack attempts and guard/judge events.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/benchmarks/jailbreak/organic_transfer_task.py

P08. Input: jailbreak_corpus.parquet or sample CSV. Output: cluster events, transfer edges, replay events.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/benchmarks/mcpdrift/release_drift_task.py

P05. Input: snapshot manifest pairs. Output: release snapshot events, diff feature events, scanner finding events, drift labels.

Consumed by M4/M11/M12 and paper-specific plugins.

configs/experiments/*.yaml

Input to matrix_runner. Each config has paper_id, rq_id, dataset, taskpack, model/evaluator, baseline, seeds, output_path.

Consumed by M4/M11/M12 and paper-specific plugins.





4.x.3 Detailed development steps

Implement BaseTaskPack and BaseEvaluator before paper-specific task packs.

Create a dummy model/guard/evaluator so the harness can be tested without API keys.

Implement P15 GuardBreakerTaskPack: read behaviors, generate static attacks, run one adaptive backend, log guard labels, target responses, judge scores, query counts.

Implement P08 OrganicTransferTaskPack: load corpus, filter compliant/refusal labels, embed/cluster or load precomputed clusters, compute directed transfer matrix and support thresholds.

Implement P05 ReleaseDriftTaskPack: read repo/version pairs, compute basic diffs, run scanner JSON if available, emit drift features and gold labels.

Add resume logic: if a run folder already has finished.ok, skip unless --force is set.

Add exception handling as M2 ErrorEvent; failed runs are evidence, not silent missing cells.

4.x.4 Acceptance test and paper unblocking effect

Acceptance test: `python -m agentseclab_x.core.matrix_runner --config configs/experiments/smoke_p15_p08_p05.yaml` runs one tiny scenario per paper and writes events.jsonl, metrics.csv, and run_index.csv.

Paper

First runnable taskpack

P15

GuardBreakerTaskPack: behaviors x attack backend x defense pipeline x seed.

P08

OrganicTransferTaskPack: corpus -> clusters -> transfer edges -> matrices.

P05

ReleaseDriftTaskPack: release pairs -> diff features -> drift labels -> scanner baselines.





4. M11 — Paper and Artifact Factory

Priority: Day 2 afternoon

Purpose: M11 ensures each paper has a reproducible artifact package from the first successful run. It prevents final-week scrambling.

4.x.1 Codebases to reuse and why

Codebase/source

Part to reuse

How to use

JailbreakBench

artifact release conventions

Use README/data structure patterns for P15/P08.

HarmBench

behavior documentation style

Use behavior/category documentation patterns.

USENIX/NDSS artifact expectations

README, smoke test, Open Science appendix

Generate artifact shell from day one.





4.x.2 Files to create with input/output contracts

File

What goes inside / input

Output / consumed by

agentseclab_x/artifacts/make_artifact.py

Inputs: paper_id and run IDs. Output: artifact zip/folder with README, configs, sample traces, tables, figures, data card, smoke command.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/artifacts/templates/README.md

Template. Contains install, smoke run, expected outputs, safety constraints, reproduction commands.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/artifacts/templates/DATA_CARD.md

Template. Contains dataset source, inclusion criteria, labels, splits, safety filtering, limitations.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/artifacts/templates/OPEN_SCIENCE.md

Template. Contains artifact availability, reproducibility, compute cost, excluded materials, ethics/safety.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/papers/P15_GuardBreaker/

Contains paper_manifest.yaml, RQ_STATUS.md, threat_model.md, tables/, figures/, cases/, appendix/, artifact_notes.md.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/papers/P08_CrossJailbreak1M/

Same structure plus data_access.md and privacy_filtering.md.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/papers/P05_MCPDrift/

Same structure plus release_label_policy.md and scanner_baselines.md.

Consumed by M4/M11/M12 and paper-specific plugins.

scripts/anonymize_artifact.py

Inputs: artifact folder. Output: anonymized artifact folder. Removes usernames, absolute paths, API logs, credentials, and unsafe payload text.

Consumed by M4/M11/M12 and paper-specific plugins.





4.x.3 Detailed development steps

Create paper folders on Day 1; do not wait for final results.

Generate README and data-card drafts from M0 manifests and M2 schema automatically.

Add a smoke config for each paper: p15_smoke.yaml, p08_smoke.yaml, p05_smoke.yaml. These should run in under five minutes on a laptop using dummy/local data.

Export only safe artifacts: configs, hashes, synthetic canaries, sampled sanitized traces, aggregate tables, and documentation. Do not export raw harmful prompts unless separately approved and sanitized.

Add artifact status to RQ_STATUS.md: smoke_passed, result_regenerated, table_regenerated, anonymized, ready_for_internal_review.

4.x.4 Acceptance test and paper unblocking effect

Acceptance test: `python -m agentseclab_x.artifacts.make_artifact --paper P15 --runs runs/P15/...` creates a zip containing README, configs, sample events, metrics, tables, figures, and data card. Repeat for P08 and P05.

Paper

Artifact folder must contain

P15

attack configs, defense-pipeline configs, sample sanitized events, adaptive/static table, composition table.

P08

data-card, privacy filtering note, cluster assignment sample, transfer matrix sample, replay config.

P05

release label policy, snapshot manifests, diff features, scanner outputs, drift tables.





4. M12 — Reporting Layer

Priority: Day 2 evening to Day 3 morning

Purpose: M12 turns M2 event logs into RQ dashboards, tables, figures, and failure cases. It includes a lightweight metric registry for the three selected papers.

4.x.1 Codebases to reuse and why

Codebase/source

Part to reuse

How to use

Jailbreak security plan

one-row-per-attempt reporting and heatmaps

Use for P15 adaptive attack tables and P08 transfer matrices.

HarmBench/JailbreakBench

behavior-level result tables

Use for P15/P08 reporting.

Pandas/Matplotlib/NetworkX

lightweight analysis and figure generation

Use to avoid manual spreadsheets and regenerate figures from logs.





4.x.2 Files to create with input/output contracts

File

What goes inside / input

Output / consumed by

agentseclab_x/reporting/result_schema.py

Defines universal result row. Input: M2 events. Output: normalized rows with paper_id, rq_id, metric values, confidence fields.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/aggregate.py

Inputs: run folders or events.jsonl. Output: results.parquet/csv and aggregate metrics.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/rq_dashboard.py

Inputs: results + paper manifests. Output: RQ_STATUS.md and dashboard.csv with NOT_STARTED/DATA_READY/RUNNING/RESULT_READY/TABLE_READY/DRAFT_READY.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/make_tables.py

Inputs: results.parquet and paper_id. Output: paper tables in CSV, Markdown, and LaTeX/DOCX-ready text.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/make_figures.py

Inputs: results.parquet. Output: PNG/PDF heatmaps, line plots, transfer graphs, drift timelines.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/export_cases.py

Inputs: events + metrics. Output: sanitized true positives, false negatives, false positives, representative case studies.

Consumed by M4/M11/M12 and paper-specific plugins.

agentseclab_x/reporting/templates/*.yaml

Per-paper table definitions for P15/P08/P05.

Consumed by M4/M11/M12 and paper-specific plugins.





4.x.3 Detailed development steps

Define universal result row: run_id, paper_id, rq_id, scenario_id, dataset_id, attack_id, model, defense, seed, primary_metric, TSR, ASR, utility, cost, latency, ci_low, ci_high, status.

Implement P15 metrics: static_ASR, adaptive_ASR, adaptive_gain, guard_evasion_rate, composition_gain_loss, feedback_ablation_effect, queries_to_bypass.

Implement P08 metrics: n_clusters, cluster_purity, transfer_T_ij, support_n, bootstrap_CI_width, clique_count, vulnerability_partial_order_edges, replay_success_rate.

Implement P05 metrics: drift_precision, drift_recall, feature_auc, changed_component_F1, first_risky_version_accuracy, scanner_agreement, early_warning_lead.

Generate paper-specific tables: P15 adaptive-vs-static table and composition matrix; P08 transfer heatmap and cluster table; P05 drift-feature ablation and localization table.

Export sanitized case studies: no raw harmful prompt by default; show hashes, categories, and safe summaries.

4.x.4 Acceptance test and paper unblocking effect

Acceptance test: `python -m agentseclab_x.reporting.rq_dashboard --papers P15 P08 P05 --runs runs/` writes dashboards and at least one table per paper using smoke results; with real results it marks RQ1-RQ2 as TABLE_READY.

Paper

Main M12 outputs

P15

Table 1 adaptive vs static ASR; Table 2 defense-composition gain/loss; Figure 1 feedback ablation.

P08

Figure transfer heatmap; Table strategy clusters; Graph safety cliques/partial order.

P05

Table drift-feature ablation; Table changed-component localization; Figure release timeline.





5. Paper-Specific Implementation Plans Using Only the Five Core Modules

The following plans intentionally use M0, M2, M4, M11, and M12 only. Where the full Top-15 plan would use M5/M6/M8/M9/M10, this document implements a minimal paper-local plugin and logs it through M2. After the first seven days, promote the stable plugin into the full shared module.

5.1 P15 GuardBreaker

RQ

Question

Core-module-only implementation

RQ1

How much does adaptive PAIR/TAP/GPTFuzz/AutoDAN/GCG-style search increase ASR over non-adaptive attacks against single defenses?

Use P15 AttackAdapter plugins under M4. Run static template baseline and one adaptive backend with matched query budgets. M12 computes adaptive_gain = adaptive_ASR - static_ASR.

RQ2

Does composing multiple guards monotonically improve security, or can composition create new bypass surfaces?

Define defense_pipeline in configs/defenses.yaml: none, pre_guard, post_guard, pre+post, pre+target-refusal+post. M4 runs the same attacks across pipelines; M12 computes composition_gain_loss.

RQ3

Which feedback signals provide the most leverage to an adaptive attacker?

Stretch with same modules: configure feedback_signal = accept/reject, score, label, latency, target_response. M12 outputs feedback ablation table.





P15 files to create

File

Input

Output / content

papers/P15_GuardBreaker/paper_manifest.yaml

Manual paper metadata

RQs, primary metrics, module list, target venues, owner, dataset/behavior list.

configs/experiments/p15_rq1_adaptive_vs_static.yaml

behavior set, attack backends, defenses, target models, budget

M4 matrix over static vs adaptive runs.

configs/experiments/p15_rq2_composition.yaml

same behavior set, defense pipeline list

M4 matrix over defense compositions.

agentseclab_x/benchmarks/jailbreak/guardbreaker_task.py

M0 config + behavior records + attack adapter

M2 events: AttackCandidateEvent, GuardDecisionEvent, JudgeScoreEvent.

papers/P15_GuardBreaker/plugins/attack_adapters.py

EasyJailbreak/PAIR/GPTFuzz/AutoDAN wrappers

common generate/adapt/reset/cost interface.

papers/P15_GuardBreaker/plugins/guard_pipelines.py

defense names and order

serial pipeline object that emits guard decisions.

papers/P15_GuardBreaker/analysis/p15_metrics.yaml

M12 metric definitions

adaptive_gain, guard_evasion_rate, queries_to_bypass, composition_gain_loss.





P15 7-day steps

Day 1: Create paper manifest, behavior subset, target model list, and defense pipeline list. Use 50-100 safe benchmark behaviors or sanitized IDs; avoid raw dangerous text in release artifacts.

Day 2: Implement GuardBreakerTaskPack and dummy attack backend; confirm M2 logs attack candidates, guard labels, judge scores, and costs.

Day 3: Integrate one real adaptive backend first: PAIR or GPTFuzz. Run RQ1 static vs adaptive on 3 attacks x 3 guards x 2 target models x small budget. Generate P15 Table 1.

Day 3 evening: Run RQ2 composition matrix: no defense, pre-guard, post-guard, pre+post. Generate composition gain/loss table. This is the paper finish line for RQ1-RQ2.

Day 4-5: Add feedback-channel ablation for RQ3 if RQ1/RQ2 are stable: accept/reject only, guard label, guard score, target response.

Day 6: Add sanity checks: same query budget, same seed set, same behavior set, benign utility controls, failure cases.

Day 7: Freeze artifact, create sanitized representative cases, write RQ_STATUS.md, update threat_model.md and limitations.

5.2 P08 CrossJailbreak-1M

RQ

Question

Core-module-only implementation

RQ1

What jailbreak strategy clusters emerge from large-scale organic conversations?

M4 runs an offline corpus pipeline. It loads conversations, filters candidate jailbreak/unsafe records, embeds or uses precomputed embeddings, clusters, emits ClusterEvent rows.

RQ2

Do successful jailbreaks transfer asymmetrically across models, forming safety cliques or vulnerability partial orders?

M4 computes cluster-level directed transfer T[i,j]; M12 produces heatmap, support table, clique table, and partial-order edge table.

RQ3

Can top organic clusters be replay-validated on current models under controlled budgets?

Stretch with same M4 harness: use top 10 clusters and evaluate current target models with sanitized prompts or internal approval-only samples.





P08 files to create

File

Input

Output / content

papers/P08_CrossJailbreak1M/paper_manifest.yaml

paper metadata

RQs, data access assumptions, privacy filtering, artifact policy.

configs/experiments/p08_rq1_clusters.yaml

corpus path, embedding model, clustering params

M4 clustering pipeline config.

configs/experiments/p08_rq2_transfer.yaml

cluster file, compliance labels, model list

M4 transfer matrix pipeline config.

agentseclab_x/benchmarks/jailbreak/organic_transfer_task.py

jailbreak_corpus.parquet or sample CSV

ClusterEvents, TransferEdgeEvents, matrix outputs.

papers/P08_CrossJailbreak1M/plugins/lmsys_loader.py

LMSYS Parquet or local sample

normalized records: conv_id, model, language, prompt_hash, response_hash, moderation flag.

papers/P08_CrossJailbreak1M/plugins/compliance_labeler.py

responses + scorer

compliance_label, confidence, evaluator version.

papers/P08_CrossJailbreak1M/plugins/cluster_transfer.py

embeddings/clusters/compliance labels

cluster assignment table, T matrix, support table, bootstrap CIs.

papers/P08_CrossJailbreak1M/privacy_filtering.md

manual policy

how raw prompts/responses are filtered, hashed, summarized, and excluded from public artifacts.





P08 7-day steps

Day 1: Confirm data path: full LMSYS-Chat-1M, local preprocessed sample, or internal sanitized subset. Create data_access.md documenting what is available.

Day 2: Build lmsys_loader.py and produce a small normalized corpus with conv_id, model, language, harm/moderation flag, prompt_hash, response_hash, and safe summary.

Day 3: Run RQ1 clustering on 10k-50k candidates or the maximum available local sample. Output cluster_assignments.parquet, cluster_summary.csv, and M2 ClusterEvents.

Day 3 evening: Run RQ2 transfer on cluster-level compliance labels. Output transfer_matrix.csv, support_matrix.csv, and bootstrap_CI.csv. This is the RQ1-RQ2 finish line.

Day 4-5: Run replay validation for RQ3 on top 10 clusters if you have model access and approval to use sanitized cluster representatives.

Day 6: Add robustness checks: minimum support thresholds, alternate clustering method, high-confidence-only labels.

Day 7: Package sanitized atlas, matrix, cluster summaries, and artifact README; exclude raw harmful text unless separately approved.

5.3 P05 MCPDrift

RQ

Question

Core-module-only implementation

RQ1

Can semantic, permission, schema, and destination drift predict security-relevant MCP updates better than hash/version changes?

Use a paper-local P05 diff plugin under M4 instead of full M6. It loads snapshot pairs, computes feature rows, compares against labels, and M12 reports precision/recall/AUC.

RQ2

Which drift features best localize the changed component that introduced risk?

Each DiffFeatureEvent includes changed_component. M12 computes localization accuracy/F1 by feature group.

RQ3

Can drift scoring provide early warning before runtime attack success or incident-like behavior?

Stretch: add time/release index and scanner finding time. M12 computes lead time against labeled first-risky version.





P05 files to create

File

Input

Output / content

papers/P05_MCPDrift/paper_manifest.yaml

paper metadata

RQs, repo list, label policy, drift features, baselines.

configs/experiments/p05_rq1_drift_detection.yaml

repo list, version pairs, feature flags

M4 release-drift task config.

configs/experiments/p05_rq2_localization.yaml

diff feature rows, labels

changed-component localization experiment.

agentseclab_x/benchmarks/mcpdrift/release_drift_task.py

snapshot manifest pairs

M2 ReleaseSnapshotEvents, DiffFeatureEvents, DriftLabelEvents.

papers/P05_MCPDrift/plugins/repo_collector.py

GitHub URLs or local repos

snapshot_manifest.jsonl with commit/tag/version IDs.

papers/P05_MCPDrift/plugins/diff_features.py

two snapshots

hash_delta, semantic_delta, schema_delta, permission_delta, endpoint_delta, dependency_delta.

papers/P05_MCPDrift/plugins/scanner_runner.py

snapshot path + scanner config

scanner findings normalized into M2 ScannerFindingEvent.

papers/P05_MCPDrift/release_label_policy.md

manual label guidelines

definition of security_relevant, changed_component, first_risky_version, evidence notes.





P05 7-day steps

Day 1: Select 30-50 MCP servers/repos first. Prioritize repos with visible tags/releases/commit history and readable schemas/manifests.

Day 2: Implement repo_collector.py and create snapshot manifests. If releases are sparse, use commit windows or curated v0/v1 pairs; record this in limitations.

Day 3: Implement diff_features.py with four feature groups: semantic description change, schema broadening, permission/scope change, new external destination. Output diff_features.parquet and M2 DiffFeatureEvents.

Day 3-4: Label 50-150 release/update pairs using release_label_policy.md. Minimum viable P05 needs 50 high-quality labels; strong result needs 150+.

Day 4: Run RQ1 drift detection and RQ2 localization. Generate feature ablation and localization tables. This is the RQ1-RQ2 finish line.

Day 5: Add scanner baseline with Snyk agent-scan and optional Gitleaks/TruffleHog/Semgrep. Compute scanner agreement and early-warning pilot.

Day 6: Add robustness check: benign-looking description/schema evasions and temporal holdout.

Day 7: Package snapshot manifests, label policy, feature tables, scanner outputs, and sanitized examples.

6. Seven-Day Execution Schedule and Finish-Line Mapping

Day

Module focus

P15 finish line

P08 finish line

P05 finish line

Day 1 AM

M0 configs, run IDs, paper manifests

P15 config and run folders initialized.

P08 data access and corpus manifest initialized.

P05 repo list and label ledger initialized.

Day 1 PM

M2 event schema and validators

P15 attack/guard/judge events validate.

P08 dataset/cluster/transfer events validate.

P05 snapshot/diff/scanner events validate.

Day 2 AM

M4 runner and taskpack API

P15 smoke run with dummy attack/guard.

P08 smoke run on small sample.

P05 smoke run on one release pair.

Day 2 PM

M11 artifact factory + M12 reporting skeleton

P15 RQ dashboard exists.

P08 RQ dashboard exists.

P05 RQ dashboard exists.

Day 3

P15/P08 experiments

RQ1-RQ2 table-ready: adaptive vs static and composition.

RQ1-RQ2 table-ready: clusters and transfer matrix.

Feature extraction starts; labels in progress.

Day 4

P05 experiments

RQ3 feedback pilot if stable.

RQ3 replay pilot if data/model access ready.

RQ1-RQ2 table-ready: drift detection and localization.

Day 5

Ablations and baselines

Feedback-channel ablation.

Support threshold/clustering robustness.

Scanner baseline and early-warning pilot.

Day 6

Failure cases and robustness

Representative failure cases and benign utility.

Replay-validation cases and confidence intervals.

Evasive update and temporal holdout.

Day 7

M11/M12 freeze

Artifact + draft sections: threat model, method, results.

Artifact + draft sections: data, method, transfer results.

Artifact + draft sections: data, diff features, results.





7. Concrete Commands to Implement First

# 1. Create the repo skeleton and paper folders

python scripts/init_repo.py --papers P15 P08 P05 --modules M0 M2 M4 M11 M12



# 2. Validate configs

python scripts/validate_configs.py --papers P15 P08 P05



# 3. Run smoke matrix after M4 exists

python -m agentseclab_x.core.matrix_runner --config configs/experiments/smoke_p15_p08_p05.yaml



# 4. Generate RQ dashboard after M12 exists

python -m agentseclab_x.reporting.rq_dashboard --papers P15 P08 P05 --runs runs/



# 5. Generate paper-specific tables

python -m agentseclab_x.reporting.make_tables --paper P15 --runs runs/P15

python -m agentseclab_x.reporting.make_tables --paper P08 --runs runs/P08

python -m agentseclab_x.reporting.make_tables --paper P05 --runs runs/P05



# 6. Package artifacts

python -m agentseclab_x.artifacts.make_artifact --paper P15 --runs runs/P15 --out artifacts/P15.zip

python -m agentseclab_x.artifacts.make_artifact --paper P08 --runs runs/P08 --out artifacts/P08.zip

python -m agentseclab_x.artifacts.make_artifact --paper P05 --runs runs/P05 --out artifacts/P05.zip

8. Appendix: Minimal Schemas

8.1 Experiment config skeleton

paper_id: P15

rq_id: RQ1

experiment_id: p15_rq1_adaptive_vs_static_v0

seed: 1

dataset:

  id: harmbench_subset_v0

  path: data/p15/behaviors_subset.csv

model:

  target: local_or_api_model_name

attacks:

  static: [template_baseline]

  adaptive: [pair_or_gptfuzz]

defenses:

  pipelines: [none, pre_guard, post_guard, pre_post]

budget:

  max_queries: 20

  max_tokens: 12000

outputs:

  run_dir: runs/P15/RQ1/

8.2 Minimal M2 event

{

  "event_id": "evt_000001",

  "run_id": "P15_RQ1_...",

  "paper_id": "P15",

  "rq_id": "RQ1",

  "scenario_id": "behavior_0001",

  "step_id": 3,

  "event_type": "GuardDecisionEvent",

  "timestamp": "2026-08-11T12:00:00-06:00",

  "actor": "guard",

  "source_channel": "model_output",

  "trust_level": 0.5,

  "content_hash": "sha256:...",

  "parent_event_ids": ["evt_000000"],

  "outcome": "blocked_or_allowed",

  "payload": {

    "guard_name": "wildguard_or_llamaguard",

    "label": "safe_or_unsafe",

    "score": 0.84,

    "defense_pipeline": "pre_post"

  }

}

8.3 Universal result row

run_id,paper_id,rq_id,scenario_id,dataset_id,model,attack,defense,seed,primary_metric,TSR,ASR,utility,cost,latency,ci_low,ci_high,status

P15_...,P15,RQ1,behavior_0001,harmbench_subset_v0,target_model,pair,pre_post,1,adaptive_gain,NA,0.42,0.95,1.23,18.4,0.31,0.52,TABLE_READY

9. Final Go/No-Go Criteria

Paper

Green by Day 3/4

Yellow fallback

Red condition

P15

RQ1 and RQ2 tables have real adaptive/static and composition values.

Use fewer attacks/guards but keep matched budgets.

No adaptive backend runs at all.

P08

Cluster table and transfer matrix generated from real or approved internal sample.

Use smaller sample and label as pilot; skip replay.

No data access and no internal sample.

P05

50+ high-quality labeled update pairs and drift/localization tables.

Use 20-50 hand-labeled cases as measurement-protocol pilot.

No release pairs or no labels.





Implementation principle: keep all raw harmful text, canary values, and potentially sensitive repository content out of public artifacts by default. The publishable artifact should contain manifests, hashes, sanitized summaries, code, configs, representative safe traces, and aggregate metrics.



this is the development plann for the paper and modules for the papers. 



Task: is to update the readme file based on the attached text for the 3 papers and the modulesFocused Module-First Development Specification



P15 GuardBreaker · P08 CrossJailbreak-1M · P05 MCPDrift



Detailed implementation plan for M0, M2, M4, M11, and M12



Prepared for Md Jahangir Alam | Scope: build the first reusable substrate and finish RQ1-RQ2 evidence for P15, P08, and P05 in seven days. This is a separate write-up expanding Section 16.3 of the module-first Top-15 plan.



1. Operating Goal and Scope



For now, develop only P15, P08, and P05 using the substrate modules M0, M2, M4, M11, and M12. The plan treats these five modules as the paper factory: experiment identity, event logging, episode execution, artifact packaging, and reporting. Paper-specific analysis code is allowed only as thin plugins under papers/P15, papers/P08, and papers/P05; once stable, those plugins can later be promoted into M6, M8, M9, or M10.



Paper



Core claim



Mandatory 7-day RQs



Finish line with only M0/M2/M4/M11/M12



P15 GuardBreaker



Composed jailbreak defenses can look robust in static matrices but fail when attacks adapt to the deployed pipeline.



RQ1 adaptive-vs-static ASR; RQ2 defense-composition monotonicity; RQ3 feedback-signal leverage if budget allows.



Day 3: attack/eval configs, adaptive-vs-static table, composition gain/loss table, artifact README.



P08 CrossJailbreak-1M



Organic jailbreak attempts form transfer clusters and model vulnerability partial orders different from synthetic benchmark rankings.



RQ1 strategy clusters; RQ2 asymmetric cross-model transfer; RQ3 replay validation if data/model access is ready.



Day 3: cluster assignments, transfer matrix, safety-clique graph/table, artifact README.



P05 MCPDrift



Semantic, schema, permission, and destination drift predicts security-relevant MCP updates better than hash/version changes.



RQ1 drift detection; RQ2 changed-component localization; RQ3 early-warning/scanner comparison if labels are ready.



Day 4: 50-150 labeled release/update pairs, drift feature table, localization table, artifact README.







2. Critical Path: Build These Modules First



Order



Module



Concrete output



Which paper finishes because of it



Day 1 AM



M0 Reproducibility Core



paper manifests, IDs, config schemas, registries, cache, ledger, run folders



All three papers can register experiments and create reproducible run folders.



Day 1 PM



M2 Event and Provenance Schema



validated JSONL event schema with P15/P08/P05-specific event types



P15/P08/P05 can log comparable evidence without paper-specific logging.



Day 2 AM



M4 Benchmark Harness



task interface, adapter interface, evaluator interface, matrix runner, dummy task pack



P15/P08 can run RQ1-RQ2 pilots; P05 can run static diff tasks as M4 task packs.



Day 2 PM



M11 Paper/Artifact Factory



paper folders, data cards, README templates, artifact bundles



All three have submission-style artifact shells from first result.



Day 2 PM-3 AM



M12 Reporting Layer



result aggregator, RQ dashboard, paper-specific table templates, failure-case exporter



P15/P08 get Day-3 tables; P05 gets Day-4 tables after labels land.







2.1 Minimal Repository Tree



agentseclab_x/



  core/



    ids.py                 # M0: stable IDs and run hashes



    config.py              # M0: typed config loaders and validators



    registry.py            # M0: model/task/attack/defense/paper registries



    cache.py               # M0: response/tool-output cache



    events.py              # M2: event dataclasses/Pydantic models



    event_writer.py         # M2: JSONL append/validate/write helpers



    runner.py              # M4: run_episode()



    matrix_runner.py        # M4: run_matrix()



  benchmarks/



    base_task.py            # M4: TaskPack interface



    jailbreak/              # P15/P08 task packs



    mcpdrift/               # P05 release/update task pack



  evaluators/



    base.py                 # M4: evaluator interface



  reporting/



    aggregate.py            # M12: event -> result rows



    rq_dashboard.py         # M12: RQ status tracker



    make_tables.py          # M12: paper tables



    make_figures.py         # M12: paper figures



    export_cases.py         # M12: representative cases



  artifacts/



    make_artifact.py        # M11: bundle builder



    templates/              # M11: README, data card, Open Science appendix



  papers/



    P15_GuardBreaker/



    P08_CrossJailbreak1M/



    P05_MCPDrift/



  configs/



    models.yaml



    papers.yaml



    datasets.yaml



    attacks.yaml



    defenses.yaml



    experiments/



  schemas/



    event_schema.json



    result_schema.json



  tests/



3. Existing Codebases to Reuse: Why and How



Codebase / source



Use for



Why this codebase



Concrete reuse instruction



JailbreakBench / jailbreakbench



P15 and P08; M0/M4/M11/M12



Standardized jailbreak behaviors, artifacts, scoring conventions, and reproducibility patterns.



Mirror behavior IDs and artifact folders; use its behavior/scoring separation as your configs/datasets.yaml and reporting templates.



HarmBench



P15 and P08; evaluator utilities



HarmBench provides harmful-behavior taxonomy and classifier/evaluation culture for automated red-teaming.



Use behaviors as seed categories; wrap classifier/judge output into M2 JudgeScoreEvent; reuse behavior-level tables.



StrongREJECT



P08 and P15 output scoring



Richer refusal/compliance scoring than binary keyword refusal; useful for organic compliance labels.



Call evaluator in a paper-specific scoring plugin; store score, refusal flag, and explanation hash as M2 JudgeScoreEvent.



EasyJailbreak package / framework



P15 attack integration



Selector/Mutator/Constraint/Evaluator abstraction maps cleanly to adaptive attack loops.



Implement an AttackAdapter wrapper that exposes generate(), adapt(), reset(), cost(); do not modify upstream package.



PAIR / JailbreakingLLMs



P15 adaptive black-box baseline



PAIR is a known iterative black-box attack engine and is already listed in the source plans.



Wrap PAIR as attack_backend=pair in configs/attacks.yaml; log every candidate as AttackCandidateEvent.



GPTFuzz



P15 mutation search baseline



Mutation/fuzzing operators support adaptive objective search and feedback-channel ablations.



Expose mutators as P15 plugins; each mutation receives previous prompt, guard result, and target result.



AutoDAN



P15 stealth-oriented optimization baseline



Optimization baseline for stealthy jailbreak prompts; useful as a stronger but slower stretch backend.



Keep optional; run on local/open model only if Day-3 P15 table is already complete.



llm-attacks / GCG



P15 white-box suffix baseline



White-box baseline for local models; useful for contrast with black-box adaptation.



Use only when local model gradients are available; otherwise mark as deferred.



LMSYS-Chat-1M dataset



P08 organic data source



Provides real-world conversations, model names, language tags, and moderation flags.



Load streaming Parquet; create P08 jailbreak_corpus.parquet with conv_id, model, language, prompt, response, moderation flag, compliance label.



modelcontextprotocol/servers



P05 benign corpus and schema examples



Reference/community MCP server corpus with manifests and schemas.



Clone selected repos with commit hashes; extract tool names, parameter schemas, manifests, release tags.



modelcontextprotocol/python-sdk



P05 replay-compatible schema understanding



Official SDK for local MCP server/client behavior when you later add replay; not required for static RQ1-RQ2.



Use schema examples and optional local replay; record server metadata as ReleaseSnapshotEvent.



Snyk agent-scan



P05 scanner baseline



Security scanner for AI agents, MCP servers, and skills.



Run scanner on selected snapshots; record findings as ScannerFindingEvent; use as baseline against drift score.



GitPython / git CLI



P05 release history collection



Fast, deterministic collection of commits, tags, diffs, and changed files.



Use git clone --bare or GitPython to get release pairs; write snapshot manifests and diff records.



Gitleaks / TruffleHog / Semgrep



P05 optional static baselines



Secret/static scans provide simple baseline signals for risky updates.



Run only on controlled public/local snapshots; save JSON output and normalize into M2 ScannerFindingEvent.







4. M0 — Reproducibility Core



Priority: Day 1 morning



Purpose: Every run, paper, RQ, dataset, attack, defense, model, seed, and artifact has an immutable identity. M0 prevents the three papers from becoming three unrelated codebases.



4.x.1 Codebases to reuse and why



Codebase/source



Part to reuse



How to use



JailbreakBench



Behavior/artifact and scoring conventions



Use for paper manifests, behavior IDs, and reproducibility culture for P15/P08.



HarmBench



Behavior taxonomy and evaluator style



Use for P15/P08 behavior categories and metric naming.



PurpleLlama CybersecurityBenchmarks



Runner/model adapter/result organization



Use organization patterns for configs/results even though these three papers are not cyber-range heavy.







4.x.2 Files to create with input/output contracts



File



What goes inside / input



Output / consumed by



agentseclab_x/core/ids.py



Inputs: config dict, git commit, seed, timestamp policy. Output: stable run_id, scenario_id, config_hash, artifact_id. Contains make_run_id(), make_config_hash(), short_id().



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/config.py



Inputs: YAML files from configs/. Output: validated ExperimentConfig, PaperManifest, DatasetManifest, ModelConfig, AttackConfig, DefenseConfig objects. Contains schema validation and defaults.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/registry.py



Inputs: configs/models.yaml, datasets.yaml, attacks.yaml, defenses.yaml, papers.yaml. Output: registry lookup objects used by M4. Contains get_model(), get_taskpack(), get_attack(), get_defense(), get_paper().



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/cache.py



Inputs: request hash, provider/model, prompt/tool args. Output: cached response JSON or cache miss. Contains read_cache(), write_cache(), cache_key().



Consumed by M4/M11/M12 and paper-specific plugins.



configs/papers.yaml



Inputs: manually written paper entries. Output: paper metadata for P15/P08/P05 including RQs, primary metric, module list, output folder, owner.



Consumed by M4/M11/M12 and paper-specific plugins.



configs/experiments/*.yaml



Inputs: per-RQ experiment definitions. Output: executable experiment matrix for M4.



Consumed by M4/M11/M12 and paper-specific plugins.



artifacts/experiment_ledger.csv



Inputs: append-only run metadata. Output: daily ledger consumed by M12 and M11.



Consumed by M4/M11/M12 and paper-specific plugins.



scripts/freeze_manifest.py



Inputs: experiment folder. Output: manifest.json containing configs, code commit, dependency lock hash, dataset version, run IDs.



Consumed by M4/M11/M12 and paper-specific plugins.







4.x.3 Detailed development steps



Initialize repository and configs/ directory; create P15, P08, P05 paper manifests first.



Define ID hierarchy: paper_id -> rq_id -> dataset_id -> scenario_id -> run_id. Use stable hashes rather than manual names.



Create default experiment templates: p15_rq1_adaptive_vs_static.yaml, p15_rq2_composition.yaml, p08_rq1_clusters.yaml, p08_rq2_transfer.yaml, p05_rq1_drift_detection.yaml, p05_rq2_localization.yaml.



Add a config validator that fails fast when a required field is absent.



Add run-folder constructor that creates runs/{paper_id}/{rq_id}/{run_id}/ with config.yaml, manifest.json, events.jsonl, metrics.csv, logs/, tables/, figures/.



Add cache and cost ledger hooks even if you start with dummy/local outputs; do not retrofit logging after experiments.



4.x.4 Acceptance test and paper unblocking effect



Acceptance test: `python scripts/init_experiment.py --paper P15 --rq RQ1 --config configs/experiments/p15_rq1_adaptive_vs_static.yaml` creates a run folder with config.yaml, manifest.json, events.jsonl, metrics.csv, and README_stub.md. Repeat for P08 RQ1 and P05 RQ1.



Paper



Unlocked after M0



P15



Experiment configs for adaptive/static runs and defense-composition matrix.



P08



Corpus mining and cluster/transfer run manifests.



P05



Release-pair manifest and label ledger.







4. M2 — Unified Event and Provenance Schema



Priority: Day 1 afternoon



Purpose: All evidence becomes one validated JSONL format. M2 is what allows M12 to generate tables without paper-specific parsers.



4.x.1 Codebases to reuse and why



Codebase/source



Part to reuse



How to use



AgentDojo



Task/injection/defense/security result separation



Use the separation principle, then generalize into a unified event log.



CyberSecEval-MCP plan



Tool, argument, observation, trust label, policy boundary fields



Use field vocabulary for P05/P15 and future compatibility.



LMSYS-Chat-1M metadata



conversation_id, model, language, moderation fields



Normalize P08 records into M2 DatasetRecord and ModelResponseEvent.







4.x.2 Files to create with input/output contracts



File



What goes inside / input



Output / consumed by



schemas/event_schema.json



Inputs: event JSON objects. Output: validation pass/fail. Defines required fields and paper-specific extensions.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/events.py



Inputs: Python event constructor calls. Output: typed event dictionaries. Classes: RunStart, DatasetRecord, AttackCandidateEvent, ModelResponseEvent, GuardDecisionEvent, JudgeScoreEvent, ClusterEvent, TransferEdgeEvent, ReleaseSnapshotEvent, DiffFeatureEvent, ScannerFindingEvent, FinalMetricEvent.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/provenance.py



Inputs: parent_event_ids and source metadata. Output: provenance chains and content hashes. Contains hash_content(), make_parent_chain(), trust_summary().



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/event_writer.py



Inputs: Event objects and run directory. Output: append-only events.jsonl with schema validation. Contains write_event(), load_events(), validate_event_file().



Consumed by M4/M11/M12 and paper-specific plugins.



tests/test_event_schema.py



Inputs: hand-authored traces. Output: pytest pass/fail for clean run, successful attack, blocked attack, failed scanner, transfer edge, release drift.



Consumed by M4/M11/M12 and paper-specific plugins.



docs/EVENT_SCHEMA.md



Inputs: schema. Output: human-readable documentation and examples for students.



Consumed by M4/M11/M12 and paper-specific plugins.







4.x.3 Detailed development steps



Create a base event with universal fields: event_id, run_id, paper_id, rq_id, scenario_id, step_id, event_type, timestamp, actor, source_channel, trust_level, content_hash, parent_event_ids, outcome.



Add P15 fields: attack_name, attack_family, defense_pipeline, feedback_signal, candidate_text_hash, guard_labels, judge_scores, query_count, adaptive_round.



Add P08 fields: conv_id, source_model, target_model, cluster_id, compliance_label, language, harm_category, transfer_source, transfer_target, support_n, ci_low, ci_high.



Add P05 fields: repo_id, server_name, version_from, version_to, changed_component, diff_type, permission_delta, schema_delta, destination_delta, scanner_name, scanner_finding, gold_label.



Ensure every event can be anonymized: text fields stored as hash by default; safe text snippets go in cases/ only after manual review.



Write hand-authored example traces before writing adapters; this makes M12 implementation deterministic.



4.x.4 Acceptance test and paper unblocking effect



Acceptance test: `pytest tests/test_event_schema.py` validates at least six traces: P15 adaptive candidate, P15 defense-composition run, P08 cluster event, P08 transfer edge, P05 diff feature, P05 scanner finding. M12 must compute at least one dummy metric from each.



Paper



M2 event types needed



P15



AttackCandidateEvent, GuardDecisionEvent, JudgeScoreEvent, FeedbackSignalEvent, FinalMetricEvent.



P08



DatasetRecord, ConversationEvent, ComplianceLabelEvent, ClusterEvent, TransferEdgeEvent, ReplayEvent.



P05



ReleaseSnapshotEvent, DiffFeatureEvent, DriftLabelEvent, ScannerFindingEvent, FinalMetricEvent.







4. M4 — Benchmark Harness



Priority: Day 2 morning



Purpose: M4 executes experiments defined by M0 and logs them through M2. For these three papers, M4 is mostly offline/semioffline: jailbreak runs, organic-data analysis runs, and release-diff tasks.



4.x.1 Codebases to reuse and why



Codebase/source



Part to reuse



How to use



JailbreakBench



behavior/scoring/run separation



Use for P15/P08 taskpack structure.



HarmBench



red-team evaluation utilities



Use as evaluator baseline where applicable.



modelcontextprotocol/servers + git



release/schema corpus



Use for P05 release drift taskpack.







4.x.2 Files to create with input/output contracts



File



What goes inside / input



Output / consumed by



agentseclab_x/benchmarks/base_task.py



Defines TaskPack interface. Input: config and data path. Output: iterable scenarios with success_oracle and violation_oracle.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/evaluators/base.py



Defines Evaluator interface. Input: M2 event file or model outputs. Output: metric events and result rows.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/runner.py



Input: one ExperimentConfig. Output: one run folder with events.jsonl and metrics.csv. Contains run_episode() and run_offline_analysis().



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/matrix_runner.py



Input: experiment matrix YAML. Output: multiple run folders and aggregate run index. Supports checkpoint/resume.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/benchmarks/jailbreak/guardbreaker_task.py



P15. Input: behaviors, attack backends, defense pipelines. Output: attack attempts and guard/judge events.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/benchmarks/jailbreak/organic_transfer_task.py



P08. Input: jailbreak_corpus.parquet or sample CSV. Output: cluster events, transfer edges, replay events.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/benchmarks/mcpdrift/release_drift_task.py



P05. Input: snapshot manifest pairs. Output: release snapshot events, diff feature events, scanner finding events, drift labels.



Consumed by M4/M11/M12 and paper-specific plugins.



configs/experiments/*.yaml



Input to matrix_runner. Each config has paper_id, rq_id, dataset, taskpack, model/evaluator, baseline, seeds, output_path.



Consumed by M4/M11/M12 and paper-specific plugins.







4.x.3 Detailed development steps



Implement BaseTaskPack and BaseEvaluator before paper-specific task packs.



Create a dummy model/guard/evaluator so the harness can be tested without API keys.



Implement P15 GuardBreakerTaskPack: read behaviors, generate static attacks, run one adaptive backend, log guard labels, target responses, judge scores, query counts.



Implement P08 OrganicTransferTaskPack: load corpus, filter compliant/refusal labels, embed/cluster or load precomputed clusters, compute directed transfer matrix and support thresholds.



Implement P05 ReleaseDriftTaskPack: read repo/version pairs, compute basic diffs, run scanner JSON if available, emit drift features and gold labels.



Add resume logic: if a run folder already has finished.ok, skip unless --force is set.



Add exception handling as M2 ErrorEvent; failed runs are evidence, not silent missing cells.



4.x.4 Acceptance test and paper unblocking effect



Acceptance test: `python -m agentseclab_x.core.matrix_runner --config configs/experiments/smoke_p15_p08_p05.yaml` runs one tiny scenario per paper and writes events.jsonl, metrics.csv, and run_index.csv.



Paper



First runnable taskpack



P15



GuardBreakerTaskPack: behaviors x attack backend x defense pipeline x seed.



P08



OrganicTransferTaskPack: corpus -> clusters -> transfer edges -> matrices.



P05



ReleaseDriftTaskPack: release pairs -> diff features -> drift labels -> scanner baselines.







4. M11 — Paper and Artifact Factory



Priority: Day 2 afternoon



Purpose: M11 ensures each paper has a reproducible artifact package from the first successful run. It prevents final-week scrambling.



4.x.1 Codebases to reuse and why



Codebase/source



Part to reuse



How to use



JailbreakBench



artifact release conventions



Use README/data structure patterns for P15/P08.



HarmBench



behavior documentation style



Use behavior/category documentation patterns.



USENIX/NDSS artifact expectations



README, smoke test, Open Science appendix



Generate artifact shell from day one.







4.x.2 Files to create with input/output contracts



File



What goes inside / input



Output / consumed by



agentseclab_x/artifacts/make_artifact.py



Inputs: paper_id and run IDs. Output: artifact zip/folder with README, configs, sample traces, tables, figures, data card, smoke command.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/artifacts/templates/README.md



Template. Contains install, smoke run, expected outputs, safety constraints, reproduction commands.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/artifacts/templates/DATA_CARD.md



Template. Contains dataset source, inclusion criteria, labels, splits, safety filtering, limitations.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/artifacts/templates/OPEN_SCIENCE.md



Template. Contains artifact availability, reproducibility, compute cost, excluded materials, ethics/safety.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/papers/P15_GuardBreaker/



Contains paper_manifest.yaml, RQ_STATUS.md, threat_model.md, tables/, figures/, cases/, appendix/, artifact_notes.md.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/papers/P08_CrossJailbreak1M/



Same structure plus data_access.md and privacy_filtering.md.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/papers/P05_MCPDrift/



Same structure plus release_label_policy.md and scanner_baselines.md.



Consumed by M4/M11/M12 and paper-specific plugins.



scripts/anonymize_artifact.py



Inputs: artifact folder. Output: anonymized artifact folder. Removes usernames, absolute paths, API logs, credentials, and unsafe payload text.



Consumed by M4/M11/M12 and paper-specific plugins.







4.x.3 Detailed development steps



Create paper folders on Day 1; do not wait for final results.



Generate README and data-card drafts from M0 manifests and M2 schema automatically.



Add a smoke config for each paper: p15_smoke.yaml, p08_smoke.yaml, p05_smoke.yaml. These should run in under five minutes on a laptop using dummy/local data.



Export only safe artifacts: configs, hashes, synthetic canaries, sampled sanitized traces, aggregate tables, and documentation. Do not export raw harmful prompts unless separately approved and sanitized.



Add artifact status to RQ_STATUS.md: smoke_passed, result_regenerated, table_regenerated, anonymized, ready_for_internal_review.



4.x.4 Acceptance test and paper unblocking effect



Acceptance test: `python -m agentseclab_x.artifacts.make_artifact --paper P15 --runs runs/P15/...` creates a zip containing README, configs, sample events, metrics, tables, figures, and data card. Repeat for P08 and P05.



Paper



Artifact folder must contain



P15



attack configs, defense-pipeline configs, sample sanitized events, adaptive/static table, composition table.



P08



data-card, privacy filtering note, cluster assignment sample, transfer matrix sample, replay config.



P05



release label policy, snapshot manifests, diff features, scanner outputs, drift tables.







4. M12 — Reporting Layer



Priority: Day 2 evening to Day 3 morning



Purpose: M12 turns M2 event logs into RQ dashboards, tables, figures, and failure cases. It includes a lightweight metric registry for the three selected papers.



4.x.1 Codebases to reuse and why



Codebase/source



Part to reuse



How to use



Jailbreak security plan



one-row-per-attempt reporting and heatmaps



Use for P15 adaptive attack tables and P08 transfer matrices.



HarmBench/JailbreakBench



behavior-level result tables



Use for P15/P08 reporting.



Pandas/Matplotlib/NetworkX



lightweight analysis and figure generation



Use to avoid manual spreadsheets and regenerate figures from logs.







4.x.2 Files to create with input/output contracts



File



What goes inside / input



Output / consumed by



agentseclab_x/reporting/result_schema.py



Defines universal result row. Input: M2 events. Output: normalized rows with paper_id, rq_id, metric values, confidence fields.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/aggregate.py



Inputs: run folders or events.jsonl. Output: results.parquet/csv and aggregate metrics.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/rq_dashboard.py



Inputs: results + paper manifests. Output: RQ_STATUS.md and dashboard.csv with NOT_STARTED/DATA_READY/RUNNING/RESULT_READY/TABLE_READY/DRAFT_READY.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/make_tables.py



Inputs: results.parquet and paper_id. Output: paper tables in CSV, Markdown, and LaTeX/DOCX-ready text.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/make_figures.py



Inputs: results.parquet. Output: PNG/PDF heatmaps, line plots, transfer graphs, drift timelines.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/export_cases.py



Inputs: events + metrics. Output: sanitized true positives, false negatives, false positives, representative case studies.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/templates/*.yaml



Per-paper table definitions for P15/P08/P05.



Consumed by M4/M11/M12 and paper-specific plugins.







4.x.3 Detailed development steps



Define universal result row: run_id, paper_id, rq_id, scenario_id, dataset_id, attack_id, model, defense, seed, primary_metric, TSR, ASR, utility, cost, latency, ci_low, ci_high, status.



Implement P15 metrics: static_ASR, adaptive_ASR, adaptive_gain, guard_evasion_rate, composition_gain_loss, feedback_ablation_effect, queries_to_bypass.



Implement P08 metrics: n_clusters, cluster_purity, transfer_T_ij, support_n, bootstrap_CI_width, clique_count, vulnerability_partial_order_edges, replay_success_rate.



Implement P05 metrics: drift_precision, drift_recall, feature_auc, changed_component_F1, first_risky_version_accuracy, scanner_agreement, early_warning_lead.



Generate paper-specific tables: P15 adaptive-vs-static table and composition matrix; P08 transfer heatmap and cluster table; P05 drift-feature ablation and localization table.



Export sanitized case studies: no raw harmful prompt by default; show hashes, categories, and safe summaries.



4.x.4 Acceptance test and paper unblocking effect



Acceptance test: `python -m agentseclab_x.reporting.rq_dashboard --papers P15 P08 P05 --runs runs/` writes dashboards and at least one table per paper using smoke results; with real results it marks RQ1-RQ2 as TABLE_READY.



Paper



Main M12 outputs



P15



Table 1 adaptive vs static ASR; Table 2 defense-composition gain/loss; Figure 1 feedback ablation.



P08



Figure transfer heatmap; Table strategy clusters; Graph safety cliques/partial order.



P05



Table drift-feature ablation; Table changed-component localization; Figure release timeline.







5. Paper-Specific Implementation Plans Using Only the Five Core Modules



The following plans intentionally use M0, M2, M4, M11, and M12 only. Where the full Top-15 plan would use M5/M6/M8/M9/M10, this document implements a minimal paper-local plugin and logs it through M2. After the first seven days, promote the stable plugin into the full shared module.



5.1 P15 GuardBreaker



RQ



Question



Core-module-only implementation



RQ1



How much does adaptive PAIR/TAP/GPTFuzz/AutoDAN/GCG-style search increase ASR over non-adaptive attacks against single defenses?



Use P15 AttackAdapter plugins under M4. Run static template baseline and one adaptive backend with matched query budgets. M12 computes adaptive_gain = adaptive_ASR - static_ASR.



RQ2



Does composing multiple guards monotonically improve security, or can composition create new bypass surfaces?



Define defense_pipeline in configs/defenses.yaml: none, pre_guard, post_guard, pre+post, pre+target-refusal+post. M4 runs the same attacks across pipelines; M12 computes composition_gain_loss.



RQ3



Which feedback signals provide the most leverage to an adaptive attacker?



Stretch with same modules: configure feedback_signal = accept/reject, score, label, latency, target_response. M12 outputs feedback ablation table.







P15 files to create



File



Input



Output / content



papers/P15_GuardBreaker/paper_manifest.yaml



Manual paper metadata



RQs, primary metrics, module list, target venues, owner, dataset/behavior list.



configs/experiments/p15_rq1_adaptive_vs_static.yaml



behavior set, attack backends, defenses, target models, budget



M4 matrix over static vs adaptive runs.



configs/experiments/p15_rq2_composition.yaml



same behavior set, defense pipeline list



M4 matrix over defense compositions.



agentseclab_x/benchmarks/jailbreak/guardbreaker_task.py



M0 config + behavior records + attack adapter



M2 events: AttackCandidateEvent, GuardDecisionEvent, JudgeScoreEvent.



papers/P15_GuardBreaker/plugins/attack_adapters.py



EasyJailbreak/PAIR/GPTFuzz/AutoDAN wrappers



common generate/adapt/reset/cost interface.



papers/P15_GuardBreaker/plugins/guard_pipelines.py



defense names and order



serial pipeline object that emits guard decisions.



papers/P15_GuardBreaker/analysis/p15_metrics.yaml



M12 metric definitions



adaptive_gain, guard_evasion_rate, queries_to_bypass, composition_gain_loss.







P15 7-day steps



Day 1: Create paper manifest, behavior subset, target model list, and defense pipeline list. Use 50-100 safe benchmark behaviors or sanitized IDs; avoid raw dangerous text in release artifacts.



Day 2: Implement GuardBreakerTaskPack and dummy attack backend; confirm M2 logs attack candidates, guard labels, judge scores, and costs.



Day 3: Integrate one real adaptive backend first: PAIR or GPTFuzz. Run RQ1 static vs adaptive on 3 attacks x 3 guards x 2 target models x small budget. Generate P15 Table 1.



Day 3 evening: Run RQ2 composition matrix: no defense, pre-guard, post-guard, pre+post. Generate composition gain/loss table. This is the paper finish line for RQ1-RQ2.



Day 4-5: Add feedback-channel ablation for RQ3 if RQ1/RQ2 are stable: accept/reject only, guard label, guard score, target response.



Day 6: Add sanity checks: same query budget, same seed set, same behavior set, benign utility controls, failure cases.



Day 7: Freeze artifact, create sanitized representative cases, write RQ_STATUS.md, update threat_model.md and limitations.



5.2 P08 CrossJailbreak-1M



RQ



Question



Core-module-only implementation



RQ1



What jailbreak strategy clusters emerge from large-scale organic conversations?



M4 runs an offline corpus pipeline. It loads conversations, filters candidate jailbreak/unsafe records, embeds or uses precomputed embeddings, clusters, emits ClusterEvent rows.



RQ2



Do successful jailbreaks transfer asymmetrically across models, forming safety cliques or vulnerability partial orders?



M4 computes cluster-level directed transfer T[i,j]; M12 produces heatmap, support table, clique table, and partial-order edge table.



RQ3



Can top organic clusters be replay-validated on current models under controlled budgets?



Stretch with same M4 harness: use top 10 clusters and evaluate current target models with sanitized prompts or internal approval-only samples.







P08 files to create



File



Input



Output / content



papers/P08_CrossJailbreak1M/paper_manifest.yaml



paper metadata



RQs, data access assumptions, privacy filtering, artifact policy.



configs/experiments/p08_rq1_clusters.yaml



corpus path, embedding model, clustering params



M4 clustering pipeline config.



configs/experiments/p08_rq2_transfer.yaml



cluster file, compliance labels, model list



M4 transfer matrix pipeline config.



agentseclab_x/benchmarks/jailbreak/organic_transfer_task.py



jailbreak_corpus.parquet or sample CSV



ClusterEvents, TransferEdgeEvents, matrix outputs.



papers/P08_CrossJailbreak1M/plugins/lmsys_loader.py



LMSYS Parquet or local sample



normalized records: conv_id, model, language, prompt_hash, response_hash, moderation flag.



papers/P08_CrossJailbreak1M/plugins/compliance_labeler.py



responses + scorer



compliance_label, confidence, evaluator version.



papers/P08_CrossJailbreak1M/plugins/cluster_transfer.py



embeddings/clusters/compliance labels



cluster assignment table, T matrix, support table, bootstrap CIs.



papers/P08_CrossJailbreak1M/privacy_filtering.md



manual policy



how raw prompts/responses are filtered, hashed, summarized, and excluded from public artifacts.







P08 7-day steps



Day 1: Confirm data path: full LMSYS-Chat-1M, local preprocessed sample, or internal sanitized subset. Create data_access.md documenting what is available.



Day 2: Build lmsys_loader.py and produce a small normalized corpus with conv_id, model, language, harm/moderation flag, prompt_hash, response_hash, and safe summary.



Day 3: Run RQ1 clustering on 10k-50k candidates or the maximum available local sample. Output cluster_assignments.parquet, cluster_summary.csv, and M2 ClusterEvents.



Day 3 evening: Run RQ2 transfer on cluster-level compliance labels. Output transfer_matrix.csv, support_matrix.csv, and bootstrap_CI.csv. This is the RQ1-RQ2 finish line.



Day 4-5: Run replay validation for RQ3 on top 10 clusters if you have model access and approval to use sanitized cluster representatives.



Day 6: Add robustness checks: minimum support thresholds, alternate clustering method, high-confidence-only labels.



Day 7: Package sanitized atlas, matrix, cluster summaries, and artifact README; exclude raw harmful text unless separately approved.



5.3 P05 MCPDrift



RQ



Question



Core-module-only implementation



RQ1



Can semantic, permission, schema, and destination drift predict security-relevant MCP updates better than hash/version changes?



Use a paper-local P05 diff plugin under M4 instead of full M6. It loads snapshot pairs, computes feature rows, compares against labels, and M12 reports precision/recall/AUC.



RQ2



Which drift features best localize the changed component that introduced risk?



Each DiffFeatureEvent includes changed_component. M12 computes localization accuracy/F1 by feature group.



RQ3



Can drift scoring provide early warning before runtime attack success or incident-like behavior?



Stretch: add time/release index and scanner finding time. M12 computes lead time against labeled first-risky version.







P05 files to create



File



Input



Output / content



papers/P05_MCPDrift/paper_manifest.yaml



paper metadata



RQs, repo list, label policy, drift features, baselines.



configs/experiments/p05_rq1_drift_detection.yaml



repo list, version pairs, feature flags



M4 release-drift task config.



configs/experiments/p05_rq2_localization.yaml



diff feature rows, labels



changed-component localization experiment.



agentseclab_x/benchmarks/mcpdrift/release_drift_task.py



snapshot manifest pairs



M2 ReleaseSnapshotEvents, DiffFeatureEvents, DriftLabelEvents.



papers/P05_MCPDrift/plugins/repo_collector.py



GitHub URLs or local repos



snapshot_manifest.jsonl with commit/tag/version IDs.



papers/P05_MCPDrift/plugins/diff_features.py



two snapshots



hash_delta, semantic_delta, schema_delta, permission_delta, endpoint_delta, dependency_delta.



papers/P05_MCPDrift/plugins/scanner_runner.py



snapshot path + scanner config



scanner findings normalized into M2 ScannerFindingEvent.



papers/P05_MCPDrift/release_label_policy.md



manual label guidelines



definition of security_relevant, changed_component, first_risky_version, evidence notes.







P05 7-day steps



Day 1: Select 30-50 MCP servers/repos first. Prioritize repos with visible tags/releases/commit history and readable schemas/manifests.



Day 2: Implement repo_collector.py and create snapshot manifests. If releases are sparse, use commit windows or curated v0/v1 pairs; record this in limitations.



Day 3: Implement diff_features.py with four feature groups: semantic description change, schema broadening, permission/scope change, new external destination. Output diff_features.parquet and M2 DiffFeatureEvents.



Day 3-4: Label 50-150 release/update pairs using release_label_policy.md. Minimum viable P05 needs 50 high-quality labels; strong result needs 150+.



Day 4: Run RQ1 drift detection and RQ2 localization. Generate feature ablation and localization tables. This is the RQ1-RQ2 finish line.



Day 5: Add scanner baseline with Snyk agent-scan and optional Gitleaks/TruffleHog/Semgrep. Compute scanner agreement and early-warning pilot.



Day 6: Add robustness check: benign-looking description/schema evasions and temporal holdout.



Day 7: Package snapshot manifests, label policy, feature tables, scanner outputs, and sanitized examples.



6. Seven-Day Execution Schedule and Finish-Line Mapping



Day



Module focus



P15 finish line



P08 finish line



P05 finish line



Day 1 AM



M0 configs, run IDs, paper manifests



P15 config and run folders initialized.



P08 data access and corpus manifest initialized.



P05 repo list and label ledger initialized.



Day 1 PM



M2 event schema and validators



P15 attack/guard/judge events validate.



P08 dataset/cluster/transfer events validate.



P05 snapshot/diff/scanner events validate.



Day 2 AM



M4 runner and taskpack API



P15 smoke run with dummy attack/guard.



P08 smoke run on small sample.



P05 smoke run on one release pair.



Day 2 PM



M11 artifact factory + M12 reporting skeleton



P15 RQ dashboard exists.



P08 RQ dashboard exists.



P05 RQ dashboard exists.



Day 3



P15/P08 experiments



RQ1-RQ2 table-ready: adaptive vs static and composition.



RQ1-RQ2 table-ready: clusters and transfer matrix.



Feature extraction starts; labels in progress.



Day 4



P05 experiments



RQ3 feedback pilot if stable.



RQ3 replay pilot if data/model access ready.



RQ1-RQ2 table-ready: drift detection and localization.



Day 5



Ablations and baselines



Feedback-channel ablation.



Support threshold/clustering robustness.



Scanner baseline and early-warning pilot.



Day 6



Failure cases and robustness



Representative failure cases and benign utility.



Replay-validation cases and confidence intervals.



Evasive update and temporal holdout.



Day 7



M11/M12 freeze



Artifact + draft sections: threat model, method, results.



Artifact + draft sections: data, method, transfer results.



Artifact + draft sections: data, diff features, results.







7. Concrete Commands to Implement First



# 1. Create the repo skeleton and paper folders



python scripts/init_repo.py --papers P15 P08 P05 --modules M0 M2 M4 M11 M12







# 2. Validate configs



python scripts/validate_configs.py --papers P15 P08 P05







# 3. Run smoke matrix after M4 exists



python -m agentseclab_x.core.matrix_runner --config configs/experiments/smoke_p15_p08_p05.yaml







# 4. Generate RQ dashboard after M12 exists



python -m agentseclab_x.reporting.rq_dashboard --papers P15 P08 P05 --runs runs/







# 5. Generate paper-specific tables



python -m agentseclab_x.reporting.make_tables --paper P15 --runs runs/P15



python -m agentseclab_x.reporting.make_tables --paper P08 --runs runs/P08



python -m agentseclab_x.reporting.make_tables --paper P05 --runs runs/P05







# 6. Package artifacts



python -m agentseclab_x.artifacts.make_artifact --paper P15 --runs runs/P15 --out artifacts/P15.zip



python -m agentseclab_x.artifacts.make_artifact --paper P08 --runs runs/P08 --out artifacts/P08.zip



python -m agentseclab_x.artifacts.make_artifact --paper P05 --runs runs/P05 --out artifacts/P05.zip



8. Appendix: Minimal Schemas



8.1 Experiment config skeleton



paper_id: P15



rq_id: RQ1



experiment_id: p15_rq1_adaptive_vs_static_v0



seed: 1



dataset:



  id: harmbench_subset_v0



  path: data/p15/behaviors_subset.csv



model:



  target: local_or_api_model_name



attacks:



  static: [template_baseline]



  adaptive: [pair_or_gptfuzz]



defenses:



  pipelines: [none, pre_guard, post_guard, pre_post]



budget:



  max_queries: 20



  max_tokens: 12000



outputs:



  run_dir: runs/P15/RQ1/



8.2 Minimal M2 event



{



  "event_id": "evt_000001",



  "run_id": "P15_RQ1_...",



  "paper_id": "P15",



  "rq_id": "RQ1",



  "scenario_id": "behavior_0001",



  "step_id": 3,



  "event_type": "GuardDecisionEvent",



  "timestamp": "2026-08-11T12:00:00-06:00",



  "actor": "guard",



  "source_channel": "model_output",



  "trust_level": 0.5,



  "content_hash": "sha256:...",



  "parent_event_ids": ["evt_000000"],



  "outcome": "blocked_or_allowed",



  "payload": {



    "guard_name": "wildguard_or_llamaguard",



    "label": "safe_or_unsafe",



    "score": 0.84,



    "defense_pipeline": "pre_post"



  }



}



8.3 Universal result row



run_id,paper_id,rq_id,scenario_id,dataset_id,model,attack,defense,seed,primary_metric,TSR,ASR,utility,cost,latency,ci_low,ci_high,status



P15_...,P15,RQ1,behavior_0001,harmbench_subset_v0,target_model,pair,pre_post,1,adaptive_gain,NA,0.42,0.95,1.23,18.4,0.31,0.52,TABLE_READY



9. Final Go/No-Go Criteria



Paper



Green by Day 3/4



Yellow fallback



Red condition



P15



RQ1 and RQ2 tables have real adaptive/static and composition values.



Use fewer attacks/guards but keep matched budgets.



No adaptive backend runs at all.



P08



Cluster table and transfer matrix generated from real or approved internal sample.



Use smaller sample and label as pilot; skip replay.



No data access and no internal sample.



P05



50+ high-quality labeled update pairs and drift/localization tables.



Use 20-50 hand-labeled cases as measurement-protocol pilot.



No release pairs or no labels.







Implementation principle: keep all raw harmful text, canary values, and potentially sensitive repository content out of public artifacts by default. The publishable artifact should contain manifests, hashes, sanitized summaries, code, configs, representative safe traces, and aggregate metrics.Focused Module-First Development Specification



P15 GuardBreaker · P08 CrossJailbreak-1M · P05 MCPDrift



Detailed implementation plan for M0, M2, M4, M11, and M12



Prepared for Md Jahangir Alam | Scope: build the first reusable substrate and finish RQ1-RQ2 evidence for P15, P08, and P05 in seven days. This is a separate write-up expanding Section 16.3 of the module-first Top-15 plan.



1. Operating Goal and Scope



For now, develop only P15, P08, and P05 using the substrate modules M0, M2, M4, M11, and M12. The plan treats these five modules as the paper factory: experiment identity, event logging, episode execution, artifact packaging, and reporting. Paper-specific analysis code is allowed only as thin plugins under papers/P15, papers/P08, and papers/P05; once stable, those plugins can later be promoted into M6, M8, M9, or M10.



Paper



Core claim



Mandatory 7-day RQs



Finish line with only M0/M2/M4/M11/M12



P15 GuardBreaker



Composed jailbreak defenses can look robust in static matrices but fail when attacks adapt to the deployed pipeline.



RQ1 adaptive-vs-static ASR; RQ2 defense-composition monotonicity; RQ3 feedback-signal leverage if budget allows.



Day 3: attack/eval configs, adaptive-vs-static table, composition gain/loss table, artifact README.



P08 CrossJailbreak-1M



Organic jailbreak attempts form transfer clusters and model vulnerability partial orders different from synthetic benchmark rankings.



RQ1 strategy clusters; RQ2 asymmetric cross-model transfer; RQ3 replay validation if data/model access is ready.



Day 3: cluster assignments, transfer matrix, safety-clique graph/table, artifact README.



P05 MCPDrift



Semantic, schema, permission, and destination drift predicts security-relevant MCP updates better than hash/version changes.



RQ1 drift detection; RQ2 changed-component localization; RQ3 early-warning/scanner comparison if labels are ready.



Day 4: 50-150 labeled release/update pairs, drift feature table, localization table, artifact README.











2. Critical Path: Build These Modules First



Order



Module



Concrete output



Which paper finishes because of it



Day 1 AM



M0 Reproducibility Core



paper manifests, IDs, config schemas, registries, cache, ledger, run folders



All three papers can register experiments and create reproducible run folders.



Day 1 PM



M2 Event and Provenance Schema



validated JSONL event schema with P15/P08/P05-specific event types



P15/P08/P05 can log comparable evidence without paper-specific logging.



Day 2 AM



M4 Benchmark Harness



task interface, adapter interface, evaluator interface, matrix runner, dummy task pack



P15/P08 can run RQ1-RQ2 pilots; P05 can run static diff tasks as M4 task packs.



Day 2 PM



M11 Paper/Artifact Factory



paper folders, data cards, README templates, artifact bundles



All three have submission-style artifact shells from first result.



Day 2 PM-3 AM



M12 Reporting Layer



result aggregator, RQ dashboard, paper-specific table templates, failure-case exporter



P15/P08 get Day-3 tables; P05 gets Day-4 tables after labels land.











2.1 Minimal Repository Tree



agentseclab_x/



  core/



    ids.py                 # M0: stable IDs and run hashes



    config.py              # M0: typed config loaders and validators



    registry.py            # M0: model/task/attack/defense/paper registries



    cache.py               # M0: response/tool-output cache



    events.py              # M2: event dataclasses/Pydantic models



    event_writer.py         # M2: JSONL append/validate/write helpers



    runner.py              # M4: run_episode()



    matrix_runner.py        # M4: run_matrix()



  benchmarks/



    base_task.py            # M4: TaskPack interface



    jailbreak/              # P15/P08 task packs



    mcpdrift/               # P05 release/update task pack



  evaluators/



    base.py                 # M4: evaluator interface



  reporting/



    aggregate.py            # M12: event -> result rows



    rq_dashboard.py         # M12: RQ status tracker



    make_tables.py          # M12: paper tables



    make_figures.py         # M12: paper figures



    export_cases.py         # M12: representative cases



  artifacts/



    make_artifact.py        # M11: bundle builder



    templates/              # M11: README, data card, Open Science appendix



  papers/



    P15_GuardBreaker/



    P08_CrossJailbreak1M/



    P05_MCPDrift/



  configs/



    models.yaml



    papers.yaml



    datasets.yaml



    attacks.yaml



    defenses.yaml



    experiments/



  schemas/



    event_schema.json



    result_schema.json



  tests/



3. Existing Codebases to Reuse: Why and How



Codebase / source



Use for



Why this codebase



Concrete reuse instruction



JailbreakBench / jailbreakbench



P15 and P08; M0/M4/M11/M12



Standardized jailbreak behaviors, artifacts, scoring conventions, and reproducibility patterns.



Mirror behavior IDs and artifact folders; use its behavior/scoring separation as your configs/datasets.yaml and reporting templates.



HarmBench



P15 and P08; evaluator utilities



HarmBench provides harmful-behavior taxonomy and classifier/evaluation culture for automated red-teaming.



Use behaviors as seed categories; wrap classifier/judge output into M2 JudgeScoreEvent; reuse behavior-level tables.



StrongREJECT



P08 and P15 output scoring



Richer refusal/compliance scoring than binary keyword refusal; useful for organic compliance labels.



Call evaluator in a paper-specific scoring plugin; store score, refusal flag, and explanation hash as M2 JudgeScoreEvent.



EasyJailbreak package / framework



P15 attack integration



Selector/Mutator/Constraint/Evaluator abstraction maps cleanly to adaptive attack loops.



Implement an AttackAdapter wrapper that exposes generate(), adapt(), reset(), cost(); do not modify upstream package.



PAIR / JailbreakingLLMs



P15 adaptive black-box baseline



PAIR is a known iterative black-box attack engine and is already listed in the source plans.



Wrap PAIR as attack_backend=pair in configs/attacks.yaml; log every candidate as AttackCandidateEvent.



GPTFuzz



P15 mutation search baseline



Mutation/fuzzing operators support adaptive objective search and feedback-channel ablations.



Expose mutators as P15 plugins; each mutation receives previous prompt, guard result, and target result.



AutoDAN



P15 stealth-oriented optimization baseline



Optimization baseline for stealthy jailbreak prompts; useful as a stronger but slower stretch backend.



Keep optional; run on local/open model only if Day-3 P15 table is already complete.



llm-attacks / GCG



P15 white-box suffix baseline



White-box baseline for local models; useful for contrast with black-box adaptation.



Use only when local model gradients are available; otherwise mark as deferred.



LMSYS-Chat-1M dataset



P08 organic data source



Provides real-world conversations, model names, language tags, and moderation flags.



Load streaming Parquet; create P08 jailbreak_corpus.parquet with conv_id, model, language, prompt, response, moderation flag, compliance label.



modelcontextprotocol/servers



P05 benign corpus and schema examples



Reference/community MCP server corpus with manifests and schemas.



Clone selected repos with commit hashes; extract tool names, parameter schemas, manifests, release tags.



modelcontextprotocol/python-sdk



P05 replay-compatible schema understanding



Official SDK for local MCP server/client behavior when you later add replay; not required for static RQ1-RQ2.



Use schema examples and optional local replay; record server metadata as ReleaseSnapshotEvent.



Snyk agent-scan



P05 scanner baseline



Security scanner for AI agents, MCP servers, and skills.



Run scanner on selected snapshots; record findings as ScannerFindingEvent; use as baseline against drift score.



GitPython / git CLI



P05 release history collection



Fast, deterministic collection of commits, tags, diffs, and changed files.



Use git clone --bare or GitPython to get release pairs; write snapshot manifests and diff records.



Gitleaks / TruffleHog / Semgrep



P05 optional static baselines



Secret/static scans provide simple baseline signals for risky updates.



Run only on controlled public/local snapshots; save JSON output and normalize into M2 ScannerFindingEvent.











4. M0 — Reproducibility Core



Priority: Day 1 morning



Purpose: Every run, paper, RQ, dataset, attack, defense, model, seed, and artifact has an immutable identity. M0 prevents the three papers from becoming three unrelated codebases.



4.x.1 Codebases to reuse and why



Codebase/source



Part to reuse



How to use



JailbreakBench



Behavior/artifact and scoring conventions



Use for paper manifests, behavior IDs, and reproducibility culture for P15/P08.



HarmBench



Behavior taxonomy and evaluator style



Use for P15/P08 behavior categories and metric naming.



PurpleLlama CybersecurityBenchmarks



Runner/model adapter/result organization



Use organization patterns for configs/results even though these three papers are not cyber-range heavy.











4.x.2 Files to create with input/output contracts



File



What goes inside / input



Output / consumed by



agentseclab_x/core/ids.py



Inputs: config dict, git commit, seed, timestamp policy. Output: stable run_id, scenario_id, config_hash, artifact_id. Contains make_run_id(), make_config_hash(), short_id().



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/config.py



Inputs: YAML files from configs/. Output: validated ExperimentConfig, PaperManifest, DatasetManifest, ModelConfig, AttackConfig, DefenseConfig objects. Contains schema validation and defaults.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/registry.py



Inputs: configs/models.yaml, datasets.yaml, attacks.yaml, defenses.yaml, papers.yaml. Output: registry lookup objects used by M4. Contains get_model(), get_taskpack(), get_attack(), get_defense(), get_paper().



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/cache.py



Inputs: request hash, provider/model, prompt/tool args. Output: cached response JSON or cache miss. Contains read_cache(), write_cache(), cache_key().



Consumed by M4/M11/M12 and paper-specific plugins.



configs/papers.yaml



Inputs: manually written paper entries. Output: paper metadata for P15/P08/P05 including RQs, primary metric, module list, output folder, owner.



Consumed by M4/M11/M12 and paper-specific plugins.



configs/experiments/*.yaml



Inputs: per-RQ experiment definitions. Output: executable experiment matrix for M4.



Consumed by M4/M11/M12 and paper-specific plugins.



artifacts/experiment_ledger.csv



Inputs: append-only run metadata. Output: daily ledger consumed by M12 and M11.



Consumed by M4/M11/M12 and paper-specific plugins.



scripts/freeze_manifest.py



Inputs: experiment folder. Output: manifest.json containing configs, code commit, dependency lock hash, dataset version, run IDs.



Consumed by M4/M11/M12 and paper-specific plugins.











4.x.3 Detailed development steps



Initialize repository and configs/ directory; create P15, P08, P05 paper manifests first.



Define ID hierarchy: paper_id -> rq_id -> dataset_id -> scenario_id -> run_id. Use stable hashes rather than manual names.



Create default experiment templates: p15_rq1_adaptive_vs_static.yaml, p15_rq2_composition.yaml, p08_rq1_clusters.yaml, p08_rq2_transfer.yaml, p05_rq1_drift_detection.yaml, p05_rq2_localization.yaml.



Add a config validator that fails fast when a required field is absent.



Add run-folder constructor that creates runs/{paper_id}/{rq_id}/{run_id}/ with config.yaml, manifest.json, events.jsonl, metrics.csv, logs/, tables/, figures/.



Add cache and cost ledger hooks even if you start with dummy/local outputs; do not retrofit logging after experiments.



4.x.4 Acceptance test and paper unblocking effect



Acceptance test: `python scripts/init_experiment.py --paper P15 --rq RQ1 --config configs/experiments/p15_rq1_adaptive_vs_static.yaml` creates a run folder with config.yaml, manifest.json, events.jsonl, metrics.csv, and README_stub.md. Repeat for P08 RQ1 and P05 RQ1.



Paper



Unlocked after M0



P15



Experiment configs for adaptive/static runs and defense-composition matrix.



P08



Corpus mining and cluster/transfer run manifests.



P05



Release-pair manifest and label ledger.











4. M2 — Unified Event and Provenance Schema



Priority: Day 1 afternoon



Purpose: All evidence becomes one validated JSONL format. M2 is what allows M12 to generate tables without paper-specific parsers.



4.x.1 Codebases to reuse and why



Codebase/source



Part to reuse



How to use



AgentDojo



Task/injection/defense/security result separation



Use the separation principle, then generalize into a unified event log.



CyberSecEval-MCP plan



Tool, argument, observation, trust label, policy boundary fields



Use field vocabulary for P05/P15 and future compatibility.



LMSYS-Chat-1M metadata



conversation_id, model, language, moderation fields



Normalize P08 records into M2 DatasetRecord and ModelResponseEvent.











4.x.2 Files to create with input/output contracts



File



What goes inside / input



Output / consumed by



schemas/event_schema.json



Inputs: event JSON objects. Output: validation pass/fail. Defines required fields and paper-specific extensions.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/events.py



Inputs: Python event constructor calls. Output: typed event dictionaries. Classes: RunStart, DatasetRecord, AttackCandidateEvent, ModelResponseEvent, GuardDecisionEvent, JudgeScoreEvent, ClusterEvent, TransferEdgeEvent, ReleaseSnapshotEvent, DiffFeatureEvent, ScannerFindingEvent, FinalMetricEvent.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/provenance.py



Inputs: parent_event_ids and source metadata. Output: provenance chains and content hashes. Contains hash_content(), make_parent_chain(), trust_summary().



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/event_writer.py



Inputs: Event objects and run directory. Output: append-only events.jsonl with schema validation. Contains write_event(), load_events(), validate_event_file().



Consumed by M4/M11/M12 and paper-specific plugins.



tests/test_event_schema.py



Inputs: hand-authored traces. Output: pytest pass/fail for clean run, successful attack, blocked attack, failed scanner, transfer edge, release drift.



Consumed by M4/M11/M12 and paper-specific plugins.



docs/EVENT_SCHEMA.md



Inputs: schema. Output: human-readable documentation and examples for students.



Consumed by M4/M11/M12 and paper-specific plugins.











4.x.3 Detailed development steps



Create a base event with universal fields: event_id, run_id, paper_id, rq_id, scenario_id, step_id, event_type, timestamp, actor, source_channel, trust_level, content_hash, parent_event_ids, outcome.



Add P15 fields: attack_name, attack_family, defense_pipeline, feedback_signal, candidate_text_hash, guard_labels, judge_scores, query_count, adaptive_round.



Add P08 fields: conv_id, source_model, target_model, cluster_id, compliance_label, language, harm_category, transfer_source, transfer_target, support_n, ci_low, ci_high.



Add P05 fields: repo_id, server_name, version_from, version_to, changed_component, diff_type, permission_delta, schema_delta, destination_delta, scanner_name, scanner_finding, gold_label.



Ensure every event can be anonymized: text fields stored as hash by default; safe text snippets go in cases/ only after manual review.



Write hand-authored example traces before writing adapters; this makes M12 implementation deterministic.



4.x.4 Acceptance test and paper unblocking effect



Acceptance test: `pytest tests/test_event_schema.py` validates at least six traces: P15 adaptive candidate, P15 defense-composition run, P08 cluster event, P08 transfer edge, P05 diff feature, P05 scanner finding. M12 must compute at least one dummy metric from each.



Paper



M2 event types needed



P15



AttackCandidateEvent, GuardDecisionEvent, JudgeScoreEvent, FeedbackSignalEvent, FinalMetricEvent.



P08



DatasetRecord, ConversationEvent, ComplianceLabelEvent, ClusterEvent, TransferEdgeEvent, ReplayEvent.



P05



ReleaseSnapshotEvent, DiffFeatureEvent, DriftLabelEvent, ScannerFindingEvent, FinalMetricEvent.











4. M4 — Benchmark Harness



Priority: Day 2 morning



Purpose: M4 executes experiments defined by M0 and logs them through M2. For these three papers, M4 is mostly offline/semioffline: jailbreak runs, organic-data analysis runs, and release-diff tasks.



4.x.1 Codebases to reuse and why



Codebase/source



Part to reuse



How to use



JailbreakBench



behavior/scoring/run separation



Use for P15/P08 taskpack structure.



HarmBench



red-team evaluation utilities



Use as evaluator baseline where applicable.



modelcontextprotocol/servers + git



release/schema corpus



Use for P05 release drift taskpack.











4.x.2 Files to create with input/output contracts



File



What goes inside / input



Output / consumed by



agentseclab_x/benchmarks/base_task.py



Defines TaskPack interface. Input: config and data path. Output: iterable scenarios with success_oracle and violation_oracle.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/evaluators/base.py



Defines Evaluator interface. Input: M2 event file or model outputs. Output: metric events and result rows.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/runner.py



Input: one ExperimentConfig. Output: one run folder with events.jsonl and metrics.csv. Contains run_episode() and run_offline_analysis().



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/core/matrix_runner.py



Input: experiment matrix YAML. Output: multiple run folders and aggregate run index. Supports checkpoint/resume.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/benchmarks/jailbreak/guardbreaker_task.py



P15. Input: behaviors, attack backends, defense pipelines. Output: attack attempts and guard/judge events.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/benchmarks/jailbreak/organic_transfer_task.py



P08. Input: jailbreak_corpus.parquet or sample CSV. Output: cluster events, transfer edges, replay events.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/benchmarks/mcpdrift/release_drift_task.py



P05. Input: snapshot manifest pairs. Output: release snapshot events, diff feature events, scanner finding events, drift labels.



Consumed by M4/M11/M12 and paper-specific plugins.



configs/experiments/*.yaml



Input to matrix_runner. Each config has paper_id, rq_id, dataset, taskpack, model/evaluator, baseline, seeds, output_path.



Consumed by M4/M11/M12 and paper-specific plugins.











4.x.3 Detailed development steps



Implement BaseTaskPack and BaseEvaluator before paper-specific task packs.



Create a dummy model/guard/evaluator so the harness can be tested without API keys.



Implement P15 GuardBreakerTaskPack: read behaviors, generate static attacks, run one adaptive backend, log guard labels, target responses, judge scores, query counts.



Implement P08 OrganicTransferTaskPack: load corpus, filter compliant/refusal labels, embed/cluster or load precomputed clusters, compute directed transfer matrix and support thresholds.



Implement P05 ReleaseDriftTaskPack: read repo/version pairs, compute basic diffs, run scanner JSON if available, emit drift features and gold labels.



Add resume logic: if a run folder already has finished.ok, skip unless --force is set.



Add exception handling as M2 ErrorEvent; failed runs are evidence, not silent missing cells.



4.x.4 Acceptance test and paper unblocking effect



Acceptance test: `python -m agentseclab_x.core.matrix_runner --config configs/experiments/smoke_p15_p08_p05.yaml` runs one tiny scenario per paper and writes events.jsonl, metrics.csv, and run_index.csv.



Paper



First runnable taskpack



P15



GuardBreakerTaskPack: behaviors x attack backend x defense pipeline x seed.



P08



OrganicTransferTaskPack: corpus -> clusters -> transfer edges -> matrices.



P05



ReleaseDriftTaskPack: release pairs -> diff features -> drift labels -> scanner baselines.











4. M11 — Paper and Artifact Factory



Priority: Day 2 afternoon



Purpose: M11 ensures each paper has a reproducible artifact package from the first successful run. It prevents final-week scrambling.



4.x.1 Codebases to reuse and why



Codebase/source



Part to reuse



How to use



JailbreakBench



artifact release conventions



Use README/data structure patterns for P15/P08.



HarmBench



behavior documentation style



Use behavior/category documentation patterns.



USENIX/NDSS artifact expectations



README, smoke test, Open Science appendix



Generate artifact shell from day one.











4.x.2 Files to create with input/output contracts



File



What goes inside / input



Output / consumed by



agentseclab_x/artifacts/make_artifact.py



Inputs: paper_id and run IDs. Output: artifact zip/folder with README, configs, sample traces, tables, figures, data card, smoke command.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/artifacts/templates/README.md



Template. Contains install, smoke run, expected outputs, safety constraints, reproduction commands.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/artifacts/templates/DATA_CARD.md



Template. Contains dataset source, inclusion criteria, labels, splits, safety filtering, limitations.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/artifacts/templates/OPEN_SCIENCE.md



Template. Contains artifact availability, reproducibility, compute cost, excluded materials, ethics/safety.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/papers/P15_GuardBreaker/



Contains paper_manifest.yaml, RQ_STATUS.md, threat_model.md, tables/, figures/, cases/, appendix/, artifact_notes.md.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/papers/P08_CrossJailbreak1M/



Same structure plus data_access.md and privacy_filtering.md.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/papers/P05_MCPDrift/



Same structure plus release_label_policy.md and scanner_baselines.md.



Consumed by M4/M11/M12 and paper-specific plugins.



scripts/anonymize_artifact.py



Inputs: artifact folder. Output: anonymized artifact folder. Removes usernames, absolute paths, API logs, credentials, and unsafe payload text.



Consumed by M4/M11/M12 and paper-specific plugins.











4.x.3 Detailed development steps



Create paper folders on Day 1; do not wait for final results.



Generate README and data-card drafts from M0 manifests and M2 schema automatically.



Add a smoke config for each paper: p15_smoke.yaml, p08_smoke.yaml, p05_smoke.yaml. These should run in under five minutes on a laptop using dummy/local data.



Export only safe artifacts: configs, hashes, synthetic canaries, sampled sanitized traces, aggregate tables, and documentation. Do not export raw harmful prompts unless separately approved and sanitized.



Add artifact status to RQ_STATUS.md: smoke_passed, result_regenerated, table_regenerated, anonymized, ready_for_internal_review.



4.x.4 Acceptance test and paper unblocking effect



Acceptance test: `python -m agentseclab_x.artifacts.make_artifact --paper P15 --runs runs/P15/...` creates a zip containing README, configs, sample events, metrics, tables, figures, and data card. Repeat for P08 and P05.



Paper



Artifact folder must contain



P15



attack configs, defense-pipeline configs, sample sanitized events, adaptive/static table, composition table.



P08



data-card, privacy filtering note, cluster assignment sample, transfer matrix sample, replay config.



P05



release label policy, snapshot manifests, diff features, scanner outputs, drift tables.











4. M12 — Reporting Layer



Priority: Day 2 evening to Day 3 morning



Purpose: M12 turns M2 event logs into RQ dashboards, tables, figures, and failure cases. It includes a lightweight metric registry for the three selected papers.



4.x.1 Codebases to reuse and why



Codebase/source



Part to reuse



How to use



Jailbreak security plan



one-row-per-attempt reporting and heatmaps



Use for P15 adaptive attack tables and P08 transfer matrices.



HarmBench/JailbreakBench



behavior-level result tables



Use for P15/P08 reporting.



Pandas/Matplotlib/NetworkX



lightweight analysis and figure generation



Use to avoid manual spreadsheets and regenerate figures from logs.











4.x.2 Files to create with input/output contracts



File



What goes inside / input



Output / consumed by



agentseclab_x/reporting/result_schema.py



Defines universal result row. Input: M2 events. Output: normalized rows with paper_id, rq_id, metric values, confidence fields.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/aggregate.py



Inputs: run folders or events.jsonl. Output: results.parquet/csv and aggregate metrics.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/rq_dashboard.py



Inputs: results + paper manifests. Output: RQ_STATUS.md and dashboard.csv with NOT_STARTED/DATA_READY/RUNNING/RESULT_READY/TABLE_READY/DRAFT_READY.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/make_tables.py



Inputs: results.parquet and paper_id. Output: paper tables in CSV, Markdown, and LaTeX/DOCX-ready text.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/make_figures.py



Inputs: results.parquet. Output: PNG/PDF heatmaps, line plots, transfer graphs, drift timelines.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/export_cases.py



Inputs: events + metrics. Output: sanitized true positives, false negatives, false positives, representative case studies.



Consumed by M4/M11/M12 and paper-specific plugins.



agentseclab_x/reporting/templates/*.yaml



Per-paper table definitions for P15/P08/P05.



Consumed by M4/M11/M12 and paper-specific plugins.











4.x.3 Detailed development steps



Define universal result row: run_id, paper_id, rq_id, scenario_id, dataset_id, attack_id, model, defense, seed, primary_metric, TSR, ASR, utility, cost, latency, ci_low, ci_high, status.



Implement P15 metrics: static_ASR, adaptive_ASR, adaptive_gain, guard_evasion_rate, composition_gain_loss, feedback_ablation_effect, queries_to_bypass.



Implement P08 metrics: n_clusters, cluster_purity, transfer_T_ij, support_n, bootstrap_CI_width, clique_count, vulnerability_partial_order_edges, replay_success_rate.



Implement P05 metrics: drift_precision, drift_recall, feature_auc, changed_component_F1, first_risky_version_accuracy, scanner_agreement, early_warning_lead.



Generate paper-specific tables: P15 adaptive-vs-static table and composition matrix; P08 transfer heatmap and cluster table; P05 drift-feature ablation and localization table.



Export sanitized case studies: no raw harmful prompt by default; show hashes, categories, and safe summaries.



4.x.4 Acceptance test and paper unblocking effect



Acceptance test: `python -m agentseclab_x.reporting.rq_dashboard --papers P15 P08 P05 --runs runs/` writes dashboards and at least one table per paper using smoke results; with real results it marks RQ1-RQ2 as TABLE_READY.



Paper



Main M12 outputs



P15



Table 1 adaptive vs static ASR; Table 2 defense-composition gain/loss; Figure 1 feedback ablation.



P08



Figure transfer heatmap; Table strategy clusters; Graph safety cliques/partial order.



P05



Table drift-feature ablation; Table changed-component localization; Figure release timeline.











5. Paper-Specific Implementation Plans Using Only the Five Core Modules



The following plans intentionally use M0, M2, M4, M11, and M12 only. Where the full Top-15 plan would use M5/M6/M8/M9/M10, this document implements a minimal paper-local plugin and logs it through M2. After the first seven days, promote the stable plugin into the full shared module.



5.1 P15 GuardBreaker



RQ



Question



Core-module-only implementation



RQ1



How much does adaptive PAIR/TAP/GPTFuzz/AutoDAN/GCG-style search increase ASR over non-adaptive attacks against single defenses?



Use P15 AttackAdapter plugins under M4. Run static template baseline and one adaptive backend with matched query budgets. M12 computes adaptive_gain = adaptive_ASR - static_ASR.



RQ2



Does composing multiple guards monotonically improve security, or can composition create new bypass surfaces?



Define defense_pipeline in configs/defenses.yaml: none, pre_guard, post_guard, pre+post, pre+target-refusal+post. M4 runs the same attacks across pipelines; M12 computes composition_gain_loss.



RQ3



Which feedback signals provide the most leverage to an adaptive attacker?



Stretch with same modules: configure feedback_signal = accept/reject, score, label, latency, target_response. M12 outputs feedback ablation table.











P15 files to create



File



Input



Output / content



papers/P15_GuardBreaker/paper_manifest.yaml



Manual paper metadata



RQs, primary metrics, module list, target venues, owner, dataset/behavior list.



configs/experiments/p15_rq1_adaptive_vs_static.yaml



behavior set, attack backends, defenses, target models, budget



M4 matrix over static vs adaptive runs.



configs/experiments/p15_rq2_composition.yaml



same behavior set, defense pipeline list



M4 matrix over defense compositions.



agentseclab_x/benchmarks/jailbreak/guardbreaker_task.py



M0 config + behavior records + attack adapter



M2 events: AttackCandidateEvent, GuardDecisionEvent, JudgeScoreEvent.



papers/P15_GuardBreaker/plugins/attack_adapters.py



EasyJailbreak/PAIR/GPTFuzz/AutoDAN wrappers



common generate/adapt/reset/cost interface.



papers/P15_GuardBreaker/plugins/guard_pipelines.py



defense names and order



serial pipeline object that emits guard decisions.



papers/P15_GuardBreaker/analysis/p15_metrics.yaml



M12 metric definitions



adaptive_gain, guard_evasion_rate, queries_to_bypass, composition_gain_loss.











P15 7-day steps



Day 1: Create paper manifest, behavior subset, target model list, and defense pipeline list. Use 50-100 safe benchmark behaviors or sanitized IDs; avoid raw dangerous text in release artifacts.



Day 2: Implement GuardBreakerTaskPack and dummy attack backend; confirm M2 logs attack candidates, guard labels, judge scores, and costs.



Day 3: Integrate one real adaptive backend first: PAIR or GPTFuzz. Run RQ1 static vs adaptive on 3 attacks x 3 guards x 2 target models x small budget. Generate P15 Table 1.



Day 3 evening: Run RQ2 composition matrix: no defense, pre-guard, post-guard, pre+post. Generate composition gain/loss table. This is the paper finish line for RQ1-RQ2.



Day 4-5: Add feedback-channel ablation for RQ3 if RQ1/RQ2 are stable: accept/reject only, guard label, guard score, target response.



Day 6: Add sanity checks: same query budget, same seed set, same behavior set, benign utility controls, failure cases.



Day 7: Freeze artifact, create sanitized representative cases, write RQ_STATUS.md, update threat_model.md and limitations.



5.2 P08 CrossJailbreak-1M



RQ



Question



Core-module-only implementation



RQ1



What jailbreak strategy clusters emerge from large-scale organic conversations?



M4 runs an offline corpus pipeline. It loads conversations, filters candidate jailbreak/unsafe records, embeds or uses precomputed embeddings, clusters, emits ClusterEvent rows.



RQ2



Do successful jailbreaks transfer asymmetrically across models, forming safety cliques or vulnerability partial orders?



M4 computes cluster-level directed transfer T[i,j]; M12 produces heatmap, support table, clique table, and partial-order edge table.



RQ3



Can top organic clusters be replay-validated on current models under controlled budgets?



Stretch with same M4 harness: use top 10 clusters and evaluate current target models with sanitized prompts or internal approval-only samples.











P08 files to create



File



Input



Output / content



papers/P08_CrossJailbreak1M/paper_manifest.yaml



paper metadata



RQs, data access assumptions, privacy filtering, artifact policy.



configs/experiments/p08_rq1_clusters.yaml



corpus path, embedding model, clustering params



M4 clustering pipeline config.



configs/experiments/p08_rq2_transfer.yaml



cluster file, compliance labels, model list



M4 transfer matrix pipeline config.



agentseclab_x/benchmarks/jailbreak/organic_transfer_task.py



jailbreak_corpus.parquet or sample CSV



ClusterEvents, TransferEdgeEvents, matrix outputs.



papers/P08_CrossJailbreak1M/plugins/lmsys_loader.py



LMSYS Parquet or local sample



normalized records: conv_id, model, language, prompt_hash, response_hash, moderation flag.



papers/P08_CrossJailbreak1M/plugins/compliance_labeler.py



responses + scorer



compliance_label, confidence, evaluator version.



papers/P08_CrossJailbreak1M/plugins/cluster_transfer.py



embeddings/clusters/compliance labels



cluster assignment table, T matrix, support table, bootstrap CIs.



papers/P08_CrossJailbreak1M/privacy_filtering.md



manual policy



how raw prompts/responses are filtered, hashed, summarized, and excluded from public artifacts.











P08 7-day steps



Day 1: Confirm data path: full LMSYS-Chat-1M, local preprocessed sample, or internal sanitized subset. Create data_access.md documenting what is available.



Day 2: Build lmsys_loader.py and produce a small normalized corpus with conv_id, model, language, harm/moderation flag, prompt_hash, response_hash, and safe summary.



Day 3: Run RQ1 clustering on 10k-50k candidates or the maximum available local sample. Output cluster_assignments.parquet, cluster_summary.csv, and M2 ClusterEvents.



Day 3 evening: Run RQ2 transfer on cluster-level compliance labels. Output transfer_matrix.csv, support_matrix.csv, and bootstrap_CI.csv. This is the RQ1-RQ2 finish line.



Day 4-5: Run replay validation for RQ3 on top 10 clusters if you have model access and approval to use sanitized cluster representatives.



Day 6: Add robustness checks: minimum support thresholds, alternate clustering method, high-confidence-only labels.



Day 7: Package sanitized atlas, matrix, cluster summaries, and artifact README; exclude raw harmful text unless separately approved.



5.3 P05 MCPDrift



RQ



Question



Core-module-only implementation



RQ1



Can semantic, permission, schema, and destination drift predict security-relevant MCP updates better than hash/version changes?



Use a paper-local P05 diff plugin under M4 instead of full M6. It loads snapshot pairs, computes feature rows, compares against labels, and M12 reports precision/recall/AUC.



RQ2



Which drift features best localize the changed component that introduced risk?



Each DiffFeatureEvent includes changed_component. M12 computes localization accuracy/F1 by feature group.



RQ3



Can drift scoring provide early warning before runtime attack success or incident-like behavior?



Stretch: add time/release index and scanner finding time. M12 computes lead time against labeled first-risky version.











P05 files to create



File



Input



Output / content



papers/P05_MCPDrift/paper_manifest.yaml



paper metadata



RQs, repo list, label policy, drift features, baselines.



configs/experiments/p05_rq1_drift_detection.yaml



repo list, version pairs, feature flags



M4 release-drift task config.



configs/experiments/p05_rq2_localization.yaml



diff feature rows, labels



changed-component localization experiment.



agentseclab_x/benchmarks/mcpdrift/release_drift_task.py



snapshot manifest pairs



M2 ReleaseSnapshotEvents, DiffFeatureEvents, DriftLabelEvents.



papers/P05_MCPDrift/plugins/repo_collector.py



GitHub URLs or local repos



snapshot_manifest.jsonl with commit/tag/version IDs.



papers/P05_MCPDrift/plugins/diff_features.py



two snapshots



hash_delta, semantic_delta, schema_delta, permission_delta, endpoint_delta, dependency_delta.



papers/P05_MCPDrift/plugins/scanner_runner.py



snapshot path + scanner config



scanner findings normalized into M2 ScannerFindingEvent.



papers/P05_MCPDrift/release_label_policy.md



manual label guidelines



definition of security_relevant, changed_component, first_risky_version, evidence notes.











P05 7-day steps



Day 1: Select 30-50 MCP servers/repos first. Prioritize repos with visible tags/releases/commit history and readable schemas/manifests.



Day 2: Implement repo_collector.py and create snapshot manifests. If releases are sparse, use commit windows or curated v0/v1 pairs; record this in limitations.



Day 3: Implement diff_features.py with four feature groups: semantic description change, schema broadening, permission/scope change, new external destination. Output diff_features.parquet and M2 DiffFeatureEvents.



Day 3-4: Label 50-150 release/update pairs using release_label_policy.md. Minimum viable P05 needs 50 high-quality labels; strong result needs 150+.



Day 4: Run RQ1 drift detection and RQ2 localization. Generate feature ablation and localization tables. This is the RQ1-RQ2 finish line.



Day 5: Add scanner baseline with Snyk agent-scan and optional Gitleaks/TruffleHog/Semgrep. Compute scanner agreement and early-warning pilot.



Day 6: Add robustness check: benign-looking description/schema evasions and temporal holdout.



Day 7: Package snapshot manifests, label policy, feature tables, scanner outputs, and sanitized examples.



6. Seven-Day Execution Schedule and Finish-Line Mapping



Day



Module focus



P15 finish line



P08 finish line



P05 finish line



Day 1 AM



M0 configs, run IDs, paper manifests



P15 config and run folders initialized.



P08 data access and corpus manifest initialized.



P05 repo list and label ledger initialized.



Day 1 PM



M2 event schema and validators



P15 attack/guard/judge events validate.



P08 dataset/cluster/transfer events validate.



P05 snapshot/diff/scanner events validate.



Day 2 AM



M4 runner and taskpack API



P15 smoke run with dummy attack/guard.



P08 smoke run on small sample.



P05 smoke run on one release pair.



Day 2 PM



M11 artifact factory + M12 reporting skeleton



P15 RQ dashboard exists.



P08 RQ dashboard exists.



P05 RQ dashboard exists.



Day 3



P15/P08 experiments



RQ1-RQ2 table-ready: adaptive vs static and composition.



RQ1-RQ2 table-ready: clusters and transfer matrix.



Feature extraction starts; labels in progress.



Day 4



P05 experiments



RQ3 feedback pilot if stable.



RQ3 replay pilot if data/model access ready.



RQ1-RQ2 table-ready: drift detection and localization.



Day 5



Ablations and baselines



Feedback-channel ablation.



Support threshold/clustering robustness.



Scanner baseline and early-warning pilot.



Day 6



Failure cases and robustness



Representative failure cases and benign utility.



Replay-validation cases and confidence intervals.



Evasive update and temporal holdout.



Day 7



M11/M12 freeze



Artifact + draft sections: threat model, method, results.



Artifact + draft sections: data, method, transfer results.



Artifact + draft sections: data, diff features, results.











7. Concrete Commands to Implement First



# 1. Create the repo skeleton and paper folders



python scripts/init_repo.py --papers P15 P08 P05 --modules M0 M2 M4 M11 M12







# 2. Validate configs



python scripts/validate_configs.py --papers P15 P08 P05







# 3. Run smoke matrix after M4 exists



python -m agentseclab_x.core.matrix_runner --config configs/experiments/smoke_p15_p08_p05.yaml







# 4. Generate RQ dashboard after M12 exists



python -m agentseclab_x.reporting.rq_dashboard --papers P15 P08 P05 --runs runs/







# 5. Generate paper-specific tables



python -m agentseclab_x.reporting.make_tables --paper P15 --runs runs/P15



python -m agentseclab_x.reporting.make_tables --paper P08 --runs runs/P08



python -m agentseclab_x.reporting.make_tables --paper P05 --runs runs/P05







# 6. Package artifacts



python -m agentseclab_x.artifacts.make_artifact --paper P15 --runs runs/P15 --out artifacts/P15.zip



python -m agentseclab_x.artifacts.make_artifact --paper P08 --runs runs/P08 --out artifacts/P08.zip



python -m agentseclab_x.artifacts.make_artifact --paper P05 --runs runs/P05 --out artifacts/P05.zip



8. Appendix: Minimal Schemas



8.1 Experiment config skeleton



paper_id: P15



rq_id: RQ1



experiment_id: p15_rq1_adaptive_vs_static_v0



seed: 1



dataset:



  id: harmbench_subset_v0



  path: data/p15/behaviors_subset.csv



model:



  target: local_or_api_model_name



attacks:



  static: [template_baseline]



  adaptive: [pair_or_gptfuzz]



defenses:



  pipelines: [none, pre_guard, post_guard, pre_post]



budget:



  max_queries: 20



  max_tokens: 12000



outputs:



  run_dir: runs/P15/RQ1/



8.2 Minimal M2 event



{



  "event_id": "evt_000001",



  "run_id": "P15_RQ1_...",



  "paper_id": "P15",



  "rq_id": "RQ1",



  "scenario_id": "behavior_0001",



  "step_id": 3,



  "event_type": "GuardDecisionEvent",



  "timestamp": "2026-08-11T12:00:00-06:00",



  "actor": "guard",



  "source_channel": "model_output",



  "trust_level": 0.5,



  "content_hash": "sha256:...",



  "parent_event_ids": ["evt_000000"],



  "outcome": "blocked_or_allowed",



  "payload": {



    "guard_name": "wildguard_or_llamaguard",



    "label": "safe_or_unsafe",



    "score": 0.84,



    "defense_pipeline": "pre_post"



  }



}



8.3 Universal result row



run_id,paper_id,rq_id,scenario_id,dataset_id,model,attack,defense,seed,primary_metric,TSR,ASR,utility,cost,latency,ci_low,ci_high,status



P15_...,P15,RQ1,behavior_0001,harmbench_subset_v0,target_model,pair,pre_post,1,adaptive_gain,NA,0.42,0.95,1.23,18.4,0.31,0.52,TABLE_READY



9. Final Go/No-Go Criteria



Paper



Green by Day 3/4



Yellow fallback



Red condition



P15



RQ1 and RQ2 tables have real adaptive/static and composition values.



Use fewer attacks/guards but keep matched budgets.



No adaptive backend runs at all.



P08



Cluster table and transfer matrix generated from real or approved internal sample.



Use smaller sample and label as pilot; skip replay.



No data access and no internal sample.



P05



50+ high-quality labeled update pairs and drift/localization tables.



Use 20-50 hand-labeled cases as measurement-protocol pilot.



No release pairs or no labels.











Implementation principle: keep all raw harmful text, canary values, and potentially sensitive repository content out of public artifacts by default. The publishable artifact should contain manifests, hashes, sanitized summaries, code, configs, representative safe traces, and aggregate metrics.







this is the development plann for the paper and modules for the papers. 
