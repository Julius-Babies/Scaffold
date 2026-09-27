# Changelog

Every Scaffold version, newest first. *Migration* lists what a project has to do
by hand when upgrading to that version, beyond taking over the changed files.

## Unreleased

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
