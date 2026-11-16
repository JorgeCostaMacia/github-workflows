# Changelog

What a repository that calls these workflows has to act on, newest first. Versions that only touched the docs, the comments or the internals of a workflow are not listed.

## 1.1.0

`github-release` creates the GitHub Release with `gh` instead of `softprops/action-gh-release`, and takes the tag's section of the caller's `CHANGELOG.md` as its notes (GitHub's generated notes when there is none). Nothing to change in the call; a library whose CHANGELOG headings are not the bare version (`## 1.2.0`) gets generated notes until they are, and its paragraphs read best one per line, since a Release keeps every line break. The caller's tag filter can narrow to `v*.*.*`.
