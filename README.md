# ASTI reusable GitHub workflows

This repository contains ASTI's reusable GitHub workflows. Note that this repository is temporarily public and will be
made private as soon as defining reusable workflows in private repositories is supported (see [this issue](https://github.com/github/roadmap/issues/51)).

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
