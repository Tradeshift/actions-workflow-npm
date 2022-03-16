# actions-workflow-npm

Shared workflows for npm packages, such as creating pre-releases and semantic
release automation.

## npm publish

```yaml
# .github/workflows/npm-publish.yml
name: npm publish
on:
  issue_comment:
    types: [created]

jobs:
  npm-publish:
    uses: tradeshift/actions-workflow-npm/.github/workflows/comment-npm-publish.yml@v1
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      npm-read-token: ${{ secrets.NPM_TOKEN }}
```

## Dependency tree auto-update
Shared workflow that auto-updates the dependency tree for the lock file of your node project.

By regenerating the lockfiles, the dependency tree will be updated to pull in the latest packages that match the dependency ranges in `package.json`. For each of those dependencies, the sub-dependencies are updated, and so on. 

None of these updates should be a breaking change, since they respect the [version ranges](https://semver.npmjs.com/). They'll potentially save you and your team a lot of time by preventing `Dependabot` vulnerability alerts that you'd have to deal with manually otherwise. 

The lock file will be generated using the `npm`/`yarn` version that matches the `node` version specified on the `.nvmrc` file for your project. The workflow also removes entries for `package-lock.json` or `yarn.lock` on your `.gitignore` file.

To enable it for a repo, create a new workflow with the following contents:

```yaml
# .github/workflows/dependency-auto-update.yml
name: Dependency tree auto-update
on:
  schedule:
    - cron:  '0 11 * * 1,4' # Frequency of your preference, this one runs Mondays and Thursdays at 11am
  workflow_dispatch: # Allow running manually
jobs:
  update:
    uses: tradeshift/actions-workflow-npm/.github/workflows/dependency-tree-update.yml@v1 # Reference to the shared workflow
    secrets:
      gpg-key: ${{ secrets.TRADESHIFTCI_GPG_KEY }} # The client key to use for commit author and signing
      github-token: ${{ secrets.GH_TOKEN }} # Token used to checkout code and create PR. Using a personal access token to have workflows run on the created PR.
      npm-token: ${{ secrets.NPM_TOKEN }} # Token used to authenticate to the private GitHub npm registry
    with:
      path: . # Optional paramater in case your application is not at the root of your repo, otherwise it defaults to "."
      # runs-on: self-hosted # Optional paramater to define where to run the workflow, otherwise it defaults to ubuntu-latest. More information at https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idruns-on
```

If you have multiple subfolders with managed node modules you can specify multiple folders on the same schedule like this:

```yaml
# .github/workflows/dependency-auto-update.yml
name: Dependency tree update
on:
  schedule:
    ...
jobs:
  update:
    uses: tradeshift/actions-workflow-npm/.github/workflows/dependency-tree-update.yml@v1
    secrets:
      ...
    with:
      path: . # Main path
  update-submodule:
    uses: tradeshift/actions-workflow-npm/.github/workflows/dependency-tree-update.yml@v1
    secrets:
      ...
    with:
      path: ./client/ # Module path
```
