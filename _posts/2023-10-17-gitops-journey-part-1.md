---
layout: post
title:  "The GitOps Journey - part 1 - Building containers"
date:   2023-10-17
cover_image: gitops-example/required-check.png
author: Kaj Fehlhaber
---
In the past couple of months, I've been part a journey which led us to incrementally automate more and more of a service deployed in Kubernetes.
During this period it became clear that there is a lot of talk about GitOps, but not very many hands-on, concrete, blogs and examples on the matter.

This is my attempt at contributing this area.

This will be the first part of a series of blog posts which I will be publishing where I will be building examples of real world CI/CD tooling.
The posts will follow the process of iteratively adding functionality to the pipeline. Starting with this post with a simple setup without any
versioning of Infrastructure-as-Code (IaC) or being able to handle multiple stages.

**Want to get to the code right away?** Check out the accompanied [Github repository](https://github.com/fehlhabers/gitops-example)! 👈

## GitOps - what is it?
In short GitOps basically means that everything that you see in the trunk of a repository is also what is deployed.
It's much like Kubernetes works; you specify a *desired state* and the system tries to reach this state by deploying or tearing down resources.

There are some different definitions regarding GitOps, but it usually boils down to:

📝 Infrastructure-as-Code

🔍 Pull Requests to deploy changes

🚀 Highly evolved CI/CD

And it makes perfect sense, since you want these parts to work in tandem in order to give the result you want: To deploy your service by simply
merging a pull request, automatically having it scanned for vulnerabilities and having an audit trail out-of-the box.

Unfortunately, there is no one-size fits all solution out there, so setting up a repository or multiple repositories to enable GitOps is not trivial.
Different technologies need different approaches, where CI/CD pipelines are built in a way to make a repository act as a GitOps repository.
Let's dive into an example with some concrete technologies and the challenges and solutions to them!

## Goal of the journey
The journey of this blog post series will take us through a couple of concepts and technologies. I will be using some popular tools and technologies when it comes to cloud computing,
but these are by no means the only or best way for you. They are a mix of technologies I want to explore and technologies I'm familiar with.

The setup consists of:

- *Containers* for applications
- *CodeQL* & *Github Advanced Security* for vulnerability scanning
- *Helm* charts to package *Kubernetes* resources
- *ArgoCD* for deploying our Helm charts
- Either *Crossplane* or *Terraform/OpenTofu* for cloud-specific resources
- *Github Actions* as the CI/CD base which glues all together
- All of this in a way which supports *multiple stages* and zero-downtime

This can become a quite daunting task in a complex environment, so we'll split it up in this series in order to focus on *one thing at a time*.
The focus area of the first part is therefore to just be able to create the *base flow for building containers!*

# Let's start! 
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

## 📦 The container
I've chosen to create a very simple Golang application to be the example at hand. See [the gitops-example repo](https://github.com/fehlhabers/gitops-example/apps/cruncher) for full info.
However, this can be done in any language as long as it can be containerized. A strength of Golang is that the app can be built into a single binary
which can be added to `scratch`, eliminating pulling in any potential vulnerabilities in a runtime. In this case, the container weighs in at only **9MB**.


**Dockerfile using the builder pattern** 
```Dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.21-alpine AS builder

WORKDIR /src
COPY . .
RUN go mod download
RUN go build -o /bin/app .
RUN useradd -u 10001 appuser

##########################################################
FROM scratch
COPY --from=builder /bin/app app
COPY --from=builder /etc/passwd /etc/passwd
EXPOSE 8080
CMD ["./app"]
```
*The builder has the needed parts to build the binary and copies the app & the user info in order to run as non-root*

Now that the application has all the needed setup to be a running container in a secure way, it's time to build a pipeline which can handle many of these applications.

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

<br>
<br>
<br>
![](/assets/images/kaj.jpg){:style="margin-left: 0px;margin-right: auto;"}
**Kaj Fehlhaber**<br>
Freelancing Software Consultant<br>
_Accelerating the DevOps Journey of Teams!_
