# The mytop branch

`main` of this fork is upstream's, kept as it is so it can be synced from brahmkshatriya/compose-native.
Our patches live on the `mytop` branch, on top of a point of `main`, and are published under our own version
(`MYTOP_VERSION`, for example `1.13.0-beta01-mytop.1`): never to Maven Central and never under upstream's
version numbers. `mytop` is the fork's default branch, because a scheduled workflow runs only from the default
branch and `main` must stay an exact copy of upstream's.

## What is patched

- `compose/desktop/desktop-native/src/nativeInterop/cinterop/drag_support.cpp`: X11 window ids are read from
  the SDL window again before each X request. SDL can give an `SDL_Window` a new X window (for example when a
  renderer is created on a window made for OpenGL); the drag manager kept the id from `kdrag_create`, so
  closing the window ended in `BadWindow (X_DeleteProperty)` and exit code 1. The window shadow's removal had
  the same cached id.

## How a build is consumed

`.github/workflows/mytop-publish.yml` (a push to `mytop`) builds the desktop-native closure for linuxX64,
linuxArm64 and mingwX64 with the same shards as upstream's release workflow, assembles one Maven repository
and attaches it to a GitHub release `v<MYTOP_VERSION>` of this fork:

- `compose-native-maven-<version>.tar.gz`: the repository (`dev/brahmkshatriya/...`), including the Gradle plugin
- `compose-native-maven-<version>.tar.gz.sha256`

A consumer downloads both, checks the sha256, unpacks the archive to a folder and adds it as a file
repository for the group `dev.brahmkshatriya.*`, with every `dev.brahmkshatriya` dependency and the
`dev.brahmkshatriya.compose` plugin at `<version>`. Everything else (native Skiko, JetBrains artifacts)
still resolves from Maven Central as upstream's documentation says. A released version is never replaced;
bump `MYTOP_VERSION` for a new build.

Upstream's `publish-compose-native-central.yml` is removed on `mytop` so that no tag here can start a
Maven Central publication.

## Staying current with upstream

`.github/workflows/mytop-sync.yml` runs every day (and on request, Actions > Sync upstream into mytop > Run
workflow). It never rewrites history, never pushes to `mytop` and never changes `MYTOP_VERSION`:

1. `main` is fast-forwarded from upstream's `main`.
2. If upstream has commits `mytop` lacks, a branch `sync/<date>` is made from `mytop` and upstream's `main` is
   merged into it (a merge commit). If that does not merge cleanly, no branch is left and the run summary says
   so.
3. A clean branch is built with the same pipeline as a push to `mytop` (without a release).
4. The run summary names the branch and whether its build passed.

Nothing is opened publicly. Run the workflow with "Make a sync branch and build it even when upstream has
nothing new" to check the pipeline.

The workflow cannot push changes to files under `.github/workflows` (a `GITHUB_TOKEN` has no such
permission). If upstream changes one, the merge is refused and the summary says so; merge `main` into `mytop`
by hand then.

## Taking an update (the person's steps)

1. Open the run summary of the sync. If the build of `sync/<date>` passed, continue; if not, look at the
   failed job first.
2. Check what changed upstream: `git log mytop..sync/<date>` on a checkout, or compare `mytop...sync/<date>` on
   GitHub. Our patch (`drag_support.cpp`) is the part to watch.
3. On the sync branch, change `MYTOP_VERSION` to the next suffix: `1.13.0-beta01-mytop.2` for a new build of the
   same upstream release, or `<new upstream version>-mytop.1` when the sync moved upstream's version (they are
   in `gradle.properties`: `compose.native.upstream.version.COMPOSE` and the others). Never reuse a released
   version.
4. Merge the branch into `mytop` (a fast-forward or a merge commit; never a rebase or a force push). The push
   builds and, when it passes, attaches the repository to release `v<MYTOP_VERSION>`.
5. In each consumer, change the version and the sha256 it pins to the new release (mytop-ui's spike workflow:
   `CN_VERSION` and `CN_SHA256`).
6. Delete the `sync/<date>` branch when it is merged.

A conflict in a sync: merge by hand in a checkout (`git fetch origin; git checkout -b sync/<date> origin/mytop;
git merge origin/main`), resolve, and continue from step 3. If upstream fixed what our patch fixes, drop the
patch in that merge and say so in its message.
