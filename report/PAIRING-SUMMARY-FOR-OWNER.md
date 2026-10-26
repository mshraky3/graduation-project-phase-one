# How devices find, join and trust each other: one page for Mahmoud

Written 26 Oct 2026. Full design: `C1-device-discovery-pairing-trust.md`. Nothing is tested; every number is a design choice or a calculation.

## The design in six sentences
1. Each device makes its own secret key at first start; its identity is a fingerprint of that key, never its name.
2. Devices announce themselves on the home Wi-Fi with almost nothing, and the announcement is only a hint: trust comes from checking the key after connecting.
3. To join, the user opens a two-minute "Add a device" window on one device, then scans its QR code with the other, or, with no camera, compares a 6-digit number on both screens; the user confirms on both.
4. Pairing only means "may connect"; every feature stays off until the user switches it on.
5. A third or fourth device pairs with every other device; an optional "introduce" step saves typing but still asks the user, with a number to compare, on each device.
6. A lost device is removed from any other device; the removal spreads by itself, and a device that was off hears about it when it next meets another of your devices.

## What I changed from the first draft, and why
- **The 6-digit number was not safe.** Someone controlling the network could keep making fake keys until both screens matched (about 100 seconds on one laptop, by my estimate). The new message order (commit, then a fresh random value from each side, then reveal) leaves one blind guess per attempt: 1 in 1,000,000, and the user sees the mismatch.
- **The QR route now also has a number to compare.** Otherwise someone who photographed the QR code could connect first and the user would just tap "Yes".
- **A device proves its key inside the connection**, because Flutter's receiving side cannot accept an unknown certificate. This is the fallback Ali's document already names; Ali has to test it.
- **Removal is permanent for that key.** To add the device again it must be reset to get a new key.
- **A claim in the outline was wrong.** The KDE Connect security case was mainly a denial-of-service problem; the pairing weakness was a fakeable name. I could not confirm "pairing without a human check", so the design does not say that.

## Three decisions for you to check
1. **Two minutes, 6 digits, 3 tries:** is that the right balance? (8 digits would be 100 times harder to fake.)
2. **Introducing a device is not "one tap":** it needs a tap and a number check on each device, so a stolen unlocked device cannot add strangers to your other devices.
3. **Removal cannot reach a device that is off:** until it meets another device of yours, it still trusts the lost one. Only a server would fix this, and we decided against a server.

## What is still unsure
- Whether Flutter's connection and key code works as needed on all five systems: Ali must test (section 9 of the full design).
- I could not open the standards texts or the original advisory pages here; those points are marked "not checked". Facts about KDE Connect, Syncthing, LocalSend, Flutter's source code and Apple's rules were read from the projects' own files.
- Nobody has tested whether real users really compare the numbers.
- The boundary with Ali's security design (feature permissions, encrypted storage, what to do about a lost device's copies) needs his agreement.
