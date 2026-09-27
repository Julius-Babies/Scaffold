# Onboarding

This is for people. Agents follow [AGENTS.md](AGENTS.md), which covers the same
ground as step-by-step procedures.

## The idea

Trails, Overmail and future projects share the same CI, the same release flow and
the same conventions. Instead of copying these between projects and letting them
drift apart, they live here, in one place, with version numbers. A project says
"I am on Scaffold v0.2", and that one sentence tells you what its CI does.

Projects still differ. That is fine, as long as each difference is written down in
the project's `.scaffold/README.md` together with the reason for it.

## Reading a project

Open `.scaffold/README.md` in the project:

```markdown
# Scaffold

This project is based on [Scaffold v0.2](https://github.com/Julius-Babies/Scaffold/tree/v0.2).

## Parameters
…the values the Scaffold components ask the project to fill in…

## Deviations
…everything else that differs from Scaffold v0.2, and why…
```

To understand a Scaffold-derived file in the project, read the component README
at that tag, then the deviations. Nothing else should differ.

## Starting a new project

1. Ask an agent to *"apply Scaffold vX.Y"* to the repository, or follow
   [Applying Scaffold](AGENTS.md#applying-scaffold-to-a-project) yourself.
2. Do the manual steps from the component READMEs: repository secrets, labels,
   issue types.
3. Check that the project has a `.scaffold/README.md` and that its `AGENTS.md` has
   the Scaffold rules, both from [templates/](templates/).

## Upgrading a project

Ask an agent to *"upgrade to Scaffold vX.Y"*. Before merging, read the
[CHANGELOG.md](CHANGELOG.md) entries between the old and the new version: some
upgrades need manual steps (new secrets, new labels) that no pull request can do.

## Changing something

In a project, a change to a Scaffold-derived file falls into one of two cases:

- **It would help every project** (a bug fix, a better check, a new convention):
  make it in Scaffold, release a new version, upgrade the project. If it cannot
  wait, make it in the project as a deviation marked as an upstream candidate,
  and move it to Scaffold later.
- **Only this project needs it** (an extra secret, a different module name):
  make it in the project and add it to the deviations table.

## Releasing a Scaffold version

1. Move the `Unreleased` entries in [CHANGELOG.md](CHANGELOG.md) under a new
   version heading. List any manual steps under *Migration*.
2. Commit, then tag and push:

   ```sh
   git tag v0.2
   git push origin v0.2
   ```

Bump `MINOR` for every release. `MAJOR` goes to 1 once the structure has settled;
until then, *Migration* is what tells you whether an upgrade needs care.
Tags are never moved or deleted once pushed: projects refer to them.
