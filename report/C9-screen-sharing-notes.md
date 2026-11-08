# C9: Screen sharing and remote control (first notes)

Owner: Ahmed. Status: first version, 29 Oct 2026 (due Fri 30 Oct; final Mon 2 Nov). Phase 1 is design only: no code.
Names follow B1 (device app, feature module, core services, communication layer). "Shared device" = the one whose screen is shown; "viewer" = the one that watches and maybe controls. **Not checked** = not confirmed in a primary source or not tested.

## 1. What each system requires
Started from B6 section 1 and checked against the official docs.

| System | (a) Capture the screen | (b) Accept mouse and keyboard from another device |
|---|---|---|
| **Windows** | `Windows.Graphics.Capture`: the user picks the screen or window in a "secure system UI" (`GraphicsCapturePicker`), and "a yellow notification border is drawn by the system around the actively captured item". Checked (Microsoft docs; this is the border of this API only: other capture APIs may not show it, **not checked**). Capture without the picker (Desktop Duplication): **not checked**. | No permission. `SendInput` works, but "applications are permitted to inject input only into applications that are at an equal or lesser integrity level": windows run as administrator cannot be controlled, and the failure is silent. Checked (Microsoft docs). UAC prompts and the lock screen: **not checked**. |
| **macOS** | **Screen Recording** permission (Privacy & Security). ScreenCaptureKit: "Request screen recording permission from the person before capturing content"; the system picker `SCContentSharingPicker` (macOS 14+). Checked (Apple docs). B6 says it must be re-approved from time to time [B6-14]: **not checked in Apple docs**. | **Accessibility** permission is what B6 says [B6-14]. Apple's page for `AXIsProcessTrustedWithOptions` only says it "returns whether the current process is a trusted accessibility client", and the `CGRequestPostEventAccess` page has no text, so that sending mouse and keyboard events needs exactly this permission (or a separate "post event" one) is **not checked in Apple docs**. |
| **Linux** | X11: no permission (**not checked**). Wayland: a system dialog through xdg-desktop-portal (ScreenCast) each time [B6-15]. | X11: no permission (XTest, **not checked**). Wayland: the same portal dialog (RemoteDesktop) [B6-15]. |
| **Android** | **MediaProjection**: "Your app must request user consent before each media projection session"; a token is used only once (Android 14+ throws an error if it is reused). Needs a foreground service of type `mediaProjection`. Since Android 14 the user can share one app instead of the whole screen. Capture also stops when the screen is locked (Android docs list this among the reasons a session ends; the exact Android version: **not checked**). Checked (Android docs). | An **accessibility service** the user turns on in Settings [B6-4]; it can perform taps and swipes (`dispatchGesture`, **not checked**). Google Play: needs a "prominent" in-app disclosure and "affirmative user action for consent"; we are not an accessibility tool, and the policy forbids apps that act autonomously through the API. Checked (Play policy). **Not checked:** whether Google Play would accept a remote-control app like ours, and typing text. |
| **iPhone / iPad** | Only a broadcast the user starts (ReplayKit broadcast extension, started from a system broadcast button, `RPSystemBroadcastPickerView`; iOS 12 to 27 per Apple docs). Apple's ScreenCaptureKit page says it "replaces ReplayKit for screen streaming and mirroring" and lists iOS 27; for our minimum iOS 15 (B6) ReplayKit stays, and what ScreenCaptureKit changes on iOS 27: **not checked**. Limits of the extension: **not checked**. | **Not possible**: no public API found to inject touches or keys (RustDesk cannot [B6-4]); **not checked in Apple docs**. iPhone/iPad can only be viewed, or be the viewer. |

**Result:** every system can be viewed after a user approval on the shared device. Control works on Windows, macOS, Linux and Android (Android only after the user turns on an accessibility service; Play acceptance not checked); iPhone/iPad is view-only. Each device advertises "can be viewed" and "can be controlled" (B1 rule 5).

## 2. Does live video need its own connection?
**Yes.** Video is large and never stops; on the same TLS connection as messages, one big frame would hold up a clipboard item or a "Stop" command behind it. B6 section 3 already plans "live streams on their own connection". The request, the approval, "Stop" and the WebRTC setup messages stay on the normal messages connection, with the same trust (C1). Mouse and keyboard events go with the stream (section 3).

