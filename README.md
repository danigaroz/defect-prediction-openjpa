# Defect Prediction on Apache OpenJPA

[![Language](https://img.shields.io/badge/language-Java%20%2B%20Python-blue)]()
[![Institution](https://img.shields.io/badge/institution-Tor%20Vergata%20Rome-red)]()

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

- **Languages:** Java (pipeline, JUnit tests, Weka integration), Python (report plots)
- **ML:** Weka 3.8+ (walk-forward pipeline via `M2WekaPipeline`)
- **Static analysis:** PMD 6/7 (code smells), CK tool (OO metrics)
- **Testing:** JUnit 5, Randoop, PIT mutation testing
- **Build:** Maven 3.6+
- **CI/CD:** GitHub Actions
- **Data:** JIRA REST API (anonymous), git log

## Repository structure (author's contribution)

```text
├── tools/
│   ├── GetVersionsFromJIRA/    JIRA REST API — release retrieval (M1)
│   ├── GetTicketsID/           JIRA REST API — bug ticket retrieval (M1)
│   └── M2WekaPipeline/         WalkForward.java + WhatIfAnalysis.java (M2, M3)
├── m4/                         LLM-assisted refactoring experiments + test suites (BB, LLM, CF)
├── m4_mutation/                PIT mutation testing, Randoop tests, Nelson reliability estimation
├── .github/workflows/          CI pipeline — compiles, tests, runs PIT on every push
├── dataset_labeled.csv         Final labelled dataset (10,290 rows × 66 columns)
├── smells_per_class.csv        PMD smells aggregated by class
├── commit_metrics.csv          Process metrics from git log
├── ticket_av.csv               Bug ticket linkage and AV resolution
├── ticket_files.csv            Files touched per bug ticket
├── m2_results.csv              Walk-forward results (288 rows = 12 iterations × 24 configs)
└── A.csv / B.csv / Bplus.csv / C.csv   Dataset variants for M3 What-If analysis
```

## How to reproduce

### Prerequisites

| Tool | Version | Purpose |
|---|---|---|
| JDK | 11 | All Java compilation and execution |
| Maven | 3.6+ | Build system for tools and M4 |
| Git | any | Process metrics extraction (`git log`) |
| PMD | 6.x / 7.x | Code smell extraction per class per release |
| CK tool | latest jar | Chidamber & Kemerer OO metrics |
| Weka | 3.8+ jar | Walk-forward ML pipeline |

### M1 — Dataset construction

```bash
# 1. Retrieve releases from JIRA
cd tools/GetVersionsFromJIRA
mvn package
java -jar target/GetVersionsFromJIRA.jar

# 2. Retrieve bug tickets
cd ../GetTicketsID
mvn package
java -jar target/GetTicketsID.jar

# 3. Extract process metrics
git log --all --name-only > commit_metrics.csv

# 4. Extract code smells (run per release)
pmd check -d <src_dir> -R rulesets/java/design.xml -f csv > smells_per_class.csv

# 5. Extract OO metrics (run per release)
java -jar ck.jar <src_dir>
```

### M2 — Walk-forward defect prediction

```bash
cd tools/M2WekaPipeline
mvn compile
mvn exec:java -Dexec.mainClass="WalkForward"
# Results written to m2_results.csv
```

### M3 — What-If counterfactual analysis

```bash
# From the same directory
mvn exec:java -Dexec.mainClass="WhatIfAnalysis"
```

### M4 — Mutation testing (local)

```bash
cd m4_mutation
mvn clean compile
mvn test
mvn org.pitest:pitest-maven:mutationCoverage
# HTML report: target/pit-reports/<timestamp>/index.html
```

For Randoop test generation (iteration 3):

```bash
java -classpath randoop.jar randoop.main.Main gentests \
  --testclass=Math --time-limit=60
# Move generated tests to src/test/java/ and re-run mvn test + mutationCoverage
```

### M4 — CI/CD (automatic)

Every push to `m4_mutation/` triggers the GitHub Actions workflow, which runs the full M4 pipeline and uploads the PIT HTML report as a downloadable artifact.

## Note on the use of AI

LLM-assisted refactoring is part of the study itself (M4). All pipeline design, experimental methodology, interpretation of results and written content are the author's own work.

## Context

Università degli Studi di Roma Tor Vergata, A.A. 2025–26 (Erasmus exchange from URJC Madrid).
