---
title: Contributing Code
summary: Fix bugs and add new features to help make Silverstripe CMS better.
icon: code
---

# Contributing code - submitting bugfixes and enhancements

The Silverstripe CMS core and supported modules are hosted on [GitHub](https://github.com) - mostly in the [Silverstripe GitHub organisation](https://github.com/silverstripe/). To contribute code, you will need to [create a GitHub account](https://docs.github.com/en/get-started/onboarding/getting-started-with-your-github-account).

This documentation assumes you are fairly confident with git and GitHub. If that isn't the case, you may want to read some guides for [GitHub](https://docs.github.com/en/get-started/quickstart), [git](https://docs.github.com/en/get-started/using-git), and [pull requests](https://docs.github.com/en/pull-requests).

> [!TIP]
> Note: By supplying code to the Silverstripe CMS core team in issues and pull requests, you agree to assign copyright of that code to Silverstripe Limited, on the condition that Silverstripe Limited releases that code under the BSD license.
>
> We ask for this so that the ownership in the license is clear and unambiguous, and so that community involvement doesn't stop us from being able to continue supporting these projects. By releasing this code under a permissive license, this copyright assignment won't prevent you from using the code in any way you see fit.

## Before you start working {#before-you-start}

There are a few things that you should do before you start working on a fix:

> [!NOTE]
> If you want to contribute changes to documentation, please read through the [contributing documentation](./documentation) page.

### Consider if your change should be its own module

Not every feature belongs in the core modules - consider whether the change you want to make belongs in core or whether it would be more appropriate for you to [create a new module](/developer_guides/extending/modules/#create).

### Check for an existing pull request

Check to see if someone else has already submitted a pull request for this change by searching on GitHub. If they have, consider collaborating with them by reviewing their PR.

### Make or find a GitHub issue

Whether you're fixing a bug, updating documentation, making an enhancement for an existing feature, or even a brand new feature, you must link your pull request to a GitHub issue.

If there's an existing GitHub issue, there may already be some discussion there about the preferred approach. Make sure you read through the comments.

If there *isn't* an existing issue, you should create one. Make sure you mention in your issue that you intend to make a pull request to implement the change (especially if this is for a new feature).

If you are planning to develop an extensive feature or fix a bug that could have wide-reaching effects, try to get some discussion in your issue before you do much coding. Make it clear in the issue that you want to discuss it before working on it, and consider discussing the problem in one of the [community channels](https://www.silverstripe.org/community/) (and summarise the discussion in the issue afterward).

Refer to [Contributing Issues](./issues_and_bugs/) for more information about finding and creating GitHub issues.

## Step-by-step: how to contribute code {#step-by-step}

> [!WARNING]
> The examples below assume you are making a change that applies to the `6.1` branch.
>
> Please adjust the commands as appropriate for the version of Silverstripe CMS that you're targeting. See [picking the right version](#picking-the-right-version).

### Editing files directly on GitHub

If you see a typo or another small fix that needs to be made, and you don't have an installation set up for contributions, you can edit files directly in the GitHub web interface.

After you have edited the file, GitHub will offer to create a pull request for you. This pull request will be reviewed along with other pull requests.

Make sure you read the [picking the right version](#picking-the-right-version), [create the pull request](#create-the-pr), and [receive and respond to feedback](#receive-feedback) sections below.

### Step 1: picking the right version {#picking-the-right-version}

The Silverstripe CMS project follows [Semantic Versioning](https://semver.org), which clarifies what to expect from different releases and also guides you in choosing the right branch to base your pull request on.

As we follow semantic versioning, we name the branches in repositories accordingly:

- `<digits>` (e.g. `6`) branches contain all changes for upcoming major or minor releases. These are called "major release branches" or "minor release branches", depending on whether they represent the next major release or the next minor release.
- `<digits>.<digits>` (e.g. `6.1`) branches contain all changes for upcoming patch releases. These are called "patch release branches".

If after reading this section you are still unsure what branch your pull request should go to, consider asking either in the GitHub issue that you address with your PR or in one of the various [community channels](https://www.silverstripe.org/community/).

> [!TIP]
> Refer to our [definition of public API](/project_governance/public_api/) for the following sections.

Any updates to third party dependencies in `composer.json` should aim to target the default branch for a minor release if possible. Targeting a patch release branch is acceptable if updating dependencies is required to fix a high impact or critical bug and is unlikely to result in regressions.

#### For changes to public API or new/enhanced features

If you are introducing new public API, introducing new features, or enhancing an existing feature, you should generally use the default branch of the repository where you want to contribute to. That would usually target the next minor release of the module.

#### For bug fixes that don't introduce new API

If you are fixing a bug that doesn't require public API changes, use the highest patch release branch available for the lowest supported major release line the bug applies to. You can see the currently supported release lines for Silverstripe CMS on [the roadmap](https://www.silverstripe.org/software/roadmap/). You can find which major release lines of core and supported modules apply to that version by checking the relevant [supported modules](/project_governance/supported_modules/) page.

For example, if your bug fix is applicable for Silverstripe CMS 5, and is for the `silverstripe/admin` module, you would target the `2.4` branch.

#### For API breaking changes

Do not make a pull request that includes a breaking change, including changing public API, unless there is a major release branch ready to merge into.
For example if the latest stable release is `6.2.7`, the major release branch would be `7`.

### Step 2: install the project {#install-the-project}

Install the project through composer. The process is described in detail in the [getting started](../getting_started/composer#contributing) docs.

```bash
composer create-project --keep-vcs silverstripe/installer ./your-website-folder 6.1.x-dev
```

Note that if you already have a working project and would like to implement the change in the context of that project, you will need to make sure you have the full source of the module using the [`composer reinstall`](https://getcomposer.org/doc/03-cli.md#reinstall) command:

```bash
# re-install the module using prefer-source.
# replace <org>/<module> with the module you're making changes to (e.g. silverstripe/framework)
composer reinstall <org>/<module> --prefer-source
```

### Step 3: prepare your working directory {#prepare-your-working-directory}

- Create a [fork](https://help.github.com/articles/about-forks/) of the module you want to contribute to by going to the repository in your browser, clicking the "fork" button, and following the instructions.

- Add your fork as a "remote" to the module you want to contribute to. This is where you will be pushing changes to.

    ```bash
    cd vendor/<org>/<module>
    git remote add pr git@github.com:<your-github-user>/<the-repo-name>.git
    ```

- Create a working branch.

    ```bash
    # make sure you're starting from the correct branch first
    cd vendor/<org>/<module>
    git checkout --track origin/6.1
    # then create your working branch
    git checkout -b <your-branch-name>
    ```

> [!TIP]
> Use a descriptive name for your branch. For example if you are fixing a bug related to swapping preview modes targetting the `6.1` branch: `pulls/6.1/fix-preview-modes`

### Step 4: work on your pull request {#work-on-your-pr}

Work on the code as much as you want and commit as often as you need to, but keep the following in mind:

- Adhere to our [coding conventions](/contributing/coding_conventions)
- Most pull requests only need a single commit, but complex changes might be better served with multiple commits. In either case, each commit must have a clear and distinct purpose
- Commit messages should conform to our [commit message guidelines](#commit-messages)
- Document new API through [PHPDoc](https://en.wikipedia.org/wiki/PHPDoc) comments. These are used in IDEs and in our [API documentation](https://api.silverstripe.org/).
- Add [unit tests](../developer_guides/testing/unit_testing) which prove your change works and which will prevent future regressions
- Add [behat tests](https://github.com/silverstripe/silverstripe-behat-extension) for any complex behaviour and for any JavaScript functionality changes
- Avoid making unrelated changes (such as fixing coding standards) which are not the focus of your pull request. Those sorts of changes increase the work required to review your pull request.
- It's better to submit multiple pull requests with separate bits of functionality than a big pull request containing lots of changes. If your pull request contains lots of unrelated changes you will be asked to submit them as separate pull requests.
- If you are adding a new feature or changing the way an existing feature behaves, you will need to also create a pull request against [silverstripe/developer-docs](https://github.com/silverstripe/developer-docs) to update the documentation and add information about the change to the changelog.
- If you have updated any source files for CSS or JavaScript, you'll need to build and commit the dist files. See [client-side build tooling](build_tooling) for details.

#### Commit messages

We try to maintain a consistent record of descriptive commit messages.
Most importantly: Keep the first line short, and add more detail below.
This ensures commits are easy to browse and quickly see what the purpose of the commit is.

Our [changelog](https://docs.silverstripe.org/en/changelogs/) generation tool relies upon commit prefixes
to categorize commits and produce more readable output. The prefixes are a single case-insensitive term
at the beginning of the commit message.

| Prefix | Description                                                                                                                                      |
| ---    | ---                                                                                                                                              |
| API    | Addition of a new public/protected API, or modification/removal/deprecation of an existing API.                                                  |
| NEW    | New feature or major enhancement (both for users and developers)                                                                                 |
| ENH    | Improvements of existing functionality (with no API changes), UI/UX enhancements, refactoring and configuration updates.                         |
| FIX    | Bugfix on something developers or users are likely to encounter.                                                                                 |
| DOC    | Any documentation changes.                                                                                                                       |
| DEP    | Dependency version updates (updates for `composer.json`, package.json etc)                                                                         |
| TLN    | Translation commits                                                                                                                              |
| MNT    | Maintenance commits that have no impact on users and applications (e.g. CI configs) - ommitted from the changelog                                |
| Merge  | PR merges and merge-ups - ommitted from the changelog                                                                                            |

If you can't find the correct prefix for your commit, it is alright to leave it untagged. The commit will then fall into "Other" category.

Example: Good commit message

```text
FIX Allow multiple iterations of eager-loaded DataLists

Previously, a second iteration would add every relation item to the
relation list a second time - so with each iteration your relation list
count doubled (though it was the same records time and again).
```

### Step 5: create the pull request {#create-the-pr}

When you are ready, push your branch to your GitHub fork. It's also a good idea to do this during development if the process is taking more than one day since this effectively backs up your work for you.

Only submit a pull request for work you think is ready to merge. Work in progress is best discussed in an issue (you can link to the code in your fork if you need to refer to it).

```bash
cd vendor/<org>/<module>
git push pr <your-branch-name>
```

Then [create a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork) on GitHub. If you are raising multiple pull requests that work together to solve a problem, make sure you link to them and indicate what the dependencies are (e.g. one PR might require another in order to work and for the tests to pass - in that case we need to know which to merge first).

If there is a template for the pull request description, follow it as closely as you can. If there isn't, then provide a quick summary of what your changes are, why you're making them, and a link to the relevant GitHub issue. Make sure to include steps to manually test the effects of the change.

It's also a good idea to add a link to your PR in the relevant GitHub issue. Add the link in the issue description if you have edit rights, and in a new comment otherwise. Doing this improves the chances of your PR being noticed.

Following these additional guidelines for your pull request will improve the chances that your change will be merged:

- Link to [the relevant GitHub issue](#make-or-find-a-github-issue) from your pull request description
- Link to your pull request from that issue (in the description if you can edit it, or in a comment otherwise)
- Explain your implementation. If there's anything which you needed to do a deep dive to find the best way to do it, or anything potentially controversial, etc, you can add a comment to your own pull request explaining what you did and why you did it that way.

### Step 6: receive and respond to feedback {#receive-feedback}

Once your pull request is created, it's not the end of the road.

#### Automated feedback

Most of the core and supported repositories have an automated GitHub Actions workflow which will run on your pull request. When it finishes running, check for any failed builds.

If you think a build has failed for reasons unrelated to the changes you've made, point that out in a comment. If the failure *is* related to your changes, then make any adjustments necessary to resolve the problems.

#### Peer review feedback

The core team will review the pull request as time permits. They will most likely have some questions for you and may ask you to make some changes, so make sure you have [configured your GitHub notifications](https://docs.github.com/en/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/configuring-notifications) appropriately.

- Try to respond to feedback in a timely manner. PRs that go for a while without a response from the author are considered stale, and will be politely chased up. If a response still isn't received, the PR will eventually be closed.
- If you don't agree with a requested change, provide a clear reason why. Bonus points for showing precedent in the existing codebase. But be open to accepting alternative view points - if a member of the core team insists that you make the change after responding to your reasoning, it's often best to defer to their judgment.

#### Resolving merge conflicts

If other changes are merged in before yours, your pull request may end up with merge conflicts. You'll need to resolve those by rebasing your branch on top of the target branch, and then manually resolving the merge conflicts.

> [!WARNING]
> Using `--force-with-lease` is necessary after a rebase, because otherwise GitHub will reject your push. This is because your commit hashes will have changed. But beware that you are explicitly telling GitHub that you are intentionally overriding data. Make sure you have the correct branch name when doing that step to avoid accidentally overriding other branches in your forked repository.

```bash
cd vendor/<org>/<module>
git checkout 6.1
git pull
git checkout <your-branch-name>
git rebase 6.1
# if there are merge conflicts, resolve them at this stage then run git rebase --continue
git push pr <your-branch-name> --force-with-lease
```
