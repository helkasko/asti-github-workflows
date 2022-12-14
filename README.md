# ASTI reusable GitHub workflows

This repository contains ASTI's reusable GitHub workflows. Note that this repository is temporarily public and will be
made private as soon as defining reusable workflows in private repositories is supported (see [this issue](https://github.com/github/roadmap/issues/51)).

- [Requirements](#requirements)
- [Usage](#usage)
  - [Auto-rebase Dependabot PR](#auto-rebase-dependabot-pr)
  - [Dependabot auto-approve](#dependabot-auto-approve)
  - [Dependabot auto-label](#dependabot-auto-label)
  - [Dependabot auto-merge](#dependabot-auto-merge)
  - [Gradle dependency submission](#gradle-dependency-submission)
- [Provided reusable workflows](#provided-reusable-workflows)
  - [Auto-rebase Dependabot PR](#auto-rebase-dependabot-pr-1)
  - [Dependabot auto-approve](#dependabot-auto-approve-1)
  - [Dependabot auto-label](#dependabot-auto-label-1)
  - [Dependabot auto-merge](#dependabot-auto-merge-1)
  - [Gradle dependency submission](#gradle-dependency-submission-1)
- [Development](#development)
  - [Versioning](#versioning)
  - [Publishing a new release](#publishing-a-new-release)
- [Changelog](#changelog)

## Requirements

These reusable workflows assume that Dependabot alerts, security updates, and version updates are being used. Thus, you
should have the following settings enabled under `Settings > Code security and analysis`:

- Dependency graph
- Dependabot alerts
- Dependabot security updates
- Dependabot version updates (you have a `dependabot.yml` file in the repository)

Additionally, ensure that you have configured the following settings for the repository where you use the provided
workflows:

- Enable `Settings > General > Allow auto-merge`
  - The `Dependabot auto-merge` workflow is based on Dependabot separately triggering an auto-merge for each PR whereby
    the PR will be automatically merged once all of the required status check have successfully completed. For ensuring
    that Dependabot can automatically enable auto-merge for a PR, this setting needs to be enabled.
- Under the `Settings > Branches > Branch protection rules` section for the default target branch (`main` or `master`),
  enable the following settings:
  - Enable `Require a pull request before merging` and under it the `Require approvals` setting
    - This way, Dependabot can't merge a PR without an explicit approval
  - Enable `Require status checks to pass before merging`. **NOTE!** Remember to also add the required status checks
    that need to pass before the PR can be merged under the `Status checks that are required` section.
      - This way, Dependabot can't merge a PR before the required status checks have completed successfully
  - Enable `Require branches to be up to date before merging`
    - With this you ensure that the tests and other status checks of the Dependabot PR have been run against the latest
      version of the repository's target branch. If this setting hasn't been enabled, it's possible that the status
      checks of the Dependabot PR have passed for a branch that is not up-to-date with the target branch. This could
      lead to the tests passing for the PR branch but once the PR is merged to the target branch, the tests fail causing
      the target branch to be broken.
  - Enable `Restrict who can push to matching branches`
    - This ensures that not anyone belonging to the repository's organization can merge to the repository's target
      branch. Instead, only organization admins, repository admins, and users with the "Maintain" role can merge PRs
      when the PRs required status checks have successfully completed.
    - **NOTE!** For the `Dependabot auto-merge` workflow to work when this setting is enabled, a personal access token
      needs to be used in the `Dependabot auto-merge` workflow.
- Under `Settings > Actions > General`, ensure that you have the following settings configured:
  - Under `Actions permissions`, select `Allow all actions and reusable workflows`
    - This way, you can use any publicly provided GitHub actions and the reusable workflows defined in this repository
  - Under `Workflow permissions`, select `Read and write permissions`
    - The reusable workflows require that the default `GITHUB_TOKEN` can also write to, e.g., pull requests
  - Under `Workflow permissions`, enable `Allow GitHub Actions to create and approve pull requests`
    - This way, the Dependabot PRs can automatically be approved

In addition, you need to ensure that the user whose personal access token is being used in the workflows:
- is granted `Write` access to the repository under `Settings > Collaborators and teams` so that, e.g., pull requests
  can be automatically merged
- is granted access to security alerts under `Settings > Code security and analysis > Access to alerts` so that
  Dependabot PRs related to security can be successfully labelled

Finally, you need to make sure that the repository has access to the required secrets. Under
`Settings > Secrets > Dependabot`, ensure that the personal access token that is used in the workflows is listed as one
of the secrets. For instance, you could create a personal access token for a machine user with the `repo` scope and add
the created personal access token as a secret for your organization so that the secret is automatically available to all
of the organization's repositories.

**NOTE!** To avoid automatically merging breaking dependency updates, you need to ensure that all required checks are
done in the repository's CI pipeline. For instance, if Storybook is used in the application, you need to ensure that
Storybook is successfully built in the CI pipeline. Consequently, you need to ensure that all relevant Yarn scripts and
Gradle tasks are run in the CI pipeline. If, nonetheless, it turns out that Dependabot was able to automatically merge a
breaking change to the target branch, think about how this can be avoided in the future by using an automated check.

## Usage

### Auto-rebase Dependabot PR

Create a file `.github/workflows/auto-rebase-dependabot-pr.yml` with the following content (replace `master` with `main`
if `main` is your default target branch):

```yml
name: Auto-rebase Dependabot PR
on:
  push:
    branches: [master]

jobs:
  rebase-dependabot-pr:
    permissions:
      contents: read
      issues: read
      pull-requests: write
    uses: helkasko/asti-github-workflows/.github/workflows/auto-rebase-dependabot-pr.yml@v1
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Dependabot auto-approve

Create a file `.github/workflows/dependabot-auto-approve.yml` with the following content:

```yml
name: Dependabot auto-approve
on: pull_request

jobs:
  approve-pr:
    permissions:
      contents: read
      issues: write
      pull-requests: write
      repository-projects: write
    uses: helkasko/asti-github-workflows/.github/workflows/dependabot-auto-approve.yml@v1
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Dependabot auto-label

Create a file `.github/workflows/dependabot-auto-label.yml` with the following content:

```yml
name: Dependabot auto-label
on: pull_request

jobs:
  label-pr:
    permissions:
      contents: read
      issues: write
      pull-requests: write
      repository-projects: write
    uses: helkasko/asti-github-workflows/.github/workflows/dependabot-auto-label.yml@v1
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      personal-access-token: ${{ secrets.KASKOASTI_INFRA_PERSONAL_ACCESS_TOKEN }}
```

### Dependabot auto-merge

Create a file `.github/workflows/dependabot-auto-merge.yml` with the following content:

```yml
name: Dependabot auto-merge
on: pull_request

jobs:
  merge-pr:
    permissions:
      contents: write
      pull-requests: write
    uses: helkasko/asti-github-workflows/.github/workflows/dependabot-auto-merge.yml@v1
    secrets:
      personal-access-token: ${{ secrets.KASKOASTI_INFRA_PERSONAL_ACCESS_TOKEN }}
```

### Gradle dependency submission

Create a file `.github/workflows/gradle-dependency-submission.yml` with the following content:

```yml
name: Submit Gradle Dependencies
# Trigger the workflow to run on each change to build.gradle since any changes to dependencies should be reflected in
# the GitHub dependency graph. Also, in order to keep the dependency graph up-to-date in case there's no changes to
# build.gradle for a longer period of time, trigger the workflow to run weekly.
on:
  push:
    branches: [master]
    paths: ['build.gradle']
  schedule:
    # Run every Saturday at 01:00 UTC
    - cron: '0 1 * * 6'

jobs:
  submit-gradle-dependencies:
    permissions:
      # The Dependency Submission API requires write permission
      contents: write
    uses: helkasko/asti-github-workflows/.github/workflows/gradle-dependency-submission.yml@v1
    secrets:
      # Set the private Nexus registry credentials that are passed to Gradle
      registry-url: <replace_this_with_the_registry_url>
      registry-username: ${{ secrets.DEPENDABOT_NEXUS_USERNAME }}
      registry-password: ${{ secrets.DEPENDABOT_NEXUS_PASSWORD }}
```

## Provided reusable workflows

The following reusable workflows are provided by this repository.

### Auto-rebase Dependabot PR

This GitHub workflow is used to automatically rebase open Dependabot pull requests one at a time. Note that this
workflow skips Dependabot PRs that are labelled with the `manual review needed` label. Currently, the
`manual review needed` label is automatically only added to Dependabot PRs that include a major version update.

**NOTE!** For this workflow to work properly, the `rebase-strategy` setting should be set to `disabled` in the
`dependabot.yml` configuration file.

By default, Dependabot already automatically rebases open pull requests when it detects any changes to a pull request.
However, this seems to work only when conflicts emerge between the PR and the target branch. Thus, when the
`Require branches to be up to date before merging` setting is enabled in GitHub, Dependabot doesn't always automatically
rebase a PR if there are no conflicts between the PR and the target branch despite the PR not being up-to-date. Thus,
the only way to trigger a rebase for these Dependabot PRs is to manually rebase the PR, e.g., by commenting
`@dependabot rebase` on the PR.

Additionally, another problem with Dependabot's automatic rebase is that it doesn't trigger the rebases sequentially
but rather concurrently. This, in turn, causes a lot of unnecessary CI builds when the
`Require branches to be up to date before merging` setting is enabled since a PR might be rebased multiple times (see
[this issue](https://github.com/dependabot/dependabot-core/issues/2204)). To mitigate this, this GitHub action is
triggered on each push to the master / main branch and triggers Dependabot to only rebase the oldest PR. This way, you
can ensure that the Dependabot PRs are only rebased one at a time, i.e., sequentially and thus minimizing the amount of
rebases and CI builds needed. For instance, assuming that Dependabot opens 5 PRs, the amount of CI runs with
Dependabot's default rebase mechanism compared to this workflow would be as follows:
- Using the default mechanism and assuming that Dependabot does an automatic rebase after each merge, you would end up
  with 5+4+3+2+1=15 CI runs (one CI run for each PR when it is opened and one CI run for each remaining open PR when a
  PR is merged).
- Using this workflow, you would end up with 5+1+1+1+1=9 CI runs (one CI run for each PR when it is opened and one CI
  run after each merged PR).

**Known issues:**
- Currently, this workflow doesn't work very well in the case that a CI run of one of the Dependabot PRs fails for some
  reason. In that case, the automatic Dependabot PR merge and rebase process can get stuck and requires somebody to fix
  the failing PR before the process can automatically continue. One possible solution would be to first process all
  Dependabot PRs where the CI run has successfully completed and only after that process any Dependabot PRs where the
  CI run has failed.

### Dependabot auto-approve

This GitHub workflow automatically approves Dependabot PRs. Note that this workflow only approves PRs that are concerned
with updating a minor or a patch version of a dependency. Thus, this workflow doesn't automatically approve PRs that are
concerned with a major version update since a major version update typically requires a manual review to ensure that
there are no breaking changes. Consequently, this workflow labels such PRs with the `manual review needed` label. This
label is automatically created by the workflow if it doesn't already exist so there's no need to create the label
manually beforehand.

### Dependabot auto-label

This GitHub workflow helps in automatically categorizing the Dependabot PRs using labels. Using this workflow isn't
required but can be useful to more easily get a better understanding of what type of an update a PR is concerned with
without having to manually dig the info from the PR itself.

This workflow relies on the [Dependabot Fetch Metadata](https://github.com/dependabot/fetch-metadata) GitHub action and
labels the Dependabot PRs with the following labels:

- `development` - Related to development code, e.g., updating a development dependency
- `production` - Related to production code, e.g., updating a production dependency
- `security` - Related to security, e.g., fixing a security vulnerability in a dependency update
- `transitive` - Related to the update of a transitive, i.e., indirect dependency

Note that this workflow creates the labels automatically if they don't already exist. Thus, it's not necessary to create
the labels manually beforehand.

**NOTE!** Labeling security-related PRs requires using the `alert-lookup` setting of the `Dependabot Fetch Metadata`
GitHub action. This in turn requires using a personal access token with the `security_events` scope when using the
workflow. Also, it's necessary to give the user whose personal access token is used access to security alerts (see
[Granting access to security alerts](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-security-and-analysis-settings-for-your-repository#granting-access-to-security-alerts)).

### Dependabot auto-merge

This GitHub workflow automatically enables auto-merge for a Dependabot PR. In other words, when auto-merge has been
enabled for a PR, the PR will automatically be merged once all of the required status checks complete successfully. Note
that this workflow enables auto-merge only for PRs that are concerned with updating a minor or a patch version of a
dependency.

**NOTE!** For ensuring that Dependabot can enable auto-merge when the `Restrict who can push to matching branches`
setting has been enabled, you need to make sure to provide the workflow a personal access token of a user who has access
to the repository.

### Gradle dependency submission

This GitHub workflow is used to calculate the dependencies for a Gradle project and submit the resulting dependency list
to the GitHub Dependency Submission API in order to get Dependabot security alerts for the dependencies.

GitHub's dependency graph uses static analysis to scan the dependencies from manifest files or lockfiles in
repositories. However, for some ecosystems and tools such as Gradle the dependencies cannot reliably be parsed
statically. As such, the GitHub dependency graph doesn't support these kinds of package ecosystems. For a list of
supported package ecosystems, see [the Supported package ecosystems section](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-the-dependency-graph#supported-package-ecosystems)
on GitHub.

Since the dependency graph is used as a basis for detecting vulnerable dependencies, it hasn't previously been
possible to get Dependabot security alerts for unsupported package ecosystems. However, GitHub has now added a new
Dependency Submission API which allows submitting the dynamically calculated dependency list to GitHub's dependency
graph (see [this blog post](https://github.blog/2022-06-17-creating-comprehensive-dependency-graph-build-time-detection/)).
Thus, this workflow utilizes the [gradle-dependency-submission](https://github.com/mikepenz/gradle-dependency-submission)
GitHub action to submit the dependencies of a Gradle project to the GitHub dependency graph.

**NOTE!** The `gradle-dependency-submission` GitHub action requires the project to use Gradle version 7.5+.

## Development

### Versioning

The ASTI reusable GitHub workflows adhere to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

### Publishing a new release

To publish a new version, follow these steps:

- Create a new PR for incrementing the version
- Make sure that you've listed your changes in the [changelog](CHANGELOG.md). A good practice is to list your changes
  during development under the `Unreleased` title. That way, you can easily move the list under the correct version when
  it's time to increment the version.
- Based on Semantic Versioning, decide what's the correct version number for the new version and add the new version to
  the [changelog](CHANGELOG.md).

Ask for another developer to review your PR. Once the PR has been merged to `main`, remember to tag the merge commit in
the `main` branch with the new version:

- Create an annotated tag: `git tag -s -a v1.x.x -m "v1.x.x"` (**NOTE!** Make sure that you create an annotated tag
  which contains, e.g., author metadata. Also, make sure that you tag the correct commit. If you haven't configured PGP
  signing, you can remove the `-s` flag).
- Push the tag to GitHub: `git push origin v1.x.x`
- Create a new release (remember to replace the version number with the correct version):
  ```sh
  gh release create \
    "v1.x.x" \
    --notes "See [changelog for version v1.x.x](https://github.com/helkasko/asti-github-workflows/blob/main/CHANGELOG.md#1.x.x)."
  ```
- Update the `v1` tag to point to the new version:
  ```sh
  git fetch --all --tags
  git checkout v1.x.x # Check out the latest release
  git tag -f -s -a v1 -m "v1" # Force update the tracking tag to point to the latest release
  git push -f origin v1
  ```

## Changelog

This project uses a changelog to keep track of all notable changes between versions. See [CHANGELOG](CHANGELOG.md) for
more details.
