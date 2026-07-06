# Feature Ideas

A shortlist of proposed, additive enhancements for HMM Discovery. These are
**design notes only** — nothing here is implemented yet. Each idea is scoped to
build on existing modules (`core/`, `ui/`, `databases/`) without changing the
current guided workflow or its outputs. Contributions and feedback welcome.

## 1. Searchable run history

**What:** Extend the existing "recent projects" list (`core/sessions.py`,
`load_recents`) into a fuller run-history panel: for each completed run, record
the seed family name, database(s) searched, hit count, and finish time, and let
users filter/sort past runs from the sidebar.

**Why:** Researchers often run the same family against several databases over
days; a compact history makes it easy to reopen the right project and compare
outcomes.

**Sketch:** Persist a small JSONL index next to the current recents file, append
one record on run completion, and render it in `ui/step_00_setup.py`. Purely
additive to the existing recents mechanism.

## 2. Parameter presets

**What:** Named, savable presets for the search/build parameters (e.g. HMMER
E-value thresholds, iteration count, trimming options) so a user can pick
"Sensitive", "Balanced", or "Strict" — or save their own — instead of setting
each control by hand.

**Why:** Lowers the barrier for the no-code audience and improves
reproducibility by capturing a known-good configuration.

**Sketch:** Store presets as small JSON blobs; add a preset dropdown to the
relevant step UI that populates the existing inputs. No new pipeline logic —
presets only set values the app already accepts.

## 3. Batch input (multiple seed families in one session)

**What:** Accept several seed FASTA files (or a folder of them) and queue one
discovery run per family, reusing the existing `AsyncJobRunner`.

**Why:** Users studying a panel of related families currently repeat the whole
wizard per family; batching removes that repetition.

**Sketch:** Add an optional "batch mode" on the input step that iterates the
existing single-family pipeline per file and writes each result to its own
project directory. Single-family behavior stays the default and unchanged.

## 4. Shareable result links / bundles

**What:** A one-click "export shareable bundle" that packages the report,
figures, and reproducibility metadata into a single self-contained archive (and,
where a deployment is served over HTTP, a stable link to a read-only results
page).

**Why:** Makes it easy to hand results to a collaborator or attach them to a
manuscript without sharing the whole project directory.

**Sketch:** Build on the existing export step (`ui/step_09_export.py`) to zip the
already-generated artifacts. The link form is only relevant for hosted
deployments and can degrade gracefully to "download bundle" locally.

## 5. Run comparison view

**What:** Select two past runs and show a side-by-side diff of hit counts,
shared vs. unique hits, and key parameters.

**Why:** When tuning parameters or comparing databases, a direct comparison is
far clearer than reopening each run separately.

**Sketch:** Reads the per-run summaries already produced by `core/run_summary.py`
and renders a comparison table/plot in a new analysis sub-tab. Read-only over
existing outputs — no changes to how runs are produced.
