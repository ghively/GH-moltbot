# Cross-Platform Compatibility Analysis

> **Analysis of Apple-specific features and Linux compatibility strategy**
> Generated: 2025-01-28
> Status: Fork Planning Document

---

## Executive Summary

Moltbot was designed with **macOS-first integration** but has a foundation that can work cross-platform. This document analyzes Apple-specific dependencies and provides a roadmap for **full Linux compatibility** while maintaining macOS feature parity.

### Key Findings

| Category | macOS-Only | Already Cross-Platform | Action Needed |
|----------|-----------|----------------------|---------------|
| **Core Gateway** | ✅ | ✅ | None |
| **iMessage (native)** | ✅ | ❌ | Use BlueBubbles |
| **iMessage (BlueBubbles)** | ❌ | ✅ | None - already works! |
| **macOS App** | ✅ | ❌ | Web UI or separate native apps |
| **iOS App** | ✅ | ❌ | Separate mobile apps |
| **Apple Notes** | ✅ | ❌ | Use Notion/Obsidian APIs |
| **Apple Reminders** | ✅ | ❌ | Use Things/Trello APIs |
| **Service Management** | ✅ (launchd) | ❌ | Add systemd support |
| **Browser Detection** | ✅ (osascript) | ❌ | Add Linux methods |
| **Camera/Screen** | ✅ (via macOS app) | ❌ | Use browser tools |
| **Bonjour Discovery** | ✅ (NWBrowser) | ❌ | Use avahi/bonjour |

---

## Part 1: Feature-by-Feature Analysis

### 1. Core Gateway System ✅ Cross-Platform

**Status:** Fully compatible with Linux

**Components:**
- WebSocket server (uses `ws` package - cross-platform)
- HTTP server (Express/Hono - cross-platform)
- Authentication (token/password/Tailscale - cross-platform)
- Session management (filesystem-based - cross-platform)
- Configuration (YAML files - cross-platform)

**Files:**
- `src/gateway/*.ts` - All cross-platform
- `src/config/*.ts` - All cross-platform
- `src/agents/*.ts` - All cross-platform

**Linux Compatibility:** ✅ No changes needed

---

### 2. iMessage Integration ⚠️ Platform-Dependent

#### 2.1 Native iMessage (macOS-only)

**Location:** `src/imessage/` and `extensions/imessage/`

**macOS Dependency:**
- Requires `imsg` CLI tool (external binary)
- `imsg` reads Apple Messages.db directly
- Uses AppleScript for sending

**Files:**
```
src/imessage/
├── client.ts          # JSON-RPC client to imsg binary
├── send.ts            # Send via imsg
├── monitor.ts         # Watch for messages
└── probe.ts           # Check if imsg available

extensions/imessage/
├── index.ts
└── src/channel.ts
```

**Linux Compatibility:** ❌ **NOT COMPATIBLE**

**Why:** Requires macOS Messages database and AppleScript

**Solution:** Use BlueBubbles instead (see below)

---

#### 2.2 BlueBubbles Channel ✅ Already Cross-Platform!

**Location:** `extensions/bluebubbles/`

**Status:** This is already a **cross-platform alternative** to native iMessage!

**How it Works:**
- BlueBubbles is a third-party server that runs on macOS
- Exposes iMessage via REST API + WebSocket
- Can run on one Mac and be accessed from any device
- Linux server can connect to BlueBubbles over network

**Files:**
```
extensions/bluebubbles/
├── index.ts           # Plugin registration
├── src/channel.ts     # Channel implementation
├── src/monitor.ts     # Webhook-based monitoring
├── src/send.ts        # Send via BlueBubbles API
├── src/reactions.ts   # Message reactions
└── src/attachments.ts # Media handling
```

**Linux Compatibility:** ✅ **FULLY COMPATIBLE**

**Deployment Strategy:**
1. Run BlueBubbles server on a Mac (or in Docker on Mac with host)
2. Run Moltbot gateway on Linux
3. Configure gateway to connect to BlueBubbles
4. Full iMessage functionality from Linux!

