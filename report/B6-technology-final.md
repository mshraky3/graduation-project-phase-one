# Technology Choices

## Introduction

The technology behind the app decides what the user can expect from it. The choices below give the user one app that looks and works the same on Windows, macOS, Linux, Android, iPhone and iPad; devices that find each other and talk directly, with no internet and no server; small AI models that run on one of the user's own computers; and stored data that is encrypted on every device. Each choice below states the decision, the reason for it and its limitation. A later section explains how the main facts behind the choices were checked, and which systems the team can test.

## 1. Operating systems

**Decision.** The app supports five systems, with the minimum versions that Flutter 3.47 supports: Windows 10, macOS 12, Linux (Ubuntu 20.04 or Debian 10), Android 7.0, and iOS and iPadOS 15.

**Reason.** All five systems are kept in the design. The limits of each system are handled by capabilities: each device tells the others what it can do, and a feature offers an action only when both devices support it.

**Limitation.** iPhone and iPad cannot be controlled from another device, never take the AI role, and see only the photo library and the folders that the user picks. Apps for iPhone, iPad and Mac can be built only on a Mac with Xcode.

**What will be tested.** No member of the team has a Mac or a Linux machine.
- **Windows and Android** (phones and tablets) are tested on the team's own devices. They are the main pair for the prototype.
- **Linux** is tested in part: only Ubuntu 24.04 with GNOME on Wayland, started from a USB stick or a second partition on a team member's Windows computer. Other desktops and X11 are checked against their documentation only.
- **macOS, iPhone and iPad** are not built and not tested. Their parts of the design are checked against Apple's documentation only.

## 2. One app for all systems

**Decision.** One Flutter (Dart) app runs on all five systems. Small native parts handle screen capture, input control, the Android background service and the clipboard.

**Reason.** One code base keeps the behaviour the same on every device. LocalSend and RustDesk show that this approach works for features close to the ones of this project.

**Limitation.** The app does not have a fully native look, and some native code is still needed for each system. The team has to learn Dart.

## 3. How devices talk to each other

**Decision.**
- **Discovery.** Devices find each other on the local network with mDNS/Bonjour, through the Bonsoir package.
- **The connection.** Every connection uses TLS 1.3. The connecting device checks that the other device's certificate matches the one stored at pairing (its SHA-256 must equal the stored device id). The connecting device then proves its own key with a signed challenge inside the encrypted channel.
- **Messages and files.** Messages use one long-lived connection. Files use a separate connection, in chunks, that can resume after a drop.
- **Live streams.** Screen Sharing and Remote Control and Extra Screen use WebRTC. WebRTC is set up over the message connection, uses no STUN or TURN server, and its DTLS fingerprint is carried inside the paired connection.

**Reason.** Dart's TLS server cannot check an unknown client certificate for each connection (section 7), so the client proves its key inside the channel instead. On iPhone, Bonjour needs no special Apple entitlement, while custom broadcast or multicast would. WebRTC already provides video compression, delivery over UDP and quality that adapts to the Wi-Fi, which a custom stream would have to build.

**Limitation.** The proof inside the channel is the project's own small protocol, so it needs a review and test vectors. Dart cannot read the exported keys of a TLS session, so the proof is bound to the hash of the server's certificate instead. WebRTC makes the app larger and adds a second layer of encryption.

## 4. AI models and how they run

**Decision.** Exact copies are found with SHA-256 fingerprints on every device, with no AI. For similar photos and search by meaning, two small models run with ONNX Runtime on one laptop or computer, the AI device: a multilingual CLIP ViT-B/32 model for photos and multilingual-e5-small for file names and text. The device that holds a photo sends the AI device a 224 × 224 preview, which the AI device deletes after use; a video is sent as 3 frames. The minimum AI device is a 64-bit laptop or computer with 4 processor cores, 8 GB of memory (16 GB suggested), about 3 GB of free disk space and no graphics card.

**Reason.** The models need no graphics card and fit in the memory of an ordinary laptop. A first build takes an estimated 2 hours for 100,000 photos, which is acceptable for a one-time job; after that, only new photos are processed.

**Limitation.** Larger models would be more accurate. The quality of photo search with Arabic words is not known (section 7). The app has no chat assistant.

## 5. Stored data

**Decision.** Every device stores its data in SQLite, through the `sqlite3` Dart package. The package's default build includes full-text search (FTS5), and the app is built with the package's SQLCipher option. The device's private key is kept in the operating system's key store through the `flutter_secure_storage` package: Keychain on Apple systems, Android Keystore, libsecret on Linux, and a file protected by Windows DPAPI on Windows.

**The three security rules of the system.** The main database is encrypted with a random key that is kept in the operating system's key store. The vault key is random and is stored only wrapped by a key made from the master password, so the master password can be changed. A device accepts a file list only if the owner device's signature is valid and its change number is higher than the one it already holds.

**Reason.** SQLite with SQLCipher is one storage engine for all five systems, already bundled with the package.

**Limitation.** On Windows, the key store protects the private key from other user accounts, but, by inference, not from other programs that run as the same user.

**What will be tested.** The encryption of the main database, and how much it slows search and synchronisation, is tested on Windows, Android and Ubuntu 24.04. The password vault is tested on Windows and Android. The Keychain on Apple systems is not tested (section 1).

## 6. Phones and tablets in the background

**Decision.** iPhone and iPad are reachable only while the app is open. Android has an optional "Stay connected" mode, which runs a foreground service of the connected-device type and keeps its network while the phone rests (Doze).

