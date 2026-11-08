# C9: Screen sharing and remote control (final)

Owner: Ahmed. Status: final, 8 Nov 2026 (first version `C9-screen-sharing-notes.md`, 29 Oct; final was due Mon 2 Nov). Phase 1 is design only: no code.
Changed in this version: section 3 is now the decision (WebRTC, B6 final); the macOS, iPhone/iPad and Android "not checked" points were looked up again in Apple's and Google's own pages (section 1); stream settings added (section 3); step-by-step diagram (section 5) and requirements (section 6) added.
Names follow `B1-architecture-v2.md` (device app, UI, feature module, core services, communication layer). Technology from `B6-technology-final.md`; trust from C1 v2 (draft); permissions from `C11-security-privacy.md` (first version). "Shared device" = the one whose screen is shown; "viewer" = the one that watches and maybe controls.
**[PROPOSAL]** = my design choice or number, to be agreed or tuned in Phase 2. **not checked** = not confirmed in a primary source or by a test.

## 1. What each system requires

| System | (a) Capture the screen | (b) Accept mouse and keyboard from another device |
|---|---|---|
| **Windows** | `Windows.Graphics.Capture`: the user picks the screen or window in a "secure system UI" (`GraphicsCapturePicker`), and "a yellow notification border is drawn by the system around the actively captured item". Checked (Microsoft docs). The older Desktop Duplication API (Windows 8+) has no picker, and its pages mention no border or other sign (searched, not found); it cannot capture the secure desktop: "only an application that runs at LOCAL_SYSTEM can access the secure desktop". Checked (Microsoft docs). We use the picker, so the user always sees the border. | No permission. `SendInput` works, but "applications are permitted to inject input only into applications that are at an equal or lesser integrity level": windows run as administrator cannot be controlled, and "this function fails when it is blocked by UIPI". UAC prompts: "The consent and credential prompts are displayed on the secure desktop by default" and "Only Windows processes can access the secure desktop", so the viewer cannot see or answer a UAC prompt; the user at the PC must. Checked (Microsoft docs). The lock screen: searched, not found in these pages, **not checked** (we assume the same as the secure desktop). |
| **macOS** | **Screen & System Audio Recording** permission: in System Settings the user turns recording on or off "for each app listed" (Apple Support). ScreenCaptureKit: "Request screen recording permission from the person before capturing content"; after the first grant "you need to restart the app to enable capture" (Apple sample); the system picker `SCContentSharingPicker` (macOS 14+) is "the recommended approach". Checked (Apple docs). That macOS asks again from time to time (B6 v0 [B6-14]): **not found in Apple's pages, not checked**. | **Accessibility** permission. Apple Support: "When a third-party app tries to access and control your Mac through accessibility features, you receive an alert", and the user "must specifically grant the app access" in Privacy & Security. Checked (Apple Support). That posting mouse and keyboard events (`CGEvent.post`) needs exactly this switch: the API pages (`CGRequestPostEventAccess`, `CGPreflightPostEventAccess`) still have no description, so **not checked in Apple docs**; Phase 2 test. |
| **Linux** | X11: the X.Org security page controls only who may **connect** to the display ("Any client on a host in the host access control list is allowed access to the X server", or a matching cookie); it lists no further permission for reading the screen. So no dialog on X11 (checked for the connection rule; that nothing else is asked is our reading of the page). Wayland: a system dialog through xdg-desktop-portal (ScreenCast) each time [B6-15]. | X11: the XTEST extension allows "limited synthesis of input device events, almost as if a cooperative user had moved the pointing device or pressed a key"; its specification lists no access control beyond the connection (checked, X.Org spec). Wayland: the same portal dialog (RemoteDesktop) [B6-15]. |
| **Android** | **MediaProjection**: "Your app must request user consent before each media projection session"; a token is used only once. Needs a foreground service of type `mediaProjection`. Since Android 14 the user can share one app instead of the whole screen. The session ends when "the screen is being locked", when the user taps the system's status bar chip, or when another projection starts. Checked (Android docs). | An **accessibility service** the user turns on in Settings [B6-4]. It taps and swipes with `dispatchGesture` (API level 24, so from Android 7, our minimum): "The gesture will be dispatched as if it were performed directly on the screen by a user"; the service must declare `canPerformGestures`. Checked (Android API reference). Google Play: needs a "prominent" in-app disclosure and "affirmative user action for consent". Checked (Play policy). The policy page does not mention remote control at all (searched for "remote": not found); it forbids use that lets an app "autonomously initiate, plan, and execute actions" and allows "deterministic, rule-based automation". Our control is done live by a person, so it is not autonomous **[our reading]**; whether Google Play accepts it: **not checked** (only a review can say). Typing text: **not checked**. |
| **iPhone / iPad** | iOS 15 to 26: only a broadcast the user starts (ReplayKit broadcast extension from the system broadcast button `RPSystemBroadcastPickerView`, iOS 12+). Apple's ScreenCaptureKit page now lists iOS/iPadOS 27 and says it "replaces ReplayKit for screen streaming and mirroring. A broadcast extension is no longer necessary", with the system picker `SCContentSharingPicker` (iOS 27+) and background modes for capture while the app is in the background. Checked (Apple docs). So: ReplayKit for iOS 15 to 26, ScreenCaptureKit on iOS 27+ **[PROPOSAL]**. `flutter_webrtc` supports both iOS modes ("In-App" and "Broadcast Extension"; the extension needs iOS 14+) [flutter_webrtc wiki]. Limits of the broadcast extension (memory, time): searched Apple's ReplayKit pages (`RPBroadcastSampleHandler`, `RPSystemBroadcastPickerView`, `RPScreenRecorder`), not found, **not checked**. | **Not possible**: no public API found to inject touches or keys (RustDesk cannot [B6-4]); absence **not checked in Apple docs** (an absence cannot be quoted). iPhone/iPad can only be viewed, or be the viewer. |

