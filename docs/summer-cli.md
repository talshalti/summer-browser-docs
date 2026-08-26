# Summer CLI

Summer CLI is a permanent, bundled Summer App at `sap://cli/`. It
provides a real local pseudo-terminal (PTY), not a command runner: interactive
shells and terminal programs receive a terminal device, resize events, control
characters, ANSI output, and their usual line-editing behavior.

PTY sessions advertise `TERM=xterm-256color` and `COLORTERM=truecolor` on every
platform. The renderer supplies readable normal and bright ANSI palettes for
both light and dark themes; programs that honor `NO_COLOR` remain free to
disable their own colored output.

xterm's DOM renderer generates scoped runtime style rules for ANSI colors and
terminal dimensions, so this exact built-in page permits inline styles. Its CSP
continues to allow scripts only from the packaged app and blocks every image,
font, connection, object, form, base URL, and framing source. Terminal output is
written through xterm's parser and is never interpreted as HTML or CSS.

Summer CLI supports Windows, Linux, and the notarized direct-download
macOS edition. Summer discovers fixed, locally installed profiles:

- Windows PowerShell, Command Prompt, PowerShell 7, Git Bash, WSL, Python, and
  Node.js on Windows;
- the user's absolute `$SHELL`, then installed Bash, Z shell, Fish, POSIX shell,
  Python, and Node.js on Linux and macOS.

When an `ipython` or `ipython3` launcher is installed, Summer exposes it as a
first-class **IPython** REPL profile on every desktop platform. Windows discovery
also checks the selected Python installation's directory and adjacent `Scripts`
directory, covering standard Python and virtual-environment layouts without
starting Python during discovery. Summer does not install IPython or show a
profile that would fail solely because its module is absent.

The renderer selects an opaque profile identifier; it cannot supply an
executable path or launch arguments. Users can run programs available from an
opened shell with the operating-system permissions of Summer Browser itself.
Summer CLI never elevates privileges or requests an administrator prompt.

Summer has two macOS distributions. The Developer ID-signed, Apple-notarized
DMG distributed through Summer's website is not App Sandbox constrained and
provides the full host terminal. The Mac App Store edition retains Apple's App
Sandbox and does not start terminal processes; its CLI page directs users
to the notarized download. This is a release-channel boundary enforced again in
the main process through Electron's `process.mas` marker, not a renderer-only
notice. Do not add temporary sandbox exceptions or broaden MAS entitlements to
simulate host access.

## Trust boundary

The app package serves only static HTML, CSS, and JavaScript. It does not own a
process API and does not expose shell access through the public Summer App API,
manifest permissions, fetch actions, `window.summi`, or generic IPC.

The privileged flow is:

```text
exact top-level sap://cli/ document
    -> terminal-only preload wrapper
    -> authenticated browser IPC
    -> browser-owned TerminalService
    -> node-pty / Windows ConPTY or Unix PTY
```

The preload exposes `window.summerTerminal` only when the top-level document is
the exact canonical `sap://cli/` URL. The main process remains authoritative
and revalidates every request. Authorization requires all of the following:

- the sender is its attached top-level main frame;
- the frame URL and current WebContents URL are exactly `sap://cli/`, with
  no credentials, port, path variation, query, or fragment;
- the WebContents is the live contents of a current Summer tab;
- the contents use the regular user session, never the private session;
- the installed `cli` record is enabled and resolves by real path to the
  exact bundled directory and exact expected entry file;
- the reserved `cli` package ID cannot be replaced by an installed app.

Renderer checks are defense in depth. Websites, subframes, copied app packages,
disabled apps, stale documents, and private windows are rejected in the main
process. The page receives opaque session IDs only, never PIDs, native handles,
Electron objects, filesystem APIs, or raw IPC.

The existing Summer App module loader imports app server modules into the main
process and is not a security boundary for untrusted server code. The terminal
does not broaden that surface, but isolating all third-party Summer App server
modules would be a separate browser-wide security project.

## Sessions and process policy

A shell starts only after a visible user action. Page load, tab restoration, and
app activation do not spawn a process. The browser chooses the executable and
fixed argument vector for a discovered profile and always spawns without a
command-string shell layer. Initial working directories must be absolute,
existing directories; otherwise Summer uses the user's home directory.

Each PTY belongs to the requesting main-frame document and WebContents. The
service closes it idempotently after an explicit close, PTY exit, main-frame
navigation or reload, renderer crash, tab/WebContents destruction, app
deactivation, or browser shutdown. No detached background terminal sessions are
supported. As with other terminals, a program that deliberately detaches into a
separate operating-system job may outlive its parent shell.

Terminal input and output are never logged or persisted by Summer. Shells may
maintain their own ordinary history files according to their native settings.
The UI does not persist transcripts, session history, or terminal scrollback.
The only command persistence owned by the CLI UI is the explicit
**Favorite commands** list: when a user chooses to save a named command, that
entry is stored in the CLI app's local browser-profile storage until the
user edits or removes it. Users should not save passwords, tokens, or other
secrets as favorites.

## Resource limits and streaming

All IPC values are validated at runtime. The service bounds sessions per tab and
globally, terminal dimensions, input chunks, title and metadata sizes, and queued
output. PTY output is an ordered byte-preserving stream: the browser does not
parse ANSI escape sequences and the renderer gives chunks directly to xterm.

Output events carry monotonically increasing sequence numbers. The renderer
acknowledges data only after xterm accepts it. The service pauses the PTY above a
high-water mark and resumes below a low-water mark. If the hard queue ceiling is
exceeded, the service reports an explicit overflow and terminates the session
instead of silently dropping or reordering output.

## Renderer behavior

