# Miro Coords Panel

A tiny, zero-build Web SDK app for **Miro** that shows the **center coordinates** of the currently selected object and generates a **shareable URL** to your website page that embeds a Miro board, centered on that object.

- No servers.  
- Just two static files (`index.html`, `panel.html`).  
- Host on **GitHub Pages** (free) and wire it into Miro via the Developer Dashboard.

---

## ✨ What it does

- Displays the selected item’s **center `{x, y}`**, size, and rotation.
- Lets you paste your **website page URL** (the page that embeds your Miro board in an `<iframe>`).
- Auto-generates a **deep link** of the form:

  ```
  https://your-site/page?cx=<x>&cy=<y>&z=1
  ```

  where:
  - `cx`, `cy` are the center coordinates (board units)
  - `z` is a **zoom factor** (defaults to `1`, you can edit after copying)

> Pair this with your embedded-iframe page that reads `cx`, `cy`, `z` and sets Miro’s `moveToViewport` accordingly (sample code included below).

---

## 🗂 Repo contents

```
miro-coords-panel/
├─ index.html   # App entry: registers toolbar icon, opens the panel
└─ panel.html   # Panel UI: shows coordinates, builds share links
```

No bundlers, no frameworks, no external deps. The `miro` object is provided by Miro when your app runs **inside** a board.

---

## ✅ Prerequisites

- A Miro account with a **Developer Team** (or a team where you can install an app).
- Permission to edit boards where you’ll use this app.
- A GitHub account (for GitHub Pages hosting).

---

## 🚀 Quick start (5–10 minutes)

