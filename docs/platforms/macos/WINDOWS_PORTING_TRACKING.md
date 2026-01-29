# macOS Change Tracking for Windows Porting Parity

> **Purpose:** Track all macOS app changes with Windows compatibility considerations
> **Status:** Windows port on hold, maintaining feature parity awareness
> **Last Updated:** 2026-01-28

---

## Overview

The macOS app (`apps/macos/`) and the future Windows app should maintain **feature parity**. This document tracks changes to the macOS app and notes what needs to be considered for the future Windows port.

## Strategy: Forked Codebases

```
apps/macos/          # macOS app (Swift/SwiftUI)
    │
    ├── Document changes here
    │
    └── When Windows development begins:
        │
        ├── Fork to apps/windows/
        │   # Use this document as reference
        │   # Implement features tracked below
        │
        └── Maintain feature parity going forward
```

**Key Principles:**
1. **Separate codebases** - macOS and Windows will be forked apps, not shared code
2. **Feature parity** - Both platforms should have equivalent functionality
3. **Document first** - All macOS changes should document Windows implications
4. **API mapping** - Maintain awareness of macOS → Windows API equivalents

---

## Change Tracking Template

When adding or modifying features in the macOS app, use this template:

```markdown
### Feature: [Feature Name]

**Date Added:** YYYY-MM-DD
**macOS File:** apps/macos/Sources/Moltbot/[File].swift
**Priority for Windows:** P0 / P1 / P2 / Not Applicable

**Description:**
[Brief description of what the feature does]

**macOS Implementation:**
- APIs used: [List macOS APIs]
- Dependencies: [External dependencies]
- Platform-specific: [Yes/No]

**Windows Considerations:**
- Windows API equivalent: [API name]
- Implementation complexity: [Low/Medium/High]
- Estimated effort: [X days]
- Alternative approaches: [Any alternatives]

**Code Example:**
[Relevant code snippet showing platform-specific parts]

**Testing:**
- How to test on macOS: [Test steps]
- Windows test considerations: [What to test on Windows]

**See Also:**
- [Related documentation]
- [Similar features]
```

---

## Tracked Changes

### Core Functionality

#### Menu Bar System Tray

**macOS Implementation:**
- File: `Sources/Moltbot/MenuBar.swift`
- API: `MenuBarExtra` (SwiftUI)
- Priority: **P0** (Required)

**Windows Equivalent:**
- API: `NotifyIcon` (Windows Forms) or `TaskbarIcon` (WPF)
- Effort: 3-5 days
- Notes: Must support left-click to show chat, right-click for context menu

---

#### Gateway WebSocket Client

**macOS Implementation:**
- File: `Sources/Moltbot/GatewayConnectionController.swift`
- API: `Foundation.URLSession` with WebSocket
- Priority: **P0** (Required)

**Windows Equivalent:**
- API: `ClientWebSocket` (System.Net.WebSockets)
- Effort: 2-3 days
- Notes: Same protocol, just different WebSocket library

---

#### Service/Agent Auto-Start

**macOS Implementation:**
- File: `Sources/Moltbot/LaunchAgentManager.swift`
- API: `launchctl` commands
- Priority: **P0** (Required)

**Windows Equivalent:**
- API: Task Scheduler or Registry Run key
- Effort: 5-7 days
- Notes:
  - Registry: `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`
  - Task Scheduler: More robust, supports triggers

---

### UI Components

#### Canvas Panel (Browser View)

**macOS Implementation:**
- File: `Sources/Moltbot/CanvasWindowController.swift`
- API: `WKWebView` (WebKit)
- Priority: **P0** (Required)

**Windows Equivalent:**
- API: `WebView2` (Microsoft.Web.WebView2)
- Effort: 5-7 days
- Notes:
  - WebView2 uses Edge Chromium engine
  - Must be bundled or rely on system installation
  - Fallback: IE WebBrowser control (not recommended)

---

#### Exec Approval Dialogs

**macOS Implementation:**
- File: `Sources/Moltbot/ExecApprovals/`
- API: `NSAlert`, `NSPanel`
- Priority: **P1** (Important)

