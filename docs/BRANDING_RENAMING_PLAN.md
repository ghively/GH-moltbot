# BobBot Rebranding Plan

> **Comprehensive rebranding strategy from Moltbot/Clawdbot to BobBot**
> **Generated:** 2026-01-28
> **Status:** Planning Phase
> **Scope:** 2,169+ files across entire codebase

---

## Executive Summary

This document provides a **complete roadmap** for rebranding Moltbot (formerly Clawdbot) to BobBot. The rebranding encompasses the package name, binary names, environment variables, state directories, documentation, UI text, mascot, and all user-facing elements.

### Branding Changes

| Element | Current | New | Notes |
|---------|---------|-----|-------|
| **Product Name** | Moltbot | **BobBot** | Primary brand name |
| **Package Name** | moltbot | **bobbot** | npm package name |
| **Binary Name** | moltbot | **bobbot** | CLI command |
| **Legacy Binary** | clawdbot | bobbot (alias) | Keep with deprecation warning |
| **Mascot** | 🦞 Lobster (Clawd) | **🤖 Robot (Bob)** | Character/mascot |
| **Tagline** | "EXFOLIATE! EXFOLIATE!" | "Your personal AI assistant" | Simple, descriptive |
| **State Directory** | ~/.clawdbot/ | **~/.bobbot/** | User data directory |
| **Environment Vars** | CLAWDBOT_* | **BOBBOT_*** | All env vars |
| **Website** | molt.bot | bob.bot (or bobbot.github.io) | When ready |
| **Docs** | docs.molt.bot | bob.bot/docs | When ready |

---

## Scope Analysis

### Files Requiring Changes: **2,169 files**

| Category | Files | Effort | Priority |
|----------|-------|--------|----------|
| **Critical** | ~10 | 1-2 hours | P0 |
| **High Priority** | ~200 | 8-16 hours | P0 |
| **Medium Priority** | ~800 | 40-80 hours | P1 |
| **Low Priority** | ~1,000+ | 20-40 hours | P2 |
| **Optional** | ~100 | 4-8 hours | P3 |
| **Compatibility** | ~50 | 8-16 hours | P1 |

**Total Estimated Effort:** 81-162 hours (2-4 weeks for one person)

**With Automation (sed/find & replace):** 40-80 hours (1-2 weeks)

---

## Part 1: Critical Changes (P0 - Required)

These changes **must** be made for the fork to be rebranded as BobBot.

### 1.1 Package Configuration

**File:** `package.json`

**Changes Required:**

```json
{
  "name": "bobbot",  // was: "moltbot"
  "version": "2026.1.27-beta.1",
  "description": "Personal AI assistant with multi-channel support",  // was: "WhatsApp gateway CLI..."
  "bin": {
    "bobbot": "./bobbot.mjs",  // was: "moltbot": "./moltbot.mjs"
    "clawdbot": "./bobbot.mjs",  // legacy alias (keep with warning)
    "moltbot": "./bobbot.mjs"   // legacy alias (keep with warning)
  },
  "repository": "git@github.com:YOURUSERNAME/bobbobot.git",  // update when forked
}
```

**Action Items:**
- [ ] Update `name` field
- [ ] Update `description`
- [ ] Update `bin` object
- [ ] Add legacy aliases for backward compatibility
- [ ] Update `repository` URL after forking

---

### 1.2 CLI Entry Point

**File:** `moltbot.mjs`

**Options:**
1. **Rename to `bobbot.mjs`** (recommended)
2. Keep `moltbot.mjs` but update internal references

**If renaming:**
```bash
mv moltbot.mjs bobbot.mjs
```

**Update references in:**
- `package.json` (bin field)
- `scripts/` (if they reference the filename)
- `README.md` (installation instructions)

**Action Items:**
- [ ] Rename file to `bobbot.mjs`
- [ ] Update package.json bin reference
- [ ] Search for "moltbot.mjs" references in codebase

---

### 1.3 README.md

**File:** `README.md`

**Changes Required:**

```markdown
# 🤖 BobBot — Personal AI Assistant

<p align="center">
  <!-- Replace logo image -->
  <img src="./docs/bobbobot-logo.png" alt="BobBot" width="400">
</p>

<p align="center">
  <strong>Your personal AI assistant</strong>
</p>
```

**Section Updates:**

1. **Title:** Change from "🦞 Moltbot" to "🤖 BobBot"
2. **Logo:** Replace `docs/whatsapp-clawd.jpg` with new BobBot logo
3. **Badges:** Update repository references (after forking)
4. **Description:** Update product name throughout
5. **Commands:** Update from `moltbot` to `bobbot`
6. **Tagline:** Remove or replace "EXFOLIATE! EXFOLIATE!"
7. **Links:** Update website, docs, Discord URLs (when ready)

**Search and Replace Patterns:**
```bash
# Primary replacements
s/Moltbot/BobBot/g
s/moltbot/bobbot/g
s/🦞/🤖/g  # mascot emoji

# URL updates (after forking)
s/github.com/moltbot\\/moltbot/github.com/YOURUSERNAME\\/bobbobot/g
s/molt\.bot/bob\.bot/g  # or bobbot.github.io
s/docs\.molt\.bot/bob\.bot\\/docs/g
```

**Action Items:**
- [ ] Update title and badges
- [ ] Replace logo image
- [ ] Update all text references
- [ ] Update command examples
- [ ] Update URLs (after forking)

---

### 1.4 License File

**File:** `LICENSE`

**Current:** MIT License referring to "Moltbot" or "Clawdbot"

**Action:** No changes needed (MIT License is generic, doesn't mention product name)

---

## Part 2: Environment Variables (P0 - Critical)

### 2.1 Environment Variable Mapping

**Pattern:** All `CLAWDBOT_*` environment variables → `BOBBOT_*`

**Complete Mapping:**

| Old Variable | New Variable | Files Using It |
|--------------|--------------|----------------|
| `CLAWDBOT_GATEWAY_TOKEN` | `BOBBOT_GATEWAY_TOKEN` | 200+ files |
| `CLAWDBOT_GATEWAY_PASSWORD` | `BOBBOT_GATEWAY_PASSWORD` | 50+ files |
| `CLAWDBOT_STATE_DIR` | `BOBBOT_STATE_DIR` | 30+ files |
| `CLAWDBOT_CONFIG_PATH` | `BOBBOT_CONFIG_PATH` | 20+ files |
| `CLAWDBOT_DEBUG` | `BOBBOT_DEBUG` | 15+ files |
| `CLAWDBOT_SKIP_CHANNELS` | `BOBBOT_SKIP_CHANNELS` | 10+ files |
| `CLAWDBOT_SKIP_PROVIDERS` | `BOBBOT_SKIP_CHANNELS` | 5+ files |
| `CLAWDBOT_NODE_EXEC_HOST` | `BOBBOT_NODE_EXEC_HOST` | 5+ files |
| `CLAWDBOT_NODE_EXEC_FALLBACK` | `BOBBOT_NODE_EXEC_FALLBACK` | 5+ files |
| `CLAWDBOT_NO_RESPAWN` | `BOBBOT_NO_RESPAWN` | 5+ files |
| `CLAWDBOT_RAW_STREAM` | `BOBBOT_RAW_STREAM` | 5+ files |
| `CLAWDBOT_DISABLE_ROUTE_FIRST` | `BOBBOT_DISABLE_ROUTE_FIRST` | 3+ files |

**Total:** 15+ environment variables

---

### 2.2 Environment Variable Migration Strategy

**Phase 1: Support Both (Transition Period)**

In `src/infra/env.ts` (or wherever env vars are read):

```typescript
// Support both old and new environment variables
export function getGatewayToken(): string {
  // Try new BOBBOT_* first
  const newVar = process.env.BOBBOT_GATEWAY_TOKEN?.trim();
  if (newVar) return newVar;

  // Fall back to legacy CLAWDBOT_* with deprecation warning
  const oldVar = process.env.CLAWDBOT_GATEWAY_TOKEN?.trim();
  if (oldVar) {
    console.warn(
      "CLAWDBOT_GATEWAY_TOKEN is deprecated. Use BOBBOT_GATEWAY_TOKEN instead."
    );
    return oldVar;
  }

  throw new Error("BOBBOT_GATEWAY_TOKEN environment variable not set");
}
```

**Phase 2: Add Warning to All Env Var Access**

Create helper function:

```typescript
// src/infra/env-legacy.ts
const deprecatedVars = new Map<string, string>([
  ["CLAWDBOT_GATEWAY_TOKEN", "BOBBOT_GATEWAY_TOKEN"],
  ["CLAWDBOT_GATEWAY_PASSWORD", "BOBBOT_GATEWAY_PASSWORD"],
  ["CLAWDBOT_STATE_DIR", "BOBBOT_STATE_DIR"],
  // ... add all mappings
]);

export function getEnvWithDeprecationCheck(oldKey: string): string | undefined {
  const value = process.env[oldKey];
  if (value) {
    const newKey = deprecatedVars.get(oldKey);
    if (newKey) {
      console.warn(
        `[DEPRECATED] ${oldKey} is deprecated. Use ${newKey} instead.`
      );
    }
  }
  return value;
}
```

**Phase 3: Update All References**

Search pattern for each environment variable:

```bash
# Find all CLAWDBOT_GATEWAY_TOKEN references
grep -r "CLAWDBOT_GATEWAY_TOKEN" src/ --include="*.ts" --include="*.js"

# Replace with pattern (manually review each):
# process.env.CLAWDBOT_GATEWAY_TOKEN → getEnvWithDeprecationCheck("CLAWDBOT_GATEWAY_TOKEN")
# or just: process.env.BOBBOT_GATEWAY_TOKEN
```

**Action Items:**
- [ ] Create env var mapping table
- [ ] Implement legacy support helper
- [ ] Add deprecation warnings
- [ ] Update all 200+ file references
- [ ] Test that both old and new vars work

---

## Part 3: State Directory Migration (P0 - Critical)

### 3.1 Directory Path Changes

**Old:** `~/.clawdbot/`
**New:** `~/.bobbot/`

**Files Requiring Updates:**

```typescript
// src/daemon/paths.ts
export const STATE_DIR = ".bobbot";  // was: "clawdbot"

// src/daemon/paths.test.ts
// Update test expectations

// src/config/config-paths.ts
// Update default paths

// src/config/sessions/paths.ts
// Update session path logic
```

**Directory Structure:**

```
~/.bobbot/                        # New state directory
├── config.yaml                    # Gateway config
├── auth-profiles.json             # API keys
├── sessions/                       # Session transcripts
├── agents/                        # Agent workspace
│   └── workspace/                # Agent files
├── logs/                          # Log files
└── plugins/                       # Plugin data
```

---

### 3.2 Migration Script

**Create:** `scripts/migrate-state-dir.ts`

```typescript
import fs from "node:fs/promises";
import path from "node:path";

const OLD_DIR = path.join(process.env.HOME!, ".clawdbot");
const NEW_DIR = path.join(process.env.HOME!, ".bobbot");

export async function migrateStateDir(): Promise<boolean> {
  // Check if old directory exists
  try {
    await fs.access(OLD_DIR);
  } catch {
    // Old dir doesn't exist, nothing to migrate
    console.log("No old state directory found (already migrated or fresh install)");
    return false;
  }

  // Check if new directory already exists
  try {
    await fs.access(NEW_DIR);
    console.log("New state directory already exists, skipping migration");
    return false;
  } catch {
    // New dir doesn't exist, proceed with migration
  }

  console.log(`Migrating state directory: ${OLD_DIR} → ${NEW_DIR}`);

  // Create new directory
  await fs.mkdir(NEW_DIR, { recursive: true });

  // Move contents
  const entries = await fs.readdir(OLD_DIR, { withFileTypes: true });

  for (const entry of entries) {
    const oldPath = path.join(OLD_DIR, entry.name);
    const newPath = path.join(NEW_DIR, entry.name);

    if (entry.isDirectory()) {
      await fs.rename(oldPath, newPath);
    } else if (entry.isFile()) {
      await fs.rename(oldPath, newPath);
    }
  }

  // Remove old directory (should be empty now)
  await fs.rmdir(OLD_DIR);

  console.log("Migration complete!");
  return true;
}
```

**Call migration on startup:**

```typescript
// src/index.ts or src/entry.ts
import { migrateStateDir } from "./scripts/migrate-state-dir.js";

// Early in initialization
await migrateStateDir();
```

**Action Items:**
- [ ] Update state directory constant
- [ ] Create migration script
- [ ] Add migration call to entry point
- [ ] Test migration with existing data
- [ ] Document migration in README

---

## Part 4: Source Code Changes (P1 - High Priority)

### 4.1 Core Constants

**File:** `src/daemon/constants.ts`

**Changes:**

```typescript
// Old
export const BINARY_NAME = "moltbot";
export const CONFIG_DIR = ".clawdbot";
export const STATE_DIR = ".clawdbot";
export const ENV_PREFIX = "CLAWDBOT_";

// New
export const BINARY_NAME = "bobbot";
export const CONFIG_DIR = ".bobbot";
export const STATE_DIR = ".bobbot";
export const ENV_PREFIX = "BOBBOT_";
```

---

### 4.2 CLI Display Name

**File:** `src/cli/banner.ts` (if exists)

**Changes:**

```typescript
// Old
const BANNER = `
╔═══════════════════════════════════════════════════════════════
║           🦞 Moltbot — Personal AI Assistant          ║
║           EXFOLIATE! EXFOLIATE!                        ║
╚═══════════════════════════════════════════════════════════════
`;

// New
const BANNER = `
╔═══════════════════════════════════════════════════════════════
║           🤖 BobBot — Personal AI Assistant            ║
║           Your personal AI assistant                    ║
╚═══════════════════════════════════════════════════════════════
`;
```

---

### 4.3 Error Messages

**Pattern:** Search for hardcoded references

```bash
# Find error messages mentioning old brand
grep -r "moltbot\|Moltbot\|Clawdbot\|clawdbot" src/ --include="*.ts" | grep -i "error\|fail\|invalid"
```

**Example updates:**

```typescript
// Old
throw new Error(`Moltbot gateway is not running`);

// New
throw new Error(`BobBot gateway is not running`);
```

**Action Items:**
- [ ] Search for brand names in error messages
- [ ] Update user-facing error messages
- [ ] Update debug log messages
- [ ] Update exception messages
- [ ] Update test assertion messages

---

## Part 5: Documentation (P0 - Critical)

### 5.1 Root Documentation Files

**Files to Update:**

1. **README.md** - Main README
2. **CONTRIBUTING.md** - Contribution guidelines
3. **DEVELOPMENT_GUIDE.md** - Dev guide
4. **ARCHITECTURE.md** - Architecture docs
5. **API_REFERENCE.md** - API docs
6. **ADDON_DEVELOPMENT.md** - Add-on guide
7. **AGENTS.md** - Agent documentation

**Search and Replace Pattern:**

```bash
# For each documentation file
sed -i 's/Moltbot/BobBot/g' FILENAME.md
sed -i 's/moltbot/bobbot/g' FILENAME.md
sed -i 's/Clawdbot/BobBot/g' FILENAME.md
sed -i 's/clawdbot/bobbot/g' FILENAME.md
sed -i 's/🦞/🤖/g' FILENAME.md  # mascot emoji
```

**Content Updates:**

1. **Logo images:**
   - Remove: `docs/whatsapp-clawd.jpg`
   - Add: New BobBot logo (create/upload)

2. **Taglines:**
   - Remove: "EXFOLIATE! EXFOLIATE!"
   - Add: "Your personal AI assistant" or nothing

3. **Character references:**
   - Replace: "Clawd" → "Bob" or "Bob the Robot"
   - Remove lobster-themed references

**Action Items:**
- [ ] Update all root-level *.md files
- [ ] Replace logo images
- [ ] Update all product references
- [ ] Update badges and URLs
- [ ] Remove/replace taglines
- [ ] Check docs/ subdirectory

---

### 5.2 Documentation in docs/

**Subdirectories:**

```
docs/
├── platforms/
│   ├── macos/
│   └── linux/
├── REPO_STRUCTURE.md
├── CORE_ARCHITECTURE.md
├── DEVELOPMENT_GUIDE.md
└── ... (50+ files)
```

**Automated Update:**

```bash
# Update all markdown files
find docs/ -name "*.md" -type f -exec sed -i 's/Moltbot/BobBot/g' {} +
find docs/ -name "*.md" -type f -exec sed -i 's/moltbot/bobbot/g' {} +
find docs/ -name "*.md" -type f -exec sed -i 's/Clawdbot/BobBot/g' {} +
find docs/ -name "*.md" -type f -exec sed -i 's/clawdbot/bobbot/g' {} +
find docs/ -name "*.md" -type f -exec sed -i 's/🦞/🤖/g' {} +
```

**Manual Updates Needed:**

- [ ] Review all docs for mascot/character references
- [ ] Update architecture diagrams mentioning "Clawd"
- [ ] Update examples using "moltbot" command
- [ ] Update screenshots showing old UI

---

### 5.3 GitHub Templates

**Files:**

```
.github/
├── ISSUE_TEMPLATE/
│   ├── bug_report.md
│   ├── config.yml
│   └── feature_request.md
└── dependabot.yml
```

**Updates:**

- [ ] Update bug report template
- [ ] Update feature request template
- [ ] Update config.yml (if it mentions brand)
- [ ] Update dependabot.yml (if needed)

---

## Part 6: UI and Frontend (P1 - High Priority)

### 6.1 Web UI (ui/)

**Files to Update:**

```
ui/
├── index.html
├── src/ui/
│   ├── app.ts
│   ├── app-settings.ts
│   ├── app-channels.ts
│   ├── assistant-identity.ts
│   ├── device-auth.ts
│   ├── device-identity.ts
│   └── ... (100+ files)
└── package.json
```

**Automated Updates:**

```bash
# UI TypeScript files
find ui/src/ -name "*.ts" -o -name "*.tsx" -exec sed -i 's/Moltbot/BobBot/g' {} +
find ui/src/ -name "*.ts" -o -name "*.tsx" -exec sed -i 's/moltbot/bobbot/g' {} +
find ui/src/ -name "*.ts" -o -name "*.tsx" -exec sed -i 's/clawdbot/bobbot/g' {} +
```

**Manual Updates:**

1. **ui/index.html** - Update title
2. **ui/src/styles/base.css** - Update CSS if branded colors
3. **Package.json** - Update name/description
4. **UI Text Components:**
   - Header/title bars
   - Welcome messages
   - Help text
   - Error messages
   - Button labels

**Color Scheme (Optional):**

Current: Lobster-themed (reds, oranges?)
Consider: BobBot could use robot colors:
- Blues (tech, trustworthy)
- Grays (professional)
- Greens (friendly)
- Or keep original palette (colors work fine)

**Action Items:**
- [ ] Update all UI text
- [ ] Update HTML title
- [ ] Update package.json
- [ ] Review and update CSS (if branded colors)
- [ ] Update icons/images (if any)

---

### 6.2 Terminal UI (TUI)

**File:** `src/tui/tui.ts`

**Updates:**

```typescript
// Update display strings
const BANNER = "🤖 BobBot - Personal AI Assistant";
```

**Action Items:**
- [ ] Update TUI banner
- [ ] Update TUI help text
- [ ] Update command descriptions

---

## Part 7: Testing Infrastructure (P1 - Medium Priority)

### 7.1 Test Files

**Pattern:** 200+ test files contain brand references

**Search:**
```bash
# Find all test files with brand references
find test/ -name "*.test.ts" -o -name "*.e2e.test.ts" | xargs grep -l "moltbot\|clawdbot"
```

**Updates Needed:**

```typescript
// Old
describe("Moltbot gateway", () => {
  it("should start moltbot", () => {
    // ...
  });
});

// New
describe("BobBot gateway", () => {
  it("should start bobbot", () => {
    // ...
  });
});
```

**Action Items:**
- [ ] Update all test descriptions
- [ ] Update test fixture data
- [ ] Update test comments
- [ ] Update mock data
- [ ] Run tests to ensure they pass

---

### 7.2 Test Configuration

**Files:**

```
vitest.config.ts
vitest.e2e.config.ts
vitest.live.config.ts
```

**Updates:**
- [ ] Update config comments
- [ ] Update test directory names (if any)

---

## Part 8: Build and Tooling (P1 - Medium Priority)

### 8.1 Build Scripts

**Files:**

```
scripts/
├── build-docs-list.mjs
├── postinstall.js
├── run-node.mjs
└── ... (many scripts)
```

**Search:**
```bash
grep -r "moltbot\|clawdbot" scripts/
```

**Action Items:**
- [ ] Update script comments
- [ ] Update script output messages
- [ ] Update error messages
- [ ] Test all build scripts

---

### 8.2 CI/CD Configuration

**Files:**

```
.github/workflows/
├── ci.yml
├── install-smoke.yml
└── auto-response.yml
```

**Updates:**
- [ ] Update workflow names (optional)
- [ ] Update job names (optional)
- [ ] Update comments
- [ ] Update repository references (when forked)

---

### 8.3 Docker Configuration

**Files:**

```
Dockerfile
Dockerfile.sandbox-browser
render.yaml
```

**Updates:**
- [ ] Update image names (when building)
- [ ] Update comments
- [ ] Update container names

---

## Part 9: Extensions and Skills (P2 - Low Priority)

### 9.1 Extensions Directory

**Pattern:** Extensions may reference the brand

**Search:**
```bash
grep -r "moltbot\|clawdbot" extensions/
```

**Action Items:**
- [ ] Update extension README.md files
- [ ] Update extension descriptions
- [ ] Update code comments
- [ ] Test extension loading

---

### 9.2 Skills Directory

**Pattern:** Skills may reference the brand

**Search:**
```bash
grep -r "moltbot\|clawdbot" skills/
```

**Action Items:**
- [ ] Update skill README.md files
- [ ] Update skill descriptions
- [ ] Update tool descriptions
- [ ] Test skill execution

---

## Part 10: Compatibility Layer (P1 - Critical)

### 10.1 Binary Aliases

**Keep backward compatibility by supporting old binary names:**

**package.json:**
```json
{
  "bin": {
    "bobbot": "./bobbot.mjs",
    "moltbot": "./bobbot.mjs",  // legacy alias
    "clawdbot": "./bobbot.mjs"   // legacy alias with warning
  }
}
```

**Deprecation Warning in binary:**

```typescript
// In bobbot.mjs entry point
const LEGACY_BINARIES = ["moltbot", "clawdbot"];

if (LEGACY_BINARIES.includes(process.argv[1])) {
  console.warn(
    `\x1b[33m⚠️  WARNING: ${process.argv[1].toUpperCase()} is deprecated.\x1b[0m`
  );
  console.warn(
    `\x1b[33mPlease use 'bobbot' instead. This alias will be removed in a future version.\x1b[0m`
  );
  console.log(); // Blank line
  // Continue with normal execution
}
```

---

### 10.2 Environment Variable Compatibility

**Implementation:**

```typescript
// src/infra/env-compat.ts
export interface EnvConfig {
  // New (BobBot)
  gatewayToken?: string;
  gatewayPassword?: string;
  stateDir?: string;

  // Legacy (Moltbot/Clawdbot) - with deprecation
  CLAWDBOT_GATEWAY_TOKEN?: string;
  CLAWDBOT_GATEWAY_PASSWORD?: string;
  CLAWDBOT_STATE_DIR?: string;
}

export function resolveEnvConfig(): EnvConfig {
  return {
    // New vars
    gatewayToken:
      process.env.BOBBOT_GATEWAY_TOKEN ??
      process.env.CLAWDBOT_GATEWAY_TOKEN,
    gatewayPassword:
      process.env.BOBBOT_GATEWAY_PASSWORD ??
      process.env.CLAWDBOT_GATEWAY_PASSWORD,

    // State directory
    stateDir:
      process.env.BOBBOT_STATE_DIR ??
      process.env.CLAWDBOT_STATE_DIR,

    // Warn about legacy usage
    ...(process.env.CLAWDBOT_GATEWAY_TOKEN && {
      _legacy: true,
      _warning: "CLAWDBOT_* env vars are deprecated, use BOBBOT_*"
    })
  };
}
```

---

### 10.3 State Directory Migration

**Already covered in Part 3.2**

Ensures existing users don't lose their data.

---

## Part 11: Optional Aesthetic Changes (P2 - Optional)

### 11.1 Mascot and Emoji

**Current:** 🦞 (lobster)
**New:** 🤖 (robot)

**Files to update:**

```bash
# Find emoji usage
grep -r "🦞" . --include="*.md" --include="*.ts"

# Replace
find . -name "*.md" -type f -exec sed -i 's/🦞/🤖/g' {} +
```

**Note:** This is optional - you could keep no mascot or use something different.

---

### 11.2 Tagline/Catchphrase

**Current:** "EXFOLIATE! EXFOLIATE!" (lobster molting reference)
**New:** "Your personal AI assistant" (simple)

**Files:**
- README.md
- Any marketing materials
- CLI banner (src/cli/banner.ts)

**Action Items:**
- [ ] Remove "EXFOLIATE! EXFOLIATE!" from README
- [ ] Update CLI banner
- [ ] Remove from docs

---

### 11.3 Visual Assets

**Files to Replace:**

1. **docs/whatsapp-clawd.jpg** - Logo image
   - Replace with: New BobBot logo

2. **README-header.png** (if exists)
   - Replace with new header image

3. **Favicon** (if any)
   - ui/public/favicon.ico
   - ui/public/logo.png

4. **Screenshots** in documentation
   - Remove or retake with BobBot branding

**Action Items:**
- [ ] Create BobBot logo
- [ ] Replace all logo images
- [ ] Update favicons
- [ ] Update screenshots in docs
- [ ] Check assets in apps/macos/, apps/ios/, apps/android/

---

## Part 12: Repository Forking (When Ready)

### 12.1 Repository Name Options

**Option 1:** Keep as `moltbot` fork
- Your fork: `github.com/YOURUSER/moltbot`
- Pros: Familiar name, recognizable
- Cons: Still has "Moltbot" in URL

**Option 2:** Rename to `bobbobot`
- Your fork: `github.com/YOURUSER/bobbobot`
- Pros: Clean break, new identity
- Cons: Loses name recognition, breaks existing links

**Option 3:** Rebranded fork of moltbot
- Your fork: `github.com/YOURUSER/bobbobot`
- Docs: "A fork of moltbot, rebranded as BobBot"
- Pros: Clear attribution, new identity
- Cons: More complex description

**Recommendation:** Option 3 (rebranded fork)

---

### 12.2 Forking Process

**Step 1: Fork on GitHub**

1. Go to https://github.com/moltbot/moltbot
2. Click "Fork" button
3. Rename your fork to `bobbobot` (optional, under Settings)
4. Clone locally

**Step 2: Update Remote URLs**

```bash
# After cloning
cd bobbobot

# Update remote origin
git remote set-url origin git@github.com:YOURUSER/bobbobot.git

# Keep upstream as original moltbot (for updates)
git remote add upstream git@github.com:moltbot/moltbot.git
```

**Step 3: Update Repository References**

```bash
# Update all URLs in docs
find . -name "*.md" -type f -exec sed -i 's/github.com\\/moltbot\\/moltbot/github.com\\/YOURUSER\\/bobbobot/g' {} +
find . -name "*.md" -type f -exec sed -i 's/molt\\.bot/bob\\.bot/g' {} +

# Update package.json
# (handled in Part 1.1)
```

---

## Part 13: Domain Names (When Ready)

### 13.1 Primary Domain

**Current:** molt.bot
**New Options:**
- **bob.bot** (premium, if available)
- **bobbobot.dev**
- **bobbobot.io**
- **bobbobot.github.io** (free GitHub Pages)

### 13.2 Documentation Domain

**Current:** docs.molt.bot
**New:** bob.bot/docs or bobbot.github.io/docs

### 13.3 NPM Package

**Current:** `moltbot` (global npm package)
**New Options:**
- **bobbot** (global, if available)
- `@youruser/bobbot` (scoped)

**Action Items:**
- [ ] Check if "bobbot" is available on npm
- [ ] Register domain name (bob.bot or alternative)
- [ ] Set up GitHub Pages if using bobbot.github.io
- [ ] Update all URL references in docs
- [ ] Update npm package name

---

## Part 14: Implementation Phases

### Phase 1: Critical Changes (Week 1) ✅ REQUIRED

**Goal:** Make it build and run as BobBot

**Tasks:**
- [ ] Update package.json (name, bin, description)
- [ ] Rename CLI entry point (moltbot.mjs → bobbot.mjs)
- [ ] Update README.md (title, logo, badges, text)
- [ ] Implement environment variable compatibility (support both old and new)
- [ ] Implement state directory migration
- [ ] Update core constants
- [ ] Test that `bobbot` command works
- [ ] Test that old env vars still work (with warnings)
- [ ] Test migration from ~/.clawdbot/ to ~/.bobbot/

**Estimated Time:** 8-16 hours

---

### Phase 2: Code and Tests (Week 1-2) ✅ RECOMMENDED

**Goal:** Update all code references

**Tasks:**
- [ ] Update all 200+ source code files with brand references
- [ ] Update all test files (200+ tests)
- [ ] Update test configuration files
- [ ] Run full test suite and fix failures
- [ ] Update build scripts
- [ ] Update CI/CD configuration

**Estimated Time:** 40-80 hours

---

### Phase 3: Documentation and UI (Week 2) ✅ RECOMMENDED

**Goal:** Complete visual rebranding

**Tasks:**
- [ ] Update all documentation (docs/**/*.md)
- [ ] Update UI text and labels
- [ ] Update HTML title
- [ ] Create new logo
- [ ] Replace all images/assets
- [ ] Update GitHub templates
- [ ] Update CONTRIBUTING.md
- [ ] Update LICENSE (if mentions brand)

