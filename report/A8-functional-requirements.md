# A8: Functional requirements (first version)

Owner: Ahmed. Status: first version, 8 Nov 2026. Chapter 3 of the report. Phase 1 is design only: no code.
Collected from the requirement lists of C1, C3, C4, C5, C6, C7, C8, C9, C10 and C11, and from the text of C2 (C2 has no list). **C9 final and C10 are drafts in review** (not pushed yet): their rows will be rechecked when they are pushed. 78 requirements in total. Names follow `B1-architecture-v2.md`.
**[PROPOSAL]** = a design choice or number still to be agreed or tuned. **not checked** = not confirmed in a primary source or by a test.

## How this list was made

- Every "The app must ..." line in the designs was read; lines that say the same thing in two designs were **merged** into one requirement, and the source column lists every origin (for example "C8-7, SEC-1" = C8 requirement 7 and C11 requirement SEC-1).
- Wording is made uniform: "The app must ..." (the subject is the device app on every device unless another device is named).
- Measurable qualities (speed, size, privacy, battery, languages) are in **A9**. Each design's test is in **A10**.
- Where two designs disagree, the stricter rule is written here and the point is listed in section 13 for the owners to settle.

## 1. General rules (all features)

| ID | Requirement: the app must ... | Source |
|---|---|---|
| FR-1 | send feature data to a device only if that device is trusted (C1) and that feature is switched on for it, and check this again on the receiving side | SEC-1, C1 §8, C3-8, C5-4, C7-6, C8-7, C8-8, FS-2, C9-2, C10-1 |
| FR-2 | keep every feature off by default on each device | SEC-2, C8-1, C9-1 |
| FR-3 | exchange each device's capabilities after authentication (host the AI role, be viewed, be controlled, act as extra screen, sync clipboard in the background) and offer an action only when both devices support it | B1 rule 5, C8-13, C9-5, C10-4 |
| FR-4 | show every device as online or offline and, for an offline device, the date of its last data; disable actions that need the offline device and name the device to turn on | C3-9, C5-7, C5-8, C3 §5 |
| FR-5 | ask for the system's fingerprint, face or PIN before changing the delete, password or control permissions | SEC-10 |

## 2. Pairing and trust (C1)

| ID | Requirement: the app must ... | Source |
|---|---|---|
| FR-6 | create a key pair and a self-signed certificate on first start, never replace them, and use the SHA-256 of the certificate as the device id | FR-C1-1 |
| FR-7 | keep the private key in the system key store and out of cloud backups | FR-C1-2 |
| FR-8 | announce only the protocol version and an 8-byte id prefix, never a label, file name or capability | FR-C1-3 |
| FR-9 | open the pairing service and port only while a pairing window is open, and close it after 120 s, on success, on cancel, or after 3 failed confirmations | FR-C1-4, FR-C1-5 |
| FR-10 | offer pairing by QR code and by comparing a 6-digit code; in the QR route abort if the certificate does not match the QR, and accept the one-time secret only once inside the window | FR-C1-6, -7, -8 |
| FR-11 | store a new device only after both users tapped Match; in the code route send the commitment before the nonce | FR-C1-9, FR-C1-10 |
| FR-12 | show a device label as "unverified" until it is trusted, and strip control and right-to-left characters from it | FR-C1-11 |
| FR-13 | refuse every connection from a certificate that is not trusted (except the pairing port during a window), a known id with a different certificate (with a notice), and every removed id | FR-C1-12, -13, -18 |
| FR-14 | on every connection check the certificate hash and a fresh signed challenge, and refuse TLS below 1.3 or a lower protocol version | FR-C1-14, FR-C1-22 |
| FR-15 | allow typing an address by hand, and show a clear message when the Wi-Fi blocks devices or the iOS Local Network permission is off | FR-C1-15, FR-C1-20 |
| FR-16 | pair each new device with every other device; an optional "Introduce my new device" step, if offered, must show a prompt and a 6-digit code on every target device and store nothing before both users tap Match | FR-C1-16 |
| FR-17 | let the user remove a lost device from any trusted device, spread the removal to online devices at once and to others at their next contact, say what the removed device keeps, and offer to change the vault key | FR-C1-17, FR-C1-19, SEC-8, SEC-9 |
| FR-18 | mark a device offline after 3 missed 10-second heartbeats | FR-C1-21 |

