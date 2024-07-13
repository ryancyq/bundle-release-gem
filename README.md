# Bundle and Release Gem GitHub Action

This action allows you to release ruby gem via [rubygems/release-gem] with an out of date branch.

## Scenario

Given the scenario in the continous integration environment, you may want to release a new version of the gem after passing the test suite.

The following events occurred in sequence:
1. git commit triggers release workflow.
2. workflow runs test suite + releas gem action.
3. another git commit pushed to the same remote branch.
4. release gem action from point 2 executed.
5. failed to due `git push` release tag together with `git push` current HEAD to remote.

Currently, [rubygems/release-gem@v1] is unable to support the scenario above as described in the [bundler issue] and [rubygem release action issue].

## Workaround

To workaround the limitation, this action will create temporary branch (`releases/vX.X.X`) in the remote.
Then trigger [rubygems/release-gem] to allow `rake release` to push release tag and current HEAD to remote.
Temporary branch will be deleted from remote as part of the cleanup step in the action.

## Usage

```yaml
# .github/workflows/release.yml
jobs:
  rubygems:
    runs-on: ubuntu-latest

    permissions:
      id-token: write # mandatory for trusted publishing
      contents: write # mandatory for pushing the release branch/tag on git

    steps:
      - uses: actions/checkout@v4
      - uses: ruby/setup-ruby@v1
        with:
          bundler-cache: true
          ruby-version: ruby

      - uses: ryancyq/bundle-release-gem@v1
        with:
          git-remote: "origin"                  # default to "origin"
          git-release-branch-prefix: "releases" # default to "releases"
          release-await: false                  # default to false, rubygems/release-gem "await-release" option
          release-trusted: true                 # default to true, rubygems/release-gem "setup-trusted-publisher" option
```

### Compatabality with [rubygems/release-gem]

- All inputs in [rubygems/release-gem] are supported:
  - `await-release`
  - `setup-trusted-publisher`
- `git-remote` will not be propagated to [rubygems/release-gem] as it is not supported.

[rubygems/release-gem]: https://github.com/rubygems/release-gem/
[rubygems/release-gem@v1]: https://github.com/rubygems/release-gem/tree/v1
[bundler issue]: https://github.com/rubygems/rubygems/issues/7818
[rubygem release action issue]: https://github.com/rubygems/release-gem/issues/8
