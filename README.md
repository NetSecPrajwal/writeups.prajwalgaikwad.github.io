# pg/writes — writeups.prajwalgaikwad.in

Clean minimal dark blog built on Jekyll + GitHub Pages.

---

## Deploy in 5 minutes

### 1. Create a new GitHub repo
Name it anything — e.g. `writes` or `blog`.

### 2. Push this folder
```bash
cd blog
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin https://github.com/NetSecPrajwal/YOUR-REPO-NAME.git
git push -u origin main
```

### 3. Enable GitHub Pages
`Settings → Pages → Source: Deploy from branch → main / (root) → Save`

### 4. Set custom domain
In `Settings → Pages → Custom domain`, type:
```
writeups.prajwalgaikwad.in
```

### 5. Add DNS record (Hostinger)
In your Hostinger DNS panel — add ONE new record:
```
Type:   CNAME
Name:   writeups
Target: NetSecPrajwal.github.io
TTL:    600
```
Your existing A records for the main portfolio stay untouched.

Wait 10–30 minutes, then enable **Enforce HTTPS**.

---

## Writing a post

Create a file in `_posts/` named `YYYY-MM-DD-slug.md`:

```markdown
---
title: "Your Post Title"
description: "One line summary shown in the post list."
category: CTF
tags: [nmap, linux, privilege-escalation]
---

Your content here in plain Markdown.
```

**Valid categories:** `CTF`, `Research`, `Security`, `Personal`, `Notes`

Push to GitHub → live in ~60 seconds. No build step needed.

---

## File structure
```
blog/
├── _config.yml          ← Site settings
├── CNAME                ← Custom subdomain
├── index.html           ← Homepage with post list + category filter
├── about/
│   └── index.md         ← About page
├── _layouts/
│   ├── default.html     ← Nav + footer wrapper
│   ├── post.html        ← Individual post template
│   └── page.html        ← Static page template
├── _posts/              ← Your posts go here
└── assets/
    └── style.css        ← All styles
```