## 3. AI device (C2)

C2 has no requirement list; these lines are taken from its answers Q1 to Q3 and worded by me [PROPOSAL, Mohammed to confirm].

| ID | Requirement: the app must ... | Source |
|---|---|---|
| FR-19 | suggest one AI device among the laptops and PCs that meet the minimum hardware, ranked by memory, cores, free disk and how often it is on; let the user confirm or pick another; never a phone or tablet; save the choice on every device | C2 Q1 |
| FR-20 | when the AI device is off, keep all non-AI features working, show "AI paused: *device* is off" with the last results and their date, and offer (not force) moving the role to another capable online device | C2 Q1, C5-9, B1 rule 4 |
| FR-21 | when the role moves, rebuild the similarity data on the new device from the lists it holds and delete it on the old one; never copy similarity data | C2 Q1 |
| FR-22 | ask the owner device for a 224 × 224 preview of a photo (3 frames of a video) only while it is online and only if "Allow the AI device to see photo previews" is on for it; delete each preview after use; say when a device's photos are left out | C2 Q3, C3 §4, FR-C4-4, C5-10, SEC-6 |

## 4. File indexing and offline records (C3)

| ID | Requirement: the app must ... | Source |
|---|---|---|
| FR-23 | list the files in the folders the user allows and show each device's scan scope | FR-C3-1 |
| FR-24 | store per file its path, name, size, modified date, type, change number and "deleted" mark, and never its content | FR-C3-2 |
| FR-25 | make a perceptual hash for each photo, and fill the partial and full hash only when another feature needs them | FR-C3-3, FR-C4-2 |
| FR-26 | notice new, changed and deleted files (desktops: system notifications plus a check every 6 hours; phones and tablets: at each app start) and re-read a file only when its size or date changed | FR-C3-4, FR-C3-5 |
| FR-27 | keep a read-only copy of the list of every device that shares its list, receive only the changes after the last change number, and send a full list only to a device that has none or was away more than 30 days | FR-C3-6, FR-C3-7 |
| FR-28 | accept a list passed on by a third device only if the owner device's signature is valid *(see conflict K2)* | C3 §3, SEC-7 |
| FR-29 | delete the list of a removed device as soon as it learns of the removal; the AI device also deletes that device's similarity data | FR-C3-10, C4 §6 |
| FR-30 | store each device's free and total space, updated at each scan and each connect | FR-C3-11 |

## 5. Duplicate files (C4)

| ID | Requirement: the app must ... | Source |
|---|---|---|
| FR-31 | treat files with equal size and equal SHA-256 as exact copies, checked in three steps (size, partial hash, full hash), with only the owning device reading a file | FR-C4-1, FR-C4-2 |
| FR-32 | find resized copies of a photo with the perceptual hash, without the AI device (distance ≤ 5 [PROPOSAL]) | FR-C4-3, C4 §3 |
| FR-33 | find similar photos on the AI device (similarity ≥ 0.90 [PROPOSAL]) and similar videos from 3 frames | FR-C4-4, C4 §3 |
| FR-34 | show groups as exact, same photo or similar, on one or several devices, with the space that can be freed; show a match that needs a missing hash as "possible" | FR-C4-5, NFR-C4-1 |
| FR-35 | delete a file only on its owning device, after the user confirmed, after the owner allowed deletes from the asking device, after re-hashing it and confirming the kept copy still exists; never delete the last copy; log every delete for 30 days *(see conflict K3)* | FR-C4-6, SEC-4 |
| FR-36 | move deleted files to the system trash where there is one | FR-C4-7 |
| FR-37 | share the ignore list, so an ignored group does not appear again on any device | FR-C4-8 |

