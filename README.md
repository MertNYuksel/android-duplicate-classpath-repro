# Duplicate classpath reproduction (minimal Android app)

Minimal **empty-activity** project. How the partner SDK is wired determines whether AGP’s duplicate-class check fails.

## Prerequisites

- JDK 21+
- Android SDK platform **35** (or adjust `compileSdk` in `app/build.gradle.kts`)
- Partner **`pom`** + **`aar`** (not committed; use `app/libs/` or `local-maven/` per steps below)

## Steps we tried

Fixed SDK bits + fixed `build.gradle` dependency graph ⇒ **the tasks below always produce the same result** (reproduced multiple times on this tree).

### 1) Direct AAR only (`implementation(files("libs/….aar"))`)

- Put the **`.aar`** in `app/libs/` (gitignored).
- **No POM** → Gradle does not resolve transitives.

| Task | Result |
|------|--------|
| `./gradlew :app:checkDebugDuplicateClasses` | **Passes** |
| `./gradlew :app:assembleDebug` | **Fails** at Kotlin compile (POM transitives are absent) |

Duplicate-class failures declared in the POM are **not** exercised in this setup.

### 2) POM + AAR via local Maven (`implementation("group:artifact:version")`)

- Layout: `local-maven/<group path>/<version>/artifact-version.{aar,pom}` (gitignored).
- In `settings.gradle.kts`, add `maven { url = uri("${rootDir}/local-maven") }`.
- Declare the SDK with Maven coordinates so the **POM transitives** are applied.

| Task | Result |
|------|--------|
| `./gradlew :app:checkDebugDuplicateClasses` | **Fails** — duplicate `androidx.compose.material3.*` (`material3-android-*.aar` vs `material3.aar`, same version) |
| `./gradlew :app:checkDebugDuplicateClasses` | **Fails** — fat SDK AAR: duplicate `org.chromium.net.*` (classes inside the AAR vs `cronet-api` from the POM) |
| `./gradlew :app:assembleDebug` | **Fails** — stops on the same duplicate-class errors as above |

## License

Sample project for integration debugging; no warranty.