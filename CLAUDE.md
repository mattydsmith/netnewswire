# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

### Building and Testing
- **Full build and test**: `./buildscripts/build_and_test.sh` - Builds both macOS and iOS targets and runs all tests
- **Quiet build and test**: `./buildscripts/quiet_build_and_test.sh` - Same as above with less verbose output
- **Manual Xcode builds**:
  - macOS: `xcodebuild -project NetNewsWire.xcodeproj -scheme NetNewsWire -destination "platform=macOS,arch=arm64" build`
  - iOS: `xcodebuild -project NetNewsWire.xcodeproj -scheme NetNewsWire-iOS -destination "platform=iOS Simulator,name=iPhone 17" build`
  - visionOS: `xcodebuild -project NetNewsWire.xcodeproj -scheme NetNewsWire-iOS -destination "platform=visionOS Simulator,name=Apple Vision Pro" build`

### Testing
- Run all tests: Use the `NetNewsWire.xctestplan` which includes tests from all modules
- Individual test runs follow same xcodebuild pattern with `test` action instead of `build`

### Setup

**IMPORTANT: You must run setup before building for the first time!**

- **First-time setup**: Run `./setup.sh` to configure development environment and code signing
  - You'll need your Apple Developer Team ID (found in Xcode → Settings → Accounts, or via `security find-identity -v -p codesigning`)
  - Enter an organization identifier in reverse-domain format (e.g., `com.yourname`)
  - For Personal Team (free Apple ID), any identifier works for simulator builds
- **Manual setup**: Create `SharedXcodeSettings/DeveloperSettings.xcconfig` in parent directory

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
- **visionOS**: Runs the iOS target in compatibility mode ("Designed for iPad" on Apple Vision Pro)
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

### Enabling CloudKit/iCloud in Development Builds

CloudKit syncing is disabled in development builds but can be enabled with proper Apple Developer account setup. Unlike Feedly/Inoreader which require third-party API keys, CloudKit only needs entitlements and Apple Developer Portal configuration.

**TODO: Steps to enable CloudKit in development builds**

- [ ] **Configure Apple Developer Portal**
  - Go to [developer.apple.com](https://developer.apple.com) → Certificates, Identifiers & Profiles
  - Select your App ID (or create one)
  - Enable CloudKit capability
  - Create/select CloudKit container: `iCloud.YOUR-ORG-ID.NetNewsWire-Dev`
  - Note: Can use different container ID than production to avoid conflicts

- [ ] **Update Development Entitlements**
  - Edit `iOS/Resources/NetNewsWire-dev.entitlements`
  - Add CloudKit entitlements (see `NetNewsWire.entitlements` for reference):
    - `com.apple.developer.icloud-container-environment` = `Development`
    - `com.apple.developer.icloud-container-identifiers` = `iCloud.YOUR-ORG-ID.NetNewsWire-Dev`
    - `com.apple.developer.icloud-services` = `CloudKit`
    - `aps-environment` = `development`

- [ ] **Remove UI Blocking Code**
  - `iOS/Settings/AddAccountViewController.swift:133` - Remove or modify `isDeveloperBuild` check for CloudKit
  - `Mac/Preferences/Accounts/AddAccountsView.swift:92` - Remove or modify `isDeveloperBuild` check for iCloud section

- [ ] **Update DeveloperSettings.xcconfig**
  - Ensure `DEVELOPMENT_TEAM` is set to your Apple Team ID
  - Verify `ORGANIZATION_IDENTIFIER` matches your CloudKit container prefix
  - CloudKit requires valid provisioning profile with iCloud capability

- [ ] **Test CloudKit Functionality**
  - Build and run with development entitlements
  - Add iCloud account in Settings/Preferences
  - Verify feed sync works across devices
  - Check CloudKit Dashboard for data (developer.apple.com → CloudKit Console)

**Key Differences:**
- **Feedly/Inoreader**: Require OAuth secrets in `SecretKey.swift.gyb` (environment variables)
- **CloudKit**: Only requires entitlements configuration and Apple Developer account
- **Reader View**: Requires Mercury API keys in `SecretKey.swift.gyb`

## Troubleshooting

### "Multiple commands produce" Build Errors

If you encounter errors like:
```
Multiple commands produce '/path/to/NetNewsWire.app/Info.plist'
Multiple commands produce '/path/to/NetNewsWire.app/stylesheet.css'
Multiple commands produce '/path/to/NetNewsWire.app/template.html'
```

**Solution:**
1. In Xcode, select the **NetNewsWire-iOS** target
2. Go to **Build Phases** tab
3. Expand **"Copy Bundle Resources"** section
4. Remove these files if present: `Info.plist`, `stylesheet.css`, `template.html`
5. Clean Build Folder (Cmd+Shift+K) and rebuild

**Why this happens:** These files are already handled by:
- `Info.plist` → Managed by the `INFOPLIST_FILE` build setting
- `stylesheet.css` and `template.html` → Auto-included by File System Synchronized Groups

### Code Signing Warnings on Personal Team

Provisioning profile warnings are normal when using a free Apple Developer account (Personal Team) for simulator builds. These warnings don't prevent simulator builds from succeeding - the app will run fine on simulators despite the warnings.

### First Build After Clone

1. **Run setup script first:** `./setup.sh` (required!)
2. **Clean derived data:** Delete `~/Library/Developer/Xcode/DerivedData/*` if you encounter persistent build issues
3. **Restart Xcode** after running setup to ensure configuration is loaded

## visionOS Support

### Current Status
visionOS support via compatibility mode has been explored but encounters build system conflicts with File System Synchronized Groups when adding `xros xrsimulator` to `SUPPORTED_PLATFORMS` in xcconfig files.

### Alternative Approaches
To run NetNewsWire on Vision Pro simulator:
1. **Add platforms via Xcode GUI** instead of xcconfig files:
   - Select NetNewsWire-iOS target → Build Settings
   - Search for "Supported Platforms"
   - Manually add `xros` and `xrsimulator`
   - Set "Supports XR Designed for iPhone iPad" to YES
2. This may avoid conflicts with File System Synchronized Groups

### Architecture Notes (if visionOS is enabled)
- Would use existing iOS codebase with UIKit (no separate visionOS target)
- Would run as "Designed for iPad" app in visionOS compatibility mode
- The three-pane split view interface translates well to spatial computing
- All major frameworks (UIKit, WebKit, BackgroundTasks) are visionOS-compatible

## Code Formatting

Prefer idiomatic modern Swift.

Prefer `if let x` and `guard let x` over `if let x = x` and `guard let x = x`.

Don’t use `...` or `…` in Logger messages.

Guard statements should always put the return in a separate line.

Don’t do force unwrapping of optionals.

## Things to Know

Just because unit tests pass doesn’t mean a given bug is fixed. It may not have a test. It may not even be testable — it may require manual testing.
