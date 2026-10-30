# My Sway setup: session restore, workspace history, a few small scripts

Sharing the config and helper scripts I have built up on a laptop (Debian 13, Sway 1.10, one external monitor). Everything is plain bash or Python plus `swaymsg` and `jq`. Config and scripts are linked at the bottom. Take what is useful, tell me what you would do differently.

## What it is built around

Claude Code is my main tool and the whole setup is built around it. Each workspace is one project folder with Claude Code running inside Neovim (the claudecode.nvim plugin), in a foot terminal. In the shared scripts that launch line is plain `nvim`; swap in whatever you run. Two browsers have fixed jobs:

- **Firefox** shows a local low-code app that holds my CRM and my knowledge base.
- **Vivaldi** is there so Claude Code can use the internet when headless browsing and a scraping service cannot get the page.

Because of that, a session is a set of projects, each usually a terminal next to a browser, not a set of apps.

## The workflow

I keep one folder per workspace and want my layout back after a login. Five things do that:

1. **Mod+1 ... 0** switches workspace. If the workspace is empty, `ws-open` starts the apps saved for it (terminal in the right folder, browser, editor), in the saved columns and widths.
2. **Mod+Shift+s** (`sway-save`) saves what is open now: folder, app kind, column and row, width in percent. It saves only the workspaces that have windows right now, so closing the ones you do not want before saving is how you pick a session.
3. **Login** runs `ws-open all` from `sway-startup`, so the saved session comes back.
4. **Mod+Shift+n** (`ws-new`) lists every workspace number you ever used, with the folder last used and the date (a `*` marks the open ones). Pick a number, confirm or change the folder, and it opens workspace "N: folder" with an editor in that folder. The history file is never pruned, so a workspace you used months ago is still on the list.
5. **Workspace 9 is a freezer.** `sway-freeze` sends SIGSTOP to every process behind windows on workspace 9 while it is hidden and SIGCONT when you look at it. RAM stays used but the CPU does not. Good for a heavy browser on a laptop with 8 GB. **Read the warning below before you use it.**

Other things I use every day: Mod+x swaps the focused window with the next one, Mod+Shift+space floats a window at 70% x 70% centred (and back), Mod+e flips horizontal/vertical for the window you are in, and `autotiling` alternates the split direction.

## Keys

Mod is the Super key. I override four Sway defaults: Mod+b (was splith) and Mod+v (was splitv) are launchers, Mod+e runs `sway-flip` instead of `layout toggle split`, and Mod+Shift+space runs `sway-float` instead of `floating toggle`. Mod+Shift+n and Mod+Shift+s are free in the stock config.

**Windows**
- Mod+h j k l: focus. Mod+Shift+h j k l or arrows: move the window
- Mod+x: swap with the next tiled window
- Mod+f: fullscreen. Mod+Shift+space: float 70% x 70% centred, or back to tiled. Mod+space: focus floating/tiled
- Mod+Shift+q: close. Mod+r: resize mode (h j k l, Esc)
- Mod+minus / Mod+Shift+minus: show / send to scratchpad

**Layout**
- Mod+Ctrl+v: next window below. Mod+Shift+v: next window beside
- Mod+s stacked, Mod+w tabbed, Mod+e flip horizontal/vertical (`sway-flip`), Mod+a focus parent

**Workspaces**
- Mod+1 ... 0: go to workspace (restores its saved apps if empty). Mod+Shift+1 ... 0: move window there
- Mod+Tab: back to the previous one. Mod+Left / Mod+Right: previous / next. Mod+Up / Mod+Down: first / last existing
- Mod+Shift+n: new workspace with folder (`ws-new`). Mod+Shift+s: save session (`sway-save`)
- Ctrl+Shift+q: close every window on all other workspaces, no confirmation (`sway-close-others`)

**Switching and launching**
- Alt+Tab: previous window, most recently used first (`sway-mru`)
- Alt+Shift+Tab or Mod+Shift+Tab: window list in fuzzel, workspace number first, exact search (`sway-windows`)
- Mod+b: all apps, last launched first. Mod+v: only the apps in a recent list you curate by hand (`sway-launcher`)
- Mod+Return: terminal (foot). Mod+Shift+f: Firefox. Mod+n: file manager. Mod+Shift+w: Wi-Fi picker (nmtui in a small floating terminal)

**System**
- Ctrl+Alt+v: clipboard history (cliphist + fuzzel, text and images). Ctrl+Alt+Shift+v: delete an entry
- Ctrl+Alt+l: lock. Print / Shift+Print: screenshot (whole screen / selection, saved and copied)
- Mod+Shift+c: reload. Mod+Shift+e: log out (asks first)
- Volume, brightness, mute and media keys as printed on the keyboard

## Scripts