**Windows Equivalent:**
- API: `Window` (WPF) or `Form` (WinForms)
- Effort: 2-3 days
- Notes: Same UX, just different windowing system

---

#### Permission Manager

**macOS Implementation:**
- File: `Sources/Moltbot/PermissionManager.swift`
- API: TCC (Transparency, Consent, and Control)
- Priority: **P1** (Important)

**Windows Equivalent:**
- API: No direct equivalent (Windows permission model is different)
- Effort: 3-4 days
- Notes:
  - Windows doesn't have TCC
  - Check permissions at runtime, catch exceptions
  - Provide user guidance for enabling permissions

---

### Hardware Access

#### Camera Capture

**macOS Implementation:**
- File: `Sources/Moltbot/CameraCaptureService.swift`
- API: `AVFoundation` (AVCaptureDevice)
- Priority: **P1** (Important)

**Windows Equivalent:**
- API: `MediaCapture` (Windows.Media.Capture)
- Effort: 4-6 days
- Notes:
  - Must request camera permission
  - Handle multiple cameras
  - Privacy indicator required

---

#### Screen Recording

**macOS Implementation:**
- File: `Sources/Moltbot/ScreenRecordService.swift`
- API: `CGDisplayStream` or `ScreenCaptureKit`
- Priority: **P1** (Important)

**Windows Equivalent:**
- API: `GraphicsCapture` (Windows.Graphics.Capture)
- Effort: 4-6 days
- Notes:
  - Requires Windows 11 (or Windows 10 with latest updates)
  - Must handle window vs screen vs monitor selection

---

#### Microphone Access

**macOS Implementation:**
- API: `AVAudioSession` (AVFoundation)
- Priority: **P2** (Nice to have)

**Windows Equivalent:**
- API: `AudioCapture` (Windows.Media.Capture)
- Effort: 3-4 days
- Notes: Similar to camera, just audio-only

---

### System Integration

#### Bonjour/mDNS Discovery

**macOS Implementation:**
- File: `Sources/Moltbot/GatewayDiscovery/`
- API: `NWBrowser` (Network framework)
- Priority: **P1** (Important)

**Windows Equivalent:**
- API: Bonjour SDK for Windows or Zeroconf
- Effort: 5-7 days
- Notes:
  - Bonjour SDK for Windows is available but less common
  - Alternative: Implement raw mDNS using sockets
  - Must test interoperability with macOS/Linux gateways

---

#### Browser Detection

**macOS Implementation:**
- File: `Sources/Moltbot/BrowserDetection.swift`
- API: `NSWorkspace`, `LaunchServices`, `osascript`
- Priority: **P2** (Nice to have)

**Windows Equivalent:**
- API: Registry queries, `AssocQueryString`
- Effort: 2-3 days
- Notes:
  - Check `HKEY_CLASSES_ROOT\http\shell\open\command`
  - Fallback to common installation paths

---

#### Notifications

**macOS Implementation:**
- API: `NSUserNotificationCenter`
- Priority: **P2** (Nice to have)

**Windows Equivalent:**
- API: `ToastNotificationManager` (Windows.UI.Notifications)
- Effort: 2-3 days
- Notes:
  - Requires app shortcut in Start Menu
  - Must handle action buttons in toasts

---

#### Deep Linking (Custom URL Scheme)

**macOS Implementation:**
- API: `Info.plist` (CFBundleURLTypes)
- Priority: **P1** (Important)

**Windows Equivalent:**
- API: Registry (HKEY_CLASSES_ROOT\moltbot)
- Effort: 1-2 days
- Notes:
  - Register URL protocol handler
  - Handle app activation with URL

---

### Platform-Specific Features

#### Location Services

**macOS Implementation:**
- API: `CoreLocation` (CLLocationManager)
- Priority: **P2** (Nice to have)

**Windows Equivalent:**
- API: `Geolocator` (Windows.Devices.Geolocation)
- Effort: 3-4 days
- Notes:
  - Requires user permission
  - Different privacy model on Windows

**Status:** Consider omitting from initial Windows release

---

#### Speech Recognition