**Reason.** iOS stops the app's connections when it suspends the app. On Android, a foreground service is the only reliable way to stay reachable, and the system allows it.

**Limitation.** On Android, the mode shows a notification all the time and uses battery. On iPhone and iPad, nothing works while the app is closed.

## 7. How the main facts were checked

- **Dart's TLS server.** The server side of Dart's TLS has no callback that can accept an unknown client certificate for each connection; such a callback exists only on the client side. The only alternative is to add each paired device's certificate to the server's trust store in advance, which is documented only for client connections and offers no way to remove a certificate. The connecting device therefore proves its key inside the channel. Basis: the Dart SDK source code, read at commit `ae783c4` of 30 October 2026.
- **AI speed.** A published measurement of the same image model processed 51,126 previews in 57 minutes 39 seconds, about 14.8 previews per second, on a 4-core, 8-thread Intel i7-8559U processor with no graphics card. For 100,000 photos this gives 100,000 ÷ 14.8, about 6,760 seconds, or about 1.9 hours. The estimate of about 2 hours does not include making and sending the previews, or videos. Sending the previews (about 15 KB each, about 1.5 GB for 100,000 photos) takes an estimated few minutes on a home network.
- **Android in the background.** Android's network policy source code allows network access during Doze and Battery Saver to a process that runs a foreground service. Two exceptions remain: Android's "Low Power Standby" mode, which allows only apps on the screen or on its allowlist, and the battery managers of some phone makers.
- **Linux test system.** Ubuntu 24.04 is within Flutter's supported range, and it is the system named in the evaluation plan.
- **Arabic.** The photo model lists Arabic among its training languages, but no published Arabic score was found for it. The text model was measured by its authors on Arabic passages (MIRACL): nDCG@10 of 71.4 for the small model.
- **The Windows key store.** Version 4.2.2 of `flutter_secure_storage` stores values in one file, `flutter_secure_storage.dat`, encrypted with Windows DPAPI and no extra secret. Microsoft states that typically only a user with the same logon credential can decrypt such data, and that a roaming profile can carry it to another computer.

## 8. Limitations

The following has not yet been verified: the first AI build time with ONNX Runtime, including videos; the quality of photo search with Arabic words; Android's background behaviour on older (7 to 10) and newer versions and on real phones; a Dart package for ECDSA signing on all systems; the Bonsoir package on all systems; WebRTC with no internet at all and the size it adds to the app; whether screen streaming needs native code; and whether Apple would allow Local Push Connectivity for the app. The Apple systems remain design only, because the team has no Mac.

## 9. Summary

The app is one Flutter app for five systems. Devices find each other with mDNS, talk over TLS 1.3 with keys that are checked at every connection, and stream screens with WebRTC, all on the local network. Two small AI models run on one ordinary laptop or computer. Data is stored in SQLite and encrypted with SQLCipher, with keys kept by the operating system or made from the master password. Windows, Android and Ubuntu 24.04 will be tested; macOS, iPhone and iPad are checked against their documentation only.

## References (opened 31 October 2026)

1. Dart SDK, `sdk/lib/io` (`secure_server_socket.dart`, `secure_socket.dart`, `security_context.dart`), commit `ae783c4`: https://github.com/dart-lang/sdk/tree/main/sdk/lib/io
2. Dart SDK, `runtime/bin` (`secure_socket_filter.cc`, `security_context.cc`, `security_context_macos.cc`): https://github.com/dart-lang/sdk/tree/main/runtime/bin
3. Flutter, supported deployment platforms (Flutter 3.47): https://docs.flutter.dev/reference/supported-platforms
4. `sqlite3` Dart package, hook options (FTS5; SQLCipher and SQLite3MultipleCiphers builds): https://github.com/simolus3/sqlite3.dart/blob/main/sqlite3/doc/hook.md
5. RustDesk, remote control on Android and iOS (battery managers): https://rustdesk.com/blog/rustdesk-remote-control-android-ios
6. Android Developers, power management restrictions: https://developer.android.com/topic/performance/power/power-details
7. Android Open Source Project, `NetworkPolicyManager.java` and `NetworkPolicyManagerService.java`, mirror commit `1cdfff5`: https://github.com/aosp-mirror/platform_frameworks_base
8. `flutter_secure_storage_windows` 4.2.2, source code and README: https://github.com/juliansteenbakker/flutter_secure_storage/tree/main/flutter_secure_storage_windows
9. Microsoft, `CryptProtectData` function: https://learn.microsoft.com/windows/win32/api/dpapi/nf-dpapi-cryptprotectdata
10. Mustafa.net, "Immich without a GPU" (measured on 10 October 2026): https://mustafa.net/2026/10/10/immich-without-gpu-intel-nuc/
11. clip-ViT-B-32-multilingual-v1 model card: https://huggingface.co/sentence-transformers/clip-ViT-B-32-multilingual-v1
12. L. Wang et al., "Multilingual E5 Text Embeddings: A Technical Report", Table 6: https://arxiv.org/abs/2402.05672
13. Apple, TN3179, Understanding local network privacy (Bonjour and the multicast entitlement; opened 21 October 2026): https://developer.apple.com/documentation/technotes/tn3179-understanding-local-network-privacy
14. LocalSend README and RustDesk README (Flutter; opened 21 October 2026): https://github.com/localsend/localsend and https://github.com/rustdesk/rustdesk
15. Team documents: Implementation and Evaluation Plan (sections 3 and 4); Device Discovery, Pairing and Trust; Security and Privacy; Password Management; Duplicate File Detection; Screen Sharing and Remote Control.
