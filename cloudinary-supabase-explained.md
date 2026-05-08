# Cloudinary + Supabase Integration — Aqua Star
## How It Works (Plain English)

---

## THE FLOW

```
Admin picks image
      ↓
Browser sends image to Cloudinary
      ↓
Cloudinary stores the image, returns a URL
(e.g. https://res.cloudinary.com/dst3d5tjb/image/upload/aquastar/photo.jpg)
      ↓
Browser sends that URL + label + category to Supabase
      ↓
Supabase stores it in the "gallery" table
      ↓
Any visitor anywhere opens the site
      ↓
Browser fetches rows from Supabase
      ↓
Displays images using the Cloudinary URLs
```

---

## PART 1 — CLOUDINARY

**What it does:** Stores the actual image files. Think of it as Google Drive for images.

**Your credentials:**
- Cloud Name: `dst3d5tjb`
- Upload Preset: `aquastar_uploads`

### The upload URL
```
https://api.cloudinary.com/v1_1/dst3d5tjb/image/upload
```

### How the upload works in code

```javascript
// Step 1: Create a FormData object with the image file
const formData = new FormData();
formData.append('file', imageFile);           // the actual image file
formData.append('upload_preset', 'aquastar_uploads'); // your preset (unsigned)
formData.append('folder', 'aquastar');        // optional: organizes in a folder

// Step 2: POST it to Cloudinary
const response = await fetch(
  'https://api.cloudinary.com/v1_1/dst3d5tjb/image/upload',
  { method: 'POST', body: formData }
);

// Step 3: Get the URL back
const data = await response.json();
const imageURL = data.secure_url;
// imageURL is now something like:
// "https://res.cloudinary.com/dst3d5tjb/image/upload/v1234/aquastar/photo.jpg"
```

### Why "unsigned" preset?
- Normally Cloudinary requires a secret API key to upload
- An "unsigned" preset removes that requirement
- This means the browser can upload directly WITHOUT exposing any secret
- Safe for frontend-only apps like this one

### What Cloudinary returns (the full response object)
```json
{
  "secure_url": "https://res.cloudinary.com/dst3d5tjb/image/upload/v123/aquastar/photo.jpg",
  "public_id": "aquastar/photo",
  "width": 1200,
  "height": 800,
  "format": "jpg",
  "bytes": 245000
}
```
We only need `secure_url` — that's the permanent link to the image.

---

## PART 2 — SUPABASE

**What it does:** Stores the image URLs + metadata (label, category, date).
Think of it as a spreadsheet in the cloud that everyone can read.

**Your credentials:**
- Project URL: `https://zyovbymdxroaqluzhjgx.supabase.co`
- Anon Key: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` (the long key you provided)

### The database table: `gallery`

| Column       | Type        | Example value                                      |
|--------------|-------------|---------------------------------------------------|
| id           | uuid        | auto-generated                                    |
| created_at   | timestamp   | auto-generated                                    |
| url          | text        | https://res.cloudinary.com/dst3d5tjb/...          |
| label        | text        | "Betta Fish"                                      |
| category     | text        | "fish"                                            |

### How Supabase REST API works

Supabase gives you a REST API automatically for every table.
You talk to it using fetch() with your anon key in the headers.

**Headers needed for every request:**
```javascript
const headers = {
  'Content-Type': 'application/json',
  'apikey': 'YOUR_ANON_KEY',
  'Authorization': 'Bearer YOUR_ANON_KEY'
};
```

### READ all photos
```javascript
// GET /rest/v1/gallery?order=created_at.desc
const response = await fetch(
  'https://zyovbymdxroaqluzhjgx.supabase.co/rest/v1/gallery?order=created_at.desc',
  { headers }
);
const photos = await response.json();
// photos is an array like:
// [
//   { id: "abc", url: "https://...", label: "Betta Fish", category: "fish", created_at: "..." },
//   { id: "def", url: "https://...", label: "Macaw", category: "birds", created_at: "..." }
// ]
```

### READ photos filtered by category
```javascript
// GET /rest/v1/gallery?category=eq.fish&order=created_at.desc
// "eq" means "equals"
const response = await fetch(
  'https://zyovbymdxroaqluzhjgx.supabase.co/rest/v1/gallery?category=eq.fish&order=created_at.desc',
  { headers }
);
const fishPhotos = await response.json();
```

### INSERT a new photo
```javascript
// POST /rest/v1/gallery
const response = await fetch(
  'https://zyovbymdxroaqluzhjgx.supabase.co/rest/v1/gallery',
  {
    method: 'POST',
    headers: { ...headers, 'Prefer': 'return=representation' },
    body: JSON.stringify({
      url: 'https://res.cloudinary.com/dst3d5tjb/...',
      label: 'Betta Fish',
      category: 'fish'
    })
  }
);
const inserted = await response.json();
```

### DELETE a photo
```javascript
// DELETE /rest/v1/gallery?id=eq.PHOTO_ID
const response = await fetch(
  'https://zyovbymdxroaqluzhjgx.supabase.co/rest/v1/gallery?id=eq.abc-123',
  { method: 'DELETE', headers }
);
```

---

## PART 3 — HOW THEY WORK TOGETHER IN index.html

### The helper functions in the code

```javascript
// ── Cloudinary config ──
const CLD_CLOUD  = 'dst3d5tjb';
const CLD_PRESET = 'aquastar_uploads';
const CLD_URL    = `https://api.cloudinary.com/v1_1/${CLD_CLOUD}/image/upload`;

