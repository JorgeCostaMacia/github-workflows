# Contributing

This repository follows GitFlow, like the library repositories (the `gitflow` skill): work lands on `develop` through
`feature/` and `bugfix/` branches, and a version leaves through a `release/X.Y.Z` branch merged into `develop` and
`main` and tagged `vX.Y.Z`. The one step of its own is moving the major tag, below.

Remote is always `origin`.

## Versions

There is no version file: the version is the tag.

- **`vX.Y.Z`** is the release: annotated, on the `main` merge of its release branch. Pushing it creates its GitHub
  Release (`release.yml`). It never moves: a mistake is fixed with the next version.
- **`vX`**, the major tag, is the one callers pin (`@v1`). Every release moves it to the commit of its `vX.Y.Z`; it is
  the only tag that is ever moved, and the only push that needs `--force`.

A release is a minor or a patch when every caller keeps working (a fix, a new optional input, a comment), and a new
major when one would break (a renamed or removed workflow, input or output, a new required input). A new major gets
its own major tag, and the previous one stays where it was until every repo has moved off it.

## Changelog

[CHANGELOG.md](CHANGELOG.md) says what a calling repository has to act on (the `changelog` skill): every major, and a
minor that changes what a caller gets or must provide. It is written on the release branch, one paragraph per line;
the tag's Release takes that section as its notes. A version that only touches the docs, the comments or the
internals has no entry.

## Releasing

```bash
# from develop, with the work merged
git checkout -b release/1.2.0 develop
# write the CHANGELOG.md entry, if the version asks something of its callers, and commit it

# the GitFlow finish: develop, then main, the tag on main, one atomic push
git checkout develop && git merge --no-ff --no-edit release/1.2.0
git checkout main && git merge --no-ff --no-edit release/1.2.0
git tag -a v1.2.0 -m "Release v1.2.0"
git push --atomic origin main develop v1.2.0
git branch -d release/1.2.0

# move the major tag to the release, pointing at its commit (not at the v1.2.0 tag)
git tag -fa v1 -m "v1" "v1.2.0^{}"
git push origin v1 --force
```

## Trying a change before releasing it

A workflow only really runs when a repository calls it. To try one before the release, a library's feature branch
points at this repository's branch instead of the major tag (`uses: …/dotnet-build-nuget.yml@develop`, or
`@feature/<name>`), and its Develop runs it; the pin goes back to `@v1` before that branch is merged.

## Checking

There is nothing to build: a workflow is checked with [actionlint](https://github.com/rhysd/actionlint), which also
runs shellcheck on every `run:` script. Develop runs it on every push to `develop` and every pull request; locally:

```bash
docker run --rm -v "$PWD:/repo" -w /repo rhysd/actionlint:1.7.7 -no-color
```
