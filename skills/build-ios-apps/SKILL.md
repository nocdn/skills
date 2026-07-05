---
name: build-ios-apps
description: "Build and iterate native iOS apps targeting iOS 26 and iOS 27. Use when creating, modifying, testing, debugging, running, installing, or verifying SwiftUI/UIKit apps with Xcode, xcodebuild, xcrun simctl/devicectl, npx serve-sim, Browser or Computer Use, simulator QA, dummy data, runtime logs, and connected iPhone deployment."
---

# Build iOS Apps

Use this skill for end-to-end native iOS work: research current APIs, build the app, verify it in Simulator through `serve-sim` and browser/computer tools, install it on a connected iPhone, and report the implementation and test evidence.

## Ground Rules

- Target iOS 26 and keep iOS 27 compatibility in mind. Set the deployment target to iOS 26 unless the user specifies otherwise. Use the newest installed Xcode/iOS SDK that supports iOS 26 or 27; if unavailable, report the installed SDKs and closest runnable target.
- Before using Apple APIs, third-party libraries, frameworks, or sample code, search current documentation. Prefer Apple Developer Documentation, WWDC session docs/videos/transcripts, Swift Evolution proposals, package README/API docs, release notes, and maintainer issue discussions.
- Search Apple Developer Forums and library forums when behavior is unclear. Treat dated forum answers as supporting evidence, not authoritative docs.
- For fast-moving platform features such as iOS 26/27, Liquid Glass, SwiftUI observation/navigation, App Intents, privacy permissions, signing, and device tooling, verify docs during the task even if you remember the API.
- Use native Apple frameworks by default: Swift, SwiftUI, UIKit when needed, SwiftData/Core Data, URLSession, App Intents, Observation, Combine, XCTest, and XCUITest.
- Build UI with native, unstyled controls unless the user asks for custom styling. Prefer standard `NavigationStack`, `TabView`, `List`, `Form`, `ToolbarItem`, `Button`, `Toggle`, `Picker`, sheets, alerts, SF Symbols, Dynamic Type, and accessibility labels. Avoid web-like custom controls and decorative themes unless requested.
- Add no dependency unless it materially reduces risk or complexity. When adding one, verify current docs and version compatibility.

## Workflow

1. Discover the project.
   - Read `README`, project instructions, `Package.swift`, `.xcodeproj`, `.xcworkspace`, `project.yml`, `Tuist`, `fastlane`, and relevant app/test files.
   - Identify bundle ID, schemes, deployment target, signing setup, simulator targets, connected devices, and existing test commands.
   - Use `xcodebuild -list`, `xcrun simctl list runtimes`, and `xcrun simctl list devices` when an Xcode project exists.

2. Research current API and tooling.
   - Search Apple docs and relevant library docs before implementation.
   - Search `serve-sim` docs and run its help if available:
     ```sh
     npx serve-sim --help
     ```
   - If `npx` is not on `PATH`, use the available package runner (`pnpm dlx`, `bunx`, etc.) and put Node on `PATH` if needed.
   - Check for new `serve-sim` commands/options that could improve testing; do not assume the CLI surface is frozen.

3. Implement the app.
   - Keep changes scoped and idiomatic to the existing architecture.
   - Prefer local dummy data factories or fixtures that exercise empty, small, typical, large, long-text, error, loading, permission-denied, offline, and edge-value states.
   - Keep state ownership clear and testable. Avoid hard-coding simulator-only behavior into production code.
   - Add compact runtime diagnostics before testing. Read [Runtime Logging And Device Testing](references/runtime-logging-and-device-testing.md) when adding or debugging logs.

4. Build and test from the command line.
   - Run formatter, lint, typecheck, or equivalent project checks when present.
   - Run unit tests and UI tests. For Xcode projects, use an existing project command or:
     ```sh
     xcodebuild test -scheme <scheme> -destination 'platform=iOS Simulator,name=<device>,OS=latest'
     ```
   - Capture build/test logs to a readable file such as `./tmp/ios-build.log` or `./.codex/ios-build.log`.

5. Verify interactively in Simulator.
   - Boot a current iOS 26/27 simulator when available; otherwise use the newest installed iOS Simulator and report the gap.
   - Install and launch with `xcrun simctl install booted <App.app>` and `xcrun simctl launch booted <bundle-id>` or project-native tooling.
   - Start `serve-sim` for browser control:
     ```sh
     npx serve-sim
     # preview defaults to http://localhost:3200
     ```
   - Use Browser Use, Computer Use, or the in-app browser if available to open `http://localhost:3200`, inspect the running app visually, click/tap through every feature, and capture screenshots when useful.
   - Use `serve-sim tap <x> <y>`, `type`, `button`, `rotate`, `memory-warning`, `permissions`, `ui`, and `camera` when they fit the feature. Prefer `tap` for simple taps. Use normalized `0..1` coordinates.
   - Test with the dummy data spread, rotation, Dynamic Type, color scheme, accessibility, and failure states where relevant. Fix issues and repeat build -> simulator -> `serve-sim` verification until passing.

6. Install and run on the connected iPhone after simulator verification.
   - Always attempt this after making changes and completing simulator testing.
   - Discover devices with `xcrun devicectl list devices` and/or `xcodebuild -showdestinations -scheme <scheme>`.
   - Build for a real device with the existing signing setup. Use project tooling or:
     ```sh
     xcodebuild -scheme <scheme> -destination 'platform=iOS,id=<device-id>' build
     ```
   - Install and launch with current `devicectl` commands when supported by the installed Xcode/device OS:
     ```sh
     xcrun devicectl device install app --device <device-id> <App.app>
     xcrun devicectl device process launch --device <device-id> <bundle-id>
     ```
   - If no physical iPhone is connected, trusted, paired, provisioned, or compatible, state the exact blocker and command-output summary. Do not imply hardware validation happened.
   - After install/launch, ensure runtime logs can be retrieved from the app or system logs for later debugging.

7. Debug physical-device feedback.
   - When the user reports a problem from the iPhone, read the app's persisted runtime logs and relevant device/system logs first.
   - Fix from evidence, then rerun: command-line tests -> simulator install -> `serve-sim` browser verification -> real-device rebuild/install/launch.
   - Keep the physical iPhone as the final validation target for every fix unless it is unavailable.

8. Report after verification.
   - Summarize what was implemented, how it works, and why key choices were made.
   - Include exact simulator/device/OS/Xcode used, whether iOS 26/27 SDKs were available, `serve-sim` URL/commands used, and any physical-device blocker.
   - Mention remaining risks or untested paths. Do not stop immediately after coding; stop after testing/reporting.

## Serve-sim Notes

`serve-sim` streams a booted Apple Simulator to a browser and forwards input over a local preview UI. The default preview is `http://localhost:3200`; helper stream ports normally start at `3100`.

Common current commands:

```sh
npx serve-sim [device]
npx serve-sim --detach -q
npx serve-sim --list -q
npx serve-sim --kill
npx serve-sim tap 0.5 0.5
npx serve-sim type "text"
npx serve-sim button home
npx serve-sim rotate landscape_left
npx serve-sim memory-warning
npx serve-sim permissions --help
npx serve-sim ui --help
npx serve-sim camera --help
```

Prefer JSON output (`-q`) for machine parsing. Stop helpers when done unless the user asks to keep the preview running.
