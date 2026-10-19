# How it works

## One app, one data folder per profile

Claude Desktop is an Electron app. Electron allows one running instance per **data folder** and keeps
the sign-in inside that folder. Starting the official app with `--user-data-dir=<folder>` therefore
gives a second, fully separate instance with its own login. That is all a profile is.

| System  | Profile data                                  | Launcher                                                  |
|---------|-----------------------------------------------|-----------------------------------------------------------|
| Windows | `%LOCALAPPDATA%\Claude-<name>`                | Start menu shortcut `Claude <name>`                       |
| macOS   | `~/Library/Application Support/Claude-<name>` | `~/Applications/Claude <name>.app`                        |
| Linux   | `~/.config/Claude-<name>`                     | `~/.local/share/applications/multi-claude-<name in lowercase>.desktop` |

Nothing inside Claude changes, so app updates apply to every profile.

On Windows, newer Claude versions move their data out of the app package to
`%LOCALAPPDATA%\Claude-Data` and from then on use that folder whatever `--user-data-dir` says, so a
profile would just open the regular Claude. Claude skips that move when `LOCALAPPDATA` is written as a
`\\?\` path, which names the same folder, so `open` starts profiles that way. As a result, a
profile's Claude and everything it starts (Claude Code sessions, terminals, MCP servers) see
`LOCALAPPDATA=\\?\C:\Users\…\AppData\Local`. Most programs accept it; PowerShell's `Join-Path` does
not. While a profile runs, Claude also removes Explorer's **Open with > Claude** entry; the regular
Claude adds it back the next time it starts.

## Colored icons

`add` takes the Claude icon already installed on the computer and recolors it, so no artwork ships
with this project.

- **Windows and Linux** use the same algorithm: every colored pixel takes the target hue, saturation
  and brightness are scaled so the logo's brand color lands exactly on the target, and the white
  glyph stays white. Windows does it in C# (`System.Drawing`, compiled in memory) and writes a
  multi-size `.ico`; Linux does it with a small standard-library Python PNG reader and writer.
- **macOS** gets close with built-in Core Image filters (hue, saturation, gamma), run through
  JavaScript for Automation, then builds the `.icns` with `sips` and `iconutil`.

## Its own taskbar button (Windows)

Windows groups taskbar buttons by **AppUserModelID**, and every Claude window declares the same one,
so a second profile would otherwise stack onto the regular Claude button. A profile's Start menu
shortcut carries the ID `MultiClaude.<name>` and runs `multi-claude open <name>`, which:

1. starts Claude with the profile's data folder (if the profile is already open, Claude focuses it);
2. keeps running, hidden, and gives the profile's windows the same ID and relaunch properties, so
   the window and its pinned button become one colored button;
3. exits when the profile closes. Each profile has at most one watcher.

If tagging ever fails, the window simply appears under the regular Claude button.

## Sign-in

A browser sign-in returns through a `claude://` link, and the system gives that link to whichever
Claude is open, usually the regular one. **Sign in with a code** avoids the link entirely, so it works
with every Claude open. With single sign-on, quit every other Claude first, including other profiles
(on Windows, also from the tray). After that, each profile keeps its own login.

## What profiles share

Each profile has its own account, conversations, list of Claude Code sessions, scheduled tasks, MCP
configuration and app settings. They share what exists once per user: the `~/.claude` folder
(Claude Code settings, skills, memory and session transcripts), global keyboard shortcuts, the
browser's pairing with Claude in Chrome, and on Windows the packaged app's startup entry. The README
lists how to keep your regular Claude in charge of these.

Browsers find Claude through one native-messaging entry per user
(`com.anthropic.claude_browser_extension`): a registry key on Windows, a JSON manifest in each
browser's `NativeMessagingHosts` folder on macOS and Linux. Each Claude instance writes its own
manifest path there; on Windows, the packaged app registers it inside its package's redirected
AppData folder. Profile launchers run `multi-claude open`, which starts a small watcher for as long as
the profile is open; whenever a profile has taken the entry, the watcher points it back to the
regular Claude's own manifest, in Chrome, Edge, Brave and Chromium. It never touches the entry if the
regular Claude has not paired.

## Installation

The install commands download a versioned release archive. The release workflow writes that version
and the archive's SHA-256 into the installer itself, so the installer only accepts the exact archive
it was published with, and the release carries a build provenance attestation. Everything installs
in your user folder (on Windows the command is also added to your user PATH; on macOS and Linux it
is linked into `~/.local/bin`); nothing needs admin rights.

Plain scripts were chosen over a compiled installer: an unsigned executable triggers SmartScreen and
antivirus heuristics, and every new build starts with no reputation. Scripts can be read before they
run.
