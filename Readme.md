# GitHub Actions for Data Engineering

This repository contains my notes and practice exercises while learning **GitHub Actions**, with a focus on how GitHub Actions can be used in **data engineering projects**.

I already know the basics of Git and GitHub, so this learning path focuses on understanding GitHub Actions from the ground up.

---

## What is GitHub Actions?

GitHub Actions is a CI/CD and automation platform built into GitHub.

It allows us to automatically run commands and workflows when something happens in a repository.

For example:

```text
Git push
   ↓
GitHub Actions starts
   ↓
Checkout repository
   ↓
Set up environment
   ↓
Run commands
   ↓
Tests / checks / build
```

For data engineering, this can eventually be used for:

* Python testing
* SQL testing
* Data-quality checks
* dbt workflows
* Building Docker images
* Deployments
* Scheduled data jobs
* Automation

---

# 1. Workflow Files

GitHub Actions workflows are stored in:

```text
.github/workflows/
```

Example:

```text
project/
│
├── .github/
│   └── workflows/
│       └── home.yml
│
├── hello.py
├── requirements.txt
└── README.md
```

The workflow file can have either a `.yml` or `.yaml` extension.

The filename itself doesn't matter. What matters is that the workflow is inside:

```text
.github/workflows/
```

---

# 2. Basic Workflow Structure

Our first workflow looked like this:

```yaml
name: My First Action

on:
  push:

jobs:
  hello:
    runs-on: ubuntu-latest

    steps:
      - name: Say hello
        run: echo "Hello from GitHub Actions!"
```

The main concepts are:

```text
Workflow
   ↓
Jobs
   ↓
Steps
   ↓
Commands / Actions
```

---

# 3. `name`

The `name` field gives the workflow a name.

```yaml
name: My First Action
```

This is the name shown in the GitHub Actions UI.

---

# 4. `on`

The `on` section defines **when the workflow should run**.

For example:

```yaml
on:
  push:
```

This means the workflow runs when a push occurs.

Other events we learned about include:

```yaml
on:
  pull_request:
```

and:

```yaml
on:
  workflow_dispatch:
```

`workflow_dispatch` allows us to manually start a workflow from GitHub.

---

# 5. Jobs

A workflow contains one or more jobs.

Example:

```yaml
jobs:
  hello:
    runs-on: ubuntu-latest
```

Here, `hello` is the name of the job.

A workflow can contain multiple jobs:

```yaml
jobs:

  test:
    runs-on: ubuntu-latest

  build:
    runs-on: ubuntu-latest
```

Jobs are independent by default and can potentially run in parallel.

---

# 6. Runners

We used:

```yaml
runs-on: ubuntu-latest
```

This tells GitHub to run the job on an Ubuntu runner.

The runner is the machine where our commands are executed.

Conceptually:

```text
GitHub Repository
       ↓
GitHub Actions
       ↓
Ubuntu Runner
       ↓
Execute our workflow
```

The runner is a temporary environment used to execute the job.

---

# 7. Steps

A job contains steps.

Example:

```yaml
steps:

  - name: Step 1
    run: echo "Hello"

  - name: Step 2
    run: echo "World"
```

Steps inside a job run sequentially.

Think of it as:

```text
JOB
 │
 ├── Step 1
 │
 ├── Step 2
 │
 └── Step 3
```

---

# 8. `run`

`run` is used to execute a shell command.

Example:

```yaml
- name: Say hello
  run: echo "Hello from GitHub Actions!"
```

We also used:

```yaml
run: ls -la
```

to inspect the files available on the runner.

---

# 9. `uses`

`uses` allows us to use an existing GitHub Action.

For example:

```yaml
- name: Checkout repository
  uses: actions/checkout@v4
```

and:

```yaml
- name: Set up Python
  uses: actions/setup-python@v5
```

The distinction is:

```text
run
 ↓
Execute a command

uses
 ↓
Use an existing GitHub Action
```

---

# 10. `actions/checkout`

We learned that the GitHub Actions runner is a fresh environment.

Therefore, we need to get our repository's code onto the runner.

We use:

```yaml
- name: Checkout repository
  uses: actions/checkout@v4
```

Conceptually:

```text
GitHub Repository
       ↓
actions/checkout
       ↓
Runner gets repository files
```

After checkout, the runner can access files such as:

```text
hello.py
README.md
requirements.txt
src/
tests/
```

---

# 11. Setting Up Python

We used:

```yaml
- name: Set up Python
  uses: actions/setup-python@v5
  with:
    python-version: '3.12'
```

