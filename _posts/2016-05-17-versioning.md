---
layout: post
title:  Versioning and Releases
author: Elsa Gonsiorowski
category: ross-dev
---

Is the history of your repository safe?
If there are continuous integration tests, does that mean every merged commit is safe?
Turns out, the way in which branches were merged into master made some false assumption, which, in turn, hurt the end users.
This post lays out the old and the new, documenting the way in which ROSS will be versioned.

## ROSS Releases

ROSS has only had a few "official" releases:

- **ROSS 5.0** *Oct 15, 2009*: <br />
  In late 2009, after 10 years of development, ROSS was made open source and released on SourceForge.net.
- **RNF 1.1** *Nov 6, 2012*: <br />
  ROSS Net Framework was a model for network simulation built within ROSS itself.
- **ROSS Legacy** *Jan 21, 2015*: <br />
  This was more of a tagged-version rather than a release.
  It was supposed to mark the last stable commit before Simplified ROSS begun to be merged.
- **Simplified ROSS** *Jan 23, 2015*: <br />
  This release came after the gonsie/SR version was completely and successfully merged.

Today, most of these releases are useless.
The ROSS API has since changed and the lack of a naming pattern for the releases has only lead to confusion.

## The Old Way

The ROSS development team has had two goals:

- Each commit should be stable.
  I.e, each commit hash is essentially a version number that should be eternally safe for the end user.
- The master branch should contain all of the commit history.

Turns out, these desires are incompatible in git.[^1]
By merging the commit history of the branch into master, there became interleaved commit history.
This is best explained through an example.

### The Merge of the Realtime Scheduler

The timeline of events is as follows:

- *Jan 5*: Professor Carothers begins work and makes some commits on the realtime branch.
- *Feb 3*: The master branch gets a small/bug-fix commit [d5a9253](https://github.com/ross-org/ROSS/commit/d5a9253cf13e3aa1b5e5b5c5550538f0b15d58a7)
- *Mar 1*: A user downloads the latest version of ROSS (d5a9253) and begins developing a new model.
- *Apr 19*: The realtime branch is merged into master via PR [#79](https://github.com/ross-org/ROSS/pull/79)
- *May 1*: The user would like to share their code with a team member and state that their model works with commit d5a9253.
  When the team member downloads ROSS and checks out commit d5a9253, the some commits from the realtime branch now appear on the master branch, before d5a9253.
  These commits, for a partially implemented feature, break ROSS.

Thus, there is no guarantee that the once stable d5a9253 version will remain valid after a branch merge.

![Merge Gif]({{ site.baseurl }}/images/merge.gif)

## The New Way

As of August 2018, ROSS will follow the [Semantic Versioning](https://semver.org/) specifications for ROSS version numbers, instead of relying on commits to master to be version numbers.
This will help with reproducibility of experiments detailed in publications, as well as help users ensure that they are using the correct version of ROSS, especially when using other software that depends on ROSS.
The most recent official release, Simplified ROSS in January 2015, is now considered to be version 6.0.0.
Since many changes have been made to ROSS between January 2015 and now, we are tagging the most recent commit, 26100bb, as version 7.0.0.
From this point on, we'll increment the major/minor/patch numbers in the version appropriately.

We will continue to utilize *squash commits* to merge any changes.
Squash commits have several implications:

1. *The squash-on-merge option must be selected by the person doing the merge.*
1. The individual commits are not placed in the history of the master branch.
   However, they do remain available through the pull request page.
2. One positive outcome is that the blame on any file will be simplified since there is now only one commit associated with the entire change.
3. Once a feature branch is merged into master, it should be **deleted from any local repositories**.
   There are possible issues if someone attempts to re-merge the branch (including commits previously added in a squash).

![Squash Gif]({{ site.baseurl }}/images/squash.gif)


### Footnotes

[^1]: If you are interested on continuous integration testing and the validity of each commit, check out this article: [Not Rocket Science](http://graydon.livejournal.com/186550.html).