## 6. Unified search (C5)

| ID | Requirement: the app must ... | Source |
|---|---|---|
| FR-38 | search the file names of all trusted devices, including switched-off ones, from the lists it holds, with no other device online | FR-C5-1 |
| FR-39 | find a name from part of a word of 3 letters or more, ignoring case, Arabic vowel marks and the forms of alef, ya and ta marbuta | FR-C5-2 |
| FR-40 | filter results by type, date, device and size | FR-C5-3 |
| FR-41 | send a search by meaning only to the AI device, and the AI device must return only file references and scores and not store the query or its vector | FR-C5-4, FR-C5-5 |
| FR-42 | show name matches and meaning matches in two separate sections | FR-C5-6 |
| FR-43 | keep the search history only on the device where it was typed, and let the user clear it | FR-C5-11 |

## 7. Direct file sharing (C6)

| ID | Requirement: the app must ... | Source |
|---|---|---|
| FR-44 | let the user choose one or more files and one paired device | FS-1 |
| FR-45 | ask the receiving user to accept or decline, unless auto-accept for own devices is on (off by default) | FS-3, SEC-2 |
| FR-46 | show progress on both devices and let either user cancel at any time | FS-4 |
| FR-47 | send in chunks on a separate file connection, keep received chunks when it drops, resume with only the missing chunks, and show "Paused" and then "Failed – tap to retry" | FS-6, FS-7, FS-8, B6 final choice 3 |
| FR-48 | check the fingerprint of the finished file and not save a file that does not match | FS-9 |
| FR-49 | keep a transfer for an offline device waiting up to 24 hours [PROPOSAL] and delete the temporary files of cancelled or expired transfers | FS-10, FS-13 |
| FR-50 | never overwrite a file (save under a new name) and tell the user when the receiver already has an identical file | FS-11, FS-12 |
| FR-51 | save received files where each system allows and tell the user where | FS-14 |

## 8. Password manager (C7)

| ID | Requirement: the app must ... | Source |
|---|---|---|
| FR-52 | keep the vault in a separate SQLCipher file with a random key stored only wrapped (Argon2id from the master password) or behind biometrics in the key store, and never store or send the master password, the wrapping key or the plain vault key | FR-C7-1, FR-C7-2 |
| FR-53 | lock the vault on app close, on screen lock and after 5 minutes without use; allow biometrics only through the key store; ask for the master password after a restart | FR-C7-3, FR-C7-4 |
| FR-54 | import logins from a CSV file and offer to delete the file afterwards | FR-C7-5 |
| FR-55 | sync the vault only with trusted devices that joined it, encrypted with the vault key, and keep both versions when one entry changed on two devices | FR-C7-6, -7, -8 |
| FR-56 | mark copied passwords as secret, never send them through clipboard sync, and clear them after 30 seconds where the system allows | FR-C7-9, C8-6 |
| FR-57 | show the recovery code once at setup and say that a forgotten master password cannot be reset | FR-C7-10 |

## 9. Clipboard sync (C8)

| ID | Requirement: the app must ... | Source |
|---|---|---|
| FR-58 | send copied text up to 1 MB automatically to trusted online devices with receiving on, where the system allows; send an image (PNG or JPEG, up to 10 MB) only when the user taps "Send to other devices" | C8-2, C8-3 |
| FR-59 | offer a "Send clipboard" button on every device; it is the way to send on Android, iPhone/iPad and GNOME on Wayland | C8-4 |
| FR-60 | refuse items over the limit with "Too large to sync. Use file sharing" | C8-5 |
| FR-61 | never send items that another app marks as secret | C8-6 |
| FR-62 | put a received item on the clipboard ready to paste, not send it on again, and on iPhone/iPad write it as local only | C8-9, C8-14 |
| FR-63 | not store or queue clipboard items, and say "No device online to receive it" when nobody could receive | C8-10, C8-11, SEC-6 |
| FR-64 | let the user pause sync for 1 hour or until turned on, and turn sending or receiving off per device | C8-12 |

