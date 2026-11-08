# A9: Non-functional requirements (first version)

Owner: Ahmed. Status: first version, 8 Nov 2026. Chapter 3 of the report. Phase 1 is design only: no code.
The qualities the whole app must have, in the eight areas of the task card: privacy, security, performance, working without internet, battery, supported systems, ease of use, Arabic and English. Each line has **how it is measured** and **the target**, taken from the numbers already in the designs. The functional requirements are in `A8-functional-requirements.md`; how each test is run is in `A10-evaluation-plan.md`.
**Status words:** **measured** = we ran it; **source** = a published number; **estimate** = worked out, not run; **[PROPOSAL]** = a target I or the owner chose, to tune in Phase 2; **no number yet** = no design gives one.

## 1. Privacy

| ID | Requirement | How measured | Target | Status | Source |
|---|---|---|---|---|---|
| NFR-1 | No data leaves the local network: no cloud, no server, no account, no usage data | Run every feature with the router's internet cable unplugged, and capture traffic: count packets to non-local addresses | 0 packets; every feature works | [PROPOSAL] test | B1 rule 1, SEC-11, FS-5, NFR-C5-2, FR-C7-12, C8-7, NFR-C1-1 |
| NFR-2 | No file content leaves its device except AI previews; no device stores another device's previews, clipboard items or screen frames | Inspect app storage and temp folders on every device after a test session | nothing found | [PROPOSAL] test | NFR-C4-2, SEC-6, C8-10, C9-14, C10-11 |
| NFR-3 | Search queries stay on the device; the AI device keeps no query text or vector | Inspect the AI device's storage after 100 searches | nothing found | [PROPOSAL] test | FR-C5-5, FR-C5-11 |

## 2. Security

| ID | Requirement | How measured | Target | Status | Source |
|---|---|---|---|---|---|
| NFR-4 | Every connection is TLS 1.3, server certificate pinned, client proves its key with a fresh signed challenge | Connect with a TLS 1.2 client; replay a recorded authentication | both refused | design | FR-C1-14, FR-C1-22, B6 final choice 3 |
| NFR-5 | A man in the middle cannot make both screens show the same pairing code | Test relay between two devices, N = 1,000 runs | codes differ in all runs; chance ≤ 10⁻⁶ per attempt (6 digits) | argument in C1 §3.2 | NFR-C1-4, FR-C1-10 |
| NFR-6 | An unauthenticated peer is limited | Flood test | ≤ 16 connections, ≤ 5 s each, first message ≤ 2 KB | [PROPOSAL] | NFR-C1-5 |
| NFR-7 | Stored data is protected: private key and trusted list not readable by other apps; vault in SQLCipher with Argon2id (64 MiB, 3 passes, 1 thread); main database with a random key *(conflict K1 in A8)* | Read the files from another app and another user account; open a copied database | unreadable | vault: design; main database: [PROPOSAL], pending | NFR-C1-6, FR-C7-1, SEC-5 |
| NFR-8 | Windows limit stated honestly: DPAPI protects the key from other user accounts, not from other programs of the same user | — | stated in the report | source | B6 final choice 5 |
| NFR-9 | A removed device is shut out quickly | Three devices, one offline; time the removal | online devices within 5 s; others at their next contact | [PROPOSAL] | FR-C1-17 |

## 3. Performance