**Estimated Time:** 8-16 hours

---

### Phase 4: Polish and Cleanup (Week 2-3) ✅ OPTIONAL

**Goal:** Remove legacy references

**Tasks:**
- [ ] Replace emoji mascot (🦞 → 🤖)
- [ ] Update taglines
- [ ] Remove "EXFOLIATE! EXFOLIATE!"
- [ ] Update character references (Clawd → Bob)
- [ ] Review all code for remaining old brand references
- [ ] Add final deprecation notices
- [ ] Document backward compatibility

**Estimated Time:** 4-8 hours

---

## Part 15: Search and Replace Patterns

### 15.1 Sed Commands (Batch Updates)

**CAUTION:** Test on small subset first!

```bash
# Create backup
git checkout -b rebrand-backup

# Test pattern on one file
sed -i 's/Moltbot/BobBot/g' README.md

# If correct, apply to all:

# 1. Documentation
find . -name "*.md" -type f -exec sed -i 's/Moltbot/BobBot/g' {} +
find . -name "*.md" -type f -exec sed -i 's/moltbot/bobbot/g' {} +
find . -name "*.md" -type f -exec sed -i 's/Clawdbot/BobBot/g' {} +
find . -name "*.md" -type f -exec sed -i 's/clawdbot/bobbot/g' {} +

# 2. TypeScript/JavaScript
find src/ -name "*.ts" -o -name "*.js" -exec sed -i 's/Moltbot/BobBot/g' {} +
find src/ -name "*.ts" -o -name "*.js" -exec sed -i 's/moltbot/bobbot/g' {} +
find src/ -name "*.ts" -o -name "*.js" -exec sed -i 's/Clawdbot/BobBot/g' {} +
find src/ -name "*.ts" -o -name "*.js" -exec sed -i 's/clawdbot/bobbot/g' {} +

# 3. JSON files (be careful with formatting)
find . -name "package.json" -o -name "*.json" | \
  jq '.name |= gsub("moltbot"; "bobbot") | \
  jq '.description |= gsub("WhatsApp gateway CLI"; "Personal AI assistant") | \
  jq '.bin["bobbot"] |= .bin["moltbot"]' \
  # Apply to each file individually

# 4. HTML
find ui/ -name "*.html" -exec sed -i 's/Moltbot/BobBot/g' {} +

# 5. Emoji (optional)
find . -name "*.md" -type f -exec sed -i 's/🦞/🤖/g' {} +
```

