# My Sway setup: session restore, workspace history, a few small scripts

**Version 1.1, 2026-11-25.** Author contact: dominik@hudzik.info

Sharing the config and helper scripts I have built up on a laptop (Debian 13, Sway 1.10, one external monitor). Everything is plain bash or Python plus `swaymsg` and `jq`. Config and scripts are linked at the bottom. Take what is useful, tell me what you would do differently.

## What it is built around

Claude Code is my main tool and the whole setup is built around it. Each workspace is one project folder with Claude Code running inside Neovim (the claudecode.nvim plugin), in a foot terminal. In the shared scripts that launch line is plain `nvim`; swap in whatever you run. Browsers have fixed jobs:

- **qutebrowser, one per workspace.** Every HTML page Claude Code makes for me (reports, question pages, mind maps) opens with `qb-here`, which walks the process tree from the terminal to its window and opens the page in the qutebrowser of that same workspace, with its own profile folder. Pages never land on the workspace I happen to be looking at.
- **Firefox** shows a local low-code app that holds my CRM and my knowledge base.
- **Vivaldi** is there so Claude Code can use the internet when headless browsing and a scraping service cannot get the page.

Because of that, a session is a set of projects, each usually a terminal next to a browser, not a set of apps.

## The workflow

I keep one folder per workspace and want my layout back after a login. These do that:

1. **Mod+1 ... 0** (or the numpad) switches workspace. If the workspace is empty, `ws-open` starts the apps saved for it (terminal in the right folder, browser, editor), in the saved columns and widths.
2. **Mod+Ctrl+s** (`sway-save`) saves what is open now: folder, app kind, column and row, width in percent. It saves only the workspaces that have windows right now, so closing the ones you do not want before saving is how you pick a session. Ctrl+Alt+F4 saves and powers off.
3. **Login** runs `ws-open all` from `sway-startup`, so the saved session comes back.
4. **Mod+Ctrl+n** (`ws-new`) lists every workspace number you ever used, with the folder last used and the date (a `*` marks the open ones). Pick a number, confirm or change the folder, and it opens workspace "N: folder" with an editor in that folder. The history file is never pruned, so a workspace you used months ago is still on the list. **Mod+Shift+n** (`ws-rename`) renames the current workspace after the folder open in the focused window.
5. **Mod+Ctrl+z freezes a workspace** (`ws-freeze`): SIGSTOP to the process trees behind the windows on the focused workspace, SIGCONT on the next press. Frozen windows are dimmed. Terminals and processes that also own windows on other workspaces are skipped. RAM stays used but the CPU does not. Good for a heavy browser on a laptop with 8 GB. **Read the warning below before you use it.**
6. **`tile-rule` decides where a new window goes:** the first three windows on a workspace sit side by side, the fourth and later open below. It replaced `autotiling`, which kept splitting in directions I did not want. It also renames each workspace to "N: folder" after the folder of the session running in it (set `SESSION` at the top of the script to the process you run, `claude` by default).

Other things I use every day: Mod+x swaps the focused window with the next one, Mod+Shift+space floats a window at 70% x 70% centred (and back), Mod+e flips horizontal/vertical for the window you are in, Mod+Ctrl+minus picks a window from the scratchpad in fuzzel.

## Keys

Mod is the Super key. I override some Sway defaults: Mod+b (was splith) and Mod+v (was splitv) are launchers, Mod+e runs `sway-flip` instead of `layout toggle split`, Mod+Shift+space runs `sway-float` instead of `floating toggle`, Mod+q closes the focused window and Mod+Shift+q closes the whole workspace, Mod+Shift+c checks the config before it reloads.

**Windows**
- Mod+h j k l: focus. Mod+Shift+h j k l or arrows: move the window
- Mod+x: swap with the next tiled window
- Mod+f: fullscreen. Mod+Shift+space: float 70% x 70% centred, or back to tiled. Mod+space: focus floating/tiled
- Mod+q: close the focused window. Mod+Shift+q or Mod+Ctrl+q: close every window on this workspace, no confirmation
- Mod+r: resize mode (h j k l, Esc)
- Mod+minus / Mod+Shift+minus: show / send to scratchpad. Mod+Ctrl+minus: pick a scratchpad window in fuzzel (`sp-pick`)

