

# Ubuntu Dotfiles / larsoyd

These are my personal patches and configuration overrides for the GNOME Shell environment, specifically targeting **Dash to Panel** and **Desktop Icons NG (DING)**.

## Patched Extensions

### 1. Dash to Panel

**Installation Path:**
`~/.local/share/gnome-shell/extensions/dash-to-panel@jderose9.github.com`

**Patch Applied:** `metro.patch`

**Description:** This patch modifies the "Metro" style running indicators. By default, the indicators are rectangles that can awkwardly bump into each other. This patch transforms them into slightly inset, rounded capsules and centers them under the icon. It utilizes the existing dot size and color settings, changing only the geometry for a more pleasing aesthetic.

---

### 2. Desktop Icons NG (DING)

**Installation Path:**
`/usr/local/share/gnome-shell/extensions/ding@rastersoft.com`

> **Important Note:**
> Ubuntu’s GNOME session mode explicitly refuses to load user copies of system extensions (even if found in `~/.local`). To ensure edits take effect, patched Ubuntu extensions must be placed in `/usr/local/share/...`. GNOME prefers `/usr/local/share` over `/usr/share` on a standard Ubuntu setup, allowing these patches to persist without being overwritten by `apt`.

--

**Patch Applied:** `ding_unselect.patch`

**Description:** Fixes a persistent issue where double-clicking a desktop launcher leaves the icon selected. This patch schedules `unselectAll()` on idle, ensuring it runs after the click/release logic finishes.

--

# DING Desktop Icon Label Shadow Override on Ubuntu (What Worked)

## Summary

I wanted to make the Desktop Icons NG (DING) desktop label text shadow slightly more prominent using CSS.

The CSS syntax was not the problem. The real issue was **where Ubuntu was loading DING from** and how Ubuntu treats some system/session-managed extensions.

The fix was to patch the **active packaged DING stylesheet** and protect that patch from `apt` overwrites using `dpkg-divert`.

DING is a GNOME Shell extension that provides desktop icons, and current package layouts can include an `app/stylesheet.css` inside the extension directory. 

## What failed

### 1) GNOME Shell user stylesheet did not affect DING labels
Custom CSS in the GNOME Shell user stylesheet worked for panel-related styling (for example Dash to Panel), but it did not affect DING desktop icon labels because DING uses its own stylesheet path (`ding@rastersoft.com/.../app/stylesheet.css`) rather than the shell theme stylesheet. This is consistent with DING packaging layouts that include a dedicated `app/stylesheet.css`. 

### 2) `~/.local/share/gnome-shell/extensions/...` copy did not override Ubuntu’s active DING
On this Ubuntu setup/session mode, the user-copy approach did not take effect for the built-in DING extension. (This matched observed behavior from logs showing DING launching from `/usr/share/...` rather than the user copy.) Ubuntu/ GNOME discussions around mode extensions describe loading the system version for mode-managed extensions. 

### 3) `/usr/local/share/...` override also did not take effect here
Normally, XDG data directory search order prefers `/usr/local/share` before `/usr/share` (default `XDG_DATA_DIRS` order), so this is a reasonable idea in general. The XDG spec explicitly defines `/usr/local/share/:/usr/share/` as the default and says earlier directories take precedence. 

However, in this case Ubuntu still effectively used the packaged DING path in `/usr/share/...`, so the practical fix was to patch the packaged file directly in a package-safe way.

## Working solution

### `dpkg-divert` on the active DING stylesheet file

`dpkg-divert` exists specifically to override package-installed files and prevent `dpkg` from overwriting your local version during package operations. The Ubuntu manpage explicitly describes diversions as a way to force `dpkg` not to install a file in its original location and notes admins can use it to override files that are not conffiles. 

### The What

1. Cleaned up previous override attempts (`~/.local` and `/usr/local` copies).
2. Added a local diversion for:
   - `/usr/share/gnome-shell/extensions/ding@rastersoft.com/app/stylesheet.css`
