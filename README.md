# The One-Branch Git Workflow

Wnat if we could have a single git branch for our workflow and tag commits that we want to mark as releases?
What if we could have a clearer git history that is human readable and easy to track down releases?
What if all this reduced the amount of steps in the CI pipeline as well?

## Features

1. Git version tagging must be used to reference all commits that are release candidates. Git works very different than mercurial and SVN in this way. A branch can contain a release and simultaneously contain commits not in that release. This makes the need for maintaining multiple branches redundant, which leads to feature 2.
2. The development branch is the **only branch**. It is the master/default/source-of-all-truth branch. This eliminates trying to keep 3+ separate branches (codebases) in sync with each other when you want to introduce production deployment and a deployable release staging area.

## Examples

Let's walk through scenarios for staging release candidates, making releases, and applying hotfixes.
You can follow along with these examples by creating a new folder and following the commands exactly as shown.

### Scenario 1 - Staging a release

Before a release goes out, you may want to create a staging area where a release candidate can be generated.

```bash
# Create repo
git init one-branch-git-workflow 
cd one-branch-git-workflow

# Git created a "master" branch. Working with other developers we don't want to commit directly
# to master and instead create a throw-away branch and perhaps open a "Pull Request" (GitHub
# terminology)/"Merge Request" (GitLab terminology) from this branch to master.
git checkout -b staging

# Create and commit something
touch README.md
git add README.md && git commit -m "Initial commit"

# Immediately tag our release candidate so we can deploy it
# CI should be able to parse the tag name from the webhook
# "v" prefix for version tag and "-rc*" suffix for release candidate
git tag v1.0.0-rc0
git push origin staging --tags

# Everything looked good? Create a new "version 1.0.0" release
git tag v1.0.0
git checkout master
git merge staging
git push origin master --tags

# No longer necessary
git branch -D staging
git push origin :staging
```

Now do a `git log` and things should look similar to our history on the old git workflow.
Notice also we have a tag on the commit that can be referenced at any point no matter where your current working branch is.

```bash
commit 31966eb6b4f592b8c3fd030051c26130888cfc41 (HEAD -> master, tag: v1.0.0-rc0, tag: v1.0.0)
Author: Jay Thomas <jay@gfax.ch>
Date:   Tue Oct 6 12:55:59 2020 -0400

    Initial commit
```