---

### 15.2 Environment Variable Search Patterns

**Find all env var usages:**

```bash
# Find all CLAWDBOT_ env vars
grep -r "CLAWDBOT_" src/ --include="*.ts" --include="*.js" -h
```

**Replace pattern:**

```typescript
// Old
process.env.CLAWDBOT_GATEWAY_TOKEN

// New (with compatibility)
process.env.BOBBOT_GATEWAY_TOKEN || process.env.CLAWDBOT_GATEWAY_TOKEN
```

---

### 15.3 Comprehensive Replace Script

**Create:** `scripts/rebrand.sh`

```bash
#!/bin/bash
set -e

echo "🤖 BobBot Rebranding Script"
echo "================================="

# Dry run by default
DRY_RUN=${DRY_RUN:-true}

# Function to replace in files
replace_in_files() {
  local pattern="$1"
  local replacement="$2"
  shift 2
  local files=("$@")

  if [ "$DRY_RUN" = true ]; then
    echo "Would replace '$pattern' → '$replacement' in ${#files[@]} files"
  else
    echo "Replacing '$pattern' → '$replacement'..."
    find "${files[@]}" -type f -exec sed -i "s/$pattern/$replacement/g" {} +
  fi
}

# Documentation
echo "Updating documentation..."
replace_in_files "Moltbot" "BobBot" "*.md"
replace_in_files "moltbot" "bobbot" "*.md"
replace_in_files "Clawdbot" "BobBot" "*.md"
replace_in_files "clawdbot" "bobbot" "*.md"
replace_in_files "🦞" "🤖" "*.md"

# Source code
echo "Updating source code..."
replace_in_files "Moltbot" "BobBot" "src/**/*.ts" "src/**/*.js"
replace_in_files "moltbot" "bobbot" "src/**/*.ts" "src/**/*.js"
replace_in_files "Clawdbot" "BobBot" "src/**/*.ts" "src/**/*.js"
replace_in_files "clawdbot" "bobbot" "src/**/*.ts" "src/**/*.js"

# UI
echo "Updating UI..."
replace_in_files "Moltbot" "BobBot" "ui/**/*.{ts,tsx,html}"
replace_in_files "moltbot" "bobbot" "ui/**/*.{ts,tsx,html}"
replace_in_files "Clawdbot" "BobBot" "ui/**/*.{ts,tsx,html}"
replace_in_files "clawdbot" "bobbot" "ui/**/*.{ts,tsx,html}"

# Test files
echo "Updating tests..."
replace_in_files "moltbot" "bobbot" "test/**/*.{ts,js}"
replace_in_files "Clawdbot" "BobBot" "test/**/*.{ts,js}"
replace_in_files "clawdbot" "bobbot" "test/**/*.{ts,js}"

# Done
echo "================================="
echo "Rebranding complete!"
echo ""
echo "Next steps:"
echo "1. Review changes: git diff"
echo "2. Test: npm run build && npm test"
echo "3. Commit: git add . && git commit -m 'rebrand: Moltbot → BobBot'"
```

