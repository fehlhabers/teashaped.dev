---
title:  "The GitOps Journey 2 - Keep Main Sane"
date:   2023-10-25
authors:
  - name: Kaj Fehlhaber
draft: true
---
![Cover Image](cover.jpg)
The first objective of this journey will be to be able to have an infrastructure which supports building applications and deploying these
to different stages. This is the backbone of any GitOps setup.

I'll be focusing on a workflow which:

1. Lints & scans application changes before allowing merge to main
2. Automatically tags changed applications based on commit messages
3. Publishes built container with tag to container registry

Let's go through them one by one!

## 🤯 Keep your main sane!
In a GitOps setting, your main branch is the truth of what is deployed and the goal should be that all versions should be in a deployable
state. In order to realistically achieve this, we will need some kind of protection mechanism for our main.

In Github, this can be achieved by using branch protection. Let's configure our repository with branch protection and some sane configuration.

**Basic Pull Request setting**
![Github PR settings](/assets/images/gitops-example/pr-settings.png)
*PRs are a good way to describe what each version does, since all commits are a squashed PR with information*

This is the configuration I would recommend in order to being able to quickly merge and removing old branches. Only allowing squashing and defaulting the 
message to title and description allows for focusing effort in writing a good title and description and allowing auto merged PRs with wanted result.

**Branch protection settings**
![Require PR](/assets/images/gitops-example/require-pr.png) 
*Enable branch protection and select that PR is required. This is a pre-requisite for this approach.*

![Required checks](/assets/images/gitops-example/required-check.png) 
*We want to add required checks later on, but before we can do this we need to create the workflow first.*

These are sane configurations in order to keep main from receiving changes which could break deployments or introduce vulnerabilities. But now let's
start to create some application and Github actions to get started! Remember these settings for later!

#### ❗ A word about PRs and trunk based development (TBD)
Maybe talking about branch protection will trigger some people since it is synonymous with PR reviews and such insidious workflows. The approach I suggest
is purely TBD, where small updates are done on short lived branches - preferably in mobs or pairs.

PRs act as the vessel to describe the change with minimal effort and where the automated checks which would have been on main, instead are done in the PR.

## Using CodeQL
When running CodeQL, the configuration is stored in Github Advanced Security in a way which requires all runs to be in the same workflow. Because of this, the same workflow
needs to get triggered by both pull request, push (and preferably on schedule to identify new vulnerabilities).

Since I'm setting up a mono repository which has both apps & infrastructure, I'm only triggering the action on the apps directory:
```yaml
on:
  push:
    branches:
      - main
    paths:
      - "apps/**"
    ignore-paths:
      - "**/*.md"

  pull_request:
    branches:
      - main
    paths:
      - "apps/**"
    ignore-paths:
      - "**/*.md"

  schedule:
    - cron: '30 3 * * 0'
```

If any app has been updated, let's run checks for the apps! This can be done in parallel for all apps using the `matrix` strategy.
```yaml
jobs:
  build-scan:
    name: Build & Scan
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false # Want to run other checks even if one fails
      matrix:
        app: [${{ fromJson(vars.APPS) }}] # Read variable from github containing all apps
    permissions:
      actions: read
      contents: read
      security-events: write
    defaults:
      run:
        working-directory: apps/${{ matrix.app }} # All run steps are defaulted to the app directory
```

CodeQL scans can take some time, so make sure to keep the apps small and with few dependencies. It's important to run the scan for only the app working-directory
so that only the changed app is analyzed. This example uses go and here it's a good practice to use the `setup-X` actions since they incorporate caching of dependencies.
```yaml
    steps:
    - uses: actions/checkout@v4

    - uses: github/codeql-action/init@v2
      with:
        languages: go

    - uses: actions/setup-go@v2
      with:
        go-version-file: apps/${{ matrix.app }}/go.mod
        cache-dependency-path: apps/${{ matrix.app }}/go.mod

    - uses: github/codeql-action/autobuild@v2
      with:
        working-directory: apps/${{ matrix.app }}
    - uses: github/codeql-action/analyze@v2
      with:
        category: ${{ matrix.app }}
```

...Now let's recap **required checks for merging**! The way Github works with required checks is that it accepts successful jobs with the **specific name** supplied in the settings.
In my case I used `Pre-checks Passed`. For this particular case it might not look like it's needed - however, when you start having different behavior depending on 
which directory was changed, they all need the same name as the **master**. This example *does not contain linting*, but you likely want this as well.
```yaml
  pre-check:
    name: Pre-checks Passed
    runs-on: ubuntu-latest
    needs: [build-scan]
    if: always()
    steps:
      - run: |
          if [ ${{ contains(needs.*.result, 'failure') }} = 'true' ]; then
            echo "At least one of the checks failed!"
            exit 1
          fi
          echo "Success!"
```
*This is a nifty way to have one action which acts as the master for all other checks*

