# Appendix A: Detailed requirements

This appendix gives the complete requirements of the tool. Chapter 3 states them in short form; here every rule is given in full, so that each one can be traced to the feature it comes from and checked in a test. There are 78 functional requirements (FR-1 to FR-78) and 38 non-functional requirements (NFR-1 to NFR-38), in one numbering. The column "Source feature" names the feature that the requirement comes from.

Every requirement reads as a sentence that begins with "The tool must ...". Unless another device is named, the subject is the tool on every device. A *paired device* is a device that the user has joined to the tool by confirming a code on both devices; the *AI device* is the one laptop or desktop computer that the user has chosen to run the AI analysis.

Numbers such as time limits, sizes and similarity thresholds are design targets. They have not yet been verified by measurement and will be tuned when the tool is built and tested.

> **Note for the team (to be removed before submission):** rows whose source feature is marked **[Scope pending]** belong to the five features whose place in the project scope is waiting for the supervisor's decision: Clipboard Synchronisation, Screen Sharing and Remote Control, Extra Screen, Direct File Sharing, and the offline-device records part of File Indexing and Offline-Device Records. Each of these rows can be deleted on its own. The rows listed under "Rows that depend partly on a scope-pending feature" at the end of the appendix need one phrase removed instead.

## A.1 Functional requirements

### All features

| No. | The tool must ... | Source feature |
|---|---|---|
| FR-1 | send a feature's data to a device only if that device is a paired device and the feature is switched on for it, and check this again on the receiving device | All features |
| FR-2 | keep every feature switched off on each device until the user turns it on | All features |
| FR-3 | let paired devices tell each other, after they have proved their identity, what they can do (run the AI analysis, be viewed, be controlled, act as an extra screen, synchronise the clipboard in the background), and offer an action only when both devices support it | All features |
| FR-4 | show every device as online or offline and, for an offline device, the date of its most recent data; disable the actions that need the offline device and name the device that must be switched on | File Indexing and Offline-Device Records [Scope pending] |
| FR-5 | ask for the system's fingerprint, face or PIN check before the user changes the permission to delete files, the password vault settings or the permission to control the device | Security and Privacy |

### Device Discovery, Pairing and Trust

| No. | The tool must ... | Source feature |
|---|---|---|
| FR-6 | create a key pair and a self-signed certificate when it starts for the first time, never replace them, and use the SHA-256 fingerprint of the certificate as the device's identity | Device Discovery, Pairing and Trust |
| FR-7 | keep the device's private key in the system key store and out of cloud backups | Device Discovery, Pairing and Trust |
| FR-8 | announce on the network only its protocol version and an 8-byte prefix of its identity, never a device name, a file name or a capability | Device Discovery, Pairing and Trust |
| FR-9 | open the pairing service and its port only while a pairing window is open, and close it after 120 seconds, on success, on cancel, or after 3 failed confirmations | Device Discovery, Pairing and Trust |
| FR-10 | offer pairing by scanning a QR code and by comparing a 6-digit code; in the QR route, stop if the certificate does not match the QR code, and accept the one-time secret only once and only inside the pairing window | Device Discovery, Pairing and Trust |
| FR-11 | add a new device only after the users of both devices have tapped "Match"; in the code route, send the commitment before the random value it hides | Device Discovery, Pairing and Trust |
| FR-12 | show a device's name as "unverified" until the device is paired, and remove control characters and right-to-left marks from the name | Device Discovery, Pairing and Trust |
| FR-13 | refuse every connection from a certificate that is not paired (except to the pairing port while a pairing window is open), from a known identity that presents a different certificate (and show a notice), and from every removed device | Device Discovery, Pairing and Trust |
| FR-14 | check, on every connection, the other device's certificate fingerprint and a fresh signed challenge, and refuse TLS versions below 1.3 and protocol versions lower than the one stored | Device Discovery, Pairing and Trust |
| FR-15 | let the user type a device's address by hand, and show a clear message when the Wi-Fi network blocks devices from seeing each other or when the iPhone or iPad "Local Network" permission is off | Device Discovery, Pairing and Trust |
| FR-16 | pair each new device with every other device; if an optional "Introduce my new device" step is offered, it must show a prompt and a 6-digit code on every target device and store nothing until both users tap "Match" | Device Discovery, Pairing and Trust |
| FR-17 | let the user remove a lost device from any paired device, pass the removal at once to the devices that are online and to the others when they next connect, tell the user what the removed device still holds, and offer to change the vault key | Device Discovery, Pairing and Trust |
| FR-18 | mark a device as offline after 3 missed heartbeats sent every 10 seconds | Device Discovery, Pairing and Trust |

