# Implementation and Evaluation Plan

## 1. Introduction

This chapter explains how the tool will be built and tested in Phase 2, and what will count as success. It sets out the order in which the features are built, how each feature is tested and on which devices, which risks could stop a feature and what is done instead, and which operating systems the team can and cannot test.

The success criteria are the numbers and tests already named in the feature designs. Requirement numbers (FR-n for functional, NFR-n for non-functional) refer to the requirements appendix.

## 2. Build order

The tool is built in stages. Each stage ends with something that can be shown, so that if time runs short the earlier stages still form a working demonstration.

| Stage | What is built | Why at this point |
|---|---|---|
| 0 | The app shell (Flutter), local storage (SQLite), Device Discovery, Pairing and Trust (pairing, reconnection and removal, device capabilities), and the feature switches of Security and Privacy | Every feature needs a trusted connection and the switches |
| 1 | File Indexing and Offline-Device Records (file lists and their synchronisation), Unified Search by name, Direct File Sharing, Clipboard Synchronisation | They need no AI and no native screen capture, and they prove that the trusted connection carries real data |
| 2 | Duplicate File Detection for exact copies and "same photo" copies, with safe deletion; the delete permission and deletion log of Security and Privacy | Builds on the file lists; needs no AI |
| 3 | AI Analysis on the AI device, Duplicate File Detection for similar photos, Unified Search by meaning | The heaviest part (models, previews, and a first build of about two hours) |
| 4 | Screen Sharing and Remote Control, then Extra Screen (the same WebRTC stream), then Password Management | These need native code for each system and have the most points still to verify |

## 3. Which systems can be tested

No member of the team has a Mac or a Linux machine. The tool is designed for five systems, but it will **not** be tested on all five. The table below states what is tested, what is not, and how each untested part is checked instead.

| System | Tested in Phase 2? | How it is tested, or how it is checked instead |
|---|---|---|
| **Windows** | **Tested** | On the team's own Windows PCs and laptops. Windows and Android are the main pair for the prototype: every feature in section 4 is tested on them. |
| **Android** (phone and tablet) | **Tested** | On the team's own phones and tablets. Remote control of Android is tested only after the accessibility service is turned on. Which models the team has, and whether one runs the minimum version (Android 7), is still to be collected. |
| **Linux** | **Tested in part** | Only on Ubuntu 24.04 (GNOME on Wayland), started from a USB stick or a second partition on a team member's Windows PC. A virtual machine is not used for the Wayland and network tests, because its system dialogs and network may behave differently. Linux on X11 and other desktops (KDE, Sway) are not tested; they are checked by review of the documentation and the design. |
| **macOS** | **Not tested** | Apps for Apple systems can only be built on a Mac with Xcode, which the team does not have. Checked instead by review of Apple's documentation and of the design. |
| **iPhone and iPad** | **Not tested** | Cannot be built or tested without a Mac. Checked instead by review of Apple's documentation and of the design. This covers the iPad as an extra screen, screen sharing from an iPhone or iPad, writing clipboard items as "local only", password AutoFill and every Apple system sign. |

In the report, every result names the system on which it was measured, and a feature that ran only on Windows and Android is never described as tested on all five systems. If a Mac becomes available (for example in a university laboratory), the Apple parts are added to the tests; until then they remain design only.

## 4. Tests for each feature

Devices: **W** = Windows PC or laptop, **A** = Android phone, **T** = Android tablet, **L** = Ubuntu 24.04 from a USB stick or second partition. macOS, iPhone and iPad are not available (section 3).

