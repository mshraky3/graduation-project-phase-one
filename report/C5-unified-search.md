# Unified Search

## Introduction

When files are spread over a laptop, a phone and a tablet, the user often has to open each device to find one document or photo. Unified Search lets the user search once, from any device, and find a file on any of their devices, including a device that is switched off. The user can search by part of a file's name, with filters such as type and date, or describe what they are looking for in plain words, for example "beach at sunset". Each result shows which device holds the file, whether that device is on, and how old the information is, and offers the actions that are possible at that moment. This section explains the two kinds of search, what the user sees, and how the search behaves when the AI device is off.

## 1. Two kinds of search

| | **By name** | **By meaning** |
|---|---|---|
| Finds | File names (part of a word is enough), with filters: type, date, device and size | Photos from a description ("beach at sunset"), and file names with the same meaning ("invoice" also finds "bill_march.pdf") |
| Runs on | The device where the user types, using its own list and its copies of the other devices' lists | The AI device, using its similarity data |
| Needs another device | No | The AI device |
| How | The full-text search built into SQLite | The words are turned into a vector by the photo model or the text model and compared with the stored vectors |

The screen shows the two kinds in two sections, "Name matches" and "By meaning". Name matches appear at once and always work; meaning matches arrive a moment later and only when the AI device is on. Two sections keep the screen stable and make it clear which part is paused.

## 2. Search by name

- **Index:** each device builds one search index over the names in all the lists it holds, and updates it whenever list changes arrive.
- **Arabic names are cleaned first.** In a test of SQLite's full-text search, "الفاتورة" did not find a file named "الفَاتُورَة", because the vowel marks remained, and "فاتورة" did not find "الفاتورة", because the article "ال" is joined to the word. The app therefore stores a cleaned copy of each name: vowel marks and the elongation character are removed, the forms أ, إ and آ are written as ا, ى as ي, ة as ه, and all letters are made lower case. With the trigram method on the cleaned copy, "الفاتورة", "فاتورة" and "فاتوره" all found both test files, and "port" found "report_v2_final.docx".
- **Short queries:** the trigram method needs at least three letters; for one or two letters the app searches only the beginning of names.
- **Filters:** type (photo, video, document, other), date range, device, size, and "Only devices that are on".
- **Size:** the search index adds about 7 MB per 100,000 names.

## 3. Search by meaning

- **What is sent:** the searching device sends the AI device the words, the filters and a request number, over the paired, encrypted connection on the local network. Nothing goes to the internet.
- **What comes back:** at most 50 results for photos and 50 for names, each with only the device, the path and a score. The searching device adds the name, size, date, online or offline state and age from its own copies of the lists.
- **The search words are not kept.** The AI device uses the words and their vector in memory, forgets them once it has answered, and writes them to no file or log. The searching device keeps its last 10 searches only on itself, so the user can repeat them, and offers "Clear history".
- **What is searched:** only files that already have similarity data. Photos on a device whose "Allow the AI device to see photo previews" switch is off are not included, and the app says so: "Photos from iPad are not included: previews are off". New photos on a device that has been off are included after it returns and sends its previews.
- **Arabic and English:** the text model's quality in Arabic has been measured by its authors (Arabic Wikipedia search, nDCG@10 of 71.4). The photo model lists Arabic among its languages, but its quality in Arabic has not been measured. The app therefore does not promise that photo search in Arabic works as well as in English.

## 4. When the AI device is off

- Search by name works as usual.
- The "By meaning" section shows "Search by meaning paused: Laptop is off. Showing name matches only." If another capable device is on, the app adds "Move the AI role to Computer?"; it never switches by itself.
- A search is not kept to be answered later, because the user is looking now and a late answer would be surprising.

## 5. What the user sees for each result

- File name, type icon, folder, size and date modified.
- **Device:** its name and kind, and "Online" or "Offline". For a switched-off device, the age of its list: "List from 3 days ago"; for a list older than 7 days, "This list may be out of date".
- **Preview:** from the file itself if it is on this device; for a photo on another device that is on, one small preview requested from that device and not kept; none for a switched-off device.
- **Actions:**

| Action | File on this device | File on another device that is on | File on a device that is off |
|---|---|---|---|
| "Open" | Opens the file | The other device sends the file here first (Direct File Sharing), then it opens | Disabled: "Turn on Laptop to use this file" |
| "Send to..." | Direct File Sharing from this device | The device that holds the file sends it to the chosen device | Disabled |
| "Delete" | Duplicate File Detection rules: only the owner device deletes, after the user confirms and the checks pass | A request to the owner device, under the same rules | Disabled; the request waits |
| "Show duplicates" | Opens the duplicate group of this file, if there is one | The same | The same, from the last list |

