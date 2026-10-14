# FlutCheck 🚀

## Validates native Android AAR artifacts against headless Flutter build environments.

Created and maintained by **Harrison Kuria** Under **Tacit Works**


## The Problem

An Android Archive (.aar) that compiles flawlessly inside a native Android Studio environment can easily crash or fail to sync when dropped into a cross-platform Flutter application. 
This is because Flutter injects its own opinionated Gradle wrappers, Kotlin Gradle Plugin (KGP) rules, and strict AndroidX version mappings.
Waiting for your client’s development team to report compilation or dependency errors is an expensive, reactive way to handle quality assurance.



## The Solution

FlutCheck lets you shift left. 
It is a lightweight, headless integration smoke tester packaged as a GitHub Composite Action. 
It dynamically captures your compiled native .aar binary, drops it into a sandbox Flutter environment, 
performs direct configurations on the host Gradle tree, and triggers an automated build check—ensuring you catch ecosystem mismatches long before your consumers do.

## How It Works

1. Isolates a Headless Sandbox: Provisions a pristine, sterile Flutter project (headless_harness) isolated to the Android platform layer.
2. Maps Your Artifact: Copies your native .aar directly into the harness tracking tree, safely decoupling your custom build configurations from the validation matrix.
3. Automated Gradle Surgery: Injects local flatDir repositories and explicit binary implementations right into Flutter’s internal generated build.gradle structures.
4. Compile-Time Integrity Validation: Triggers flutter build apk --debug.
   If there is any Kotlin metadata clash, D8 desugaring collision, or version mismatch, the pipeline fails internally before the asset is packaged for release.


   
## Quick Start Integration

Add FlutCheck directly into your native repository's Continuous Integration setup (e.g., .github/workflows/ci.yml).
```
name: Native Library Validation Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  validate-cross-platform:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Setup Java Build Environment
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: Setup Flutter Toolchain
        uses: subosito/flutter-action@v2
        with:
          channel: 'stable'

      - name: Build Native Android SDK
        run: ./gradlew :my-sdk-module:assembleRelease

      # Invoke the TacitWorks verification utility
      - name: Run FlutCheck Verification
        uses: tacitworks/flutcheck@v1
        with:
          aar-path: 'my-sdk-module/build/outputs/aar/my-sdk-module-release.aar'
```

## Advanced Usage: Cross-Platform Testing Matrix

To anticipate upcoming breaking ecosystem adjustments upstream, you can map FlutCheck against a multi-channel configuration array:
```
strategy:
  fail-fast: false
  matrix:
    flutter-channel: [ stable, beta ]

steps:
  # ... Environment configuration steps ...
  - name: Setup Flutter Toolchain
    uses: subosito/flutter-action@v2
    with:
      channel: ${{ matrix.flutter-channel }}

  - name: Run FlutCheck Matrix
    uses: tacitworks/flutcheck@v1
    with:
      aar-path: 'build/outputs/aar/sdk-release.aar'
```

# License

This project is licensed under the MIT License - see the LICENSE file for details.

