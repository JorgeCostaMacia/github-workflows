# github-workflows — working in this repo

The **reusable GitHub Actions workflows** the `JorgeCostaMacia.*` .NET library repos call from their own Develop, Main and Release. Nothing runs here on its own: every workflow is `on: workflow_call`, and it runs in the caller's context, with the caller's permissions and token.

## What belongs here

A workflow that more than one library repo repeats, and that only differs between them by what can be an input. What is particular to one repo stays in that repo's own workflow.

## Layout

All in `.github/workflows/`, flat, since GitHub does not allow reusable workflows in subfolders. Names read `<stack>-<action>-<variant>`, so each family sorts together:

- `dotnet-build-nuget` — convention guards, restore, build, format check, test and pack of a library repo, as the `nuget` artifact.
- `github-release` — the GitHub Release of the pushed `vX.Y.Z` tag, with `gh`: its notes are the tag's section of the caller's `CHANGELOG.md`, or GitHub's generated notes when there is none. Nothing attached.

One file beside them is this repository's own (`on: push`, not `workflow_call`):

- `release.yml` — the GitHub Release of each `vX.Y.Z` tag, by calling `github-release` from the same commit; never of the major tag, which moves on every release.

`assets/` holds the icon and the social preview; `social-preview.png` is rendered from `social-preview.svg`, and is what the README shows and what is uploaded as the repo's social preview.

## The design, and why

- **The push to nuget.org stays in the caller.** The Trusted Publishing policy on nuget.org names the repository and its workflow file, and the docs say nothing about a login made inside a reusable workflow. The caller's `main.yml` keeps `NuGet/login` and the push (it downloads the `nuget` artifact), so the policy names `main.yml` as before, and a workflow of this repo never gets `id-token: write`: whoever moves a tag here cannot mint a publishing key.
- **nuget.org only.** The libraries consume nothing private, so there is no feed credential to pass.
- **Self-documented.** Each workflow's header says what it does, how to call it and what the calling repo needs, and each step explains what the code does not say. No separate docs.
- **Inputs are not re-validated.** A wrong solution name fails on its own, with a clear enough error.

## Versioning

The version is the tag. Callers pin the major tag (`@v1`), which every release moves to its `vX.Y.Z` commit; a `vX.Y.Z` tag never moves. Renaming or removing a workflow, input or output, or adding a required input, is a new major. What a release asks of its callers goes in [CHANGELOG.md](CHANGELOG.md), on the release branch. The steps are in [CONTRIBUTING.md](CONTRIBUTING.md).

The `protect-version-tags` ruleset covers `v*`: no deletion and no force push, with the repository admin as the only bypass, which is what lets the major tag move.

## CI

Only `release.yml`: there is nothing to build or check. A workflow is only truly tried when a caller runs it, so before every release a library's feature branch pins `@develop` and runs the change (CONTRIBUTING.md).

## Style, Git

- Workflows, comments and docs in English. LF line endings (`.gitattributes`, `.editorconfig`).
- Workflows up to 120 columns, except a command or an error message that reads better whole. Markdown follows the `code-style-md` skill: one paragraph per line, never wrapped by hand.
- Secrets and inputs reach scripts as environment variables, never pasted into the script.
- Actions pinned by major (`actions/checkout@v7`); never an action that downloads code at run time. What `gh`, already on the runner, does in a line (a Release) takes no third-party action.
- GitFlow, like the library repos, plus moving the major tag after each release finish (CONTRIBUTING.md).
- Commits with the repo's own identity (`git config --local`), the personal one.

## Relevant skills

- `gitflow`: branches, releases and tags.
- `changelog`: when a release needs an entry, and its format (the Release takes it as its notes).
- `code-style-md`: how every `.md` here is written.
