# Publishing the Super Stock 500 site

A static site (no build step) for https://kdevelopment2026.github.io/super-stock-500/. It is a local git repo with no
remote and is **not published yet**. Nothing below has been run.

The App Store listing already points at these URLs, and App Review needs the privacy and support pages to load:

| Field in App Store Connect | URL |
| --- | --- |
| Marketing URL | https://kdevelopment2026.github.io/super-stock-500/ |
| Support URL | https://kdevelopment2026.github.io/super-stock-500/support.html |
| Privacy Policy URL | https://kdevelopment2026.github.io/super-stock-500/privacy.html |

So publish the site **before** submitting version 1.0 for review.

## 1. Check it locally

```sh
cd "Super Stock 500 iOS/site"
python3 -m http.server 8090
# open http://127.0.0.1:8090/ ; the 404 page only works on GitHub Pages (absolute /super-stock-500/ paths)
```

## 2. Publish to GitHub Pages

The repository must be called `super-stock-500` under the `kdevelopment2026` account, so that the URL matches the
canonical tags, sitemap and App Store fields.

```sh
cd "Super Stock 500 iOS/site"
gh repo create kdevelopment2026/super-stock-500 --public --source . --remote origin --push
gh api -X POST repos/kdevelopment2026/super-stock-500/pages -f "source[branch]=main" -f "source[path]=/"
gh api repos/kdevelopment2026/super-stock-500/pages --jq .html_url   # wait ~1 minute, then open it
```

(Or on github.com: create the public repo `super-stock-500`, push `main`, then Settings > Pages > Deploy from a
branch > `main` / root.) Enforce HTTPS stays on by default.

Then check:

```sh
for p in "" support.html privacy.html robots.txt sitemap.xml 35805b78bf41a5f4d33feea06cafe3bc.txt; do
  curl -s -o /dev/null -w "%{http_code}  $p\n" "https://kdevelopment2026.github.io/super-stock-500/$p"; done
```

All should print `200`.

## 3. Google Search Console

GitHub Pages project sites cannot hold a DNS record for `github.io`, so use the **URL prefix** property with the
HTML-tag method.

1. https://search.google.com/search-console > Add property > **URL prefix** >
   `https://kdevelopment2026.github.io/super-stock-500/`.
2. Choose **HTML tag**. Copy the `<meta name="google-site-verification" content="...">` tag into the `<head>` of
   `index.html` (just under the canonical link), commit, push, wait for Pages to redeploy, then press **Verify**.
3. Sitemaps > add `sitemap.xml` > Submit.
4. URL inspection > paste the home page URL > **Request indexing**. Repeat for `support.html` and `privacy.html`.
5. After a few days: Enhancements should list the FAQ structured data; check it with
   https://search.google.com/test/rich-results (the page has `VideoGame` / `MobileApplication`, `Offer` and
   `FAQPage` JSON-LD, and deliberately no ratings).

## 4. Bing Webmaster Tools (also feeds DuckDuckGo, Yahoo and Ecosia)

1. https://www.bing.com/webmasters > Add a site > `https://kdevelopment2026.github.io/super-stock-500/`. If Search
   Console is already verified, **Import from Google Search Console** is the quickest route. Otherwise choose the
   meta-tag method: add `<meta name="msvalidate.01" content="...">` to `index.html`, push, then Verify.
2. Sitemaps > Submit sitemap > `https://kdevelopment2026.github.io/super-stock-500/sitemap.xml`.
3. URL Inspection > the home page > Request indexing.

### IndexNow (instant notification to Bing, Yandex, Seznam and others)

The key file is already in the site root: `35805b78bf41a5f4d33feea06cafe3bc.txt` (its content is the key). After
the site is live, and again after any content change, notify IndexNow once:

```sh
curl -s -X POST "https://api.indexnow.org/indexnow" -H "Content-Type: application/json; charset=utf-8" -d '{
  "host": "kdevelopment2026.github.io",
  "key": "35805b78bf41a5f4d33feea06cafe3bc",
  "keyLocation": "https://kdevelopment2026.github.io/super-stock-500/35805b78bf41a5f4d33feea06cafe3bc.txt",
  "urlList": [
    "https://kdevelopment2026.github.io/super-stock-500/",
    "https://kdevelopment2026.github.io/super-stock-500/support.html",
    "https://kdevelopment2026.github.io/super-stock-500/privacy.html"
  ]
}'
```

`200` or `202` means accepted. A key file in a subfolder only covers URLs under that folder, which is all this site
has. Don't send unchanged URLs repeatedly.

## 5. After the app goes live

- The "See it on the App Store" button and the Smart App Banner (`apple-itunes-app`, app id 6827441422) only work
  once the app is on sale; until then the App Store link shows "not available".
- Once the app has real ratings, an `aggregateRating` may be added to the JSON-LD **only** with the true App Store
  figures. Never add made-up ratings.
- If the price of the Full Grid changes, update the `£2.99` in `index.html` (the FAQ text and the JSON-LD `Offer`).
- Update `<lastmod>` in `sitemap.xml` when a page changes, then ping IndexNow as above.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | home: hero, features, screenshots, tracks, free vs Full Grid table, FAQ; JSON-LD |
| `privacy.html` | privacy policy (Data Not Collected; purchases through Apple StoreKit) |
| `support.html` | controls, controllers, restoring purchases, contact |
| `404.html` | not-found page for GitHub Pages (noindex) |
| `style.css` | the only stylesheet; fonts are self-hosted in `assets/fonts` (Saira Condensed, Barlow; SIL OFL) |
| `assets/` | icons, Open Graph image (1200 x 630), screenshots (JPEG + WebP) captured from the game |
| `robots.txt`, `sitemap.xml` | crawl rules and the sitemap (with image entries) |
| `35805b78bf41a5f4d33feea06cafe3bc.txt` | IndexNow key |
| `.nojekyll` | serve files as they are (no Jekyll processing) |
