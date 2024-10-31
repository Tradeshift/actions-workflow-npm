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
