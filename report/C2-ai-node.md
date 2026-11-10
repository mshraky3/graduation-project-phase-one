# The AI Device

## Introduction

Some of the assistant's most useful work needs a small artificial-intelligence model: recognising photos that look alike, and finding a photo or a file from a description in plain words. These models need more memory and processing power than a phone can spare. The system therefore gives this role to one laptop or computer in the user's network, called the AI device. All other features, such as finding exact copies, sending files and syncing the clipboard, work on every device without it. This section explains what the AI device offers the user, how it is chosen, what happens when it is switched off or replaced, and how it sees photos that are stored on other devices.

## 1. What the user gains

- **Similar photos:** the AI device finds photos that look alike even when they are not exact copies: resized, cropped or edited photos, or several shots of the same moment, across all devices (AI Analysis, used by Duplicate File Detection).
- **Finding photos by words:** the user types "beach at sunset" or the same words in Arabic, and sees matching photos (used by Unified Search).
- **Finding files by meaning:** the user finds files whose names mean the same thing in other words, so that "invoice" also finds "bill_march.pdf". Searching the text inside documents is not included, because it would need the contents of files from other devices.
- **Models:** a multilingual CLIP ViT-B/32 model for photos and the multilingual-e5-small model for file names. Both run on the device's normal processor inside the app. There is no chat assistant.
- Exact copies of a file are found on every device without the AI device (Duplicate File Detection).

## 2. How the AI device is chosen

- **The app suggests, the user decides.** Each device tells the others what it can do. Only a laptop or computer that meets the minimum hardware below can take the AI role, never a phone or a tablet. The app ranks these devices by memory, number of processor cores, free disk space, and whether the device is usually on and plugged in. It suggests the best one, and the user confirms it or picks another: "Laptop can run the AI assistant (similar photos, search by meaning). Use it?"
- **Why the user decides:** the strongest device is not always the one that stays on. A gaming computer, for example, may be off most of the day; the user knows this, the app does not.
- **Minimum hardware:** a 64-bit laptop or computer (Windows 10, macOS 12 or Linux), 4 processor cores, 8 GB of memory, about 3 GB of free disk space, and no graphics card. The two models take about 1.5 GB and the similarity data about 200 MB per 100,000 photos, which fits in 8 GB next to the user's other programs. 16 GB is recommended when the user works on the same device at the same time, so that the assistant does not slow it down.
- **One AI device at a time.** The choice is saved on every device, so all of them know where to send requests.

## 3. When the AI device is switched off or replaced

- **Switched off:** exact-copy detection, file sharing, clipboard sync and search by name keep working. Similar-photo detection and search by meaning show "AI paused: Laptop is off", with the last results and their date. If another capable device is on, the app asks "Move the AI role to Computer?"; it never switches by itself.
- **Replaced:** the user picks the new device in the settings. The new device builds the similarity data again from the file lists it already holds, and the old device deletes its copy; similarity data is never copied between devices. If the AI device is removed as lost, the other devices show "Choose a new AI device".
- **Time to build the similarity data:** about 2 hours per 100,000 photos on a 4-core, 8-thread laptop processor. This is an estimate from a published measurement of the same image model (51,126 previews in 57 minutes 39 seconds, about 14.8 per second, without a graphics card). It does not include making and sending the previews, or videos, and a slower processor takes longer. After the first build, only new photos are processed.

## 4. Where the file lists are kept

- **Every device keeps a copy of the other devices' lists.** This is the only way to see a switched-off device's files from any device, and it lets a new AI device start without waiting for the others to come online.
- **What is stored for each file:** the device, path and name, size, last modified date and type, and three fingerprints: a perceptual hash for photos, made by the owner device while it indexes, and a partial hash and a full SHA-256, which are filled only when the file matches another file in size. File contents, pictures and similarity data are not stored in the lists.
- **Size:** about 20 MB per 100,000 files, plus about 7 MB for the search index. A phone that holds the lists of four devices with 100,000 files each stores about 110 MB.
- **Freshness:** when two devices connect, they exchange only the changes since the last update, and while both are on, changes are sent as they happen. Each list shows the date of its last update. Phones and tablets receive updates only while the app is open, so their copies can be older than a laptop's.
- File Indexing and Offline-Device Records gives the full design, including the rule that a list is accepted only with the owner's valid signature and a higher change number.

## 5. How the AI device sees photos on other devices

The AI device needs the pixels of a photo to understand it, but the file lists carry no pictures.

- **One small preview, sent on request.** The AI device asks the owner device for a preview of each new or changed photo. The owner makes a 224 × 224 pixel JPEG: the short side is resized to 224 pixels and the centre is cut out, which is exactly the model's own input, so a larger preview would only be shrunk again. Each preview is about 15 KB.
- **Not kept.** The AI device turns the preview into similarity data and then deletes it. Only the similarity data stays, and only on the AI device.
- **Only while the owner device is on.** Photos on a switched-off device wait in a queue until it returns. Phones answer only while the app is open, so their photos are processed in batches at those times.
- **The user is in control.** Each device has the switch "Allow the AI device to see photo previews", which is off until the user turns it on. When it is off, that device's photos are left out of similar-photo results and photo search, and the app says so.
- **Network traffic:** about 1.5 GB once for 100,000 photos, over the local network only; after that, only new photos.
- **Videos:** the owner sends three frames per video, taken at 10 %, 50 % and 90 % of its length, as 224 × 224 previews in the same way. If taking frames is not possible on a phone, that phone offers exact video copies only.
- **Alternative that was rejected:** letting every device compute its own similarity data would place 1.5 GB of models on phones and break the rule of one AI device.

## 6. Limitations

The build time is an estimate from a published measurement and has not yet been verified with the runtime the app will use. Taking video frames on each system, and the quality of photo search with Arabic words, have not yet been verified either; the text model's Arabic quality has been measured by its authors, the photo model's has not. These points will be tested during implementation.

## 7. Summary

One laptop or computer, suggested by the app and confirmed by the user, runs the AI models. It finds similar photos and answers searches by meaning, using small previews that it deletes after use and that are sent only when the user allows it. When it is off, only these two abilities pause; everything else keeps working, and the user is offered another capable device.

## Sources (opened 10 November 2026)

1. Technology choices, final version: AI model, minimum hardware, build-time measurement, Arabic quality (team document).
2. Duplicate File Detection design: hashes, preview size, video frames (team document).
3. Security and Privacy design: photo previews, permissions (team document).
4. Device Discovery, Pairing and Trust design: removing a lost device (team document).
5. Mustafa.net, "Immich without a GPU", measurement of 10 October 2026: https://mustafa.net/2026/10/10/immich-without-gpu-intel-nuc/