**Configuration Example:**
```yaml
channels:
  bluebubbles:
    enabled: true
    serverUrl: "https://bluebubbles-server:8080"
    apiKey: "${BLUEBUBBLES_API_KEY}"
```

**Recommendation:** ✅ **USE BLUEBUBBLES** - No code changes needed!

---

### 3. macOS Native App ❌ macOS-Only

**Location:** `apps/macos/`

**Purpose:** Menu bar companion app with system integration

**Features:**
| Feature | macOS Dependency | Linux Alternative |
|---------|-----------------|-------------------|
| **Menu bar UI** | NSStatusBar, AppKit | None (use Web UI) |
| **LaunchAgent mgmt** | launchctl | systemd |
| **Permission management** | TCC dialogs | Browser-based approval |
| **Camera access** | AVFoundation | Browser tools |
| **Screen recording** | NSScreen, AVFoundation | Browser tools |
| **Microphone** | AVFoundation | Browser tools |
| **Location services** | CLLocationManager | Not available |
| **Bonjour discovery** | NWBrowser | avahi-daemon |
| **Browser detection** | NSWorkspace, LaunchServices | xdg-utils |
| **Notification center** | NSUserNotification | Browser notifications |
| **XPC services** | NSXPCConnection | Not needed |
| **Sparkle updates** | Sparkle Framework | Web-based updates |

**Files:** 100+ Swift files
- `Sources/Moltbot/*.swift` - Main app
- `Sources/MoltbotIPC/*` - IPC protocol
- `Sources/MoltbotProtocol/*` - Shared protocol

**Linux Compatibility:** ❌ **NOT COMPATIBLE**

**Solution:** Web UI (`ui/`) already provides most functionality

**Recommendation:** For Linux, use the **Web UI** instead of native macOS app

---

### 4. iOS App ❌ iOS-Only (Separate Platform)

**Location:** `apps/ios/`

**Purpose:** Mobile companion app

**Features:**
- Chat interface
- Voice wake ("Hey Siri" style)
- Screen recording
- Camera capture
- Location services

**Linux Compatibility:** ❌ **SEPARATE PLATFORM**

**Status:** iOS app is a **client**, not server-side

**Recommendation:** Keep as-is (iOS app connects to Linux gateway just fine)

---

### 5. Apple Notes Skill ⚠️ macOS-Only

**Location:** `skills/apple-notes/`

**How it Works:** Uses AppleScript to access Apple Notes database

**macOS Dependency:**
```bash
osascript -e 'tell application "Notes" to get notes'
```

**Linux Compatibility:** ❌ **NOT COMPATIBLE**

**Alternatives:**
| Platform | Solution | Status |
|----------|----------|--------|
| **Notion** | `skills/notion/` | ✅ Already exists |
| **Obsidian** | `skills/obsidian/` | ✅ Already exists |
| **Bear Notes** | `skills/bear-notes/` | ⚠️ macOS-only (API) |

**Recommendation:** Document that Apple Notes skill is macOS-only. Recommend Notion/Obsidian for Linux users.

---

### 6. Apple Reminders Skill ⚠️ macOS-Only

**Location:** `skills/apple-reminders/`

**How it Works:** Uses AppleScript to access Reminders database

**macOS Dependency:**
```bash
osascript -e 'tell application "Reminders" to get reminders'
```

**Linux Compatibility:** ❌ **NOT COMPATIBLE**

**Alternatives:**
| Platform | Solution | Status |
|----------|----------|--------|
| **Things** | `skills/things-mac/` | ⚠️ macOS-only (API) |
| **Trello** | `skills/trello/` | ✅ Already exists |
| **Generic tasks** | Build new skill | 🔄 TODO |

**Recommendation:** Document that Apple Reminders is macOS-only. Recommend Trello for cross-platform.

---

### 7. Service Management (launchd) ⚠️ Platform-Specific

**Location:** `src/daemon/launchd.ts` and `src/daemon/service-audit.ts`

**Current Implementation:**
- macOS: Uses `launchctl` and `LaunchAgent` plist files
- Linux: Not implemented
- Windows: Not implemented

