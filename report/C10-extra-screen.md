# Extra Screen

## 1. What the user sees and gains

Extra Screen lets the user turn a tablet or phone they already own into a second monitor for their computer. Instead of buying a monitor, the user places an iPad or Android tablet next to the laptop, taps "Use as extra screen", and can drag windows onto it and use it by touch.

Two devices take part. The **host device** is the PC or laptop that gains the extra screen. The **screen device** is the tablet or phone that shows it.

What the user gets depends on the host device:

- On **Windows** (with the tool's own signed display driver) and on **Linux with Wayland** (where the desktop offers a virtual monitor), the tool adds a **true extra screen**: the computer treats the tablet as a new monitor and the desktop becomes larger.
- **macOS has no public way** for an app to add a screen, so on a Mac the tool **mirrors** the existing screen instead: the tablet shows a copy of the Mac's screen. The same happens on any other computer where an extra screen cannot be added, and the tool tells the user so.

The user stays in control at all times:

- The user must approve on **both devices, every time** a session starts: once on the screen device by tapping "Use as extra screen", and once on the host device by choosing "Allow". Nothing is remembered between sessions.
- Either device can stop the session at any moment. When it stops, the extra screen is removed and its windows move back to the main screen, so nothing is lost on a screen nobody can see.
- The picture travels only between the user's own paired devices on the local network, encrypted, and is never stored.

## 2. Adding an extra screen on the host device

| Operating system | How a new (virtual) screen can be added | What it needs |
|---|---|---|
| **Windows** | The **Indirect Display Driver** model: a driver model that runs in user mode, made for monitors that are not connected to a graphics card output, including "creating virtual monitors" and "streaming the display output over a network to a remote client". Spacedesk works in this way, with its own driver. | **A signed driver.** It runs in user mode, so it contains no kernel code, but it must be installed by an administrator and signed. Since Windows 10 version 1607, new kernel-mode drivers must be signed through Microsoft's developer portal. Attestation signing covers kernel-mode and user-mode drivers and needs an extended validation (EV) code signing certificate; it is meant for testing, and a public release needs the Windows Hardware Compatibility Program. For development, test signing mode lets a test-signed driver load and shows "Test Mode" on the desktop. |
| **macOS** | **No public way.** Apple documents no way for an app to add a display. Tools that do so use `CGVirtualDisplay`, which is not in Apple's public documentation. Spacedesk offers a macOS driver in beta (macOS 14.4 and later) with an extend and a mirror setting. Apple's own way is Sidecar, which works only with an iPad. | A **private interface**, which is not allowed on the Mac App Store and may stop working with any update. The tool does not use it, so on macOS the tool **mirrors** the screen (section 5). |
| **Linux with X11** | `xrandr --setmonitor` only divides **existing** outputs. A new output needs the **EVDI** kernel module (which works with the standard Linux display system, so xrandr can drive it) or the `VirtualHeads` option of the old `intel` X driver, which creates "fake outputs". | A **kernel module** that must be built for each kernel, or a specific X driver. Not used in the first release: the tool mirrors instead. |
| **Linux with Wayland** | The ScreenCast portal has a source type **"Extend with new virtual monitor"** (`VIRTUAL`). The desktop creates the monitor and passes its picture to the tool, after the user approves in the system dialog. GNOME (`xdg-desktop-portal-gnome`) and KDE Plasma (`xdg-desktop-portal-kde`, with a resolution picker) support it; the portal used by Sway and other wlroots desktops (`xdg-desktop-portal-wlr`) does not. KDE has a reported bug that fixes the size at 1920 × 1080. | **No driver.** The same portal and system dialog as Screen Sharing and Remote Control. The portal could remember the approval (`persist_mode`), but the tool does not use this (section 4). |

**Result.** A true extra screen is possible on **Windows, with the tool's own signed driver**, and on **Linux with Wayland, through the system portal and without a driver**. macOS has no public way, and Linux with X11 would need an extra kernel module, so on both the tool mirrors the screen.

## 3. What the screen device can do

| Screen device | Showing the stream | Sending touch back as mouse input | Limits |
|---|---|---|---|
| **iPad** | Yes, while the app is open (WebRTC video through `flutter_webrtc`). | Yes. The app reads the touches and sends them; the host device moves the mouse (the same input path as Screen Sharing and Remote Control). | Works **only while the app is in front**: when the user leaves the app or the iPad locks, iOS suspends the app and the stream stops. `isIdleTimerDisabled` keeps the screen awake while the app is open. Apple Pencil pressure and hover are not used in the first release. An iPad can never be the host device, because there is no public way to add a display to it or to control it. |
| **iPhone** | Same as iPad. | Same as iPad. | Same as iPad, with a small screen. Allowed, but not the main case. |
| **Android tablet** | Yes. | Yes, as on the iPad. | Receiving needs no special permission. The app keeps the screen on with `FLAG_KEEP_SCREEN_ON`; if the app goes into the background, the screen may turn off normally. Background work is not needed, because the user is looking at the screen. |

## 4. The stream

**Shared with Screen Sharing and Remote Control.** The picture is sent with WebRTC, as decided in the Technology Choices chapter. The setup messages travel over the trusted messages connection, no STUN or TURN server is used, the DTLS fingerprint travels inside that connection, and sessions run only between paired devices. Touch and keyboard input travel on the WebRTC data channel.

**Different from Screen Sharing and Remote Control:**

- **Continuous, with low delay.** The extra screen is used like a monitor, so the target is a delay **under 100 ms** from drawing to showing on the same Wi-Fi, to be tested in Phase 2. When the Wi-Fi is slow, the picture quality is lowered before the frame rate (WebRTC already adapts).
- **The size comes from the screen device.** The virtual screen is created at the tablet's size (for example 2048 × 1536 for a common iPad, or 1920 × 1200 for an Android tablet), so the picture is not scaled. When mirroring (section 5), the host device's size is used and black bars are added.
- **The mouse can reach the extra screen.** When the screen is extended, the operating system treats it as a normal monitor, and the user drags windows onto it. A touch on the tablet becomes an absolute mouse position on that screen only (on Wayland with `NotifyPointerMotionAbsolute` of the RemoteDesktop portal; on Windows with `SendInput`).
- **Approval on both devices, every time.** The user starts the session on the screen device ("Use as extra screen") and confirms it on the host device ("iPad wants to be an extra screen: Allow / Deny"). The operating system's own dialog is added where the system requires one (the Wayland portal dialog, or Screen & System Audio Recording on macOS). Nothing is remembered between sessions: the portal's `persist_mode` is not used.
- **A clean stop.** When the stream stops, the host device removes the virtual screen, so windows move back and are not lost on a monitor nobody can see.

## 5. Mirroring when an extra screen cannot be added

If the host device cannot add a screen, the tool **mirrors** an existing screen, using the same capture as Screen Sharing and Remote Control and the same approval on both devices every time. The user sees "Extra screen is not available on this computer; showing a copy of your screen".

| Host device | First release |
|---|---|
| Windows 10 and 11 | **Extend** when the tool's signed driver is installed; otherwise **mirror** |
| Linux with Wayland, desktop offers a virtual monitor (GNOME, KDE Plasma) | **Extend** |
| Linux with Wayland, desktop without a virtual monitor (Sway and other wlroots desktops) | **Mirror** |
| Linux with X11 | **Mirror** |
| macOS | **Mirror only** (Apple's Sidecar remains available to iPad owners who want to extend) |
| Android, iPhone, iPad | **Not offered**: these are screen devices only |

Each host device tells the other devices whether it offers extend or mirror, and each device that can show the stream says so, so the screen device shows the right button.

## 6. Comparison with Spacedesk and Apple Sidecar

| | Spacedesk | Apple Sidecar | Extra Screen in this tool |
|---|---|---|---|
| Host device | Windows 10 and 11 (macOS 14.4 and later in beta) | Mac, macOS 10.15 and later | Windows (extend), Linux (extend or mirror), macOS (mirror only) |
| Screen device | Android, iOS, Windows, web browser | iPad only (iPadOS 13 and later) | iPad, iPhone, Android tablet |
| Connection | Local network, no account | Same Apple Account, wireless within 10 m or USB | The tool's own pairing, local network only |
| Touch back | Yes (touch or mouse) | Apple Pencil; wider touch support from version 27 | Touch as mouse; Apple Pencil not in the first release |

**Taken from these products:** the driver approach on Windows (Spacedesk), and the one-tap start and the removal of the screen at the end (Sidecar).

**Done differently:** no Apple account is needed and devices of different brands work together; the stream uses the tool's own pairing and encryption rather than an open network service (Spacedesk needs a password before it accepts keyboard input from phones); and the tool openly falls back to mirroring where extending is not possible.

## 7. Starting an extra screen, step by step

The diagram shows a Windows laptop with the tool's driver as the host device and an iPad as the screen device.

![Extra Screen: start, use and stop](../diagrams/C10-extra-screen-sequence.png)

1. and 2. The user taps "Use as extra screen" on the iPad. The request, with the iPad's screen size, goes to the laptop over the trusted messages connection.
3. The laptop's core services check that the iPad is paired and that Extra Screen is allowed for it, and the user confirms on the laptop. If the user chooses Deny, or nobody answers, nothing more happens and the iPad shows "Not allowed".
4. to 6. The laptop's feature module asks the display driver for a new screen at the iPad's size; Windows adds it and reports that the new screen is ready.
7. and 8. The WebRTC setup messages (offer, answer and fingerprint) travel over the messages connection; the video and the data channel then open directly between the two devices.
9. Pictures of the new screen stream to the iPad.
10. to 12. A touch on the iPad travels back on the data channel and becomes a mouse movement or click on the extra screen only.
13. to 15. Either side taps Stop, or the iPad app leaves the front. The stream closes and the laptop removes the virtual screen.

An Android tablet follows the same steps. On Linux with Wayland, step 5 is the system portal dialog. On macOS and Linux with X11, no virtual screen is added and the existing screen is mirrored (section 5).

## 8. Requirements

The requirements appendix gives these requirements as FR-72 to FR-78; the rules shared by all features are FR-1 to FR-3 and NFR-2.

1. The tool must allow a device to be used as an extra screen only between paired devices of the same user on the local network.
2. The tool must add a true extra screen at the screen device's size on Windows (with the tool's own signed display driver) and on Linux with Wayland (where the desktop offers a virtual monitor).
3. The tool must mirror the existing screen on macOS, and wherever an extra screen cannot be added, and must tell the user so.
4. The tool must tell the other devices whether a host device offers extend or mirror, and whether a device can show the stream.
5. The tool must send the screen as a WebRTC stream set up over the trusted messages connection, with no STUN or TURN server.
6. The tool must turn touches on the screen device into mouse input on the extra screen only.
7. The tool must remove the virtual screen when the session ends for any reason, so that no window is left on a screen nobody can see.
8. The tool must keep the screen device awake while it shows the extra screen, and must stop the session when the app leaves the front on iPhone and iPad.
9. The tool must let the user stop the extra screen from either device at any time.
10. The tool must ask for the operating system's permission where the system requires it (the Wayland portal dialog, Screen & System Audio Recording on macOS), and must not use private interfaces in the first release.
11. The tool must ask the user on both devices, every time, before an extra screen starts, and must never store the pictures.

## 9. Limits and points still to verify

No member of the team has a Mac or a Linux machine, so every macOS and Linux statement in this design comes from the documentation and has not been tested. The following points have also not yet been verified in a primary source:

- Windows: whether a user-mode display driver falls under the same signing rule as kernel-mode drivers, and the cost of an EV certificate.
- macOS: how Spacedesk's beta driver creates its screen (not described in its manual).
- Linux with X11: the `dummy` X driver, and building EVDI for each kernel.
- Linux with Wayland: which GNOME and KDE releases first offered the virtual monitor source.
- iPhone and iPad: the full page of `isIdleTimerDisabled` (only its summary was read).

## 10. Open points

| Point | Settled by |
|---|---|
| Windows driver: a test-signed driver on the team's own test PC is enough for Phase 2; a public release needs an EV certificate and Microsoft signing. If Phase 2 does not go beyond test signing, other Windows PCs get mirroring only. | Project lead |
| Test that the virtual monitor works on Ubuntu 24.04 with GNOME (started from a USB stick or a second partition) | Evaluation Plan |
| macOS tests need a Mac with Xcode | Project lead |
| Delay target of 100 ms and the test devices | Evaluation Plan |
| Names of the capability flags | Device Discovery, Pairing and Trust |
| WebRTC with no internet at all, the app size, and whether capture needs native code | Technology Choices |

## References

All pages were opened on 8 November 2026.

1. Microsoft, "Indirect display driver model overview": https://learn.microsoft.com/en-us/windows-hardware/drivers/display/indirect-display-driver-model-overview
2. Microsoft, driver signing: https://learn.microsoft.com/en-us/windows-hardware/drivers/install/kernel-mode-code-signing-policy--windows-vista-and-later- ; https://learn.microsoft.com/en-us/windows-hardware/drivers/dashboard/driver-signing-offerings ; https://learn.microsoft.com/en-us/windows-hardware/drivers/dashboard/code-signing-attestation ; https://learn.microsoft.com/en-us/windows-hardware/drivers/install/the-testsigning-boot-configuration-option
3. xdg-desktop-portal, ScreenCast portal (`VIRTUAL`, `persist_mode`): https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.ScreenCast.html
4. xdg-desktop-portal, RemoteDesktop portal (`NotifyPointerMotionAbsolute`): https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.RemoteDesktop.html
5. Portal source code (support for the virtual monitor): https://github.com/GNOME/xdg-desktop-portal-gnome (`src/screencast.c`), https://github.com/KDE/xdg-desktop-portal-kde (`src/screencast.cpp`), https://github.com/emersion/xdg-desktop-portal-wlr (`src/screencast/`)
6. KDE bug report, virtual monitor fixed at 1920 × 1080: https://bugs.kde.org/show_bug.cgi?id=512620
7. xrandr manual page: https://man.archlinux.org/man/xrandr.1
8. DisplayLink, EVDI: https://github.com/DisplayLink/evdi ; intel(4) manual page, `VirtualHeads`: https://manpages.debian.org/testing/xserver-xorg-video-intel/intel.4.en.html
9. Apple, `CGVirtualDisplay` is absent from Apple's documentation data (https://developer.apple.com/tutorials/data/documentation/coregraphics/cgvirtualdisplay.json returns 404, while the public class at https://developer.apple.com/tutorials/data/documentation/coregraphics/cgdisplaystream.json returns 200)
10. Apple Support, Sidecar (requirements, same Apple Account, wireless within 10 m or USB, Apple Pencil, wider touch from version 27): https://support.apple.com/102597
11. Apple, `isIdleTimerDisabled`: https://developer.apple.com/documentation/uikit/uiapplication/isidletimerdisabled
12. Android Developers, "Keep the screen on": https://developer.android.com/develop/background-work/background-tasks/awake/screen-on
13. Spacedesk manual: https://manual.spacedesk.net/ ; system requirements: https://manual.spacedesk.net/SystemRequirements1.html ; mouse, keyboard, touch and pen input: https://manual.spacedesk.net/MouseKeyboardTouchscreenAndPenIn.html
14. Project report: Review of Existing Systems (Spacedesk and Sidecar); Technology Choices, final version (WebRTC); Screen Sharing and Remote Control (stream, input and requirements); Security and Privacy (approval on both devices every time, no stored pictures); Evaluation Plan (devices available for testing).
