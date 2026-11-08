# B3: Data-flow diagram, level 1 (first version)

Owner: Mohammed. Status: first version, 8 Nov 2026.
The main processes of the system, the data stores and the data that flows between them. It should sit on top of Samer's context diagram (B2), but **B2 is not in the repository yet** (checked 8 Nov). So the three outside parties below are **[assumed]**; if B2 names them differently, B3 takes B2's names.
Names follow `B1-architecture-v2.md`; data stores match `B5-data-model.md`. Phase 1 is design only: no code.

## 1. Diagram

Editable source: `diagrams/B3-dfd-level1.drawio`; image: `diagrams/B3-dfd-level1.png`.

![B3 data-flow diagram, level 1](../diagrams/B3-dfd-level1.png)

**Outside parties [assumed until B2]:** E1 User; E2 the operating system of each device (file system, photo library, clipboard, screen and input); E3 a new device that is not yet paired (it can only start pairing, B1 rule 2).

**Data stores:** D1 device registry, feature switches and settings; D2 file lists and search index; D3 duplicate groups, delete requests and log; D4 transfers; D5 vault (encrypted); D6 vectors (AI device only). Every store is in the device's local storage (B5).

**Not drawn:** processes 2.0 to 8.0 also exchange data with the same process on the user's other trusted devices, through the communication layer and only after process 1.0 has checked trust. Drawing them would double the diagram; the sequence diagrams of each feature show them.

## 2. Processes

| # | Process | Input | Output | Runs on |
|---|---|---|---|---|
| 1.0 | Pair and trust devices (C1) | pair request and code match (E1); certificate and signed proof (E3, other devices) | trusted devices and their keys, removals (D1) | every device |
| 2.0 | Index files and share lists (C3) | names, sizes, dates and bytes for hashes (E2); folders to scan (E1); list changes from other devices; switches (D1) | own list and copies (D2); free space (D1); list changes to other devices; previews to 9.0 | every device |
| 3.0 | Find duplicates (C4) | sizes and hashes (D2); similar groups (D3, from 9.0); choices and confirmations (E1) | groups, delete requests, log (D3); delete in the system trash (E2, through the owner) | every device |
| 4.0 | Search (C5) | query (E1); names and ages (D2); file references from 9.0 | results (E1); meaning query to 9.0 | every device |
| 5.0 | Share files (C6) | file, target, accept (E1); file bytes (E2) | file bytes to the other device and to its E2; transfer state (D4) | every device |
| 6.0 | Manage passwords (C7) **[assumed, C7 not written]** | master password, entries (E1) | encrypted entries (D5) | every device that uses it |
| 7.0 | Sync clipboard (C8) | copied item (E2); switches (D1) | item to paste (E2 on the other device); nothing stored | every device |
| 8.0 | Screen sharing, control, extra screen (C9, C10) | allow view or control (E1); screen frames, input events (E2) | frames to the viewer, input to the shared device; nothing stored | every device (control: not iPhone/iPad) |
| 9.0 | AI analysis (C2) | file records (D2); previews from 2.0 (not kept); meaning queries from 4.0 | vectors (D6); similar groups (D3); file references to 4.0 | AI device only |

## 3. Checked against the feature designs (8 Nov)

| Design | Fits? | What does not fit, and who should look |
|---|---|---|
| C1 pairing (Mahmoud, PR 38) | Yes | Nothing. Removals and tombstones are kept in D1 |
| C2, C3, C5 (mine) | Yes | - |
| C4 duplicates (Ali, PR 43) | Yes | D3 needed a place for the ignore list and the deletion log; added. C4 needs free space per device in D1: added in C3 and B5 |
| C6 file sharing (Samer, on main) | Mostly | **Tell Samer:** (1) FS-12 "you already have this file" needs the full hash, which C3 and C4 now fill only when needed, so the receiver must hash its files of the same size at that moment; (2) open point 6 asks "Mohammed (C4)", but C4 is Ali's, and open point 5 names "Ahmed (C11)", but C11 is Ali's; (3) C6 says "control channel", B1 calls it the messages connection; (4) C5 needs a "send this file to that device" request from a third device (C5, open point 1) |
| C8 clipboard (Ahmed, PR 39) | Yes | Nothing stored, per-device switches in D1, matches |
| C9 screen sharing (Ahmed, PR 40) | Yes | Nothing stored, matches. C10 not written yet: 8.0 assumes it works like C9 |
| C7 passwords, C11 security | Not written | 6.0 and D5 are assumed; whether D1 and D2 are encrypted waits for C11 (B5, open point 1) |
| B2 context diagram (Samer) | Not found | The three outside parties are assumed; Samer to confirm or send his names |

## 4. Open points and who settles them

| # | Point | Who |
|---|---|---|
| 1 | Names and number of outside parties | Samer (B2) |
| 2 | Process 6.0 and store D5 | C7 owner, Ali (C11) |
| 3 | The four C6 points in section 3 | Samer (C6) |
| 4 | Whether a level-2 diagram is needed for any process (for example 2.0 or 3.0) | Mahmoud |

## Sources (opened 8 Nov 2026)

1. Mahmoud, `B1-architecture-v2.md` (rules and part names) and `C1-device-discovery-pairing-trust.md` (pull request 38).
2. Ali, `C4-duplicate-detection.md` (pull request 43) and `B6-technology-final.md` (pull request 42).
3. Samer, `research/C6-file-sharing.md` (on main).
4. Ahmed, `C8-clipboard-sync.md` (pull request 39) and `C9-screen-sharing-notes.md` (pull request 40).
5. Mohammed, `C2-ai-node.md`, `C3-file-indexing.md`, `C5-unified-search.md`, `B5-data-model.md`.