### AI Analysis (on the AI device)

| No. | The tool must ... | Source feature |
|---|---|---|
| FR-19 | suggest one AI device among the laptops and desktop computers that meet the minimum hardware, ranked by memory, processor cores, free disk space and whether the device is usually switched on and plugged in; let the user confirm it or pick another; never choose a phone or tablet; and save the choice on every device | AI Analysis (on the AI device) |
| FR-20 | keep every feature that does not need the AI working when the AI device is off, show "AI paused: *device name* is off" with the last results and their date, and offer (not force) moving the AI role to another suitable device that is online | AI Analysis (on the AI device) |
| FR-21 | rebuild the similarity data on the new AI device from the file lists it holds when the AI role moves, and delete it on the old device; similarity data is never copied | AI Analysis (on the AI device) |
| FR-22 | ask the device that owns a photo for a 224 × 224 preview (3 frames for a video) only while that device is online and only if "Allow the AI device to see photo previews" is on for it; delete each preview after use; and tell the user when a device's photos are left out | AI Analysis (on the AI device) |

### File Indexing and Offline-Device Records

| No. | The tool must ... | Source feature |
|---|---|---|
| FR-23 | list the files in the folders that the user allows, and show which folders are scanned on each device | File Indexing and Offline-Device Records |
| FR-24 | store for each file its location, name, size, date of change, type, change number and a "deleted" mark, and never its content | File Indexing and Offline-Device Records |
| FR-25 | make a perceptual fingerprint of each photo, and work out the partial and full file fingerprints only when another feature needs them | File Indexing and Offline-Device Records |
| FR-26 | notice new, changed and deleted files (on computers through system notifications plus a check every 6 hours; on phones and tablets each time the tool starts), and read a file again only when its size or date has changed | File Indexing and Offline-Device Records |
| FR-27 | keep a read-only copy of the file list of every device that shares its list, receive only the changes made after the last change number it holds, and send a full list only to a device that has none or has been away for more than 30 days | File Indexing and Offline-Device Records [Scope pending] |
| FR-28 | accept a file list passed on by a third device only if the owner device's signature on it is valid and its change number is higher than the one already held | File Indexing and Offline-Device Records [Scope pending] |
| FR-29 | delete the file list of a removed device as soon as it learns of the removal; the AI device must also delete that device's similarity data | File Indexing and Offline-Device Records [Scope pending] |
| FR-30 | store the free and total storage space of each device, updated at every scan and every connection | File Indexing and Offline-Device Records |

### Duplicate File Detection

| No. | The tool must ... | Source feature |
|---|---|---|
| FR-31 | treat files with the same size and the same SHA-256 fingerprint as exact copies, checked in three steps (size, partial fingerprint, full fingerprint), with only the device that holds a file reading it | Duplicate File Detection |
| FR-32 | find resized copies of a photo with its perceptual fingerprint (a difference of 5 or less), without the AI device | Duplicate File Detection |
| FR-33 | find similar photos on the AI device (a similarity of 0.90 or more), and similar videos from 3 frames of each video | Duplicate File Detection |
| FR-34 | show duplicate groups as "exact", "same photo" or "similar", on one or several devices, with the storage space that can be freed; and show a match that still needs a missing fingerprint as "possible" | Duplicate File Detection |
| FR-35 | delete a file only on the device that holds it, and only after the user has confirmed, the owner of that device has allowed deletions from the asking device, the file has been fingerprinted again and the kept copy has been found to still exist; never delete the last copy; and record every deletion in a log kept for 30 days | Duplicate File Detection |
| FR-36 | move deleted files to the system's trash or recycle bin where the system has one | Duplicate File Detection |
| FR-37 | share the list of ignored groups between devices, so that a group the user has ignored does not appear again on any device | Duplicate File Detection |

