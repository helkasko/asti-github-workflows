# Changelog

All notable changes to this project will be documented in this file. If possible, each change should also include a
short reasoning for why the change was introduced.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Additionally, breaking changes should be prefixed with **BREAKING:** (see [this discussion](https://github.com/olivierlacan/keep-a-changelog/issues/41)
for more details). Also, remember to document the steps required to fix anything that's been broken by a breaking
change.

In addition, internal changes such as changes to development tools can be distinguished from external API changes by
using the prefix _INTERNAL:_ (see [this](https://github.com/olivierlacan/keep-a-changelog/issues/163) and
[this](https://github.com/olivierlacan/keep-a-changelog/issues/30) discussion).

## Unreleased

<!-- List the changes in your PR under the Unreleased title. You can also copy this list to your PR summary. -->

## <a name="1.3.1"/>[1.3.1] - 2023-01-02

### Fixed

- The `Submit Gradle Dependencies` workflow so that GitHub will process and send security alerts for the submitted
  dependencies.

  GitHub Dependency Graph has a limit on how big manifest files are processed including whether Dependabot alerts will
  be sent for the manifest. Manifests over 0.5 MB in size are only processed for enterprise accounts (see
  [this section](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/troubleshooting-the-dependency-graph#are-there-limits-which-affect-the-dependency-graph-data)
  in the GitHub docs).

  This same limit also affects manifests sent through the Dependency Submission API. To restrict the size of the
  manifest, the amount of dependencies needs to be restricted. By default, the `mikepenz/gradle-dependency-submission`
  GitHub action processes the dependencies for all configurations which can result in a manifest that is too big. The
  dependencies might still be shown in the GitHub UI on the Dependency Graph page without any errors. However, no
  Dependabot alerts will be sent for the dependencies.

  To restrict the size of the manifest, the `testCompileClasspath` configuration filter is used. Since the
  `testImplementation` configuration extends the `implementation` configuration, the `testCompileClasspath`
  configuration will also include the dependencies in the `compileClasspath` configuration. This way, the resulting
  manifest will include production dependencies as well as dependencies needed for testing.
- _INTERNAL:_ An incorrect link in the README file

## <a name="1.3.0"/>[1.3.0] - 2022-12-23

### Added

- A new reusable GitHub workflow for calculating the dependencies for a Gradle project and submitting the resulting
  dependency list to the GitHub Dependency Submission API in order to get Dependabot security alerts for the
  dependencies. For more information, see the [Provided reusable workflows](README.md#provided-reusable-workflows) in
  the README.

## <a name="1.2.2"/>[1.2.2] - 2022-11-16

### Fixed

- The `Dependabot auto-label` and `Dependabot auto-merge` workflows failing when pushing custom changes to a Dependabot
  PR.

  These two workflows rely on the given personal access token (PAT) and if the PAT has only been defined in the
  Dependabot secrets, any workflow runs triggered by other than Dependabot don't have access to the PAT resulting in the
  workflows failing. One option would be to define the PAT in the action secrets so that all workflow runs would have
  access to the secret regardless of who triggered the workflows.

  However, when there are custom changes to a Dependabot PR it might be more reasonable to skip running the workflows
  altogether. For instance, in that case, it might be more reasonable to require a manual approval and merge of the
  Dependabot PR. Also, it's unnecessary to run the `Dependabot auto-label` workflow since it should've been run already
  the first time the PR was created.

  So, the issue has been fixed by skipping the workflows for any custom commits in a Dependabot PR. This has been
  achieved by checking the workflow triggerer instead of the pull request author when deciding whether the workflow
  should be run.

## <a name="1.2.1"/>[1.2.1] - 2022-11-11

### Fixed

- The automatic creation of the `manual review needed` label failing with the error "too many arguments" caused by
  missing double quotes.

## <a name="1.2.0"/>[1.2.0] - 2022-11-10

### Added

- Documentation about:
  - the requirements and required repository settings needed for using the workflows
  - how to use the workflows
  - what the provided reusable workflows do and how they work
- A license file for indicating that the repository is provided under the MIT license
- A changelog to better document what's changed and why so that future development and maintenance would hopefully be
  a bit easier (based on the [keep a changelog](https://keepachangelog.com) conventions)
- _INTERNAL:_ A file defining the [code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
  who are responsible for the repository
- _INTERNAL:_ Versioning for `asti-github-workflows` based on [Semantic Versioning](https://semver.org) so that it's
  easier to understand what type of changes each release contain and to better communicate breaking changes
- _INTERNAL:_ Instructions for publishing a new release

## <a name="1.1.0"/>[1.1.0] - 2022-11-09

### Added

- Support for labelling transitive dependency updates in the `Dependabot auto-label` workflow

## <a name="1.0.0"/>[1.0.0] - 2022-11-08

This marks the `1.0.0` release which defines the public API of `asti-github-workflows`. This also means that the public
API should be stable and the way in which the version is incremented after this release is dependent upon how the public
API changes.

### Added

- Four reusable GitHub workflows for automatically approving, merging, labelling, and rebasing Dependabot's pull
  requests making it easier to reuse these common workflows without having to duplicate them in each using repository.
  For more information, see the [Provided reusable workflows](README.md#provided-reusable-workflows) in the README.

[1.3.1]: https://github.com/helkasko/asti-github-workflows/compare/v1.3.0...v1.3.1
[1.3.0]: https://github.com/helkasko/asti-github-workflows/compare/v1.2.2...v1.3.0
[1.2.2]: https://github.com/helkasko/asti-github-workflows/compare/v1.2.1...v1.2.2
[1.2.1]: https://github.com/helkasko/asti-github-workflows/compare/v1.2.0...v1.2.1
[1.2.0]: https://github.com/helkasko/asti-github-workflows/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/helkasko/asti-github-workflows/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/helkasko/asti-github-workflows/releases/tag/v1.0.0
