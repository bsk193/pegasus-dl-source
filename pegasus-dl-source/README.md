# Private Pegasus DL+ Catalog

A hand-curated catalog for [Pegasus DL+](https://github.com/pegasus-ps5/pegasus-dl).
Every entry points at a file on our own B2 storage — no third-party catalogs,
no scene-page provenance.

- Catalog file: `catalog.json`
- Posters: `posters/<TITLEID>.jpg`, served by the same Pages site
- Current entries: 1 (Ghost of Tsushima DC, USA, v02.024, FPKG)

## Live URL

```
https://bsk193.github.io/pegasus-dl-source/pegasus-dl-source/catalog.json
```

Add it in Pegasus DL+ → Sources. Pushes to `main` redeploy automatically
(Pages serves the repo root of `main`). Repo: `bsk193/pegasus-dl-source`.

## Add an entry

1. Upload the file to the B2 `x-games` bucket under `ps5/`.
2. Grab its metadata from the bucket's own headers:

   ```bash
   curl -sI "https://f003.backblazeb2.com/file/x-games/ps5/<File>.pkg" \
     | grep -iE 'content-length|x-bz-info-large_file_sha1'
   ```

3. Append a package object to `packages` in `catalog.json`:

   ```json
   {
     "titleId": "PPSA02225",
     "title": "Game Title",
     "category": "game",
     "version": "02.024",
     "filename": "File.pkg",
     "sizeBytes": 65940261190,
     "checksumSha1": "<from x-bz-info-large_file_sha1>",
     "posterUrl": "https://bsk193.github.io/pegasus-dl-source/pegasus-dl-source/posters/<TITLEID>.jpg",
     "description": "Region: USA\nFirmware: 4.xx+",
     "downloadLinks": [
       { "name": "USA (v02.024) (FPKG)", "url": "https://f003.backblazeb2.com/file/x-games/ps5/File.pkg" }
     ]
   }
   ```

4. Drop a poster at `posters/<TITLEID>.jpg`, commit, push. Live ~a minute later.

### Notes

- `titleId` groups all links under one game tile; `downloadLinks[].name`
  labels each link (use it to distinguish region / version / format).
- `sizeBytes` + `checksumSha1` come from B2's response headers, so a wrong or
  swapped file at the URL is detectable after download
  (`sha1sum File.pkg` should match `checksumSha1`).
- The B2 native URL is unauthenticated: anyone holding the exact URL can
  fetch the file. Treat it as obscure, not secret.
- Offline/LAN fallback: from this folder run
  `python3 -m http.server 8080` and add
  `http://<LAN-IP>:8080/catalog.json` (plaintext — trusted LAN only).
