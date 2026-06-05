# HMM Discovery — Run Guide

A practical decision guide for every tab. Use this alongside the interactive
[User Guide](../www/guide.html) to understand *what* each control does and
*why* you would choose one option over another.

---

## Before You Start

### System check
Open the **Database Setup** tab immediately after loading the app. Look for the
green banner:

> ✅ Full-run environment is installed.

If it is yellow or red, click **Check Tools** and follow the instructions to
install missing dependencies before doing anything else.

### Choose your run mode

| Mode | When to use | Expected time |
|------|-------------|---------------|
| **Quick environment check** | First time setup, or after updating | 2–5 min |
| **Demo run** | Verify the app works before using your data | 5–10 min |
| **Focused protein search** | You have curated seeds and want to search 1–2 fast DBs | 5–20 min |
| **Full phage discovery run** | Research-grade search across INPHARED + RefSeq + annotation | 30–90 min |
| **Exhaustive all-database run** | Submission-ready; includes GPD, GVD, bacterial proteins | 2–6 hours |

Start with Demo → Focused → Full. Do not jump straight to exhaustive on a
first project.

---

## Database Setup Tab

### What to do here
This tab is for registering databases — you do not run searches here. Searches
happen in Step 4.

### Which databases to enable for your biology

| Research question | Recommended databases |
|-------------------|-----------------------|
| Phage protein family | INPHARED proteins + RefSeq viral proteins |
| Catch unannotated phage genes | INPHARED genomes (six-frame) + RefSeq viral genomes (six-frame) |
| Cross-kingdom host-range check | SwissProt (to confirm hits are viral-specific) |
| Gut-phage diversity | GPD + GVD-AVrC |
| Annotation context only | Pfam (domain scan) + VOGDB VFAM |
| Full phage discovery | All of the above except bacterial proteins |
| Bacterial contaminant check | RefSeq bacterial proteins |

> **Rule of thumb:** Start with INPHARED proteins (~15 sec) as a fast sanity
> check. If you see expected hits there, expand to genomic databases.

### Streaming vs local
- **🟢 Stream** — Data flows live from a remote server directly into `hmmsearch`.
  No disk space required. Requires stable internet.
- **🔵 Local (instant)** — Already on your disk. Searches in seconds.

### Adding a single genome target
Use **Add Custom Database → Single Genome Target** to register a nucleotide
FASTA you want to scan. Enable six-frame ORF mode in Step 4 to catch genes
that annotation tools miss.

### What "full-run environment is installed" means
All required tools (MAFFT, trimAl, hmmbuild, hmmsearch) are found on `PATH`.
Optional tools (IQ-TREE, MEME, CD-HIT) are also shown; the app works without
them but those analysis sub-tabs will be disabled.

---

## Step 1 — Input Sequences

### Accepted formats

| Format | Use when |
|--------|----------|
| `.fasta` / `.fa` / `.faa` | Standard protein or nucleotide FASTA |
| `.fna` | Nucleotide FASTA (triggers ORF prediction) |
| `.gb` / `.gbk` | GenBank flat file; coordinates and annotations preserved |
| `.gz` | Any of the above gzip-compressed |
| Folder | Batch mode — processes all FASTA/GenBank files inside |

### Protein vs nucleotide: how the app decides
The app inspects the sequence alphabet. If >80% of characters are amino-acid
letters (A, D, E, F, G, H, I, K, L, M, N, P, Q, R, S, T, V, W, Y) the file
is treated as protein. Everything else is treated as nucleotide and triggers
ORF prediction.

> **If your sequences are misclassified**, check for unusual characters in
> headers or for very short sequences. Rename the file extension to `.faa`
> (protein) or `.fna` (nucleotide) to force the correct type.

### What to look for in the Input Summary

| Stat | Green flag | Red flag |
|------|-----------|----------|
| Sequence count | 5–500 for a seed set | < 3 (too few to build a robust HMM) |
| Avg length | Matches your expected protein | Very short (< 50 aa) or inconsistent |
| Type detected | Matches what you expect | Wrong type (check alphabet) |
| Duplicates | 0 | > 0 — deduplicate before proceeding |

### When to use batch mode
Use a folder if you have multiple FASTA files from different sources and want
the app to treat them all as one input. The app concatenates and deduplicates
before alignment.

---

## Step 2 — Multiple Sequence Alignment

### Aligner choice

| Aligner | Choose when |
|---------|-------------|
| **MAFFT (recommended)** | Default. Best balance of accuracy and speed for < 500 sequences |
| **Clustal Omega** | Use if MAFFT produces a very gappy alignment on divergent sequences |