### Unified Search

| No. | The tool must ... | Source feature |
|---|---|---|
| FR-38 | search the file names of all paired devices, including devices that are switched off, from the file lists it holds, with no other device online | Unified Search |
| FR-39 | find a name from part of a word of 3 letters or more, ignoring letter case, Arabic vowel marks and the different forms of alef, ya and ta marbuta | Unified Search |
| FR-40 | let the user filter results by file type, date, device and size | Unified Search |
| FR-41 | send a search by meaning only to the AI device; the AI device must return only references to files and their scores, and must not store the query or its vector | Unified Search |
| FR-42 | show file-name results and meaning results in two separate sections | Unified Search |
| FR-43 | keep the search history only on the device where the search was typed, and let the user clear it | Unified Search |

### Direct File Sharing

| No. | The tool must ... | Source feature |
|---|---|---|
| FR-44 | let the user choose one or more files and one paired device to send them to | Direct File Sharing [Scope pending] |
| FR-45 | ask the receiving user to accept or decline, unless "Accept automatically from my own devices" is on (it is off by default) | Direct File Sharing [Scope pending] |
| FR-46 | show the progress on both devices and let either user cancel at any time | Direct File Sharing [Scope pending] |
| FR-47 | send files in parts over a separate file connection, keep the parts already received when the connection drops, continue by sending only the missing parts, and show "Paused" and then "Failed – tap to retry" | Direct File Sharing [Scope pending] |
| FR-48 | check the fingerprint of each finished file and not save a file whose fingerprint does not match | Direct File Sharing [Scope pending] |
| FR-49 | keep a transfer to an offline device waiting for up to 24 hours, and delete the temporary files of cancelled or expired transfers | Direct File Sharing [Scope pending] |
| FR-50 | never overwrite a file (save it under a new name instead), and tell the user when the receiving device already has an identical file | Direct File Sharing [Scope pending] |
| FR-51 | save received files where each system allows, and tell the user where they were saved | Direct File Sharing [Scope pending] |

### Password Management

| No. | The tool must ... | Source feature |
|---|---|---|
| FR-52 | keep the password vault in a separate encrypted SQLCipher file whose random key is stored only wrapped by a key derived from the master password with Argon2id, or behind the fingerprint or face check in the system key store; and never store or send the master password, the wrapping key or the unwrapped vault key | Password Management |
| FR-53 | lock the vault when the tool closes, when the screen locks and after 5 minutes without use; allow the fingerprint or face check only through the system key store; and ask for the master password after the device restarts | Password Management |
| FR-54 | import saved logins from a CSV file and offer to delete the file afterwards | Password Management |
| FR-55 | synchronise the vault only with paired devices that the user has added to it, encrypted with the vault key, and keep both versions when the same entry was changed on two devices | Password Management |
| FR-56 | mark copied passwords as secret, never send them to other devices through clipboard synchronisation, and clear them from the clipboard after 30 seconds where the system allows | Password Management |
| FR-57 | show the recovery code once, when the vault is set up, and tell the user that a forgotten master password cannot be reset | Password Management |

### Clipboard Synchronisation

