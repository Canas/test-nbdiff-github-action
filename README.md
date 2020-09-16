This repo is to try out a Github action that comments PRs with Jupyter Notebook diffs (vía [nbdime](https://github.com/jupyter/nbdime)) if available. Sample is available in the only open PR.

nbdiff.yaml
```
name: Generate notebook diff

on: ["pull_request"]

jobs:
  check-diff:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Fetch target branch
        run: git fetch origin ${{ github.event.pull_request.base.ref }}:${{ github.event.pull_request.base.ref }}

      - name: Setup Python
        uses: actions/setup-python@v1
        with:
          python-version: "3.6"

      - name: Install requirements
        run: pip3 install nbdime

      - name: Run and store diff
        run: |
          nbdiff ${{ github.event.pull_request.base.ref }} --no-color > diff.log
          sed -i '1s/^/```diff\n&/' diff.log
          sed -i '$s/$/\n&```/' diff.log

      - name: Get comment body
        id: get-comment-body
        run: |
          body=$(cat diff.log)
          body="${body//'%'/'%25'}"
          body="${body//$'\n'/'%0A'}"
          body="${body//$'\r'/'%0D'}"
          echo ::set-output name=body::$body

      - name: Create comment
        uses: peter-evans/create-or-update-comment@v1
        with:
          issue-number: ${{ github.event.pull_request.number }}
          body: ${{ steps.get-comment-body.outputs.body }}

```