| Script | What it does |
|---|---|
| `sway-save` | saves open windows (folder, kind, column, row, width %) to `workspaces.map`, and every folder ever used to `workspaces.history` |
| `ws-open` | starts the saved apps for workspace N (or `all`, or `l` to list), only if N is empty |
| `ws-new` | fuzzel list of every workspace number ever used, then opens it with an editor in a folder |
| `sway-startup` | login: restores the saved session |
| `sway-freeze` | freezes the processes behind workspace 9 while it is hidden |
| `sway-mru` | most-recently-used window switch (daemon follows focus events) |
| `sway-windows` | fuzzel window picker with workspace number first |
| `sway-launcher` | app launcher ordered by last start, `--recent` for a curated list |
| `sway-swap`, `sway-flip`, `sway-float` | swap two windows, flip split direction, toggle a centred 70% float |
| `sway-close-others` | close all windows except on the current workspace |
| `sway-outputs` | external monitor present: laptop panel off; none: on (follows hotplug) |
| `ws-edge`, `bat-min`, `wallpaper-daily` | first/last workspace, battery minutes for waybar, one wallpaper per day from a folder |

## Known weak spots (an outside review found these, I have not fixed them all)

- **`sway-freeze` is a blunt tool.** SIGSTOP on a whole process tree makes network and ssh connections time out on the far side, breaks audio streams, and stops every job started from a terminal there (builds, servers). A killed daemon leaves processes stopped until it runs again. Do not put a call, a stream or a terminal with a running job on workspace 9. A cgroup freezer (`systemctl freeze` on a user scope) would be cleaner.
- **Ctrl+Shift+q kills windows with no confirmation** and it is also Quit in Firefox and Chromium, which the compositor now takes first. I keep it on purpose. Rebind it if you do not want that.
- **No lock on idle.** Screens power off after 10 minutes, nothing locks. Add `swaylock` to the swayidle line (and `before-sleep`) if you need it.
- **The blank mode binds keycodes 9 to 255** so the first key only wakes the screen. It is 250 generated lines, Ctrl+key chords are not caught, and it drops you out of resize mode.
- **`sway-save` replaces the whole map.** Pressing it right after login, before the restore has finished, saves a smaller session than you had. `workspaces.map.prev` keeps one older copy.
- **Workspace names are hard-coded** in the config, `ws-open`, `sway-startup` and `ws-new`. Change them in all four.
- **No lock around `ws-open`:** pressing Mod+1 twice very fast can start the apps twice.
- **`cliphist` stores everything you copy in plain text** under `~/.cache/cliphist`, including copies from a password manager. Filter or clear it.
- **Folder names:** `ws-open`, `sway-save` and `ws-new` only accept folder names made of letters, digits and `. _ / + @ % : = ~ -`. Spaces and quotes are rejected on purpose, because the command line goes through sway's own `sh -c`.

## Gotchas I hit

- **Sway's environment has no `~/.local/bin` in PATH.** Use full paths in bindings, and for windows you start from scripts put `env PATH=$HOME/.local/bin:\$PATH` in front of the command.
- **A `swaymsg output ...` that changes nothing still fires an output event.** A subscribe daemon must act only when the state differs, or it loops. Mine looped for hours.
- **A reload re-runs `exec_always` and every `output * bg` line.** If a script sets the wallpaper once a day, a fixed `output * bg #808080 solid_color` line wipes the picture on each reload. The shared config leaves that line out.
- **`swaymsg workspace N` for a missing workspace creates one named just "N"**, so named workspaces lose their names. Use `workspace number N`.
- **Waking from `output * power off`:** I bind every keycode in a "blank" mode so the first key press only wakes the screen and is not typed into a window.
- **`pkill -f` matches its own command line.** Anchor the pattern or kill by pid.

## Files

Config and scripts are one folder: `config/sway-config`, `config/waybar-config`, `config/waybar-style.css`, `config/foot.ini`, `config/fuzzel.ini`, `config/mako-config`, and `bin/` with the scripts above. Theme is Catppuccin Mocha. Link: https://github.com/hudzik/AI-setup (folder `sway/`)

Dependencies (Debian package names where I know them): sway, swayidle, swaylock, swaynag (comes with sway), foot, fuzzel, waybar, mako-notifier, jq, python3, psmisc (pstree), procps (pgrep), upower, grim, slurp, wl-clipboard, cliphist, wlsunset, brightnessctl, playerctl, wireplumber (wpctl), network-manager (nmcli, nmtui), libnotify-bin (notify-send), yad, autotiling (pip or a package), thunar, firefox, neovim, policykit-1-gnome, gnome-keyring, network-manager-gnome (nm-applet), blueman.

The config has no personal data in it, but before you use it: delete the `output` lines for ports you do not have (`swaymsg -t get_outputs`), set your keyboard layout and put your own coordinates in the wlsunset line (the ones there are a placeholder). The polkit path is the Debian one. The scripts are tested on my machine only; read them before you bind them to keys.
