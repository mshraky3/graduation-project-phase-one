# Data Model

## Introduction

The assistant keeps everything on the user's own devices: there is no server and no cloud copy. This section describes what each device stores, where each piece of data lives, how the pieces relate to each other, and how they are protected. For the user, three points matter most. First, every device can show the files of all devices, because it keeps a copy of their lists. Second, everything a device stores is encrypted on its disk, and passwords have a second layer of protection. Third, some data is never stored at all, such as photo previews after use, clipboard items and screen images.

## 1. Where data lives

| Place | What it holds | How it is protected |
|---|---|---|
| **Main database**, on every device | Device registry and settings; file lists and search index; duplicate groups and delete log; transfers | **The main database is encrypted with a random key that is kept in the operating system's key store.** The encryption also detects any change to the file |
| **Vault**, on every device that uses Password Management | Vault entries | A separate encrypted file. **The vault key is random and is stored only wrapped by a key made from the master password.** An optional second copy of the vault key is kept behind the fingerprint or face check in the key store |
| **Operating system key store**, on every device | The device's private key, the main database key, and the optional fingerprint copy of the vault key | Protected by the operating system: Keychain on Apple systems, Android Keystore, libsecret on Linux, and a protected file on Windows |
| **AI device only**, inside its main database | Vectors (similarity data) and the queue of previews waiting for a device that is off | The same encryption as the main database |
| **Nowhere** | Photo previews after use, search words on the AI device, clipboard items, screen images, the master password and the key made from it, the one-time pairing secret after pairing | Not stored |
| **Between devices** | Everything that travels | Encrypted connections (TLS 1.3) between paired devices only; vault changes are additionally encrypted with the vault key |

**File lists from other devices.** A device accepts a file list only if the owner device's signature is valid and its change number is higher than the one it already holds. A list passed on by a third device can therefore be trusted, and an old list cannot replace a newer one.

## 2. Data stores and entities

The six data stores below are the same as in the data-flow diagram.

### Device registry and settings (every device)

| Entity | Main fields | Key and relations |
|---|---|---|
| Device | Device id (SHA-256 of its certificate), certificate, label, kind, paired on, introduced by, state (paired or removed), removed on and by, last known addresses, last seen, capabilities, protocol version, free and total space, scan scope, last scan, last change number received, list updated on | Key: device id. One row for each known device, this one included. A removed device keeps its row as a permanent removal record |
| Feature switch | Device, feature, on or off | Key: device and feature. Features: list sharing, clipboard sending and receiving, previews to the AI device, search by meaning, deletion by other devices, automatic acceptance of files, screen viewing and control. All are off until the user turns them on |
| Setting | Key, value | The AI device, scanned and excluded folders, the smallest file checked for duplicates (64 KB), the clipboard pause |

### File lists and search index (every device)

| Entity | Main fields | Key and relations |
|---|---|---|
| File record | Device, path, name, cleaned name, size, modified date, type, perceptual hash (photos), partial hash and full SHA-256 (filled only when needed), change number, "deleted" mark and date, "cloud only" | Key: device and path. Many for each device |
| Search index | The cleaned name split into groups of three letters | One for each file record; built from the file records and can be rebuilt at any time |
| Search history | The last 10 searches | Only on the device where they were typed |

### Duplicate groups and delete log (every device)

| Entity | Main fields | Key and relations |
|---|---|---|
| Duplicate group | Group id, kind (exact, same photo, similar), on one device or several, space that can be freed, computed on, made by | Exact and same-photo groups are computed on each device; similar groups come from the AI device |
| Group member | Group, device, path, suggested to keep | Many for each group; points to a file record |
| Ignored group | The hashes of its members, ignored on, by | Shared with all devices, so that "Ignore" works everywhere |
| Delete request | Request id, file to delete, expected full hash, copy to keep, asked by, asked on, state | Points to two file records; waits while the owner device is off |
| Deletion log | What, when, asked by which device, which copy was kept | Kept for 30 days |

### Transfers (every device)