**Usage:**

```bash
# Dry run (safe)
bash scripts/rebrand.sh

# Actual changes
DRY_RUN=false bash scripts/rebrand.sh
```

---

## Part 16: Verification Checklist

### 16.1 Pre-Commit Verification

**Before committing rebranding:**

- [ ] All tests pass: `pnpm test`
- [ ] Build succeeds: `pnpm build`
- [ ] UI builds: `pnpm ui:build`
- [ ] CLI works: `node dist/bobbot.mjs --version`
- [ ] Old env vars work (with warnings)
- [ ] New env vars work
- [ ] Migration from ~/.clawdbot/ to ~/.bobbot/ works

---

### 16.2 Post-Commit Verification

**After committing rebranding:**

- [ ] Clean install works: `pnpm install && pnpm build`
- [ ] Fresh install (no old state) works
- [ ] Upgrade from old version works
- [ ] All documentation is updated
- [ ] No broken links
- [ ] No old brand references in UI
- [ ] No old brand references in error messages
- [ ] Backward compatibility maintained

---

## Part 17: Migration Guide for Users

### 17.1 User Migration Announcement

**Create:** `docs/MIGRATION_GUIDE.md`

```markdown
# Moltbot → BobBot Migration Guide

## What Changed?

We've rebranded from Moltbot to BobBot. Here's what you need to know.

## What You Need To Do

### 1. Update Your Installation

```bash
# Uninstall old version
npm uninstall -g moltbot

