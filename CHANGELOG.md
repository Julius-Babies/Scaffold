# Changelog

Every Scaffold version, newest first. *Migration* lists what a project has to do
by hand when upgrading to that version, beyond taking over the changed files.

## Unreleased

Support for stacked pull requests (`gh stack`), see *Stacked pull requests* in
`github/README.md`.

### Changed

- `github/`: pull requests are resolved to their issues through closing keywords
  in the description as well, since GitHub does not always link them for a pull
  request that targets another branch.
- `github/`: the release changelog finds its issues through the pull requests
  merged since the last release, matched by merge commit, instead of the commit
  subjects. Every layer of a stack is included, and pull request numbers in merge
  commit subjects are no longer taken for issues.
- `github/`: the deploy builds what all pull requests merged since the last
  successful deploy need, not just the last one. Before, a merged stack only
  looked at the labels of its bottom layer.
- `github/`: deploys run one at a time.
- `github/`: in a stack, a Feature's changelog entry is only required in the
  topmost layer that closes the issue.

### Migration

- None required. To use `gh stack`, enable stacked pull requests in the
  repository settings.

## v0.1

### Added

- `github/`: CI workflows taken over from Overmail (itself a superset of Trails):
  label-driven deploy with Docker image and Android release, changelog check,
  label sync between issues and pull requests. The project name and Docker image
  are parameters (`APP_NAME`, `DOCKER_IMAGE`).

### Migration

- Create `.scaffold/README.md` from `templates/scaffold-README.md` and add
  `templates/AGENTS-section.md` to the project's `AGENTS.md`.
- Set `APP_NAME` and `DOCKER_IMAGE` in `.github/workflows/deploy.yaml`.
- Trails: the Mapbox and Play signing inputs, the GitHub Packages auth and the
  `buildServerJar` task become deviations, see `github/README.md`.
