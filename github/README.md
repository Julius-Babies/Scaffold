# CI workflows

GitHub Actions setup shared by Trails and Overmail. Maps to `.github/` in the
project (this README is not copied). How to apply and upgrade it is described in
[AGENTS.md](../AGENTS.md).

```
.github/
├── workflows/
│   ├── deploy.yaml          Builds and releases on every push to main
│   ├── changelog.yaml       Checks the changelog entries of a pull request
│   └── sync-labels.yaml     Keeps project:* labels in sync between issues and PRs
├── actions/
│   └── setup-android-project/action.yaml   JDK, Gradle, Android SDK, signing
├── check_changelog.main.kts                Used by changelog.yaml
└── generate_changelog.main.kts             Used by deploy.yaml
```

## Expected project layout

The workflows assume the repository layout Trails and Overmail share:

| Path | Purpose |
| --- | --- |
| `gradlew` | Gradle wrapper at the root |
| `:server` | Ktor server, `:server:buildFatJar` produces `server/build/libs/server-all.jar` (`ktor { fatJar { archiveFileName.set("server-all.jar") } }`) |
| `web/` | SvelteKit app built with Bun, `bun run build` writes to `web/build` |
| `Dockerfile` | At the root, copies `server-all.jar` and `web_build/` from the build context |
| `:app:android` | Android app, `assembleRelease` writes to `app/android/build/outputs/apk/release/` |
| `docs/changelog/issues/<issue>/changelog.<type>.json` | Changelog entries, see [Changelog](#changelog) |

## Labels and issue conventions

Everything is driven by `project:*` labels and GitHub issue types. Create these
labels in the target repository:

| Label | Meaning |
| --- | --- |
| `project:app` | Touches the mobile app. Builds the APKs, creates a release, listed in the changelog. |
| `project:server` | Touches the Ktor server. Rebuilds the Docker image. |
| `project:webapp` | Touches the SvelteKit app. Rebuilds the Docker image (it ships in the same image). |
| `project:other` | Documentation, CI, tooling. Deliberately builds nothing. |

Issue types (organisation setting) used by the changelog: `Feature`, `Bug`, `Task`.

Issues are linked to pull requests through a closing reference (`Closes #12`) in
the PR description. If there is none, the issue number is taken from the branch
name: `feat/12-some-title` or `12-some-title`.

Commit subjects on main must contain the issue number (`feat(app #12): …`),
because the release changelog is collected from the commits since the last release.

## Workflows

### `deploy.yaml`

Runs on every push to `main` and manually via *Run workflow*.

1. **setup** – generates the build tag (`YYYYMMDD_HHMM`), timestamp and display
   date, and reads the `project:*` labels of the merged pull request to decide what
   to build. Without any labels (direct push, manual run) everything is built. A
   pull request labelled only `project:other` builds nothing; one without any
   deciding label also builds nothing, but warns.
2. **build-server** / **build-web** / **build-docker** – on `project:server` or
   `project:webapp`: builds the fat jar and the web app, then builds the Docker image
   and pushes it as `$DOCKER_IMAGE:latest` to GHCR.
3. **build-android** – on `project:app`: builds the signed release APKs. The build
   gets `BUILD_TAG` and `BUILD_TIMESTAMP` as environment variables so the app can
   embed its version.
4. **create_release** – only when the APKs were built (a release is the app's
   delivery vehicle). Generates the changelog, and creates a GitHub release
   `v<build tag>` with the APKs, `changelog*.json` and `CHANGELOG.md` as the body.

A manual run defaults to a **draft**: the Docker image is built but not pushed, and
the release is created as a draft that is not marked as latest. Useful to rehearse
a release.

### `changelog.yaml`

Runs on every pull request. `check_changelog.main.kts` checks, for every issue the
pull request closes, that the changelog entry exists and has the right shape. The
result is posted as a pull request comment (replacing the previous one) and written
to the step summary. The check fails on broken or misnamed entries and on a missing
entry for a feature; everything else is a warning.

Issues that carry neither `project:app` nor an entry directory need no changelog.

### `sync-labels.yaml`

Copies `project:*` labels between an issue and the pull requests that close it, in
both directions, so it does not matter which one gets labelled. A single
label/unlabel is mirrored exactly; opening, reopening or editing a pull request
brings both sides up to the union of their labels. It cannot loop: events caused by
`GITHUB_TOKEN` do not trigger workflows, and every write is skipped if the target
already agrees.

## Changelog

Each issue that should appear in the release notes gets one file under
`docs/changelog/issues/<issue>/`, named after its issue type:

| Issue type | File | Fields |
| --- | --- | --- |
| Feature | `changelog.feature.json` | `title` and `description`, required |
| Bug | `changelog.bug.json` | `description` only, entry optional |
| Task | `changelog.task.json` | `description` optional, entry optional |

Any field can be translated under `localized`, missing fields fall back to the default (English):

```json
{
  "title": "Download emails",
  "description": "You can download emails as an .eml file now.",
  "localized": {
    "de": {
      "title": "E-Mails herunterladen",
      "description": "Du kannst E-Mails jetzt als .eml-Datei herunterladen."
    }
  }
}
```

`generate_changelog.main.kts` collects all issues referenced in the commit subjects
since the last release, keeps those labelled `project:app`, and writes to
`build/changelog/`:

- `changelog.json` (English) and `changelog.<language>.json` for every language
  found, grouped into `features`, `fixes` and `tasks` and keyed by issue number.
  These are attached to the release so the app can read them.
- `CHANGELOG.md` in English, used as the release body.

Both scripts run locally as well, e.g. `kotlin .github/check_changelog.main.kts` on
a feature branch, or `kotlin .github/generate_changelog.main.kts v20260101_1200`.
They need `kotlin` and `gh` on the path.

## Parameters

The values a project fills in. They go into the parameters table of the project's
`.scaffold/README.md`, everything else that differs is a deviation.

| Parameter | Where | Meaning |
| --- | --- | --- |
| `APP_NAME` | `env` in `workflows/deploy.yaml` | Names the release and its APKs |
| `DOCKER_IMAGE` | `env` in `workflows/deploy.yaml` | Image pushed to GHCR, lowercase, without tag |
| Server JDK | `java-version` in `build-server` of `workflows/deploy.yaml` | Must match the toolchain of `:server` |
| Android JDK | `java-version` in `actions/setup-android-project` | JDK the Android build runs on |

## Adopting it in a project

1. Take over this directory as `.github/`, see [AGENTS.md](../AGENTS.md).
2. Fill in the [parameters](#parameters).
3. Create the `project:*` labels and enable the issue types.
4. Add the repository secrets:

   | Secret | Used for |
   | --- | --- |
   | `KEYSTORE_FILE` | Base64 encoded release keystore (`base64 -i keystore.jks`) |
   | `KEYSTORE_PASSWORD` | Store and key password, key alias is `key0` |

   `GITHUB_TOKEN` covers GHCR, releases, labels and comments; no further setup needed.
5. Have the Android build read its signing config from `local.properties`
   (`signing.default.file`, `signing.default.storepassword`,
   `signing.default.keyalias`, `signing.default.keypassword`), which the setup
   action writes.

### Known deviations

Examples of deviations projects need, and how they are built. Each one belongs in
the deviations table of the project's `.scaffold/README.md`.

Extra secrets for the Android build go into `actions/setup-android-project` as
inputs and are passed in from `build-android`. Trails, for example, adds:

- `mapbox_public_token` → `mapbox.public-token` in `local.properties`
- `mapbox_secret_token` → `mapbox.token` in `~/.gradle/gradle.properties`
- `play_developer_signing_id` → appended to `app/android/src/main/assets/adi-registration.properties`

Trails also pulls private packages from GitHub Packages. The server build then
passes `-Pmaven.pkg.github.com.user=${{ secrets.GH_USERNAME }}` and
`-Pmaven.pkg.github.com.token=${{ secrets.GH_REGISTRY_TOKEN }}` to Gradle, and
`build-web` writes an `.npmrc` before `bun install`:

```yaml
- name: Set up GitHub Packages npm auth
  run: |
    echo "@Julius-Babies:registry=https://npm.pkg.github.com" > .npmrc
    echo "//npm.pkg.github.com/:_authToken=${{ secrets.GH_REGISTRY_TOKEN }}" >> .npmrc
    echo "engine-strict=true" >> .npmrc
```
