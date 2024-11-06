---
title: GitHub Actions
layout: note
tags:
  - git
  - ci/cd
  - tests
  - yaml
---
Once you have your unit tests set up on your project, you can use GitHub Actions to run the tests, build the project, and even generate a build package whenever you upload a pull request. If tests or build fails, the PR fails.

In your project root, make a Yaml file at `.github/workflows/`. I called mine `android-ci.yml` for my android project. Here is the contents:

```yaml
name: Android CI
run-name: ${{ github.actor }} is running CI tests
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - run: echo "⭐ Running Android CI ⭐"
    - uses: actions/checkout@v4
    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
    - name: Grant execute permission for gradlew
      run: chmod +x ./gradlew
      
    - run: echo "✅ Testing Starts Here"
    - name: Run unit tests
      run: ./gradlew test --info
      
    - run: echo "✅ Building Starts Here"
    - name: Build Release APK
      run: ./gradlew assembleRelease --stacktrace
      
    - run: echo "✅ Deployment Starts Here"
    - name: Upload Release APK
      uses: actions/upload-artifact@v3
      with:
        name: release-apk
        path: app/build/outputs/apk/release/*.apk
```

The emojis are corny, but they allow me to easily see where the different sections of the process start.

You can see the actions run after you push by going to the top of your repository's GitHub page and clicking on **Actions**. You might have multiple Yaml files in your project, and you can select them on the left, under **Workflows**.

If you look at the log of individual CI runs, you can click on the latest run. From there you get a breakdown: There's the triggering action and branch. There's a button to get to the output log of the build. There's a section that shows an overview of any warnings or errors.

Then at the bottom is the Artifacts section. This is where your successful build can be obtained. I believe by default on free accounts, GitHub puts a limit on how many Artifacts it will keep available, and possibly for how long. So it might be best to create an actual **Release** on GitHub if you want to keep it around for a while.
