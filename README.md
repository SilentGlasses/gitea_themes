# Nord Themes for Gitea

A set of standalone Gitea themes based on the official Nord palette. Gitea keeps the existing page structure and interactions while these files provide the color, typography, navigation, form, code, diff, and semantic-state variables.

## Available themes

| Theme                  | Mode  | Best for                                         | Accent |
| ---------------------- | ----- | ------------------------------------------------ | ------ |
| **Nord Polar Night**   | Dark  | Neutral everyday dark mode                       | Cyan   |
| **Nord Snow Storm**    | Light | Bright workspaces and daytime use                | Blue   |
| **Nord Frost**         | Dark  | A cool blue-green interface                      | Teal   |
| **Nord Aurora**        | Dark  | A more expressive purple/orange interface        | Violet |
| **Nord Accessibility** | Dark  | Stronger text, border, input, and focus contrast | Teal   |

> [!NOTE]
> **Nord Accessibility** is a higher-contrast variant of **Polar Night**, not a separate layout or large-text mode.
>
> It keeps the same dark Nord surfaces while making structure and controls easier to distinguish:
>
> - Nord6 borders are used for inputs, panels, and console boundaries;
> - Nord5/Nord6 are used more often for secondary text and placeholders
> - Nord7/Nord8 provide clearer interactive accents.
>
> Choose it when muted borders, placeholder text, or low-contrast code comments are difficult to read.

### Installation

1. Run the installation commands on the remote Gitea server.
2. Transfer the CSS files from the machine where this repository is checked out with `scp` or another deployment method.
3. Configure and restart the Gite server
4. Enjoy the themes!!!

> [!NOTE]
> The commands below use `/var/lib/gitea/custom` because that is the working `CustomPath` for my installation.
>
> If your running service reports a different path, replace it everywhere with your `<CustomPath>`.

### Verify the active CustomPath

The `CustomPath` used by the running service is authoritative. Check the systemd command and resolve the path with the same configuration:

```bash
sudo systemctl show gitea --property=ExecStart --property=User
```

You should see a response like:

```
ExecStart={ path=/usr/local/bin/gitea ; [...]
User=git
```

Verify using the `path=` from the result above:

```
sudo -u git /usr/local/bin/gitea --config /etc/gitea/app.ini help | grep -i -A2 'CustomPath'
```

> [!NOTE]
> If the command reports `/etc/gitea/custom` instead of `/var/lib/gitea/custom`, install the files under `/etc/gitea/custom/public/assets/css/`

### 1. Create the custom theme directory

```bash
sudo install -d -o git -g git -m 0755 \
  /var/lib/gitea/custom/public/assets/css/

sudo ls -ld /var/lib/gitea/custom/public/assets/css/
```

The directory should be owned by the `git` service account.

### 2. Copy the Nord themes

Copy the theme files from the machine containing this repository if you cloned it to a different computer:

```bash
scp theme-nord-polar-night.css git.domain.com:/tmp/
scp theme-nord-snow-storm.css git.domain.com:/tmp/
scp theme-nord-frost.css git.domain.com:/tmp/
scp theme-nord-aurora.css git.domain.com:/tmp/
scp theme-nord-accessibility.css git.domain.com:/tmp/
```

Then, on the Gitea server:

```bash
sudo install -o git -g git -m 0644 \
  /tmp/theme-nord-polar-night.css \
  /var/lib/gitea/custom/public/assets/css/
sudo install -o git -g git -m 0644 \
  /tmp/theme-nord-snow-storm.css \
  /var/lib/gitea/custom/public/assets/css/
sudo install -o git -g git -m 0644 \
  /tmp/theme-nord-frost.css \
  /var/lib/gitea/custom/public/assets/css/
sudo install -o git -g git -m 0644 \
  /tmp/theme-nord-aurora.css \
  /var/lib/gitea/custom/public/assets/css/
sudo install -o git -g git -m 0644 \
  /tmp/theme-nord-accessibility.css \
  /var/lib/gitea/custom/public/assets/css/
```

The resulting directory should contain:

```text
/var/lib/gitea/custom/public/assets/css/
├── theme-nord-polar-night.css
├── theme-nord-snow-storm.css
├── theme-nord-frost.css
├── theme-nord-aurora.css
└── theme-nord-accessibility.css
```

### 3. Verify the files

```bash
sudo find /var/lib/gitea/custom/public/assets/css/ \
  -maxdepth 1 \
  -type f \
  -name 'theme-*.css' \
  -print

sudo ls -la /var/lib/gitea/custom/public/assets/css/
```

Each file should be owned by `git:git` and have mode `0644`:

