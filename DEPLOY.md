# Aqua Star — Vercel Deployment Guide

---

## STEP 1 — Create a GitHub Repository

1. Go to github.com → sign in (or create free account)
2. Click the **+** icon top right → **New repository**
3. Name it: `aquastar`
4. Set to **Public** (required for free Vercel)
5. Click **Create repository**

---

## STEP 2 — Upload your files to GitHub

You have two options:

### Option A — Drag & Drop (easiest)
1. Open your new repo on GitHub
2. Click **uploading an existing file**
3. Drag these files into the browser:
   - `index.html`  ← the main file
   - `.gitignore`  ← prevents .env from uploading
   - `DEPLOY.md`   ← optional
   - **DO NOT upload `.env`** — it has your secret keys
4. Click **Commit changes**

### Option B — Git CLI
```bash
git init
git add index.html .gitignore DEPLOY.md
git commit -m "Initial deploy"
git remote add origin https://github.com/YOURUSERNAME/aquastar.git
git push -u origin main
```

---

## STEP 3 — Deploy on Vercel

1. Go to **vercel.com** → click **Sign Up** → choose **Continue with GitHub**
2. Click **Add New Project**
3. Find your `aquastar` repo → click **Import**
4. Vercel auto-detects it as a static site
5. Leave all settings as default
6. Click **Deploy**
7. Wait ~30 seconds
8. Your site is live at: `https://aquastar.vercel.app`

---

## STEP 4 — Custom Domain (optional)

If you have a domain like `aquastarmangalore.com`:

1. In Vercel dashboard → your project → **Settings** → **Domains**
2. Click **Add Domain** → type your domain
3. Vercel gives you DNS records to add at your domain registrar
4. Done — SSL certificate is automatic and free

---

## STEP 5 — Update the site anytime

Just update `index.html` on GitHub:
- Go to your repo → click `index.html` → click the pencil icon → edit → commit
- Vercel automatically redeploys in ~20 seconds

Or use Git:
```bash
git add index.html
git commit -m "Update products section"
git push
```

---

## PERFORMANCE CHECKLIST (already done in code)

- [x] Smooth scroll — `scroll-behavior: smooth`
- [x] 60 FPS canvas — `requestAnimationFrame` only, no `setInterval`
- [x] Images lazy loaded — `loading="lazy"` on all img tags
- [x] Font preconnect — `<link rel="preconnect">` for Google Fonts
- [x] `-webkit-font-smoothing: antialiased` — crisp text
- [x] `text-rendering: optimizeSpeed` — faster text rendering
- [x] `will-change: transform` on canvas — GPU acceleration hint
- [x] Mobile tap highlight removed — `-webkit-tap-highlight-color: transparent`
- [x] Cloudinary CDN — images served from global edge network
- [x] Supabase — database reads are fast (hosted in Mumbai region)

---

## SEO CHECKLIST (already done in code)

- [x] Meta description with keywords
- [x] Meta keywords tag
- [x] Open Graph tags (WhatsApp/Facebook link preview)
- [x] Twitter Card tags
- [x] Canonical URL
- [x] Schema.org structured data (PetStore type — Google rich results)
- [x] Semantic HTML (section, nav, footer tags)
- [x] Alt text on all images
- [x] Page title optimized

---

## WHAT'S NOT IN SCOPE (requires backend/paid services)

| Feature | What's needed |
|---------|--------------|
| Secure payment gateway | Razorpay/Stripe + backend server |
| Instagram/Facebook feed | Meta API + OAuth |
| Order management | Backend database + auth |

These require a proper backend (Node.js/Python server).
The current setup is a frontend-only static site.

---

## YOUR LIVE STACK AFTER DEPLOY

```
Vercel          → hosts index.html (free, global CDN)
Cloudinary      → stores images (free tier: 25GB)
Supabase        → stores gallery data (free tier: 500MB)
Google Fonts    → typography (free)
Total cost      → ₹0/month
```
