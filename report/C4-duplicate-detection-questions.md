# C4: Duplicate-file detection, questions the design must answer

Owner: Ali. Status: questions list, 31 Oct 2026 (was due Thu 29 Oct). Answered in `C4-duplicate-detection.md`. Phase 1 is design only: no code. Names follow `B1-architecture-v2.md`; technology from `B6-technology-final.md`.

Starting points from others: every device keeps a copy of the other devices' file lists (path, name, size, date, type, hash; no contents) (C2); exact copies need no AI, similar files need the AI device (B1 rule 4); photo previews may go to the AI device, not kept, with a switch per device (approved in B1 v2's status line; decision 14 is still open on details); B1's safe assumption for decision 9, which C4 must turn into a decision: nothing is deleted without the user's confirmation, and only by the device that owns the file.

## A. Exact copies
1. What counts as an exact copy: same bytes, whatever the name, folder or date?
2. How are candidates found across four devices with about 100,000 files each without reading every byte of every file? Which device reads a file to hash it, and when (indexing, on demand, charging, phone app open)?
3. Which files are left out: empty or tiny files, system and app folders, unreadable files, files only in a cloud and not on the device (iCloud "Optimize Storage", OneDrive placeholders)?
4. Can copies on a switched-off device be found from its last known record?

## B. Similar photos and videos
5. What is "similar": the same photo resized or saved again, a crop or edit, a burst of nearly identical shots? Which method finds which kind, and which part works without the AI device?
6. How does the AI device see photos stored on other devices, and what does it keep afterwards?
7. When are two photos "similar enough", and how do we avoid false groups (black-and-white versions, plain backgrounds, screenshots of the same app)?
8. How are videos compared? What does the user see while the AI device is off?

## C. Showing the results
9. How are groups shown: exact versus similar; copies on one device versus copies spread over several devices?
10. What does each file show (device, folder, size, date, resolution, preview), including files on a device that is off?
11. What can the user do: keep one, choose what to delete, ignore a group, act on many groups at once? How is the space to be freed shown?

## D. Which copy is kept, and who deletes
12. Which copy does the app suggest keeping, and when should it suggest nothing (a copy on two devices may be the user's backup)?
13. Who deletes, and how: moved to the system's trash or deleted for good? Do Android and iOS show their own confirmation first?
14. What if the owning device is off, the file changed since the scan, or the copy to keep is gone? How is the last copy never deleted?
15. May one device ask another to delete while nobody is at the other device (a C11 permission)?

## E. Storage
16. What does C4 add to each device's index, and how big is it per 100,000 files?
17. Where are the groups, the "ignored" choices and the AI vectors kept; are they shared between devices?
18. When are results computed again, how is an old result marked, and what happens when a device is removed (C1) or the AI device changes (C2)?

## What C4 needs from others
Mohammed (C2, C3, B5): new index fields and free space per device. Mahmoud (C1): "trusted device" for delete requests; a removed device's records. Ali (C11): the permission "my other devices may delete duplicates here". D1, D2: the duplicates screen and clean-up journey. Samer (C5): may reuse the AI device's photo vectors.
