# Kayali — Discover Your Scent

An immersive brand activation web experience for Kayali Oudgasm fragrances. A single-page interactive kiosk that guides visitors through:

1. A welcome slideshow with a **live community trending widget**
2. A 5-question sensory quiz
3. A 3-layer digital fragrance experience
4. A real-time **3D bottle configurator** with drag-to-rotate and live customization
5. A **personalized collection page** showing where the visitor sits relative to the global community
6. A purchase moment with bottle engraving

Designed for big-screen showcase at brand activation booths, but works on any modern device.

---

## File structure

All files must live in the same folder for image references to work.

```
/
├── index.html                  ← entry point
├── README.md                   ← this file
├── bottle-vanilla.png          ← Vanilla Oud product shot
├── bottle-cafe.png             ← Café Oud product shot
├── bottle-chocolate.png        ← Chocolate Oud product shot
├── bottle-musk.png             ← Milky Musk Oud product shot
├── campaign-burner.png         ← Branded hero scene with incense burner
├── lifestyle-vanilla.webp      ← Gold-lit model with Vanilla Oud
└── lifestyle-cafe.webp         ← Copper-silk model with Café Oud
```

If any image fails to load, confirm it's in the same directory as `index.html`.

---

## Deploying to GitHub Pages

1. Create a new GitHub repository (public).
2. Upload all files from this folder to the repo root.
3. In the repo, go to **Settings → Pages**.
4. Under **Source**, choose the `main` branch and `/ (root)` folder.
5. Click **Save**. GitHub will give you a URL like `https://yourusername.github.io/your-repo-name/` — usually live within a minute.

No build step, no install required.

---

## Live trending sync with Firebase (optional but recommended)

The kiosk includes a **live community match tracker**: a "Tonight's Most Loved" widget on the welcome slideshow, and a personal positioning card on the collection page ("You're among the 23% of Kayali fans who match with Rose Oud"). These update in real time as visitors complete quizzes.

**Without Firebase:** the data is stored per-device in `localStorage`. Each kiosk maintains its own running tally — fine for a single-screen activation.

**With Firebase:** all kiosks read from and write to a shared Firestore document, so trending data syncs live across every device viewing the page. Visitor #47 on Kiosk A will see the leaderboard reshuffle the moment Visitor #48 finishes their quiz on Kiosk B.

### Setting up Firebase (about 5 minutes)

1. Go to **https://console.firebase.google.com** and create a new project (free tier is plenty).
2. In the project, click the web icon (`</>`) to add a web app. Give it any name — copy the `firebaseConfig` object that appears.
3. Open `index.html` and find the section labeled `FIREBASE — live cross-device trending sync` (around line 4190). Replace these placeholder values with your real config:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

4. Back in Firebase console, go to **Build → Firestore Database → Create database**. Choose **Start in test mode** to start (you can lock it down later — see below).
5. Save and reload the kiosk. Open the browser console — you should see `[Kayali] Firebase live sync active.`

The kiosk will create a single document at `/kayali/trending` with one count per fragrance, seeded with the baseline numbers. From then on, every quiz completion atomically increments the matching fragrance counter, and `onSnapshot` pushes the updated counts to every connected device instantly.

### Production security rules (recommended)

Test mode allows anyone to read or write. Once you've confirmed it works, replace your Firestore rules with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /kayali/trending {
      allow read: if true;
      allow write: if true;  // tighten further if you add auth
    }
  }
}
```

This restricts reads/writes to just the trending document. For a public kiosk this is sufficient.

### Resetting the data

To start the trending tally fresh (e.g. between events), delete the `/kayali/trending` document in the Firestore console. The kiosk will re-create it on the next visit with the baseline seed.

---

## Running locally

Because the page loads images via relative paths and uses ES modules for Three.js and Firebase, opening `index.html` directly via `file://` may fail in some browsers. Serve it locally:

```bash
python3 -m http.server 8000
# or
npx serve
```

Then open `http://localhost:8000`.

---

## Browser requirements

- Chrome / Edge **89+**
- Firefox **108+**
- Safari **16.4+**

WebGL is required for the 3D configurator. All recent desktop and mobile browsers support it. Three.js loads from `unpkg.com`; Firebase loads from `gstatic.com`. Both are blocked by some corporate firewalls — if your venue's network is restrictive, host them yourself.

---

## The flow

1. **Welcome** — auto-looping slideshow (4s per slide):
   - Slide 1: hero introduction with the Vanilla bottle
   - Slide 2: the full Oudgasm collection
   - Slide 3: **live trending widget** showing tonight's most loved scent
2. **Profile intro** — primer for the quiz
3. **Quiz** — 5 questions, auto-advance on answer selection
4. **Sensory Experience** — 3 layers (Opening · Heart · Dry Down) with floating note cards and per-layer atmosphere shifts
5. **Result + Configurator + Purchase** — matched fragrance reveal, lifestyle moment, note breakdown, **3D bottle configurator** with engraving + finish customization, and Add to Cart
6. **Collection** — full catalog of all 4 fragrances with detailed notes, personal positioning ("You're in the X% who match with…"), tonight's #1 leader card, and a Match Strength ranking based on the quiz. Tap any card to load that fragrance into the configurator.

The "↻ Retake The Journey" link resets the kiosk for the next visitor.

---

## Updating fragrance data

Open `index.html`, find the `const fragrances = { ... }` block. Each fragrance has:

```js
vanilla: {
  name: "Vanilla Oud",
  code: "No. 36",
  description: "...",
  price: "$145",
  image: "bottle-vanilla.webp",
  lifestyleImage: "lifestyle-vanilla.webp",
  lifestyleCaption: "Soft & Unforgettable",
  summary: { top: "...", heart: "...", base: "..." },
  layers: [ /* Opening, Heart, Dry Down with note details */ ]
}
```

To swap in real photography for Café and Chocolate (currently using fallbacks), drop new files into the folder and update the `image` and `lifestyleImage` paths.

---

## The 3D bottle configurator

Lives on the result page. Built with Three.js using:

- **Drag-to-rotate** with damping (inertia), polar angle clamped so it never flips
- **Idle floating** animation when not interacting
- **PBR materials** — transmission, IOR, and clearcoat for the glass; flat-shaded faceted cap for crisp diamond cuts
- **Studio environment** via `RoomEnvironment` for HDRI-style reflections
- **Custom-built bottle geometry** matching the Kayali silhouette: chamfered 8-sided body with sloped shoulders, single-mesh diamond-cut cap
- **Live label** — drawn to a canvas, updates instantly when the user types an engraving

Customizable via the form: engraving (max 18 characters), bottle color (black), glass finish (clear/frosted/tinted), cap finish (matte/gloss/metallic).

To add more colors or finishes, edit the `glassFinishes` and `capFinishes` objects in the Three.js module near the bottom of `index.html`.

---

## Tech notes

- Single HTML file plus seven image assets — no build, no framework, no install step.
- Inter font (Google Fonts), Three.js 0.162 (unpkg), Firebase 10.7 (gstatic) — all from CDN.
- Trending data persists in `localStorage` by default, optionally syncs live via Firestore.
- Responsive down to mobile widths.
