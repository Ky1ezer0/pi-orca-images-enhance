# pi-orca-images-enhance

Inline images for the [Pi coding agent](https://pi.dev) inside [Orca](https://github.com/stablyai/orca), with temporary image previews and a command to open macOS Preview.

This project is a fork of **[JlMoreno00/pi-orca-images](https://github.com/JlMoreno00/pi-orca-images)**. The original extension provides inline image fixes, tab-switch recovery, and pasted image labels. This fork adds image links and a macOS Preview command for a larger view and zoom, without writing preview files into your project.

## What this fork adds

### Open image links

Each rendered image gets an **Open image** link beneath it. **Cmd-click the link on macOS** to open the image in an Orca image tab, where you can use Orca’s zoom controls.

This keeps the preview inside Orca. Clicking the image itself and hovering to zoom are not supported.

### Open an image with a command

Run:

```text
/orca-images open
```

On macOS, this opens the **last image rendered by Pi** in **Preview**, using `open -a Preview`. Cmd-clicking the file link is handled by the terminal; in our Orca test it opened an Orca image tab instead.

### Temporary preview files

Previews are saved in a private `pi-orca-images-*` directory under the operating system’s temporary directory. **No new preview files are written to your project.**

- Identical images reuse a file within each loaded extension instance.
- Files remain available while Pi is running, including across `/reload`.
- Temporary directories are removed on normal Pi process exit. Links expire when their files are removed.
- Forced termination or a crash can leave temporary files behind; cleanup is not a hard disk quota. A long-running session can still accumulate previews.
- Older `.pi/image-previews/` folders are not automatically deleted. You can remove those old preview copies when you no longer need them.
- The preview uses the bytes available to Pi; it cannot restore resolution already removed by a tool.

## Try this fork

From a local checkout, run:

```bash
npm install
pi --no-extensions -e ./src/index.ts
```

This loads the fork explicitly and disables other extensions for the test session, avoiding a second copy of the original extension.

Inside Pi, run:

```text
/orca-images test
```

A color gradient and an **Open image** link should appear. Cmd-click the link to open the gradient in Orca, or run `/orca-images open`.

To use this local temporary-preview version in normal Pi sessions, remove or disable any previously installed copy, then install this checkout:

```bash
pi install /absolute/path/to/pi-orca-images-enhance
```

To install this fork from GitHub instead:

```bash
pi install git:github.com/Ky1ezer0/pi-orca-images-enhance
```

If the original package is installed, remove or disable it first so both copies do not load together. Restart Pi after installation.

> `pi install npm:pi-orca-images` installs the original upstream package, not the enhancements in this fork.

## Features inherited from upstream

The following features come from [JlMoreno00/pi-orca-images](https://github.com/JlMoreno00/pi-orca-images).

### Inline image fixes

Orca renders terminal images through xterm.js’s image addon. The original extension addresses three rendering problems:

| Problem | Fix |
|---|---|
| Images do not appear | Sets `PI_IMAGE_PROTOCOL` from Orca’s `ORCA_IMAGE_PROTOCOL` hint at startup. |
| Only the first row appears | Enables Pi’s “draw images last” render path while rendering. |
| Images disappear after switching tabs | Re-sends visible images several times over five seconds after focus returns. |

The tab recovery works around [stablyai/orca#32714](https://github.com/stablyai/orca/issues/32714).

![Upstream comparison showing inline images before and after the rendering fix](docs/before-after.png)

### Pasted image labels

Paste or drop an image file path into Pi’s prompt to display it as a bold, colored `[Image 1]` label. Labels are numbered per message, move and delete as one unit, and expand back to their original paths when you submit the prompt.

Supported inputs include plain paths, `~/` paths, quoted paths, backslash-escaped paths, and `file://` URLs. The paste must consist only of existing image file paths; mixed text is left unchanged.

Recognized extensions include PNG, JPEG, GIF, WebP, BMP, TIFF, HEIC, HEIF, and AVIF. Image labels work outside Orca too, including with Pi’s clipboard paste.

![Upstream demonstration of pasted image labels](docs/image-labels.gif)

## Commands

| Command | Effect | Source |
|---|---|---|
| `/orca-images` or `/orca-images status` | Shows image detection, the active protocol, repaint fix status, and image-label status. | Upstream |
| `/orca-images test` | Displays a test gradient without sending it to the model. | Upstream |
| `/orca-images redraw` | Re-sends visible images immediately. | Upstream |
| `/orca-images open` | Opens the last rendered image in macOS Preview. | **This fork** |

## Configuration

Orca rendering fixes and preview links activate through automatic Orca detection. Pasted image labels can work in any terminal.

| Setting | Effect |
|---|---|
| `PI_ORCA_IMAGES=0` | Disables the Orca image fixes and this fork’s preview links and open command. |
| `PI_IMAGE_LABELS=0` | Disables pasted image labels. |
| `PI_IMAGE_PROTOCOL=…` | Preserves your explicit protocol choice. This currently also prevents the fork’s preview feature from activating. |
| `terminal.images` in Pi settings | Applies Pi’s protocol setting on top of the extension’s choice. |
| `terminal.showImages: false` | Keeps inline images hidden. |

## Requirements and limitations

- **Orca:** Inline images enabled under **Settings → Terminal → Inline Images**. Orca v1.4.229 or later exports the image protocol hint used for detection.
- **Pi:** Upstream targets Pi 1.4. This fork’s link and open-command workflow was also tested with Pi 1.5.2 on macOS.
- **Fullscreen mode:** Required for the repaint fix and tab recovery. In regular mode, protocol detection still works.
- **Temporary storage:** The OS temporary directory must be writable. The open command requires macOS with Preview installed; terminal link behavior depends on the terminal.
- **Terminal multiplexers:** Automatic activation is skipped inside tmux and screen.
- **Hover and direct clicks:** Use the **Open image** link; neither hover-to-zoom nor clicking the image itself is implemented.
- **Internal Pi APIs:** Image labels, repaint fixes, and preview links rely on internal Pi methods or fields. Future Pi changes may require compatibility updates. If preview data cannot be read or saved, links are omitted.

## Development

```bash
npm install
npm run check
pi --no-extensions -e ./src/index.ts
```

`npm run check` runs TypeScript checking and the test suite. Preview tests cover link generation, saving image bytes, opening the last rendered image, error handling, and restoring the renderer.

The original documentation images in `docs/` are upstream captures of Pi 1.4 in Orca 1.4.229 on macOS. To regenerate the upstream banner:

```bash
python3 scripts/banner.py
```

The banner script requires Pillow, Inter, and JetBrains Mono.

## Credits and license

Original extension by **[JlMoreno00](https://github.com/JlMoreno00)**: [pi-orca-images](https://github.com/JlMoreno00/pi-orca-images).

Fork enhancements in **[Ky1ezer0/pi-orca-images-enhance](https://github.com/Ky1ezer0/pi-orca-images-enhance)**: Open image links, the `/orca-images open` command, and temporary preview files.

Licensed under [MIT](LICENSE). The original license and attribution are preserved.
