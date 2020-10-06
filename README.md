# The One-Branch Git Workflow

What if we could have a single git branch for our workflow and tag commits that we want to mark as releases?
What if we could have a clearer git history that is human readable and easy to track down releases?
What if all this reduced the amount of steps in the CI pipeline as well?

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
- [Features](#features)
- [Examples](#examples)
  - [Scenario 1 - Staging a release](#scenario-1---staging-a-release)
  - [Scenario 2 - Asynchronous work](#scenario-2---asynchronous-work)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


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

```
commit 31966eb6b4f592b8c3fd030051c26130888cfc41 (HEAD -> master, tag: v1.0.0-rc0, tag: v1.0.0)
Author: Jay Thomas <jay@gfax.ch>
Date:   Tue Oct 6 12:55:59 2020 -0400

    Initial commit
```

### Scenario 2 - Asynchronous work

Continuing off the previous scenario's repo, let's stage multiple release candidates while `master` diverges with separate commits.

```bash
# Let's add a new feature to go into the release candidate
touch file.b
git add . && commit -m "Add file B"
# Recreating staging branch. Keep in mind the name is
# arbitrary as the branch's existence is inconsequential.
git checkout -b staging
# Immediately tag our rc with a version
git tag v2.0.0-rc0

# But let's add a new "feature" to master while testing staging
git checkout master
touch file.c
git add . && git commit -m "Add file C"

# Go back to testing staging. Uh oh, we found a bug we need to fix.
git checkout staging
echo "my file fixes" > README.md  # Make the fix

# Fix it within staging. master and staging have diverged.
git add . && git commit -am "Fix README"
git tag v2.0.0-rc1

# ...Everything looks good. Cut a release
git tag v2.0.0
git checkout master
git merge staging

# No longer needed. Delete it to avoid confusion that there is still
# a pending release. Clear the canvas for the next RC so to speak.
git branch -D staging

# Push everything up
git push origin master :staging --tags
```

Now if you `git log` on master you can see it contains the latest staging as well as the new work that is exclusive to development:

```bash
commit b119b5d73b8efb02177fffa4538d4651ca91f08c (HEAD -> master)
Merge: 8ed8494 59ed050
Author: Jay Thomas <jay@gfax.ch>
Date:   Mon Nov 25 17:49:11 2019 -0500

    Merge branch 'staging' into master

commit d637b8fecabe6a4f68f9b4846f1c2773576ec8d0
Author: Jay Thomas <jay2@gfax.ch>
Date:   Tue Nov 26 10:19:41 2019 -0500

    Add file C

commit 8cbf6f6af52391d90a3be708cd64f8dd5514b600 (tag: v2.0.0-rc1, tag: v2.0.0)
Author: Jay Thomas <jay@gfax.ch>
Date:   Mon Nov 25 16:20:31 2019 -0500

    Fix file A

commit dad9b824a9d070637d975853d5d4ea785585cba3 (tag: v2.0.0-rc0)
Author: Jay Thomas <jay@gfax.ch>
Date:   Mon Nov 25 16:18:10 2019 -0500

    Add file B

commit 2dabe9ae38501d8228bb168540364ab979ad56bc (tag: v1.0.0-rc0, tag: v1.0.0)
Author: Jay Thomas <jay@gfax.ch>
Date:   Mon Nov 25 15:38:24 2019 -0500

    Add file A
```

But we don't want file C in the release, and it's not. Have a look at `git log v2.0.0`:

```bash
commit 8cbf6f6af52391d90a3be708cd64f8dd5514b600 (tag: v2.0.0-rc1, tag: v2.0.0)
Author: Jay Thomas <jay@gfax.ch>
Date:   Mon Nov 25 16:20:31 2019 -0500

    Fix file A

commit dad9b824a9d070637d975853d5d4ea785585cba3 (tag: v2.0.0-rc0)
Author: Jay2 Thomas <jay2@gfax.ch>
Date:   Tue Nov 26 10:19:26 2019 -0500

    Add file B

commit 2dabe9ae38501d8228bb168540364ab979ad56bc (tag: v1.0.0-rc0, tag: v1.0.0)
Author: Jay Thomas <jay@gfax.ch>
Date:   Mon Nov 25 15:38:24 2019 -0500

    Add file A
```