## 6. iPhone, iPad and Android without "All files access"

- An iPhone or iPad holds copies of the other devices' lists like any device, so it can search every device's files by name, and by meaning while the AI device is on.
- Its own files in the results are only those the app may see: the app's own files, the photo library if the user allows it, and the folders the user picked. Its device row says so: "iPhone: photo library only".
- Its copies are updated only while the app is open, so they can be older than a laptop's; the age is always shown.
- "Open" for a file from another device saves it in the app's own storage, with "Save to Files" or "Save to Photos".
- An Android phone without "All files access" shows only its photos and videos as its own files, with the same kind of note.

## 7. Searching by meaning, step by step

**Figure: searching by meaning from the phone while the laptop is the AI device.** Source file: `diagrams/C5-search-sequence.drawio`.

![Unified Search: searching by meaning from the phone, the laptop is the AI device](../diagrams/C5-search-sequence.png)

The user types "beach at sunset" on the phone. The phone first searches the names in all the lists it holds and shows the name matches at once. It then checks that the AI device is on and paired. If the AI device is off, the user sees "Search by meaning paused" and the name matches only. Otherwise, the phone sends the words to the laptop over the paired connection. The laptop checks that the phone is paired and allowed to search by meaning, turns the words into a vector, compares it with the stored vectors and returns at most 50 file references with scores; it then forgets the words. The phone adds the names, devices and ages from its own lists and shows the "By meaning" section, where the user can open, send or delete a result.

## 8. Requirements

- FR-SRC-1 The app must search the names of the files of all paired devices, including switched-off devices, from the lists it holds, with no other device online.
- FR-SRC-2 The app must find a name from part of a word of three letters or more, ignoring upper and lower case, Arabic vowel marks and the different forms of alef, ya and ta marbuta.
- FR-SRC-3 The app must let the user filter results by type, date, device and size.
- FR-SRC-4 The app must send a search by meaning only to the AI device, only over the paired connection, and only if search by meaning is turned on for the searching device.
- FR-SRC-5 The AI device must return only file references and scores, and must not store the search words or their vector.
- FR-SRC-6 The app must show name matches and meaning matches in two separate sections.
- FR-SRC-7 The app must show for each result its device, whether that device is online, and for a switched-off device the date of its list.
- FR-SRC-8 The app must disable "Open", "Send to..." and "Delete" for a file on a switched-off device and name the device to turn on.
- FR-SRC-9 The app must show "Search by meaning paused" with the name of the AI device when the AI device is off, and keep showing name matches.
- FR-SRC-10 The app must say when a device's photos are left out because its preview switch is off.
- FR-SRC-11 The app must keep the search history only on the device where it was typed, and let the user clear it.
- NFR-SRC-1 Name matches must appear while the user types, within half a second for 400,000 names.
- NFR-SRC-2 No search and no result may leave the user's local network.

## 9. Limitations and open points

The following has not yet been verified: the speed of name search over 400,000 names on a phone; whether the SQLite version inside the app supports the trigram method (the cleaned names were tested only in a desktop version of SQLite); and the quality of photo search with Arabic words. Sending a file that is on another device to a third device ("Send to..." in section 5) needs a matching step in Direct File Sharing. The size of the name vectors on the AI device is estimated at about 1.5 KB per name, about 600 MB for 400,000 names. The search screen will follow the screen designs.

## 10. Summary

Unified Search gives the user one search box for all their devices. Search by name always works, even with every other device switched off, and handles Arabic names correctly. Search by meaning adds photos and files found from a description whenever the AI device is on, without keeping the search words. Every result shows where the file is, whether that device is on and how old the information is, so the user always knows what can be done with it.

## Sources (opened 10 November 2026)

1. Technology choices, final version: SQLite full-text search, models, Arabic quality, phones in the background (team document).
2. Duplicate File Detection design: previews, delete rules (team document).
3. Direct File Sharing design: sending and saving files (team document).
4. Existing systems study: none of the six applications studied offers one search across all devices (team document).
5. Test of SQLite 3.42 full-text search with the unicode61 and trigram methods on Arabic and English file names, 8 November 2026.
