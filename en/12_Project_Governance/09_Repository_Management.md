---
title: GitHub repository management
summary: Information about management of the various GitHub repositories
iconBrand: github
---

# GitHub repository management

This document outlines the rules and processes for managing the GitHub repositories for supported modules.

## Branches

### Naming convention

Branches with a single integer (e.g. `1`) represent either the next *major* release (if there have been no tagged releases for that release line yet) or the next *minor* release.

Branches with an integer, a dot, and another integer (e.g. `1.2`) represent either the *next patch release* for the current minor, or a patch release for a previous minor version.

> [!NOTE]
> Branches with any other naming strategy should not be the target of any pull requests. Only branches following this naming convention are included in merge-ups and are used as the base for new tagged releases.

For example, if we have the following tags:

```text
1.1.17
1.2.0
```

Then:

- `1.1` is the patch release branch for the previous minor version. High severity security fixes should target this branch.
- `1.2` is the next-patch release branch for the current minor version. Bugfixes should target this branch.
- `1` is the next-minor release branch. Enhancements should target this branch.
- `2` is the next-major release branch. Changes that break backwards compatibility should target this branch.

### Branch protection

Some actions are restricted to avoid bad actors or accidental mistakes from causing problems. This section outlines the branch protection rules, and who they apply to.

If any exceptions to these rules need to be made for any reason, each exception must be approved in writing by the Silverstripe CMS Product Owner and only apply for a specified amount of time.

#### Branch protection rules

##### Updating branch and tag protection rulesets

Use the `rulesets` command in [silverstripe/module-standardiser](https://github.com/silverstripe/module-standardiser) to add and update the branch and tag protection rulesets for all support modules via the GitHub API.

These rulesets restrict non-admins from various activities such as merging pull-requests without a review, creating branches, and pushing tags.

##### Updating branch protection rule

A branch protection rule to protect against deleting branches is added by going to `https://github.com/silverstripe/<repository>/settings/branches`, clicking "Add rule", inputting `[0-9]*` as the branch name pattern, and clicking create (without checking any of the checkboxes).

There is no REST API endpoint available to add this rule with a pattern, so it must be done manually.

The rule is used in additional to the branch ruleset to remove the ability for admins to bypass deleting protected branches.

#### Deleting branches

Any branches following the documented naming convention cannot be deleted by anyone.

If a branch following that naming convention is created by mistake, a Core Committer can rename and then delete the branch.

#### Access for directly pushing commits

By default, users don't have access to directly push commits unless they are given that permission either directly or by being in a group which has that permission.

The use cases for access to directly push commits to branches (as opposed to using a pull request) are:

- Manually resolving merge conflicts for a merge-up between branches. Our CI pipeline automatically merges up commits between branches, but occasionally will encounter a merge conflict which needs to be manually resolved, committed, and pushed.

Senior developers in the CMS Squad and all members of the Core Committers team are given access to directly push commits to supported module repositories.

#### Access for pushing tags

By default, users don't have access to push tags unless they are given that permission either directly or by being in a group which has that permission.

The use cases for access to push tags are:

- Tagging a new minor release. We use a tool to help with this process, but the person using the tool still needs access to push the tag with their GitHub account.
- Tagging a new major release. This is effectively the same process as tagging a new minor release.

Our CI pipeline includes a process which automatically tags patch releases on patch-release branches if there are commits which warrant a tag. In theory there should be no reason for someone to manually tag a patch release for a supported module. If CI mistakenly thinks there are no commits which warrant a patch release, the logic for checking commits should be updated and CI should be rerun against that branch.

Senior developers in the CMS Squad and all members of the Core Committers team are given access to push tags to supported module repositories.
