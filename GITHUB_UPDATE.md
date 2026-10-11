# Updating the existing GitHub repository

This checkout already points to `https://github.com/regmi-sijan/PM2.5_Study.git`. The preparation includes current and historical analyses, source/merged workbooks, reports and saved results. Virtual environments, caches, temporary workbook export files and execution logs are excluded.

## Review before publishing

From the repository root:

```sh
git remote -v
git status --short
git add --dry-run -- README.md GITHUB_UPDATE.md .gitignore Ana_SK Ana_SR
```

The workbook rename is intentional. `Ana_SK/Meterological data ALAbama .xlsx` becomes `Ana_SK/Meterological_data_ALAbama.xlsx` without changing file contents. The analysis copy and merged workbook are included explicitly by ignore-rule exceptions. `pooled_county_study/results/` is included to preserve this completed experiment; new runs should use separate directories.

## Verify and commit

After activating the pooled-study environment:

```sh
cd Ana_SR/pooled_county_study
python -m unittest -v test_pipeline.py
python verify_results.py
cd ../..
git add -- README.md GITHUB_UPDATE.md .gitignore Ana_SK Ana_SR
git diff --cached --stat
git diff --cached --check
git commit -m "Add reproducible pooled PM2.5 study and documented analyses"
```

Review the staged summary before committing. Saved historical metadata records the original local execution paths; code resolves its inputs relative to the repository and does not require those paths. No credentials or environment files are part of the intended update.

## Push to the existing repository

```sh
git branch --show-current
git fetch origin
git status -sb
git push origin HEAD
```

If the remote has advanced, reconcile its changes before pushing. Do not force-push. GitHub authentication and repository write access are required. This preparation does not itself commit or push changes.

## Complete rerun

To perform another full training run rather than verify saved results, use the pooled README commands with an unused output directory. `--smoke` is only a quick pipeline check; it is not the saved full experiment. Dataset regeneration is described in `data/README.md`.