**Files:**
```
src/daemon/
├── launchd.ts           # macOS launchd integration
├── launchd-plist.ts     # Generate plist files
└── service-audit.ts     # Detect service type
```

**Linux Compatibility:** ⚠️ **NEEDS SYSTEMD SUPPORT**

**Solution:** Add systemd service support

**Implementation Plan:**

1. Create `src/daemon/systemd.ts`:
```typescript
// Systemd service management for Linux
export function installSystemdService(config: ServiceConfig): Promise<void>;
export function uninstallSystemdService(): Promise<void>;
export function startSystemdService(): Promise<void>;
export function stopSystemdService(): Promise<void>;
export function getServiceStatus(): Promise<ServiceStatus>;
```

2. Generate systemd unit file:
```ini
[Unit]
Description=Moltbot Gateway
After=network.target

[Service]
Type=simple
User=moltbot
WorkingDirectory=/opt/moltbot
ExecStart=/usr/bin/moltbot gateway
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

3. Update `src/daemon/service-audit.ts`:
```typescript
export function detectServiceManager(): "launchd" | "systemd" | "sc" | "unknown" {
  if (process.platform === "darwin") return "launchd";
  if (process.platform === "linux") {
    // Check if systemd is active
    try {
      execSync("systemctl --version");
      return "systemd";
    } catch {
      return "unknown";
    }
  }
  if (process.platform === "win32") return "sc";
  return "unknown";
}
```

4. Update CLI:
```bash
moltbot daemon install  # Auto-detects platform
```

**Recommendation:** 🔄 **IMPLEMENT SYSTEMD SUPPORT**

---

### 8. Browser Detection ⚠️ Platform-Specific

**Location:** `src/browser/chrome.executables.ts`

**Current Implementation:**

**macOS:**
```typescript
// Uses osascript to query LaunchServices
const script = `
  tell application "System Events"
    get POSIX path of (path to application "Google Chrome")
  end tell
`;
```

**Windows:** Uses Registry queries

**Linux:** ⚠️ **NOT IMPLEMENTED**

**Solution for Linux:**

1. Check common Linux paths:
```typescript
const LINUX_CHROME_PATHS = [
  "/usr/bin/google-chrome",
  "/usr/bin/chromium",
  "/usr/bin/chromium-browser",
  "/opt/google/chrome/google-chrome",
  "/usr/bin/brave-browser",
  "/usr/bin/microsoft-edge",
  "/usr/bin/vivaldi",
  "/usr/bin/opera",
];
```

2. Use `which` command:
```bash
which google-chrome
which chromium-browser
```

3. Check xdg-mime for default browser:
```bash
xdg-settings get default-web-browser
```

4. Read desktop entries:
```typescript
// Parse /usr/share/applications/*.desktop
// Look for: Categories=WebBrowser
```

**Implementation:** Add Linux detection to `chrome.executables.ts`

**Recommendation:** 🔄 **IMPLEMENT LINUX BROWSER DETECTION**

---

### 9. Camera & Screen Recording ⚠️ Indirectly macOS-Only

**Location:** Camera/screen features are in macOS app (`apps/macos/`)

**Gateway Tools:**
- `src/agents/tools/browser-tool.ts` - Cross-platform!
- Uses Playwright (cross-platform headless browser)

**macOS App Features:**
- `Sources/Moltbot/CameraCaptureService.swift` - Native camera
- `Sources/Moltbot/ScreenRecordService.swift` - Native screen recording

**Linux Compatibility:**
- ❌ Native macOS app features not available
- ✅ Browser-based tools work fine!

**Solution:** Use Playwright-based browser tools for Linux

**Recommendation:** Document that for camera/screen on Linux, use browser-tool (works cross-platform)

---

### 10. Bonjour/mDNS Discovery ⚠️ Platform-Specific

**Location:** `src/infra/bonjour-discovery.ts`

**Current Implementation:**
- macOS: Uses `@homebridge/ciao` (Bonjour implementation)
- Linux: Not implemented
- Windows: Not implemented

**Linux Solution:** Use **avahi-daemon**

**Implementation:**

1. Install avahi:
```bash
sudo apt-get install avahi-daemon libavahi-client-dev
```

2. Use `avahi` Node.js package:
```bash
pnpm add avahi-publish
```

3. Create `src/infra/avahi-discovery.ts`:
```typescript
import { mdns } from 'avahi-publish';

