---
name: run-ast-grep-rules
description: Run the ast-grep Java rules from this repo against a codebase and report the findings. Use when asked to run these rules, scan or lint a Java repo with ast-grep, audit a repo for the JPA/performance/Java-hygiene rules, or produce an ast-grep violation report.
---

# Run the ast-grep rules

Run this repo's Java rules over a target repo, let the user pick which categories to run, and finish with a minimal findings report.

Rules live under three categories: `jpa/`, `performance/`, `java/`.

## Step 1: Resolve the rules

Completion criterion: you hold `RULES_ROOT` (the directory holding the category subdirectories) and `SG` (the ast-grep binary), or you have handed off to the user for a missing path.

Resolve `RULES_ROOT` by the first rule that hits:

1. `sgconfig.yml` at the repo root whose `ruleDirs` point at a directory containing `jpa/`, `performance/`, and `java/`. Use that directory.
2. A `rules/` directory found by walking up from the target repo root, containing those three subdirectories.
3. The rules bundled with this repo: the `rules/` directory of the checkout that contains `.agents/skills/run-ast-grep-rules/` — three levels up from this file.

If none of the three holds the category dirs, ask the user where the rules live and stop.

Confirm the CLI with `ast-grep --version` (fall back to `sg --version`). If it is missing, tell the user to install `@ast-grep/cli` (`bun install -g @ast-grep/cli`, `npm install -g @ast-grep/cli`, or Homebrew) and stop.

## Step 2: Ask which categories to run

Ask the user which categories to run with a multiple-choice question, with all categories as the default:

- `All categories (Recommended)`
- `JPA / persistence`
- `Performance`
- `Java hygiene & correctness`

Allow multiple selections. Treat `All categories`, an empty selection, or a custom answer naming all of them as every category in `RULES_ROOT`. Map the labels to the directory names `jpa`, `performance`, `java`.

Completion criterion: you have the concrete category list to scan.

## Step 3: Scan

Write a temporary sgconfig whose `ruleDirs` list the absolute paths of the selected category directories, for example:

```yaml
ruleDirs:
  - /path/to/rules/jpa
  - /path/to/rules/performance
```

Then run the scan against the target (default `.`), asking for JSON so the report can be counted:

```bash
ast-grep scan --config "$TMP_CONFIG" --json --include-metadata "$TARGET"
```

- If the repo defines its own `sgconfig.yml`, prefer `ast-grep scan --filter` with the selected rule ids, or fall back to the temporary config above; never edit the repo's config.
- Count findings with `jq` (`. | length`, and group by `.ruleId`). If `jq` is unavailable, rerun with `--report-style short` instead of `--json` (the two flags conflict) and read the lines directly.
- A non-zero exit from `ast-grep scan` means findings were reported, not that the run failed. Only treat a parse/config error as failure.

Completion criterion: ast-grep ran to completion and you have the findings (possibly empty).

## Step 4: Report

Completion criterion: the user sees one minimal report, or an explicit `No findings.`

Keep it to a short block: the totals, the counts by rule, and one command to reproduce. For example:

```
ast-grep: 5 findings (5 warnings) in 1 file — categories: jpa, performance
  no-save-in-loop            1
  parameterized-logging      3
  no-simpledateformat-in-loop 1
Reproduce: ast-grep scan --config /tmp/sg-rules.yml .
```

When nothing fired: `No findings for categories: <list>.`

Do not paste raw ast-grep diagnostics into the report; offer the reproduce command for detail instead.