MAFFT accuracy mode is set to **auto** by default (the app picks `L-INS-i` for
< 200 sequences, `G-INS-i` for larger sets). Only override if you have a
specific reason.

### When to disable trimAl
trimAl removes poorly conserved alignment columns. Disable it if:
- Your sequences are already well-trimmed
- You want to preserve short terminal extensions for synteny analysis
- The trimmed alignment loses critical residues (check the HMM logo in Step 3)

Default: **automated1** (recommended in most cases).

### Reading the Alignment Quality stats

| Stat | Good | Warning |
|------|------|---------|
| % identity (avg) | 30–85% | < 20%: sequences may be too divergent; > 95%: seeds too similar |
| % gaps (avg) | 0–5% | > 10%: check for outlier sequences or misalignment |
| Alignment length | Close to average sequence length | Much longer → fragmented or misaligned regions |

> **If identity is < 20%**, consider splitting your seed set into sub-families
> and running separate HMMs. One broad HMM will have poor specificity.

---

## Step 3 — Build Profile HMM

### HMM name
Use a short, meaningful name with no spaces: e.g., `gp75_phage`, `tail_fiber_K`,
`RBP_podoviridae`. This name appears in output file headers and methods text.

### What LENG and NSEQ mean
- **LENG** — Profile length (number of match states). Should be close to your
  average aligned sequence length after trimming.
- **NSEQ** — Number of sequences used to build the profile. Should match your
  input count.

### Seed self-recovery: the most important quality check

| Recovery rate | Interpretation |
|---------------|----------------|
| **100%** | Perfect — all seeds score above 45 bits against their own HMM |
| **> 80%** | Good — acceptable for most discovery runs |
| **60–80%** | Borderline — check which seeds failed; they may be outliers |
| **< 60%** | Problem — seeds are too divergent or the alignment is poor. Fix before searching |

> **Bit score range matters too.** If your min score is 50 bits and max is 220
> bits, your seed set spans two distinct families. Consider splitting.

### What the HMM logo shows
Each column height represents information content (bits). Tall columns = highly
conserved positions. Use this to visually confirm the HMM captured the right
conserved core and not just a few identical sequences.

---

## Step 4 — Database Search

This is the most important tab. Read carefully.

### Database selection guide by research question

#### Phage protein family (fast, start here)
- ✅ **INPHARED proteins** (~15 sec) — Fast protein-level check across curated phage proteomes
- ✅ **RefSeq viral proteins** (2–5 min) — RefSeq-annotated viral proteins

If you get expected hits here, expand to genomic databases.

#### Catch unannotated / annotation-missed genes
- ✅ **INPHARED genomes** (8–12 min, six-frame) — All INPHARED phage genomes, translated in six frames
- ✅ **RefSeq viral genomes** (10–30 min, six-frame) — RefSeq viral genomes, six-frame scan

> Use this for short genes, overlapping genes, genes with atypical start codons,
> or genes where standard callers (Prodigal) consistently miss annotations.
> The gp75 family is a good example: 0 protein DB hits, 152 genomic hits.

#### Confirm viral specificity
- ✅ **SwissProt** (1–3 min) — If your family is truly viral-specific, SwissProt should return 0 hits.
  If it returns many hits, you may have a broadly conserved domain.

#### Gut phage / environmental diversity
- ✅ **Gut Phage Database (GPD)** (15–30 min)
- ✅ **GVD-AVrC** (30–90 min) — Large gut/environmental catalogue

Only include these when your biological question explicitly involves gut or
environmental phage diversity.

#### Annotation context (add to any run)
- ✅ **Pfam (domain scan)** — Auto-setup (~5 min once), then seconds per search. Attaches
  known domain family context to each hit.
- ✅ **VOGDB VFAM** — Viral ortholog/family annotation using HMMER. Release 230, 39,585 VFAMs.
  Best way to ask "which viral family does this hit belong to?"

#### Contaminant / host-background check
- ✅ **RefSeq bacterial proteins** (15–30 min) — Use to confirm your hits are not broadly
  bacterial. If you get many bacterial hits, you may have a conserved housekeeping domain.

### ORF scan mode (for nucleotide databases)

| Mode | Use when |
|------|----------|
| **Exhaustive six-frame ORFs** | Looking for short, overlapping, noncanonical, or unannotated genes. Discovery-first. More candidates, more noise. |
| **Prodigal predicted genes** | You want conventional annotations only. Faster, less noise, but misses unusual genes. |

> **Default recommendation:** Use six-frame for discovery runs. Use Prodigal
> only when you already have high-confidence hits and want clean annotation.

