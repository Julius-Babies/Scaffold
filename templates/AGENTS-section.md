<!--
Goes into the AGENTS.md of every project based on Scaffold, as its own section,
so agents working on the project follow the rules without having to open Scaffold.
-->

## Scaffold

This project is based on [Scaffold](https://github.com/Julius-Babies/Scaffold);
[.scaffold/README.md](.scaffold/README.md) names the version and lists every
deviation from it. Files derived from Scaffold: `.github/`.

When changing one of these files:

1. **Check whether the change belongs in Scaffold.** If it would help every
   project based on Scaffold (a fix, a better check, a new convention), tell the
   user and propose making it in Scaffold instead. Only make it here if the user
   agrees or it is genuinely specific to this project.
2. **Document it.** Every difference from the Scaffold version goes into the
   deviations table of `.scaffold/README.md`, with its reason, in the same change.
   Mark it as an upstream candidate if it should move to Scaffold later.
3. **Never change the Scaffold version by hand.** Upgrading follows the procedure
   in Scaffold's AGENTS.md.
