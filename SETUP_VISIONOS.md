# NetNewsWire 7.0 - visionOS Setup Guide

This guide walks through setting up a fresh clone of NetNewsWire 7.0 with visionOS support using a paid Apple Developer account.

## Prerequisites

- macOS with Xcode 16+ installed
- Paid Apple Developer account
- visionOS simulator installed (Xcode → Settings → Platforms)
- Git installed

## Step 1: Fresh Clone

```bash
# Navigate to your development directory
cd ~/Documents/Dev

# Backup old version if it exists
mv netnewswire netnewswire-backup-$(date +%Y%m%d)

# Clone fresh NetNewsWire 7.0
git clone https://github.com/Ranchero-Software/NetNewsWire.git netnewswire
cd netnewswire
```

## Step 2: Run Setup Script

NetNewsWire requires initial configuration before building.

```bash
./setup.sh
```

**When prompted:**

1. **Developer Team ID:**
   - Find in: Xcode → Settings → Accounts → [Your Apple ID] → Team ID
   - Or run: `security find-identity -v -p codesigning`
   - For paid account, use your **Organization Team ID** (format: ABC123XYZ4)
   - NOT the Personal Team ID

2. **Organization Identifier:**
   - Use reverse-domain format: `com.yourname` or `com.mttsmth`
   - This creates bundle IDs like: `com.mttsmth.NetNewsWire.iOS-DEBUG`

**The script creates:** `../SharedXcodeSettings/DeveloperSettings.xcconfig`

## Step 3: Verify iPhone Build Works

Before adding visionOS support, confirm the base project builds:

```bash
# Open the project
open NetNewsWire.xcodeproj
```

**In Xcode:**

1. Select **NetNewsWire-iOS** scheme (top toolbar)
2. Select **iPhone 17 Pro** destination (next to scheme)
3. **Product → Clean Build Folder** (Cmd+Shift+K)
4. **Product → Build** (Cmd+B)

**Expected result:** Build succeeds (code signing warnings are normal)

If build fails, STOP and troubleshoot before proceeding.

## Step 4: Add visionOS Support (GUI Method)

**Why GUI instead of xcconfig?** NetNewsWire 7.0 uses File System Synchronized Groups which conflict with adding visionOS platforms via xcconfig files. The GUI method avoids this.

### 4a. Add Platforms to Main App Target

1. **Select NetNewsWire-iOS target** (left sidebar under TARGETS)
2. **Build Settings tab** (top)
3. **Search for:** `Supported Platforms`
4. **Click on the value** for "Supported Platforms"
5. **Add:** `xros` and `xrsimulator` to the list
   - Should read: `iphoneos iphonesimulator xros xrsimulator`
6. **Search for:** `Supports XR`
7. **Set:** "Supports XR Designed for iPhone iPad" to **YES**

### 4b. Add Platforms to Extension Targets

Repeat the same steps for each extension target:

- **NetNewsWire iOS Share Extension**
- **NetNewsWire iOS Intents Extension**
- **NetNewsWire iOS Widget Extension**

For each:
1. Select the target
2. Build Settings → Search "Supported Platforms"
3. Add `xros xrsimulator`
4. Search "Supports XR" → Set to YES

## Step 5: Build for Vision Pro

**In Xcode:**

1. **Product → Clean Build Folder** (Cmd+Shift+K)
2. **Quit and restart Xcode** (to clear any caches)
3. **Reopen:** `open NetNewsWire.xcodeproj`
4. Select **NetNewsWire-iOS** scheme
5. Select **Apple Vision Pro** destination
6. **Product → Build** (Cmd+B)

**Expected result:** Build succeeds!

## Step 6: Run on Vision Pro Simulator

**In Xcode:**

1. Ensure **Apple Vision Pro** is selected as destination
2. **Product → Run** (Cmd+R)
3. The Vision Pro simulator launches
4. NetNewsWire appears as "Designed for iPad" app

