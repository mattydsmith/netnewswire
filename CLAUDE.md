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

## visionOS Support

### Configuration
visionOS support is enabled in `xcconfig/NetNewsWire_iOSapp_target.xcconfig`:
- `SUPPORTED_PLATFORMS` includes `xros xrsimulator`
- `SUPPORTS_XR_DESIGNED_FOR_IPHONE_IPAD = YES`

This allows the iOS target (NetNewsWire-iOS) to run on Apple Vision Pro in compatibility mode.

### Architecture Notes
- Uses the existing iOS codebase with UIKit (no separate visionOS target)
- Runs as "Designed for iPad" app in visionOS compatibility mode
- The three-pane split view interface (sidebar, timeline, detail) translates well to visionOS
- UIDevice.current.userInterfaceIdiom returns `.pad` on visionOS
- All major frameworks used (UIKit, WebKit, BackgroundTasks, etc.) are visionOS-compatible

### Testing Considerations
When testing on visionOS, pay attention to:
- UISplitViewController behavior and window management
- Gesture interactions (swipe actions, drag-and-drop)
- WKWebView article rendering and tap zones
- Background task scheduling
- Navigation bar and toolbar appearance
- Share extension functionality

### Known Limitations
- Currently uses compatibility mode rather than native visionOS SDK
- No visionOS-specific optimizations (ornaments, spatial features, etc.)
- Extensions (share extension, widgets) may need separate visionOS configuration

## Code Formatting

Prefer idiomatic modern Swift.

Prefer `if let x` and `guard let x` over `if let x = x` and `guard let x = x`.

Don’t use `...` or `…` in Logger messages.

Guard statements should always put the return in a separate line.

Don’t do force unwrapping of optionals.

## Things to Know

Just because unit tests pass doesn’t mean a given bug is fixed. It may not have a test. It may not even be testable — it may require manual testing.