| No. | The tool must ... | Source feature |
|---|---|---|
| FR-58 | send copied text of up to 1 MB automatically to the paired devices that are online and have receiving switched on, where the system allows this; and send an image (PNG or JPEG, up to 10 MB) only when the user taps "Send to other devices" | Clipboard Synchronisation [Scope pending] |
| FR-59 | offer a "Send clipboard" button on every device; on Android, iPhone, iPad and on Linux with GNOME on Wayland, this button is the way to send | Clipboard Synchronisation [Scope pending] |
| FR-60 | refuse items above the size limit with the message "Too large to sync. Use file sharing" | Clipboard Synchronisation [Scope pending] |
| FR-61 | never send an item that another application has marked as secret | Clipboard Synchronisation [Scope pending] |
| FR-62 | place a received item on the clipboard ready to paste, not send it on to other devices, and on iPhone and iPad write it as local to that device only | Clipboard Synchronisation [Scope pending] |
| FR-63 | not store or queue clipboard items, and show "No device online to receive it" when no device could receive an item | Clipboard Synchronisation [Scope pending] |
| FR-64 | let the user pause synchronisation for 1 hour or until it is turned on again, and switch sending or receiving off for each device | Clipboard Synchronisation [Scope pending] |

### Screen Sharing and Remote Control

| No. | The tool must ... | Source feature |
|---|---|---|
| FR-65 | start a session only after the user of the shared device accepts that request, ask for viewing and for control separately ("Allow view", "Allow view and control", "Deny"), and not offer control of a device that cannot be controlled (iPhone and iPad; Android without the accessibility service); unattended access is not allowed | Screen Sharing and Remote Control [Scope pending] |
| FR-66 | let the user of the shared device choose the whole screen or one window or app through the system's own picker where there is one (Windows, macOS 14 and later, Android 14 and later, Linux on Wayland, iOS 27 and later), and share only the picture, never the sound, in the first release | Screen Sharing and Remote Control [Scope pending] |
| FR-67 | send the picture with WebRTC set up only over the encrypted message connection between the paired devices (local network only, with no outside STUN or TURN server), end the session if the DTLS fingerprint differs from the one received over that connection, and keep the request, the answer, "Stop" and the set-up messages on the message connection, with the picture on its own connection | Screen Sharing and Remote Control [Scope pending] |
| FR-68 | accept mouse and keyboard events only from the viewing device of the current session, and only if control was allowed for that session | Screen Sharing and Remote Control [Scope pending] |
| FR-69 | show on the shared device a bar that stays on top, reading "Being viewed by …" or "Controlled by …", with a Stop button; end screen capture within 1 second of Stop on either device; and keep the local mouse and keyboard working | Screen Sharing and Remote Control [Scope pending] |
| FR-70 | stop the session when either device is switched off, sleeps, locks or leaves the network, show "Connection lost", ask for a new approval after 30 seconds, and show "Cannot start the stream on this network" when the devices cannot reach each other directly | Screen Sharing and Remote Control [Scope pending] |
| FR-71 | show on Android, before the user turns on the accessibility service for control, the in-app explanation that Google Play requires | Screen Sharing and Remote Control [Scope pending] |

### Extra Screen

| No. | The tool must ... | Source feature |
|---|---|---|
| FR-72 | add a true extra screen at the screen device's resolution on Windows (with the tool's own signed display driver) and on Linux with Wayland (where the desktop offers a virtual monitor), and mirror the existing screen on macOS and wherever an extra screen cannot be added, telling the user so | Extra Screen [Scope pending] |
| FR-73 | turn touches on the screen device into mouse events on the extra screen only | Extra Screen [Scope pending] |
| FR-74 | remove the extra screen when the session ends for any reason | Extra Screen [Scope pending] |
| FR-75 | keep the screen device awake while it shows the extra screen, and stop the session when the tool leaves the foreground on an iPhone or iPad | Extra Screen [Scope pending] |
| FR-76 | let the user stop the extra screen from either device at any time | Extra Screen [Scope pending] |
| FR-77 | ask for the system's permission where the system requires it (the Wayland screen portal, Screen & System Audio Recording on macOS), and use no private system interface in the first release | Extra Screen [Scope pending] |
| FR-78 | ask the user on both devices, every time, before an extra screen session starts, both for an extra screen and for a mirrored screen | Extra Screen [Scope pending] |

