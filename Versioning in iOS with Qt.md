---
title: Versioning in iOS with Qt
layout: note
tags:
  - iOS
  - Qt6
---

Handling versioning in an iOS app developed with C++/Qt involves a combination of version control practices and ensuring that your app's versioning aligns with iOS App Store requirements. 

### Use Git for Source Control
- **Repository Initialization**: Initialize a Git repository if you haven’t already:
  ```sh
  git init
  ```

- **Commit Regularly**: Commit changes regularly with meaningful commit messages. This helps track the history of changes.
  ```sh
  git add .
  git commit -m "Your commit message"
  ```

- **Branching Strategy**: Use branches for feature development, bug fixes, and releases. Common strategies include Git Flow or GitHub Flow.
  ```sh
  git checkout -b feature/your-feature-name
  ```

### Semantic Versioning
Adopt Semantic Versioning (SemVer) for your app versioning, which follows the format `MAJOR.MINOR.PATCH`:

- **MAJOR**: Increments for incompatible API changes.
- **MINOR**: Increments for backward-compatible functionality.
- **PATCH**: Increments for backward-compatible bug fixes.

### Update Version Numbers in `Info.plist`
- **CFBundleShortVersionString**: This is the version number displayed to users (e.g., 1.2.0).
- **CFBundleVersion**: This is the build number, which should increment with every release (e.g., 123).

Edit `Info.plist` to reflect the correct version and build number.

```xml
<key>CFBundleShortVersionString</key>
<string>1.2.0</string>
<key>CFBundleVersion</key>
<string>123</string>
```

### Automate Versioning with Scripts
Automate the update of version numbers and build numbers using scripts. This can be part of your build pipeline.

**Example script to update `CFBundleVersion`:**

```sh
#!/bin/bash

# Increment build number
buildNumber=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "path/to/your/Info.plist")
buildNumber=$((buildNumber + 1))
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $buildNumber" "path/to/your/Info.plist"

echo "Updated build number to $buildNumber"
```

### Tagging Releases
Tag your releases in Git to keep track of different versions.

```sh
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0
```

### Continuous Integration (CI)
Set up a CI pipeline (e.g., GitHub Actions, Travis CI, CircleCI) to automate building, testing, and deploying your app. Ensure that your CI pipeline updates the build number and possibly even the version number automatically.

**Example GitHub Actions Workflow:**

```yaml
name: iOS Build

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: macos-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Xcode
        uses: maxim-lobanov/setup-xcode@v1

      - name: Increment build number
        run: ./scripts/increment_build_number.sh

      - name: Build and test
        run: xcodebuild -workspace YourApp.xcworkspace -scheme YourApp -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 12,OS=latest' clean build test
```

### Documenting Versions
Maintain a `CHANGELOG.md` to document changes in each release.

```markdown
# Changelog

## [1.2.0] - 2024-06-06
### Added
- New feature XYZ

### Fixed
- Bug ABC
```


# App Store Submissions

For the App Store to identify a new `*.ipa` file as distinct from one that's already been uploaded, you need to change the **CFBundleVersion** value in your `Info.plist` file. This value is known as the build number and must be incremented with each submission to the App Store, even if the version number (CFBundleShortVersionString) remains the same.

### Key Values in `Info.plist`

1. **CFBundleShortVersionString**: This is the version number of your app that users see. It's typically in the format `MAJOR.MINOR.PATCH` (e.g., `1.2.0`).

2. **CFBundleVersion**: This is the build number. It is an internal version number that must be unique for every App Store submission. It can be any string, but it is usually a monotonically increasing number (e.g., `123`, `124`, `125`, etc.).

### Example of Updating `Info.plist`

```xml
<key>CFBundleShortVersionString</key>
<string>1.2.0</string>
<key>CFBundleVersion</key>
<string>126</string>
```

### Steps to Increment the Build Number

1. **Open `Info.plist`**: Locate your `Info.plist` file in your project.

2. **Edit `CFBundleVersion`**: Increment the value of `CFBundleVersion` to a new unique number.

3. **Rebuild Your App**: Rebuild your app to generate a new `*.ipa` file with the updated build number.

### Automating Build Number Increment

You can automate the incrementing of the build number using a script. Here’s an example of a simple shell script to increment the build number in `Info.plist`.

```sh
#!/bin/bash

# Path to Info.plist
plistPath="path/to/your/Info.plist"

# Get the current build number
currentBuildNumber=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "$plistPath")

# Increment the build number
newBuildNumber=$((currentBuildNumber + 1))

# Set the new build number
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $newBuildNumber" "$plistPath"

echo "Updated build number to $newBuildNumber"
```

You can run this script before each build to ensure the build number is incremented automatically.

### Continuous Integration

In a CI/CD pipeline (e.g., GitHub Actions, Jenkins, etc.), you can integrate this script to run automatically, ensuring that each build has a unique build number.

**Example GitHub Actions Workflow:**

```yaml
name: iOS Build

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: macos-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Xcode
        uses: maxim-lobanov/setup-xcode@v1

      - name: Increment build number
        run: ./scripts/increment_build_number.sh

      - name: Build and archive
        run: xcodebuild -workspace YourApp.xcworkspace -scheme YourApp -sdk iphoneos -configuration Release archive -archivePath ${{ github.workspace }}/build/YourApp.xcarchive
```