// ── Supabase config ──
const SB_URL = 'https://zyovbymdxroaqluzhjgx.supabase.co';
const SB_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
const SB_HDR = {
  'Content-Type': 'application/json',
  'apikey': SB_KEY,
  'Authorization': 'Bearer ' + SB_KEY
};

// ── Fetch from Supabase ──
async function sbFetch(path, opts = {}) {
  const r = await fetch(SB_URL + path, { headers: SB_HDR, ...opts });
  if (!r.ok) throw new Error(await r.text());
  return r.json();
}

// ── Get all photos (or filtered by category) ──
async function sbGetGallery(cat) {
  const q = cat && cat !== 'all'
    ? `?category=eq.${cat}&order=created_at.desc`
    : `?order=created_at.desc`;
  return sbFetch('/rest/v1/gallery' + q);
}

// ── Save a photo record ──
async function sbInsertPhoto(row) {
  return sbFetch('/rest/v1/gallery', {
    method: 'POST',
    body: JSON.stringify(row),
    headers: { ...SB_HDR, 'Prefer': 'return=representation' }
  });
}

// ── Delete a photo record ──
async function sbDeletePhoto(id) {
  return fetch(SB_URL + `/rest/v1/gallery?id=eq.${id}`, {
    method: 'DELETE',
    headers: SB_HDR
  });
}
```

### The full upload sequence (what happens when admin clicks "Add to Gallery")

```javascript
async function apSubmitUpload() {
  // 1. Validate inputs
  const label = document.getElementById('ap-up-label').value.trim();
  const cat   = document.getElementById('ap-up-cat').value;

  // 2. For each selected file:
  for (const file of apPendingFiles) {

    // 3. Upload image to Cloudinary
    const fd = new FormData();
    fd.append('file', file.blob);
    fd.append('upload_preset', CLD_PRESET);
    fd.append('folder', 'aquastar');

    const cldRes  = await fetch(CLD_URL, { method: 'POST', body: fd });
    const cldData = await cldRes.json();
    const url     = cldData.secure_url;  // permanent image URL

    // 4. Save URL + metadata to Supabase
    await sbInsertPhoto({ url, label, category: cat });
  }

  // 5. Reload gallery from Supabase to show new photos
  await apLoadGallery();
}
```

---

## PART 4 — WHAT STAYS IN LOCALSTORAGE

Not everything moved to Supabase. Here's what's where:

| Data | Storage | Key |
|------|---------|-----|
| Gallery photos (URLs) | **Supabase** | `gallery` table |
| Products (with base64 images) | **localStorage** | `aquastar_products_v2` |
| Section notes (text) | **localStorage** | `aquastar_sections_v1` |

Products still use localStorage because they store base64 images directly.
To move products to Cloudinary+Supabase too, the same pattern applies.

---

## PART 5 — DEPLOYING TO VERCEL

1. Create a GitHub account if you don't have one
2. Create a new repository called `aquastar`
3. Upload `index.html` to the repository
4. Go to vercel.com → Sign up with GitHub
5. Click "Add New Project" → Import your `aquastar` repo
6. Vercel detects it's a static HTML file automatically
7. Click Deploy
8. Your site is live at `aquastar.vercel.app`

Every time you update `index.html` and push to GitHub, Vercel redeploys automatically.

---

## PART 6 — SECURITY NOTE

The Supabase anon key is safe to use in frontend code.
It only has permission to read/write the `gallery` table (because RLS is off in test mode).

For production, you should enable Row Level Security (RLS) in Supabase:
- Allow anyone to READ (SELECT) from gallery
- Only allow INSERT/DELETE when a specific condition is met

But for now, test mode works fine for a pet store.

---

## SUMMARY

```
Cloudinary  = image file storage (like Google Drive for photos)
Supabase    = database (stores URLs, labels, categories)
Vercel      = web hosting (serves your HTML file to visitors)
localStorage = still used for products and section notes
```

The beauty of this setup: **zero monthly cost** on free tiers,
and images are served from Cloudinary's global CDN — fast everywhere.
