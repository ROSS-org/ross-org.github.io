---
layout: post
title:  Git Branching Workflow
author: Caitlin Ross
category: ross-dev
---

We're now following a new Git development model for ROSS, based on [this post by Vincent Driessen](https://nvie.com/posts/a-successful-git-branching-model/).
For anyone using ROSS and not doing development work, nothing changes.
Stick to the master branch, which will at the most recent ROSS release.
For ROSS devs, continue reading.

*Note:*  There will be more details added soon in regards to versioning, but for now the basics are listed.

### ROSS Branches

- `origin/master`: Production-ready state
- `origin/develop`: Has latest development changes for the next release.

Rules have been added to both branches to ensure that any branch being merged is up to date and that the Travis CI checks pass.
At this point, any contributor should be able to merge into develop, but only ROSS-org/repo admin can merge to master.


### Feature Development

If you're adding a new feature to ROSS, you'll typically branch off of `develop`.
Either way, when your feature is ready, submit a PR for merging to `develop`.

### Release Branches

When the recent commits in a develop branch are ready for release, a new release branch can be created.
It should be branched off from `develop` and be merged back into both `develop` and `master`.
We'll follow the naming convention of `release-*`, with the version number replacing the `*`.

Bug fixes can be done in this branch, but no new features should be added directly to release branches.

When merging into `master`, that commit should be tagged with the version number on `master`.

### Hotfix Branches

If a critical bug is found, at hotfix branch can be branched off of `master`.
The branch should use the naming convention `hotfix-*` with the version number.
When ready the branch should be merged into both `develop` and `master`.
Don't forget after merging into `master` that the commit should be tagged with the new version number.
