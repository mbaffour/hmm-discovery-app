# HMM Discovery Deployment Guide

HMM Discovery is a no-code Shiny-for-Python app for profile-HMM protein-family discovery, database search, hit interpretation, synteny analysis, and reproducible export.

Project website: <https://mbaffour.github.io/hmm-discovery/>

Repository: <https://github.com/mbaffour/hmm-discovery>

## Local Conda/Mamba Deployment

```bash
git clone https://github.com/mbaffour/hmm-discovery.git
cd hmm-discovery
conda env create -f environment.yml
conda activate hmm-discovery
./run_app.sh
```

Open <http://127.0.0.1:8081>.

The app is designed so ordinary users do not need to write code after launch. They can create projects, load FASTA/GenBank files, build HMMs, select databases, run single-genome scans, use exhaustive six-frame ORFs, summarize runs, clear caches, choose export folders, and package results from the interface.

## Docker Deployment

```bash
docker build -t hmm-discovery .
docker run --rm -p 8081:8081 hmm-discovery
```

For shared machines, mount a persistent project directory:

```bash
docker run --rm -p 8081:8081 \
  -v "$PWD/projects:/app/projects" \
  hmm-discovery
```

## Website And Blog Deployment

The public website, blog-style landing page, interactive guide, and presentation links live under `www/`.

GitHub Pages is deployed automatically by `.github/workflows/pages.yml` whenever `main` is pushed. The workflow publishes `www/` to:

<https://mbaffour.github.io/hmm-discovery/>

## Deployment Checks

Before publishing a new release:

```bash
python3 -m compileall -q app.py core ui pipeline databases scripts
python3 scripts/run_all_database_benchmark.py --dry-run --preset all --out /tmp/hmm-discovery-dryrun
```

Expected dry-run registry:

- INPHARED genomes
- INPHARED proteins
- SwissProt
- RefSeq viral proteins
- RefSeq viral genomes
- Gut Phage Database (GPD)
- GVD-AVrC
- RefSeq bacterial proteins
- Pfam sequences
- Pfam domain scan
- VOGDB VFAM annotation

RefSeq bacterial proteins should expand to the current NCBI chunk count recorded by the dry run.

## Data Hygiene

Do not commit:

- private seed FASTA files
- unpublished genomes
- downloaded database caches
- project result folders
- logs, PID files, HMM build outputs, tblout/domtblout files, or export ZIPs

The repository includes only synthetic demo data and public documentation assets.

## Production Notes

For institutional deployment, run the app behind authentication and a reverse proxy, mount a persistent project volume, and keep large database caches outside the Git repository. Each run should export `METHODS_TEXT.txt`, `reproducibility.json`, `RUN_SUMMARY.md`, database provenance tables, and the result ZIP for auditability.