## 10. Screen sharing and remote control (C9)

From the C9 final draft, section 6 (17 items, in the project folder, rechecked after its review on 8 Nov; not yet pushed). C9-1 is merged into FR-2, C9-2 (trusted) into FR-1, C9-5 into FR-3, C9-14 into A9 NFR-2.

| ID | Requirement: the app must ... | Source |
|---|---|---|
| FR-65 | start a session only after the user on the shared device accepts that request, ask for view and for control separately ("Allow view / Allow view and control / Deny"), and not offer control on a device that cannot be controlled (iPhone/iPad; Android without the accessibility service); no unattended access | C9-2, C9-3, SEC-3 |
| FR-66 | let the user of the shared device pick the whole screen or one window through the system picker where there is one (Windows, macOS 14+, Android 14+, Linux on Wayland, iOS 27+), and share only the picture, never audio, in v1 | C9-4, C9-13 |
| FR-67 | send the stream with WebRTC set up only over the trusted messages connection (no STUN or TURN, local-network candidates only), end the session if the DTLS fingerprint differs from the one received there, and keep request, answer, Stop and setup on the messages connection with the video on its own connection | C9-6, C9-7, C9-8, C10-5, B6 final choice 3 |
| FR-68 | accept mouse and keyboard events only from the viewer of the current session and only if control was allowed for it | C9-9 |
| FR-69 | show on the shared device an always-on-top bar "Being viewed by …" or "Controlled by …" with Stop, end capture within 1 s of Stop on either device [PROPOSAL], and keep the local mouse and keyboard working | C9-10, C9-11, C9-12 |
| FR-70 | stop the session when either device goes off, sleeps, locks or leaves the network, show "Connection lost", ask for a new approval after 30 s [PROPOSAL], and show "Cannot start the stream on this network" when the devices cannot reach each other directly | C9-15, C9-16 |
| FR-71 | on Android, show the in-app disclosure that Google Play requires before the user turns on the accessibility service for control | C9-17 |

## 11. Extra screen (C10)

From the C10 first version, sections 4 and 7 (11 items, in the project folder, rechecked after its review on 8 Nov; not yet pushed). C10-1 is merged into FR-1, C10-4 into FR-3, C10-5 into FR-67, the "never store frames" part of C10-11 into A9 NFR-2.

| ID | Requirement: the app must ... | Source |
|---|---|---|
| FR-72 | create a virtual screen at the screen device's resolution where the computer allows it (Windows with our driver, Linux on Wayland through the portal `VIRTUAL`), and otherwise mirror an existing screen and tell the user | C10-2, C10-3 |
| FR-73 | turn touches on the screen device into mouse events on the extra screen only | C10-6 |
| FR-74 | remove the virtual screen when the session ends for any reason | C10-7 |
| FR-75 | keep the screen device awake while it shows the extra screen, and stop the session when the app leaves the foreground on iPhone/iPad | C10-8 |
| FR-76 | let the user stop the extra screen from either device at any time | C10-9 |
| FR-77 | ask for the system permission where the system demands it (Wayland portal, macOS Screen Recording) and use no private API in v1 | C10-10 |
| FR-78 | ask the user on both devices every time before an extra screen starts, in extend and in mirror mode *(see conflict K4)* | C10-11, C10 §3, §4, SEC-3 |

## 12. Security and privacy rules that are not above (C11)

