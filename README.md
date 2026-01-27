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
