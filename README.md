# Universal APK Generate Action

[![CI](https://github.com/snnaplab/universal-apk-generate-action/actions/workflows/ci.yml/badge.svg)](https://github.com/snnaplab/universal-apk-generate-action/actions/workflows/ci.yml)

This is a GitHub Action that creates a universal APK file from an AAB(Android App Bundle) file.
APK file is useful for testing on real devices.

## Inputs

All inputs are required. Cannot be omitted.

### `aab-path`

Path to the AAB file used for generation.

### `keystore-base64`, `keystore-password`, `key-alias`, `key-password`

Items required to sign the APK file.
See: https://developer.android.com/studio/publish/app-signing

Since these are confidential information, it is recommended to use the values of [secerts](https://docs.github.com/en/actions/security-guides/encrypted-secrets).

#### *NOTE:*

For `keystore-base64`, specify the string generated from the keystore file as follows:

```
$ cat release.keystore | base64
```

## Outputs

### `apk-path`

Path to the generated APK file.

## Usage

Minimal usage to generate an APK file from an existing AAB file:

```yaml
- uses: snnaplab/universal-apk-generate-action@v1
  id: apk-generate
  with:
    aab-path: 'app/build/outputs/bundle/release/app-release.aab'
    keystore-base64: ${{ secrets.KEYSTORE_BASE64 }}
    keystore-password: ${{ secrets.KEYSTORE_PASSWORD }}
    key-alias: ${{ secrets.KEY_ALIAS }}
    key-password: ${{ secrets.KEY_PASSWORD }}
- run: ls '${{ steps.apk-generate.outputs.apk-path }}'
```

## Workflow example

Build when release tag is pushed and upload AAB file and APK file to Artifacts:

```yaml
name: Release Build

on:
  push:
    tags:
      - 'release/**'

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Check out Android project
        uses: actions/checkout@v3
      - name: Set up JDK
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: 11
      - name: Build with Gradle
        run: ./gradlew bundleRelease
      - name: Get AAB file path
        id: aab-path
        run: echo "path=$(find . -regex '^.*/build/outputs/bundle/.*\.aab$' -type f | head -1)" >> $GITHUB_OUTPUT
      - name: Upload AAB file to Artifacts
        uses: actions/upload-artifact@v3
        with:
          path: ${{ steps.aab-path.outputs.path }}
      - name: Generate Universal APK file  
        uses: snnaplab/universal-apk-generate-action@v1
        id: apk-generate
        with:
          aab-path: ${{ steps.aab-path.outputs.path }}
          keystore-base64: ${{ secrets.KEYSTORE_BASE64 }}
          keystore-password: ${{ secrets.KEYSTORE_PASSWORD }}
          key-alias: ${{ secrets.KEY_ALIAS }}
          key-password: ${{ secrets.KEY_PASSWORD }}
      - name: Upload APK file to Artifacts
        uses: actions/upload-artifact@v3
        with:
          path: ${{ steps.apk-generate.outputs.apk-path }}
```

## License

[MIT](LICENSE)