export function startAvahiDiscovery(options: DiscoveryOptions) {
  return mdns.createBrowser(mdns.tcp('moltbot'))
    .on('serviceUp', (service) => {
      console.log('Gateway found:', service.name);
    })
    .start();
}
```

**Alternative:** Use `bonjour-hap` package (cross-platform)

**Recommendation:** 🔄 **IMPLEMENT AVAHI SUPPORT FOR LINUX**

---

### 11. Code Signing & Packaging ❌ macOS-Only

**Location:** `scripts/codesign-mac-app.sh`, `scripts/create-dmg.sh`, etc.

**macOS-Specific Tools:**
- `codesign` - Code signing
- `hdiutil` - DMG creation
- `PlistBuddy` - plist manipulation
- `xcrun` - Xcode tools
- `security` - Keychain access

**Linux Compatibility:** ❌ **NOT APPLICABLE**

**Linux Packaging Alternatives:**
| Type | macOS | Linux |
|------|-------|-------|
| **Package** | .app, .dmg | .deb, .rpm, .AppImage |
| **Installer** | DMG | apt, yum, dnf |
| **Code Signing** | codesign | GPG signing |
| **Service** | LaunchAgent | systemd |

**Linux Packaging Scripts to Add:**
```bash
scripts/package-deb.sh      # Create .deb package
scripts/package-rpm.sh      # Create .rpm package
scripts/package-appimage.sh # Create AppImage
scripts/install-systemd.sh  # Install systemd service
scripts/sign-package.sh     # GPG signing
```

**Recommendation:** 🔄 **ADD LINUX PACKAGING SCRIPTS**

---

## Part 2: Cross-Platform Compatibility Matrix

### Complete Feature Breakdown

| Feature | macOS | Linux | Windows | Notes |
|---------|-------|-------|---------|-------|
| **Core Gateway** | ✅ | ✅ | ✅ | Fully cross-platform |
| **Web UI** | ✅ | ✅ | ✅ | Browser-based |
| **TUI** | ✅ | ✅ | ✅ | Terminal-based |
| **CLI** | ✅ | ✅ | ✅ | Node.js CLI |
| **iMessage (native)** | ✅ | ❌ | ❌ | Requires macOS |
| **iMessage (BlueBubbles)** | ✅ | ✅ | ✅ | **Use this for Linux!** |
| **Telegram** | ✅ | ✅ | ✅ | Bot API - cross-platform |
| **WhatsApp** | ✅ | ✅ | ✅ | Web - cross-platform |
| **Discord** | ✅ | ✅ | ✅ | Bot API - cross-platform |
| **Slack** | ✅ | ✅ | ✅ | Socket Mode - cross-platform |
| **Signal** | ✅ | ✅ | ✅ | signal-cli - works on Linux |
| **Google Chat** | ✅ | ✅ | ✅ | Webhook - cross-platform |
| **macOS App** | ✅ | ❌ | ❌ | Native macOS only |
| **iOS App** | ✅ | ✅* | ✅* | *Connects to any gateway |
| **Android App** | ✅ | ✅* | ✅* | *Connects to any gateway |
| **Apple Notes skill** | ✅ | ❌ | ❌ | AppleScript - macOS only |
| **Apple Reminders skill** | ✅ | ❌ | ❌ | AppleScript - macOS only |
| **Notion skill** | ✅ | ✅ | ✅ | API - cross-platform |
| **Obsidian skill** | ✅ | ✅ | ✅ | Cross-platform |
| **Trello skill** | ✅ | ✅ | ✅ | API - cross-platform |
| **GitHub skill** | ✅ | ✅ | ✅ | API - cross-platform |
| **Weather skill** | ✅ | ✅ | ✅ | API - cross-platform |
| **Browser tools** | ✅ | ✅ | ✅ | Playwright - cross-platform |
| **Camera (native)** | ✅ | ❌ | ❌ | macOS app only |
| **Camera (browser)** | ✅ | ✅ | ✅ | Playwright works on Linux |
| **Screen recording (native)** | ✅ | ❌ | ❌ | macOS app only |
| **Screen recording (browser)** | ✅ | ✅ | ✅ | Playwright works on Linux |
| **Location services** | ✅ | ❌ | ❌ | iOS/macOS only |
| **Bonjour discovery** | ✅ | ⚠️ | ⚠️ | Needs avahi on Linux |
| **LaunchAgent** | ✅ | ❌ | ❌ | macOS only |
| **systemd service** | ❌ | ⚠️ | ❌ | Needs implementation |
| **Code signing** | ✅ | ❌ | ❌ | Platform-specific |

**Legend:**
- ✅ Fully supported
- ⚠️ Partially supported or needs implementation
- ❌ Not supported

---

## Part 3: Implementation Roadmap

### Priority 1: Core Linux Server Support ✅ Already Done

These features already work on Linux:

- [x] Core gateway (WebSocket, HTTP, RPC)
- [x] Configuration system
- [x] Agent runtime
- [x] Most channels (Telegram, WhatsApp, Discord, Slack, Signal, Google Chat)
- [x] Most skills (Notion, Obsidian, Trello, GitHub, Weather, etc.)
- [x] Browser tools (Playwright)
- [x] Web UI
- [x] TUI
- [x] CLI
- [x] BlueBubbles integration (iMessage alternative)

**Status:** ✅ **Moltbot already runs on Linux!**

---

### Priority 2: Service Management (systemd) 🔄 TODO

**Files to Create/Modify:**
```
src/daemon/systemd.ts           # NEW - systemd integration
src/daemon/service-units.ts     # NEW - unit file generation
src/daemon/index.ts             # MODIFY - export systemd functions
src/daemon/service-audit.ts     # MODIFY - add systemd detection
src/cli/daemon-cli/register.ts  # MODIFY - use systemd on Linux
```

**Implementation Steps:**
1. Create systemd service manager
2. Generate `.service` unit files
3. Add `systemctl` wrapper functions
4. Update daemon install CLI
5. Test on Ubuntu/Debian/Fedora

**Estimated Effort:** 4-6 hours

---

### Priority 3: Browser Detection (Linux) 🔄 TODO

**Files to Modify:**
```
src/browser/chrome.executables.ts   # MODIFY - add Linux paths
src/browser/detection.ts            # MODIFY - add Linux logic
src/infra/browser-detection.ts      # NEW - abstract platform detection
```

**Implementation Steps:**
1. Add Linux browser paths
2. Implement `which` command checks
3. Add xdg-utils integration
4. Parse `.desktop` files
5. Test on Ubuntu/Fedora

**Estimated Effort:** 2-3 hours

---

### Priority 4: Bonjour Discovery (avahi) 🔄 TODO

**Files to Create/Modify:**
```
src/infra/avahi-discovery.ts       # NEW - avahi implementation
src/infra/discovery.ts             # MODIFY - platform abstraction
src/gateway/server-discovery.ts    # MODIFY - use avahi on Linux
```

**Implementation Steps:**
1. Add avahi dependency
2. Implement mDNS browser using avahi
3. Abstract discovery interface
4. Update gateway to use correct implementation
5. Test on Ubuntu/Avahi

**Estimated Effort:** 3-4 hours

---

### Priority 5: Documentation 🔄 TODO

**Files to Create:**
```
docs/platforms/linux.md            # NEW - Linux setup guide
docs/platforms/linux/systemd.md    # NEW - systemd service setup
docs/platforms/linux/install.md    # NEW - Linux installation
docs/platforms/linux/troubleshoot.md # NEW - Linux issues
docs/CROSS_PLATFORM_GUIDE.md       # NEW - Cross-platform features
docs/PLATFORM_MATRIX.md            # NEW - Feature compatibility matrix
```

**Estimated Effort:** 2-3 hours

---

### Priority 6: Packaging (Linux) 🔄 TODO

**Scripts to Create:**
```
scripts/package-deb.sh            # NEW - .deb package
scripts/package-rpm.sh            # NEW - .rpm package
scripts/package-appimage.sh       # NEW - AppImage
scripts/install-systemd.sh        # NEW - systemd installer
scripts/sign-package.sh           # NEW - GPG signing
```

**Estimated Effort:** 6-8 hours

---

## Part 4: Feature Alternatives for Linux

### Apple Notes → Notion/Obsidian

| Platform | Solution | Skill | Status |
|----------|----------|-------|--------|
| **macOS** | Apple Notes (AppleScript) | `apple-notes` | ✅ Works |
| **Linux/All** | Notion API | `notion` | ✅ Cross-platform |
| **Linux/All** | Obsidian (local vault) | `obsidian` | ✅ Cross-platform |

**Recommendation:** For Linux users, recommend Notion or Obsidian instead of Apple Notes

---

### Apple Reminders → Trello/Things

| Platform | Solution | Skill | Status |
|----------|----------|-------|--------|
| **macOS** | Reminders (AppleScript) | `apple-reminders` | ✅ Works |
| **Linux/All** | Trello API | `trello` | ✅ Cross-platform |
| **macOS** | Things (macOS app) | `things-mac` | ⚠️ macOS-only |

**Recommendation:** For Linux users, recommend Trello instead of Apple Reminders

---

### Native iMessage → BlueBubbles

| Platform | Solution | Extension | Status |
|----------|----------|-----------|--------|
| **macOS** | Native (imsg binary) | `imessage` | ✅ Works on macOS |
| **All** | BlueBubbles API | `bluebubbles` | ✅ Cross-platform! |

**BlueBubbles Architecture for Linux:**
```
┌─────────────────┐         ┌──────────────────┐
│  Linux Server   │         │  macOS (BlueBubbles)│
│  (Moltbot)      │◄───────►│  Server           │
│                 │ WebSocket│                  │
│  extensions/    │         │  iMessage Bridge  │
│  bluebubbles/   │         └──────────────────┘
└─────────────────┘                  │
                                    │
                             ┌──────┴─────┐
                             │  iMessage  │
                             │  Servers   │
                             └────────────┘