xterm renders terminal output without converting it to HTML. The app uses only
packaged scripts, styles, and fonts under a strict Content Security Policy with
`frame-ancestors 'none'` and no network connections. OSC clipboard operations
and automatic output-link opening are not enabled.

The UI provides multiple sessions, a shell/profile picker, searchable scrollback,
clear/restart/close actions, responsive layout, visible running and exit status,
and keyboard-accessible controls. Terminal text remains left-to-right even when
the surrounding interface is right-to-left. Multi-line paste requires explicit
confirmation, while single-line paste behaves normally. `Ctrl+C` remains terminal
input; copy and paste use the conventional shifted shortcuts.

Command autocomplete is profile-aware and uses three ordered layers:

1. session history and small REPL-specific suggestions in the renderer;
2. browser-owned structured providers for safe local context;
3. the shell's native Tab completion when Summer has no applicable result.

The browser-owned catalog covers more than 100 common developer, cloud,
database, infrastructure, mobile, AI, and blockchain commands. Its compact
command grammar supplies command names, subcommands, nested subcommands, and
options without importing or executing each tool. Summer filters first-token
suggestions to executables actually present on the current host, so catalog
coverage does not advertise unavailable commands.

Local providers add candidates that a static grammar cannot know. They include
folders and files; environment-variable names; npm scripts, workspaces,
dependencies, and project executables; Git branches, tags, and remotes; test and
configuration files; Vite modes; Make, Just, and Taskfile targets; Docker Compose
services; Kubernetes contexts and configured namespaces; AWS profile names;
Cargo features; local Python modules; and systemd units. For example, `cd ` opens
a bounded folder picker, `npm run ` shows package scripts, and `docker compose up
` shows services from the local Compose file.

The default discovery mode is local, bounded, and read-only. It parses known
files and enumerates limited directories; it does not invoke a CLI, evaluate
project code, contact a network service, return environment-variable values, or
silently execute a selected command.

An unchecked-by-default **CLI autocomplete** option enables richer completion
from supported installed tools. The preference is remembered locally for the
CLI app and consent is sent with every completion request. When enabled,
Summer may pass the current command line to an allowlisted executable's
documented completion interface:
Cobra's contextual protocol, `dotnet complete`, `winget complete`, or Cargo's
installed-command listing. These calls can load the tool's local configuration
or plugins and may contact a configured service, so users who do not want that
behavior can leave the option off.

CLI completion never turns the typed line into a command string. Summer resolves
only allowlisted binary executables from `PATH`, uses fixed argument grammars
with `shell: false`, supplies no stdin, removes secret-looking environment
variables and prompt/pager behavior, and bounds concurrency, runtime, output,
and result count. A failure, timeout, unsupported version, unavailable tool, or
unsafe shell expression falls back to the static catalog or the active shell's
native Tab behavior. It never executes a selected completion.

All completion requests go through the same exact-app, owned-session
main-process authorization as PTY input; websites and other Summer Apps cannot
call the channel. The reusable taxonomy and provider backlog live in
[`packages/cli/AUTOCOMPLETE_CATALOG.md`](../packages/cli/AUTOCOMPLETE_CATALOG.md).

Suggestions appear after typing a command prefix; use Up/Down to select, Tab to
complete, Escape to dismiss, or Ctrl+Space to request suggestions. The list is
anchored above the active terminal cursor and remains inside the visible terminal
surface. When Summer has no applicable structured suggestion, Tab is returned to
the shell so native aliases, functions, custom completions, and tool-specific
logic keep working. The optional History panel
(`Ctrl+Shift+H`) lists up to 100 unique commands
from the active session and offers an explicit **Run again** action. History is
memory-only, scoped to one terminal session, clearable at any time, and never
written to Summer storage. Native shell history remains available independently.

The optional **Favorite commands** panel stores up to 50 named, single-line
commands in an explicit versioned local format. Favorites can be added directly
or from session history, edited, reordered, removed, inserted at an empty prompt,
or run at an empty prompt. Opening the panel, selecting an entry, editing it, or
reordering it never executes anything; **Run** is the only favorite action that
adds Enter. Malformed persisted data is ignored, rendered fields use text nodes,
and commands and names are length-bounded.

Saved favorites also appear as named buttons in the main CLI shortcut bar. A
button loads the command without pressing Enter: with no running session it first
opens the selected profile, then inserts the command; with an active session it
inserts only at an empty tracked prompt. The user can review or edit the command
before running it. Shortcuts are disabled while a session is starting or while
the active prompt contains input, so they cannot be appended to a partially typed
command. The complete command is available as the button tooltip, and the Manage
action opens the full favorites editor. Only the full editor's explicit **Run**
action adds Enter.

## Packaging and verification

`node-pty` is a production dependency because each packaged desktop application
needs a native PTY binding and helper. Its native payload is explicitly unpacked
from ASAR. Windows uses ConPTY; Linux packages must contain an executable ELF
`spawn-helper` beside `pty.node`; universal macOS packages retain and verify the
separate arm64 and x86_64 prebuilds. The xterm packages are compiled into the
built-in app assets and do not receive Node.js access.

Release verification covers:

- both Windows portable and Store/AppX targets, including the Store package's
  full-trust desktop declaration;
- Linux AppImage payload validation for `pty.node` and executable
  `spawn-helper`, followed by a harmless PTY smoke test on a Linux host;
- both native slices of the Darwin PTY addon and helper, Developer ID signing,
  hardened runtime, Apple notarization, ticket stapling, Gatekeeper assessment,
  and a harmless PTY smoke test on Intel and Apple-silicon Macs for the direct
  download;
- continued App Sandbox entitlements and unavailable CLI state in the MAS
  edition.

All PTY smoke tests use a disposable browser profile and temporary working
directory. Unit tests use an injected fake PTY adapter and never start a user's
real shell.
