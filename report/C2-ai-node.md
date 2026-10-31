# C2: The AI device and the file lists

Owner: Mohammed. Status: second version, 31 Oct 2026. Answers open decisions 7 and 8 in `B1-architecture-v1.md`, and decision 14 (photo previews) in `B1-architecture-v2.md`. Uses Ali's technology choices in `B6-technology-v0.md`.
Phase 1 is design only: no code.

## What the AI assistant offers the user

- **Similar photos:** finds photos that look alike even when they are not exact copies (resized, cropped, edited, or several shots of the same moment), across all devices.
- **Find photos by words:** the user types "beach at sunset" or "عيد ميلاد" and gets matching photos, in Arabic or English.
- **Search files by meaning:** finds files whose names mean the same thing even with different words (for example "invoice" also finds "bill_march.pdf"). Searching the text inside documents is not in the first version (it would need file contents from other devices, like the photo previews below).
- **Model (Ali's pick in B6, section 4):** CLIP ViT-B/32 with its multilingual text encoder for photos, and multilingual-e5-small for file names. Both run on the normal CPU with ONNX Runtime inside the device app. There is no chat model.
- Exact copies (same SHA-256 hash) are found on every device without the AI (C4).

## Q1: How is the AI device chosen, and what happens when it is off or replaced?

- **Recommendation: the app suggests, the user decides.** Each device advertises its capabilities (rule 5 in B1). Only a laptop or PC that meets the minimum hardware below can host the AI role, never a phone or tablet. The app ranks these devices by memory, number of CPU cores, free disk, and whether it is usually on and plugged in. It suggests the best one, and the user confirms or picks another.
- **Minimum hardware:** I agree with Ali's proposal: a 64-bit laptop or PC (Windows 10, macOS 12, Linux), 4 CPU cores, 8 GB RAM, about 3 GB free disk, no graphics card needed. Reason: the two models take about 1.5 GB and the similarity data about 200 MB per 100,000 photos, which fits in 8 GB next to the user's other programs. I suggest 16 GB if the user works on that device at the same time, so the AI does not slow it down. None of these numbers are measured yet (not checked).
- **Why not fully automatic:** the strongest device is not always the one that stays on (a gaming PC may be off most of the day). The user knows this; the app does not.
- **Only one AI device at a time.** The choice is saved on every device, so all of them know where to send AI requests.
- **When the AI device is off:** hash duplicates, file sharing, clipboard and local search keep working. Similar-file detection and search by meaning show "AI paused: *Laptop-1* is off", with the last results and their date. If another capable device is online, the assistant offers "Move the AI role to *PC-2*?" (it asks, it does not switch by itself).
- **When the AI device is replaced:** the user picks the new device in settings. The new device rebuilds the similarity data from the file lists it already has (see Q2), and the old device deletes its copy. No similarity data is copied between devices. If the AI device is removed as lost (C1, section 1.7), the other devices show "Choose a new AI device".
- **Rebuild time for 100,000 files:** not checked. It waits for Ali's speed figure.

## Q2: Does every device keep a copy of the other devices' file lists?

- **Recommendation: yes, every device keeps a copy** (this matches the B1 assumption). It is the only way to see a switched-off device's files from anywhere, and it lets a new AI device start without waiting for the others to come online.
- **What is stored per file:** device name, path and file name, size, last modified date, file type, and a fingerprint (hash). No file contents, no thumbnails, and no AI similarity data (that stays on the AI device only).
- **How big it can get:** about 200 bytes per file, so roughly 20 MB for 100,000 files, maybe up to 40 MB with database overhead (my estimate, not checked). For four devices with 100,000 files each, a phone stores about 80 to 160 MB, which is acceptable.
- **How fresh it is:** when two devices connect, they exchange only the changes since the last sync, not the whole list. While both are online, changes are sent as they happen. Every list shows "last updated" with its date, so the user knows when it may be old.
- **Limit:** phones and tablets receive updates only while the app is open (B6, section 6), so their copies can be older than the laptop's.
- The full design of scanning and sharing the lists is in `C3-file-indexing.md`.

## Q3: How does the AI device see photos that live on other devices?

The AI device needs the pixels of a photo to understand it, but the file lists carry no thumbnails. My proposal (to agree with Ali, C4):

- **One small preview, sent on request.** The AI device asks the owner device for a preview of each new or changed photo. The owner makes a 224 × 224 JPEG (the input size of CLIP) and sends it over the secure channel. Size: about 10 to 20 KB each (not checked).
- **Not kept.** The AI device turns the preview into its similarity data and then deletes it. Only the similarity data stays, on the AI device only.
- **Only when the owner is online.** Photos on a switched-off device wait in a queue until it comes back. Phones answer only while the app is open, so their photos are processed in batches then.
- **The user controls it.** Each device has a switch "Allow the AI device to see photo previews", off until the user turns it on (as in B1 v2). If it is off, that device's photos are left out of similar-photo results and search by words, and the app says so.
- **Traffic:** 100,000 photos × about 15 KB is about 1.5 GB, once, over the local network (not checked). After that, only new photos.
- **Rejected alternative:** every device computes its own similarity data. This would put the 1.5 GB of models on phones and break the "one AI role" rule in B1.
- **Videos:** not in the first version (B6 says videos are not checked). Later: a few frames per video, sent the same way.

## Still open
- Rebuild time for 100,000 files: waits for Ali's speed figure (B6).
- The photo preview answer (Q3) still needs Ali's agreement (C4).
- Size estimate: to be checked with C3 and C5 now that the storage is SQLite (B6, section 5).
