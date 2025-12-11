# Golsie Music Website Scripts

Complete JavaScript for the Golsie music website with hybrid GitHub + Webflow deployment.

## 🏗️ Architecture

This project uses a **hybrid deployment strategy**:
- **Primary:** GitHub CDN (fast, unlimited size, easy updates)
- **Fallback:** Webflow-hosted (reliable, always available)
- **Auto-switching:** Seamless fallback if GitHub fails

## 📦 Repository Files

### Main Script
- **golsie-full.js** - Complete unminified script (58KB, hosted on GitHub)

### Webflow Integration Files
- **webflow-loader.html** - CDN loader script (goes in Custom Code → Head)
- **webflow-fallback-minified.html** - Minified fallback (goes in Custom Code → Footer)
- **IMPLEMENTATION-GUIDE.md** - Complete setup instructions

## 🚀 Deployment

### GitHub CDN (Primary)
The site loads from jsDelivr CDN with automatic fallback chain:

**CDN URL:**
```
https://cdn.jsdelivr.net/gh/Shahaf1/golsie-webflow-website@v100/golsie-full.js
```

**Raw GitHub URL (fallback):**
```
https://raw.githubusercontent.com/Shahaf1/golsie-webflow-website/main/golsie-full.js
```

**Benefits:**
- ⚡ Fast global delivery (~50-200ms from CDN)
- 📦 No file size limits
- 🔄 Easy updates (just push to GitHub)
- 🌍 99.9% uptime
- 💾 Automatic caching
- 🛡️ Automatic fallback to raw GitHub if CDN fails
- 🎯 Version control with tags

---

### Loader Configuration

The loader in **Webflow Custom Code → Head** controls all loading logic:

```javascript
var GITHUB_TAG = 'v100'; // Version tag (change to force refresh)
var TIMEOUT = 5000;      // CDN timeout (5 seconds)
var FALLBACK_ENABLED = true; // Enable Webflow fallback

// Code-level force options (for permanent overrides)
var CODE_FORCE_RAW_GITHUB = false; // Always use raw GitHub
var CODE_FORCE_FALLBACK = false;   // Always use Webflow fallback
```

**Loading sequence:**
1. Try jsDelivr CDN (`@v100` tag)
2. On failure/timeout → Try raw GitHub
3. On GitHub failure → Load Webflow fallback

---

### Webflow Fallback
If both GitHub sources fail, the site automatically loads a minified version from Webflow's servers (Custom Code → Footer).

**Fallback triggers:**
- CDN network error or timeout
- Raw GitHub fetch failure
- Script parsing error
- Manual force via `?fallback` parameter

## 🧪 Testing & Loading Modes

The loader supports **3 loading strategies** with automatic fallback:

### 1. **Normal Mode (CDN Primary)** ⭐ Recommended
```
https://golsie-music.design.webflow.com
```
**Behavior:**
1. Attempts jsDelivr CDN (fast, cached)
2. Falls back to raw GitHub on CDN failure
3. Falls back to Webflow on GitHub failure

**Body class:** `git-js`  
**Speed:** ~50-200ms (CDN cached)

---

### 2. **Raw GitHub Mode** (Bypass CDN)
```
https://golsie-music.design.webflow.com?raw
```
**Behavior:**
- Skips jsDelivr CDN entirely
- Fetches directly from GitHub raw URL
- Instant updates (no CDN cache wait)
- Use for immediate testing after Git push

**Body class:** `forcegit-js`  
**Speed:** ~500-2000ms (varies by location)  
**Use case:** Testing new code immediately after pushing to GitHub

---

### 3. **Fallback Mode** (Webflow Only)
```
https://golsie-music.design.webflow.com?fallback
```
**Behavior:**
- Skips GitHub entirely
- Loads Webflow-hosted version only
- Tests fallback system

**Body class:** `webflow-js`  
**Speed:** ~100-300ms  
**Use case:** Testing fallback functionality

---

## 🔧 Code-Level Force Options

Edit the loader script to permanently force a specific mode:

```javascript
var CODE_FORCE_RAW_GITHUB = false; // Set true: Always use raw GitHub
var CODE_FORCE_FALLBACK = false;   // Set true: Always use Webflow
```

---

## 📊 Loading Priority & Fallback Chain

```
┌─────────────────────────────────────────┐
│  NORMAL MODE (Default)                   │
└─────────────────────────────────────────┘
         │
         ↓
┌─────────────────────────────────────────┐
│  1. jsDelivr CDN                         │
│     cdn.jsdelivr.net/gh/Shahaf1/...     │
│     • Fast (50-200ms)                    │
│     • Cached globally                    │
│     • Body class: git-js                 │
└─────────────────────────────────────────┘
         │
         │ (on timeout/error)
         ↓
┌─────────────────────────────────────────┐
│  2. GitHub Raw                           │
│     raw.githubusercontent.com/...        │
│     • Direct from GitHub                 │
│     • No cache delay                     │
│     • Body class: forcegit-js            │
└─────────────────────────────────────────┘
         │
         │ (on fetch/parse error)
         ↓
┌─────────────────────────────────────────┐
│  3. Webflow Fallback                     │
│     Embedded in Footer                   │
│     • Always reliable                    │
│     • Full functionality                 │
│     • Body class: webflow-js             │
└─────────────────────────────────────────┘
```

---

## 🎯 Body Classes

The loader automatically adds classes to track which version loaded:

| Class | Source | When Used | Speed |
|-------|--------|-----------|-------|
| `.git-js` | jsDelivr CDN | Normal load (99%+ traffic) | ~50-200ms |
| `.forcegit-js` | GitHub Raw | CDN failed OR `?raw` param | ~500-2000ms |
| `.webflow-js` | Webflow Fallback | All GitHub failed OR `?fallback` param | ~100-300ms |

