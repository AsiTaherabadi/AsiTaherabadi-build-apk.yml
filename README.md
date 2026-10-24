
name: Build RuniMaf Android APK

on:
  workflow_dispatch:
  push:

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Extract Android project
        run: unzip -o 'runimaf-android-app (1).zip' -d extracted

      - name: Find project folder
        id: project
        run: |
          PROJECT_DIR=$(find extracted -type f \( -name settings.gradle -o -name settings.gradle.kts \) -printf '%h\n' | head -n 1)
          if [ -z "$PROJECT_DIR" ]; then
            echo "Gradle project not found in ZIP"
            exit 1
          fi
          echo "dir=$PROJECT_DIR" >> "$GITHUB_OUTPUT"

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - uses: android-actions/setup-android@v3

      - name: Install Android SDK
        run: sdkmanager 'platform-tools' 'platforms;android-35' 'build-tools;35.0.0'

      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.9'

      - name: Build APK
        run: gradle --no-daemon -p "${{ steps.project.outputs.dir }}" assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: runimaf-android-apk
          path: extracted/**/build/outputs/apk/debug/*.apk
          if-no-files-found: error