```

**Deployment:**
1. Install BlueBubbles on a Mac (or Mac VM)
2. Run BlueBubbles server
3. Configure Moltbot on Linux to connect
4. Full iMessage functionality!

**Recommendation:** ✅ **USE BLUEBUBBLES FOR LINUX**

---

### macOS App → Web UI

| Feature | macOS App | Web UI | Linux Alternative |
|---------|-----------|--------|-------------------|
| **Chat interface** | Native | ✅ Web UI | Use Web UI |
| **Configuration** | Native | ✅ Web UI | Use Web UI |
| **Approvals** | Native dialogs | ✅ Web UI | Use Web UI |
| **Menu bar** | Menu bar | ❌ N/A | Use systemd tray |
| **Camera** | Native AVFoundation | ❌ N/A | Use browser-tool |
| **Screen** | Native NSScreen | ❌ N/A | Use browser-tool |
| **Bonjour** | NWBrowser | ⚠️ Needs avahi | Implement avahi |
| **Browser detect** | NSWorkspace | ⚠️ Needs impl | Implement xdg |

**Recommendation:** For Linux, use Web UI (`ui/`) instead of macOS app

---

## Part 5: Configuration Strategy

### Platform-Aware Configuration

**config.yaml** should be platform-agnostic:

```yaml
# Platform detection happens automatically
gateway:
  port: 18789
  bind: auto  # loopback on macOS, lan on Linux (headless)

