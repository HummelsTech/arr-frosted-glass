# Frosted Glass — *Arr Theme

A translucent frosted glass dark theme for Sonarr, Radarr, Prowlarr and Bazarr. Semi-transparent panels with backdrop-blur effects over a customizable background.

![Frosted Glass Sonarr](screenshots/sonarr.png)

## Features

- Translucent sidebar, header, cards and modals with `backdrop-filter: blur()`
- Liquid Glass inspired inner bevel on panels
- Fully transparent toolbar with subtle rounded buttons
- Customizable background image or gradient
- Works with all *Arr apps that share the same UI framework
- Designed to layer on top of ThemePark's `dark` base theme

## Supported Apps

| App | Status | Notes |
|-----|--------|-------|
| Sonarr v4 | Tested | Full support including series detail pages |
| Radarr v5 | Tested | Full support including movie detail pages |
| Prowlarr | Tested | Table-based layout supported |
| Bazarr | Tested | Subtitle management UI supported |

## Installation

### Prerequisites

- [Nginx Proxy Manager](https://nginxproxymanager.com/) (NPM) as reverse proxy
- [ThemePark](https://docs.theme-park.dev/) DOCKER_MODS with `TP_THEME=dark` as base theme

### Step 1: Host the CSS

Host `frosted-glass.css` on a web server accessible from the browser. Options:

**Option A — Self-host via your web server:**
```bash
# Copy to any web-accessible location
cp frosted-glass.css /var/www/html/themes/
```

**Option B — Use a CDN / raw GitHub:**
```
https://raw.githubusercontent.com/TheHumLab/arr-frosted-glass/main/frosted-glass.css
```

### Step 2: Configure Docker (ThemePark base)

Add ThemePark DOCKER_MODS to your *Arr containers with `TP_THEME=dark`:

```yaml
services:
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    environment:
      - DOCKER_MODS=ghcr.io/themepark-dev/theme.park:sonarr
      - TP_THEME=dark
    # ... rest of your config
```

### Step 3: Inject CSS via Nginx Proxy Manager

For each *Arr proxy host in NPM, add this to the **Custom Nginx Configuration** (click the gear icon ⚙️):

Find the `location /` block and add (or replace existing `sub_filter`):

```nginx
sub_filter '</head>' '<link rel="stylesheet" href="YOUR_CSS_URL/frosted-glass.css" /></head>';
sub_filter_once on;
```

Replace `YOUR_CSS_URL` with where you hosted the CSS in Step 1.

**Example with GitHub raw URL:**
```nginx
sub_filter '</head>' '<link rel="stylesheet" href="https://raw.githubusercontent.com/TheHumLab/arr-frosted-glass/main/frosted-glass.css" /></head>';
sub_filter_once on;
```

> **Note:** If you already have Authentik forward-auth or other `sub_filter` rules in the `location /` block, add the CSS `sub_filter` line alongside them. The `sub_filter_once on;` only needs to appear once.

### Step 4: Custom Background (Optional)

The default theme uses a dark gradient. To use a custom background image:

1. Host your image on a web server
2. Create an override CSS file:

```css
body::before {
  background: center / cover no-repeat url('YOUR_IMAGE_URL') !important;
  filter: blur(4px) saturate(1.2) !important;
}
```

3. Add a second `<link>` in the NPM sub_filter **after** the main CSS:

```nginx
sub_filter '</head>' '<link rel="stylesheet" href="YOUR_CSS_URL/frosted-glass.css" /><link rel="stylesheet" href="YOUR_CSS_URL/my-background.css" /></head>';
sub_filter_once on;
```

> **Tip:** Use a blurred/bokeh wallpaper for the best frosted glass effect. The CSS applies an additional `blur(4px)` on top.

## Customization

### Adjusting transparency

Edit the `rgba()` alpha values in the CSS:
- **Sidebar:** `rgba(25, 28, 45, 0.2)` — lower = more transparent
- **Header:** `rgba(25, 28, 45, 0.55)` — higher = more opaque
- **Cards:** `rgba(30, 30, 30, 0.45)` — adjust to taste

### Accent color

The theme uses orange (`rgb(255, 152, 0)`) as accent color, matching the *Arr apps' default. Search and replace in the CSS to change it.

### Background blur

The background image blur is controlled by:
```css
filter: blur(4px) saturate(1.2);
```
Increase `blur()` for a smoother look, decrease for a sharper background.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| CSS not loading | Check browser DevTools Network tab for 404s. Verify the CSS URL is accessible. |
| No transparency | ThemePark's CSS may override. Ensure `TP_THEME=dark` (not a theme with solid backgrounds). |
| Background not showing | Check that `body::before` isn't blocked. The image URL must be accessible from the browser. |
| Sub_filter not working | NPM needs `proxy_set_header Accept-Encoding "";` before `sub_filter` to disable gzip. |
| Sidebar still opaque | Restart the *Arr container after changing `TP_THEME`. Clear browser cache. |

## Credits

- [ThemePark](https://theme-park.dev/) for the excellent dark base theme
- Inspired by Apple's Liquid Glass and the [Frosted Glass](https://github.com/wessamlauf/homeassistant-frosted-glass-themes) Home Assistant theme

## License

MIT
