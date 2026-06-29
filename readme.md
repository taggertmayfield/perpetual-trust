[README (3).md](https://github.com/user-attachments/files/29478309/README.3.md)
# Perpetual Trust — Land Holdings Map

An interactive satellite map of the trust's Richland County, IL parcels. Click a
parcel to see and edit its details — structures, values, income, taxes, notes —
all backed by Firestore so it persists across devices.

Three holdings are pre-loaded:

| Parcel             | Acres | Notes                                   |
|--------------------|-------|-----------------------------------------|
| North Woods & Pond | 26.65 | Woods + pond, east of County Rd 1150    |
| South Field        | 49.54 | Large field S of E Elbow Ln, along Rt 130 |
| East Field         | 12.13 | Field N of E Elbow Ln                    |

---

## Files

```
perpetual-trust/
├─ index.html      ← the whole app (Leaflet map + Firebase)
├─ assets/         ← original parcel reference images
│   ├─ parcel-northwoods-2665.jpg
│   ├─ parcel-southfield-4954.jpg
│   ├─ parcel-eastfield-1213.jpg
│   └─ holdings-overview.jpg
└─ README.md
```

It runs immediately with **no setup** in local-only mode (edits last for the
session). Wire up Firebase to make everything permanent.

---

## 1. Put it on GitHub Pages

From inside the `perpetual-trust` folder:

```bash
git init
git add .
git commit -m "Initial parcel map"
git branch -M main
git remote add origin https://github.com/taggertmayfield/perpetual-trust.git
git push -u origin main
```

Create the empty `perpetual-trust` repo on GitHub first, then in
**Settings → Pages**, set Source = `main` / root. It'll publish at:

```
https://taggertmayfield.github.io/perpetual-trust/
```

---

## 2. Connect Firebase (so edits save)

Easiest path: **reuse your existing quality-audit project**. The app writes to a
brand-new `trust_parcels` collection, so it won't touch your audit data.

1. Open `index.html` and find the `FIREBASE_CONFIG` block near the top of the
   `<script type="module">` section.
2. Paste your project's web config in (Firebase console → Project settings →
   Your apps → Web app → SDK setup → Config). You can paste the same config you
   already use for quality-audit (project `quality-audit-56241`).
3. Make sure your Firestore rules allow the `trust_parcels` collection. A simple
   starter (tighten later with Auth):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /trust_parcels/{doc} {
      allow read, write: if true;   // personal tool — lock down with Auth when ready
    }
    // ... your existing quality-audit rules stay as they are ...
  }
}
```

The status chip in the header turns green ("Saved to cloud") once it connects.

> Want it in its own Firebase project instead? Create a new project, enable
> Firestore, and paste that config instead. Same `trust_parcels` collection.

---

## 3. Using it

- **Click a parcel** (on the map or in the Parcels list) → details drawer opens.
- **Edit details** → update overview, add/remove structures, income lines, tax
  years, and notes. Save writes the parcel to Firestore.
- **Edit boundary** → the seeded shapes are rough rectangles. Hit this, then
  click on the satellite image to trace the real property lines (drag points to
  fine-tune, right-click a point to delete). Save once and it's permanent.
- The **header ledger** auto-totals across all parcels: acreage, improvement
  value, annual income, annual taxes, and net per year.

---

## 4. Data model (`trust_parcels/{parcelId}`)

So we both know the shape as you send more info over time:

```js
{
  name:      "North Woods & Pond",
  acres:     26.65,
  parcelId:  "",                    // county APN
  location:  "Richland County, IL · ...",
  color:     "#7BA05E",             // map color
  image:     "assets/...jpg",       // reference image (optional)
  boundary:  [ {lat, lng}, ... ],   // polygon (lat/lng objects — Firestore-safe)

  structures: [ { name, type, year, value, notes } ],
  income:     [ { source, amount, frequency, notes } ],  // frequency: annual | monthly | one-time
  taxes:      [ { year, amount, status, notes } ],       // status: paid | due
  notes:      "free text",
  updatedAt:  1719000000000
}
```

Doc IDs for the seeded three are `northwoods`, `southfield`, `eastfield`.
Annual income = annual amounts + monthly×12 (one-time excluded from the run-rate).
Current tax = the most recent tax year on record.

When you're ready, just send me the numbers (structures + values, rents/CRP,
tax bills, APNs) and I'll either hand you ready-to-paste Firestore entries or you
can type them straight into the Edit panel.
