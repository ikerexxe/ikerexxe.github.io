---
layout:     post
title:      "Github Actions first impressions"
date:       2022-04-12 12:00:00 +0100
categories: ci
background: '/assets/figures/2022-04-12-github-actions.png'
---

# Introduction

Lately I've been working on including several Github Actions in [SSSD](https://github.com/SSSD/sssd)'s CI workflow and I would like to share my impressions.

This tool facilitates the automation of all the workflows of a given repository. From CI/CD, to issue management, to automatic PR merging. Everything you can think of that can be automated. Each workflow can use commands, scripts or, better yet, already defined actions, which can be reused in several workflows. These actions are available in the [GitHub Marketplace](https://github.com/marketplace?type=actions).

# Create a workflow

Creating a workflow is quite easy. It's written in a yaml file under the `.github/workflows/` directory. The most important things to define are  the name of the workflow, the trigger (PR, push, schedule, etc.), the runner (Ubuntu, Windows or MacOS) and the steps (commands, scripts, actions...).

# GitHub Marketplace

Undoubtedly the best feature of Github Actions is the Marketplace. Github already provides several actions such as the [CodeQL static analyzer](https://codeql.github.com/), but the power comes from the community. Several thousand actions are already available in the Marketplace and I don't doubt that there are more to come. My team is using several of these actions to perform static analysis of the code. I encourage you to check the market to see what's out there and select the actions that suit you best.

# CodeQL workflow example

Name the worflow and indicate the triggers.

```
name: "Static code analysis"
on:
  push:
    branches: [master]
  pull_request:
    branches: [master]
  schedule:
    # Everyday at midnight
    - cron: '0 0 * * *'
```

Define a name for the CodeQL analyzer, indicate the runner and the necessary permissions.

```
jobs:
  codeql:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
```

Define the steps for the workflow, namely: checkout the repository, install the dependencies, configure and build the project and finally perform the analysis.

```
    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Install dependencies
      id: dependencies
      uses: ./.github/actions/install-dependencies

    - name: Initialize CodeQL
      uses: github/codeql-action/init@v1
      with:
        languages: cpp, python
        queries: +security-and-quality

    - name: Configure sssd
      uses: ./.github/actions/configure

    - name: Build sssd
      working-directory: x86_64
      run: |
        PROCESSORS=$(/usr/bin/getconf _NPROCESSORS_ONLN)
        make -j$PROCESSORS

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v1
```

If you want to take a deeper look at it the complete workflow can be found [here](https://github.com/SSSD/sssd/blob/master/.github/workflows/static-code-analysis.yml).

# Conclusion

The power comes from the community and this tool isn't an exception. While the tool is currently good, the more actions that are added to the marketplace the more traction it will get. Which, will lead to even more people using it. I think we are at an early stage but Github Actions has a lot of future. I clearly recommend its use.
