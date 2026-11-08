# D2: User flows (first version)

Owner: Mohammed. Status: first version, 8 Nov 2026.
Three journeys from the user's point of view. Each step says what the user sees and does; the "If something goes wrong" part says what happens when it fails. The flows follow the designs: pairing (C1, pull request 38), file lists (C3), duplicates (C4, pull request 43), search (C5) and file sharing (C6). The screens themselves are D1's (wireframes). Phase 1 is design only: no code. **[PROPOSAL]** = my choice where no design decides it yet.

Example user: Sara, with a Windows laptop and an Android phone.

## Flow A: first set-up and connecting two devices

| # | Sara sees | Sara does | Behind the screen |
|---|---|---|---|
| 1 | Welcome screen on the laptop: "Your devices, one network, no internet, no cloud" | Taps Start | The app makes the laptop's key pair and keeps the private key in the system key store (C1, 1.1) |
| 2 | "Name this device" with a suggestion ("Sara's laptop") | Keeps or changes the name | The name is only a label, never used for trust (C1, 1.1) |
| 3 | "All features are off until you turn them on" and a list of features | Reads it, taps Continue | Every feature switch is off (B1 rule 6) |
| 4 | Laptop: "Add a device" | Taps it | A 2-minute pairing window opens (C1, 1.3) |
| 5 | Laptop shows a QR code and a timer | Installs and opens the app on the phone, names it, taps "Join a device", scans the QR code | The phone checks that the laptop's certificate matches the code (C1, route 1) |
| 6 | Both screens show the same 6-digit number | Checks the numbers are equal, taps **Match** on both | Each device saves the other as trusted only after its own user confirmed (C1, 1.3) |
| 7 | "Paired. Choose what to turn on with *Sara's phone*" | Turns on list sharing, duplicates, search, file sharing | Feature switches saved per device (B5) |
| 8 | Laptop: "Which folders?" with Desktop, Documents, Downloads, Pictures, Videos, Music ticked | Keeps them | C3 scan scope |
| 9 | Phone: Android asks for "All files access" | Allows it (or not: then "Photos and videos only") | C3, section 2 |
| 10 | "Scanning: 40,000 files so far" on both, which can be paused | Waits or keeps using the device | First scan in the background, then the lists are exchanged (C3) |
| 11 | Laptop: "*Sara's laptop* can run the AI assistant (similar photos, search by meaning). Use it?" | Taps Yes, and turns on "Allow the AI device to see photo previews" on the phone | AI device chosen by the user after the app's suggestion (C2) |

**If something goes wrong:**
- **The numbers do not match (step 6):** Sara taps "Do not match". Nothing is saved on either device; after 3 failed tries the window closes and she starts again (C1, 1.3).
- **The 2-minute window ends:** "Time is up. Tap Add a device to try again." Nothing is saved.
- **The phone has no camera, or cannot scan:** on the phone she taps "Use a number instead", picks the laptop from the list and compares the 6-digit number (C1, route 2).
- **The devices cannot see each other** (a router that blocks traffic between devices): "Cannot find the laptop? Scan the QR code or type its address" (C1, 1.8).
- **Sara refuses "All files access":** the phone lists only photos and videos, and its device row says so (C3).
- **No laptop or PC meets the AI minimum:** step 11 is skipped; "Similar photos and search by meaning need a laptop or PC with 8 GB RAM" (C2).

## Flow B: cleaning duplicates and freeing space

| # | Sara sees | Sara does | Behind the screen |
|---|---|---|---|
| 1 | Phone home screen: "Phone storage almost full" [PROPOSAL] and a Duplicates button | Taps Duplicates | The phone compares sizes and hashes in its lists (C4, section 2) |
| 2 | Three tabs: **Exact copies**, **Same photo, other size**, **Similar**; at the top "You can free 3.2 GB" per device | Opens Exact copies | Groups labelled "On one device" or "On several devices" (C4, section 4) |
| 3 | A group: the same video on the phone and on the laptop, "On several devices: may be your backup", nothing ticked | Taps **Free space on Sara's phone** | Suggests deleting only the phone copies whose other copy stays on the laptop (C4, section 5) |
| 4 | A list of 120 phone files, each with its laptop copy, and "3.2 GB will be freed" | Checks, taps **Delete from phone** | - |
| 5 | "Checking the laptop copies..." | Waits | For each file, the phone re-hashes its copy and asks the laptop whether its copy still exists with the same hash (C4, safety checks) |
| 6 | Android's own prompt "Move 120 items to trash?" | Taps Allow | Android's trash request (C4; Android 11 and later) |
| 7 | "3.2 GB freed. The files are in the trash for 30 days." | Done | Deletion log kept 30 days (C4) |
| 8 | Similar tab: groups of burst photos, nothing ticked, a star on the best photo | Ticks two blurry ones, deletes | Similar groups are never deleted in bulk (C4) |