## A.2 Non-functional requirements

Each row gives what is measured and the target. The column "Basis" says where the target comes from: **Design** (follows from the design itself), **Design target** (chosen in the design, to be confirmed by measurement), **Estimate** (worked out, not run), **Published figure** (taken from a published source), **Measured** (measured by the team), or **Open** (no target has been set yet).

### Privacy

| No. | Requirement | How it is measured | Target | Basis | Source feature |
|---|---|---|---|---|---|
| NFR-1 | No data leaves the local network: no cloud service, no server, no user account and no usage data | Run every feature with the router's internet cable unplugged and record the network traffic; count the packets sent to addresses outside the local network | 0 packets; every feature works | Design target | All features |
| NFR-2 | No file content leaves its device except the photo previews for the AI device, and no device stores another device's previews, clipboard items or screen images | Inspect the tool's storage and temporary folders on every device after a test session | Nothing found | Design target | All features |
| NFR-3 | Search queries stay on the device where they were typed; the AI device keeps no query text or vector | Inspect the AI device's storage after 100 searches | Nothing found | Design target | Unified Search |

### Security

| No. | Requirement | How it is measured | Target | Basis | Source feature |
|---|---|---|---|---|---|
| NFR-4 | Every connection uses TLS 1.3 with the other device's certificate pinned, and the connecting device proves its key with a fresh signed challenge | Connect with a TLS 1.2 client; replay a recorded identity check | Both refused | Design | Security and Privacy |
| NFR-5 | An attacker placed between two devices cannot make both screens show the same pairing code | Test relay between two devices, 1,000 runs | The codes differ in every run; the chance of success is at most 1 in 1,000,000 per attempt (6 digits) | Design | Device Discovery, Pairing and Trust |
| NFR-6 | A device that has not proved its identity is limited | Flood test | At most 16 connections, each closed after at most 5 seconds, with a first message of at most 2 KB | Design target | Device Discovery, Pairing and Trust |
| NFR-7 | Stored data is protected: the private key and the list of paired devices cannot be read by other applications; the main database is encrypted with a random key kept in the system key store; the vault is encrypted with SQLCipher using a random key wrapped by a key derived from the master password with Argon2id (64 MiB, 3 passes, 1 thread) | Read the files from another application and another user account; open a copied database | Unreadable | Design | Security and Privacy; Password Management |
| NFR-8 | The Windows limit is stated honestly: the Windows data protection service protects the key from other user accounts, not from other programs of the same user | Not measured | Stated in the report | Published figure | Security and Privacy |
| NFR-9 | A removed device is shut out quickly | Three devices, one of them offline; time the removal | Online devices within 5 seconds; the others at their next connection | Design target | Device Discovery, Pairing and Trust |

### Performance

