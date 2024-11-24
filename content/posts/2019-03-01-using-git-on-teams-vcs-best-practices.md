---
author: Josh Lee
date: "2019-03-01T15:59:08Z"
excerpt: There are a lot of ways to use git to collaborate with other developers.
  I’ve found the following principles to be useful for keeping my sanity when working
  with git.
tags:
- git
title: 'Using git on Small Teams: Best Practices'
url: /using-git-on-teams-vcs-best-practices/
---

There are a *lot* of ways to use git to collaborate with other developers. I’ve found the following principles to be useful for keeping my sanity when working with git. My experience has mostly been with small teams of two to six developers, but these principles may apply to any size team. Of course, “best practices” are elusive, and the following is just my established method.

## Commandments:

1. Do not commit vendor code.
2. Do not commit compiled code.
3. Do not commit commented-out code — just delete it.
4. Make small commits.
5. Explain the why, not the what, in your commit message.
6. Create a new branch for every task / small feature / bug fix.
7. Rebase instead of merge to resolve conflicts.
8. Use pull requests. Get reviews.
9. Do not commit or push directly to the master branch.
10. Deploy from the master branch to staging.

### “Master” branch only contains reviewed code, merged via PR

When an entire QA department is not available, the best QA gains can be had by ensuring every single line of code is seen by at least 2 developers before being merged into the master branch. 

### One branch per feature/fix/refactor

There are myriad branching strategies all discussed at length online. My recommendation for small, distributed teams without dedicated QA Engineers is to follow a simplified version of the “git-flow” model. Rather than manage the complexity of release and integration branches, the simplified model has feature and fix branches coming off of master and being merged directly back into master. Keeping each feature or concern on a separate branch makes it much simpler to back out of specific changes, to identify the source of bugs, and to understand how the codebase is changing over time. 

### Keep commits small

Every commit should be as small as possible without breaking functionality, with a good commit message explaining WHY the change was made. For consistency, I like to write commit messages in the present tense, starting with a verb e.g.: “Adds compatibility with Twitter Bootstrap 4.” By keeping commit messages descriptive about the why (rather than the what, which should be self evident in a small commit), other team members can clearly reason about what they might need to change for their future tasks. 

### Don’t commit commented-out code

The whole point of a VCS (Version Control System) is that you don’t have to do this… the history of you deleting the line is maintained. And if you make a small commit with a good message it will also be easy to find. 

### Deploy from the master branch

Deployments should happen from the master branch, not a specific environment or test branch. The same git ref (commit) on master should be deployable to any environment, so that production can be a binary copy of what was tested on staging. Which brings us to… 

### Promote artifacts, not code

Best practices dictate that an “artifact” (or package) be created for staging deployments, and that artifact then be deployed as-is to production, without the build process being re-run. This eliminates a potential avenue for disparity between environments and allows you to always know bit-for-bit what code is running in production. 

### Start with a single repository per product

Large teams can benefit from multiple repositories and a submoduleing system. For teams under ten, I generally prefer to keep all code required to run the entire product in a single repository. This allows the application to be managed in it’s entirety and deployed as a single unit. Internal tools that are re-used between projects should be made available via a package manager such as npm, bower, composer, etc. 

### Don’t commit vendor code

Committing vendor code can cause merge conflicts and disparity with packaged libraries. Vendor code should always be .gitignored and redownloaded directly from the source by a build server. This includes internal tools mentioned above, managed via package manager. 

### Don’t commit compiled/transpiled code

Compiled files (usually css, js) are often minified and will cause a merge conflict on every merge unless great care is taken to avoid this. Having the build server re-run compilers ensures consistency between environments. 

### Only merge using Pull Requests

Merges make for a very messy git history, making it harder to find information about the code. Merges should only happen into master via pull request. If you are working on a feature branch, and there are new commits on master which you would like to include in your branch, use a rebase instead of a merge. With master updated and your feature branch checked out, type `git rebase -i master` — you’ll be prompted to edit the “TODO” using your preferred command line editor. This lets you squash, rename, or delete (drop) commits. 

### Use a soft reset for those “squirrell” moments

As developers, we love to fix things as soon as we see them. If, after coding for a while, you find that you have committed changes in many areas not related to the feature you are currently working on, a soft reset will allow you to completely undo all of the commits while maintaining the changes in your working directory. You can then commit the changes to individual feature/fix branches as necessary. This is also useful when you would like to rebase but also have the option to SPLIT (instead of squashing) commits. From your branch, run:

```
git checkout -b my_branchname_backup
git checkout my_branchname
git reset master
```

Thats it. *`my_branchname`* will now be at the last commit on master, and all of your changes will still be there in your working folder. 