**Layout**
- Mod+Ctrl+v: next window below. Mod+Shift+v: next window beside
- Mod+s stacked, Mod+w tabbed, Mod+e flip horizontal/vertical (`sway-flip`), Mod+a focus parent

**Workspaces**
- Mod+1 ... 0 or numpad: go to workspace (restores its saved apps if empty). Mod+Shift+1 ... 0 or numpad: move window there
- Mod+Tab: back to the previous one. Mod+Left / Mod+Right: previous / next. Mod+Up / Mod+Down: first / last existing
- Mod+Ctrl+n: new workspace with folder (`ws-new`). Mod+Shift+n: rename after the folder (`ws-rename`)
- Mod+Ctrl+s or Mod+Shift+g: save session (`sway-save`)
- Mod+Ctrl+z: freeze / wake the apps on this workspace (`ws-freeze`)

**Switching and launching**
- Alt+Tab: previous window, most recently used first (`sway-mru`)
- Alt+Shift+Tab or Mod+Shift+Tab: window list in fuzzel, workspace number first, exact search (`sway-windows`)
- Mod+b: all apps, last launched first. Mod+v: only the apps in a recent list you curate by hand (`sway-launcher`)
- Mod+Return: terminal (foot). Mod+n: file manager. Mod+Shift+w: Wi-Fi picker (nmtui in a small floating terminal)

**System**
- Ctrl+Alt+v: clipboard history (cliphist + fuzzel, text and images). Ctrl+Alt+Shift+v: delete an entry
- One clipboard: `clip-sync` copies every selection to the clipboard and back, so Ctrl+V, Shift+Insert and middle-click paste the same text
- Ctrl+Alt+l: lock. Print / Shift+Print: screenshot (whole screen / selection, saved and copied)
- Mod+Shift+c: check the config with `sway -C`, reload only if it is valid (`sway-reload`). Mod+Shift+e: log out (asks first)
- Ctrl+Alt+F4: save the session, then power off (`sway-poweroff`)
- Volume, brightness, mute and media keys as printed on the keyboard

## Scripts

| Script | What it does |
|---|---|
| `sway-save` | saves open windows (folder, kind, column, row, width %) to `workspaces.map`, and every folder ever used to `workspaces.history` |
| `ws-open` | starts the saved apps for workspace N (or `all`, or `l` to list), only if N is empty |
| `ws-new` | fuzzel list of every workspace number ever used, then opens it with an editor in a folder |
| `ws-rename` | renames the focused workspace to "N: folder" from the focused window |
| `sway-startup` | login: restores the saved session |
| `ws-freeze` | freezes / wakes the apps behind the focused workspace, terminals skipped |
| `tile-rule` | three windows side by side, the fourth and later below |
| `qb-here` | opens a URL in the qutebrowser of the workspace the calling terminal sits on |
| `sway-mru` | most-recently-used window switch (daemon follows focus events) |
| `sway-windows` | fuzzel window picker with workspace number first |
| `sp-pick` | fuzzel picker for scratchpad windows |
| `sway-launcher` | app launcher ordered by last start, `--recent` for a curated list |
| `sway-swap`, `sway-flip`, `sway-float` | swap two windows, flip split direction, toggle a centred 70% float |
| `sway-reload` | validates the config, reloads only if it is valid |
| `sway-poweroff` | saves the session, then powers off even if the save fails |
| `clip-sync` | keeps the primary selection and the clipboard the same |
| `sway-outputs` | external monitor present: laptop panel off; none: on (follows hotplug) |
| `ws-edge`, `bat-min`, `wallpaper-daily` | first/last workspace, battery minutes for waybar, one wallpaper per day from a folder |

## Known weak spots (an outside review found these, I have not fixed them all)