# Install new version
npm install -g bobbot
```

### 2. Update Environment Variables

If you have environment variables set:

**Old (Deprecated):**
```bash
export CLAWDBOT_GATEWAY_TOKEN="your-token"
```

**New (Use this):**
```bash
export BOBBOT_GATEWAY_TOKEN="your-token"
```

**Transition Period:** Both work for now, but old will be removed in v0.2.

### 3. Update State Directory (Automatic)

Your config and data automatically migrate from:
- `~/.clawdbot/` → `~/.bobbot/`

This happens on first run - no action needed!

### 4. Update Binary Name

**Old commands:**
```bash
moltbot --version
clawdbot --version
```

**New command:**
```bash
bobbot --version
```

**Legacy aliases:** `moltbot` and `clawdbot` still work but show deprecation warning.

## Checklist

- [ ] Uninstall old Moltbot package
- [ ] Install BobBot package
- [ ] Update environment variables
- [ ] Update scripts/cron jobs
- [ ] Update binary name in commands
- [ ] Test that everything works

## Need Help?

See: [Troubleshooting](#troubleshooting)
```

---

### 17.2 Breaking Changes Documentation

**Create:** `docs/BREAKING_CHANGES.md`

```markdown
# Breaking Changes: Moltbot → BobBot

## Version 2026.2.0

### Environment Variables

| Old | New | Status |
|-----|-----|--------|
| `CLAWDBOT_GATEWAY_TOKEN` | `BOBBOT_GATEWAY_TOKEN` | Deprecated |
| `CLAWDBOT_GATEWAY_PASSWORD` | `BOBBOT_GATEWAY_PASSWORD` | Deprecated |
| `CLAWDBOT_STATE_DIR` | `BOBBOT_STATE_DIR` | Deprecated |

### Binary Names

| Old | New | Status |
|-----|-----|--------|
| `moltbot` | `bobbot` | Deprecated (alias) |
| `clawdbot` | `bobbot` | Deprecated (alias) |

### State Directory

| Old | New | Migration |
|-----|-----|----------|
| `~/.clawdbot/` | `~/.bobbot/` | Automatic |

## Migration Timeline

- **v2026.1.x:** Both old and new supported
- **v2026.2.0:** Old marked deprecated
- **v2027.1.0:** Old removed (breaking change)
```

