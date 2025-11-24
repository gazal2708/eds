# BYOM Setup - Step by Step Guide

This guide shows how to set up BYOM (Bring Your Own Markup) for Edge Delivery Services when you have an existing site with Configuration Service enabled.

## Prerequisites

- ✅ Site already exists with Configuration Service
- ✅ AEM Code Sync app installed on GitHub repository
- ✅ Adobe LDAP account access to admin.hlx.page

## Setup Steps

### Step 1: Create Your HTML File

Create an HTML file in your repo root with proper EDS semantic structure:

```bash
# File: byom-demo.html
# Must follow EDS block structure with <header>, <main>, <footer>
# Include blocks like: hero, cards, columns, etc.
```

**Important**: File name must match the URL path (e.g., `/byom-demo` → `byom-demo.html`)

### Step 2: Push HTML File to GitHub

Commit and push your HTML file to GitHub main branch:

```bash
git add byom-demo.html
git commit -m "Add BYOM demo page"
git push
```

**Note**: You do NOT need a matching Google Doc in Drive. BYOM overlay works independently based on the configuration.

### Step 3: Login to Admin Portal

Open browser and login to: `https://admin.hlx.page/config/{org}/sites/{site}.json`

Use your Adobe LDAP credentials.

### Step 4: Configure BYOM (Browser Console)

Open **browser DevTools Console** (F12) in the admin portal login window and run:

**Choose your configuration:**

**Option A: BYOM Only (Recommended for new projects)**
```javascript
fetch('https://admin.hlx.page/config/{org}/sites/{site}/content.json', {
  method: 'POST',
  headers: {'content-type': 'application/json'},
  credentials: 'include',
  body: JSON.stringify({
    "source": {
      "url": "https://raw.githubusercontent.com/{org}/{repo}/main/",
      "type": "markup",
      "suffix": ".html"
    }
  })
})
.then(r => r.json())
.then(data => console.log('Success:', data));
```

**Option B: Hybrid (Google Drive + BYOM Overlay)**
Use this ONLY if you need to keep existing Google Drive content and selectively override specific pages with BYOM.

```javascript
fetch('https://admin.hlx.page/config/{org}/sites/{site}/content.json', {
  method: 'POST',
  headers: {'content-type': 'application/json'},
  credentials: 'include',
  body: JSON.stringify({
    "contentBusId": "YOUR_CONTENT_BUS_ID",  // Get from existing config
    "source": {
      "url": "https://drive.google.com/drive/folders/YOUR_FOLDER_ID",
      "type": "google",
      "id": "YOUR_FOLDER_ID"
    },
    "overlay": {
      "url": "https://raw.githubusercontent.com/{org}/{repo}/main/",
      "type": "markup",
      "suffix": ".html"
    }
  })
})
.then(r => r.json())
.then(data => console.log('Success:', data));
```

**When to use Option B (Hybrid)?**
- You have existing content in Google Drive that authors still use
- You want to override ONLY specific pages with custom HTML (e.g., product catalog, landing pages)
- Authors continue using Google Docs for most content, devs add BYOM for specialized pages

**Example:** Homepage, about, blog → Google Docs | Product pages, dashboards → BYOM HTML

**Replace**: `{org}`, `{site}`, `YOUR_CONTENT_BUS_ID`, `YOUR_FOLDER_ID`

### Step 5: Preview Your Page (REQUIRED)

In the browser console, run the preview API call for your page:

```javascript
fetch('https://admin.hlx.page/preview/{org}/{repo}/main/byom-demo', {
  method: 'POST',
  credentials: 'include'
})
.then(r => r.json())
.then(result => console.log('✅ Page previewed:', result));
```

**⚠️ Critical**: The preview call is REQUIRED - without it, changes won't reflect even after overlay configuration.

### Step 6: Test Your BYOM Page

Wait 5-10 seconds, then visit: `https://main--{site}--{org}.aem.page/{your-path}`

You should see the HTML content from your `.html` file, not the Google Doc!

## Verification

To verify BYOM is working:

```javascript
// Check config has overlay
fetch('https://admin.hlx.page/config/{org}/sites/{site}/content.json', {
  credentials: 'include'
})
.then(r => r.json())
.then(data => console.log('Has overlay:', !!data.overlay));
```

Should return: `Has overlay: true`

## Quick Reference

### Workflows Summary

**Enable BYOM:**
```bash
# 1. Create & push HTML
git add page.html && git commit -m "Add page" && git push

# 2. Configure overlay (browser console at admin.hlx.page)
POST /config/{org}/sites/{site}/content.json + overlay object

# 3. Preview page (browser console)
POST /preview/{org}/{repo}/main/page

# 4. Visit: https://main--{site}--{org}.aem.page/page
```

**Add more pages:**
```bash
# 1. Push new HTML files
# 2. Reconfigure overlay (same config)
# 3. Preview each new page
```

**Remove overlay from specific page:**
```bash
# Option 1: Delete HTML file + reconfigure + preview
# Option 2: Remove overlay object from config + preview
```

**Critical Notes:**
- File name = URL path (`products.html` → `/products`)
- Preview API call REQUIRED (code sync NOT needed)
- Use browser console (terminal auth won't work for LDAP)
- No Google Docs needed

**Troubleshooting:**
- **404**: File name mismatch or missing `.html` suffix in config
- **Old content**: Re-run preview call, hard refresh
- **401/403**: Login to admin.hlx.page first

---

## Adding More HTML Files

```bash
# Push new files
git add products.html about.html && git commit -m "Add pages" && git push
```

```javascript
// 1. Reconfigure overlay (refreshes file list)
fetch('https://admin.hlx.page/config/{org}/sites/{site}/content.json', {
  method: 'POST',
  headers: {'content-type': 'application/json'},
  credentials: 'include',
  body: JSON.stringify({
    "contentBusId": "YOUR_CONTENT_BUS_ID",
    "source": {
      "url": "https://drive.google.com/drive/folders/YOUR_FOLDER_ID",
      "type": "google",
      "id": "YOUR_FOLDER_ID"
    },
    "overlay": {
      "url": "https://raw.githubusercontent.com/{org}/{repo}/main/",
      "type": "markup",
      "suffix": ".html"
    }
  })
})
.then(r => r.json())
.then(data => console.log('Overlay reconfigured:', data));

// 2. Preview each page
fetch('https://admin.hlx.page/preview/{org}/{repo}/main/products', {
  method: 'POST',
  credentials: 'include'
})
.then(r => r.json())
.then(result => console.log('Previewed:', result));
```

## Overlaying Homepage (/)

Create `index.html`, push to GitHub, then:

```javascript
// 1. Reconfigure overlay
// (same config as above)

// 2. Preview homepage
fetch('https://admin.hlx.page/preview/{org}/{repo}/main/', {
  method: 'POST',
  credentials: 'include'
})
.then(r => r.json())
.then(result => console.log('Homepage previewed:', result));
```

## Removing BYOM Overlay

**Remove overlay from all pages:**
```javascript
// POST content.json without overlay object
fetch('https://admin.hlx.page/config/{org}/sites/{site}/content.json', {
  method: 'POST',
  headers: {'content-type': 'application/json'},
  credentials: 'include',
  body: JSON.stringify({
    "contentBusId": "YOUR_CONTENT_BUS_ID",
    "source": {
      "url": "https://drive.google.com/drive/folders/YOUR_FOLDER_ID",
      "type": "google",
      "id": "YOUR_FOLDER_ID"
    }
    // No overlay = BYOM removed
  })
})
.then(r => r.json())
.then(data => console.log('Overlay removed:', data));

// Preview each affected page
fetch('https://admin.hlx.page/preview/{org}/{repo}/main/page', {
  method: 'POST',
  credentials: 'include'
})
.then(r => r.json())
.then(result => console.log('Now using Google Drive:', result));
```

**Remove overlay from specific page only:**
```bash
# Delete HTML file + reconfigure overlay + preview
git rm page.html && git commit -m "Remove BYOM" && git push
# Then run reconfigure overlay + preview (see "Adding More HTML Files")
```

**Note:** BYOM checks GitHub first; if HTML doesn't exist, it falls back to Google Drive.

