# A10: Implementation and evaluation plan (first version)

Owner: Ahmed. Status: first version, 8 Nov 2026. Chapter 3 of the report. Phase 1 is design only: no code; this file plans Phase 2.
How each part would be built and tested in Phase 2, and what counts as success. The success criteria are the numbers and "Phase 2 tests" already named in the designs; the requirement numbers (FR-n, NFR-n) are from `A8-functional-requirements.md` and `A9-non-functional-requirements.md`.
**[PROPOSAL]** = my plan or number, to agree with the team. **not checked** = not confirmed in a source or a test.

## 1. Build order [PROPOSAL]

Each stage ends with something that can be shown, so if time runs out the earlier stages still make a demo.

| Stage | What is built | Why first |
|---|---|---|
| 0 | App shell (Flutter), local storage (SQLite), C1 pairing, reconnection and removal, capabilities, feature switches (C11 SEC-1, SEC-2) | Every feature needs a trusted connection and the switches |
| 1 | C3 file lists and their sync, C5 name search, C6 file sharing, C8 clipboard | Need no AI and no native capture; they prove the trusted connection carries real data |
| 2 | C4 exact and "same photo" duplicates with safe delete; C11 delete permission and log | Builds on the C3 lists; no AI needed |
| 3 | C2 AI device, C4 similar photos, C5 search by meaning | Heaviest part (models, previews, about 2 h first build) |
| 4 | C9 screen sharing, then C10 extra screen (same WebRTC stream), C7 vault | Need native code per system and the most "not checked" points |

## 2. Per feature: built first, devices, test data, success

Devices: **W** = Windows PC/laptop, **A** = Android phone, **T** = Android tablet, **L** = Linux (see section 4), **M/i** = Mac, iPhone, iPad (not available, section 4).