---

## Part 18: Troubleshooting

### 18.1 Common Issues

**Issue:** Tests fail after rebranding

**Solution:**
```bash
# Clear cache
rm -rf node_modules/ dist/
pnpm install
pnpm build

# Run tests
pnpm test
```

**Issue:** Build fails

**Solution:**
```bash
# Clear build artifacts
rm -rf dist/
pnpm build
```

**Issue:** Environment variables not recognized

**Solution:**
```bash
# Check if new env vars are set
echo $BOBBOT_GATEWAY_TOKEN

# Try with old env var (for compatibility)
echo $CLAWDBOT_GATEWAY_TOKEN
```

**Issue:** State directory migration fails

**Solution:**
```bash
# Manually migrate
mv ~/.clawdbot ~/.bobbot

# Or start fresh
rm -rf ~/.clawdbot/
```

---

## Part 19: Rollback Plan

### 19.1 If Rebranding Goes Wrong

**Emergency Rollback:**

```bash
# Revert to pre-rebrand commit
git revert HEAD~1..HEAD

# Tag as pre-rebrand
git tag pre-rebrand-backup

# Force push
git push --force
```

---

## Part 20: Summary

### Files to Change: 2,169

**Breakdown by Priority:**

| Priority | Files | Time | Block |
|----------|-------|------|-------|
| **Critical** | 10 | 1-2h | Core identity |
| **High** | 200 | 8-16h | Docs, UI, tests |
| **Medium** | 800 | 40-80h | Code, config |
| **Low** | 1,000+ | 20-40h | Comments, internal |
| **Optional** | 100 | 4-8h | Aesthetics |
| **Compat** | 50 | 8-16h | Migration layer |