### 1) Enable GitHub Pages (hosting)
1. Create a new public repo (e.g., `miro-coords-panel`).
2. Add `index.html` and `panel.html` (see files in this repo).
3. In the repo: **Settings → Pages → Build and deployment**
   - Source: **Deploy from a branch**
   - Branch: **main**; Folder: **/** (root)
   - Click **Save**
4. After a short delay, you’ll get a URL like:
   ```
   https://<your-user>.github.io/miro-coords-panel/
   ```
   Your entrypoint will be:
   ```
   https://<your-user>.github.io/miro-coords-panel/index.html
   ```

### 2) Create a Miro Web SDK app
1. Go to the **Miro Developer Dashboard** and **Create new app**.
2. **App type:** Web SDK (no OAuth required).
3. **Scopes:**  
   - `boards:read` (required to read selection)  
   - `boards:write` (optional; needed only if you later add “drop a dot”, etc.)
4. **App URL / sdkUri:**  
   ```
   https://<your-user>.github.io/miro-coords-panel/index.html
   ```
5. Give the app a name (e.g., “Coords Panel”) and icon.  
6. **Install** the app to your team.

### 3) Use it on a board
- Open a board in the team where the app is installed.  
- Open **Apps** (puzzle icon) → click **Coords Panel** (first time registers a toolbar icon).  
- Click the toolbar icon to open the panel.  
- Select any object to see coordinates and generate a link.

---

## 🧩 How the panel works

- When you select an item, the panel reads:
  - `x`, `y` (item center, board units)
  - `width`, `height`
  - `rotation`
- Paste your website page URL (the page that **embeds** your Miro board in an `<iframe>`).
- The panel builds a shareable link:
  ```
  <your-page>?cx=<x>&cy=<y>&z=1
  ```
- Copy and share that link.

---

## 🔗 Example share URLs

Using your example page:

- Default zoom (z omitted → your embed defaults it to 1):
  ```
  https://parkerthatch.com/pages/pt-mood-board?cx=1200&cy=300
  ```

- Zoom in 1.5×:
  ```
  https://parkerthatch.com/pages/pt-mood-board?cx=500&cy=-250&z=1.5
  ```

- Zoom out 0.5×:
  ```
  https://parkerthatch.com/pages/pt-mood-board?cx=0&cy=0&z=0.5
  ```

---

## 🧱 Your embedded page (iframe) — sample

On your site page that hosts the Miro **live-embed** iframe, read `cx, cy, z` and compute `moveToViewport`. Keep your baseline viewport (board units) as the “zoom=1” size you like (from your current embed). Example:

```html
<!-- Container styles (your original approach) -->
<style>
  .miro-responsive-container { position:relative; width:100%; height:60vh; overflow:hidden; }
  .miro-responsive-container iframe { position:absolute; inset:0; width:100%; height:100%; border:0; }
</style>

<div class="miro-responsive-container">
  <iframe id="miro-frame" allow="fullscreen; clipboard-read; clipboard-write" allowfullscreen></iframe>
</div>

<script>
  // Base embed URL (your board, view-only UI)
  const BOARD_BASE = "https://miro.com/app/live-embed/uXjVJMVXY-s=/?embedMode=view_only_without_ui";
  const EMBED_ID   = "984083661127";

  // Baseline viewport at z=1 (from your previous embed: -712,-643,1455,1322)
  const BASE_W = 1455;
  const BASE_H = 1322;

  function getParams() {
    const q = new URLSearchParams(location.search);
    const num = (k, d) => {
      const v = q.get(k); const n = Number(v);
      return Number.isFinite(n) ? n : d;
    };
    // Default: if cx/cy missing, use original initial view
    return { cx: q.has('cx') ? num('cx', 0) : null,
             cy: q.has('cy') ? num('cy', 0) : null,
             z:  Math.max(0.01, num('z', 1)) };
  }

  function setSrcFromCenter(cx, cy, z) {
    const vw = BASE_W / z;
    const vh = BASE_H / z;
    const x  = cx - vw/2;
    const y  = cy - vh/2;
    const src = `${BOARD_BASE}&moveToViewport=${x},${y},${vw},${vh}&embedId=${EMBED_ID}`;
    document.getElementById('miro-frame').src = src;
  }

  function setSrcOriginal() {
    const src = `${BOARD_BASE}&moveToViewport=-712,-643,1455,1322&embedId=${EMBED_ID}`;
    document.getElementById('miro-frame').src = src;
  }

  (function init() {
    const { cx, cy, z } = getParams();
    if (cx === null || cy === null) setSrcOriginal();
    else setSrcFromCenter(cx, cy, z);
  })();
</script>
```

**Behavior**
- If `cx`/`cy` are **absent**, it loads your original initial view.
- If present, it centers on `(cx, cy)` and applies `z` (defaulting to 1).

---

## 🔒 Privacy & security

- The app is entirely **client-side**. No data is sent anywhere except:
  - Miro SDK calls inside your board context
  - Optional use of `navigator.clipboard.writeText()` when you click **Copy**
- No external analytics, no trackers.

---

## 🛠 Troubleshooting

- **I don’t see the app icon.**  
  Open it once from the **Apps** drawer to register the toolbar icon.

- **Permission error** in the panel.  
  Ensure the app has `boards:read` (and you’re an **Editor** on the board).

- **Colleague can’t use it.**  
  They need it installed/enabled for the same team (or you can publish/share it within the team).

- **GitHub Pages 404**  
  Confirm Pages is enabled for the repo (**Settings → Pages**), and files live in the repo **root**. Wait ~1 minute after enabling.

- **Copied link doesn’t center correctly.**  
  Verify your site page uses the same baseline viewport values (`BASE_W`, `BASE_H`) you expect for `z=1`, and that `moveToViewport` is computed as top-left from center.

---

## 🧪 Extending the panel (optional ideas)

- Add a numeric **zoom input** in the panel; default to `1`.
- Add a **“Drop marker at center”** button:
  ```js
  await miro.board.createShape({ shape: 'circle', x: it.x, y: it.y, width: 8, height: 8 });
  ```
  (Requires `boards:write`.)

- Add a **“Copy link for current viewport”** (read viewport via `await miro.board.viewport.get()` and convert to `cx, cy, z`).

---

## 🔧 Updating & versioning

- Edit `index.html` or `panel.html` in GitHub → **Commit**.  
- GitHub Pages redeploys automatically.  
- If the panel seems cached, close/reopen it or reload the board.

---

## ❓FAQ

**Q: Do I need to include the Miro SDK `<script>`?**  
A: No. The `miro` object is injected by Miro when your app runs inside a board.

**Q: Can I host somewhere else?**  
A: Yes—Netlify, Vercel, Cloudflare Pages, S3/CloudFront, your own domain—all fine. Just point the **App URL (sdkUri)** to your hosted `index.html`.

**Q: Can I make this available to my whole company?**  
A: Yes—install to the relevant team(s), or go through your org’s process for publishing internal apps.

---

## 📄 License

MIT License — feel free to adapt for your team.

```
MIT License

Copyright (c) <YEAR> <YOUR NAME>

Permission is hereby granted, free of charge, to any person obtaining a copy...
(standard MIT text — replace with your details if desired)
```

---

## 🙌 Acknowledgements

- Built on the **Miro Web SDK** (v2).  
- Thanks to the Parker Thatch team for the embed pattern and UX needs that inspired this tool.