**System signs on the shared device (what the user sees from the system, not from us):**
- Windows: yellow border around the captured screen or window (checked).
- Android: the "large and prominent" status bar chip on Android 15 QPR1 or later; the user can tap it to stop (checked, Android docs). Older Android versions: **not checked**.
- macOS: Apple's menu bar pages describe privacy indicators (for example "a purple dot indicates the system audio is being recorded"), but **no screen-recording indicator** and not which app records. So the macOS sign is **not checked**; we rely on our own bar (section 4).
- iPhone/iPad: a sign during a broadcast (red status bar or similar): not in Apple's ReplayKit developer pages, and Apple's iPhone guide page could not be read in full: searched, not found, **not checked**.

**Result:** every system can be viewed after a user approval on the shared device. Control works on Windows, macOS, Linux and Android (Android only after the user turns on an accessibility service; Play acceptance not checked); iPhone/iPad is view-only. Each device advertises "can be viewed" and "can be controlled" (B1 rule 5).

## 2. Live video needs its own connection
Video is large and never stops; on the same TLS connection as messages, one big frame would hold up a clipboard item or a "Stop" command behind it. B1 and B6 final keep "live streams on their own connection". The request, the approval, "Stop" and the WebRTC setup messages stay on the normal messages connection, with the same trust (C1). Mouse and keyboard events go with the stream (section 3).

## 3. Decision: WebRTC for the live stream (B6 decision 10)
**Decided** in `B6-technology-final.md` choice 3 (Ali, 31 Oct): live streams for C9 and C10 use **WebRTC**, set up over our existing trusted messages connection. It was my proposal in the first version; I disagree with nothing in B6.

**Why (my reasons, kept):** WebRTC already does what a live stream needs and we would otherwise build: video compression, sending over UDP so a lost packet does not freeze the picture (TLS runs over TCP, which waits for every lost packet), lowering quality when Wi-Fi is slow, and a data channel for mouse and keyboard events. `flutter_webrtc` supports screen capture on all five systems [B6-20], and RustDesk shows that a home-made stream is a lot of work.

