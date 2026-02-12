# Features to Make This Valentine Site Extra Special (Static-Only)

This repo is a **single-page static site**: everything lives in [`index.html`](index.html) (Tailwind via CDN, vanilla JS, `canvas-confetti`).

This document proposes features that **stay static** (no backend, no build step) and explains **exactly how to implement them** without breaking the current “Yes/No” flow.

---

## Baseline (what you already have)

In [`index.html`](index.html):

- **Core elements**:
  - `#imageDisplay` (the GIF)
  - `#valentineQuestion` (the headline text)
  - `#responseButtons` (wraps `#yesButton` and `#noButton`)
- **State & handlers**:
  - “No” click increments `noClickCount`, swaps GIF, grows “Yes” button, changes “No” text.
  - “Yes” click swaps final GIF, sets question text to `Yayyy!! :3`, hides `#responseButtons`, and fires heart confetti.

We’ll add new sections *below* the existing question area, keep the current behavior intact, and simply **reveal extra content** after “Yes”.

---

## Feature 1 — Love letter reveal (preview → unlock)

### What it is
- A beautiful **love letter card** with:
  - A short preview (always visible)
  - A “Read the full letter” action that only unlocks **after “Yes”**

### How to implement (static)

#### 1) Add markup (HTML)
In [`index.html`](index.html), inside the existing `<section class="flex flex-col ...">` (the one that currently contains the image, question, and buttons), add this **after** the `#responseButtons` div:

```html
<!-- Love letter -->
<section id="letterCard" class="w-full max-w-xl mt-8">
  <div class="rounded-2xl bg-white/80 backdrop-blur border border-pink-200 shadow-sm p-5">
    <div class="flex items-start justify-between gap-3">
      <div>
        <h2 class="text-2xl font-semibold text-[#bd1e59]">A letter for you</h2>
        <p class="text-sm text-pink-900/70 mt-1">Open when you’re ready.</p>
      </div>
      <button
        id="openLetterButton"
        class="inline-flex items-center justify-center rounded-md px-3 py-2 text-sm font-medium bg-pink-600 text-white hover:bg-pink-500 transition"
        type="button"
      >
        Read
      </button>
    </div>

    <div class="mt-4 text-pink-950/90 leading-relaxed">
      <p id="letterPreview">
        <!-- Replace this with your real preview line -->
        I’ve been thinking about you a lot lately, and I wanted to tell you something…
      </p>

      <div id="letterLockedNote" class="mt-3 text-sm text-pink-900/70">
        P.S. The full letter unlocks after you press “Yes”.
      </div>

      <div id="letterBody" class="mt-4 hidden">
        <!-- Replace this with your full letter content -->
        <p>My love,</p>
        <p class="mt-3">
          Here’s my full letter. Write it like you’d text her, but a little more intentional.
        </p>
        <p class="mt-3">
          I love you for… (your real words here)
        </p>
        <p class="mt-3">Love,</p>
        <p>Your Name</p>
      </div>
    </div>
  </div>
</section>
```

#### 2) Add behavior (JS)
In [`index.html`](index.html), inside the existing `(() => { ... })();` block:

- Add state:

```js
let letterUnlocked = false;
```

- Add DOM refs (near the existing DOM references around `yesButton/noButton/...`):

```js
const openLetterButton = document.getElementById('openLetterButton');
const letterBody = document.getElementById('letterBody');
const letterLockedNote = document.getElementById('letterLockedNote');
```

- Add a click handler:

```js
openLetterButton.addEventListener('click', () => {
  if (!letterUnlocked) {
    // Gentle nudge; keep it cute.
    letterLockedNote.textContent = 'Hehe — press “Yes” first :)';
    return;
  }

  letterBody.classList.toggle('hidden');
  openLetterButton.textContent = letterBody.classList.contains('hidden') ? 'Read' : 'Close';
});
```

- Unlock on “Yes”: in the existing `yesButton.addEventListener('click', () => { ... })` handler, after `responseButtons.style.display = 'none';`:

```js
letterUnlocked = true;
letterLockedNote.textContent = 'Unlocked.';
letterBody.classList.remove('hidden');
openLetterButton.textContent = 'Close';
```

#### 3) Optional polish (CSS/Tailwind-only)
- Add a subtle reveal animation using Tailwind transitions:
  - Replace `hidden` toggling with `opacity-0 pointer-events-none max-h-0` → `opacity-100 pointer-events-auto max-h-[1000px]` if you want a smooth expand.
  - Keep it simple: the current approach is already effective.

---

## Feature 2 — Photo “memories” gallery (scroll + captions + enlarge)

### What it is
- A horizontally scrollable gallery of your photos with a sweet caption under each one.
- Tap/click a photo to open a simple enlarge modal.

### How to implement (static)

#### 1) Add your photos (assets)
Create a new folder:

- `images/us/`

Add ~6–15 photos. Prefer compressed formats:
- Use `.jpg` or `.webp` (best)
- Avoid huge originals directly from iPhone if possible (size matters on mobile)

Recommended naming:
- `images/us/01.webp`, `images/us/02.webp`, ...

#### 2) Add markup (HTML)
In [`index.html`](index.html), add this section after the love letter section:

```html
<!-- Memories gallery -->
<section id="gallery" class="w-full max-w-4xl mt-8">
  <div class="rounded-2xl bg-white/70 backdrop-blur border border-pink-200 shadow-sm p-5">
    <div class="flex items-center justify-between gap-3">
      <h2 class="text-2xl font-semibold text-[#bd1e59]">Us</h2>
      <div class="flex gap-2">
        <button id="galleryPrev" class="rounded-md px-3 py-2 text-sm bg-pink-100 hover:bg-pink-200 transition" type="button">
          Prev
        </button>
        <button id="galleryNext" class="rounded-md px-3 py-2 text-sm bg-pink-100 hover:bg-pink-200 transition" type="button">
          Next
        </button>
      </div>
    </div>

    <div
      id="galleryStrip"
      class="mt-4 flex gap-4 overflow-x-auto snap-x snap-mandatory pb-2"
    ></div>
    <p class="mt-3 text-sm text-pink-900/70">Tip: swipe on mobile.</p>
  </div>
</section>

<!-- Enlarge modal -->
<div id="photoModal" class="fixed inset-0 hidden items-center justify-center bg-black/60 p-4">
  <div class="w-full max-w-3xl">
    <div class="flex justify-end">
      <button id="photoModalClose" class="text-white/90 hover:text-white text-sm px-3 py-2" type="button">Close</button>
    </div>
    <img id="photoModalImg" alt="" class="w-full rounded-xl bg-white object-contain max-h-[80vh]" />
    <p id="photoModalCaption" class="text-white/90 mt-3 text-center"></p>
  </div>
</div>
```

#### 3) Add data + rendering (JS)
In [`index.html`](index.html), add:

```js
const MEMORY_PHOTOS = [
  { src: './images/us/01.webp', alt: 'Us smiling', caption: 'The day I realized I’m really lucky.' },
  { src: './images/us/02.webp', alt: 'A cute moment', caption: 'I still think about this all the time.' },
  // ...
];
```

Then add DOM refs + render function:

```js
const galleryStrip = document.getElementById('galleryStrip');
const galleryPrev = document.getElementById('galleryPrev');
const galleryNext = document.getElementById('galleryNext');

const photoModal = document.getElementById('photoModal');
const photoModalImg = document.getElementById('photoModalImg');
const photoModalCaption = document.getElementById('photoModalCaption');
const photoModalClose = document.getElementById('photoModalClose');

function renderGallery() {
  galleryStrip.innerHTML = '';

  for (const item of MEMORY_PHOTOS) {
    const card = document.createElement('button');
    card.type = 'button';
    card.className = 'snap-start text-left min-w-[220px] max-w-[260px]';

    card.innerHTML = `
      <div class="rounded-xl overflow-hidden border border-pink-200 bg-white shadow-sm">
        <img src="${item.src}" alt="${item.alt}" class="h-56 w-full object-cover" loading="lazy" />
        <div class="p-3">
          <p class="text-sm text-pink-950/90">${item.caption}</p>
        </div>
      </div>
    `;

    card.addEventListener('click', () => {
      photoModalImg.src = item.src;
      photoModalImg.alt = item.alt;
      photoModalCaption.textContent = item.caption;
      photoModal.classList.remove('hidden');
      photoModal.classList.add('flex');
    });

    galleryStrip.appendChild(card);
  }
}

function scrollGalleryByCards(direction) {
  const card = galleryStrip.querySelector('button');
  const amount = card ? card.getBoundingClientRect().width + 16 : 260;
  galleryStrip.scrollBy({ left: direction * amount, behavior: 'smooth' });
}

galleryPrev.addEventListener('click', () => scrollGalleryByCards(-1));
galleryNext.addEventListener('click', () => scrollGalleryByCards(1));
photoModalClose.addEventListener('click', () => {
  photoModal.classList.add('hidden');
  photoModal.classList.remove('flex');
});
photoModal.addEventListener('click', (e) => {
  if (e.target === photoModal) {
    photoModal.classList.add('hidden');
    photoModal.classList.remove('flex');
  }
});

renderGallery();
```

#### 4) Optional: only show the gallery after “Yes”
If you want it to feel like an unlockable surprise:
- Add `class="hidden"` to `#gallery` and then in the “Yes” handler: `document.getElementById('gallery').classList.remove('hidden');`.

---

## Feature 3 — “The actual ask” date-invite card (copies message)

### What it is
After “Yes”, show a card that makes a real plan and includes a **copy-to-clipboard** button that generates a cute prefilled message she can send you (or that you can send her).

### How to implement (static)

#### 1) Add markup (HTML)
Place this after the gallery section:

```html
<!-- Date invite -->
<section id="inviteDetails" class="w-full max-w-xl mt-8 hidden">
  <div class="rounded-2xl bg-white/80 backdrop-blur border border-pink-200 shadow-sm p-5">
    <h2 class="text-2xl font-semibold text-[#bd1e59]">Okay… it’s official.</h2>
    <p class="mt-2 text-pink-950/90">Pick what sounds best, and I’ll make it happen.</p>

    <div class="mt-4 grid gap-2">
      <label class="flex items-center gap-2 rounded-lg border border-pink-200 bg-white p-3 cursor-pointer">
        <input type="radio" name="dateIdea" value="Dinner" checked />
        <span>Dinner</span>
      </label>
      <label class="flex items-center gap-2 rounded-lg border border-pink-200 bg-white p-3 cursor-pointer">
        <input type="radio" name="dateIdea" value="Movie" />
        <span>Movie</span>
      </label>
      <label class="flex items-center gap-2 rounded-lg border border-pink-200 bg-white p-3 cursor-pointer">
        <input type="radio" name="dateIdea" value="Surprise" />
        <span>Surprise</span>
      </label>
    </div>

    <div class="mt-4">
      <label class="text-sm text-pink-900/70">When?</label>
      <input id="dateWhen" class="mt-1 w-full rounded-md border border-pink-200 px-3 py-2" placeholder="Friday 7pm?" />
    </div>

    <div class="mt-4 flex flex-wrap gap-2 items-center">
      <button
        id="copyInvite"
        class="rounded-md px-4 py-2 bg-pink-600 text-white hover:bg-pink-500 transition"
        type="button"
      >
        Copy message
      </button>
      <span id="copyInviteStatus" class="text-sm text-pink-900/70"></span>
    </div>

    <textarea id="copyFallback" class="mt-3 hidden w-full rounded-md border border-pink-200 px-3 py-2 h-28"></textarea>
  </div>
</section>
```

#### 2) Add behavior (JS)
Add DOM refs + copy logic:

```js
const inviteDetails = document.getElementById('inviteDetails');
const dateWhen = document.getElementById('dateWhen');
const copyInvite = document.getElementById('copyInvite');
const copyInviteStatus = document.getElementById('copyInviteStatus');
const copyFallback = document.getElementById('copyFallback');

function getSelectedDateIdea() {
  const selected = document.querySelector('input[name="dateIdea"]:checked');
  return selected ? selected.value : 'Dinner';
}

function buildInviteMessage() {
  const idea = getSelectedDateIdea();
  const when = (dateWhen.value || 'this week').trim();
  return `Valentine’s is official 💗 Can we do ${idea.toLowerCase()} ${when}?`;
}

copyInvite.addEventListener('click', async () => {
  const msg = buildInviteMessage();
  copyInviteStatus.textContent = '';

  try {
    await navigator.clipboard.writeText(msg);
    copyInviteStatus.textContent = 'Copied.';
  } catch {
    // Fallback: show textarea for manual copy
    copyFallback.value = msg;
    copyFallback.classList.remove('hidden');
    copyFallback.focus();
    copyFallback.select();
    copyInviteStatus.textContent = 'Copy it from the box below.';
  }
});
```

#### 3) Reveal after “Yes”
In the existing “Yes” handler in [`index.html`](index.html), after `responseButtons.style.display = 'none';`:

```js
inviteDetails.classList.remove('hidden');
```

---

## Feature 4 — “Reasons I love you” (tap-to-reveal cards)

### What it is
Interactive cards that reveal a reason when tapped. After all are seen, a small extra confetti burst.

### How to implement (static)

#### 1) Add markup (HTML)
Place after the invite section:

```html
<!-- Reasons -->
<section id="reasons" class="w-full max-w-4xl mt-8 hidden">
  <div class="rounded-2xl bg-white/70 backdrop-blur border border-pink-200 shadow-sm p-5">
    <h2 class="text-2xl font-semibold text-[#bd1e59]">Reasons I love you</h2>
    <p class="mt-2 text-sm text-pink-900/70">Tap a card.</p>
    <div id="reasonsGrid" class="mt-4 grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3"></div>
    <p id="reasonsDone" class="mt-4 text-sm text-pink-900/70 hidden">You found them all.</p>
  </div>
</section>
```

#### 2) Add behavior (JS)

```js
const reasonsSection = document.getElementById('reasons');
const reasonsGrid = document.getElementById('reasonsGrid');
const reasonsDone = document.getElementById('reasonsDone');

const REASONS = [
  'You make boring days feel easy.',
  'I love the way you laugh at your own jokes.',
  'You’re my calm and my chaos (in the best way).',
  // Add 10–30 real reasons.
];

const seenReasons = new Set();

function renderReasons() {
  reasonsGrid.innerHTML = '';

  REASONS.forEach((text, idx) => {
    const card = document.createElement('button');
    card.type = 'button';
    card.className = 'rounded-xl border border-pink-200 bg-white p-4 text-left shadow-sm hover:shadow transition';

    const title = `Reason #${idx + 1}`;
    card.innerHTML = `<div class="text-sm text-pink-900/70">${title}</div><div class="mt-2 font-medium text-pink-950">Tap to reveal</div>`;

    card.addEventListener('click', () => {
      if (seenReasons.has(idx)) return;
      seenReasons.add(idx);
      card.innerHTML = `<div class="text-sm text-pink-900/70">${title}</div><div class="mt-2 font-medium text-pink-950">${text}</div>`;

      if (seenReasons.size === REASONS.length) {
        reasonsDone.classList.remove('hidden');
        // Optional: small confetti hit on completion
        confetti({ particleCount: 25, spread: 80, startVelocity: 20, colors: ['#ff69b4', '#ff1493'] });
      }
    });

    reasonsGrid.appendChild(card);
  });
}

