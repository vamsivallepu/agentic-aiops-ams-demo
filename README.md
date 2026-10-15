# Agentic AI-OPS AMS demo

One Python notebook implementing the six stages in `AI Engineer - Excercise.docx`: historical incident clustering, cluster prioritization, versioned Knowledge Articles, an Enterprise Knowledge Graph, agent enablement, and monitored remediation.

All incidents, approvals, system actions, probes and operational durations are synthetic. The notebook is configured for live **`gemini-3.8-flash`** and contains saved outputs from a successful real API run. Offline mode remains available using deterministic templates and graph policy. Neither mode connects to SAP or Oracle.

## Run

Use Python 3.12. No uv project files are present, so setup uses pip.

macOS/Linux:

```sh
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install -r requirements.txt
python3 -m jupyterlab aiops_ams_demo.ipynb
```

Windows PowerShell:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m jupyterlab aiops_ams_demo.ipynb
```

In Jupyter, choose **Restart Kernel and Run All Cells**. The current configuration uses Gemini and requires a key and network access. Set `MODE = "offline"` for a deterministic run requiring neither after dependency installation.

For an online Jupyter environment such as Colab, select a Python 3.12 runtime, upload the notebook and `requirements.txt`, and install with `%pip install -r requirements.txt` in a separate setup cell. Restart the kernel if requested. Files are written beneath the notebook's working directory; download outputs before an ephemeral runtime ends. Other Python versions have not been verified against these pins.

## Optional Gemini mode

The configuration already selects `MODE = "gemini"` and model `gemini-3.8-flash`. Supply `GEMINI_API_KEY` through the environment or the masked prompt. The client is created once per kernel session; no key is saved in code, outputs or SQLite.

Live calls send incident evidence to Google, use `store=False`, a 30-second timeout and no SDK retries. Stateless Interactions do not override provider data-processing policies. Only use approved synthetic data. Article-generation errors stop explicitly; intake-planning errors create audited escalations without executing remediation. A model error never switches to templates. Matching generated articles are reused; workflow selections remain live.

Gemini writes evidence-grounded RCA summaries and step guidance, then selects an eligible graph-retrieved workflow and explains why. Python independently validates actions, responsible agents, ordering, approvals and recovery limits. The registry supplies tool implementations and hard permission ceilings; execution follows the selected approved graph plan.

References: [Gemini 3.8 Flash](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash), [Interactions API](https://ai.google.dev/gemini-api/docs/interactions-overview), and [structured output](https://ai.google.dev/gemini-api/docs/structured-output).

## Design and review walkthrough

1. Inspect the validated synthetic tickets and silhouette-based cluster selection. Hidden fixture labels are used only for evaluation.
2. Inspect priority components: volume, severity, impact, and tickets per active day, with equal normalized weights.
3. Follow a failed restart into a draft article revision, explicit simulated expert approval, a graph update and a successful rerun.
4. Compare standard and direct-recovery workflow options, approval waits, failed preconditions, recovery exhaustion and unknown-incident escalation.
5. Review separately authored routing cases, outcome metrics, audit events and acceptance checks.

The generator creates **56 historical incidents**: 20 connectivity, 16 storage, 12 account-lockout and eight failed-job incidents. Historical clustering uses RCA/remediation; intake routing excludes those hidden answers. Fixture labels are evaluation-only.

SQLite stores tickets, cluster summaries, immutable article versions, approvals, graph relationships, execution plans, outcomes and audit events. Every displayed intake outcome is persisted. Tool and planner exceptions create sanitized failure events. Unsupported clusters receive manual-only Knowledge Articles and human-investigation workflows.

The main demo has **10 intake cases**: five resolutions, four escalations and one approval wait. Four resolutions are autonomous. The learning demonstration separately records two attempts on the same incident: escalated before revision, resolved after approval. These are synthetic fixture outcomes, not real operational performance.

`OUTPUT_DIR` controls where the notebook stores its database, sample CSV, active articles, priorities, outcomes, routing evaluation, metrics, feedback and graph PNG. The current portable relative path points to the verified live run under `aiops_gemini_live_output/20261015T101624539901Z/`. Identical reruns reuse articles and audited execution results, while Gemini workflow selection remains live. Row ordering is canonicalized. Current invocation timing is measured separately from cached execution timings.

Earlier offline history remains preserved in `aiops_demo_output/`. The separate verified live database contains 56 tickets, four clusters, five article versions, five approvals, 12 runs and 34 audit events. The main outcome CSV reports 10 scenarios; the two learning attempts are exported separately. Use a fresh `OUTPUT_DIR` for a clean demonstration. Kernel resets alone do not erase history.

Set `INPUT_CSV` to use a local file or locally synced Google Drive/OneDrive file. The required columns and categorical values are documented in the notebook. Changed input requires a new `OUTPUT_DIR`; reusing a database with different evidence is rejected before its ticket snapshot changes. Cluster IDs are stable for identical input, not across changed datasets. The included tests always cover generic contracts; bundled-fixture scenario assertions are explicitly skipped for custom CSVs.

## Verification and limitations

The notebook includes isolated checks for malformed inputs, small clustering inputs, model JSON/selection validation, immutable articles, reordered evidence, graph integrity, approvals, planner/tool exceptions, recovery authorization and complete audit persistence.

Verified on Linux with Python 3.12.3:

- All notebook cells executed in fresh IPython processes; **63 acceptance checks passed**. Identical reruns preserved article, approval, run and event counts.
- All **six separately authored routing cases** passed: four paraphrases and two near-misses. This small evaluation is not a general accuracy claim.
- Known-procedure and manual-only custom CSVs ran successfully. Manual-only input produced four articles and persisted all 10 displayed outcomes.
- The actual Gemini SDK passed mocked wire-format/session-reuse checks. The full Gemini-mode workflow also passed with mocked provider responses. These tests do not establish live model quality or connectivity.
- Notebook structure, stored tables, and graph output were checked. The graph PNG was visually inspected.

**Successful real Gemini run, October 15, 2026:** after network access was enabled, the complete notebook passed with **14 real `gemini-3.8-flash` requests and all 63 acceptance checks**, taking **76.65 seconds**. Four requests generated Knowledge Articles and 10 selected workflows. Gemini selected `direct_recovery` for the prior-restart-failure case. Main outcomes were five resolved, four escalated and one awaiting approval; the learning case changed from escalated to resolved after its approved revision. All article versions record Gemini/model provenance. SQLite integrity and foreign keys passed, and the credential was absent from saved artifacts.

Evidence: [live verification report](aiops_gemini_live_output/20261015T101624539901Z/live_verification.json) and [exact executed notebook](aiops_gemini_live_output/20261015T101624539901Z/aiops_ams_demo_gemini.ipynb). The primary notebook contains these live outputs with an equivalent portable relative output path. Earlier DNS failures were caused by the restricted execution environment and have been superseded by this successful run.

Standard Linux Jupyter kernel execution was also retried successfully through `nbclient` in **offline mode**, passing all 63 checks. The real Gemini run used in-process IPython. Mac, Windows, hosted Jupyter and clean installation on those targets remain unverified. A pre-existing Matplotlib warning concerns unavailable 3D plotting; the notebook's 2D graph works.

This is an interview MVP: human approvals and system probes are simulated, routing requires known signals, and the fixture is deliberately separable. No real incident reduction or MTTR improvement is claimed. Production would require real integrations, expert authorization, stable cluster lineage, migration strategy, monitoring, and evaluation against operational data.
