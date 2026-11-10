# File Indexing and Offline-Device Records

## Introduction

A user who owns a laptop, a phone and a tablet often cannot remember which device holds a file, and cannot look at the files of a device that is switched off. File Indexing and Offline-Device Records solves this. Every device keeps a list of its own files and a copy of the lists of the user's other paired devices. From any device, the user can therefore see and search the files of all devices, including a device that is switched off, together with the date on which its list was last updated. The lists hold only file names and details, never the contents of the files. This section describes what is stored, how each device scans its files, how the lists travel between devices, and what the user sees when a device is switched off.

## 1. What is stored

For each file, the device stores one row in its database:

| Field | When it is filled | Size | Used by |
|---|---|---|---|
| Device, path, file name, size, last modified date, type | Always | About 150 bytes | Every feature |
| Perceptual hash (photos only) | By the owner device while it indexes the photo | 32 bytes | Duplicate File Detection, to find the same photo in another size |
| Partial hash (first 4 KB, last 4 KB and size) | Only when the file's size occurs more than once | 32 bytes | Duplicate File Detection |
| Full SHA-256 | Only when the file still matches another file after the partial hash, or when Direct File Sharing needs it | 32 bytes | Duplicate File Detection, Direct File Sharing |
| Change number, "deleted" mark and date | Always | A few bytes | Sharing the lists (section 3) |

Hashes are kept together with the path, size and modified date of the file. If any of these change, the stored partial and full hash are cleared and the perceptual hash is made again. Reading every byte of every file on every device would take hours of disk work and battery, so the full hash is computed only when it is needed.

For each device, the registry also stores the free and total space of the storage that holds its scanned files, the time of its last scan, and its scan scope (for example "iPhone: photo library only"). Duplicate File Detection uses the free space to suggest which device to clean.

**Protection.** All of this is kept in the device's main database, which is encrypted with a random key that is kept in the operating system's key store.

**Size.** A list of 100,000 files takes about 20 MB, and its search index about 7 MB more. This was measured once on 100,000 sample file names.

## 2. Scanning: how each device lists its own files

| System | What is scanned by default | Limits |
|---|---|---|
| Windows, macOS, Linux | The user's home folders: Desktop, Documents, Downloads, Pictures, Videos and Music. The user can add folders and external drives, or remove any folder | System and program folders, hidden files and caches are skipped |
| Android | With the "All files access" permission: the shared storage (DCIM, Pictures, Download, Documents and the others) | Without that permission, only photos and videos, through the media library. The app tells the user what is missing |
| iPhone, iPad | The app's own files, the photo library (if the user allows it), and the folders the user picks | Nothing else can be seen |

Files under 64 KB are listed but are left out of duplicate detection. Files that exist only in a cloud service (for example iPhone photos kept only in iCloud) are listed as "Not on this device" and are never read.

**How often:**
- **First scan:** when the user turns the feature on. It runs in the background at low priority and can be paused. The list fills step by step, and the device shows "Scanning: 40,000 files so far".
- **Laptops and computers:** the app listens to the system's file-change notifications, so changes are recorded within seconds. A quick check when the app starts and every 6 hours catches anything that was missed.
- **Phones and tablets:** a quick check each time the app opens, because iPhone and iPad run the app only while it is open, and Android does so unless its optional "Stay connected" mode is on.
- **Quick check:** compares size and modified date only, so no file is read. For a changed photo, the perceptual hash is made again; the other hashes wait until they are needed.

## 3. Sharing the lists between devices

- **Only the owner changes its list.** Every other device holds a read-only copy, so two devices never edit the same list and there are no conflicts to resolve.
- **Change numbers.** Every change to a device's own list (a file added, changed or deleted, or a hash filled in) receives the next number. Each copy remembers the last number it received from that device.
- **Signed lists.** The owner device signs every batch of changes with its private key. **A device accepts a file list only if the owner device's signature is valid and its change number is higher than the one it already holds.** A paired device that passes on a list therefore cannot change it, and an old copy cannot replace a newer one.
- **When two devices connect:** after both devices are checked as paired (Device Discovery, Pairing and Trust), each asks the other for "the changes after number N". The answer arrives in batches over the messages connection. A first full list (about 20 MB per 100,000 files) uses the file connection, so it can resume if the connection is cut.
- **While both devices are on:** new changes are sent in small batches every few seconds.
- **Deleted files:** a deleted file keeps its "deleted" mark for 30 days, so that devices that were switched off learn about it. A device that was away longer receives the full list again instead of the changes.
- **Passing a list on:** a device may pass on its copy of another device's list. For example, the phone rarely meets the computer, but both often meet the laptop. Because the list is signed by its owner, the phone can trust it.
- **Permission:** sharing the list with a device is off until the user turns it on. A device that is not allowed receives nothing.
- **Removed device:** as soon as a device learns that another device was removed, it deletes that device's list, and the AI device deletes the similarity data of its files.