### E-value threshold
Default: `1e-5`. For:
- **Distant homolog discovery**: Consider `1e-3` (more sensitive, more noise)
- **Strict specificity**: Use `1e-10` or lower
- **Starting point**: `1e-5` is appropriate for most runs

### What to do if Search Progress shows "No searches started yet"

This happens when the browser session reconnected while HMMER was running in
the background. The search **did not stop** — HMMER runs as a server-side
process regardless of the browser connection.

**What to do:**
1. **Refresh the page** — the app will recover completed databases from disk
2. Check the sidebar: if you see "🔄 Search running — results saving to disk",
   HMMER is still active
3. Go to **Step 7 → Results** — if `hits_main.tsv` exists, the search completed
4. Check project folder: `search_results/*.tblout` files contain raw hits

> **Do not click Run Selected again** — this will start a duplicate run.

### Understanding search duration estimates

| Database | Typical time | Notes |
|----------|-------------|-------|
| INPHARED proteins | ~15 sec | Protein-level, fast |
| SwissProt | 1–3 min | ~570K reviewed proteins |
| RefSeq viral proteins | 2–5 min | ~10M viral proteins |
| INPHARED genomes | 8–15 min | Six-frame of all phage genomes |
| RefSeq viral genomes | 10–30 min | Six-frame of viral genome sequences |
| GPD | 15–30 min | Gut phage diversity |
| GVD-AVrC | 30–90 min | Large environmental catalogue |

Times depend on internet speed (streaming) and CPU count.

---

## Step 5 — Score Calibration

### When to calibrate vs skip
- **Skip** if your seeds are well-characterised and seed self-recovery is 100%
  with clean bit-score separation.
- **Calibrate** when you want to tune the confidence tier thresholds using
  known positive and negative examples.

### Choosing control FASTAs

| Control type | What to use |
|-------------|-------------|
| **Positive controls** | Other confirmed members of your protein family |
| **Negative controls** | Proteins from the same host/background that you expect to score low |

The app ships with several built-in negative control FASTAs (mammalian,
archaea, fungi, plant, eukaryotic viral). Select the one closest to your
expected background.

### Reading the score distribution plot
- **Well-separated**: Positive and negative curves have a clear gap → good HMM
- **Overlapping**: HMM is not specific enough → refine seeds or increase
  sequence diversity
- The vertical thresholds (strict / moderate / loose) can be dragged to adjust
  the confidence tier boundaries

---

## Step 6 — Iterative Refinement

### When iteration helps
Use iteration when:
- Your initial seed set is small (< 10 sequences)
- High-confidence hits from Step 4 are clearly homologous but absent from seeds
- You want to expand the HMM's coverage without manually curating more seeds

### When iteration can hurt
- Do not iterate if your seed self-recovery is already < 80% — iteration will
  make the profile less specific
- Stop when hit count changes by < 5% between rounds (convergence)
- Do not run more than 3–5 iterations without manual curation of the candidates

### Remote homology mode
Enable this (lowers E-value to 1e-2) only when you are specifically hunting
very distant homologs and accept more false positives. Always calibrate
thresholds after a remote-homology run.

### Stopping criteria
Check the Convergence Plot. Stop when:
- Hit count change between rounds is < 5%
- Profile length (LENG) stabilises
- Shannon entropy stops decreasing significantly

---

## Step 7 — Results

### Confidence tiers explained

| Tier | Bit score | HMM coverage | Interpretation |
|------|-----------|-------------|----------------|
| **High-confidence homolog** | ≥ 45 bits | ≥ 60% | Strong evidence of homology; use for main conclusions |
| **Putative homolog** | 25–45 bits | ≥ 40% | Worth manual review; include with caveat |
| **Divergent candidate** | 15–25 bits | Any | Possible distant relative; treat cautiously |
| **Likely false positive** | < 15 bits | < 30% | Discard unless you have independent evidence |

### Filtering strategy
1. Start with **High-confidence** only
2. Check the **Hits per database** bar chart — unexpected DBs with many hits
   may indicate non-specific regions in your HMM
3. Use the **QC flags** to identify problematic hits

### QC flags

| Flag | Meaning | Action |
|------|---------|--------|
| `HIGH_BIAS` | High amino acid composition bias | May be a low-complexity or repeat region |
| `SHORT_ALI` | Alignment covers < 40% of HMM | May be a fragment or partial match |
| `LOW_COMPLEXITY` | Sequence has low-complexity regions | Check with SEG/CAST filter |
| `CONTIG_EDGE` | Hit near contig end | May be truncated; check GenBank |

