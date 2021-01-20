# Apple Code Signing

WPILib needs to sign all executable binaries (including dynamic libraries) that will be distributed on macOS due to [Apple requirements](https://developer.apple.com/documentation/macos-release-notes/macos-big-sur-11_0_1-universal-apps-release-notes). Furthermore, there are several layers of security in macOS with various designated requirements for each layer. The designated requirement for each layer must be satisfied for nominal operation.

|            macOS Security Feature             |                            Designated Requirement                             |
| --------------------------------------------- | ----------------------------------------------------------------------------- |
| Gatekeeper for executable code (.dylib, .app) | Developer ID Code Signature, Hardened Runtime, Secure Timestamp, Notarization |
| Gatekeeper for disk images (.dmg)             | None (Developer ID Code Signature and Notarization Recommended)               |
| Loader                                        | None (Intel), Code Signature (Apple Silicon)                                  |

When a file is downloaded from the Internet using a web browser, the `com.apple.quarantine` attribute is applied to the downloaded file. When a user (or another process) attempts to load an executable with this attribute applied, Gatekeeper will block execution while it verifies that the designated requirement is met.

The designated requirement of executable code is checked whenever it is executed by the loader.

## Signing the WPILib Installer

The WPILib Installer is distributed as a disk image which contains a .tar.gz (artifacts) and the application (.app) itself. The disk image itself is not signed or notarized; however this is not a designated requirement so Gatekeeper will open it. The application inside is signed with a valid Developer ID and secure timestamp, has the hardened runtime enabled, and is notarized -- satisfying the designated requirement for applications.

The .tar.gz archive is not included within the application itself due to Apple's requirements for notarization (code signature, hardened runtime, secure timestamp). The archive contains several third-party software which does not meet these requirements -- including the archive inside the application means that the entire application will not pass Apple's checks for notarization. This is why the notarized application is separate and uses its own SHA256 algorithm to verify that the artifacts file has not been tampered with.

---
The outer disk image can technically be signed with Developer ID for additional security; however a Gatekeeper issue in certain versions of macOS Mojave (10.14) prevents a user from opening a signed-but-not-notarized disk image.

---

## Signing other WPILib Binaries

All other binaries distributed by WPILib (allwpilib, thirdparty-opencv, etc.) are also signed with Developer ID, secure timestamp, and hardened runtime. Although this level of signature is not required for the loader, it is required if we choose to include the .tar.gz artifacts inside the installer application itself in the future (if third-party vendors also meet the designated requirement for Gatekeeper). 

Furthermore, some form of code signature is required for arm64 binaries and this is more than enough. On Intel systems, the existence of a code signature means the loader will check to ensure that it is valid, protecting the user from running malicious or damaged code.

## Special Considerations

The [tools plugin](https://github.com/wpilibsuite/wpilib-tool-plugin) is used in the process of building Java tools (i.e. SmartDashboard, Shuffleboard, PathWeaver). As part of the build process, the binaries are stripped of debug symbols (to reduce their size) and loader paths are configured. Both of these operations modify the binary and therefore invalidate the upstream code signature. Therefore, they are resigned with an ad-hoc signature. 

An ad-hoc signature has no identity associated with it -- but satisfies the designated requirement for the loader. This allows unit tests (which use the modified binaries) to be run and runnable JARs to be built. Production JARs then have their native binaries resigned with Developer ID, secure timestamp, and hardened runtime. Again, this level of signature is not required for the loader (the ad-hoc signature will do), we do it for the same reason as the other WPILib binaries.

## Command-Line Invocations

The `codesign` tool is used to sign all binaries. The invocations for the two common scenarios used in WPILib (ad-hoc, notarization-ready) are below:
```
codesign --force -s - libxyz.dylib
```

```
codesign --force --strict --timestamp --options=runtime -s $DEVELOPER_ID libxyz.dylib
```

## Signing in GitHub Actions
All code signing and notarization operations are setup to be performed automatically in GitHub Actions as part of CI. The WPILib organization has two (forked) actions used for [importing our Developer ID certificate](https://github.com/wpilibsuite/import-signing-certificate) and for [notarizing an application](https://github.com/wpilibsuite/xcode-notarize).

Once the certificate is imported, for most repos, the signing will be performed automatically when a certain build flag is passed in. For example, to build allwpilib with signed binary outputs: `./gradlew build -PdeveloperID=$DEVELOPER_ID`.

The only applications that are notarized are the WPILib installer and the VS Code standalone utility (these are the only supported entry points into the WPILib ecosystem, so they will be the only applications with the quarantine attribute applied).
