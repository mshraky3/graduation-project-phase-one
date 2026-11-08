# C11: Security and privacy design (first version)

Owner: Ali. Status: first version, 8 Nov 2026. Phase 1 is design only: no code. Builds on the pairing design C1 v2 (draft, `C1-device-discovery-pairing-trust.md`), `B1-architecture-v2.md`, `B6-technology-final.md`, C4, C7, C8 and the other team files [2]. **[PROPOSAL]** = a design choice to settle; **[NOT CHECKED]** = not confirmed in a primary source or by a test.

## 1. The line between C1 and C11 (proposal, for Mahmoud to answer)
I accept the list in C1 section 8:
- **C1 decides:** device identity (key pair, id = hash of the certificate), what is announced, how a device is admitted (QR or code), how every connection is authenticated, how a removal is made and spread (including the exchange of removal lists on every reconnect), and that an unknown device gets no feature data.
- **C11 decides:** what a trusted device **may do** (permissions per feature, section 3); how **stored data** is protected (section 4); the **lost-device policy** beyond the removal itself (section 5); the system key store per platform (B6 point 6); backups.
- **The hook between them:** before any feature data goes to a device, core services answer two questions: *is it trusted (C1)?* and *is this feature switched on for this device (C11)?* Both must be yes.

## 2. Threats and protection
Already fixed by B6 final: TLS 1.3 with the server certificate pinned and a signed challenge from the client (both directions authenticated); SQLCipher for the vault; device keys in the system key store; no internet (B1 rule 1).

| Threat | Protection | What remains |
|---|---|---|
| **Someone on the same Wi-Fi** | Can see announcements and finish TLS on the normal port, but is refused at the signed challenge; the pairing port answers only while the user has a pairing window open, and pairing needs a code or QR a human checks (C1); all traffic is TLS 1.3 | Learns that our app runs and the device ids (C1 threat 2); can disturb discovery (denial of service) |
| **Stolen unlocked phone** | Every feature off until turned on; vault locks after 5 min and on screen lock (C7); deleting, showing passwords and allowing screen control ask again (sections 3 and 5) | Whatever is open on screen; the thief can use features already allowed until the user removes the phone from another device |
| **A trusted device that misbehaves** | It gets only the features switched on for it; delete requests need the owner's own switch and are logged (C4); file lists passed on through it are **signed by the owner device**, so it cannot forge another device's list (answers C3) [PROPOSAL] | It can read what it was allowed to read, send false "remove device" messages (C1 threat 11), and hold back or delay a list it passes on |
| **Malware on one device** | Keys in the system key store; vault mostly locked; main database encrypted (section 4) | Malware running as the user can read memory while the app is open; on Windows, by our inference, it can also read the key store (DPAPI, B6 point 6). No app can fully stop this |
| **A lost device that was switched off** | User removes it from any other device; the removal spreads (C1) | A device that was off still trusts the lost one until it meets another of the user's devices (section 5) |

## 3. Permissions per feature
Every feature is **off by default** on each device and each device pair (B1 v2). "Approve" = who must say yes.

| Feature | Off by default | Who approves | Once or every time |
|---|---|---|---|
| File lists (C3) | Yes | Owner turns on "share my list" per device | Once |
| Photo previews for the AI device (C2) | Yes | Owner of the photos | Once (switch per device) |
| Duplicates: view (C4) | Uses the file lists | - | - |
| Duplicates: **delete on another device** (C4) | Yes | Owner turns on "My other devices may delete duplicates here" | Once; each delete still confirmed by the user, re-checked, and **logged 30 days** on the owner. Android photos and videos: the system's own prompt; other Android files: the app's own confirmation; iPhone/iPad prompt not checked (C4) |
| Search (C5) | Uses the file lists and AI data | - | - |
| File sharing (C6) | Yes | Receiver | Every time, unless the receiver turns on auto-accept from own devices (C6) |
| Passwords (C7) | Yes | User types the master password on each device that joins | Once per device; vault unlock each session |
| Clipboard (C8) | Yes | Each device; sending and receiving separately per device (C8) | Once; never for secrets (C8) |
| Screen sharing (C9) | Yes | User on the shared device | Every time (C9) |
| Remote control (C9) | Yes | User on the controlled device, separate from viewing | Every time; "Stop" always works (C9) |
| Extra screen (C10) | Yes | User on both devices [PROPOSAL, C10 not yet designed] | Every time |

Changing a permission needs the device unlocked and, for delete, passwords and control, the system's fingerprint, face or PIN check [PROPOSAL].

## 4. Data at rest