| Feature | Built first | Devices | Test data | Success criterion |
|---|---|---|---|---|
| Device Discovery, Pairing and Trust | Pairing with a QR code between W and A, then pairing by comparing codes between two W (a laptop without a camera) | W, A, L | Two to four devices; a test relay for the code route | All 22 tests of the pairing design pass; pairing takes 5 taps or fewer and under 60 seconds with 5 volunteers; reconnection under 3 seconds; a device is marked offline within 40 seconds; codes differ in 1,000 relay runs (NFR-5, NFR-13, NFR-14, NFR-33) |
| File Indexing and Offline-Device Records | Scan of the user's own folders, then "changes after number N" | W, A, L | 100,000 generated files and folders, as in the design's own measurement, plus real home folders | A list takes 30 MB or less per 100,000 files; catching up sends only the missing changes; an offline device's list shows its date; the scan time is measured and a target is set (NFR-12, NFR-20, FR-27) |
| Unified Search | Search by name with the cleaned Arabic copy of each name, then the filters | A (minimum phone), W | 400,000 names, including Arabic names with vowel marks and different forms of alef, ya and ta marbuta | Results appear within 0.5 seconds while typing; every Arabic test pair is found; search works with all other devices off; the trigram tokenizer is present in the SQLite of the Dart package (NFR-10, NFR-37) |
| Direct File Sharing | One file from W to A, then resuming | W, A, T | Files of 1 KB, 100 MB and 2 GB; Wi-Fi cut during a transfer; a file the receiver already has | The transfer resumes with only the missing parts; the final fingerprint matches; nothing is overwritten; "You already have this file" is shown; the speed is measured (NFR-15, FR-47 to FR-50) |
| Clipboard Synchronisation | Text between two W, then "Send clipboard" on Android | W, A, L | Text of 10 B, 1 MB and 1 MB + 1 B; a 10 MB image; a password copied from Password Management | Items over the limit are refused with the message; secret items are never sent (none in a network capture); nothing is stored; the delay is measured, with 1 second or less suggested (FR-58 to FR-64, NFR-16) |
| Duplicate File Detection | Exact copies by size and fingerprint, then similar photos, then safe deletion | W, A | About 1,000 of the team's own photos with hand-labelled copies (resized, cropped, edited, burst shots), plus exact copies of mixed files | All exact copies are found, with no false exact copy; the thresholds 5 and 0.90 are tuned on these photos and precision and recall are reported; deletion never removes the last copy (if the device holding the kept copy is off, nothing is deleted); moving to the recycle bin works on Windows from Flutter (FR-31 to FR-37, NFR-27) |
| AI Analysis | The model runs on W, then a preview request from A | W (4 cores, 8 GB of memory) | 100,000 photos from a public photo set (the set and its licence are still to be chosen) | The first build time is measured and compared with the estimate of two hours; memory stays under 8 GB; "AI paused" is shown when W is off; the AI role moves to a second W (NFR-11, NFR-22, FR-19 to FR-22) |
| Unified Search by meaning | Name vectors on W, then photo search by words | W, A | 50 English and 50 Arabic queries with expected results | Arabic photo search is measured against the same English queries (no target yet); the query is not stored (NFR-3, NFR-38) |
| Screen Sharing and Remote Control | Viewing from W to A, then control from A to W | W, A, L | The desktop, a video playing, and a window run as administrator (which cannot be controlled) | Approval every time; Stop within 1 second; the picture appears within 5 seconds; delay under 200 ms; 1920 × 1080 at 30 frames per second; works without internet; Android is controlled only after the accessibility service is turned on (NFR-17, NFR-24) |
| Extra Screen | Mirroring from W to T first (no driver), then a true extra screen with the Windows driver | W, T, L | Desktop use: dragging a window, typing, playing a video | Mirroring works wherever it is promised; a true extra screen works on W (with a test-signed driver on the team's test PC) and on L (GNOME on Wayland); approval is asked on both devices every time; the delay is under 100 ms; the virtual screen is removed at the end (NFR-18, FR-72 to FR-78). macOS (mirroring only) and the iPad as screen device are checked by review only (section 3). |
| Password Management | The vault on W, then synchronisation to A | W, A | Test logins; a password export from Chrome (CSV) | Unlocking takes about 1 second or less on the minimum phone; the vault locks after 5 minutes; both versions are kept on a conflict; CSV import works (NFR-19, FR-52 to FR-57) |
| Security and Privacy | The switches and checks (stage 0), then encryption of the main database (stage 2) | W, A, L | A fresh installation; a device with a feature turned off; a network capture | Everything is off on a fresh installation; data for a feature is refused when its switch is off; no packet leaves the local network; no previews, clipboard items or screen pictures are stored; the speed cost of encryption is measured (NFR-1, NFR-2, NFR-7) |

## 5. Risks and fallbacks

The risks below come from the points that the feature designs have not yet verified or have left open.

| # | Risk | Fallback |
|---|---|---|
| 1 | No Dart package performs ECDSA signing on all five systems, or the proof with the client key fails | A native plugin for signing; the proof inside the encrypted channel stays as designed |
| 2 | Device discovery (mDNS, Bonsoir) fails on a system, or the Wi-Fi blocks it | A QR code with the address, or a typed address (FR-15) |
| 3 | WebRTC does not work without internet, or makes the app too large | The tool's own encrypted stream with compressed frames; simpler but slower |
| 4 | The Windows display driver for Extra Screen cannot be signed for the public: a public release needs an EV certificate and Microsoft signing | Phase 2 uses a test-signed driver on the team's own test PC; other Windows PCs get mirroring only |
| 5 | A Linux desktop does not offer the virtual monitor: GNOME and KDE do, Sway and other wlroots desktops do not, and X11 has no such source | Mirroring on wlroots desktops and X11 |
| 6 | The first AI build is much slower than two hours with ONNX Runtime | Run only while plugged in and idle, in batches; store vectors at 16 bits; start with the newest photos |
| 7 | Search for photos with Arabic words gives weak results | Say so in the app; Arabic search by name still works; try another model in Phase 2 |
| 8 | The thresholds 5 and 0.90 group the wrong photos | Tune them on the team's photos; similar groups are only suggestions, and every deletion needs the user |
| 9 | Taking frames from videos fails on phones | Phones offer only exact copies of videos |
| 10 | Android's Low Power Standby or a maker's battery manager stops the background connection | Android works like the iPhone: only while the app is open |
| 11 | Android gives no photos without "All files access", or iOS forgets folders the user picked | Use the media library only; ask for the folder again; tell the user what is scanned |
| 12 | Google Play refuses the accessibility service for remote control | Android is view-only in a store build; control only in test builds |
| 13 | The trigram tokenizer is missing in the SQLite of the Dart package | The normal tokenizer on the cleaned names, plus search by prefix |
| 14 | Argon2id with 64 MiB of memory takes over 1 second on phones | 32 MiB (Bitwarden's default), still above OWASP's minimum |
| 15 | Encrypting the main database slows search or synchronisation | Measure the cost and report it; keep the encryption, which is the settled security decision, and tune the database settings |
| 16 | Flutter on computers cannot reach the recycle bin | A native call, or permanent deletion with a second confirmation |
| 17 | Apple points cannot be tested without a Mac (system signs, the permission for control on macOS, broadcast limits on iPhone and iPad) | They remain design only, checked by review, and are marked "not tested" in the report (section 3) |
| 18 | Team time, and learning Dart | The build order in section 2: each stage is a demonstration on its own |

## 6. What counts as success for Phase 2

1. Stages 0 to 2 work on Windows and Android and meet the criteria in section 4 for Device Discovery, Pairing and Trust; File Indexing and Offline-Device Records; Unified Search; Direct File Sharing; Clipboard Synchronisation; and Duplicate File Detection.
2. The first AI build and the Arabic queries are measured, even if they are slower or weaker than hoped, and the measured numbers replace the estimates in the designs and the non-functional requirements.
3. Every non-functional requirement that has no number yet receives a measured number.
4. Every risk that occurred is reported with the fallback that was used.
5. Every result names the system it was measured on, and the parts that could not be tested are listed as such (section 3).

## 7. Open points

| Point | Settled by |
|---|---|
| Accept the build order, and the stage that must work for the presentation | Project lead |
| Which phones and tablets the team has, and whether any runs Android 7 | The team |
| Access to a Mac for Apple builds; preparing the Ubuntu 24.04 USB stick | Project lead, the team |
| The photo test set (the team's own photos or a public set, and its licence) and the query set | AI Analysis and Duplicate File Detection designs |
| The suggested targets: clipboard delay of 1 second or less, 4 of 5 volunteers, about 1,000 test photos | The team |
| Accept a test-signed driver for the Phase 2 test of Extra Screen on Windows, since public signing is out of reach | Project lead, Technology Choices |

## References

All files were read on 8 and 10 November 2026.

1. Project report, feature designs and their Phase 2 tests and open points: Device Discovery, Pairing and Trust; File Indexing and Offline-Device Records; Unified Search; Direct File Sharing; Duplicate File Detection; AI Analysis; Password Management; Security and Privacy; Clipboard Synchronisation; Screen Sharing and Remote Control; Extra Screen.
2. Project report: Architecture (the open point on a Mac for Apple builds); Technology Choices, final version (Flutter, the Ubuntu 24.04 test desktop, open technical tests).
3. Project report: Requirements for Chapter 3 and the requirements appendix (FR-n and NFR-n).
