# Android GitHub Actions Workflows

Reusable GitHub Actions workflows for building Android apps with Gradle.

These workflows handle:

* Android SDK setup
* Java configuration
* Gradle caching
* Debug and Release builds
* Optional APK signing using GitHub Secrets

They are designed so you can reuse the same CI/CD logic across multiple Android repositories without copying large workflow files.

---

# Features

* Build **debug APK**
* Build **signed release APK**
* Gradle dependency caching for faster builds
* Optional keystore restoration for signed builds
* Works with **Kotlin DSL and Groovy Gradle projects**
* Compatible with **Compose, Hilt, Room, KSP, etc.**

---

# Repository Structure

```
.github/
└ workflows/
   └ android-build.yml
   └ android-release.yml
```

This file contains the reusable Android build and Android Releaseworkflow.

---

# Using the Workflow in Another Repository

## Step 1 — Add a workflow that calls the reusable workflow

Create a workflow in your Android project:

```
.github/workflows/android-ci.yml
```

Example:

```yaml
name: Android CI

on:
  push:
    branches:
      - main
      - feature-*
  pull_request:

jobs:
  build:
    uses: <OWNER>/<REPO>/.github/workflows/android-build.yml@main
    secrets: inherit
```

Replace:

```
<OWNER>/<REPO>
```

with the repository that contains the reusable workflow.

Example:

```
amrish/android-workflows
```

---

# Signing Configuration (Optional)

If you want CI to generate **signed release APKs**, you must add signing secrets.

Go to:

```
Repository → Settings → Secrets and variables → Actions
```

Add the following secrets:

| Secret            | Description                  |
| ----------------- | ---------------------------- |
| KEYSTORE_BASE64   | Base64 encoded keystore file |
| KEYSTORE_PASSWORD | Keystore password            |
| KEY_PASSWORD      | Key password                 |
| KEY_ALIAS         | Key alias                    |

---

# Converting a Keystore to Base64

### Linux / macOS

```
base64 release-keystore.jks > keystore.txt
```

### Windows PowerShell

```
$bytes = [System.IO.File]::ReadAllBytes("release-keystore.jks")
[System.Convert]::ToBase64String($bytes)
```

Copy the output and store it as the `KEYSTORE_BASE64` secret.

---

# What the Workflow Does

Each CI run performs the following:

1. Checkout repository
2. Install Android SDK
3. Setup Java 17
4. Restore keystore (if secrets exist)
5. Create `keystore.properties`
6. Run Gradle tests
7. Build debug APK
8. Build signed release APK

---

# Typical CI Flow

```
Push / PR
   ↓
GitHub Actions
   ↓
Gradle Build
   ↓
Debug APK
   ↓
Release APK
```

---

# Example Repository Layout

```
android-project
│
├ app
├ gradle
├ gradlew
├ settings.gradle.kts
│
└ .github
   └ workflows
      └ android-ci.yml
```

---

# Recommended Workflow Setup

Most Android projects use two workflows:

| Workflow   | Purpose                                 |
| ---------- | --------------------------------------- |
| Android CI | Build + tests on push and pull requests |
| Release    | Signed build when pushing version tags  |

---

# License

MIT