| No. | Requirement | How it is measured | Target | Basis | Source feature |
|---|---|---|---|---|---|
| NFR-10 | File-name search answers while the user types | 400,000 names (4 devices × 100,000) on the slowest supported phone; time from key press to results | At most 0.5 seconds | Design target | Unified Search |
| NFR-11 | First AI analysis of 100,000 photos | Model time on a laptop processor with 4 cores and 8 threads, using ONNX Runtime | About 2 hours (previews and videos not included) | Estimate from a published figure (14.8 previews per second) | AI Analysis (on the AI device) |
| NFR-12 | Scanning 100,000 files | Time the first scan on a phone and on a laptop | Set after the first measurement | Open | File Indexing and Offline-Device Records |
| NFR-13 | Reconnecting to a paired device | Same Wi-Fi network, no user action; time until "connected" | Less than 3 seconds | Design target | Device Discovery, Pairing and Trust |
| NFR-14 | Noticing that a device is offline | Disconnect a device and time it | About 40 seconds at most (3 missed heartbeats sent every 10 seconds) | Design | Device Discovery, Pairing and Trust |
| NFR-15 | A file transfer continues after the connection drops | Cut the Wi-Fi during a 2 GB transfer; count the bytes sent again | Only the missing parts are sent again; transfer speed set after the first measurement | Design target | Direct File Sharing [Scope pending] |
| NFR-16 | Clipboard delay | Copy on one device; time until the item is ready to paste on the other | 1 second at most (suggested) | Open | Clipboard Synchronisation [Scope pending] |
| NFR-17 | Screen sharing | Same Wi-Fi network | Picture within 5 seconds of approval; delay under 200 ms; up to 1920 × 1080 at 30 frames per second; Stop within 1 second | Design target | Screen Sharing and Remote Control [Scope pending] |
| NFR-18 | Extra screen | Same Wi-Fi network; time from drawing on the computer to showing on the screen device | Under 100 ms, at the screen device's resolution | Design target | Extra Screen [Scope pending] |
| NFR-19 | Unlocking the vault | Master password on the slowest supported phone | About 1 second at most | Design target | Password Management |
| NFR-20 | Storage used | Size of the tool's files on disk | File lists at most 30 MB per 100,000 files, search index included (27 MB measured on made-up names); a phone holding 4 lists of 100,000 files about 110 MB; photo vectors about 200 MB per 100,000 photos (AI device only); name vectors about 600 MB per 400,000 names | Measured (file lists); Estimate (the rest) | File Indexing and Offline-Device Records; AI Analysis (on the AI device); Unified Search |
| NFR-21 | Network use for the AI analysis | Bytes sent for photo previews | About 1.5 GB once per 100,000 photos (about 15 KB each), then only new photos | Estimate | AI Analysis (on the AI device) |
| NFR-22 | Minimum hardware for the AI device | Not measured | 64-bit laptop or desktop computer, 4 cores, 8 GB of memory (16 GB suggested), about 3 GB of free disk space, no graphics card needed | Design | AI Analysis (on the AI device) |

### Working without the internet and with devices switched off

| No. | Requirement | How it is measured | Target | Basis | Source feature |
|---|---|---|---|---|---|
| NFR-23 | Every device works alone: its own files, its own index, search of the stored file lists and exact-copy detection work with no other device online | Only one device switched on, Wi-Fi off | All of these work; "possible" is shown where a fingerprint is missing | Design | All features |
| NFR-24 | Pairing, device removal and live picture streams work with no internet connection (no outside STUN or TURN server) | A router with no internet connection; airplane mode with only Wi-Fi on | All of them work | Design (pairing); Design target (live picture streams) | Device Discovery, Pairing and Trust; Screen Sharing and Remote Control [Scope pending] |
| NFR-25 | When the AI device is switched off, only the AI features pause | Switch the AI device off | The other features are unchanged; a message is shown | Design | AI Analysis (on the AI device) |

### Battery and device load

| No. | Requirement | How it is measured | Target | Basis | Source feature |
|---|---|---|---|---|---|
| NFR-26 | The first scan runs in the background at low priority and can be paused | Watch processor use during the scan while using the device | Can be paused; processor limit set after the first measurement | Design | File Indexing and Offline-Device Records |
| NFR-27 | Files are read in full only when needed (full fingerprints are made only for likely duplicates) | Count the bytes read during a duplicate check | Far below the total size of the files (reading 500 GB would take hours) | Design | Duplicate File Detection |
| NFR-28 | The Android "stay connected" mode | Battery use per hour with the mode on and off, on the same phone | Set after the first measurement | Open | All features |
| NFR-29 | The AI device does not slow the user's work | Processor and memory use while the AI analysis runs | Set after the first measurement; 16 GB of memory suggested if the user works on that device | Open | AI Analysis (on the AI device) |

### Supported systems