**Figure: the phone catches up with the laptop's list.** Source file: `diagrams/C3-list-sync-sequence.drawio`.

![File Indexing and Offline-Device Records: the phone catches up with the laptop's list](../diagrams/C3-list-sync-sequence.png)

The user opens the app on the phone, and the phone connects to the laptop. The laptop confirms that the phone is paired and that list sharing is on for it. The phone asks for the changes after the last number it holds, number 120, and the laptop sends changes 121 to 150, signed with its key. The phone checks the signature and that the numbers are higher than its own, and only then saves them and shows "Laptop: online, list up to date". If a check fails, the phone keeps its old copy. While both devices are on, new changes arrive every few seconds and are checked in the same way. When the laptop is switched off, the phone notices after three missed heartbeats, keeps the list, and shows its age.

## 4. Photo previews for the AI device

The lists carry no pictures. When the AI device needs to understand a photo, it asks the owner device for one 224 × 224 pixel preview (about 15 KB); for a video, it asks for three frames of the same size, taken at 10 %, 50 % and 90 % of the video. The AI device deletes them after use. This happens only while the owner device is on, and only if the user has turned on "Allow the AI device to see photo previews" for that device. The section on the AI device gives the details.

## 5. Switched-off devices

**What is kept about a switched-off device:** its whole list as last received, the last change number, the time of the last update, the last time it was online, its free space and its scan scope.

**What happens when it is switched off:** nothing is deleted. The device is marked offline after three missed heartbeats (about 30 to 40 seconds), and its list stays as it was.

**What the user sees:**
- The device appears in grey, with "List from today 14:20" or "List from 3 days ago".
- A list older than 7 days carries the warning "This list may be out of date".
- Its files appear in search results and duplicate results with the device name and an "Offline" label. A duplicate that needs a hash the device has not yet computed appears as "Possible copy".
- Actions that need the file itself ("Open", "Send to...", "Delete") are disabled, with the message "Turn on Laptop to use this file".
- Phones and tablets are offline most of the time, because the app is closed, so this is the normal case for them and not an error.

## 6. Requirements

- FR-IDX-1 The app must list the files in the folders the user allows, and show the scan scope of each device.
- FR-IDX-2 The app must store, for each file, its path, name, size, modified date, type, a change number and a "deleted" mark, and never its content.
- FR-IDX-3 The app must make a perceptual hash for each photo it indexes, and fill the partial and full hash only when another feature needs them.
- FR-IDX-4 The app must detect new, changed and deleted files: on laptops and computers through the system's change notifications and a quick check every 6 hours; on phones and tablets each time the app starts.
- FR-IDX-5 The app must read a file again only when its size or modified date has changed.
- FR-IDX-6 The app must keep a read-only copy of the list of every paired device that shares its list with it.
- FR-IDX-7 The app must send only the changes after the last change number the other device holds, and the full list only to a device that has none or was away for more than 30 days.
- FR-IDX-8 The app must sign every batch of list changes with the owner device's private key.
- FR-IDX-9 The app must accept a file list only if the owner device's signature is valid and its change number is higher than the one it already holds.
- FR-IDX-10 The app must send a list only to paired devices for which the user has turned list sharing on.
- FR-IDX-11 The app must keep the list of a switched-off device and show the date of its last update.
- FR-IDX-12 The app must delete the list of a removed device as soon as it learns of the removal.
- FR-IDX-13 The app must store each device's free and total space and update it at each scan and each connection.
- NFR-IDX-1 The first scan must run in the background at low priority and must be possible to pause.
- NFR-IDX-2 A list, with its search index, must take no more than about 30 MB per 100,000 files.
- NFR-IDX-3 Viewing and searching the lists must work with no other device online.
- NFR-IDX-4 The lists must be stored in the encrypted main database.

## 7. Limitations and open points

The following has not yet been verified on real devices and will be tested during implementation: the time needed to scan 100,000 files on a phone and on a laptop; which Android permission gives access to photos and videos without "All files access"; whether iPhone and iPad keep access to a folder the user picked after the app restarts; and the list size, which was measured on sample names in a desktop version of SQLite rather than in the app itself. The extra time needed to encrypt the main database is also to be measured.

## 8. Summary

Each device lists its own files and keeps a signed, read-only copy of the lists of the user's other devices. Lists travel as small, numbered changes, are checked for the owner's signature and a higher change number, and are stored in the encrypted database. A switched-off device stays visible from its last list, with the date of that list, and actions that need the file itself are disabled until the device is on again.

## Sources (opened 10 November 2026)

1. Duplicate File Detection design: hashes, free space per device (team document).
2. Security and Privacy design: encrypted main database, signed file lists (team document).
3. Technology choices, final version: SQLite, platform limits (team document).
4. Device Discovery, Pairing and Trust design: reconnection, heartbeat, removal (team document).
5. Direct File Sharing design (team document).
6. Measurement of 100,000 sample file rows in SQLite 3.42 on Windows, 8 November 2026.
