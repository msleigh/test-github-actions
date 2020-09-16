# test-github-actions

Learn/test GitHub Actions for running CI/CD

## Setting up GitHub Actions

To start, add a directory at `.github/workflows`. GitHub will automatically
detect this.

This directory is populated with YAML (`*.yml`) file that define different
workflows that have been automated.

## Workflow YAML file syntax

A basic workflow definition file example:

```yaml
name: build

on:
  pull_request:
    branch:
      - master
  push:
    branch:
      - master

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'
      - name: Install dependencies
        run:
          python -m pip install --upgrade pip
          pip install black hypothesis pytest
      - name: Run tests
        run:
          python -m pytest
      - name: Check syntax
        run:
          black --check .
```

The `on:` block defines what actions will trigger this workflow; in this
example the workflow will be triggered either by a push or a pull request to
the `master` branch.

The `jobs:` block defines the workflow. Here we have one job called `ci`, which
is defined to run on the latest version of Ubuntu, and which consists of 4
steps (which are run sequentially). Each step is given a (human-readable) name,
and a sequence of commands to run. The first two steps set up the Python
environment in which to run the tests, and then the final two steps use PyTest
to run the provided unit tests, and Black to check the Python formatting,
respectively.

The `uses:` lines bring in preset GitHub Actions; first to check out the
repository, which is defined at a level so that it is applied across all steps;
and, specifically in the 'Set up Python' step, a pre-defined Action is used to
make Python available.

## CI Matrix

Setting up a matrix allows you to run a job in parallel in multiple different
environments, e.g. operating systems, dependency versions, etc.

For example, rather than specifying a single OS via:

```yaml
ci:
  runs-on: ubuntu-latest
```

the 'matrix' Strategy can be defined like so:

```yaml
ci:
  strategy:
    matrix:
      os: [ubuntu-latest, macos-latest]
  runs-on: ${{ matrix.os }}
```

so that the `ci` Job is run on each of the two specified operating systems.


## GitHub Secrets

GitHub allows you to create 'Secrets' (as long as your are an admin of the
repository).

On the menu bar of the repository, click 'Settings' and then click 'Secrets' in
the left pane.

GitHub's docs say:

> Secrets are environment variables that are encrypted and only exposed to
> selected actions. Anyone with collaborator access to this repository can use
> these secrets in a workflow.
>
> Secrets are not passed to workflows that are triggered by a pull request from
> a fork.

## Conditional jobs


## Caveats


- An individual job can be up to 6h long
- A workflow can be up to 72h long
- A matrix can contain up to 256 'columns'

## Creating Actions to share

A pre-made GitHub Action is just a Git repo. You can create your own generic
GitHub actions and publish them for others to use; all you need to do is define
an `action.yml` file in the root of the repository.

```yaml
nme: "Generic Action"
description: "A generi"
branding:
  icon:
  color: "gray-dark"
inputs:
  dependency-version:
    description: "The version of <dependency> to install"
    required: true
    default: "1.0"
runs:
  using: "??"
  steps:
    - run: |
        pip install <dependency>==${{ inputs.<dependency>-version }}
      shell: bash
```

To publish the Action, create a Release in the repository.

Other people can use this by adding:

```yaml
- uses: <GITHUB_USER>/<REPO_NAME>@master
```

## 
