Dash to Panel:
~/.local/share/gnome-shell/extensions/dash-to-panel@jderose9.github.com

Patch Applied: metro.patch

Description: This patch turns the METRO indicators under the icons from rectangles that bump into each other into slightly inset rounded capsules. It also centers them under the icon. It uses existing dot size + color settings, and only changes geometry.

---

Ubuntu’s GNOME session mode is explicitly refusing to load the user copies of their extensions, even though it finds it. So if we put Ubuntu extensions in ~/.local/... then edits will never take effect. - Lesson: When editing Ubuntu extensions, put your patched copy in /usr/local/share/... (system-level, but not managed by apt), because GNOME will prefer /usr/local/share over /usr/share on a normal Ubuntu setup.

DING:
/usr/local/share/gnome-shell/extensions/ding@rastersoft.com

Patch Applied: ding_unselect.patch

Description: DING Unselect addresses a major pet peeve of mine, that being that when you double-click desktop launchers on the desktop it stays selected. For some reason this is quite common on Linux, I think XFCE is the only desktop that doesn't do this. This patch fixes the problem by scheduling unselectAll() on idle and makes it run after the click/release logic finishes.

Patch Applied: ding-fixed-step-grid.patch

Description: One of the problems of DING is that the desktop launchers look off. I looked into why this is and figured it out. In DesktopGrid.createGrids() DING first decides how many rows fit by doing a floor division against the “ideal” icon cell size (desired height plus spacing). Then it forces the grid to fill the entire available height by redefining the real cell height as Math.floor(this._height / this._maxRows). That effectively distributes the leftover pixels into the per-row step. The worst part for the visual “drift” is when it converts a (column,row) into actual pixel coordinates, as it does it proportionally: localY = Math.floor(this._height * row / this._maxRows) (and same idea for X). Because each row’s position is separately floored, the rounding error accumulates unevenly as you go down. If you have a smaller font like I do it makes the extra empty vertical slack inside each cell more noticeable, so by the time you’re at row 5 or 6 it “feels” like the spacing has changed even though the icon artwork did not.

I decided to patch this by switching DING from “justified/stretchy” placement to a fixed-step grid. I achieve this without changing the overlap-avoidance logic by defining a “dead zone” implicitly. Any remainder pixels at the bottom/right no longer get distributed into row heights, and anything dropped there snaps to the last row/column. This effectively stops the “stretching” `_elementWidth/_elementHeight` of icons to fill the whole monitor and instead locks them to the fixed requested cell size (`Prefs.get_desired_*() + 4*elementSpacing`). To do this it places icons using fixed-step math (row * _elementHeight, col * _elementWidth) instead of proportional Math.floor(this._height * row / this._maxRows). Finally it clamps drag snap coordinates in the “dead zone” (bottom/right remainder) so drag highlight and drop never end up one cell beyond the last valid row/column.

---


lars@lars-HP-Notebook ~> nano ~/.config/gnome-shell/gnome-shell.css
lars@lars-HP-Notebook ~> cat ~/.config/gnome-shell/gnome-shell.css
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





lars@lars-HP-Notebook ~>

---

cat ~/.config/gtk-3.0/gtk.css
/* Decrease font size for Desktop Icons NG */
window.desktopwindow .file-label,
window.desktopwindow .file-label:backdrop,
window.desktopwindow .file-label-dark,
window.desktopwindow .file-label-dark:backdrop {
font-size: 9.5pt;
/* x-offset y-offset blur-radius color */
text-shadow: 1px 2px 3px black;
}