**Total:** 81-162 hours (2-4 weeks)

---

### Key Rebranding Elements

| Element | Old → New | Impact |
|---------|----------|--------|
| **Name** | Moltbot → BobBot | High |
| **Package** | moltbot → bobbot | High |
| **Binary** | moltbot → bobbot | High |
| **Env Vars** | CLAWDBOT_ → BOBBOT_ | High |
| **Directory** | ~/.clawdbot/ → ~/.bobbot/ | High |
| **Mascot** | 🦞 → 🤖 | Medium |
| **Tagline** | "EXFOLIATE!" → "Your assistant" | Low |
| **Character** | Clawd → Bob | Low |

---

### Execution Strategy

**Recommended Approach:**

1. **Phase 1 (Week 1):** Critical changes + compatibility layer
2. **Phase 2 (Week 1-2):** Code and test updates
3. **Phase 3 (Week 2):** Documentation and UI
4. **Phase 4 (Week 2-3):** Polish and cleanup (optional)

**With automation scripts:** Can complete in 1-2 weeks

---

### Success Criteria

Rebranding is successful when:

✅ `bobbot` command works
✅ `bobbot --help` shows BobBot branding
✅ Old `moltbot` and `clawdbot` commands still work (with warnings)
✅ Environment variables `BOBBOT_*` work
✅ Old environment variables `CLAWDBOT_*` still work (with warnings)
✅ State directory is `~/.bobbot/`
✅ Old configs automatically migrate from `~/.clawdbot/`
✅ All tests pass
✅ Documentation shows BobBot branding
✅ UI shows BobBot branding
✅ No references to "Moltbot" or "Clawd" remain in user-facing elements

---

**This document provides a complete, step-by-step guide for rebranding Moltbot to BobBot while maintaining backward compatibility and ensuring no user data is lost.**