**macOS Implementation:**
- API: `SFSpeechRecognizer` (Speech framework)
- Priority: **P2** (Nice to have)

**Windows Equivalent:**
- API: `SpeechRecognizer` (Windows.Media.SpeechRecognition)
- Effort: 5-7 days
- Notes:
  - Requires Windows privacy policy acceptance
  - Different language models available

**Status:** Consider omitting from initial Windows release

---

#### AppleScript Execution

**macOS Implementation:**
- API: `NSUserScriptTask` or `osascript` CLI
- Priority: **Not Applicable** (macOS-only)

**Windows Equivalent:**
- API: PowerShell execution
- Effort: N/A (different feature)
- Notes: This is fundamentally macOS-specific

**Status:** Replace with Windows-specific automation (PowerShell scripts)

---

## Windows API Reference Mapping

### Common macOS → Windows API Mappings

| Functionality | macOS API | Windows API | Notes |
|---------------|-----------|-------------|-------|
| **System Tray** | `MenuBarExtra` | `NotifyIcon` | WinForms or `TaskbarIcon` WPF |
| **Web Browser** | `WKWebView` | `WebView2` | Edge Chromium-based |
| **WebSocket** | `URLSession` | `ClientWebSocket` | System.Net.WebSockets |
| **HTTP** | `URLSession` | `HttpClient` | System.Net.Http |
| **Camera** | `AVCaptureDevice` | `MediaCapture` | Windows.Media.Capture |
| **Screen** | `CGDisplayStream` | `GraphicsCapture` | Windows.Graphics.Capture |
| **Microphone** | `AVAudioEngine` | `AudioCapture` | Windows.Media.Capture |
| **Bonjour** | `NWBrowser` | Bonjour SDK | Or implement mDNS |
| **File System** | `FileManager` | `System.IO.File` | .NET file I/O |
| **JSON** | `Codable` | `System.Text.Json` | .NET JSON serialization |
| **SQLite** | `sqlite3` C API | `System.Data.SQLite` | or Microsoft.Data.SQLite |
| **Registry** | N/A | `Registry` | Microsoft.Win32.Registry |
| **Notifications** | `NSUserNotification` | `ToastNotification` | Windows.UI.Notifications |
| **Auto-Start** | `launchctl` | Task Scheduler | or Registry Run key |
| **Permissions** | TCC | Runtime checks | No direct equivalent |
| **Deep Links** | `Info.plist` | Registry | HKEY_CLASSES_ROOT |
| **Process** | `Process` | `Process` | System.Diagnostics.Process |

---

## Feature Checklist for Future Windows Port

When the Windows port begins, ensure these features are implemented:

### Phase 1: Core (Required)