**If something goes wrong:**
- **The laptop is off (step 5):** "Laptop is off. Nothing was deleted. Turn on the laptop to free this space." The request waits; the last copy is never deleted (C4).
- **A file changed since the scan:** that file is skipped and listed: "Changed since the scan, not deleted".
- **The AI device is off:** the Similar tab shows "Paused: *Sara's laptop* is off" with the last results and their date (C2); the other two tabs work.
- **A device is off and a new file matches its old list:** shown as "Possible copy, check when *laptop* is back" (C4).
- **Sara taps Deny in Android's prompt:** nothing is deleted, the group stays.

## Flow C: finding a file and sending it to another device

| # | Sara sees | Sara does | Behind the screen |
|---|---|---|---|
| 1 | Search box on the phone | Types "فاتورة" | Name search in all the lists the phone holds, Arabic marks ignored (C5, section 2) |
| 2 | **Name matches** at once: "فاتورة_مارس.pdf, *Sara's laptop*, online"; a moment later **By meaning**: "bill_march.pdf" | Taps the first result | Meaning search on the AI device (C5, section 3) |
| 3 | File details: device, folder, size, date; buttons **Open**, **Send to...**, **Delete** | Taps **Send to...**, picks *Sara's phone* | The phone asks the laptop to send it (C5, open point 1, needs C6) [PROPOSAL] |
| 4 | Phone: "*Sara's laptop* wants to send فاتورة_مارس.pdf (2 MB). Accept?" | Taps Accept (or nothing, if she turned on auto-accept from her devices) | C6, section 2 |
| 5 | Progress on both devices, with Cancel | Waits | Chunks over the file connection (C6) |
| 6 | "Received. Open / Show in folder" | Taps Open | Fingerprint checked before saving (C6) |

**If something goes wrong:**
- **The laptop is off (step 2):** the result shows "*Sara's laptop*: offline, list from yesterday 21:10"; Open and Send are disabled: "Turn on *Sara's laptop* to use this file" (C3, C5).
- **The AI device is off:** only name matches; "Search by meaning paused: *Sara's laptop* is off" (C5).
- **The connection drops during the transfer:** "Paused, waiting for *Sara's laptop*", then it continues by itself from where it stopped (C6, section 3).
- **The receiving phone's app is closed:** "Waiting for *Sara's phone*" for up to 24 hours, then "Failed, tap to retry" (C6, section 4).
- **The phone already has the same file:** "You already have this file" (C6).
- **A file with the same name exists:** saved as "فاتورة_مارس (1).pdf", never overwritten (C6).

## Open points and who settles them

| # | Point | Who |
|---|---|---|
| 1 | Sending a file that is on another device (Flow C, step 3) | Samer (C6) |
| 2 | The "storage almost full" hint (Flow B, step 1) | D1, Ali (C4) |
| 3 | The screens for every step | D1 (wireframes) |
| 4 | Pairing steps follow C1 v2, which is not yet approved | Mahmoud (C1) |

## Sources (opened 8 Nov 2026)

1. Mahmoud, `C1-device-discovery-pairing-trust.md`, sections 1.1, 1.3 and 1.8 (pull request 38).
2. Ali, `C4-duplicate-detection.md`, sections 2, 4 and 5 (pull request 43).
3. Samer, `research/C6-file-sharing.md`, sections 2 to 4 (on main).
4. Mohammed, `C2-ai-node.md`, `C3-file-indexing.md`, `C5-unified-search.md`.