**What to test:**
- Three-pane layout (sidebar, timeline, detail)
- Navigation between feeds
- Article rendering in web view
- Swipe gestures
- Search functionality
- Settings panel

## Troubleshooting

### Build Error: "Multiple commands produce Info.plist"

If you see this error even after using the GUI method:

1. Select **NetNewsWire-iOS** target
2. **Build Phases** tab
3. Expand **"Copy Bundle Resources"**
4. Remove these files if present:
   - `Info.plist`
   - `stylesheet.css`
   - `template.html`
5. Clean and rebuild

**Why:** These files are handled by INFOPLIST_FILE build setting and File System Synchronized Groups.

### Vision Pro Simulator Not Available

1. **Xcode → Settings → Platforms**
2. Download **visionOS** platform/simulator
3. Restart Xcode after download completes

### Code Signing Errors

With a paid account, these should be minimal. If you see them:

1. **Select target → Signing & Capabilities**
2. Verify **Team** dropdown shows your paid team (not Personal Team)
3. Check **"Automatically manage signing"** is enabled
4. Bundle identifier should be: `com.yourorg.NetNewsWire.iOS-DEBUG`

### Build Succeeds but App Crashes on Launch

Check Console.app for crash logs. Common issues:
- Missing entitlements
- Framework linking errors
- Resource loading failures

## Next Steps

### Create a Feature Branch

```bash
git checkout -b feature/visionos-support
git add .
git commit -m "Add visionOS platform support via Xcode GUI"
git push origin feature/visionos-support
```

### Test Thoroughly

- Test all major features on Vision Pro simulator
- Compare behavior with iPhone/iPad versions
- Check for any visionOS-specific UI issues
- Test with different window sizes

### Optional: Add visionOS-Specific Enhancements

Once basic compatibility works, consider adding:

- **Ornaments** for navigation controls
- **Hover effects** on interactive elements
- **Improved depth/materials** using visionOS APIs
- **Spatial window positioning**

These would require code changes but would make the app feel more "native" to Vision Pro.

## Architecture Notes

### Why This Works

NetNewsWire 7.0 is well-suited for visionOS:

- ✅ **UICollectionView** (modern, visionOS-compatible)
- ✅ **Swift Structured Concurrency** (modern async patterns)
- ✅ **UIKit** (fully supported on visionOS 26)
- ✅ **Three-pane iPad layout** (perfect for spatial computing)
- ✅ **iOS 26+ requirement** (aligns with visionOS 26)

### Compatibility Mode

This setup runs NetNewsWire as a "Designed for iPad" app on Vision Pro:

- Uses existing UIKit iOS codebase
- No separate visionOS target needed
- Three-pane layout translates naturally to spatial computing
- All frameworks (UIKit, WebKit, BackgroundTasks) are compatible

### Known Limitations

- UICollectionView prefetching doesn't work on visionOS (minor issue)
- No native visionOS UI elements (ornaments, etc.)
- Runs in compatibility mode, not full native visionOS SDK

## Reference Information

**Your Apple Developer Account:**
- Team ID: [Your paid account Team ID]
- Organization: `com.mttsmth` (or as configured)

**Key Files Modified:**
- Xcode project file (`NetNewsWire.xcodeproj/project.pbxproj`)
  - Modified via GUI, not directly edited
  - Contains platform configuration changes

**Official NetNewsWire Resources:**
- Main repo: https://github.com/Ranchero-Software/NetNewsWire
- Version 7.0 announcement: https://netnewswire.blog/2026/02/06/netnewswire-for-ios.html
- Requirements: iOS 26+, modern UICollectionView architecture

## Success Criteria

✅ iPhone 17 Pro build succeeds
✅ Vision Pro simulator build succeeds
✅ App launches on Vision Pro simulator
✅ All three panes visible and functional
✅ Can navigate feeds and read articles
✅ No crashes during normal usage

---

**Last Updated:** 2026-02-07
**NetNewsWire Version:** 7.0 (iOS 26+)
**visionOS Support:** Compatibility mode via "Designed for iPad"
