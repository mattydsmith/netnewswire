# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

### Building and Testing
- **Full build and test**: `./buildscripts/build_and_test.sh` - Builds both macOS and iOS targets and runs all tests
- **Quiet build and test**: `./buildscripts/quiet_build_and_test.sh` - Same as above with less verbose output
- **Manual Xcode builds**:
  - macOS: `xcodebuild -project NetNewsWire.xcodeproj -scheme NetNewsWire -destination "platform=macOS,arch=arm64" build`
  - iOS: `xcodebuild -project NetNewsWire.xcodeproj -scheme NetNewsWire-iOS -destination "platform=iOS Simulator,name=iPhone 17" build`

### Testing
- Run all tests: Use the `NetNewsWire.xctestplan` which includes tests from all modules
- Individual test runs follow same xcodebuild pattern with `test` action instead of `build`

### Setup
- First-time setup: Run `./setup.sh` to configure development environment and code signing
- Manual setup: Create `SharedXcodeSettings/DeveloperSettings.xcconfig` in parent directory

## Project Architecture

### High-Level Structure
NetNewsWire is a multi-platform RSS reader with separate targets for macOS and iOS, organized as a modular architecture with shared business logic.

### Key Modules (in /Modules)
- **RSCore**: Core utilities, extensions, and shared infrastructure
- **RSParser**: Feed parsing (RSS, Atom, JSON Feed, RSS-in-JSON)
- **RSWeb**: HTTP networking, downloading, caching, and web services
- **RSDatabase**: SQLite database abstraction layer using FMDB
- **Account**: Account management (Local, Feedbin, Feedly, NewsBlur, Reader API, CloudKit)
- **Articles**: Article and author data models
- **ArticlesDatabase**: Article storage and search functionality
- **SyncDatabase**: Cross-device synchronization state management
- **Secrets**: Secure credential and API key management

### Platform-Specific Code
- **Mac/**: macOS-specific UI (AppKit), preferences, main window management
- **iOS/**: iOS-specific UI (UIKit), settings, navigation
- **Shared/**: Cross-platform business logic, article rendering, smart feeds

### Key Architectural Patterns
- **Account System**: Pluggable account delegates for different sync services
- **Feed Management**: Hierarchical folder/feed organization with OPML import/export
- **Article Rendering**: Template-based HTML rendering with custom CSS themes
- **Smart Feeds**: Virtual feeds (Today, All Unread, Starred) implemented as PseudoFeed protocol
- **Timeline/Detail**: Classic three-pane interface (sidebar, timeline, detail)

### Extension Points
- Share extensions for both platforms
- Safari extension for feed subscription
- Widget support for iOS
- AppleScript support on macOS
- Intent extensions for Siri shortcuts

### Development Notes
- Uses Xcode project with Swift Package Manager for module dependencies
- Requires `xcbeautify` for formatted build output in scripts
- API keys are managed through buildscripts/updateSecrets.sh (runs during builds)
- Some features disabled in development builds due to private API keys
- Code signing configured through SharedXcodeSettings for development
- Documentation and technical notes are located in the `Technotes/` folder

## Code Formatting

Prefer idiomatic modern Swift.

Prefer `if let x` and `guard let x` over `if let x = x` and `guard let x = x`.

Don’t use `...` or `…` in Logger messages.

Guard statements should always put the return in a separate line.

Don’t do force unwrapping of optionals.

## Things to Know

Just because unit tests pass doesn’t mean a given bug is fixed. It may not have a test. It may not even be testable — it may require manual testing.

# Instructions for Adding visionOS Support to NetNewsWire 7.0

## Context
This is a fresh clone of NetNewsWire 7.0 from the official repository. The goal is to add visionOS (Apple Vision Pro) support so the iOS app runs on Vision Pro simulator in compatibility mode ("Designed for iPad").

## Important Background
- NetNewsWire 7.0 requires iOS 26+ and uses modern UICollectionView architecture
- The project uses File System Synchronized Groups which causes conflicts when adding visionOS platforms via xcconfig files
- **Solution:** Add visionOS support via Xcode's GUI instead of modifying xcconfig files
- I have a paid Apple Developer account (not Personal Team)

## Apple Developer Account Details
- **Team ID:** MP328K29CY
- **Organization Identifier:** `com.mttsmth`

## Task Breakdown

### 1. Verify Current Setup
First, check if `./setup.sh` has been run:
```bash
ls -la ../SharedXcodeSettings/DeveloperSettings.xcconfig
```

If file doesn't exist: Tell me to run `./setup.sh` first with my Apple Developer Team ID.

If file exists: Proceed to step 2.

### 2. Verify Baseline Build
Before adding visionOS, confirm the iPhone build works:

**Goal:** Ensure NetNewsWire-iOS target builds successfully for iPhone 17 Pro simulator.

**What to check:**
- Is the project file corrupted?
- Are there any pre-existing build errors?
- Does a basic clean + build succeed?

If build fails: Investigate the specific errors and help troubleshoot before proceeding.

If build succeeds: Proceed to step 3.

### 3. Add visionOS Support via Xcode GUI
**CRITICAL:** Do NOT modify xcconfig files. Use Xcode's GUI to avoid File System Synchronized Groups conflicts.

**Process:**
I will need to manually make these changes in Xcode. Your role is to:
- Provide clear, step-by-step instructions for what to do in Xcode
- Explain exactly where to find each setting
- Tell me what values to enter

**Targets to modify:**
- NetNewsWire-iOS (main app)
- NetNewsWire iOS Share Extension
- NetNewsWire iOS Intents Extension
- NetNewsWire iOS Widget Extension

**For each target, I need to:**
1. Add `xros` and `xrsimulator` to "Supported Platforms" build setting
2. Set "Supports XR Designed for iPhone iPad" to YES

Give me exact step-by-step instructions for doing this in Xcode.

### 4. Verify visionOS Build
After I make the GUI changes, help me verify:

**Steps:**
1. Clean build folder
2. Close and restart Xcode (to clear caches)
3. Build for Apple Vision Pro simulator
4. Troubleshoot any errors that occur

**Common issues to watch for:**
- "Multiple commands produce Info.plist/stylesheet.css/template.html"
  - Solution: Remove these files from "Copy Bundle Resources" build phase
- Code signing errors with paid account
- Missing visionOS simulator runtime

### 5. Test on Vision Pro Simulator
Once build succeeds, guide me through:
- Running the app on Vision Pro simulator
- What to test (navigation, feeds, articles, settings)
- Known issues/limitations to be aware of

## Success Criteria
- ✅ iPhone build succeeds
- ✅ Vision Pro build succeeds
- ✅ App launches on Vision Pro simulator
- ✅ Three-pane layout works (sidebar, timeline, detail)
- ✅ Can navigate feeds and read articles
- ✅ No crashes during basic usage

## Key Constraints
- Do NOT modify xcconfig files - causes File System Synchronized Groups conflicts
- Use Xcode GUI only for platform configuration
- Verify each step before proceeding to next
- If errors occur - stop and troubleshoot before continuing

## What I Need From You
- **Clear Xcode GUI instructions** - step-by-step, specific menu locations
- **Proactive error checking** - verify each step succeeded before proceeding
- **Troubleshooting help** - if builds fail, help diagnose and fix
- **Testing guidance** - what to verify on Vision Pro simulator

## Additional Context
- NetNewsWire 7.0 is already well-suited for visionOS (modern UICollectionView, UIKit-based)
- This is "compatibility mode" - app runs as "Designed for iPad" on Vision Pro
- No code changes needed - just build configuration
- With paid Apple account, code signing should be straightforward