| Entity | Main fields | Key and relations |
|---|---|---|
| Transfer | Transfer id, other device, direction, file name, size, full hash, state (waiting, sending, paused, done, failed, cancelled), chunks received, temporary path, created, expires | Many for each device; a waiting transfer expires after 24 hours |

### Vault (encrypted, every device that uses Password Management)

| Entity | Main fields | Key and relations |
|---|---|---|
| Vault entry | Entry id, kind (login or secure note), name, website or app, user name, password, note, history of earlier versions, change number for each writing device, "deleted" mark (kept 90 days) | Stored inside the encrypted file. Next to it, outside the encrypted part: the vault key wrapped by the key made from the master password, the salt and settings of that key, and a second copy wrapped by the recovery code |

### Vectors (AI device only)

| Entity | Main fields | Key and relations |
|---|---|---|
| Vector | Device, path, kind (photo, video frame 1 to 3, name), the file's hash when made, 512 numbers for photos or 384 for names, model version, made on | Key: device, path and kind; points to a file record of any device |
| Preview queue | Device, path, since | Photos and videos waiting because their device is off |

## 3. Diagram

**Figure: the data model.** Source file: `diagrams/B5-data-model.drawio`.

![Data model: what each device stores](../diagrams/B5-data-model.png)

The main relations are these. One device has many file records. One file record has one search index entry, may belong to several duplicate groups through group members, and may have up to four vectors on the AI device (a photo, or three video frames, and its name). A delete request points to the file to delete and to the copy to keep. One device has many feature switches and transfers. Vectors point to file records by device and path, but they live in the AI device's database, so the link is checked whenever the lists change.

## 4. Sizes

| What | Size |
|---|---|
| File records | About 20 MB per 100,000 files (measured on sample names) |
| Search index | About 7 MB per 100,000 files (same measurement) |
| Lists on one device for 4 devices with 100,000 files each | About 110 MB |
| Photo vectors (AI device) | About 200 MB per 100,000 photos; half with 16-bit numbers |
| Name vectors (AI device) | About 1.5 KB per name, about 600 MB for 400,000 names (estimate) |
| AI models (AI device) | About 1.5 GB |
| Device registry, switches, settings, groups and logs | Under 1 MB for a normal home (estimate) |
| Vault | A few hundred KB for a few hundred entries (estimate) |

## 5. What happens to the data when...

- **A device is removed:** its row stays as a permanent removal record; its file records, group members, feature switches and waiting transfers are deleted on every device as soon as each device learns of the removal, and the AI device deletes its vectors. If it held the vault, the app offers "Change the vault key".
- **The AI device changes:** the old AI device deletes its vectors and preview queue; the new one builds them again from the file records it already holds.
- **A file changes:** its partial and full hash are cleared, its perceptual hash is made again, and its vectors are marked as old until a new preview arrives.
- **The master password changes:** only the wrapped copy of the vault key changes; the vault itself is not encrypted again.

## 6. Limitations

The sizes of the file records and the search index were measured once on sample names in a desktop version of SQLite; the other sizes are estimates. The extra time needed to encrypt the main database has not yet been verified and will be measured during implementation.

## 7. Summary

Each device holds one encrypted main database with four data stores, a separately encrypted vault when Password Management is used, and its keys in the operating system's key store. Only the AI device holds vectors. File lists from other devices are accepted only with the owner's valid signature and a higher change number, and data such as previews, clipboard items and screen images is never stored.

## Sources (opened 10 November 2026)

1. Device Discovery, Pairing and Trust design: entry fields of the trusted-device list (team document).
2. Password Management design: vault keys, vault entries, versions (team document).
3. Security and Privacy design: encrypted main database, signed file lists, data at rest (team document).
4. Technology choices, final version: SQLite, key stores (team document).
5. Duplicate File Detection design: hashes, groups, delete requests and log (team document).
6. Direct File Sharing design: transfers (team document).
7. Clipboard Synchronisation and Screen Sharing and Remote Control designs: data that is not stored (team documents).
