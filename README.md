# n8n-nodes-tiktok-scraper

An **n8n community node** that scrapes TikTok profile posts using **Puppeteer**.  
Supports video and photo posts, profile counters (followers / following / likes), cookies, proxy, concurrency, and anti-CAPTCHA-friendly pacing.

> **Note:** Puppeteer requires a running Chromium/Chrome instance. It may not work on **n8n Cloud** — self-hosting is recommended.

---

## Features

- Scrape a profile grid for **video**, **photo**, or **all** post types
- Per-item fields: `type_post`, `video_id`, view/like/comment/share/save counts, caption, hashtags, music, duration
- Timestamp extracted from the **TikTok video ID** as a fallback (works even when DOM metadata is unavailable)
- Profile counters (`followers`, `following`, `likes`) attached to every output item
- Optional **profile summary** item emitted at the start of output
- Configurable cookies, proxy URL, custom User-Agent, extra HTTP headers, and viewport
- **Block Media** mode: skips images, fonts, stylesheets, and media during scrolling for faster execution
- Per-item retry with exponential backoff

---

## Installation

### From the n8n UI (Community Nodes)

1. Go to **Settings → Community Nodes → Install**
2. Enter the package name: `n8n-nodes-tiktok-scraper`
3. After installation, search for **TikTok Scraper** in the node panel.

> If "Community Nodes" is not visible, set the environment variable:  
> `N8N_COMMUNITY_PACKAGES_ENABLED=true`

### Via custom extensions folder (self-hosted)

```bash
export N8N_CUSTOM_EXTENSIONS=/path/to/extensions
cd $N8N_CUSTOM_EXTENSIONS
npm i n8n-nodes-tiktok-scraper
# restart n8n; the node will appear automatically
```

---

## Requirements

- Node.js **>= 20.15**
- A runtime that can launch **Chromium or Chrome** for Puppeteer
- Docker: increase shared memory to avoid Chrome crashes — `shm_size: "1gb"`

---

## Node Parameters

### Required

| Parameter | Description |
|-----------|-------------|
| **Username** | TikTok handle without `@` (e.g. `tiktok`) |

### Core Options

| Parameter | Default | Description |
|-----------|---------|-------------|
| **Max Videos** | `100` | Maximum posts to scrape. `0` = unlimited |
| **Post Type** | `All` | `All`, `Video`, or `Photo` |
| **Concurrency** | `4` | Number of post tabs opened in parallel (1–10) |
| **Per-Video Delay (MS)** | `500` | Base delay between post scrapes in ms |
| **Headless** | `True` | Chromium headless mode: `True`, `New`, or `False` |
| **Timeout (MS)** | `45000` | Navigation and selector wait timeout |
| **Hard Scroll Timeout (MS)** | `600000` | Maximum total time allowed for profile scrolling |
| **User Agent** | _(random)_ | Override the default browser User-Agent string |

### Additional Options

| Parameter | Default | Description |
|-----------|---------|-------------|
| **Block Media (Faster)** | `true` | Block images, media, fonts, and stylesheets during scrolling |
| **Cookies (JSON Array)** | — | Browser session cookies as a JSON array (see example below) |
| **Emit Profile Summary** | `false` | Prepend a summary item with follower/like/following counts |
| **Executable Path** | — | Custom Chrome/Chromium binary path |
| **Extra Headers** | — | Additional HTTP headers (key/value pairs) |
| **Proxy URL** | — | e.g. `http://user:pass@host:port` |
| **Retries** | `2` | Retry attempts per post on failure (0–10) |
| **Viewport Width / Height** | `1366 × 768` | Browser viewport dimensions |

#### Cookies example

```json
[
  { "name": "ttwid", "value": "...", "domain": ".tiktok.com", "path": "/", "httpOnly": true, "secure": true },
  { "name": "sid_tt", "value": "...", "domain": ".tiktok.com", "path": "/", "httpOnly": true, "secure": true }
]
```

---

## Output Schema

Each output item contains:

```json
{
  "video_id": "7345678901234567890",
  "type_post": "video",
  "url": "https://www.tiktok.com/@user/video/7345678901234567890",
  "caption": "Sample caption #tag",
  "created_at": "2024-05-01T12:34:56.000Z",
  "created_at_ts": 1714566896,
  "views": 1200,
  "views_grid": 1200,
  "likes": 150,
  "comments": 12,
  "shares": 3,
  "saves": 0,
  "duration": 17,
  "author_username": "user",
  "music_title": "Track",
  "music_author": "Artist",
  "hashtags": ["tag"],
  "profile_following": 827,
  "profile_followers": 70700,
  "profile_likes": 321600
}
```

> Numeric fields default to `0` when the source value is missing or `null`.

When **Emit Profile Summary** is enabled, a summary item is prepended:

```json
{
  "username": "user",
  "profile_following": 827,
  "profile_followers": 70700,
  "profile_likes": 321600,
  "scraped_videos": 50
}
```

---

## Anti-CAPTCHA Tips

- Provide **valid cookies** from a real Chrome session (logged in, any CAPTCHA already solved)
- Use a **residential proxy** from the same country as your cookies
- Set **Concurrency** to `1` and **Per-Video Delay** to `1200–2000 ms`
- Keep **Block Media** enabled
- Set `Accept-Language` via Extra Headers to match your region, e.g. `vi-VN,vi;q=0.9,en-US,en;q=0.8`
- Use a realistic, consistent **User Agent**

---

## Docker Example

```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    environment:
      N8N_COMMUNITY_PACKAGES_ENABLED: "true"
      N8N_CUSTOM_EXTENSIONS: /custom
      PUPPETEER_EXECUTABLE_PATH: /usr/bin/chromium
      TZ: Asia/Ho_Chi_Minh
    volumes:
      - ./custom:/custom
      - ./n8n_data:/home/node/.n8n
    shm_size: "1gb"
```

If your base image does not include Chromium, install it (e.g. via Debian/Ubuntu packages) and set `PUPPETEER_EXECUTABLE_PATH` to the binary path.

---

## Troubleshooting

| Error | Fix |
|-------|-----|
| CAPTCHA detected | Supply cookies, use a residential proxy, lower concurrency, increase delay |
| Navigation timeout exceeded | Increase `Timeout (MS)`; verify proxy, cookies, and headers are valid |
| Chromium not found | Install Chromium or set `Executable Path` to the binary |
| No posts scraped | Profile may be private or the username is incorrect |
| Crashes on Docker | Add `shm_size: "1gb"` to your Compose service |
| Node not visible in n8n Cloud | Puppeteer is not supported on n8n Cloud; use a self-hosted instance |

---

## Development

```bash
npm run lint        # ESLint check
npm run build       # Compile TypeScript + copy icons
npm publish --access public
```

---

## Disclaimer

This project is for educational and automation purposes. Use responsibly and in compliance with TikTok's Terms of Service and applicable local laws.

---

## License

[MIT](LICENSE.md)
