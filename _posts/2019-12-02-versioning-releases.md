---
layout: post
title:  Versioning and Releases
author: Elsa Gonsiorowski
category: ross-dev
---

In addition to the [git branching workflow](https://nvie.com/posts/a-successful-git-branching-model/) development model, the ROSS project uses the [Semantic Versioning](https://semver.org/) model for version numbers.
This helps with reproducibility of experiments detailed in publications and helps users ensure that they are using the correct version of ROSS, especially when using other software that depends on ROSS.

## Versioning

According to the [Semantic Versioning](https://semver.org/) model, we increment the major/minor/patch numbers in the version depending on the type of changes.

- **Major** version is incremented when an incompatible API change is made. This can happen when an API function has a change in function signature or when an API function is removed entirely. This could also happen when new features are released.
- **Minor** version is incremented when a backward-compatible API change is made. This can happen when a API function is added or a function is marked as deprecated (but is not removed).
- **Patch** version is incremented when other patches or bug-fixes are made.

## Release Process

As partially outlined in the [workflow blog post](https://ross-org.github.io/ross-dev/git-branching-workflow.html), the release process is as follows:
1. A new branch, named `release-x.y.z` is created from `develop` (where `x.y.z` is the new version number).
2. This branch may exist for a short time and may have some patches or bug-fixes added to it. No new/major changes should be made to this branch. This branch is pushed to GitHub.
3. Through PRs, the release branch is merged into both `master` and `develop`. This should be a regular merge, *not* a squash-on-merge. If there have been no fixes to the release branch, a merge to `develop` may not be necessary.
4. Once the merge is complete, a new release is tagged on the [GitHub releases](https://github.com/ROSS-org/ROSS/releases) page.

## Development Process

For development, we will continue to utilize *squash commits* to merge any PRs on the `develop` branch.
Squash commits have several implications:

1. *The squash-on-merge option must be selected by the person doing the merge.*
1. The individual commits are not placed in the history of the master branch.
   However, they do remain available through the pull request page.
2. One positive outcome is that the blame on any file will be simplified since there is now only one commit associated with the entire change.
3. Once a feature branch is merged into master, it should be **deleted from any local repositories**.
   There are possible issues if someone attempts to re-merge the branch (including commits previously added in a squash).

## Historical ROSS Releases

Prior to 2018, ROSS has only had a few "official" releases:

- **ROSS 5.0** *Oct 15, 2009*: <br />
  In late 2009, after 10 years of development, ROSS was made open source and released on SourceForge.net.
- **RNF 1.1** *Nov 6, 2012*: <br />
  ROSS Net Framework was a model for network simulation built within ROSS itself.
- **ROSS Legacy** *Jan 21, 2015*: <br />
  This was more of a tagged-version rather than a release.
  It was supposed to mark the last stable commit before Simplified ROSS begun to be merged.
- **Simplified ROSS** *Jan 23, 2015*: <br />
  This release came after the gonsie/SR version was completely and successfully merged.
- **ROSS 7.0.0** *August 14, 2018*: <br />
  This release kicked off the beginning of ROSS's semantic versioning.
