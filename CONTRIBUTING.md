# Contributing

Unlike the library repositories, this one does **not** follow GitFlow: there is nothing to build or test here, so a
`develop` branch and release branches would add steps without adding a check. Everything is committed on `main`, the
only branch, and a release is a tag on one of its commits.

Remote is always `origin`.

## Versions

There is no version file: the version is the tag.

- **`vX.Y.Z`** is the release: annotated, message `Release vX.Y.Z`, on a commit of `main`. Pushing it creates its
  GitHub Release (`release.yml`). It never moves: a mistake is fixed with the next version.
- **`vX`**, the major tag, is the one callers pin (`@v1`). Every release moves it to the commit of its `vX.Y.Z`; it is
  the only tag that is ever moved, and the only push that needs `--force`.

A release is a minor or a patch when every caller keeps working (a fix, a new optional input, a comment), and a new
major when one would break (a renamed or removed workflow, input or output, a new required input). A new major gets
its own major tag, and the previous one stays where it was until every repo has moved off it.

## Releasing

```bash
# the change is committed on main
git checkout main && git pull

# tag the release and push it with main
git tag -a v1.1.0 -m "Release v1.1.0"
git push origin main v1.1.0

# move the major tag to the release, pointing at its commit (not at the v1.1.0 tag)
git tag -fa v1 -m "v1" "v1.1.0^{}"
git push origin v1 --force
```

## Checking

There is nothing to build: a workflow is checked with [actionlint](https://github.com/rhysd/actionlint), which also
runs shellcheck on every `run:` script.

```bash
docker run --rm -v "$PWD:/repo" -w /repo rhysd/actionlint:1.7.7 -no-color
```

Beyond that, a workflow only really runs when a repository calls it: the first run of a caller's Develop after a
release is its test, since Develop builds and pushes nothing.
