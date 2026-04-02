# 🚀 CI/CD Complete Guide for Flutter with Firebase & Fastlane

## Table of Contents

1. [What is CI/CD?](#1-what-is-cicd)
2. [Why CI/CD Matters](#2-why-cicd-matters)
3. [CI/CD Pipeline Overview](#3-cicd-pipeline-overview)
4. [Tools We Use](#4-tools-we-use)
5. [Firebase App Distribution Setup](#5-firebase-app-distribution-setup)
6. [Fastlane Setup](#6-fastlane-setup)
7. [GitHub Actions (CI)](#7-github-actions-ci)
8. [Full Pipeline Walkthrough](#8-full-pipeline-walkthrough)
9. [Folder Structure](#9-folder-structure)
10. [Common Commands Cheat Sheet](#10-common-commands-cheat-sheet)

---

## 1. What is CI/CD?

### CI = Continuous Integration

> **Simple Definition:** Every time you push code, it gets **automatically tested and built** to make sure nothing is broken.

**Without CI:**
```
You write code → You build manually → You test manually → Bugs found late 😢
```

**With CI:**
```
You push code → Server builds automatically → Tests run automatically → Bugs found early 🎉
```

### CD = Continuous Delivery / Continuous Deployment

> **Simple Definition:** After the code is built and tested, it gets **automatically delivered** to testers or users.

**Without CD:**
```
Build APK manually → Upload to Drive → Share link → Testers download → 😴 Slow
```

**With CD:**
```
Push code → Auto build → Auto upload to Firebase → Testers notified instantly → 🚀 Fast
```

### The Full Picture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CI/CD PIPELINE                           │
│                                                                 │
│   ┌──────┐    ┌───────┐    ┌──────┐    ┌──────────┐    ┌─────┐│
│   │ CODE │───>│ BUILD │───>│ TEST │───>│ DELIVER  │───>│USERS││
│   │ Push │    │  App  │    │  App │    │(Firebase)│    │ Get ││
│   └──────┘    └───────┘    └──────┘    └──────────┘    │ App ││
│                                                        └─────┘│
│   ◄──── CI (Continuous Integration) ────►                      │
│   ◄──────────── CD (Continuous Delivery) ──────────────►       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Why CI/CD Matters

| Without CI/CD | With CI/CD |
|---|---|
| Build APK manually every time | APK builds automatically on push |
| Manually share with testers | Testers get notified automatically |
| Forget to run tests | Tests run on every push |
| "Works on my machine" bugs | Consistent build environment |
| Hours spent on release process | Release in minutes |
| Human errors in signing/config | Automated, repeatable process |

---

## 3. CI/CD Pipeline Overview

Here's the journey of your code from writing to reaching users:

```
Step 1: DEVELOP
   └── You write Flutter code on your machine

Step 2: PUSH (Triggers CI)
   └── You push to GitHub/GitLab

Step 3: BUILD (CI - automated)
   ├── CI server pulls your code
   ├── Runs `flutter pub get`
   ├── Runs `flutter build apk` (Android)
   └── Runs `flutter build ipa` (iOS)

Step 4: TEST (CI - automated)
   ├── Runs `flutter test` (unit tests)
   ├── Runs `flutter test --integration` (integration tests)
   └── Reports results

Step 5: DELIVER (CD - automated)
   ├── Fastlane takes the built APK/IPA
   ├── Signs it with proper certificates
   ├── Uploads to Firebase App Distribution
   └── Notifies testers via email/Slack
```

---

## 4. Tools We Use

### 🔥 Firebase App Distribution
**What it does:** Distributes your app to testers before publishing to the store.

Think of it as a **private app store** for your team:
- Upload APK/IPA → Testers get a link → They install and test
- You can create **tester groups** (QA team, stakeholders, beta users)
- Testers get **email notifications** when a new build is ready
- You can add **release notes** for each build

### 🏎️ Fastlane
**What it does:** Automates the boring parts of building and releasing apps.

Without Fastlane:
```
1. Open Android Studio
2. Go to Build > Generate Signed Bundle/APK
3. Enter keystore password
4. Wait for build
5. Find the APK file
6. Go to Firebase Console
7. Upload APK manually
8. Add release notes
9. Select tester groups
10. Click distribute
... Every. Single. Time. 😩
```

With Fastlane:
```
$ fastlane distribute
... Done! 🎉 (One command does ALL of the above)
```

### 🤖 GitHub Actions (CI Server)
**What it does:** Runs your pipeline automatically on GitHub's servers.

- Free for public repos, generous free tier for private repos
- Triggered by events (push, pull request, schedule)
- Runs on Linux/macOS/Windows virtual machines
- You define workflows in YAML files

---

## 5. Firebase App Distribution Setup

### Step 1: Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com)
2. Click **"Add Project"**
3. Name it (e.g., `ci-cd-test`)
4. Enable/disable Google Analytics (optional)
5. Click **"Create Project"**

### Step 2: Add Your Flutter App

**For Android:**
1. In Firebase Console → Click **Android icon**
2. Enter your package name (find it in `android/app/build.gradle` → `applicationId`)
   ```
   Example: com.example.ci_cd_test
   ```
3. Download `google-services.json`
4. Place it in `android/app/google-services.json`

**For iOS:**
1. In Firebase Console → Click **iOS icon**
2. Enter your Bundle ID (find it in Xcode → Runner → General → Bundle Identifier)
3. Download `GoogleService-Info.plist`
4. Place it in `ios/Runner/GoogleService-Info.plist`

### Step 3: Add Firebase SDK to Flutter

```bash
# Install FlutterFire CLI
dart pub global activate flutterfire_cli

# Configure Firebase in your project
flutterfire configure
```

Add to `android/build.gradle.kts` (your project uses Kotlin DSL):
```kotlin
buildscript {
    dependencies {
        // Add this line
        classpath("com.google.gms:google-services:4.4.2")
    }
}
```

Add to `android/app/build.gradle.kts`:
```kotlin
plugins {
    // Add this line with your other plugins
    id("com.google.gms.google-services")
}
```

### Step 4: Set Up App Distribution

1. In Firebase Console → Go to **"App Distribution"**
2. Click **"Get Started"**
3. Create a **Tester Group:**
   - Go to **"Testers & Groups"** tab
   - Click **"Add Group"** → Name it (e.g., `qa-team`)
   - Add tester emails

### Step 5: Get Firebase CLI Token

```bash
# If you don't have Node.js, use the firebase.exe we downloaded:
.\firebase.exe login:ci

# If you have Node.js and installed it via npm:
firebase login:ci
```

> **⚠️ IMPORTANT:** Save this token securely. You'll add it as a secret in GitHub.

---

## 6. Fastlane Setup

### What is Fastlane?

Fastlane is an **automation tool** that handles:
- ✅ Building your app (APK / IPA)
- ✅ Signing your app (keystore / certificates)
- ✅ Uploading to Firebase App Distribution
- ✅ Uploading to Play Store / App Store
- ✅ Managing screenshots and metadata
- ✅ Running tests

### Prerequisites

```bash
# Install Ruby (Fastlane is built with Ruby)
# Windows: Download from https://rubyinstaller.org
# macOS: Already installed (or use brew install ruby)

# Install Fastlane
gem install fastlane

# Verify installation
fastlane --version
```

### Android Setup

#### Step 1: Initialize Fastlane

```bash
cd android
fastlane init
```

This creates the `android/fastlane/` folder with:
```
android/
└── fastlane/
    ├── Appfile       ← App configuration (package name)
    ├── Fastfile       ← Your automation scripts (lanes)
    └── Gemfile        ← Ruby dependencies
```

#### Step 2: Configure Appfile

```ruby
# android/fastlane/Appfile

json_key_file("")  # Path to service account JSON (for Play Store, optional for now)
package_name("com.example.ci_cd_test")  # Your app's package name
```

#### Step 3: Configure Fastfile

```ruby
# android/fastlane/Fastfile

default_platform(:android)

platform :android do

  # ─────────────────────────────────────
  # Lane: Build Debug APK
  # ─────────────────────────────────────
  desc "Build a debug APK"
  lane :build_debug do
    # Go to project root and build
    sh("cd ../.. && flutter clean")
    sh("cd ../.. && flutter pub get")
    sh("cd ../.. && flutter build apk --debug")
  end

  # ─────────────────────────────────────
  # Lane: Build Release APK
  # ─────────────────────────────────────
  desc "Build a release APK"
  lane :build_release do
    sh("cd ../.. && flutter clean")
    sh("cd ../.. && flutter pub get")
    sh("cd ../.. && flutter build apk --release")
  end

  # ─────────────────────────────────────
  # Lane: Distribute to Firebase
  # ─────────────────────────────────────
  desc "Build and distribute to Firebase App Distribution"
  lane :distribute do
    build_release

    firebase_app_distribution(
      app: "1:YOUR_APP_ID:android:YOUR_HASH",  # ← From Firebase Console
      groups: "qa-team",                         # ← Your tester group name
      release_notes: "New build from Fastlane 🚀",
      firebase_cli_token: ENV["FIREBASE_CLI_TOKEN"]  # ← From `firebase login:ci`
    )
  end

  # ─────────────────────────────────────
  # Lane: Deploy to Play Store (Internal Testing)
  # ─────────────────────────────────────
  desc "Deploy to Google Play Internal Testing"
  lane :deploy_internal do
    sh("cd ../.. && flutter build appbundle --release")

    upload_to_play_store(
      track: "internal",
      aab: "../../build/app/outputs/bundle/release/app-release.aab",
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true
    )
  end

end
```

#### Step 4: Add Firebase App Distribution Plugin

```bash
cd android
fastlane add_plugin firebase_app_distribution
```

### iOS Setup (macOS only)

#### Step 1: Initialize Fastlane

```bash
cd ios
fastlane init
```

#### Step 2: Configure Fastfile

```ruby
# ios/fastlane/Fastfile

default_platform(:ios)

platform :ios do

  desc "Build and distribute to Firebase"
  lane :distribute do
    sh("cd ../.. && flutter clean")
    sh("cd ../.. && flutter pub get")
    sh("cd ../.. && flutter build ipa --release --export-options-plist=ios/ExportOptions.plist")

    firebase_app_distribution(
      app: "1:YOUR_APP_ID:ios:YOUR_HASH",
      groups: "qa-team",
      release_notes: "New iOS build 🍎",
      firebase_cli_token: ENV["FIREBASE_CLI_TOKEN"],
      ipa_path: "../build/ios/ipa/ci_cd_test.ipa"
    )
  end

end
```

### How to Run Fastlane

```bash
# From the android/ directory:
cd android

# Build debug APK
fastlane build_debug

# Build release APK
fastlane build_release

# Build + Upload to Firebase
fastlane distribute

# Deploy to Play Store
fastlane deploy_internal
```

---

## 7. GitHub Actions (CI)

### What is GitHub Actions?

GitHub Actions is a **CI/CD service built into GitHub**. It runs your pipeline on GitHub's servers automatically.

### Key Concepts

| Concept | What it is | Example |
|---|---|---|
| **Workflow** | A YAML file defining your pipeline | `.github/workflows/ci.yml` |
| **Trigger** | What starts the workflow | Push to `main`, Pull Request |
| **Job** | A set of steps that run together | `build`, `test`, `deploy` |
| **Step** | A single action in a job | Run a command, use an action |
| **Runner** | The machine that runs your job | `ubuntu-latest`, `macos-latest` |
| **Secret** | Encrypted variable | API keys, tokens |

### Workflow File: Build & Test

Create `.github/workflows/ci.yml`:

```yaml
# .github/workflows/ci.yml
name: Flutter CI

# ━━━ WHEN to run this pipeline ━━━
on:
  push:
    branches: [main, develop]      # Run on push to main or develop
  pull_request:
    branches: [main]               # Run on PRs to main

# ━━━ WHAT to run ━━━
jobs:

  # ────────────────────────────
  # Job 1: Build & Test
  # ────────────────────────────
  build_and_test:
    name: 🏗️ Build & Test
    runs-on: ubuntu-latest         # Use a Linux machine

    steps:
      # Step 1: Get the code
      - name: 📥 Checkout code
        uses: actions/checkout@v4

      # Step 2: Set up Java (needed for Android)
      - name: ☕ Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'

      # Step 3: Set up Flutter
      - name: 🐦 Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.29.0'  # Use your Flutter version
          channel: 'stable'

      # Step 4: Install dependencies
      - name: 📦 Install dependencies
        run: flutter pub get

      # Step 5: Analyze code (lint check)
      - name: 🔍 Analyze code
        run: flutter analyze

      # Step 6: Run tests
      - name: 🧪 Run tests
        run: flutter test

      # Step 7: Build APK
      - name: 🔨 Build APK
        run: flutter build apk --debug

      # Step 8: Upload APK as artifact (so you can download it)
      - name: 📤 Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: app-debug
          path: build/app/outputs/flutter-apk/app-debug.apk
```

### Workflow File: Distribute to Firebase

Create `.github/workflows/distribute.yml`:

```yaml
# .github/workflows/distribute.yml
name: 🚀 Distribute to Firebase

# Only run when pushing a tag like v1.0.0
on:
  push:
    tags:
      - 'v*'

jobs:
  distribute:
    name: 📦 Build & Distribute
    runs-on: ubuntu-latest

    steps:
      - name: 📥 Checkout code
        uses: actions/checkout@v4

      - name: ☕ Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: 🐦 Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.29.0'
          channel: 'stable'

      - name: 📦 Install dependencies
        run: flutter pub get

      - name: 🧪 Run tests
        run: flutter test

      - name: 🔨 Build Release APK
        run: flutter build apk --release

      # ━━━ Distribute via Firebase ━━━
      - name: 🔥 Upload to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID }}
          serviceCredentialsFileContent: ${{ secrets.FIREBASE_CREDENTIAL_FILE_CONTENT }}
          groups: qa-team
          file: build/app/outputs/flutter-apk/app-release.apk
          releaseNotes: |
            Version: ${{ github.ref_name }}
            Commit: ${{ github.sha }}
            Built by GitHub Actions 🤖
```

### Setting Up GitHub Secrets

Your sensitive data (tokens, keys) must be stored as **GitHub Secrets**:

1. Go to your GitHub repo → **Settings** → **Secrets and variables** → **Actions**
2. Click **"New repository secret"**
3. Add these secrets:

| Secret Name | Value | How to Get It |
|---|---|---|
| `FIREBASE_APP_ID` | `1:66018666918:android:20460df1f06e3ffd3b4767` | Firebase Console → Project Settings → Your App |
| `FIREBASE_CLI_TOKEN` | `1//0abc...xyz` | Run `firebase login:ci` in terminal |
| `FIREBASE_CREDENTIAL_FILE_CONTENT` | JSON content | Firebase Console → Service Account → Generate Key |
| `KEYSTORE_BASE64` | Base64 encoded keystore | `base64 your-key.keystore` |
| `KEYSTORE_PASSWORD` | Your keystore password | You set this when creating keystore |
| `KEY_PASSWORD` | Your key password | You set this when creating keystore |
| `KEY_ALIAS` | Your key alias | You set this when creating keystore |

---

## 8. Full Pipeline Walkthrough

Here's exactly what happens when you push code:

```
YOU: git push origin main
  │
  ▼
GITHUB: "I see a push to main! Let me check .github/workflows/"
  │
  ▼
GITHUB ACTIONS: Starts a virtual machine (ubuntu-latest)
  │
  ├── 1. Checkout your code
  ├── 2. Install Java 17
  ├── 3. Install Flutter 3.29.0
  ├── 4. Run: flutter pub get
  ├── 5. Run: flutter analyze (check for lint errors)
  ├── 6. Run: flutter test (run all tests)
  ├── 7. Run: flutter build apk --release
  │       └── Creates: build/app/outputs/flutter-apk/app-release.apk
  ├── 8. Upload APK to Firebase App Distribution
  │       └── Uses your FIREBASE_APP_ID secret
  │
  ▼
FIREBASE: "New build received!"
  │
  ├── Notifies testers in "qa-team" group via email
  └── Testers click the link → Install the new build
  │
  ▼
TESTERS: "Wow, I got the new build automatically!" 🎉
```

### Visual Example of a Successful Run

```
✅ Flutter CI                          3m 42s
├── 📥 Checkout code                   2s
├── ☕ Set up Java                      8s
├── 🐦 Set up Flutter                  45s
├── 📦 Install dependencies            30s
├── 🔍 Analyze code                    15s
├── 🧪 Run tests                      20s
├── 🔨 Build APK                      1m 45s
└── 🔥 Upload to Firebase             17s
```

---

## 9. Folder Structure

After setting everything up, your project will look like this:

```
ci_cd_test/
├── .github/
│   └── workflows/
│       ├── ci.yml                 ← CI pipeline (build & test)
│       └── distribute.yml         ← CD pipeline (Firebase distribution)
│
├── android/
│   ├── fastlane/
│   │   ├── Appfile                ← Android app config
│   │   ├── Fastfile               ← Android automation lanes
│   │   └── Gemfile                ← Ruby dependencies
│   ├── Gemfile                    ← Ruby dependencies
│   ├── app/
│   │   ├── build.gradle.kts       ← App-level gradle config (Kotlin DSL)
│   │   └── google-services.json   ← Firebase config (Android)
│   └── build.gradle.kts           ← Project-level gradle config (Kotlin DSL)
│
├── ios/
│   ├── fastlane/
│   │   ├── Appfile                ← iOS app config
│   │   ├── Fastfile               ← iOS automation lanes
│   │   └── Gemfile                ← Ruby dependencies
│   └── Runner/
│       └── GoogleService-Info.plist ← Firebase config (iOS)
│
├── lib/
│   └── main.dart                  ← Your Flutter code
│
├── test/
│   └── widget_test.dart           ← Your tests
│
└── pubspec.yaml                   ← Flutter dependencies
```

---

## 10. Common Commands Cheat Sheet

### Git Commands (Triggering CI/CD)

```bash
# Push to trigger CI
git add .
git commit -m "feat: add new feature"
git push origin main

# Create a release tag to trigger distribution
git tag v1.0.0
git push origin v1.0.0
```

### Fastlane Commands

```bash
# Android
cd android
fastlane build_debug          # Build debug APK
fastlane build_release        # Build release APK
fastlane distribute           # Build + Upload to Firebase
fastlane deploy_internal      # Deploy to Play Store (internal)

# iOS (macOS only)
cd ios
fastlane distribute           # Build + Upload to Firebase
```

### Firebase CLI Commands

```bash
firebase login                # Login to Firebase
firebase login:ci             # Get CI token
firebase projects:list        # List your Firebase projects
```

### Flutter Commands

```bash
flutter test                  # Run all tests
flutter analyze               # Check code quality
flutter build apk --release   # Build release APK
flutter build appbundle       # Build App Bundle (Play Store)
flutter build ipa             # Build iOS (macOS only)
```

---

## 🎯 Quick Start Checklist

Use this checklist to set up CI/CD for your project:

- [ ] **Firebase Project** — Create project in Firebase Console
- [ ] **Add App** — Register Android/iOS app in Firebase
- [ ] **Download Config** — Get `google-services.json` / `GoogleService-Info.plist`
- [ ] **Firebase CLI** — Install and run `firebase login:ci` to get token
- [ ] **Install Fastlane** — `gem install fastlane`
- [ ] **Init Fastlane** — Run `fastlane init` in `android/` and/or `ios/`
- [ ] **Configure Fastfile** — Add build and distribute lanes
- [ ] **Add Firebase Plugin** — `fastlane add_plugin firebase_app_distribution`
- [ ] **Create GitHub Workflow** — Add `.github/workflows/ci.yml`
- [ ] **Add Secrets** — Add `FIREBASE_APP_ID`, tokens, etc. to GitHub Secrets
- [ ] **Push & Watch** — Push code and watch the magic happen! 🎉

---

> **💡 TIP:** Start simple! First get the CI workflow (build & test) working, then add CD (Firebase distribution). Don't try to set up everything at once.

> **⚠️ WARNING:** Never commit secrets (API keys, tokens, keystores) to your repository. Always use GitHub Secrets or environment variables.
