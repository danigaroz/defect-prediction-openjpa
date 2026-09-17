# Defect Prediction on Apache OpenJPA

[![Language](https://img.shields.io/badge/language-Python%20%2B%20Java-blue)]()
[![Course](https://img.shields.io/badge/institution-Tor%20Vergata%20Rome-red)]()

Can we predict which Java classes will contain bugs — before they do? A four-milestone empirical study applying walk-forward defect prediction, counterfactual analysis, LLM-assisted refactoring and mutation testing to Apache OpenJPA, a real-world open-source JPA implementation with 13 releases and 10,290 labelled instances.

**Author:** Daniel Garoz Vazquez (Erasmus exchange) · **University:** Tor Vergata, Rome

> This repository is a fork of [apache/openjpa](https://github.com/apache/openjpa) used as the case-study subject. All work described below is the author's contribution; the rest of the codebase is the original Apache OpenJPA project.

---

## Project overview

| Milestone | Topic | Headline result |
|---|---|---|
| M1 | Dataset construction | 13-release dataset, 10,290 rows, 18.6% buggy ratio, JIRA-based labelling + Proportion estimation (P median = 1.5) |
| M2 | Walk-forward defect prediction | Best classifier: Random Forest + SMOTE, **AUC = 0.897** (12 walk-forward iterations, 24 configurations) |
| M3 | What-If counterfactual analysis | 1.5% of predicted buggy classes preventable by setting NSmells to zero |
| M4 | LLM-assisted refactoring + mutation testing | Mutation score 13% → 25% → 29% across three iterations; Black-Box tests give 4× code retention improvement on the complex class |

## Methodology

| Step | Technique |
|---|---|
| Bug labelling | JIRA REST API + Proportion (P median) estimation for missing AV |
| Features | PMD code smells, git process metrics, CK object-oriented metrics |
| Classification | Random Forest, Naive Bayes, IBk — walk-forward temporal validation |
| Class imbalance | SMOTE oversampling |
| Counterfactual | What-If analysis on predicted buggy classes (M3) |
| Refactoring | LLM-assisted (M4), Black-Box, and Randoop random test suites |
| Mutation testing | PIT (PITest) — HTML reports uploaded as CI artifacts |
| CI/CD | GitHub Actions — compiles, runs JUnit, executes PIT on every push to `m4_mutation/` |

## Tech stack

- **Languages:** Python (pipeline, analysis), Java (OpenJPA codebase, JUnit tests)
- **ML:** Weka (walk-forward pipeline), pandas
- **Static analysis:** PMD (code smells), CK metrics
- **Testing:** JUnit 5, Randoop, PIT mutation testing
- **CI/CD:** GitHub Actions
- **Data:** JIRA REST API, git log

## Repository structure (author's contribution)

```text
├── tools/GetVersionsFromJIRA/    JIRA REST API — release retrieval (M1)
├── tools/GetTicketsID/           JIRA REST API — bug ticket retrieval (M1)
├── tools/M2WekaPipeline/         Walk-forward classifier evaluation (M2) + What-If analysis (M3)
├── m4/                           LLM-assisted refactoring experiments + test suites
├── m4_mutation/                  PIT mutation testing, Randoop tests, Nelson reliability estimation
├── .github/workflows/            CI pipeline (M4 mutation testing)
├── dataset_labeled.csv           Final labelled dataset (10,290 rows × 66 columns)
├── smells_per_class.csv          PMD smells aggregated by class
├── commit_metrics.csv            Process metrics from git log
├── ticket_av.csv / ticket_files.csv   Bug ticket linkage and AV resolution
├── m2_results.csv                Walk-forward results (288 rows = 12 iterations × 24 configs)
└── A.csv / B.csv / Bplus.csv / C.csv  Dataset variants for M3 What-If analysis
```

## Note on the use of AI

LLM-assisted refactoring is part of the study itself (M4). All pipeline design, experimental methodology, interpretation of results and written content are the author's own work.

## Context

Università degli Studi di Roma Tor Vergata, A.A. 2025–26 (Erasmus exchange from URJC Madrid).
