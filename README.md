# Scaffold
Collection of configs and CI workflows used in my applications to quickly build and deploy webapps with an api and mobile clients

Scaffold is the common base of those applications. It is versioned with git tags,
and every project records which version it is built on. Applying or upgrading is
usually done by an AI agent: *"Apply Scaffold v0.2 to Trails."*

- New here? Read [ONBOARDING.md](ONBOARDING.md).
- Agents: follow [AGENTS.md](AGENTS.md).
- What changed between versions: [CHANGELOG.md](CHANGELOG.md).

## Components

Each component is a directory in this repository that maps to a path in the
target repository. Its own README describes what it expects from the project and
which values the project has to fill in.

| Component | Target path | Contents |
| --- | --- | --- |
| [`github/`](github/README.md) | `.github/` | CI workflows for building, releasing and changelog handling |

Components are kept outside of their target paths here (`github/`, not
`.github/`), so they do not act on this repository itself.

## Principles

**Versions.** A version is a git tag `vMAJOR.MINOR` on this repository, e.g.
`v0.2`. A project is always based on exactly one tag, never on a branch. Every
version is described in [CHANGELOG.md](CHANGELOG.md), including what a project
has to do by hand when upgrading to it.

**Every project records its base.** The project has a `.scaffold/README.md`
naming the version with a link to its tag, see
[templates/scaffold-README.md](templates/scaffold-README.md).

**Every deviation is documented.** Projects will always need their own
peculiarities. Filling in the parameters a component asks for is not a
deviation. Anything else that differs from the Scaffold version the project is
based on is, and it is listed in the deviations table of `.scaffold/README.md`,
with a reason. Whatever differs from Scaffold without being listed there is a bug.

**Scaffold first.** Before a Scaffold-derived file is changed in a project, the
question is whether the change belongs here instead. Fixes and improvements that
would help every project go into Scaffold and reach the project through an
upgrade. Only what is genuinely specific to the project stays a deviation.