| Data | Where | Protection |
|---|---|---|
| Password vault | Every device that joined | SQLCipher with a random key wrapped by Argon2id from the master password (C7) |
| Main database: own file list, copies of others' lists, trusted-device list, settings, ignore list, deletion log | Every device | **SQLCipher with a random key kept in the system key store** [PROPOSAL; this changes B6 choice 5, where the main database had no key]. SQLCipher uses AES-256 and checks an HMAC on every page, so changes to the file are detected [1]. Protects a copied file and other user accounts; not malware running as the user |
| Similarity data (vectors) | AI device only | In the main database, so encrypted the same way |
| Device private key | Every device | System key store (Keychain, Android Keystore, libsecret, Windows DPAPI file: B6 point 6) |
| Backups | - | The app's data is excluded from cloud backups (on Android also because the plugin's key cannot be restored [B6 v0, source 29]) [NOT CHECKED per system] |

**Never stored:** files of other devices (except a file the user received with C6); photo previews (deleted after the vector is made); clipboard items (C8); screen frames (C9); the master password and the wrapping key (C7); the plain vault key, except the optional biometric copy inside the system key store (C7).

## 5. A lost device
**Rule [PROPOSAL]:** from any other device, *Devices → Lost device → Remove*. Steps: (1) the removal is written and spread as C1 designs; (2) the AI role moves if the lost device had it (C2); (3) its file list is deleted (C3; C1 suggests hiding it first and deleting after the user agrees); (4) if it had the vault, the app offers **"Change the vault key"** (recommended): an unlocked device makes a new vault key, re-encrypts the vault and sends the new key, wrapped with the master password, only to the remaining devices; each of them asks for the master password once more (its biometric copy is replaced), and the app shows a **new recovery code**, because the old one opens only the old key. This protects entries added from now on. The lost device keeps its old copy, protected only by the old master password, which nothing can erase without a server; so the app advises changing the most important **website passwords** themselves.

**Wording to the user:** "*Ali's phone* was removed. Your devices that are on now will refuse it. Devices that are switched off will refuse it after they next meet one of your devices; until then they may still accept it. If you turn one of them on away from your other devices, remove *Ali's phone* there too. The phone still holds the data it had: change the passwords of your important accounts."

**Open case (B1, C1):** a device that was off still trusts the lost one until it hears about the removal. C1 already exchanges removal lists on every connection before anything else (C1 sections 1.5 and 1.7) and proposes that, at app start, a device waits up to 5 s to reach other trusted devices before it accepts connections. C11 adds only the advice in the wording above. The gap stays if the lost device is the first one it meets; without a server it cannot be closed.

## 6. Privacy
What leaves a device goes only to the user's own trusted devices, and only for features switched on: file lists (names, sizes, dates, hashes: no contents), photo previews to the AI device (not kept), files the user sends, clipboard text the user allows, screen pictures during a session the user accepted. Nothing leaves the local network: no server, no cloud, no account (B1 rule 1). The app has no analytics, no tracking and no advertising, and sends no usage data to us [PROPOSAL].

## 7. Requirements (for B5 and A9)
- SEC-1 The app must send feature data to a device only if it is trusted (C1) and that feature is switched on for that device.
- SEC-2 The app must keep every feature off by default on each device.
- SEC-3 The app must ask the user on the target device every time before screen sharing, remote control or extra screen starts.
- SEC-4 The app must let another device delete files only if the owner turned on the delete permission, must re-check before deleting, and must log every delete for 30 days.
- SEC-5 The app must encrypt the main database with a random key kept in the system key store, and the vault as C7 requires.
- SEC-6 The app must never store other devices' photo previews, clipboard items or screen frames.
- SEC-7 The app must accept a file list passed on by a third device only if the owner device's signature is valid.
- SEC-8 The app must let the user remove a lost device from any other device and must offer to change the vault key.
- SEC-9 The app must show the user, when a device is removed, that switched-off devices learn it only at their next contact (wording in section 5).
- SEC-10 The app must ask for the system's fingerprint, face or PIN before changing the delete, password or control permissions.
- SEC-11 The app must send no data outside the local network and collect no usage data.

## Open points and who settles them
The C1/C11 line (Mahmoud); signed file lists (Mohammed C3); the 5-second wait at start (Mahmoud C1); C10 approval (Ahmed); extra fields for B5: permission per feature and device, deletion log, signatures (Mohammed). **Phase 2 tests:** backup exclusion per system; the speed cost of encrypting the main database [NOT CHECKED].

## Sources (opened 8 Nov 2026)
1. Zetetic, SQLCipher design (AES-256, HMAC-SHA512 per page, PBKDF2 default): https://www.zetetic.net/sqlcipher/design/
2. Team files: `C1-device-discovery-pairing-trust.md` sections 1.7, 4, 8 (branch `claude/pairing-design`); `B1-architecture-v2.md` rules and decisions 9, 11; `B6-technology-final.md` (choices 3 and 5, point 6) and `B6-technology-v0.md` source 29; `C3-file-indexing.md` section 2 (branch `c3-file-indexing`); `C4-duplicate-detection.md` section 5; `C6-file-sharing.md` (on main); `C7-password-management.md`; `C8-clipboard-sync.md` (branch `c8-final`); `C9-screen-sharing-notes.md` (branch `c9-screen-sharing`).
