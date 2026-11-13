# 3.x Requirements

This section states what the tool must do for the user (functional requirements) and the qualities it must have while doing it (non-functional requirements). Each requirement is one sentence that can be checked in a test. The requirements are grouped by feature, using the same feature names as the rest of the report. The complete list, with every detailed rule, measurement method and target, is given in Appendix A (Detailed requirements).

In this section, a *paired device* is a device that the user has joined to the tool by confirming a code on both devices, and the *AI device* is the one laptop or desktop computer that the user has chosen to run the AI analysis.

> **Note for the team (to be removed before submission):** requirements under a heading marked **[Scope pending]** belong to the five features whose place in the project scope is waiting for the supervisor's decision: Clipboard Synchronisation, Screen Sharing and Remote Control, Extra Screen, Direct File Sharing, and the offline-device records part of File Indexing and Offline-Device Records. Each can be deleted on its own; only three other sentences then lose a phrase: F6 and N5 ("including devices that are switched off", "and the stored lists of the other devices") if the offline-device records go, and N3 ("clipboard items or screen images") if clipboard, screen sharing and extra screen all go. Without them, the list has 10 functional and 8 non-functional requirements.

## 3.x.1 Functional requirements

### All features

**F1.** The tool must keep every feature switched off until the user turns it on, and must send a feature's data only to a paired device for which the user has switched that feature on.

### Device Discovery, Pairing and Trust

**F2.** The tool must let the user pair two devices on the same local network, either by scanning a QR code or by comparing a 6-digit code shown on both screens, and must add the new device only after the user confirms on both devices.

**F3.** The tool must let the user remove a lost or unwanted device from any paired device, after which no paired device accepts a connection from the removed device.

### File Indexing and Offline-Device Records

**F4.** The tool must keep a list of the files in the folders that the user allows on each device, holding each file's name, location, size, date and type but never its content, and must update the list when files are added, changed or deleted.

**F5. [Scope pending]** The tool must keep a copy of the file list of every paired device, so that the files of a device that is switched off can still be seen together with the date of that list, and must accept a list passed on by another device only if the owner device's signature on it is valid and its change number is higher than the one already held.

### Unified Search

**F6.** The tool must let the user search the file names of all paired devices from one search box, including devices that are switched off, and must find Arabic names whether they are written with or without vowel marks and with any form of alef, ya or ta marbuta.

**F7.** The tool must let the user search photos and documents by meaning on the AI device, and must show these results separately from the file-name results.

### Duplicate File Detection

**F8.** The tool must find exact copies of a file across all paired devices, and resized copies of a photo, and must show them in groups together with the storage space that can be freed.

**F9.** The tool must delete a duplicate file only on the device that holds it, only after the user has confirmed the deletion and the owner of that device has allowed deletions, either by turning on the allow option for the asking device or by accepting the request on their own device, and must never delete the last remaining copy of a file.

### AI Analysis (on the AI device)

**F10.** The tool must run the AI analysis on one laptop or desktop computer chosen by the user, never on a phone or tablet, and must keep every other feature working when that computer is switched off.

### Direct File Sharing [Scope pending]

**F11. [Scope pending]** The tool must let the user send one or more files directly to a paired device after the receiving user accepts, must show the progress on both devices, must continue an interrupted transfer from where it stopped, and must save the file only if it arrived complete and unchanged.

### Password Management

**F12.** The tool must keep the user's passwords in an encrypted vault whose random key is stored only after being wrapped by a key derived from the master password, must lock the vault after 5 minutes without use, and must share the vault only with the paired devices that the user has added to it.

### Clipboard Synchronisation [Scope pending]

**F13. [Scope pending]** The tool must make text copied on one device ready to paste on the user's other online paired devices, must offer a "Send clipboard" button on systems that do not allow automatic sending, and must never send an item that is marked as secret, such as a copied password.

### Screen Sharing and Remote Control [Scope pending]

**F14. [Scope pending]** The tool must let the user view the screen of another paired device, and control it only if this is allowed, after the user of that device accepts each request, and must show a bar with a Stop button on the shared device for as long as the session lasts.

### Extra Screen [Scope pending]

**F15. [Scope pending]** The tool must let the user use a paired tablet or phone as an extra screen for a computer, adding a true extra screen on Windows (with the tool's own signed display driver) and on Linux with Wayland (where the desktop offers a virtual monitor), mirroring the existing screen on macOS and wherever an extra screen cannot be added while telling the user so, and asking the user on both devices every time a session starts.

## 3.x.2 Non-functional requirements

### Security and Privacy

**N1.** The tool must work entirely on the user's local network, with no cloud service, server or user account, so that no user data leaves the network and every feature works when the internet is disconnected.

**N2.** The tool must encrypt every connection between devices with TLS 1.3, and each device must prove its identity with its own key before any data is exchanged.

**N3.** The tool must protect stored data by encrypting the main database with a random key kept in the system key store and the password vault with a random key wrapped by a key derived from the master password, and no device may store another device's file contents, clipboard items or screen images.

### Performance

**N4.** The tool must show file-name search results within 0.5 seconds of a key press for up to 400,000 file names on the slowest supported phone.

### Working without other devices

**N5.** Each device must keep working when it is the only device switched on, so that the user can still see and search its own files and the stored lists of the other devices, and only the AI features pause when the AI device is switched off.

### Supported systems

**N6.** The tool must run as one application on Windows 10 or later, macOS 12 or later, Ubuntu 20.04 or Debian 10 or later, Android 7.0 or later and iOS or iPadOS 15 or later, and must offer on each device only the actions that its system supports.

### Ease of use

**N7.** The tool must let a new user pair a device in no more than 5 taps and less than 60 seconds, and every waiting or limited state must show a message that says what is happening and what the user can do.

### Arabic and English

**N8.** The tool must offer its whole interface in Arabic, with a right-to-left layout, and in English.

## Summary

The tool has 15 functional and 8 non-functional requirements at this level. Together they describe a private tool that links the user's own devices on the local network, lets the user find, compare and share files across them, and protects the user's data without any cloud service. Appendix A gives each requirement in full detail.

### Link to Appendix A

| Short requirement | Detailed requirements in Appendix A |
|---|---|
| F1 | FR-1, FR-2 |
| F2 | FR-6 to FR-12 |
| F3 | FR-13, FR-17 |
| F4 | FR-23 to FR-26 |
| F5 | FR-4, FR-27 to FR-29 |
| F6 | FR-38 to FR-40 |
| F7 | FR-41, FR-42 |
| F8 | FR-31 to FR-34 |
| F9 | FR-5, FR-35 |
| F10 | FR-19, FR-20 |
| F11 | FR-44 to FR-48 |
| F12 | FR-52, FR-53, FR-55 |
| F13 | FR-56, FR-58, FR-59, FR-61 |
| F14 | FR-65, FR-68, FR-69 |
| F15 | FR-72, FR-78 |
| N1 | NFR-1, NFR-24 |
| N2 | NFR-4 |
| N3 | NFR-2, NFR-7 |
| N4 | NFR-10 |
| N5 | NFR-23, NFR-25 |
| N6 | NFR-30, NFR-32 |
| N7 | NFR-33, NFR-34 |
| N8 | NFR-36 |