SEC-5 (encryption of the main database) and SEC-11 (no data outside the local network) are qualities, so they are in A9 (NFR-7, NFR-1). Every other SEC line is merged above: SEC-1 → FR-1, SEC-2 → FR-2, SEC-3 → FR-65 and FR-78, SEC-4 → FR-35, SEC-6 → FR-22 and FR-63, SEC-7 → FR-28, SEC-8 and SEC-9 → FR-17, SEC-10 → FR-5.

## 13. Conflicts between designs, sent to the owners

| # | Conflict | Rule used here | Owners to settle |
|---|---|---|---|
| K1 | Main database: C11 SEC-5 encrypts it with a random key; B6 final choice 5 and C3 open point 5 (B5) open it without a key | A9 NFR-7 as [PROPOSAL], pending | Ali (B6, C11), Mohammed (B5, C3) |
| K2 | List passed by a third device: C3 §3 keeps the copy with the higher change number (no signature); C11 SEC-7 accepts it only with the owner's signature | the C11 rule (stricter) | Mohammed (C3), Ali (C11) |
| K3 | Delete from another device: C4 FR-C4-6 only needs the user's confirmation; C11 SEC-4 and SEC-10 also need the owner's delete permission and a fingerprint/PIN | both, merged in FR-35 and FR-5 | Ali (C4 and C11: align C4's wording) |
| K4 | Approval for the extra screen: SEC-3 says "on the target device"; C11's table and C10 §3 and §4 say "user on both devices, every time", for extend and mirror | C10's rule (both devices, both modes) | Ali (C11: align SEC-3 wording) |
| K5 | Wrong owner names in C6 open points: point 5 says "Ahmed (C11)" (C11 is Ali's) and point 6 says "Mohammed (C4)" (C4 is Ali's) | no change to the requirements | Samer (C6) |

## Open points and who settles them

| # | Point | Who | Status |
|---|---|---|---|
| 1 | Conflicts K1 to K5 above | the owners named | sent 8 Nov 2026, waiting |
| 2 | Recheck sections 10 and 11 when C9 final and C10 are pushed (they may change after review) | Ahmed (C9, C10) | drafts read 8 Nov |
| 3 | C10 open points (Windows driver signing, Linux desktops with `VIRTUAL`) may change FR-72 | Mahmoud, Ali (B6) | not checked |
| 4 | Confirm the C2 lines, which I worded from C2's text | Mohammed (C2) | [PROPOSAL] |
| 5 | C1 v2 is a draft not yet reviewed by its owner; its 22 requirements may change | Mahmoud (C1) | not checked |
| 6 | C6 is a first version written before B6 final; FS-6 is now confirmed by B6 final choice 3 | Samer (C6) | partly settled |

## Sources (all opened 8 Nov 2026)

1. Mahmoud, `C1-device-discovery-pairing-trust.md` §6 and §8 (branch `claude/pairing-design`, pull request 38); `B1-architecture-v2.md` rules 1 to 6 (same branch).
2. Mohammed, `C2-ai-node.md` (third version) and `C3-file-indexing.md` (final) §3, §4, §6, §7 (on main, pull request 41); `C5-unified-search.md` §8 (on main, pull request 44).
3. Ali, `C4-duplicate-detection.md` §3, §6, §8 (pull request 43); `C7-password-management.md` §9 (pull request 48); `C11-security-privacy.md` §3, §7 (pull request 49); `B6-technology-final.md` choices 3 and 5 (pull request 42).
4. Samer, `research/C6-file-sharing.md` §8 and §9 (on main).
5. Ahmed, `C8-clipboard-sync.md` §7 (branch `c8-final`, pull request 39); `C9-screen-sharing-notes.md` §1 to §4 (branch `c9-screen-sharing`, pull request 40); `C9-screen-sharing.md` §6 and `C10-extra-screen.md` §3, §4, §7 (drafts in the project folder, round 3, not pushed).
6. Mahmoud, round-3 task card for Ahmed (PDF, 3 Nov 2026), part B items 1 and 2.
