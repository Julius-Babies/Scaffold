# AGENTS.md

Instructions for AI agents, both when working on Scaffold itself and when applying
it to a project. Read [README.md](README.md) first for the concepts; people read
[ONBOARDING.md](ONBOARDING.md).

## Rules

These hold for every task below.

- **A project is based on exactly one tag.** Always work from a tag
  (`vMAJOR.MINOR`), never from `main`. If you are asked for a version that is not
  tagged, stop and ask.
- **`.scaffold/README.md` records the base.** Every project has this file, with
  the version and a link to `https://github.com/Julius-Babies/Scaffold/tree/<tag>`,
  its parameters and its deviations, following
  [templates/scaffold-README.md](templates/scaffold-README.md).
- **Every deviation is documented.** Filling in the parameters a component README
  asks for is not a deviation. Any other difference between a Scaffold-derived
  file and the tag the project is based on is, and it belongs in the deviations
  table of `.scaffold/README.md` with its reason. A difference you cannot explain
  is not yours to keep or drop silently: ask.
- **Scaffold first.** Before changing a Scaffold-derived file in a project, decide
  whether the change would help every project. If it would, tell the user and
  propose it for Scaffold. Only change Scaffold itself when the user asks you to.
  If the change has to land in the project first, record it as a deviation with
  *Upstream* set to yes.
- **Components map to paths.** Each component directory maps to a path in the
  project, see the table in [README.md](README.md#components), e.g. `github/` →
  `.github/`. The component READMEs themselves are documentation for Scaffold and
  are not copied.

## Getting a version

Work from a clone with the full history, so tags can be compared:

```sh
git clone https://github.com/Julius-Babies/Scaffold /tmp/scaffold
git -C /tmp/scaffold tag --list              # available versions
git -C /tmp/scaffold checkout v0.2           # the version to apply
```

## Applying Scaffold to a project

For a project that is not based on Scaffold yet.

1. Check out the requested tag and read [CHANGELOG.md](CHANGELOG.md) and every
   component README at that tag.
2. For each component, compare the project's target path with the component.
   Files the project does not have yet are copied. Files it already has are
   merged: take Scaffold's version, then put back what the project genuinely
   needs.
3. Fill in the parameters each component README asks for.
4. List every remaining difference in the deviations table, with a reason. For
   each one, decide whether it should be upstream (see *Scaffold first*).
5. Create `.scaffold/README.md` from
   [templates/scaffold-README.md](templates/scaffold-README.md), and add the section from
   [templates/AGENTS-section.md](templates/AGENTS-section.md) to the project's
   `AGENTS.md` (create it if there is none).
6. Report to the user: which files changed, the deviations, the upstream
   candidates, and the manual steps from the component READMEs (secrets, labels,
   settings) that you could not do.

## Upgrading a project

From the version in `.scaffold/README.md` (`vOLD`) to the requested one (`vNEW`).

1. Read the [CHANGELOG.md](CHANGELOG.md) entries after `vOLD` up to and including
   `vNEW`, especially *Migration*.
2. Get Scaffold's changes and apply them to the project's target paths:

   ```sh
   git -C /tmp/scaffold diff vOLD vNEW -- github/
   ```

   Apply the diff rather than overwriting the files, so the documented deviations
   survive. Where a change collides with a deviation, resolve it by the reason
   given for that deviation.
3. Go through the deviations table. Remove the ones `vNEW` makes unnecessary, for
   example upstream candidates that Scaffold has taken over, and update the ones
   whose shape changed.
4. Before trusting the table, verify it: diff the project's files against
   `vNEW` (with parameters and documented deviations accounted for). Any other
   difference is undocumented, so report it.
5. Update version and link in `.scaffold/README.md`.
6. Report as when applying, including the manual migration steps.

## Working on Scaffold itself

- **Keep it generic.** No project names, project URLs or project-specific
  secrets in component files. Whatever a project has to decide becomes a
  parameter, documented in the component README with a placeholder default.
- **Document every change** under `Unreleased` in [CHANGELOG.md](CHANGELOG.md).
  Anything a project has to do by hand when upgrading goes under *Migration*.
- **Keep the component README in step with its files**: layout, parameters,
  secrets and conventions it expects from a project.
- **New components** get a directory, a README and a row in the component table
  of [README.md](README.md), and a line in the file list of
  [templates/scaffold-README.md](templates/scaffold-README.md) and
  [templates/AGENTS-section.md](templates/AGENTS-section.md).
- **Do not tag or push releases** unless the user asks; the procedure is in
  [ONBOARDING.md](ONBOARDING.md#releasing-a-scaffold-version).