Use these for debugging or conditional styling:
```css
/* CDN Active (most common) */
.git-js .debug-badge::before {
  content: "CDN Active";
  background: #00ff00;
}

/* Raw GitHub Active (testing/CDN failed) */
.forcegit-js .debug-badge::before {
  content: "Raw GitHub Active";
  background: #ffaa00;
}

/* Webflow Fallback Active (rare) */
.webflow-js .debug-badge::before {
  content: "Fallback Active";
  background: #ff0000;
}
```

**Use cases:**
- Show loading indicator only on slow raw GitHub loads
- Display "testing mode" banner when `?raw` is used
- Track which version users are loading in analytics

## 🔄 Updating the Script

### Quick Updates (No Webflow Changes Required)

**Standard update flow:**
1. Edit `golsie-full.js` locally
2. Commit and push to GitHub
3. **Update version tag in loader** (see below)
4. Changes live in 1-5 minutes (CDN cache refresh)
5. **No Webflow republish needed!**

**For instant testing:**
- Use `?raw` parameter to bypass CDN cache
- Example: `yoursite.com?raw` loads directly from GitHub

---

### Version Tagging System

The loader uses **version tags** instead of `@main` for better cache control:

**In `webflow-loader.html`:**
```javascript
var GITHUB_TAG = 'v100'; // Change this to force CDN refresh
```

**How it works:**
- `@v100` → First version (initial deployment)
- `@v101` → After first update
- `@v102` → After second update
- etc.

**When to update the tag:**
1. Push changes to GitHub
2. Edit loader in Webflow Head
3. Change `GITHUB_TAG` from `'v100'` to `'v101'`
4. Publish Webflow site
5. CDN immediately fetches new version

---

### Git Release Tags (Optional)

For production releases, you can also use Git tags:

```bash
# Tag a release
git tag v1.0.1
git push origin v1.0.1

# Then update loader to:
var GITHUB_TAG = 'v1.0.1';
```

**Advantage:** Git tag = permanent reference  
**Disadvantage:** Requires Git knowledge

**Simple version numbers (`v100`, `v101`) are recommended** for most users.

---

### CDN Cache Behavior

| Branch/Tag | Cache Duration | Update Speed |
|------------|---------------|--------------|
| `@main` | 24 hours | ❌ Very slow (up to 24hrs) |
| `@v100`, `@v101`, etc. | 7 days | ✅ Instant (change tag = new URL) |
| `@commit-hash` | Permanent | ✅ Instant (new hash = new URL) |

**⚠️ Never use `@main` in production** - cache delays are too long!

## ✨ Features

### Core Systems
- Mobile menu with scroll lock
- Navigation active state detection (with parent page support)
- Background video keep-alive system
- Hash URL cleanup

### Product Gallery
- Swipe navigation (mobile)
- Keyboard navigation (desktop)
- Arrow controls
- Thumbnail selection
- Image preloading with fade transitions

### Shows System (Bandsintown API)
- Live show data fetching
- Client-side caching (2 API calls per session)
- Filter system: All / Golsie / Other / Previous
- Address toggle with Google Maps integration
- Description popup modal
- Date/time formatting

### Modal System
- Songlink integration (multi-platform music links)
- YouTube video embeds
- Spotify oEmbed with iOS fallback
- Loading states with smooth transitions
- Glassmorphic backdrop with animations

### Browser Compatibility
- iOS Safari backdrop-filter fixes
- Mobile touch event handling
- Passive event listeners
- Cross-browser transitions

## 📊 File Sizes

| File | Size | Location | Limit |
|------|------|----------|-------|
| golsie-full.js | ~58KB | GitHub | None |
| webflow-loader.html | ~2KB | Webflow Head | 50KB |
| webflow-fallback-minified.html | ~33KB | Webflow Footer | 50KB |
| **Total in Webflow** | **~35KB** | **Both sections** | **100KB** |

✅ **Well under Webflow's limits with 65KB headroom**

## 🛠️ Configuration

All settings centralized in `Config` object:

```javascript
var Config = {
  // Timing
  menuAnimationDuration: 250,
  modalAnimationDuration: 300,
  modalLoadingMinTime: 700,
  
  // Bandsintown API
  bandsintownApiKey: '43aa4e2b7dc992c34f1bf8503a4d9667',
  bandsintownArtistName: 'Golsie',
  showsMaxDisplay: 100,
  showsDefaultFilter: 'all',
  
  // Gallery
  galleryComponentLoadDelay: 500
};
```

## 🌐 Public APIs

Exposed global objects for external control:

```javascript
// Modals
window.GolsieModals.open('songlink', {...});
window.GolsieModals.close();
window.GolsieModals.isOpen();

// Menu
window.GolsieMenu.open();
window.GolsieMenu.close();
window.GolsieMenu.isOpen();

// Gallery
window.GolsieGallery.next();
window.GolsieGallery.prev();
window.GolsieGallery.goTo(index);

// Navigation
window.GolsieNav.update();

// Shows
window.GolsieShows.refresh();
window.GolsieShows.getShows();
```

## 📝 Version

**Current:** 1.0.0  
**Last Updated:** December 10, 2025  
**Repository:** https://github.com/Shahaf1/golsie-webflow-website

## 🔗 Links

- **Production Site:** https://golsie-music.design.webflow.com
- **GitHub Repo:** https://github.com/Shahaf1/golsie-webflow-website
- **CDN URL:** https://cdn.jsdelivr.net/gh/Shahaf1/golsie-webflow-website@main/golsie-full.js

## 📄 License

Proprietary - Golsie Music  
© 2025 All Rights Reserved