renderReasons();
```

#### 3) Reveal after “Yes”
In the “Yes” handler:

```js
reasonsSection.classList.remove('hidden');
```

---

## Assets plan (photos + letter)

### Photos
- Put your photos in `images/us/`.
- Prefer `.webp` or compressed `.jpg`.
- Keep each image ideally under ~300–600KB for fast mobile loads.

### Letter content
- Keep your letter in the HTML (easy and immediate), or move it to a small JS string if you want to keep markup clean.
- If you want to keep it private-ish, avoid putting truly sensitive info in the code (anyone can view-source).

---

## Integration checklist (exact insertion points)

In [`index.html`](index.html):

1) **HTML insertion point**
- Inside the `<main>` → `<section class="flex flex-col ...">` block:
  - Add new sections **after** the current `#responseButtons` element.
  - This keeps the existing layout and ensures “Yes/No” remain above your added content.

2) **JS insertion point**
- Inside the existing IIFE:
  - Add new DOM refs near the current refs at:
    - `const yesButton = ...` / `const noButton = ...` / `const imageDisplay = ...` / `const valentineQuestion = ...` / `const responseButtons = ...`
  - Add new feature state variables near the current state:
    - `let noClickCount = ...`
  - In the existing “Yes” click handler, after:
    - `responseButtons.style.display = 'none';`
  - Reveal/unlock:
    - `letterUnlocked = true;` + show `#inviteDetails`, `#reasons`, `#gallery` (if you decide to hide them initially)

### Exact anchors in your current file

Use these existing blocks as safe anchors (they’re already present in your `index.html`):

```43:69:index.html
  <main class="flex items-center justify-center h-screen">
    <section class="flex flex-col items-center p-4">
      <img
        id="imageDisplay"
        src="./images/image1.gif"
        alt="Cute kitten with flowers"
        class="rounded-md h-[300px] object-cover"
      />
      <h1 id="valentineQuestion" class="text-4xl font-bold italic text-[#bd1e59] my-4">
        Will you be my Valentine?
      </h1>
      <div id="responseButtons" class="flex gap-4 pt-5 items-center">
        <!-- Yes/No buttons -->
      </div>
    </section>
  </main>
```

- **Add new HTML sections** immediately after the `</div>` that closes `#responseButtons` (currently just before the `</section>` that closes the content block).

Your current JS is an IIFE beginning at:

```71:80:index.html
  <script>
    (() => {
      // DOM references
      const yesButton = document.getElementById('yesButton');
      const noButton = document.getElementById('noButton');
      const imageDisplay = document.getElementById('imageDisplay');
      const valentineQuestion = document.getElementById('valentineQuestion');
      const responseButtons = document.getElementById('responseButtons');
```

- **Add new DOM references** directly after `responseButtons` so everything stays grouped.

Your current “Yes” handler starts here:

```142:158:index.html
      yesButton.addEventListener('click', () => {
        imageDisplay.src = IMAGE_PATHS[IMAGE_PATHS.length - 1];
        imageDisplay.alt = ALT_TEXTS[ALT_TEXTS.length - 1];
        valentineQuestion.textContent = 'Yayyy!! :3';
        responseButtons.style.display = 'none';

        // Heart-shaped confetti for a valentine theme
        const heart = confetti.shapeFromPath({
          path: 'M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z',
        });
        // ... confetti bursts ...
      });
```

- **Add your “unlock/reveal” lines** right after `responseButtons.style.display = 'none';` and before the confetti setup. That way, even if confetti rendering is slow on some devices, the UI update happens immediately.

3) **Don’t break existing flow**
- Keep the existing “No” handler unchanged.
- Keep the existing confetti code unchanged.
- Only add new behaviors that **reveal** content on “Yes”.

---

## Manual test plan

- **Baseline regression**:
  - Click “No” 0→5 times: GIF/alt text changes, “Yes” grows, “No” text cycles.
  - Click “Yes”: final GIF, `Yayyy!! :3`, buttons hidden, confetti fires.
- **Letter**:
  - Before “Yes”: Read button shows locked note.
  - After “Yes”: letter unlocks and opens.
- **Gallery**:
  - Images render, swipe works, modal opens/closes.
- **Invite**:
  - Radio selection changes message.
  - Clipboard copy works; fallback textarea appears if blocked.

