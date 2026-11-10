# User Flows

## Introduction

A user flow describes one task from the user's point of view: what the screen shows at each step, what the user does, and what happens when something goes wrong. This section follows three common tasks: setting up the app and connecting two devices, freeing space by removing duplicate files, and finding a file on another device and getting it. Each flow matches one of the three overall sequence diagrams. In every table, the first column gives the text the interface shows, in quotation marks, and the second column gives what the user does.

The example user, Sara, has a Windows laptop and an Android phone.

## Flow A: setting up the app and connecting two devices

This flow matches the overall sequence diagram "pairing a new device and its first scan".

| # | The interface shows | Sara does | What happens in the background |
|---|---|---|---|
| 1 | "Your devices, one network, no internet, no cloud" with "Start" (laptop) | Taps "Start" | The laptop creates its key pair and keeps the private key in the operating system's key store |
| 2 | "Name this device" with the suggestion "Sara's laptop" | Keeps or changes the name | The name is only a label and is never used to decide trust |
| 3 | "All features are off until you turn them on" with "Continue" | Reads the message and taps "Continue" | Every feature switch is off |
| 4 | "Add a device" | Taps "Add a device" | A pairing window opens for 2 minutes |
| 5 | A QR code and a timer (laptop); "Join a device" (phone, after installing and naming it) | Taps "Join a device" on the phone and scans the QR code | The phone checks that the laptop's certificate matches the code |
| 6 | The same 6-digit number on both devices, with "Match" and "Doesn't match" | Checks that the numbers are equal and taps "Match" on both devices | Each device stores the other as paired only after its own user has confirmed |
| 7 | "Paired. Choose what to turn on" | Turns on "Share my file list", "Duplicates", "Search" and "File sharing" | The feature switches are saved for this device |
| 8 | "Which folders?" with Desktop, Documents, Downloads, Pictures, Videos and Music selected (laptop) | Keeps the selection | The scan scope is saved |
| 9 | Android's "All files access" request (phone) | Allows it | The phone can list its shared storage |
| 10 | "Scanning: 40,000 files so far" with "Pause" | Waits, or keeps using the device | The first scan runs in the background; the signed lists are then exchanged and checked |
| 11 | "Laptop can run the AI assistant (similar photos, search by meaning). Use it?" (laptop) | Taps "Use it" and, on the phone, turns on "Allow the AI device to see photo previews" | The laptop becomes the AI device |

**When something goes wrong:**
- **The numbers do not match (step 6):** Sara taps "Doesn't match". Nothing is stored on either device. After three failed tries the window closes and she starts again.
- **The 2-minute window ends:** "Time is up. Tap Add a device to try again." Nothing is stored.
- **The phone cannot scan the code:** on the phone, Sara taps "Use a number instead", picks the laptop from the list and compares the 6-digit number.
- **The devices cannot see each other**, for example because the router blocks traffic between devices: "Cannot find the laptop? Scan the QR code or type its address."
- **Sara refuses "All files access" (step 9):** the phone lists only its photos and videos, and its device row says "Photos and videos only".
- **No laptop or computer meets the minimum for the AI role:** step 11 does not appear, and the settings say "Similar photos and search by meaning need a laptop or computer with 8 GB of memory".

## Flow B: freeing space by removing duplicates

This flow matches the overall sequence diagram "freeing space on the phone, keeping the laptop copy".

| # | The interface shows | Sara does | What happens in the background |
|---|---|---|---|
| 1 | "Duplicates" on the phone's home screen | Taps "Duplicates" | The phone compares sizes and hashes in the lists it holds |
| 2 | Three tabs, "Exact copies", "Same photo, other size" and "Similar", with "You can free 3.2 GB" at the top | Opens "Exact copies" | Groups are labelled "On one device" or "On several devices" |
| 3 | A group with the same video on the phone and the laptop, marked "On several devices: may be your backup", nothing selected | Taps "Free space on this phone" | The app suggests deleting only the phone copies whose other copy stays on the laptop |
| 4 | A list of 120 phone files, each with its laptop copy, and "3.2 GB will be freed" | Checks the list and taps "Delete from phone" | - |
| 5 | "Checking the laptop copies..." | Waits | For each file, the phone reads its copy again, and asks the laptop whether its copy still exists with the same hash |
| 6 | Android's own message "Move 120 items to trash?" | Taps "Allow" | The files are moved to the system trash |
| 7 | "3.2 GB freed. The files are in the trash for 30 days." | - | The deletion is written to the deletion log for 30 days |
| 8 | In "Similar": groups of burst photos, nothing selected, a star on the best photo | Selects two blurred photos and deletes them | Similar photos are never deleted in bulk |

**When something goes wrong:**
- **The laptop is off (step 5):** "Laptop is off. Nothing was deleted. Turn on the laptop to free this space." The request waits; the last copy of a file is never deleted.
- **A file has changed since the scan:** that file is skipped and listed as "Changed since the scan, not deleted".
- **The AI device is off:** the "Similar" tab shows "Paused: Laptop is off" with the last results and their date; the other two tabs keep working.
- **A switched-off device's old list matches a new file:** the group appears as "Possible copy, check when Laptop is back".
- **Sara taps "Deny" in Android's message:** nothing is deleted and the group stays.

## Flow C: finding a file on another device and getting it

This flow matches the overall sequence diagram "finding a file on another device and getting it".

| # | The interface shows | Sara does | What happens in the background |
|---|---|---|---|
| 1 | The search box on the phone | Types "invoice" | The phone searches the names in all the lists it holds |
| 2 | "Name matches" at once: "invoice_march.pdf, Laptop, Online"; a moment later "By meaning": "bill_march.pdf" | Taps the first result | The search by meaning runs on the AI device |
| 3 | The file's details (device, folder, size, date) with "Open", "Send to..." and "Delete" | Taps "Open" | The phone asks the laptop to send the file |
| 4 | "Laptop wants to send invoice_march.pdf (2 MB). Accept?" | Taps "Accept" (this step is skipped if she turned on "Accept files from my devices") | The transfer starts |
| 5 | A progress bar with "Cancel" | Waits | The file travels in chunks over the file connection |
| 6 | "Received. Open" | Taps "Open" | The fingerprint is checked before the file is saved |

"Send to..." works in the same way, with another of Sara's devices as the receiving device.

**When something goes wrong:**
- **The laptop is off (step 2):** the result shows "Laptop: Offline, list from yesterday 21:10", and "Open" and "Send to..." are disabled with "Turn on Laptop to use this file".
- **The AI device is off:** only the name matches appear, with "Search by meaning paused: Laptop is off".
- **The connection drops during the transfer:** "Paused, waiting for Laptop"; the transfer continues by itself from where it stopped.
- **The receiving device's app is closed:** "Waiting for Sara's phone" for up to 24 hours, then "Failed, tap to retry".
- **The phone already has the same file:** "You already have this file".
- **A file with the same name exists:** the new file is saved as "invoice_march (1).pdf"; nothing is overwritten.

## Summary

The three flows show the same principles from the user's side. Nothing happens without the user's confirmation, every feature starts switched off, and a switched-off device never causes a silent failure: the screen always says which device to turn on. The names of buttons and messages are the same as in the feature designs and the screen designs.

## Sources (opened 10 November 2026)

1. Device Discovery, Pairing and Trust design: pairing window, routes, failures (team document).
2. Duplicate File Detection design: groups, safety checks, trash (team document).
3. Direct File Sharing design: offer, acceptance, resuming, waiting (team document).
4. File Indexing and Offline-Device Records, The AI Device and Unified Search (this report).