```text
-rw-r--r-- 1 git git ... theme-nord-accessibility.css
-rw-r--r-- 1 git git ... theme-nord-aurora.css
-rw-r--r-- 1 git git ... theme-nord-frost.css
-rw-r--r-- 1 git git ... theme-nord-polar-night.css
-rw-r--r-- 1 git git ... theme-nord-snow-storm.css
```

### 4. Configure the available themes

Edit the `app.ini` used by the running Gitea service:

```bash
sudo vi /etc/gitea/app.ini
```

Add or update the existing `[ui]` section:

```ini
[ui]
DEFAULT_THEME = gitea-auto
THEMES = nord-polar-night,nord-snow-storm,nord-frost,nord-aurora,nord-accessibility
```

`THEMES` contains names without the `theme-` prefix and `.css` extension. For example:

```text
theme-nord-polar-night.css
      └──────────────┘
      nord-polar-night
```

`DEFAULT_THEME = gitea-auto` keeps Gitea's normal automatic light/dark behavior as the default. The five Nord themes remain available in **Settings -> Appearance**.

To expose all discovered themes instead, leave `THEMES` empty:

```ini
[ui]
DEFAULT_THEME = gitea-auto
THEMES =
```

> [!NOTE]
> An explicit list is recommended, _not mandatory_ because it documents exactly which custom themes are intended to be available.

### 5. Restart Gitea

```bash
sudo systemctl restart gitea
sudo systemctl status gitea --no-pager
```

Review recent logs if the service does not start successfully:

```bash
sudo journalctl -u gitea -n 50 --no-pager
```

### 6. Select a theme

Log in to Gitea and open **Settings -> Appearance**. The selector should include:

```text
Nord Polar Night
Nord Snow Storm
Nord Frost
Nord Aurora
Nord Accessibility
```

Hard-refresh the browser after switching if an older stylesheet is cached.

## Troubleshooting

### A theme does not appear

Check all of the following on the remote server:

```bash
sudo find <CustomPath>/public/assets/css/ -maxdepth 1 -type f -name 'theme-*.css' -print
sudo grep -A10 '^\[ui\]' /etc/gitea/app.ini
sudo systemctl restart gitea
```

The filename must use this exact format:

```text
theme-<theme-name>.css
```

For example, use `theme-nord-polar-night.css`, not `nord-polar-night.css`, `nord_polar_night.css`, or `theme-nord-polar-night.theme.css`.

### The CSS URL returns 404

Use the exact stylesheet URL shown in the page source or browser Network panel. For a root-mounted Gitea instance it should look like:

```text
/assets/css/theme-nord-polar-night.css
```

Check the public URL:

```bash
curl -I https://git.domain.com/assets/css/theme-nord-polar-night.css
```

If Nginx returns the `404`, test Gitea directly on the remote server using the configured `HTTP_PORT`:

```bash
grep -E '^(HTTP_PORT|PROTOCOL|ROOT_URL)[[:space:]]*=' /etc/gitea/app.ini
curl -I http://127.0.0.1:3000/assets/css/theme-nord-polar-night.css
```

Replace `3000` with the configured port. A direct `200` with a public `404` indicates an Nginx proxy-location problem. A direct `404` indicates the file is not under the active `CustomPath` or the filename does not match.

### Check permissions

The service account must be able to traverse every parent directory and read each file:

```bash
sudo namei -l <CustomPath>/public/assets/css/theme-nord-polar-night.css
sudo chown git:git <CustomPath>/public/assets/css/theme-*.css
sudo chmod 0644 <CustomPath>/public/assets/css/theme-*.css
sudo -u git test -r <CustomPath>/public/assets/css/theme-nord-polar-night.css \
  && echo readable
```

### Parts of the interface keep the old colors

Confirm that the loaded CSS response contains the expected variables. For Polar Night, it should include:

```css
--color-body: #2E3440;
```

If the file loads but a newer Gitea release has introduced variables that are not covered by this theme pack, update the theme against that Gitea release's built-in theme contract.

## Adding another Nord theme

Install the new file under the same directory:

```bash
sudo install -o git -g git -m 0644 \
  theme-nord-new-theme.css \
  <CustomPath>/public/assets/css/
```

If `THEMES` is explicitly configured, add the name without the prefix or extension:

```ini
[ui]
DEFAULT_THEME = gitea-auto
THEMES = nord-polar-night,nord-snow-storm,nord-frost,nord-aurora,nord-accessibility,nord-new-theme
```

Restart Gitea after changing the file or configuration.

## Repository layout

```text
README.md
theme-nord-accessibility.css
theme-nord-aurora.css
theme-nord-frost.css
theme-nord-polar-night.css
theme-nord-snow-storm.css
```

Each CSS file is standalone and can be copied directly to a Gitea instance.

## Credits

Nord was created by Arctic Ice Studio. This repository maintains a small standalone Nord theme pack for Gitea.
