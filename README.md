# writeups.sh

HTB and Academy writeups for people who actually get stuck.

---

## Setup

### First time

```bash
gem install bundler
bundle install
```

### Run locally

```bash
bundle exec jekyll serve
# → http://localhost:4000
```

### Build for production

```bash
bundle exec jekyll build
npx pagefind --site _site   # builds the search index
```

---

## Adding a new writeup

1. Copy `_posts/TEMPLATE-copy-for-new-writeup.md`
2. Rename it: `_posts/YYYY-MM-DD-box-name.md`  
   e.g. `_posts/2026-03-26-lame.md`
3. Fill in the front matter (the section between `---` at the top)
4. Write your writeup using the template sections
5. `git add . && git commit -m "writeup: Box Name" && git push`
6. Done — homepage updates automatically ✓

---

## Front matter reference

| Field | Required | Example |
|-------|----------|---------|
| `title` | ✓ | `"Lame"` |
| `date` | ✓ | `2026-03-26` |
| `platform` | ✓ | `HackTheBox` or `HTB Academy` |
| `difficulty` | ✓ | `Easy`, `Medium`, or `Hard` |
| `os` | ✓ | `Linux` or `Windows` |
| `status` | ✓ | Always `Retired` |
| `stuck_summary` | ✓ | One honest sentence for the card |
| `tags` | ✓ | List of lowercase tags |

---

## Deployment

Push to `main` → GitHub Pages builds automatically.

For custom domain: add `CNAME` file to root with your domain name.

For Cloudflare: point your domain's nameservers to Cloudflare, proxy the A record.