This tells GitHub Actions to configure Python 3.12 for the job.

The important concept here is `with`.

```yaml
with:
  python-version: '3.12'
```

`with` provides configuration to an Action.

---

# 12. Running Python

We created a simple Python program:

```python
print("Hello from Python!")
print("I am running inside GitHub Actions!")
```

Then our workflow ran it with:

```yaml
- name: Run Python program
  run: python hello.py
```

The workflow therefore became:

```text
Checkout repository
        ↓
Set up Python
        ↓
Run Python program
```

---

# 13. Python Dependencies

We also learned that real Python projects require external packages.

We created:

```text
requirements.txt
```

For example:

```text
requests==2.32.3
```

Then we can install the dependencies in GitHub Actions:

```yaml
- name: Install dependencies
  run: pip install -r requirements.txt
```

The workflow becomes:

```text
Checkout
   ↓
Set up Python
   ↓
Install dependencies
   ↓
Run Python
```

This is an important pattern for data engineering projects.

---

# 14. Jobs vs Steps

This is one of the most important concepts we learned.

A **job** contains multiple steps.

Example:

```yaml
jobs:

  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: pytest
```

There is:

```text
1 Job
 ├── Checkout
 ├── Install dependencies
 └── Run tests
```

Multiple jobs look different:

```yaml
jobs:

  test:
    ...

  build:
    ...
```

which gives:

```text
Workflow
   │
   ├── Job: test
   │
   └── Job: build
```

Jobs can run independently unless we specify dependencies.

---

# 15. `needs`

We learned that jobs are independent by default.

If we want one job to wait for another, we use:

```yaml
needs:
```

Example:

```yaml
jobs:

  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Testing..."

  build:
    needs: test
    runs-on: ubuntu-latest

    steps:
      - run: echo "Building..."
```

This creates:

```text
test
 ↓
build
```

The `build` job waits for `test`.

We can also have multiple dependencies:

```yaml
build:
  needs:
    - python-tests
    - sql-checks
    - lint
```

This creates:

```text
python-tests ──┐
               │
sql-checks ────┼──→ build
               │
lint ──────────┘
```

---

# 16. Data Engineering Connection

The concepts we've learned so far can be applied to a data engineering CI pipeline.

For example:

```text
Pull Request
      ↓
GitHub Actions
      ↓
Checkout repository
      ↓
Set up Python
      ↓
Install dependencies
      ↓
Run Python tests
      ↓
Run SQL checks
      ↓
Run data-quality checks
      ↓
Build
```

Eventually, we can create workflows for:

### Code quality

```text
Python tests
SQL checks
Linting
Formatting
```

### Data quality

```text
NULL checks
Uniqueness checks
Schema checks
Business-rule checks
```

### Build

```text
Build Python package
Build Docker image
```

### Deployment

```text
Deploy pipeline
Deploy dbt project
Deploy Docker image
```

### Automation

```text
Scheduled data jobs
Manual pipeline execution
Notifications
```

---

# 17. Current Knowledge

At this point, I understand the following GitHub Actions concepts:

* [x] What GitHub Actions is
* [x] Workflow files
* [x] `.github/workflows/`
* [x] `name`
* [x] `on`
* [x] `push`
* [x] `pull_request`
* [x] `workflow_dispatch`
* [x] Jobs
* [x] Steps
* [x] Runners
* [x] `runs-on`
* [x] `run`
* [x] `uses`
* [x] `actions/checkout`
* [x] `actions/setup-python`
* [x] `with`
* [x] Installing Python dependencies
* [x] Jobs vs steps
* [x] Multiple jobs
* [x] `needs`
* [x] Basic job dependencies

---

# 18. Next Topics

The next part of the learning path will focus on GitHub Actions concepts that are especially useful for data engineering:

```text
1. Job dependencies
       ↓
2. Environment variables
       ↓
3. Secrets
       ↓
4. Artifacts
       ↓
5. Caching
       ↓
6. Pull Request CI
       ↓
7. Python testing
       ↓
8. SQL / data-quality checks
       ↓
9. Scheduled workflows
       ↓
10. Docker
       ↓
11. CI/CD
       ↓
12. Real data engineering project
```

The eventual goal is to build a realistic workflow rather than just individual examples:

```text
                    Pull Request
                         │
                         ↓
                 GitHub Actions
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
      Python tests    SQL checks    Data quality
          │              │              │
          └──────────────┼──────────────┘
                         ↓
                       Build
                         ↓
                     Deploy
```

This repository will serve as my hands-on learning environment for understanding how GitHub Actions can be applied to real-world data engineering projects.
