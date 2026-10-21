# B6: Technology choices (v0, first version)

Owner: Ali. Status: v0, 21 Oct 2026, for milestone M1 (Thu 22 Oct). Answers open decisions 1 to 5 and 12 in `B1-architecture-v1.md`; decision 10 (stream transport) is settled later by C6, C9, C10 with B6. Phase 1 is design only: no code. Names follow B1.

Sources were opened on 21 Oct 2026, mostly as summaries made by a fetch tool. **Not checked** = not confirmed in a primary source or not tested.

## 1. Which operating systems

Options: (a) all five, as the team decided; (b) drop iPhone/iPad, which has the most limits. **My pick: (a), confirmed**, because the limits below are handled by capabilities (B1 rule 5). Minimum versions = what Flutter supports (Q2) [1].

| System | Minimum | Limits |
|---|---|---|
| Windows | 10 | Discovery library (Q3) needs Windows 10 version 1903 or later [19]. |
| macOS | 12 | User must allow Local Network (macOS 15+) [5]. Screen sharing and control need Screen Recording and Accessibility permissions; Screen Recording must be re-approved from time to time [14]. |
| Linux | Ubuntu 20.04, Debian 10 | Discovery needs the Avahi service [19]. On Wayland, screen sharing and control go through a system dialog (xdg-desktop-portal) [15]. Test desktop: **not checked** (suggest Ubuntu 24.04). |
| Android | 7.0 (API 24) | Indexing all files needs "All files access" (Google Play lists on-device file search as a likely allowed use) [13]. Screen capture needs a consent prompt; being controlled needs an accessibility service [4]. |
| iPhone/iPad | 15 | Sees only its own files, the photo library and folders the user picks [9]. No way found to control it from another device (RustDesk cannot [4]; Apple docs **not checked**). Screen sharing only through a broadcast the user starts (ReplayKit): **not checked** (C9). Never the AI device. Test on real devices only [5]. |

**Open, needs the team's answer:** iPhone/iPad and Mac builds need a Mac with Xcode. Who has one?

## 2. One cross-platform app, or one per system

- (a) **Flutter (Dart):** one code base, official support for all five [1]. LocalSend and RustDesk (Flutter UI, Rust core) use it [2][3].
- (b) Kotlin + Compose Multiplatform: all five, but desktop runs on the JVM and macOS needs 13+ on Apple silicon [16].
- (c) Native app per system: best OS access, but four or more code bases for five students.
- Dropped: .NET MAUI has no official Linux support [17].

**My pick: (a)**, because one code base keeps behaviour the same on every device (B1), and two working apps close to our features use it. OS-specific parts (screen capture, input control, Android background service, clipboard) become small native plugins.
**Not checked:** the team's Dart experience; whether screen streaming is fast enough from Dart or needs native or Rust code (as in RustDesk).

## 3. How devices talk on the local network

- (a) A ready peer-to-peer library. libp2p: mDNS, peer id from a key, Noise encryption, many streams; the Dart version is young (no QUIC; its README says two copies on one Mac do not find each other) [18]. iroh 1.0 (June 2026): QUIC, connect by public key, can run local-only, but Rust with bindings for Swift, Kotlin, Python, Node, not Dart [21].
- (b) **mDNS/DNS-SD discovery + TLS 1.3 in both directions with a self-signed key per device, pinned at pairing (C1), + our own small protocol.** Same family as KDE Connect, Syncthing, LocalSend (C1).
- (c) WebRTC: encrypted, screen capture and video on all five [20], but needs our own signalling; heavy for discovery, trust and files.

**My pick: (b)**, because it needs no extra framework, matches C1 (identity = key pair), and on iPhone Bonjour needs only a declared service type and the Local Network permission, while custom UDP broadcast or multicast (KDE Connect, LocalSend) needs a special Apple entitlement [5]. Discovery package: Bonsoir (Bonjour, Android NSD, Windows DNS-SD, Avahi) [19].
Channel types, same trust: messages as frames on one long-lived connection per device; files on a separate connection, in chunks so they can resume (C6); live streams on their own connection for v0, WebRTC as the alternative for C9/C10 (decision 10).
**Not checked:** whether Dart's TLS server can verify a self-signed client certificate (if not: a signed challenge inside the channel, or a native layer); iPhone in the background (Q6). Routers with client (AP) isolation stop devices reaching each other [2], so C1's manual address/QR route is needed.

## 4. AI model and how it runs

Exact duplicates: SHA-256 on every device, no AI. Proposal for C4: a perceptual hash for photos too (finds resized copies, cheap enough for phones).

- (a) **Small embedding models inside the device app, run with ONNX Runtime** (flutter_onnxruntime plugin, CPU on all five [22]): CLIP ViT-B/32 with its multilingual text encoder (50+ languages incl. Arabic, Apache-2.0) [23] for similar photos and finding photos by words; multilingual-e5-small (~0.1B parameters, MIT) for file names and text [24].
- (b) Same with SigLIP 2 base (~0.4B parameters, Apache-2.0) [25]: maybe better, about 4× bigger; Arabic quality **not checked**.
- (c) Local chat model (Ollama, llama.cpp, 3-8B): free questions, but a separate program and several GB of RAM or a GPU; none of our features needs chat.