## 3. WebRTC or our own TLS stream (B6 decision 10)
**My opinion: WebRTC for the video, set up over our existing TLS messages connection.** Ali's first pick is our own stream; this needs agreeing with Ali (**not agreed yet**).
- **Why WebRTC:** it already does what a live stream needs and we would otherwise build: video compression, sending over UDP so a lost packet does not freeze the picture (TLS runs over TCP, which waits for every lost packet), lowering quality when Wi-Fi is slow, and a data channel for mouse and keyboard events. `flutter_webrtc` supports screen capture on all five systems [B6-20], and RustDesk shows that a home-made stream is a lot of work.
- **The cost, and how to keep our rules:** WebRTC needs signalling; we send its setup messages over our TLS messages connection, so only trusted devices can start it. No STUN or TURN servers (B1 rule 1: no internet); devices on the same network connect directly. WebRTC encrypts with its own key (DTLS); its fingerprint travels inside the setup messages over our pinned TLS connection, so the trust still comes from pairing (C1, C11). **Not checked:** that this works with no internet at all, and the app size it adds.
- **Fallback:** if WebRTC causes problems, our own TLS stream with compressed frames (e.g. JPEG) on its own connection; simpler, but slower and with more delay.

## 4. What is shared and when, switched-off devices, what the user sees
- **What is shared:** only the screen picture, never audio in v1. The user of the shared device picks the whole screen or one window/app in the system picker (Windows, macOS 14+, Android 14+, Wayland). **Control is separate** and off by default.
- **When:** only between trusted devices (B1 rule 2), and only when the user **on the shared device** accepts each request: "Laptop wants to see your screen: Allow view / Allow view and control / Deny". No unattended access in v1: nobody can watch a device with nobody at it.
- **If a device is switched off, sleeps, locks or leaves the network:** the stream stops; the viewer shows "Connection lost" over the last frame. Nothing is recorded or queued. The viewer tries again for 30 seconds; after that a new request (and approval) is needed. Android ends capture when the screen locks. We cannot switch a device on remotely.
- **What the user sees on the shared device:** the system's own sign (Windows yellow border, Android status bar chip; macOS and iOS indicators **not checked**), plus our own bar that is always on top: "Being viewed by Laptop" or "Controlled by Laptop", with a **Stop** button. Stop is also on the notification (Android) and works at once. The local mouse and keyboard always work; control is shown as "Controlled by ..." the whole time.

## Open points and who settles them
- WebRTC or own stream: agree with Ali (B6 decision 10), also with C10 (extra screen), which needs the same stream.
- Per-feature permission and encryption of the stream: C11 (Ali).
- Who has a Mac with Xcode for macOS/iOS tests (B6 section 1).

## Sources (opened 29 Oct 2026, read as text from the official docs)
- Windows screen capture: https://learn.microsoft.com/en-us/windows/apps/develop/media-authoring-processing/screen-capture
- Windows SendInput: https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-sendinput
- macOS ScreenCaptureKit: https://developer.apple.com/documentation/screencapturekit
- macOS SCContentSharingPicker: https://developer.apple.com/documentation/screencapturekit/sccontentsharingpicker
- macOS AXIsProcessTrustedWithOptions: https://developer.apple.com/documentation/applicationservices/1459186-axisprocesstrustedwithoptions
- macOS CGRequestPostEventAccess: https://developer.apple.com/documentation/coregraphics/cgrequestposteventaccess()
- Android MediaProjection: https://developer.android.com/media/grow/media-projection
- Google Play, Accessibility API policy: https://support.google.com/googleplay/android-developer/answer/10964491
- iOS ReplayKit: https://developer.apple.com/documentation/replaykit and https://developer.apple.com/documentation/replaykit/rpsystembroadcastpickerview
- [B6-n] = source n in `report/B6-technology-v0.md` (4 RustDesk, 14 LogMeIn, 15 xdg-desktop-portal, 20 flutter_webrtc).
