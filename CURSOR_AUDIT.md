# Cursor debug audit – index.html

## Step 1 – Audit: every occurrence of "cursor"

| Location | Type | What |
|----------|------|------|
| 10–11 | CSS | `*, *::before, *::after { cursor: none !important; }` – hides native mouse pointer globally |
| 32–60 | CSS | Custom cursor: `#cursor`, `#cursor-ring`, `body:has(a:hover) #cursor` |
| 660–661 | HTML | `<div class="cursor" id="cursor"></div>` and `<div class="cursor-ring" id="cursor-ring"></div>` |
| 930–939 | JS | `getElementById('cursor')`, `getElementById('cursor-ring')`, `mousemove` sets left/top |

Conflicts/duplicates: None. Only one custom-cursor implementation.  
Missing: No other cursor code found. IDs match between HTML and JS.

## Step 2 – Common causes

- **Elements in body?** Yes. Two divs at lines 660–661, right after `<body>`.
- **Hidden by CSS?** No. No `display:none`, `visibility:hidden`, `opacity:0`, `width:0`, `height:0` on `#cursor` or `#cursor-ring`. They have `opacity: 1`.
- **Override `cursor: none`?** The universal rule sets `cursor: none` (mouse pointer); it does not hide the custom cursor divs.
- **Position?** `position: fixed`, `left: 0`, `top: 0`, `transform: translate(-50%, -50%)`. JS sets `left`/`top` on mousemove. Correct.
- **z-index?** 10001. Nav 100, cookie banner 10000. Cursor on top. OK.
- **DOM ready?** Script is at end of body; no `DOMContentLoaded`. If the Cloudflare script alters the DOM, elements might be missing when our script runs. **Fix: run cursor init inside DOMContentLoaded.**
- **getElementById match?** HTML has `id="cursor"` and `id="cursor-ring"`. JS uses `'cursor'` and `'cursor-ring'`. Match. OK.
- **JS errors?** If `cursor` or `ring` were null, `cursor.style.left = ...` would throw and break the script. **Fix: null-check and run after DOMContentLoaded.**

## Step 3 – Fix

- Rewrite cursor HTML/CSS/JS as a single, conflict-free block.
- Run cursor JS inside `DOMContentLoaded`, with null-checks.
- Add temporary `console.log` to confirm elements found and mousemove firing.

## Step 4 – Test

After fix: open the site, then open DevTools (F12) → Console tab. You should see:

1. **"Cursor elements found: true"** – if you see `false`, the DOM doesn’t contain the cursor divs when the script runs.
2. **"Cursor mousemove fired"** – appears the first time you move the mouse on the page.

If both appear and the cursor is still invisible, the problem is likely CSS (e.g. another stylesheet or browser extension hiding fixed elements). If "Cursor elements found: false" appears, something is removing or renaming the elements before the script runs.