---

## Step 8 — Analysis

### Recommended order of sub-tabs

1. **Taxonomy** — Run first. Gives you a high-level view of which phyla/orders
   your hits come from. If taxonomy is unexpectedly broad, re-examine your HMM.

2. **Synteny** — Run after taxonomy. Shows genomic neighbourhood conservation.
   Requires internet (NCBI Entrez) or local GenBank files. Provide your NCBI
   email in the settings to avoid rate-limiting.

3. **Phylogenetic tree (IQ-TREE)** — Run only after you have a confident hit set.
   IQ-TREE can take 10–30 min on > 100 sequences. Export the tree in Newick for
   publication-quality visualisation in other tools (FigTree, iTOL).

4. **Presence/absence matrix** — Useful for comparative genomics; shows which
   genomes carry your gene.

5. **Sequence clustering** — Run CD-HIT or MMseqs2 to group redundant hits before
   phylogeny; reduces tree computation time.

6. **Motif discovery (MEME/FIMO)** — Use to find functional sub-sites within your
   hit set. Requires MEME Suite installed separately.

### Synteny limitations
- NCBI accession lookup requires internet access
- Hits from INPHARED / GPD / GVD are not on Entrez; provide local GenBank files
- The Placement Report shows which hits were placed and which were skipped with reasons

### IQ-TREE: when to run and when to skip
- **Run** when you have 20–200 high-confidence hits and want a publishable tree
- **Skip** when you have > 500 hits (too slow) or < 10 hits (tree is uninformative)
- Always cluster with CD-HIT (90% identity) before tree building to remove redundancy

---

## Step 9 — Export

### Always generate Run Summary first
Click **Generate Run Summary** before exporting. This creates:
- `reports/RUN_SUMMARY.md` — Human-readable summary of the full run
- `reports/run_summary.json` — Machine-readable with tool versions and parameters

These are required for **reproducibility** and for **methods sections** in papers.

### Storage Cleanup: preview before you clear
Always click **Preview Cleanup** first. Review what will be deleted.
The cleanup removes raw downloaded database caches but preserves:
- All result tables (TSV, FASTA)
- HMM profile files
- Figures and reports
- Reproducibility JSON

> **Never run cleanup while a search is in progress** — the app blocks this
> automatically if a benchmark PID is alive.

### ZIP export
The ZIP contains everything a collaborator needs to reproduce your analysis:
- Input seeds FASTA
- Alignment files
- HMM profile
- All search result tables
- Figures (PNG, SVG, PDF)
- Methods text paragraph (ready to paste into paper)
- Reproducibility JSON with tool versions, file hashes, and database provenance

### Why reproducibility JSON matters for publications
Reviewers increasingly ask:
- "What exact database version was searched?"
- "What E-value threshold was used?"
- "What tool versions were installed?"

The `reproducibility.json` answers all of these and should be uploaded as
supplementary data alongside your paper.

### What to keep vs clear after export

| Keep | Can clear after export |
|------|----------------------|
| All `.tsv` result tables | Raw downloaded database `.gz` files |
| HMM profile (`.hmm`) | Intermediate tblout files (after results compiled) |
| Alignment files | Temporary ORF translation scripts |
| Figures | Stream cache files |
| Reports and reproducibility JSON | — |
| Input seeds FASTA | — |

---

## Quick Decision Tree

```
Start
  ↓
Check environment (Database Setup) → ✅ tools installed?
  ↓ Yes
Load input (Step 1) → protein FASTA or nucleotide FASTA?
  ↓
Align + build HMM (Steps 2–3) → recovery ≥ 80%?
  ↓ Yes
Pick databases (Step 4):
  → Fast check only?          → INPHARED proteins
  → Genomic discovery?        → INPHARED genomes + RefSeq viral genomes (six-frame)
  → Full research run?        → All viral DBs + annotation (Pfam/VOGDB)
  → Gut phage?                → Add GPD + GVD
  → Confirm specificity?      → Add SwissProt + RefSeq bacterial
  ↓
Run → search progress visible in progress table
  → Disconnected? Refresh → recovered from disk
  ↓
Review results (Step 7) → filter to High-confidence
  ↓
Analyse (Step 8) → taxonomy → synteny → phylogeny
  ↓
Export ZIP + Run Summary (Step 9) → keep reproducibility.json
```

---

*For technical reference (output file formats, tool parameters, command
examples) see [www/guide.html](../www/guide.html).*

*For scientific methodology see [docs/METHODOLOGY.md](METHODOLOGY.md).*