**How it keeps our rules:**
- **Who may start it:** the offer and answer (SDP) and the network candidates travel only over our TLS messages connection, so only a trusted, authenticated device (C1 section 1.5) can start a session, and only after the user on the shared device accepts (section 4).
- **Encryption:** WebRTC encrypts with its own DTLS key. Its fingerprint is inside the SDP, which came over our pinned connection; WebRTC itself checks that the other side's DTLS certificate matches that fingerprint: the W3C WebRTC standard says the transport is "connected" only when DTLS "verified the remote fingerprint", and reports `fingerprint-failure` when the "remote certificate did not match any of the fingerprints provided in the SDP" (checked, W3C spec). So the trust still comes from pairing (C1, C11). That `flutter_webrtc` (built on Google's WebRTC library) behaves as the standard says: **not checked**, Phase 2 test.
- **No internet:** no STUN or TURN servers (B1 rule 1); in the W3C standard the server list `iceServers` defaults to empty (checked, W3C spec). Only local-network candidates are used; devices on the same network connect directly. If two devices cannot reach each other directly (for example a guest Wi-Fi that blocks devices), there is no relay: the app says "Cannot start the stream on this network". Whether the native WebRTC library hides local addresses behind `.local` (mDNS) names, as browsers do: the W3C standard only mentions "mDNS candidates"; the `flutter_webrtc` README says nothing: searched, not found, **not checked**.
- **Control events:** sent on a WebRTC data channel (reliable, ordered) beside the video, so they arrive with the same low delay and the same encryption **[PROPOSAL]**. The shared device accepts them only if the user chose "Allow view and control" for this session.

**Stream settings [PROPOSAL, to tune in Phase 2]:** video only, no audio in v1; at most the shared screen's size and 1920 × 1080; up to 30 frames per second. Targets for A9: the picture appears within 5 s after the user accepts; delay from screen to viewer under 200 ms on the same Wi-Fi; Stop ends capture within 1 s. All three are targets, **not measured**.

**What we give up (from B6):** WebRTC adds size to the app and a second encryption layer. **Still open (B6 section 4):** WebRTC with no internet at all, and the app size it adds: Phase 2 tests.

**Fallback (if WebRTC fails the Phase 2 tests):** our own TLS stream with compressed frames (for example JPEG) on its own connection; simpler, but slower and with more delay.

## 4. What is shared and when, switched-off devices, what the user sees
- **What is shared:** only the screen picture, never audio in v1. The user of the shared device picks the whole screen or one window/app in the system picker (Windows, macOS 14+, Android 14+, Wayland, iOS 27+). **Control is separate** and off by default.
- **When:** only between trusted devices (B1 rule 2), only if screen sharing is switched on for the viewer (C11 section 3), and only when the user **on the shared device** accepts each request: "Laptop wants to see your screen: Allow view / Allow view and control / Deny" (C11: every time). No unattended access in v1: nobody can watch a device with nobody at it. "Allow view and control" is not offered when the device cannot be controlled (iPhone/iPad, Android without the accessibility service).
- **If a device is switched off, sleeps, locks or leaves the network:** the stream stops; the viewer shows "Connection lost" over the last frame. Nothing is recorded or queued, and no frame is stored (C11 SEC-6). The viewer tries again for 30 seconds **[PROPOSAL]**; after that a new request (and approval) is needed. Android ends capture when the screen locks. We cannot switch a device on remotely.
- **What the user sees on the shared device:** the system's own sign where there is one (section 1), plus our own bar that is always on top: "Being viewed by Laptop" or "Controlled by Laptop", with a **Stop** button. Stop is also on the notification (Android) and works at once. The local mouse and keyboard always work; control is shown as "Controlled by ..." the whole time.

## 5. One session, step by step
Diagram: `diagrams/C9-screen-sharing-sequence.drawio` (PNG export next to it). Example: the laptop views and controls a Windows PC.

![C9 sequence: one screen-sharing session with control](../diagrams/C9-screen-sharing-sequence.png)

- Step 1: On the laptop, the user taps "View screen" on the PC and asks for view and control.
- Steps 2-3: The laptop's screen-sharing module asks core services: is the PC trusted, online, able to be viewed and controlled (capabilities, B1 rule 5), and is screen sharing on? Yes.
- Step 4: The request (session id, "view and control") goes to the PC over the trusted messages connection.
- Step 5: The PC's core services check that the laptop is trusted and that screen sharing is switched on for it (C11).
- Steps 6-7: The PC's module shows its user "Laptop wants to see your screen: Allow view / Allow view and control / Deny".
- Step 8: The PC's user taps "Allow view and control".
- Steps 9-10: The system picker opens (Windows `GraphicsCapturePicker`) and the user picks the screen; capture starts, Windows draws its yellow border, our bar shows "Controlled by Laptop" with Stop.
- Steps 11-12: The PC sends "accepted" and the WebRTC offer (with its DTLS fingerprint and local candidates only) over the messages connection; the laptop answers the same way.
- Step 13: The WebRTC connection opens directly between the two devices; each side checks that the DTLS fingerprint equals the one in the setup messages; video flows (encrypted, UDP, no server).
- Step 14: The laptop shows the PC's screen.
- Steps 15-16: The laptop's user moves the mouse or types; the events go on the data channel; the PC's module checks that control was allowed in this session and passes them to Windows (`SendInput`).
- Steps 17-18: The PC's user taps Stop (the viewer can also close): capture and WebRTC stop at once; "stopped" goes over the messages connection and the laptop shows "Session ended".

