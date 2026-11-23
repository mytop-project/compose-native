# The mytop branch

`main` of this fork is upstream's, kept as it is so it can be synced from brahmkshatriya/compose-native.
Our patches live on the `mytop` branch, on top of a point of `main`, and are published under our own version
(`MYTOP_VERSION`, for example `1.13.0-beta01-mytop.1`): never to Maven Central and never under upstream's
version numbers.

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

upstream's `publish-compose-native-central.yml` is removed on this branch so that no tag here can start a
Maven Central publication.
