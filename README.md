# voltek

Landing page for Voltek Industries.

- **Production:** https://voltekindustries.co.in/ (the client's domain). `index.html` in this repo is the production version: it is indexable and its canonical, Open Graph and JSON-LD URLs use this domain.
- **Preview:** https://voltek.framememora.com/ (the host in `CNAME`), for client review. `.github/workflows/static.yml` deploys it on every push to `main`. The workflow adjusts only the deployed copy:
  - sets `robots` to `noindex, nofollow`, so the preview stays out of search results
  - rewrites `https://voltekindustries.co.in/` URLs to the preview host, so link previews show the image
  - publishes only the site files (`index.html`, `voltek-logo.jpg`, `assets/`)

## Going live

Upload these to the web root on the client's host, as they are:

```
index.html
voltek-logo.jpg
assets/
robots.txt
sitemap.xml
```

No edits are needed.