- [ ] System tray icon with context menu
- [ ] Gateway WebSocket client connection
- [ ] Auto-start service (Task Scheduler)
- [ ] Canvas panel (WebView2)
- [ ] Exec approval UI
- [ ] Configuration window
- [ ] Status indicators
- [ ] Custom URL scheme handler (moltbot://)

### Phase 2: Integration (Important)

- [ ] Camera capture
- [ ] Screen recording
- [ ] Bonjour discovery
- [ ] Browser detection
- [ ] Notification handling
- [ ] Permission checks (runtime)

### Phase 3: Enhancement (Nice to Have)

- [ ] Location services
- [ ] Speech recognition
- [ ] Advanced settings
- [ ] Update mechanism (WinSparkle)
- [ ] Crash reporting
- [ ] Analytics/telemetry

---

## Code Review Checklist

When reviewing macOS PRs, consider:

- [ ] Does this feature have a Windows equivalent?
- [ ] Is the platform-specific code clearly marked?
- [ ] Are the dependencies documented?
- [ ] Is this feature tracked in this document?
- [ ] Should this be P0/P1/P2 for Windows?
- [ ] Are there any assumptions that won't hold on Windows?
- [ ] Is the testing approach documented?

---

## Best Practices for macOS Development

### 1. Isolate Platform-Specific Code

```swift
// ✅ Good: Platform-specific code is isolated
#if os(macOS)
import LaunchAgent
#endif

class ServiceManager {
    func install() {
        #if os(macOS)
        LaunchAgent.install()
        #endif
    }
}
```

### 2. Document Platform Assumptions

```swift
// ⚠️ Document macOS-specific assumptions

/// - Requires: macOS 11+ Big Sur or later
/// - Uses: TCC (Transparency, Consent, and Control)
/// - Note: Windows doesn't have TCC; must check permissions at runtime
func requestPermissions() async throws {
    // macOS TCC checks
}
```

### 3. Avoid Hardcoded Paths

```swift
// ❌ Bad: Hardcoded macOS path
let configPath = "/Users/\(NSUserName())/Library/Application Support/Moltbot"

// ✅ Good: Use cross-platform approach
let configPath = FileManager.default.urls(
    for: .applicationSupportDirectory,
    in: .userDomainMask
).first?.appendingPathComponent("Moltbot")
```

### 4. Abstract When Possible

```swift
// ✅ Good: Platform-agnostic interface
protocol BrowserDetector {
    func findBrowsers() async -> [Browser]
    func getDefaultBrowser() async -> Browser?
}

// Platform-specific implementations
class MacOSBrowserDetector: BrowserDetector { /* ... */ }
class WindowsBrowserDetector: BrowserDetector { /* ... */ }
```

---

## Testing Considerations

### What to Test on Windows

When a feature is added to macOS, document what needs to be tested on Windows:

1. **Functional:** Does it work the same way?
2. **Visual:** Does it look correct (DPI, scaling)?
3. **Performance:** Is it responsive?
4. **Edge Cases:** What happens on failure?
5. **Permissions:** How are permission errors handled?

### Test Matrix

| Feature | macOS Test | Windows Test | Notes |
|---------|-----------|--------------|-------|
| System tray | Click left/right | Click left/right | Same behavior |
| Canvas | Load page | Load page | Test on WebView2 |
| Camera | Capture photo | Capture photo | Test permissions |
| Auto-start | Reboot | Reboot | Test service starts |

---

## Resources for Windows Development

### Documentation

- [Windows App SDK Docs](https://aka.ms/windowsappsdk)
- [WinUI 3 Gallery](https://github.com/microsoft/WinUI-Gallery)
- [WebView2 Docs](https://docs.microsoft.com/en-us/microsoft-edge/webview2/)
- [.NET Docs](https://docs.microsoft.com/en-us/dotnet/)

### Sample Code

- [Windows App SDK Samples](https://github.com/microsoft/WindowsAppSDK-Samples)
- [WebView2 Samples](https://github.com/MicrosoftEdge/WebView2Samples)
- [WinUI 3 Samples](https://github.com/microsoft/WinUI-Gallery/tree/main/demos)

### Tools

- **Visual Studio 2022** - Windows development IDE
- **WiX Toolset** - MSI installer creation
- **WiX Toolset** WiX Toolset** - Code signing
- **Windows SDK** - Windows headers and tools

---

## Questions to Ask Before Adding macOS Features

When planning a new macOS feature, ask:

1. **Can this be implemented on Windows?**
   - If no, should we still add it to macOS?

2. **What's the Windows API equivalent?**
   - Document in this file

3. **Is there a cross-platform alternative?**
   - Consider using that instead

4. **How complex is the Windows implementation?**
   - Plan for future effort

5. **Should this be P0, P1, or P2 for Windows?**
   - Prioritize accordingly

---

## Revision History

| Date | Change | Author |
|------|--------|--------|
| 2026-01-28 | Initial document creation | Planning Phase |
| | | |

---

## Summary

**Current Status:** Windows port is on hold, Linux server support is the priority.

**Goal:** When Windows development begins, we should have:
1. Complete feature list from this document
2. Clear API mappings
3. Implementation effort estimates
4. Best practices to follow
5. Testing strategy

**Responsibility:** All macOS app developers should track changes in this document.

**Maintenance:** Update this document whenever:
- New features are added to macOS app
- APIs are used that are platform-specific
- Architecture decisions are made
- Testing approaches are defined

---

*This document ensures that when the Windows port begins, the development team has complete visibility into what needs to be implemented to maintain feature parity.*
