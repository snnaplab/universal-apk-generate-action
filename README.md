# Universal APK Generate Action

Generate Universal APK file from Android App Bundle file. APK file is Useful for testing on devices.

## Usage

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
        uses: actions/checkout@v2
      - name: Set up JDK
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
      - name: Build with Gradle
        run: ./gradlew bundleRelease
      - name: Get AAB file path
        id: aab-path
        run: echo "::set-output name=path::$(find **/build/outputs/bundle -name '*.aab' -type f | head -1)"
      - name: Upload AAB file to Artifacts
        uses: actions/upload-artifact@v2
        with:
          path: ${{ steps.aab-path.outputs.path }}
      - name: Generate Universal APK file  
        uses: hkusu/universal-apk-generate-action@v1
        id: apk-generate
        with:
          aab-path: ${{ steps.aab-path.outputs.path }}
          keystore-base64: ${{ secrets.KEYSTORE_BASE64 }}
          keystore-password: ${{ secrets.KEYSTORE_PASSWORD }}
          key-alias: ${{ secrets.KEY_ALIAS }}
          key-password: ${{ secrets.KEY_PASSWORD }}
      - name: Upload APK file to Artifacts
        uses: actions/upload-artifact@v2
        with:
          path: ${{ steps.apk-generate.outputs.apk-path }}
```

#### *NOTE:*

The secrets `KEYSTORE_BASE64` is created as follows:

```
$ cat release.keystore | base64
```

## License

MIT