# Channels - enable what works on your platform
channels:
  # Works on all platforms
  telegram:
    enabled: true
  whatsapp:
    enabled: true
  discord:
    enabled: true
  slack:
    enabled: true

  # iMessage - choose your method
  imessage:
    enabled: false  # macOS only (native)
  bluebubbles:
    enabled: true   # Cross-platform!
    serverUrl: "https://bluebubbles:8080"

  # Signal - works on Linux
  signal:
    enabled: true

# Skills - platform-aware
skills:
  # Cross-platform
  notion:
    enabled: true
  obsidian:
    enabled: true
  trello:
    enabled: true

  # macOS only
  apple-notes:
    enabled: false  # Auto-detect macOS
  apple-reminders:
    enabled: false  # Auto-detect macOS
```

### Auto-Detection Logic

**Implementation in `src/config/config.ts`:**

```typescript
function detectPlatformCapabilities() {
  const capabilities = {
    platform: process.platform,
    hasImessage: process.platform === "darwin",
    hasSystemd: process.platform === "linux" && hasSystemd(),
    hasLaunchd: process.platform === "darwin",
    hasAvahi: process.platform === "linux" && hasAvahi(),
  };

  // Auto-disable incompatible features
  if (!capabilities.hasImessage) {
    config.channels.imessage = { enabled: false };
  }

  return config;
}
```

---

## Part 6: Recommendations Summary

### Immediate Actions (No Code Changes)

1. ✅ **Use BlueBubbles for iMessage on Linux**
   - Already implemented
   - Works cross-platform
   - No code changes needed

2. ✅ **Use Web UI on Linux**
   - Already implemented
   - Cross-platform
   - No code changes needed

3. ✅ **Document platform-specific features**
   - Create `docs/platforms/linux.md`
   - Document what works/doesn't
   - Provide alternatives

### Code Changes Needed

4. 🔄 **Add systemd service support**
   - Priority: HIGH
   - Effort: 4-6 hours
   - Files: `src/daemon/systemd.ts`

5. 🔄 **Add Linux browser detection**
   - Priority: MEDIUM
   - Effort: 2-3 hours
   - Files: `src/browser/chrome.executables.ts`

6. 🔄 **Add avahi Bonjour support**
   - Priority: MEDIUM
   - Effort: 3-4 hours
   - Files: `src/infra/avahi-discovery.ts`

7. 🔄 **Create Linux packaging scripts**
   - Priority: LOW
   - Effort: 6-8 hours
   - Files: `scripts/package-*.sh`

### Future Enhancements

8. 📋 **Create Linux native companion app**
   - Use Electron or Tauri
   - Provide system tray integration
   - Replicate macOS app features

9. 📋 **Add more cross-platform notes/task integrations**
   - Any.do API
   - Todoist API
   - Microsoft To Do

---

## Part 7: Testing Strategy

### Linux Testing Checklist

- [ ] Test core gateway on Ubuntu 22.04
- [ ] Test core gateway on Debian 12
- [ ] Test core gateway on Fedora 39
- [ ] Test BlueBubbles integration from Linux
- [ ] Test all cross-platform channels (Telegram, WhatsApp, Discord, Slack)
- [ ] Test systemd service installation
- [ ] Test avahi Bonjour discovery
- [ ] Test browser tools on Linux
- [ ] Test Web UI on Linux browsers (Chrome, Firefox)
- [ ] Test TUI on Linux terminal
- [ ] Package as .deb
- [ ] Package as .rpm
- [ ] Package as AppImage

### Test Environment

**Docker Compose for Testing:**
```yaml
services:
  moltbot:
    image: moltbot:latest
    platform: linux/amd64
    ports:
      - "18789:18789"
    volumes:
      - ./config:/root/.moltbot
    environment:
      - CLAWDBOT_GATEWAY_PORT=18789

  bluebubbles:
    image: bluebubbles/server:latest
    ports:
      - "8080:8080"