Not drawn: if the PC's user taps **Deny** at step 8, the PC answers "denied" and nothing is captured; if the viewer stops, the same as steps 17-18 happens from the other side; if the connection is lost, both sides end the session as in section 4.

Other systems follow the same steps; only the picker and the input call change (section 1). On an iPhone/iPad as shared device, step 9 is the system broadcast button (iOS 15 to 26) and "Allow view and control" is not offered.

## 6. Requirements
1. The app must keep screen sharing and remote control off until the user turns them on, per device (C11).
2. The app must start a session only between trusted devices, and only after the user on the shared device accepts that request; there is no unattended access.
3. The app must ask for view and for control separately ("Allow view / Allow view and control / Deny") and must not offer control on a device that cannot be controlled (iPhone/iPad; Android without the accessibility service).
4. The app must let the user of the shared device choose the whole screen or one window/app through the system picker where the system has one (Windows, macOS 14+, Android 14+, Linux on Wayland, iOS 27+).
5. The app must advertise per device whether it can be viewed and whether it can be controlled (B1 rule 5).
6. The app must send the stream with WebRTC, set up only over the trusted messages connection, with no STUN or TURN server and only local-network candidates.
7. The app must check that the WebRTC DTLS fingerprint equals the one received over the trusted connection, and end the session if it does not.
8. The app must send the request, the answer, Stop and the WebRTC setup messages on the messages connection, and the video on its own connection.
9. The app must accept mouse and keyboard events only from the viewer of the current session, and only if the user allowed control for that session.
10. The app must show on the shared device, for the whole session, a bar that is always on top with "Being viewed by ..." or "Controlled by ..." and a Stop button.
11. The app must end capture and the stream within 1 second when Stop is tapped on either device **[PROPOSAL number]**.
12. The app must keep the local mouse and keyboard working during remote control.
13. The app must share only the picture, never audio, in v1.
14. The app must not record, store or queue screen frames (C11 SEC-6).
15. The app must stop the session when either device goes off, sleeps, locks or leaves the network, show "Connection lost", and ask for a new approval after 30 seconds **[PROPOSAL number]**.
16. The app must show "Cannot start the stream on this network" when the two devices cannot reach each other directly.
17. The app must, on Android, show the in-app disclosure that Google Play requires before the user turns on the accessibility service for control.

## Open points and who settles them
| # | Point | Who | Status |
|---|---|---|---|
| 1 | WebRTC with no internet at all; app size it adds; whether screen streaming needs native code | Ali (B6 section 4), Phase 2 test | still open |
| 2 | `flutter_webrtc` follows the standard's fingerprint check; whether it uses `.local` (mDNS) candidates | Ali (B6), Phase 2 test | standard checked, library not checked |
| 3 | C10 reuses this stream; extra-screen differences (continuous, delay, resolution) | C10 owner | waiting for C10 |
| 4 | Approval for C10: C11 proposes "user on both devices, every time" | C10 owner with Ali (C11) | C11 proposal |
| 5 | Capability flags for C1 (Mahmoud asked): "can be viewed", "can be controlled", "can view", "view needs broadcast button" (iPhone/iPad 15 to 26) | Mahmoud (C1) | my answer, to confirm |
| 6 | macOS: is posting events covered by Accessibility, and does Screen Recording need re-approval | Phase 2 test on a Mac | not checked |
| 7 | iPhone/iPad sign during a broadcast; broadcast extension limits | Phase 2 test on an iPhone | not checked |
| 8 | Google Play acceptance of a remote-control accessibility service; typing text on Android | Phase 2 | not checked |
| 9 | Who has a Mac with Xcode for macOS/iOS tests (B1 decision 15) | the team | still open |
| 10 | Delay, start time and Stop time targets | Phase 2 test (A10) | targets only |

