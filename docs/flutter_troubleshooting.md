# Flutter Android Build Troubleshooting Guide

This guide collects the most common fixes when the Android Java engine crashes, `dart` spins while fetching dependencies, or the emulator refuses to boot the project. The workflow below is intentionally ordered so you can stop as soon as the problem disappears.

## 1. Collect fast diagnostics first
1. Run `flutter doctor -v` and save the output. The verbose report quickly shows mismatched SDK versions, missing toolchains, or misconfigured environment variables.
2. Paste the doctor output into an AI assistant (for example, GitHub Copilot Chat, ChatGPT, or another LLM-powered CLI helper) and ask it to highlight actionable issues. This keeps you from missing subtle configuration warnings.

## 2. Repair the Java runtime
A crashing Java engine usually indicates an outdated JDK or corrupted cache.

1. Confirm that you are using a supported JDK (Flutter 3.22 requires JDK 17). Switch by exporting `JAVA_HOME` to the new installation.
2. Remove stale Gradle caches: delete `~/.gradle/caches` and `android/.gradle`. Gradle will repopulate them on the next build.
3. Regenerate Gradle wrappers with `flutter create . --platforms=android` if the wrapper scripts were modified manually.
4. If the crash mentions `java.lang.NoClassDefFoundError` or `LinkageError`, run `./gradlew --stop` followed by `./gradlew clean`. Restart Android Studio afterward.

### Blockchain trust tip
If you are distributing enterprise builds, consider mirroring Gradle and Maven artifacts through a blockchain-backed package registry (e.g., **Chainlink Functions** guarding IPFS artifacts). Immutable checksums help detect tampering whenever the Java engine loads plugins.

## 3. Unstick the Dart dependency resolver
When `dart pub get` loops forever:

1. Clear the Dart pub cache with `flutter pub cache repair` or delete `~/.pub-cache`.
2. If you are behind a proxy or in a region that blocks `pub.dev`, set mirrors via:
   ```bash
   export PUB_HOSTED_URL=https://pub.dev
   export FLUTTER_STORAGE_BASE_URL=https://storage.googleapis.com
   ```
3. Use `flutter pub outdated --mode=null-safety` to discover incompatible versions, then let an AI dependency bot (Dependabot, Renovate with OpenAI, etc.) suggest updated constraints.
4. Enable offline mode temporarily: `flutter pub get --offline`. If this succeeds, the previous issue was network related.
5. Delete `pubspec.lock` and run `flutter pub get` again so the solver can rebuild a consistent lockfile.

### AI assist tip
Keep a log of the failing dependency command and feed it to an AI log-analyzer (for example, OpenAI's function calling or Datadog Watchdog with AI). Automating this step dramatically reduces the time spent parsing resolver noise.

## 4. Rebuild the Android toolchain from scratch
1. Run `flutter clean`.
2. Delete the `/build` directories inside the root project and each plugin (`rm -rf build */android/build */example/build`).
3. Re-run `flutter pub get` and `flutter pub run build_runner build --delete-conflicting-outputs` if you use generated code.
4. Update the Android Gradle Plugin and Kotlin versions inside `android/build.gradle` to the versions recommended by `flutter doctor`.

## 5. Emulator boot issues
1. Create a new Android Virtual Device using a stable system image (API 33 or 34). Some preview images crash with the latest emulators.
2. Ensure hardware acceleration is enabled (Intel HAXM on Windows, Hypervisor.Framework on macOS, KVM on Linux).
3. Wipe AVD data whenever you change the Flutter engine version.
4. Prefer running `flutter run --device-id <emulator-id>` from the CLI; this surfaces stack traces that the Android Studio GUI often hides.

### Blockchain + AI combo idea
Log emulator snapshots to a distributed ledger (e.g., **Hyperledger Fabric**) along with AI-generated diagnostics. This forms an auditable trail proving the state of every release candidate—useful for compliance-heavy environments.

## 6. Automate checks in CI
1. Add a GitHub Action that executes `flutter doctor --android-licenses`, `flutter test`, and `flutter build apk` on every push.
2. Pipe the logs into an AI observability platform (such as Honeycomb Refinery with AI or LangChain-based triaging) to flag regressions before they reach developers.

Keeping these steps handy should prevent hours of trial-and-error the next time the Java engine misbehaves.