| Feature | Built first | Devices | Test data | Success criterion (source) |
|---|---|---|---|---|
| C1 Pairing | QR pairing W–A, then code comparison W–W (laptop without camera) | W, A, L | Two to four devices; a test relay for the code route | All 22 tests of C1 §6 pass; pairing ≤ 5 taps and < 60 s with 5 volunteers; reconnection < 3 s; offline mark ≤ 40 s; codes differ in 1,000 relay runs (NFR-5, NFR-13, NFR-14, NFR-33) |
| C3 Lists | Scan of own folders, then "changes after number N" | W, A, L | 100,000 made-up files and folders (as in C3's own measurement) plus real home folders | List ≤ 30 MB per 100,000 files; catch-up sends only the missing changes; offline list shows its date; scan time measured and a target set (NFR-12, NFR-20, FR-27) |
| C5 Search | Name search with the cleaned Arabic copy, then filters | A (minimum phone), W | 400,000 names incl. Arabic names with vowel marks and alef/ya/ta forms | Results ≤ 0.5 s while typing; every Arabic test pair found; works with all other devices off; trigram tokenizer present in the Dart package's SQLite (NFR-10, NFR-37, C5 open point 3) |
| C6 File sharing | One file W→A, then resume | W, A, T | Files of 1 KB, 100 MB, 2 GB; Wi-Fi cut mid-transfer; a file the receiver already has | Resumes with only missing chunks; final fingerprint matches; nothing overwritten; "You already have this file" shown; speed measured (NFR-15, FR-47 to FR-50) |
| C8 Clipboard | Text W↔W, then "Send clipboard" on Android | W, A | Text of 10 B, 1 MB and 1 MB + 1 B; 10 MB image; a password copied from C7 | Over-limit refused with the message; secret items never sent (0 in a capture); nothing stored; delay measured, ≤ 1 s suggested (FR-58 to FR-64, NFR-16) |
| C4 Duplicates | Exact copies by size and hashes, then perceptual hash, then safe delete | W, A | Own photo set [PROPOSAL: about 1,000 photos with hand-labelled copies: resized, cropped, edited, burst shots] plus exact copies of mixed files | All exact copies found, no false exact copy; thresholds 5 and 0.90 tuned on our photos and precision and recall reported; delete never removes the last copy (keep-copy device off → nothing deleted); trash works on Windows from Flutter (C4 Phase 2 tests) |
| C2 AI device | Model run on W, preview request from A | W (4 cores, 8 GB) | 100,000 photos (a public photo set: which one and its licence not checked) | First build time measured and compared with the 2 h estimate; memory stays under 8 GB; "AI paused" shown when W is off; role moves to a second W (NFR-11, NFR-22, FR-19 to FR-22) |
| C5 by meaning | Name vectors on W, then photo search by words | W, A | 50 English and 50 Arabic queries with expected results [PROPOSAL] | Arabic photo search measured against the same English queries (no target yet); query not stored (NFR-3, NFR-38) |
| C9 Screen sharing | View W→A, then control A→W | W, A, L | Desktop, a video playing, a window run as administrator (cannot be controlled, C9 §1) | Approval every time; Stop ≤ 1 s; picture ≤ 5 s; delay < 200 ms; 1080p at 30 fps; works with no internet; Android control only after the accessibility service (NFR-17, NFR-24, C9 open points) |
| C10 Extra screen | Mirror W→T first (no driver), then extend with the Windows driver | W, T, L (Wayland) | Desktop use: drag a window, type, play a video | Mirror works everywhere it is promised; extend on W (if the driver can be signed) and Linux Wayland; delay < 100 ms; virtual screen removed at the end (NFR-18, FR-72 to FR-78) |
| C7 Vault | Vault on W, then sync to A | W, A | Test logins; a Chrome CSV export | Unlock ≤ about 1 s on the minimum phone; locks after 5 min; both versions kept on conflict; CSV import (NFR-19, FR-52 to FR-57) |
| C11 Security | Switches and checks (stage 0), database encryption (stage 2) | all | Fresh install; a device with a feature off; traffic capture | Everything off on a fresh install; feature data refused when its switch is off; 0 packets outside the local network; no previews, clipboard items or frames stored; encryption speed cost measured (NFR-1, NFR-2, NFR-7) |

## 3. Risks: what could fail and the fallback

From the "not checked" and "still open" points of all designs.

| # | Risk | From | Fallback |
|---|---|---|---|
| 1 | No Dart package does ECDSA signing on all five systems, or the client-key proof fails | C1 §9, B6 §4 | Native plugin for signing; the in-channel proof stays the design |
| 2 | mDNS (Bonsoir) fails on a system or the Wi-Fi blocks it | C1 §1.8, B6 §4 | QR with address, typed address (FR-15) |
| 3 | WebRTC does not work with no internet, or adds too much app size | B6 §4, C9 open point 1 | Own TLS stream with compressed frames (C9 §3), slower |
| 4 | Windows virtual-screen driver cannot be signed or installed by the team | C10 open points | Windows gets mirror only (C10 §4) |
| 5 | Linux desktop does not offer the portal `VIRTUAL` source | C10 §1 | Mirror |
| 6 | First AI build much slower than 2 h on ONNX Runtime | C2 still open, B6 point 2 | Run while plugged in and idle, in batches; 16-bit vectors; start with the newest photos |
| 7 | Arabic photo search is weak | B6 point 5, C5 open point 4 | Say so in the app; Arabic name search (FTS) still works; try another model in Phase 2 |
| 8 | Thresholds 5 and 0.90 give wrong groups | C4 Phase 2 tests | Tune on our photos; similar groups are only suggestions and every delete needs the user |
| 9 | Taking video frames fails on phones | C2, C4 | Phones offer exact video copies only |
| 10 | Android Low Power Standby or makers' battery managers stop "stay connected" | B6 point 3 | Android works like iPhone: only while the app is open |
| 11 | Android: no photos without "All files access"; iOS forgets picked folders | C3 open points 2, 3 | Media library only; ask again for the folder; tell the user the scan scope |
| 12 | Google Play refuses the accessibility service for control | C9 open point 8 | Android view-only in a store build; control only in test builds |
| 13 | Trigram tokenizer missing in the Dart package's SQLite | C5 open point 3 | Normal tokenizer on the cleaned names plus prefix search |
| 14 | Argon2id with 64 MiB takes over 1 s on phones | C7 Phase 2 tests | 32 MiB (Bitwarden's default), still above OWASP's minimum |
| 15 | Encrypting the main database slows search or sync | C11 Phase 2 tests | Decide K1 with the measurement: encrypt or keep B6's no-key choice |
| 16 | No trash access from Flutter on desktops | C4 Phase 2 tests | Native call; or permanent delete with a second confirmation |
| 17 | macOS and iOS points stay unchecked (indicators, Accessibility for events, broadcast limits) | C9 open points 6, 7; C10 | Stay design only, marked "not tested" in the report |
| 18 | Team time and learning Dart | B6 choice 2 | The build order in section 1: each stage is a demo on its own |

## 4. Which systems the prototype will really run on

Nobody in the team has a Mac or a Linux machine (B6 §4, B1 decision 15). What that means:

- **Windows and Android: yes.** These are the main prototype pair: every feature in section 2 can be tested on them (Android control only with the accessibility service). Which Android phones and tablets the team has, and whether one is a "minimum phone" (Android 7): **not checked**, to collect from the team.
- **Linux: possible without buying anything [PROPOSAL].** Ubuntu 24.04 from a live USB or a second partition on a team PC (B6 says "test on Ubuntu 24.04"). A virtual machine is easier but its Wayland portals and network may differ: **not checked**. Without this, every Linux claim (Wayland portals, `VIRTUAL`, Avahi) stays from the docs only.
- **macOS, iPhone, iPad: no.** Flutter builds for Apple systems need a Mac with Xcode (B6 choice 1). Without one: no iOS or macOS build, so the iPad as extra screen (the main C10 example), ReplayKit sharing, iOS clipboard "local only", AutoFill and every Apple indicator stay **design only, not tested**. Ways to get one (a university Mac lab, borrowing a Mac, a rented cloud Mac): **not checked**, for Mahmoud to decide.
- **In the report** every result must say on which system it was measured, and features that ran only on Windows and Android must not be described as tested on all five.

## 5. What counts as success for Phase 2 [PROPOSAL]

1. Stage 0 to 2 work on Windows and Android with the C1, C3, C5, C6, C8 and C4 criteria above.
2. The AI first build and the Arabic queries are **measured**, even if slower or weaker than hoped, and the numbers replace the estimates in C2, B6 and A9.
3. Every A9 line marked "no number yet" has a measured number.
4. Every risk that happened is reported with the fallback that was used.

## Open points and who settles them

| # | Point | Who | Status |
|---|---|---|---|
| 1 | Accept the build order and the stage that must work for the presentation | Mahmoud | [PROPOSAL] |
| 2 | Which phones and tablets the team has; is any Android 7 | the team | not checked |
| 3 | A Mac for Apple builds; a Linux live USB | Mahmoud, the team | still open |
| 4 | The photo test set (own photos vs a public set and its licence) and the query set | Mohammed (C2, C4 with Ali) | not chosen |
| 5 | Targets I suggested (clipboard ≤ 1 s, 4 of 5 volunteers, about 1,000 test photos) | the team | [PROPOSAL] |
| 6 | Signing a Windows driver for C10 | Mahmoud, Ali (B6) | not checked |

## Sources (all opened 8 Nov 2026)

1. Phase 2 tests and open points in: `C2-ai-node.md`, `C3-file-indexing.md` §7, `C5-unified-search.md` §9 (Mohammed, on main); `C4-duplicate-detection.md` (pull request 43), `C7-password-management.md` (pull request 48), `C11-security-privacy.md` (pull request 49), `B6-technology-final.md` §4 (pull request 42) (Ali); `C1-device-discovery-pairing-trust.md` §6, §9 and `B1-architecture-v2.md` decision 15 (Mahmoud, pull request 38); `research/C6-file-sharing.md` §9 (Samer, on main).
2. Ahmed: `C8-clipboard-sync.md` (pull request 39); `C9-screen-sharing.md` §6 and open points, `C10-extra-screen.md` §1, §4 and open points (round-3 drafts, not pushed).
3. `A8-functional-requirements.md` and `A9-non-functional-requirements.md` (this round).
4. Mahmoud, round-3 task card (PDF, 3 Nov 2026), part B item 3.