```

---

## Part 8: Deployment Examples

### Linux Server Setup

```bash
# 1. Install Node.js 22+
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# 2. Install Moltbot
npm install -g moltbot@latest

# 3. Create configuration
moltbot doctor  # Interactive setup

# 4. Install systemd service
sudo moltbot daemon install

# 5. Start service
sudo systemctl start moltbot
sudo systemctl enable moltbot

# 6. Check status
sudo systemctl status moltbot
journalctl -u moltbot -f
```

### BlueBubbles + Linux Setup

```bash
# On Mac: Install BlueBubbles
# 1. Download BlueBubbles from GitHub
# 2. Run BlueBubbles server
# 3. Configure webhook URL

# On Linux: Configure Moltbot
cat > ~/.moltbot/config.yaml <<EOF
channels:
  bluebubbles:
    enabled: true
    serverUrl: "https://mac-mini.local:8080"
    apiKey: "${BLUEBUBBLES_API_KEY}"
EOF

# Start gateway
moltbot gateway
```

---

## Conclusion

### Current State

✅ **Moltbot already runs on Linux!** The core gateway, most channels, and most skills are fully cross-platform.

### What Needs Work

Only a few macOS-specific features need Linux equivalents:
1. systemd service management (instead of LaunchAgent)
2. Linux browser detection (instead of osascript)
3. avahi Bonjour support (instead of native macOS)

### What Doesn't Need Work

- ✅ Core gateway (already cross-platform)
- ✅ BlueBubbles iMessage (already cross-platform)
- ✅ Most channels (Telegram, WhatsApp, Discord, Slack, Signal, Google Chat)
- ✅ Most skills (Notion, Obsidian, Trello, GitHub, Weather, etc.)
- ✅ Web UI (browser-based, cross-platform)
- ✅ Browser tools (Playwright, cross-platform)

### Key Insight

**The macOS app is NOT required for Linux.** Use the Web UI instead!

### Recommended Approach for Fork

1. **Keep macOS app** as-is (for Mac users)
2. **Add systemd support** for Linux service management
3. **Document platform-specific features** clearly
4. **Promote BlueBubbles** as the cross-platform iMessage solution
5. **Promote Web UI** as the cross-platform interface
6. **Add Linux packaging** (.deb, .rpm, AppImage)
7. **Test thoroughly** on Ubuntu/Debian/Fedora

---

## Appendix A: Platform Detection Code

### Service Manager Detection

```typescript
// src/daemon/service-audit.ts
export function detectServiceManager(): "launchd" | "systemd" | "sc" | "unknown" {
  if (process.platform === "darwin") {
    return "launchd";
  }
  if (process.platform === "linux") {
    try {
      execSync("systemctl --version", { stdio: "ignore" });
      return "systemd";
    } catch {
      return "unknown";
    }
  }
  if (process.platform === "win32") {
    return "sc";
  }
  return "unknown";
}
```

### Feature Detection

```typescript
// src/infra/platform.ts
export const platform = {
  isDarwin: process.platform === "darwin",
  isLinux: process.platform === "linux",
  isWindows: process.platform === "win32",

  hasImessage: process.platform === "darwin",
  hasBlueBubbles: true,  // Cross-platform
  hasSystemd: process.platform === "linux" && hasSystemd(),
  hasLaunchd: process.platform === "darwin",
  hasAvahi: process.platform === "linux" && hasAvahi(),
};
```

---

## Appendix B: Resources

### Cross-Platform Libraries

| Feature | macOS | Linux | Windows |
|---------|-------|-------|---------|
| **Service mgmt** | launchctl | systemd | sc |
| **mDNS** | @homebridge/ciao | avahi | mdns-js |
| **Browser detect** | osascript | xdg-utils | Registry |
| **Packaging** | pkgbuild | debuild/rpmbuild | MSI |
| **Signing** | codesign | GPG | signtool |

### Useful Packages

```json
{
  "dependencies": {
    "systemd": "^2.0.0",           // systemd D-Bus API
    "avahi-publish": "^0.1.0",      // avahi mDNS
    "bonjour-hap": "^3.6.0",       // Cross-platform mDNS
    "xdg-desktop-portal": "^0.1.0" // Linux desktop integration
  }
}
```

---

*This document provides a complete roadmap for making Moltbot fully cross-platform while maintaining all existing functionality.*