## Sources
Opened 29 Oct 2026 (first version) and 8 Nov 2026 (this version, marked *8 Nov*). Apple developer pages read as text from Apple's own documentation data; Apple Support, Microsoft, X.Org and Google Play pages read through a page reader.
- Windows screen capture: https://learn.microsoft.com/en-us/windows/apps/develop/media-authoring-processing/screen-capture
- Windows SendInput: https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-sendinput
- macOS/iOS ScreenCaptureKit (iOS 27, "replaces ReplayKit", background modes) *8 Nov*: https://developer.apple.com/documentation/screencapturekit
- `SCContentSharingPicker` (macOS 14, iOS 27) *8 Nov*: https://developer.apple.com/documentation/screencapturekit/sccontentsharingpicker
- Apple sample "Capturing screen content in macOS" (restart after permission) *8 Nov*: https://developer.apple.com/documentation/screencapturekit/capturing-screen-content-in-macos
- `CGRequestPostEventAccess`, `CGPreflightPostEventAccess`, `CGEvent.post(tap:)` *8 Nov*: https://developer.apple.com/documentation/coregraphics/cgrequestposteventaccess() and https://developer.apple.com/documentation/coregraphics/cgevent/post(tap:)
- `AXIsProcessTrusted`: https://developer.apple.com/documentation/applicationservices/1460720-axisprocesstrusted
- Apple Support, "Control access to screen and system audio recording on Mac" *8 Nov*: https://support.apple.com/guide/mac-help/control-access-screen-system-audio-recording-mchld6aa7d23/mac
- Apple Support, "Allow accessibility apps to access your Mac" *8 Nov*: https://support.apple.com/guide/mac-help/allow-accessibility-apps-to-access-your-mac-mh43185/mac
- Apple Support, "What's in the menu bar on Mac?" (privacy indicators) *8 Nov*: https://support.apple.com/guide/mac-help/use-the-menu-bar-mchlp1446/mac
- Apple Support, "Take a screen recording on iPhone" (body not readable, 8 Nov): https://support.apple.com/guide/iphone/take-a-screen-recording-iph52f6e1987/ios
- iOS ReplayKit: https://developer.apple.com/documentation/replaykit and https://developer.apple.com/documentation/replaykit/rpsystembroadcastpickerview
- Android MediaProjection (consent, status bar chip, stop on lock) *8 Nov*: https://developer.android.com/media/grow/media-projection
- Android `AccessibilityService.dispatchGesture` *8 Nov*: https://developer.android.com/reference/android/accessibilityservice/AccessibilityService
- Google Play, Accessibility API policy (re-read 8 Nov): https://support.google.com/googleplay/android-developer/answer/10964491
- Windows Desktop Duplication API and `DuplicateOutput` *8 Nov*: https://learn.microsoft.com/en-us/windows/win32/direct3ddxgi/desktop-dup-api and https://learn.microsoft.com/en-us/windows/win32/api/dxgi1_2/nf-dxgi1_2-idxgioutput1-duplicateoutput
- Windows, "How User Account Control works" (secure desktop) *8 Nov*: https://learn.microsoft.com/en-us/windows/security/application-security/application-control/user-account-control/how-it-works
- X.Org XTEST protocol and Xsecurity *8 Nov*: https://www.x.org/releases/current/doc/xextproto/xtest.html and https://www.x.org/releases/current/doc/man/man7/Xsecurity.7.xhtml
- W3C WebRTC 1.0 (`RTCDtlsTransportState`, `fingerprint-failure`, `iceServers`), editor's draft source *8 Nov*: https://github.com/w3c/webrtc-pc/blob/main/webrtc.html
- `flutter_webrtc` README and wiki "iOS Screen Sharing" *8 Nov*: https://github.com/flutter-webrtc/flutter-webrtc and https://github.com/flutter-webrtc/flutter-webrtc/wiki/iOS-Screen-Sharing
- Apple ReplayKit pages `RPBroadcastSampleHandler`, `RPScreenRecorder`, `RPBroadcastController` *8 Nov* (searched for limits and indicator): https://developer.apple.com/documentation/replaykit
- Team files *8 Nov*: `B6-technology-final.md` choice 3 and section 4 (pull request 42); `C11-security-privacy.md` sections 3, 4 and 7 (pull request 49); `C1-device-discovery-pairing-trust.md` sections 1.5 and 9 (pull request 38); `B1-architecture-v2.md` rules 1, 2, 5 and decisions 10, 15.
- [B6-n] = source n in `report/B6-technology-v0.md` (4 RustDesk, 14 LogMeIn, 15 xdg-desktop-portal, 20 flutter_webrtc).
