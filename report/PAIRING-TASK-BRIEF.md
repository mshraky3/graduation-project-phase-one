# Task brief: the pairing design (device discovery, pairing and trust)

For a strong cloud agent. Read this whole file first. Date written: 26 Oct 2026. Owner: Mahmoud (team leader, graduation project Phase 1).

## What you are doing
Produce the best possible **design document** for how devices in our system find each other, join (pair) and are trusted, and how a lost device is removed. Phase 1 of the graduation project is **documents only: no code**. This is the owner's own task (file code C1). Think deeply and carefully. A short, correct, honest design beats a long one. The document will be read by a professor and implemented later in Phase 2.

## The project in one paragraph
A "cross-device AI" system: the same app runs on Windows, macOS, Linux, Android, iPhone and iPad; the devices find each other on the home Wi-Fi and talk directly (no internet, no cloud server, no account); one laptop or PC also takes an AI role (similar-file and similar-photo search). Each device has files; the app shows them together, shares clipboard text, shares files, shares screens, and so on. Security matters because devices exchange personal files.

## Decisions already made (do not reopen; build on them)
- Technology (Ali's choices, `report/B6-technology-v0.md`, accepted by the owner): one Flutter app for all five systems; discovery with mDNS/Bonjour (library Bonsoir); TLS 1.3 in both directions with a self-signed key pair per device, pinned at pairing; SQLite per device with an encrypted vault; keys in the operating system's key store; small AI models run inside the app.
- Architecture version 2 (`report/B1-architecture-v2.md`, approved by the owner on 26 Oct): every device pairs with every other device, plus an optional one-tap "introduce my new device to my other devices" step that still asks on each device (nobody is trusted silently through a third device); every feature is OFF until the user turns it on (pairing only means "may connect"); iPhone and iPad answer only while the app is open, cannot take the AI role, cannot be controlled from another device; small photo previews may go to the AI laptop over the local network only, not kept, off switch per device; no cloud, no account, no internet needed for pairing.
- Scope settled by the professor: all five platforms; report in English.

## Files to read (all in this branch)
- `report/C1-outline.md`: the owner's outline: the questions this design must answer, how KDE Connect, Syncthing and LocalSend do it (some points marked not verified), a plan for four diagrams.
- `report/C1-v1-agent-draft.md`: a first draft written by another agent on 26 Oct. It is a starting point only. It was NOT reviewed by the owner or the team. Check it critically: find errors, weak reasoning and wrong claims, and fix them.
- `report/B1-architecture-v2.md`, `report/B1-architecture-v1.md`, `report/B6-technology-v0.md`, `report/C2-ai-node.md`, `report/C8-clipboard-sync.md`, `research/A5-existing-systems.md`: context from the team.

## What the document must contain (file `report/C1-device-discovery-pairing-trust.md`)
1. The design in plain steps: identity of a device; discovery; pairing (QR route and the no-camera route with a short code); trust and the trusted-device list; normal reconnection; adding a third and fourth device (pair with all, plus the optional introduce step); removing a lost or sold device; what happens when discovery fails (guest Wi-Fi, router isolation, hotspot).
2. At least **four sequence diagrams** (Mermaid in fenced code blocks, which GitHub renders): first pairing by QR; first pairing by code comparison; reconnection of a known device; removal of a lost device. Each diagram must match the text.
3. A **threat model table**: attacker, what they can do, how the design stops it, what is left. Include at least: a fake mDNS announcement; an attacker on the same Wi-Fi opening pairing sessions; someone photographing the QR code; a man-in-the-middle during the code-comparison route and the guessing/grinding attack on a short code (check whether the commitment step in the draft is correct and sufficient; compare with the short authentication string idea used in ZRTP/Bluetooth numeric comparison/Signal safety numbers; state exactly how many bits the code gives and what an attacker needs); a stolen unlocked device; a stolen private key; a removed device that is offline when removal happens; a device that reinstalls the app.
4. **Requirements** written as "The app must ..." for the team's requirements chapter (functional and non-functional, numbered, each testable).
5. **Comparison** with how KDE Connect, Syncthing and LocalSend pair, with sources, plus what we take and what we avoid (for example the KDE Connect pairing-without-human-check incident, CVE-2020-26164, if you can verify it).
6. **Boundary with the security design** (another team member's task): what this design decides and what it leaves to that design (per-feature permissions, encryption of stored data, lost-device policy).
7. **Open questions and unchecked facts** for specific team members (for example: Ali must confirm whether Dart/Flutter's TLS server can verify a self-signed client certificate; if not, the fallback in his document is a signed challenge inside the channel or a small native layer: say which you recommend and why).
8. A plain-language page for the owner (file `report/PAIRING-SUMMARY-FOR-OWNER.md`), one page, **no code names or board codes** (he does not know them): what the design is in six sentences, what you changed from the first draft and why, the three most important decisions he should check, and what is still unsure.

## Rules
- Every factual claim about a protocol, library or platform must be verified against a primary source (documentation, RFC, standard, the project's own repository) if you have web access in this environment, with the link. If you cannot verify it, write **[NOT CHECKED]**. Mark your own proposals **[PROPOSAL]**. Never invent a source, a number or a quotation. If you have no web access, say so at the top and rely on what you know with [NOT CHECKED] where needed.
- Honest numbers: when you give the strength of a code (bits), the number of guesses, a timeout or a limit, show the arithmetic.
- Keep the same style as the other files in `report/`: Markdown, English, plain headings, no code to implement (pseudocode of message order is fine).
- You have only this repository: no computer, Chrome, Gmail, databases or secrets. Never print or store secrets.
- Work on a NEW branch named `claude/pairing-design` created from `origin/pairing-design-input`. Commit with a message starting `C1:`. Push the branch and open a **pull request against `main`**. **Do NOT merge it.** Do not change any other file. Do not edit `data.json`, the board or anything outside `report/`.
- At the end, post a short plain-language report: the branch and pull request, what is solid, what is [NOT CHECKED], and the questions for the owner.
