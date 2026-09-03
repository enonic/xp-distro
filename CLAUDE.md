# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is **not** an application source repo — it's a Gradle-based *packaging/distribution* project for Enonic XP.
It has no Java/Kotlin source code and no test suite. Its job is to assemble a runnable distribution by:

1. Downloading pre-built artifacts (`app-main`, `app-standardidprovider`, `app-applications`, `app-users`, optionally `app-sdk`, and the `runtime` zip) from Enonic's Maven repo (`https://repo.enonic.com/dev`), all pinned to the version in `gradle.properties`.
2. Optionally downloading and `jlink`-ing a GraalVM JDK for the target OS, producing a trimmed custom runtime image.
3. Laying out everything under `build/install/<targetBaseName>/` (or as a `.zip`/`.tgz`) combined with static files from `src/common`, `src/generic`, and `src/sdk`.

## Common commands

```bash
# Set the XP version to build in gradle.properties first (key: `version`)

# Default build: generic server build, no bundled JDK
./gradlew build

# Assemble and install distribution content locally (most common dev command)
./gradlew installDist

# Platform-specific build with bundled JDK (jlink'd GraalVM)
./gradlew installDist -Pos=linux -Ptype=server
./gradlew installDist -Pos=mac-arm64 -Ptype=sdk
./gradlew installDist -Pos=windows -Ptype=sdk

# Force re-resolution of Enonic snapshot artifacts (used in CI)
./gradlew --refresh-dependencies build

# Publish (requires repo credentials; used by CI, not local dev)
./gradlew publish -Pos=<os> -Ptype=<type> -PrepoKey=... -PrepoUser=... -PrepoPassword=...
```

There is no lint or test task — there's nothing to lint/test in this repo. Verifying a change means running
`installDist` (or `distTar`/`distZip`) and checking the resulting directory/archive layout and contents under `build/`.

### Build parameters

- `-Pos=`: `linux` | `linux-arm64` | `mac-arm64` | `windows` | `generic` (default `generic`). Only non-`generic` values trigger a JDK download + jlink.
- `-Ptype=`: `sdk` | `server` (default `server`). `sdk` additionally bundles `app-sdk` and `src/sdk` contents (includes `app-xp-welcome`); `server` does not.
- Output archive name is `enonic-xp-<os>[-<type>]` (e.g. `enonic-xp-linux-server`, `enonic-xp-generic`). Generic builds omit the type suffix.
- `distTar` (gzip, `.tgz`) is produced for `linux`, `linux-arm64`, `mac-arm64`, `generic`; `distZip` for `windows`.

## Architecture

### Build logic split

- **`build.gradle`** — top-level distribution assembly: dependency resolution (`app`/`distro` configurations), the `distributions { main { contents { ... } } }` block that composes the final layout, and Maven publishing.
- **`jdk.gradle`** (applied from `build.gradle`) — everything JDK-related, isolated here:
  - `TargetOS` enum maps each `-Pos=` value to a GraalVM download URL/filename/archive layout (Oracle GraalVM download host: `gds.oracle.com/download/graal`). GraalVM version/train are hardcoded per enum constant (`jdkVersion`, `graalTrain`) — bump these here when updating the bundled JDK.
  - Exposes helper closures (`withJava()`, `isSdkBuild()`, `isGenericBuild()`, `withDistTar()`, `withDistZip()`, `getTargetBaseName()`, etc.) back onto `ext` so `build.gradle` can call them as plain functions.
  - Defines the `downloadJdk` → `unpackJdk` → `jlink` task chain; `jlink` invokes `jlink --add-modules ALL-MODULE-PATH` with JVMCI/Graal-JIT VM options baked in (`-XX:+EnableJVMCI`, thread priority policy, etc.) via `--add-options`.
  - `[distZip, distTar, installDist]*.dependsOn jlink` wires the JDK pipeline into the standard distribution tasks.

### Distribution layout composition

The final distribution content (see `build.gradle`'s `distributions.main.contents`) is layered from multiple sources, each into a specific subpath:

- `src/sdk/` → distribution root, **only when `type=sdk`** (adds the welcome app and SDK-only config, e.g. `src/sdk/home/config/logback.xml`).
- The `distro` config (the `runtime` zip artifact, unzipped) → distribution root — this is the actual XP runtime engine.
- The `app` config (the four/five admin app jars) → `system/40/` (app level 40 — see `appLevel` in `build.gradle`).
- `src/common/` → distribution root: startup scripts (`bin/service.sh`, `bin/setenv.sh`/`.bat`), default app configs (`home/config/*.cfg`, `logback.xml`), init.d/systemd service files (`service/`), and `README.txt` (version token-substituted via `ReplaceTokens`). `.sh` files and `service/init.d/xp` get `0755` permissions.
- `src/generic/` → distribution root, **only when no JDK is bundled** (`os=generic`); provides `bin/setenv.sh`/`.bat` that assume a system-installed Java (`setenv.sh`/`.bat` are excluded from `src/common` in that case to avoid duplicates).
- The jlink'd JDK image → `jdk/` subfolder, **only when a JDK is bundled** (non-generic os); `bin/*` and `lib/jspawnhelper` get `0755` permissions.

### Runtime expectations encoded in `src/common`

- `src/common/service/xp.conf` sets `XP_HOME` and is the template for `JAVA_HOME`/`JAVA_OPTS`/`XP_OPTS` overrides consumed by `bin/service.sh` and the systemd/init.d service definitions.
- `bin/service.sh` requires Java 11+ on `PATH` (or via `XP_JAVA_HOME`) and starts `bin/server.sh` from the distribution, backgrounding it and writing its PID to the given PIDFILE.
- App configs under `src/common/home/config/*.cfg` are the default runtime configuration for `app-main` and `app-standardidprovider`.

### CI (`.github/workflows/`)

- **`gradle.yml`** — runs on every push; builds and publishes the full `{sdk, server} × {mac-arm64, linux, linux-arm64, windows}` matrix (plus one `server`/`generic` combo) to Enonic's snapshot/dev repo, gated on branch (`master`, `7.*`, `8.*`) and non-release status via `enonic/release-tools/publish-vars`.
- **`release.yml`** — triggered by `v*` tags; same build matrix, publishes to the release repo, combines changelogs across the related repos (`app-admin-home`, `app-applications`, `app-users`, `app-standardidprovider`, `app-xp-welcome`, `xp`), and creates the GitHub Release.
- **`gen_release_notes.yml`** — manual workflow to regenerate combined release notes for an existing tag.
- All CI jobs use JDK 25 (Temurin) to *run* Gradle — independent of the GraalVM version being downloaded/bundled into the distribution itself.

## Versioning

The XP version being packaged is set in `gradle.properties` (`version=...`), and drives which snapshot/release artifacts are pulled from `com.enonic.xp:*`. This is distinct from the GraalVM JDK version (`jdk.gradle`) and the JDK used to run Gradle in CI (`gradle.yml`/`release.yml`).