| ID | Requirement | How measured | Target | Status | Source |
|---|---|---|---|---|---|
| NFR-10 | Name search answers while typing | 400,000 names (4 devices × 100,000) on the minimum phone; time from key press to results | ≤ 0.5 s | [PROPOSAL], not measured | NFR-C5-1 |
| NFR-11 | First AI build of 100,000 photos | Model time on a 4-core, 8-thread laptop CPU with ONNX Runtime | about 2 h (previews and videos not included) | estimate from a published measurement (14.8 previews/s) | NFR-C4-3, C2 Q1, B6 final point 2 |
| NFR-12 | Scanning 100,000 files | Time the first scan on a phone and a laptop | **no number yet** (set after the first measurement) | not checked | C3 open point 1 |
| NFR-13 | Reconnection to a trusted device | Same Wi-Fi, no user action; time to "connected" | < 3 s | [PROPOSAL] | NFR-C1-3 |
| NFR-14 | Offline detection | Unplug a device and time it | ≤ about 40 s (3 missed 10 s heartbeats) | design | FR-C1-21 |
| NFR-15 | File transfer resumes after a drop | Cut Wi-Fi during a 2 GB transfer; count bytes sent again | only missing chunks sent. Transfer speed: **no number yet** | [PROPOSAL] | FS-7, C6 §3 |
| NFR-16 | Clipboard delay | Copy on one device, time until paste-ready on the other | **no number yet**; I suggest ≤ 1 s [PROPOSAL] | not in C8 | C8 |
| NFR-17 | Screen sharing | Same Wi-Fi | picture ≤ 5 s after approval; delay < 200 ms; up to 1920 × 1080 at 30 fps; Stop ≤ 1 s | [PROPOSAL], not measured | C9 §6 |
| NFR-18 | Extra screen | Same Wi-Fi, from drawing to showing | < 100 ms; at the screen device's resolution | [PROPOSAL], not measured | C10 §3 |
| NFR-19 | Vault unlock | Master password on the minimum phone | ≤ about 1 s | [PROPOSAL], to test | FR-C7-11, C7 §2 |
| NFR-20 | Storage | Size of the files on disk | lists ≤ 30 MB per 100,000 files incl. search index (measured 27 MB on made-up names); a phone with 4 × 100,000 files ≈ 110 MB; vectors ≈ 200 MB per 100,000 photos (AI device only); name vectors ≈ 600 MB per 400,000 names | measured once (lists); estimates (rest) | NFR-C3-2, C2 Q2, C4 §6, C5 open point 5 |
| NFR-21 | Network use for the AI | Bytes sent for previews | ≈ 1.5 GB once per 100,000 photos (≈ 15 KB each), then only new photos | estimate | C2 Q3 |
| NFR-22 | AI device minimum hardware | — | 64-bit laptop or PC, 4 cores, 8 GB RAM (16 suggested), ≈ 3 GB free disk, no GPU | not measured | B6 final choice 4, C2 Q1 |

## 4. Working without internet and with devices off

| ID | Requirement | How measured | Target | Status | Source |
|---|---|---|---|---|---|
| NFR-23 | Every device works alone: own files, own index, search of the stored lists and exact-copy detection with no other device online | Only one device on, Wi-Fi off | all these work; "possible" shown where a hash is missing | design | B1 rule 3, NFR-C3-3, NFR-C4-1, FR-C5-1 |
| NFR-24 | Pairing, removal and live streams work with no internet (no STUN or TURN) | Router with no internet; airplane mode with Wi-Fi only | all work | pairing: design; WebRTC offline: **not checked** | NFR-C1-1, B6 final section 4, C9 open point 1 |
| NFR-25 | When the AI device is off, only AI features pause | Turn the AI device off | other features unchanged; message shown | design | B1 rule 4, C2 Q1 |

## 5. Battery and device load

| ID | Requirement | How measured | Target | Status | Source |
|---|---|---|---|---|---|
| NFR-26 | The first scan runs in the background at low priority and can be paused | Watch CPU use during the scan while using the device | pausable; **no CPU number yet** | design | NFR-C3-1 |
| NFR-27 | Files are read in full only when needed (hashes filled only for candidates) | Count bytes read during a duplicate check | far below the total size (500 GB would cost hours) | design, not measured | C4 §2, C3 §1 |
| NFR-28 | Android "stay connected" mode | Battery % per hour with the mode on and off, same phone | **no number yet** | not measured | B6 final choice 6 |
| NFR-29 | The AI device does not slow the user's work | CPU and memory while the build runs | **no number yet**; 16 GB suggested if the user works on it | not measured | C2 Q1 |

## 6. Supported systems

