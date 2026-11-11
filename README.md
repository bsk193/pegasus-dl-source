# x-games — Private Pegasus DL+ Catalog

A hand-curated catalog for [Pegasus DL+](https://github.com/pegasus-ps5/pegasus-dl).
Every entry points at a file on our own B2 storage — no third-party catalogs,
no scene-page provenance.

- PS5 catalog: `catalog.json` (repo root) — PPSA title IDs, bucket `ps5/`
- PS4 catalog: `catalog-ps4.json` (repo root) — CUSA title IDs, bucket `ps4/`,
  same format
- Posters: `posters/<TITLEID>.jpg`
- Current entries: 2 PS5 (Ghost of Tsushima DC USA v02.024 · Ghost of Yōtei
  Complete Edition USA) · 0 PS4

## Live URLs

```
https://bsk193.github.io/pegasus-dl-source/catalog.json      (PS5)
https://bsk193.github.io/pegasus-dl-source/catalog-ps4.json  (PS4)
```

Add each in Pegasus DL+ → Sources (one per console library). Pushes to
`main` redeploy automatically (Pages serves the repo root of `main`).
Repo: `bsk193/pegasus-dl-source`.

## Add an entry

1. Upload the file to the B2 `x-games` bucket under `ps5/` (PS5) or `ps4/`
   (PS4), and add the entry to the matching catalog file.
2. Get the exact byte size with a ranged GET through the download host
   (needs a browser-like UA; plain curl gets Cloudflare-challenged):

   ```bash
   curl -s -A "Mozilla/5.0" -r 0-0 -o /dev/null -D - -L \
     "https://dl-xgames.oneup.pt/6c8fea364f470eee9f744cb00ee5b5ef/f/<File>.pkg" \
     | grep -i content-range        # bytes 0-0/<TOTAL>
   ```

   If the file also exists under the native B2 URL
   (`https://f003.backblazeb2.com/file/x-games/ps5/<File>.pkg`), a plain
   `curl -sI` there additionally yields `x-bz-info-large_file_sha1`.

3. Append a package object to `packages` in `catalog.json`. Multi-file games
   (BASE + DLC, parts) are multiple links under the same package; size lives
   per link, and link names should make install order obvious:

   ```json
   {
     "titleId": "PPSA26344",
     "title": "Game Title",
     "category": "game",
     "version": "02.024",
     "posterUrl": "https://bsk193.github.io/pegasus-dl-source/posters/<TITLEID>.jpg",
     "description": "Region: USA\nBASE: 110.1 GiB (118,154,628,798 bytes)\nDLC: 1.4 MB (1,386,407 bytes)",
     "downloadLinks": [
       { "name": "USA BASE (FPKG)", "url": "https://dl-xgames.oneup.pt/6c8fea364f470eee9f744cb00ee5b5ef/f/Game_BASE.pkg", "sizeBytes": 118154628798 },
       { "name": "USA DLC (FPKG) — install after BASE", "url": "https://dl-xgames.oneup.pt/6c8fea364f470eee9f744cb00ee5b5ef/f/Game_DLC.pkg", "sizeBytes": 1386407 }
     ]
   }
   ```

4. Drop a poster at `posters/<TITLEID>.jpg`, commit, push. Live ~a minute later.

### Notes

- `titleId` groups all links under one game tile; `downloadLinks[].name`
  labels each link (use it to distinguish region / version / format).
- `version` is optional — include it when the filename or source states it,
  omit it when unknown.
- Per-link `sizeBytes` comes from the ranged-GET probe, so a wrong or swapped
  file at the URL is detectable after download. A package-level
  `checksumSha1` is only valid for single-file entries whose file is on the
  native B2 URL (the CDN's `etag` is not a content hash) — don't pin a hash
  you can't verify.
- The B2 native URL is unauthenticated: anyone holding the exact URL can
  fetch the file. Treat it as obscure, not secret.
- `dl-xgames.oneup.pt` is UA-gated by Cloudflare: browser-like UAs get a
  clean 302 to the CDN; non-browser UAs (default curl/libcurl) get a JS
  challenge (403). If downloads fail on the PS5 with an HTML error instead
  of a .pkg, add a Cloudflare WAF skip rule for the
  `/6c8fea364f470eee9f744cb00ee5b5ef/f/*` path (or allow the app's UA).
- Offline/LAN fallback: from this folder run
  `python3 -m http.server 8080` and add
  `http://<LAN-IP>:8080/catalog.json` (plaintext — trusted LAN only).
