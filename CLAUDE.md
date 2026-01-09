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
- **visionOS**: Runs iOS target in compatibility mode ("Designed for iPad") - full functionality with minimal adaptation
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

## Platform Support

### Apple Vision Pro
NetNewsWire supports visionOS via "Designed for iPad" compatibility mode. The iOS target runs in a floating window on Vision Pro with full functionality.

**Configuration**: Vision Pro support is enabled via `SUPPORTS_XR_DESIGNED_FOR_IPHONE_IPAD = YES` in `xcconfig/NetNewsWire_iOSapp_target.xcconfig`.

**Testing**: Run on Vision Pro simulator using the visionOS destination (see build commands above).

**User Experience**:
- Three-pane interface works seamlessly in spatial environment
- Appears as resizable floating window in shared space
- All features (feed management, article reading, sync, settings) work without modification
- UIKit-based interface adapts well to visionOS interaction model

**Future Considerations**: A native visionOS target would require SwiftUI rewrite but is not necessary for production-quality experience.

## Enabling Optional Features

### iCloud Sync and Reader View

By default, iCloud sync and Reader View are disabled in development builds. These features require specific configuration:

#### iCloud Sync
iCloud sync is disabled when `DEVELOPER_ENTITLEMENTS = -dev` in `SharedXcodeSettings/DeveloperSettings.xcconfig`.

**To Enable**:
1. Edit `SharedXcodeSettings/DeveloperSettings.xcconfig`
2. Change `DEVELOPER_ENTITLEMENTS = -dev` to `DEVELOPER_ENTITLEMENTS =` (empty)
3. Rebuild the app

The iCloud account option will appear in Settings > Add Account > iCloud section.

#### Reader View
Reader View (article extraction) requires Mercury API credentials and depends on the developer build flag being disabled.

**Requirements**:
1. Mercury API Client ID and Secret from Feedbin's extraction service (https://extract.feedbin.com/)
2. `DEVELOPER_ENTITLEMENTS` must be empty (not `-dev`)

**To Enable**:
1. Obtain Mercury API credentials
2. Set environment variables:
   ```bash
   export MERCURY_CLIENT_ID="your-client-id"
   export MERCURY_CLIENT_SECRET="your-client-secret"
   ```
3. Regenerate secrets: `./buildscripts/updateSecrets.sh`
4. Remove `-dev` from `DEVELOPER_ENTITLEMENTS` in DeveloperSettings.xcconfig
5. Rebuild the app

**Implementation Details**:
- Secrets are stored obfuscated in `Modules/Secrets/Sources/Secrets/SecretKey.swift` (generated from `.gyb` template)
- `AppDefaults.isDeveloperBuild` checks for `DeveloperEntitlements = "-dev"` in Info.plist
- When `isDeveloperBuild = true`, features are disabled in UI (AddAccountViewController.swift:133, ArticleViewController.swift:224)
- ArticleExtractor uses Feedbin's extraction service at `https://extract.feedbin.com/parser`

## Known Build Issues and Fixes

### Theme Bundle File Conflicts (Fixed)
**Issue**: iOS builds would fail with "Multiple commands produce" errors for `Info.plist`, `stylesheet.css`, and `template.html` files from theme bundles.

**Cause**: Xcode's file system synchronization was flattening `.nnwtheme` bundle directories and copying individual files to the same destination, causing conflicts.

**Solution**: The Shared folder's `PBXFileSystemSynchronizedRootGroup` configuration in `project.pbxproj` now explicitly lists theme folders in `explicitFolders`:
```
explicitFolders = ("Resources/Appanoose.nnwtheme", "Resources/Hyperlegible.nnwtheme", "Resources/NewsFax.nnwtheme", "Resources/Promenade.nnwtheme", "Resources/Sepia.nnwtheme")
```

This ensures Xcode treats `.nnwtheme` directories as bundle resources rather than flattening their contents.

**Location**: `NetNewsWire.xcodeproj/project.pbxproj` line 505

## Code Formatting

Prefer idiomatic modern Swift.

Prefer `if let x` and `guard let x` over `if let x = x` and `guard let x = x`.

Don’t use `...` or `…` in Logger messages.

Guard statements should always put the return in a separate line.

Don’t do force unwrapping of optionals.

## Things to Know

Just because unit tests pass doesn’t mean a given bug is fixed. It may not have a test. It may not even be testable — it may require manual testing.
