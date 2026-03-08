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

# Android Signing Configuration

For CI to produce a **signed release APK**, your Android project must support loading signing credentials from `keystore.properties`.

This allows CI to dynamically inject credentials without committing secrets to the repository.

---

# Step 1 — Create `keystore.properties`

In the **root of your Android project**, create:

```
keystore.properties
```

Example content:

```
storePassword=your_keystore_password
keyPassword=your_key_password
keyAlias=release
storeFile=release-keystore.jks
```

⚠️ This file **must not be committed to git**.

Add to `.gitignore`:

```
keystore.properties
release-keystore.jks
```

---

# Step 2 — Update `build.gradle.kts`

Add the following code to your **app module's `build.gradle.kts`**.

### Load signing properties

```kotlin
import java.util.Properties
import java.io.FileInputStream

val keystoreProperties = Properties()
val keystorePropertiesFile = rootProject.file("keystore.properties")

if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(FileInputStream(keystorePropertiesFile))
}
```

---

### Add signing configuration

Inside the `android {}` block:

```kotlin
signingConfigs {
    create("release") {
        storeFile = rootProject.file(keystoreProperties["storeFile"] as String)
        storePassword = keystoreProperties["storePassword"] as String
        keyAlias = keystoreProperties["keyAlias"] as String
        keyPassword = keystoreProperties["keyPassword"] as String
    }
}
```

---

### Attach signing to the release build

Inside `buildTypes`:

```kotlin
buildTypes {
    release {
        isMinifyEnabled = false
        signingConfig = signingConfigs.getByName("release")

        proguardFiles(
            getDefaultProguardFile("proguard-android-optimize.txt"),
            "proguard-rules.pro"
        )
    }
}
```

---

# Step 3 — Add GitHub Secrets

Go to:

```
Repository → Settings → Secrets and variables → Actions
```

Add the following secrets.

| Secret            | Description                  |
| ----------------- | ---------------------------- |
| KEYSTORE_BASE64   | Base64 encoded keystore file |
| KEYSTORE_PASSWORD | Keystore password            |
| KEY_PASSWORD      | Key password                 |
| KEY_ALIAS         | Key alias                    |

---

# Converting Keystore to Base64

### Linux / macOS

```
base64 release-keystore.jks > keystore.txt
```

### Windows PowerShell

```
$bytes = [System.IO.File]::ReadAllBytes("release-keystore.jks")
[System.Convert]::ToBase64String($bytes)
```

Copy the generated output and store it in the `KEYSTORE_BASE64` secret.

---

# What the Workflow Does

Each CI run performs the following steps:

1. Checkout repository
2. Install Android SDK
3. Setup Java 17
4. Restore keystore from secrets
5. Generate `keystore.properties`
6. Run Gradle tests
7. Build Debug APK
8. Build Signed Release APK

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
