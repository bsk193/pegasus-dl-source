# Private Pegasus DL+ Catalog

A hand-curated catalog for [Pegasus DL+](https://github.com/pegasus-ps5/pegasus-dl)
(pegasus-dl). No auto-scraped junk: every entry here was picked manually.

- Catalog file: `catalog.json`
- Current entries: 1 (Ghost of Tsushima DC, USA, v02.024, FPKG)

## Serve it

**This catalog is live on GitHub Pages (repo: `bsk193/pegasus-dl-source`):**

```
https://bsk193.github.io/pegasus-dl-source/pegasus-dl-source/catalog.json
```

Add that URL in Pegasus DL+ → Sources. Push to `main` to publish updates
(Pages deploys the repo root of `main`).

Other options:

**LAN (truly private):** from this folder run:

```bash
python3 -m http.server 8080
```

Then in Pegasus DL+ on the PS5 add the source:
`http://<your-machine-LAN-IP>:8080/catalog.json`

**GitHub:** push this folder to a repo and use the raw URL
(`https://raw.githubusercontent.com/<user>/<repo>/main/pegasus-dl-source/catalog.json`).
Note: raw URLs of *private* repos need a token; use a public-but-unlisted repo
or the LAN option for real privacy.

## Add an entry

Append an object to `packages` in `catalog.json` and bump nothing else
(`version` stays 1; it is the catalog format version, not a counter):

```json
{
  "titleId": "PPSA02225",
  "title": "Game Title",
  "category": "game",
  "version": "02.024",
  "posterUrl": "https://.../poster.jpg",
  "description": "Region: USA\nFirmware: 4.xx+",
  "downloadSource": "https://origin-page.example/",
  "downloadLinks": [
    { "name": "USA (v02.024) (FPKG)", "url": "https://host.tld/f/xxxx" }
  ]
}
```

- `titleId` groups all links under one game tile in the app.
- `downloadLinks[].name` is the label shown when picking a link; use it to
  distinguish region / version / format.
- Multiple links per entry are fine (mirrors or parts); give each a clear name.
- `posterUrl`, `description`, `downloadSource`, `category`, `version` are
  optional but recommended.
- `filename` pins the exact artifact name the host should serve (title ID and
  version are usually embedded in it) — record it when the host page shows it.
- `checksum` (sha256 hex) and `sizeBytes` (exact byte count) when you know
  them; most cyberlocker pages list neither, so treat every downloaded FPKG as
  untrusted input and sanity-check the filename after download.

### Trust model

These catalogs point at mutable cyberlocker pages (often captcha-gated; use
the app's built-in browser to capture the download). Nothing here
cryptographically verifies a downloaded file, so the practical integrity
checks are: the pinned `filename`, the `titleId`/`version` fields, and the
fact that every entry was hand-picked from the page in `downloadSource`.
The catalog itself is served over HTTPS via GitHub Pages; the
`python3 -m http.server` LAN option is plaintext — use it only on a trusted
LAN.

### Getting links off dlpsgame.com

The "Link Download" sections on dlpsgame pages render as "Đang giải mã nội
dung..." because the content is base64-encoded in the HTML: fetch the page,
find the `data-payload="..."` attribute in the `div.secure-data` right under
the entry heading, and base64-decode it to get the real HTML with the host
links (akirabox / vikingfile / 1fichier / filecrypt). The intermediate
`downloadgameps3.net/archives/...` links sit behind a Cloudflare challenge, so
a plain-text browser fetch fails; opening the page in a real browser (or the
decrypted page's mirror section) reveals the direct host link.
