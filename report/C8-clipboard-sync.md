# C8: Clipboard sync (first notes)

Owner: Ahmed. Status: first notes, 17 Oct 2026. Phase 1 is design only: no code.
Names follow `B1-architecture-v1.md` (device app, UI, feature module, core services, communication layer, local storage). The numbers below are our first choice and can change after review.

## 1. What is copied, and size limits
- **Text** (plain text, UTF-8) up to **1 MB**. Formatting (bold, colours, links) is dropped in v1; only the text is sent.
- **Images** (PNG or JPEG) up to **10 MB**.
- **Not synced in v1:** files and folders (that is C6, direct file sharing), rich text, and clipboard history. Only the **latest** copied item counts.
- Anything over the limit is not sent; the user sees "Too large to sync. Use file sharing (C6)".

## 2. When it syncs
- **Text: instantly** (automatically) to every trusted device that is online and has clipboard sync on. Sent over the small-messages channel.
- **Images: only when the user asks**, by tapping "Send to other devices", because they are larger. Sent over the file-transfer channel.
- **Phones and tablets: only when the user asks** (a "Send clipboard" button in the app), for both text and images, because the OS does not let a background app read the clipboard (see point 5).
- Receiving: the item is put on the receiving device's clipboard, ready to paste. A received item is **not sent on again** (it carries the id of the device it came from), so two devices do not bounce the same item back and forth.
- Clipboard sync is **off until the user turns it on** once on each device (privacy by default).

## 3. If the other device is switched off
- A switched-off (or unreachable) device **gets nothing and does not catch up later**. Nothing is stored or queued for it: an old clipboard item that appears hours later is surprising and could leak something private.
- A phone or tablet with the app closed counts as "off" (B1 open decision 12). It receives only while the app is open.
- If the user taps "Send" and no device is online, the UI says "No device online to receive it".

## 4. Pausing, and what is never synced
**Pause:** one switch in the UI on each device, with "Pause for 1 hour" or "Pause until I turn it on". The user can also turn off **sending** or **receiving** for a single device in its trusted-device settings (core services, C1).

**Never synced:**
- Anything copied from our own password manager (C7). The C7 module never hands the copied password to C8.
- Anything that another app marks as secret. Password managers mark it like this: Windows formats `ExcludeClipboardContentFromMonitorProcessing`, `Clipboard Viewer Ignore`, `CanUploadToCloudClipboard = 0`, `CanIncludeInClipboardHistory = 0`; macOS `org.nspasteboard.ConcealedType`; Android `ClipDescription.EXTRA_IS_SENSITIVE`. (Formats from CopyQ and Android docs; not checked against every password manager.)
- Anything to or from a device that is not trusted (B1 rule 2). All items are encrypted in transit (C11) and never go to the internet (B1 rule 1).
- Nothing is saved to local storage: the item lives only on the clipboard.

## 5. What each system allows

| System | Read clipboard in background (needed for automatic sending) | Write received item | Notes |
|---|---|---|---|
| **Windows** | Yes. A desktop app can register with `AddClipboardFormatListener` and is told about every change (`WM_CLIPBOARDUPDATE`). Checked (Microsoft docs). | Yes. Not checked in background. | Can see the "secret" formats above. |
| **macOS** | Possible, but Apple is adding a privacy alert: an app that reads the clipboard without the user pasting shows "[app] is trying to access the pasteboard", with Allow/Deny in Privacy & Security ("Paste from Other Apps"). Preview since macOS 15.4; whether it is on for everyone in macOS 26/27 is **not checked**. No change notification: the app has to check `changeCount` regularly (not checked). | Yes (not checked in background). | New `detect` methods tell the type of data without reading it, so no alert. |
| **Android** | **No** since Android 10: only the app in front (or the keyboard app) can read it. Checked (Android docs). Since Android 12 a message is shown each time an app reads the clipboard. Android 13 clears the clipboard automatically after some time. | Yes (not checked in background). | Sending from Android needs the user to open the app and tap "Send clipboard". |
| **iPhone / iPad** | **No** in practice: apps are suspended in the background (not checked). Since iOS 16, reading the clipboard without the user pasting shows an "Allow Paste" prompt; iOS 16.1 adds a per-app setting (Ask / Allow / Deny). Checked (articles, not Apple docs). | Only while the app is open (not checked). | Sending needs the app open and a tap; `UIPasteControl` avoids the prompt. |
| **Linux** | Not checked (B1 lists Linux; differs between X11 and Wayland). | Not checked | KDE password managers mark secret items with `x-kde-passwordManagerHint = secret`. |

**Result:** automatic sending works only from Windows (and macOS, maybe with one alert). Phones and tablets send only on a tap and receive only while the app is open. Each device advertises a "can sync clipboard in background" capability (B1 rule 5) so the UI offers the right buttons.

## Open points and who settles them
- Encryption and per-feature permission for clipboard (C11, Ali).
- Phones in the background (B1 decision 12, B6): if B6 finds a way, phones could receive in the background.
- Do the size limits fit the chosen communication library (B6)?

## Sources (read 17 Oct 2026 through summaries)
- Android: https://developer.android.com/privacy-and-security/risks/secure-clipboard-handling
- Windows: https://learn.microsoft.com/en-us/windows/win32/dataxchg/using-the-clipboard
- macOS: https://mjtsai.com/blog/2025/05/12/pasteboard-privacy-preview-in-macos-15-4
- iOS 16: https://sarunw.com/posts/uipasteboard-privacy-change-ios16/ and https://www.macstories.net/linked/ios-and-ipados-16-1-betas-add-per-app-clipboard-access-permissions-to-the-settings-app/
- Secret clipboard formats: https://copyq-de.readthedocs.io/de/stable/security.html