| ID | Requirement | How measured | Target | Status | Source |
|---|---|---|---|---|---|
| NFR-30 | One Flutter app on Windows 10+, macOS 12+, Ubuntu 20.04 / Debian 10+, Android 7.0+, iOS/iPadOS 15+ | Install and run on each | runs on all five | decided; macOS and iOS need a Mac with Xcode (nobody has one) | B6 final choices 1, 2 |
| NFR-31 | Pairing works across systems | Test matrix | at least iPhone–Windows, Android–macOS, iPad–Linux, laptop without camera–PC | [PROPOSAL] | NFR-C1-7 |
| NFR-32 | System limits are shown through capabilities, not hidden | Check each device offers only what it can (e.g. iPhone: no control, no AI role; macOS: extra screen as mirror) | no action offered that fails | design | B1 rule 5, C9 §1, C10 §4 |

## 7. Ease of use

| ID | Requirement | How measured | Target | Status | Source |
|---|---|---|---|---|---|
| NFR-33 | First pairing is quick | Lab test with 5 volunteers | ≤ 5 taps and < 60 s | [PROPOSAL] | NFR-C1-2 |
| NFR-34 | Every waiting or limited state says what is happening and what to do ("Paused", "Failed – tap to retry", "AI paused", "List from 3 days ago", "Too large to sync", "Turn on *Laptop-1*") | Review every screen against this list | every state has a message | design; screens in D1 | C3 §5, C5, C6 §3, C8-5, C2 Q1 |
| NFR-35 | Core tasks can be done without help | 5 volunteers, 5 tasks (pair, find a file, send it, remove duplicates, share the screen) | **no number yet**; I suggest 4 of 5 people finish each task unaided [PROPOSAL] | not in any design | — |

## 8. Arabic and English

| ID | Requirement | How measured | Target | Status | Source |
|---|---|---|---|---|---|
| NFR-36 | The whole interface is in Arabic (right-to-left) and English | Review every screen in both languages | all screens, mirrored layout in Arabic | **no design covers this yet** [PROPOSAL]; for D1 | task card |
| NFR-37 | Arabic file names are found as in FR-39 (vowel marks, alef, ya, ta marbuta) | Test list of Arabic names and queries | all test pairs found | tested in Python's SQLite 3.42 only | FR-C5-2, C5 §2 |
| NFR-38 | Search by meaning in Arabic | Text model: published score; photo model: our own Arabic and English query set | text: nDCG@10 71.4 (published, Arabic MIRACL); photos: **no number yet** | text: source; photos: claimed, not measured | B6 final point 5, C5 §3 |

## Open points and who settles them

| # | Point | Who | Status |
|---|---|---|---|
| 1 | Encryption of the main database (NFR-7, conflict K1 in A8) | Ali (C11, B6), Mohammed (B5) | waiting |
| 2 | Numbers still missing: scan time, transfer speed, clipboard delay, battery, CPU, Arabic photo search, task success | Phase 2 tests (A10); owners of C3, C6, C8, B6 | no number yet |
| 3 | The suggested targets in NFR-16 and NFR-35 are mine, not from a design | Ahmed with C8 owner (me) and Mahmoud | [PROPOSAL] |
| 4 | Arabic and English interface: who designs it | Mahmoud (D1) | not in any design yet |
| 5 | macOS and iOS cannot be built or tested without a Mac (NFR-30) | the team (B1 decision 15) | still open |

## Sources (all opened 8 Nov 2026)

1. Mahmoud, `C1-device-discovery-pairing-trust.md` §3.2, §6 (branch `claude/pairing-design`, pull request 38); `B1-architecture-v2.md` rules 1 to 5 (same branch).
2. Mohammed, `C2-ai-node.md` Q1 to Q3, `C3-file-indexing.md` §1, §5, §6, `C5-unified-search.md` §2, §3, §8, §9 (on main).
3. Ali, `B6-technology-final.md` choices 1 to 6, points 2 and 5, section 4 (pull request 42); `C4-duplicate-detection.md` §2, §6, §8 (pull request 43); `C7-password-management.md` §2, §9 (pull request 48); `C11-security-privacy.md` §4, §7 (pull request 49).
4. Samer, `research/C6-file-sharing.md` §3, §8 (on main).
5. Ahmed, `C8-clipboard-sync.md` (branch `c8-final`, pull request 39); `C9-screen-sharing.md` §6 and `C10-extra-screen.md` §3, §4, §7 (round-3 drafts, not pushed).
6. Mahmoud, round-3 task card (PDF, 3 Nov 2026), part B item 2.
