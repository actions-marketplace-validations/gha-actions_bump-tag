<div align="center">
	<h1>Bump Tag</h1> 
	<br>
	<p>A Github Action to automatically bump and tag master, on merge, with the latest SemVer formatted version.</p>
	<a href="https://github.com/gha-actions/bump-tag/actions"><img src="https://github.com/gha-actions/bump-tag/workflows/Bump%20version/badge.svg?branch=main" alt="Build status"></a>
<!-- 	<a href="./coverage/lcov-report/index.html"><img src="https://img.shields.io/coveralls/github/jaynath-d/cc-suite" alt="Coverage status"></a> -->
	<a href="https://img.shields.io/github/v/tag/gha-actions/bump-tag"><img src="https://img.shields.io/github/v/tag/gha-actions/bump-tag" alt="Stable version"></a>
	<a href="https://img.shields.io/github/v/release/gha-actions/bump-tag?color=%233D9970"><img src="https://img.shields.io/github/v/release/gha-actions/bump-tag?color=%233D9970" alt="Stable version"></a>
</div>

___

## Usage

_Note: We don't recommend using the @master version unless you're happy to test the latest changes._

```yaml
# example 1: on push to master
name: Bump version
on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-22.04
    permissions:
      contents: write
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: '0'

    - name: Bump version and push tag
      uses: gha-actions/bump-tag@0.2.0 # Don't use @master or @v1 unless you're happy to test the latest version
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # if you don't want to set write permissions use a PAT token
        WITH_V: false
```

```yaml
# example 2: on merge to master from pull request (recommended)
name: Bump version
on:
  pull_request:
    types:
      - closed
    branches:
      - master

jobs:
  build:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-22.04
    permissions:
      contents: write
    steps:
    - uses: actions/checkout@v3
      with:
        ref: ${{ github.event.pull_request.merge_commit_sha }}
        fetch-depth: '0'

    - name: Bump version and push tag
      uses: gha-actions/bump-tag@0.2.0 # Don't use @master or @v1 unless you're happy to test the latest version
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # if you don't want to set write permissions use a PAT token
        WITH_V: true
        PRERELEASE: true

```

Depending if you choose example 1 or example 2 is how crafted version bumps operate when reading the commit log.

Is recommended to use on `pull_request` instead of on commit to master/main.

_NOTE: set the fetch-depth for `actions/checkout@v2` or newer to be sure you retrieve all commits to look for the semver commit message._

### Options

**Environment Variables**

- **GITHUB_TOKEN** **_(required)_** - Required for permission to tag the repo.
- **DEFAULT_BUMP** _(optional)_ - Which type of bump to use when none explicitly provided (default: `minor`).
- **DEFAULT_BRANCH** _(optional)_ - Overwrite the default branch its read from Github Runner env var but can be overwritten (default: `$GITHUB_BASE_REF`). Strongly recommended to set this var if using anything else than master or main as default branch otherwise in combination with history full will error.
- **WITH_V** _(optional)_ - Tag version with `v` character.
- **RELEASE_BRANCHES** _(optional)_ - Comma separated list of branches (bash reg exp accepted) that will generate the release tags. Other branches and pull-requests generate versions postfixed with the commit hash and do not generate any tag. Examples: `master` or `.*` or `release.*,hotfix.*,master` ...
- **CUSTOM_TAG** _(optional)_ - Set a custom tag, useful when generating tag based on f.ex FROM image in a docker image. **Setting this tag will invalidate any other settings set!**
- **SOURCE** _(optional)_ - Operate on a relative path under $GITHUB_WORKSPACE.
- **DRY_RUN** _(optional)_ - Determine the next version without tagging the branch. The workflow can use the outputs `new_tag` and `tag` in subsequent steps. Possible values are `true` and `false` (default).
- **GIT_API_TAGGING** _(optional)_ - Set if using git cli or git api calls for tag push operations. Possible values are `false` and `true` (default).
- **INITIAL_VERSION** _(optional)_ - Set initial version before bump. Default `0.0.0`. MAKE SURE NOT TO USE vX.X.X here if combined WITH_V
- **TAG_CONTEXT** _(optional)_ - Set the context of the previous tag. Possible values are `repo` (default) or `branch`.
- **PRERELEASE** _(optional)_ - Define if workflow runs in prerelease mode, `false` by default. Note this will be overwritten if using complex suffix release branches. Use it with checkout `ref: ${{ github.sha }}`
- **PRERELEASE_SUFFIX** _(optional)_ - Suffix for your prerelease versions, `beta` by default. Note this will only be used if a prerelease branch.
- **VERBOSE** _(optional)_ - Print git logs. For some projects these logs may be very large. Possible values are `true` (default) and `false`.
- **MAJOR_STRING_TOKEN** _(optional)_ - Change the default `#major` commit message string tag.
- **MINOR_STRING_TOKEN** _(optional)_ - Change the default `#minor` commit message string tag.
- **PATCH_STRING_TOKEN** _(optional)_ - Change the default `#patch` commit message string tag.
- **NONE_STRING_TOKEN** _(optional)_ - Change the default `#none` commit message string tag.
- **BRANCH_HISTORY** _(optional)_ - Set the history of the branch for finding `#bumps`. Possible values `last`, `full` and `compare` defaults to `compare`.
  - `full`: attempt to show all history, does not work on rebase and squash due missing HEAD [should be deprecated in v2 is breaking many workflows]
  - `last`: show the single last commit
  - `compare`: show all commits since previous repo tag number

### Outputs

- **new_tag** - The value of the newly created tag.
- **tag** - The value of the latest tag after running this action.
- **part** - The part of version which was bumped.

### Bumping

**Manual Bumping:** Any commit message that includes `#major`, `#minor`, `#patch`, or `#none` will trigger the respective version bump. If two or more are present, the highest-ranking one will take precedence.
If `#none` is contained in the merge commit message, it will skip bumping regardless `DEFAULT_BUMP`.

**Automatic Bumping:** If no `#major`, `#minor` or `#patch` tag is contained in the merge commit message, it will bump whichever `DEFAULT_BUMP` is set to (which is `minor` by default). Disable this by setting `DEFAULT_BUMP` to `none`.

> **_Note:_** This action **will not** bump the tag if the `HEAD` commit has already been tagged.

### Workflow

- Add this action to your repo
- Commit some changes
- Either push to master or open a PR
- On push (or merge), the action will:
  - Get latest tag
  - Bump tag with minor version unless the merge commit message contains `#major` or `#patch`
  - Pushes tag to github
  - If triggered on your repo's default branch (`master` or `main` if unchanged), the bump version will be a release tag.
  - If triggered on any other branch, a prerelease will be generated, depending on the bump, starting with `*-<PRERELEASE_SUFFIX>.1`, `*-<PRERELEASE_SUFFIX>.2`, ...