| No. | Requirement | How it is measured | Target | Basis | Source feature |
|---|---|---|---|---|---|
| NFR-30 | One Flutter application runs on Windows 10 and later, macOS 12 and later, Ubuntu 20.04 or Debian 10 and later, Android 7.0 and later, and iOS or iPadOS 15 and later | Install and run on Windows, Android and Ubuntu 24.04 (from a USB stick or a second partition); macOS, iPhone and iPad are checked by review of the documentation and the design, because they can be built and tested only on a Mac with Xcode | Runs on the systems that can be tested; the others are confirmed by review | Design | All features |
| NFR-31 | Pairing works across systems | Test matrix of the pairs that can be tested; pairs that involve Apple devices are checked by review of the documentation and the design | Android with Windows, Android with Ubuntu 24.04, and two Windows computers, one of them without a camera | Design target | Device Discovery, Pairing and Trust |
| NFR-32 | The limits of each system are shown to the user, not hidden: each device offers only what it can do | Check each device (for example, an iPhone offers no control and no AI role; macOS offers the extra screen as a mirrored screen) | No action is offered that would fail | Design | All features |

### Ease of use

| No. | Requirement | How it is measured | Target | Basis | Source feature |
|---|---|---|---|---|---|
| NFR-33 | The first pairing is quick | Test with 5 volunteers | At most 5 taps and less than 60 seconds | Design target | Device Discovery, Pairing and Trust |
| NFR-34 | Every waiting or limited state says what is happening and what to do ("Paused", "Failed – tap to retry", "AI paused", "List from 3 days ago", "Too large to sync", "Turn on *Laptop-1*") | Review every screen against this list | Every state has a message | Design | All features |
| NFR-35 | The main tasks can be done without help | 5 volunteers, 5 tasks (pair a device, find a file, send it, remove duplicates, share the screen) | 4 of the 5 volunteers finish each task without help (suggested) | Open | All features |

### Arabic and English

| No. | Requirement | How it is measured | Target | Basis | Source feature |
|---|---|---|---|---|---|
| NFR-36 | The whole interface is available in Arabic (right to left) and in English | Review every screen in both languages | All screens, with a mirrored layout in Arabic | Open | All features |
| NFR-37 | Arabic file names are found as described in FR-39 (vowel marks and the forms of alef, ya and ta marbuta) | A test list of Arabic names and queries | Every test pair is found | Measured (in SQLite 3.42 through Python only) | Unified Search |
| NFR-38 | Search by meaning works in Arabic | Text model: published score; photo model: the team's own set of Arabic and English queries | Text: nDCG@10 of 71.4 (published, on the Arabic part of MIRACL); photos: set after the first measurement | Published figure (text); Open (photos) | Unified Search; AI Analysis (on the AI device) |

## A.3 Rows that depend partly on a scope-pending feature

If a scope-pending feature is removed, these rows stay but lose one phrase:

| Row | Phrase to remove | If this feature is removed |
|---|---|---|
| FR-3 | "be viewed, be controlled" | Screen Sharing and Remote Control |
| FR-3 | "act as an extra screen" | Extra Screen |
| FR-3 | "synchronise the clipboard in the background" | Clipboard Synchronisation |
| FR-5 | "or the permission to control the device" | Screen Sharing and Remote Control |
| FR-38 | "including devices that are switched off, from the file lists it holds, with no other device online" | Offline-device records |
| FR-56 | "never send them to other devices through clipboard synchronisation," | Clipboard Synchronisation |
| NFR-2 | "clipboard items or screen images" | Clipboard Synchronisation; Screen Sharing and Remote Control; Extra Screen |
| NFR-23 | "search of the stored file lists" | Offline-device records |
| NFR-24 | "and live picture streams" | Screen Sharing and Remote Control |
| NFR-32 | "no control"; "macOS offers the extra screen as a mirrored screen" | Screen Sharing and Remote Control; Extra Screen |
| NFR-34 | "Paused", "Failed – tap to retry", "Too large to sync" | Direct File Sharing; Clipboard Synchronisation |
| NFR-35 | "send it", "share the screen" | Direct File Sharing; Screen Sharing and Remote Control |