- **`ws-freeze` is a blunt tool.** SIGSTOP on a process tree makes network connections time out on the far side, breaks audio streams and calls, and a frozen window does not redraw (the script dims it so you can tell). They stay stopped until you press Mod+Ctrl+z on that workspace again. A cgroup freezer (`systemctl freeze` on a user scope) would be cleaner. My older version froze workspace 9 automatically whenever it was hidden; I turned that off because it caught things I did not mean to stop.
- **Mod+Shift+q and Mod+Ctrl+q close every window on the workspace with no confirmation.** I keep it on purpose. Rebind it if you do not want that.
- **No lock on idle.** Screens power off after 10 minutes, nothing locks. Add `swaylock` to the swayidle line (and `before-sleep`) if you need it.
- **The blank mode binds keycodes 9 to 255** so the first key only wakes the screen. It is 250 generated lines, Ctrl+key chords are not caught, and it drops you out of resize mode.
- **`sway-save` replaces the whole map.** Pressing it right after login, before the restore has finished, saves a smaller session than you had. `workspaces.map.prev` keeps one older copy.
- **Workspace names are hard-coded** in the config, `ws-open`, `sway-startup` and `ws-new`. Change them in all four.
- **No lock around `ws-open`:** pressing Mod+1 twice very fast can start the apps twice.
- **`cliphist` stores everything you copy in plain text** under `~/.cache/cliphist`, including copies from a password manager, and `clip-sync` now sends every text selection there too. Filter or clear it.
- **Folder names:** `ws-open`, `sway-save`, `ws-new` and `ws-rename` only accept folder names made of letters, digits and `. _ / + @ % : = ~ -`. Spaces and quotes are rejected on purpose, because the command line goes through sway's own `sh -c`.

## Gotchas I hit

- **Sway's environment has no `~/.local/bin` in PATH.** Use full paths in bindings, and for windows you start from scripts put `env PATH=$HOME/.local/bin:\$PATH` in front of the command.
- **A `swaymsg output ...` that changes nothing still fires an output event.** A subscribe daemon must act only when the state differs, or it loops. Mine looped for hours.
- **A reload re-runs `exec_always` and every `output * bg` line.** If a script sets the wallpaper once a day, a fixed `output * bg #808080 solid_color` line wipes the picture on each reload. The shared config leaves that line out. Daemons that must start once (`tile-rule`, `clip-sync`) use `exec`, not `exec_always`.
- **`swaymsg workspace N` for a missing workspace creates one named just "N"**, so named workspaces lose their names. Use `workspace number N`.
- **A new window opens on the focused workspace, not on the one of the program that started it.** Opening first and moving after makes it flash on the screen you are looking at. Work out the target workspace first.
- **Waking from `output * power off`:** I bind every keycode in a "blank" mode so the first key press only wakes the screen and is not typed into a window.
- **`pkill -f` matches its own command line.** Anchor the pattern or kill by pid.
- **Test config changes in a nested headless Sway** before they touch your live session, and let `sway -C` check the file before a reload.

## Files

Config and scripts are one folder: `config/sway-config`, `config/waybar-config`, `config/waybar-style.css`, `config/foot.ini`, `config/fuzzel.ini`, `config/mako-config`, and `bin/` with the scripts above. Theme is Catppuccin Mocha. Link: https://github.com/hudzik/AI-setup (folder `sway/`)

Dependencies (Debian package names where I know them): sway, swayidle, swaylock, swaynag (comes with sway), foot, fuzzel, waybar, mako-notifier, jq, python3, psmisc (pstree), procps (pgrep), upower, grim, slurp, wl-clipboard, cliphist, wlsunset, brightnessctl, playerctl, wireplumber (wpctl), network-manager (nmcli, nmtui), libnotify-bin (notify-send), yad, python3-i3ipc (for tile-rule), qutebrowser, thunar, firefox, neovim, policykit-1-gnome, gnome-keyring.

The files contain no personal data except the contact line above, but before you use them: delete the `output` lines for ports you do not have (`swaymsg -t get_outputs`), set your keyboard layout and put your own coordinates in the wlsunset line (the ones there are a placeholder). The polkit path is the Debian one. The scripts are tested on my machine only; read them before you bind them to keys.