3. Let `dpkg-divert` rename the original to `stylesheet.css.distrib`
4. Copied the diverted original back to the original path as the editable local working file
5. Edited only the `.file-label` and `.file-label-dark` CSS blocks
6. Logged out and back in
7. Verified visible label color/shadow change (proving the correct file was finally being loaded)

## The Why

- It targets the **exact file path Ubuntu is actually loading**
- It avoids guessing about extension search precedence
- It survives package upgrades better than direct edits to package-owned files because `dpkg-divert` records the override and preserves the local replacement behavior 

## References

- DING extension page (GNOME Extensions): confirms DING project identity and upstream link. 
- Arch package file list: shows DING packaged layout with `.../app/stylesheet.css` in current builds. 
- Ubuntu `dpkg-divert` manpage: diversion mechanism and admin override use case. 
- GNOME bug discussion on mode extensions loading system version: relevant to why user overrides can fail in Ubuntu session mode. 
- XDG Base Directory spec: default `/usr/local/share/:/usr/share/` ordering and precedence semantics. 

--

**Patch Applied:** `ding-fixed-step-grid.patch`

**Description:** Switches DING from "justified/stretchy" placement to a **fixed-step grid**.
*The Problem:* DING originally calculates row height using `Math.floor(height / maxRows)`. This forces the grid to fill the entire available height, distributing leftover pixels into the row steps. Because coordinates are calculated proportionally, rounding errors accumulate. On lower rows (5 or 6), this results in visual "drift" where the spacing feels inconsistent, especially with smaller fonts.
*The Fix:* This patch implements a fixed-step grid logic:
* Icons are placed using fixed math: `row * _elementHeight`.
* A "dead zone" is implicitly defined for remainder pixels at the bottom/right.
* Items snapped to the dead zone are clamped to the last valid row/column.
* This ensures icons adhere to the requested cell size (`Prefs` + spacing) rather than stretching to fill the monitor.
 
 

---

## Style Overrides (CSS)

### GNOME Shell CSS

**File:** `~/.config/gnome-shell/gnome-shell.css`

This file tweaks the visual balance of the Dash to Panel extension. Specifically, the clock typography and the behavior of the "Show Desktop" button so it acts more as a visual separator.

1. Clock: I find the default clock text too large and ugly. To use this correctly you need to install the Panel Date Format (by KEIII) extension so you can set a custom clock format. Then set the format to 2 lines (time & date) with 24-hour time with dconf: 

```bash
dconf write /org/gnome/shell/extensions/panel-date-format/format "'%H:%M%n%d/%m/%Y'"
```

This makes the clock appear on top of the date underneath it similar to Windows. Finally, the patch forces font size down to 0.85em which you can tweak, for me it looks better on my panel and isn't clipping. It also ensures the text lines are centered instead of aligned from the left. 

2. Show Desktop: I use the "Show Desktop" button at the end of my panel to clear the screen. By default, the border stretches from the very top to the very bottom, looking like a fugly wall. I added top and bottom margins to "cut" the line. This pushes the element away from the panel edges, transforming it into a floating separator which looks much cleaner.

```css
/* Dash to Panel clock: center the lines + slightly smaller */
.dashtopanelMainPanel .panel-button.clock-display .clock {
    text-align: center !important;
    font-size: 0.85em !important;
}

/* Target the Show Desktop button acting as a separator */
.showdesktop-button {
    /* "Cut" the line by pushing the element away from the panel edges */
    margin-top: 5px !important;
    margin-bottom: 5px !important;
}

```

### GTK 3.0 CSS

**File:** `~/.config/gtk-3.0/gtk.css`

These tweaks adjust the font size and shadows on the desktop launchers (DING). For me 9.5 is the sweet spot for names on the launchers, but it is heavily dependent on the monitor and icon size. My icons are set on Tiny. Tweak accordingly.

```css
/* Decrease font size for Desktop Icons NG */
window.desktopwindow .file-label,
window.desktopwindow .file-label:backdrop,
window.desktopwindow .file-label-dark,
window.desktopwindow .file-label-dark:backdrop {
    font-size: 9.5pt;
    /* x-offset y-offset blur-radius color */
    text-shadow: 1px 2px 3px black;
}

```

---