**My pick: (a)**, because it covers similar files and search by meaning with ~0.3B parameters, runs on a normal CPU, and stays a role of the device app (B1).
**Minimum AI device (proposal, not measured):** 64-bit laptop or PC (Windows 10, macOS 12, Linux), 4 cores, 8 GB RAM (16 suggested), ~3 GB free disk (models ~1.5 GB in fp32 + ~200 MB per 100,000 photos at 512 numbers × 4 bytes). No GPU needed.
**Not checked:** CPU time for 100,000 photos (C2 rebuild time); GPU use; videos; how the AI device gets pixels of photos on other devices (C2 stores no thumbnails, so one small thumbnail must be sent: C2, C3, C4). OpenAI's CLIP card calls deployed use out of scope [26]: fine for a prototype, recheck if we publish.

## 5. How each device stores its data

- (a) **SQLite** through the `sqlite3` Dart package: bundled on all five, can use SQLCipher for encrypted files [27].
- (b) Realm: dropped; MongoDB deprecated its SDKs (end of life 30 Sept 2025), only a community version remains [28].
- (c) JSON files: simple, but no queries over 100,000+ files.

**My pick: (a).** One SQLite file per device: own file index, last known indexes of the others (offline records), trusted-device list, settings. The vault is a second SQLite file encrypted with SQLCipher; its key comes from the master password (C7, C11) and is never stored. The device's private key (C1) goes in the OS key store via `flutter_secure_storage`: Keychain (Apple), Android Keystore, libsecret (Linux) [29].
**Not checked:** full-text search (FTS5) in the bundled SQLite; which key store it uses on Windows (the page does not say); Linux needs a keyring service running [29].

## 6. Phone and iPad in the background

Options: (a) reachable only while the app is open; (b) reachable in the background through an OS mechanism.

- **iPhone/iPad: (a).** When iOS suspends the app, its connections and Bonjour listener fail; Apple advises stopping the listener first, or other devices see a hang [6]. A transfer the user started can finish with BGContinuedProcessingTask (iOS 26+, visible and cancellable, the system can end it) [7]. Local Push Connectivity can wake the app by notification on chosen Wi-Fi networks [8]; whether we can get Apple's permission to use it: **not checked**.
- **Android: (b) as an optional "stay connected" mode:** a connectedDevice foreground service with a visible notification [10]; not dataSync, which is limited to 6 hours per 24 for apps targeting Android 15+ [11]. **Not checked:** whether Doze still cuts its network (the docs do not say which rule wins) [12]; phones whose battery managers kill apps [4].

**Result for B1 (decision 12):** the B1 assumption holds. Phone and tablet are fully reachable only while the app is open; Android can do more if the user turns the mode on.

## Sources (opened 21 Oct 2026)

1. Flutter supported platforms (Flutter 3.47): https://docs.flutter.dev/reference/supported-platforms
2. LocalSend README: https://github.com/localsend/localsend/blob/main/README.md
3. RustDesk README: https://github.com/rustdesk/rustdesk/blob/master/README.md
4. RustDesk, remote control on Android and iOS (updated 14 Oct 2026): https://rustdesk.com/blog/rustdesk-remote-control-android-ios
5. Apple TN3179, Understanding local network privacy: https://developer.apple.com/documentation/technotes/tn3179-understanding-local-network-privacy
6. Apple Developer Forums, NWListener/NWConnection when the app is suspended (Apple DTS, Aug 2026): https://developer.apple.com/forums/thread/840808
7. Apple, BGContinuedProcessingTask: https://developer.apple.com/documentation/backgroundtasks/bgcontinuedprocessingtask
8. Apple, NEAppPushManager (Local Push Connectivity): https://developer.apple.com/documentation/networkextension/neapppushmanager
9. Apple, Providing access to directories: https://developer.apple.com/documentation/uikit/providing-access-to-directories
10. Android, Foreground service types: https://developer.android.com/develop/background-work/services/fgs/service-types
11. Android, Foreground service timeouts: https://developer.android.com/develop/background-work/services/fgs/timeout
12. Android, Power management restrictions: https://developer.android.com/topic/performance/power/power-details
13. Android, Manage all files on a storage device: https://developer.android.com/training/data-storage/manage-all-files
14. LogMeIn, macOS permissions for a remote-control host: https://support.logmeininc.com/pro/help/why-cant-i-connect-to-my-mac-running-on-macos-ventura-and-sonoma
15. xdg-desktop-portal, RemoteDesktop portal: https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.RemoteDesktop.html
16. Compose Multiplatform compatibility and versions: https://kotlinlang.org/docs/multiplatform/compose-compatibility-and-versioning.html
17. The Register, Avalonia brings Linux to .NET MAUI (13 Nov 2025): https://www.theregister.com/2025/11/13/dotnet_maui_linux_avalonia/
18. dart_libp2p package: https://pub.dev/packages/dart_libp2p
19. Bonsoir documentation (installation per platform): https://bonsoir.skyost.eu/docs
20. flutter_webrtc package: https://pub.dev/packages/flutter_webrtc
21. iroh 1.0 announcement: https://iroh.computer/blog/v1
22. flutter_onnxruntime package: https://pub.dev/packages/flutter_onnxruntime
23. clip-ViT-B-32-multilingual-v1 model card: https://huggingface.co/sentence-transformers/clip-ViT-B-32-multilingual-v1
24. multilingual-e5-small model card: https://huggingface.co/intfloat/multilingual-e5-small
25. SigLIP 2 base model card: https://huggingface.co/google/siglip2-base-patch16-224
26. OpenAI CLIP ViT-B/32 model card: https://huggingface.co/openai/clip-vit-base-patch32
27. sqlite3 Dart package: https://pub.dev/packages/sqlite3
28. MongoDB, Atlas Device SDKs deprecation: https://www.mongodb.com/docs/atlas/device-sdks/deprecation/
29. flutter_secure_storage package: https://pub.dev/packages/flutter_secure_storage
