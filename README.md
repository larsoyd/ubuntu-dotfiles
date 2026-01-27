

# Ubuntu Dotfiles / larsoyd

This repository contains my personal patches and configuration overrides for the GNOME Shell environment, specifically targeting **Dash to Panel** and **Desktop Icons NG (DING)**.

## Patched Extensions

### 1. Dash to Panel

**Installation Path:**
`~/.local/share/gnome-shell/extensions/dash-to-panel@jderose9.github.com`

**Patch Applied:** `metro.patch`

> **Description:** This patch modifies the "Metro" style running indicators. By default, the indicators are rectangles that can awkwardly bump into each other. This patch transforms them into slightly inset, rounded capsules and centers them under the icon. It utilizes the existing dot size and color settings, changing only the geometry for a more pleasing aesthetic.

---

### 2. Desktop Icons NG (DING)

**Installation Path:**
`/usr/local/share/gnome-shell/extensions/ding@rastersoft.com`

> **Important Note:**
> Ubuntu’s GNOME session mode explicitly refuses to load user copies of system extensions (even if found in `~/.local`). To ensure edits take effect, patched Ubuntu extensions must be placed in `/usr/local/share/...`. GNOME prefers `/usr/local/share` over `/usr/share` on a standard Ubuntu setup, allowing these patches to persist without being overwritten by `apt`.

**Patch Applied:** `ding_unselect.patch`

> **Description:** Fixes a persistent issue where double-clicking a desktop launcher leaves the icon selected. This patch schedules `unselectAll()` on idle, ensuring it runs after the click/release logic finishes.

**Patch Applied:** `ding-fixed-step-grid.patch`

> **Description:** Switches DING from "justified/stretchy" placement to a **fixed-step grid**.
> *The Problem:* DING originally calculates row height using `Math.floor(height / maxRows)`. This forces the grid to fill the entire available height, distributing leftover pixels into the row steps. Because coordinates are calculated proportionally, rounding errors accumulate. On lower rows (5 or 6), this results in visual "drift" where the spacing feels inconsistent, especially with smaller fonts.
> *The Fix:* This patch implements a fixed-step grid logic:
> * Icons are placed using fixed math: `row * _elementHeight`.
> * A "dead zone" is implicitly defined for remainder pixels at the bottom/right.
> * Items snapped to the dead zone are clamped to the last valid row/column.
> * This ensures icons adhere to the requested cell size (`Prefs` + spacing) rather than stretching to fill the monitor.
> 
> 

---

## Style Overrides (CSS)

### GNOME Shell CSS

**File:** `~/.config/gnome-shell/gnome-shell.css`

These tweaks adjust the Dash to Panel clock and the "Show Desktop" button separator.

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

    /* Optional: Ensure the line thickness setting is respected or forced */
    /* border-left-width: 1px !important; */
}

```

### GTK 3.0 CSS

**File:** `~/.config/gtk-3.0/gtk.css`

These tweaks adjust the font size and readability of icons on the desktop (DING).

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
