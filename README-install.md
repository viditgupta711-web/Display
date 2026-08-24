# Turning Display into an installable app

Drop these files into the root of the `Display` repo (same folder as `index.html`):

```
index.html          <- edit, see below
manifest.json
sw.js
icon-192.png
icon-512.png
icon-maskable-192.png
icon-maskable-512.png
```

## 1. Edit index.html

Add inside `<head>`, near your existing meta tags:

```html
<link rel="manifest" href="./manifest.json">
<link rel="apple-touch-icon" href="./icon-192.png">
```

Add just before `</body>`:

```html
<script>
  // Register the service worker — this is what makes Chrome offer "Install app".
  if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
      navigator.serviceWorker.register('./sw.js').catch(console.error);
    });
  }

  // Keep the screen awake while the app is open (Chrome on Android supports this).
  let wakeLock = null;
  async function keepAwake() {
    try {
      if ('wakeLock' in navigator) wakeLock = await navigator.wakeLock.request('screen');
    } catch (e) { /* battery saver can refuse this — not fatal */ }
  }
  document.addEventListener('visibilitychange', () => {
    if (document.visibilityState === 'visible') keepAwake();
  });
  keepAwake();
</script>
```

## 2. Push and wait

Commit and push. GitHub Pages takes 30–60 seconds to redeploy.

## 3. Install on the OnePlus 8 Pro

1. Open Chrome (not Fully Kiosk, not the Samsung browser) and go to
   `https://viditgupta711-web.github.io/Display/`
2. Pull down to hard-refresh once, so the new service worker registers.
3. Tap the ⋮ menu. You want **Install app**. If it only says *Add to home screen*,
   something in step 1 is missing — check the console via
   `chrome://inspect` from a desktop, or just re-check the file paths.
4. Tap Install. It lands in the app drawer with the clock icon and opens
   full-screen with no browser chrome.

## 4. OnePlus-specific settings worth changing

- **Settings → Battery → Battery optimisation → Clock → Don't optimise.**
  OxygenOS is aggressive; without this it will get killed in the background.
- **Settings → Display → Sleep** — set long, or rely on the wake-lock script above.
- If you want it as a bedside display, turn on **Dark mode** and drop brightness
  to minimum; the page is already black so the OLED pixels stay off.

## Updating later

Change `CACHE_VERSION` in `sw.js` (e.g. `clock-v2`) whenever you edit `index.html`.
Otherwise the installed app may keep serving the old cached page.
