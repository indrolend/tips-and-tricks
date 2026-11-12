# Tips & Tricks

A practical cheat sheet of useful computer workflows. These are small techniques worth remembering because they save time, remove repetitive steps, or make work easier to recover.

The names below are memorable labels, not commands or a formal system. Search by the situation you are in.

**Status labels:** **Tested** = observed working in an actual session. **Setup-specific** = worked in a particular machine, shell, app, or configuration. **Starting point** = useful recipe that may need adjustment.

---

## Clipboard: copy once, use everywhere

### AutoCopy — copy command output without selecting it

**When:** You want to paste terminal results into ChatGPT, Codex, an issue, or another app.

PowerShell 7 (for native commands):

```powershell
$output = & { <your-command> } 2>&1
$exit = $LASTEXITCODE
$body = ($output | ForEach-Object { "$_" }) -join [Environment]::NewLine
$receipt = $body + [Environment]::NewLine + "EXIT=$exit"
$receipt
Set-Clipboard -Value $receipt
'✓ copied'
```

Replace `<your-command>` with the command you want to run. This captures combined output, preserves the native exit code, displays the receipt, and copies it. For PowerShell cmdlets/functions, capture `$?` immediately instead of relying on a potentially stale `$LASTEXITCODE`.

macOS / zsh starting point:

```zsh
result=$(your-command 2>&1)
status=$?
printf '%s\nEXIT=%s\n' "$result" "$status" | tee /dev/tty | pbcopy
printf '✓ copied\n'
```

**Remember:** `Set-Clipboard` and `pbcopy` copy text. They do not copy files.

### CompactCopy — copy only the useful lines

**When:** A command produces pages of output but only a few lines matter.

```powershell
$result = some-command | Select-String 'PASS|FAIL|ERROR|SHA256|EXIT'
$text = $result -join [Environment]::NewLine
$text
Set-Clipboard $text
'✓ copied'
```

Adjust the patterns to the task. Keep the full log or evidence file when it may be needed later; a compact receipt is not a replacement for the raw output.

### FileCopy — copy an actual file, not its path

**When:** You need to attach a ZIP, image, video, audio file, or document to a chat.

- **Windows:** In File Explorer, select the file and press **Ctrl+C**, then paste it into the destination.
- **macOS:** In Finder, select the file and press **Command+C**, then paste it into the destination.
- **Path only:** `Set-Clipboard "C:\path\file.zip"` copies the text of the path, not the file.
- **Image clipboard:** a copied image payload is different from copying a PNG file object.

Some INDRO replay workflows provide `--clipboard` for the evidence ZIP and `--clipboard-image` for a captured image. Use those when available and verify the result.

### CopyCheck — verify the clipboard

After copying text, paste it into a harmless text field or read it back with the platform's clipboard inspection command. Do not report a successful copy just because the copy command returned without an error.

**Useful distinction:** text clipboard, image clipboard, and file-drop clipboard are three different things.

---

## Terminal: keep the useful context

### OneReceipt — make a command self-reporting

A useful command receipt often includes what ran and where, machine/project/repository, branch and commit, duration and exit code, decisive output lines, artifact paths, and evidence IDs.

For repeated work, make the command print and copy this compact receipt automatically while keeping the full log separately.

### MillisecondPrompt — show how long commands take

**Tested, Starship + PowerShell setup:** the prompt can show every command's duration, including milliseconds.

In `~/.config/starship.toml`:

```toml
[cmd_duration]
show_milliseconds = true
min_time = 0
style = "teal"
format = "[· $duration]($style)"
```

Set `min_time = 0` to show short commands too. Starship can also show directory, Git branch/status, time, and command exit status when configured.

### ProjectHistory — search the right command history

**When:** You remember doing something but cannot remember the command.

From the relevant project root:

```powershell
indro history 10
indro search "distinctive term"
```

Start with a small history count and increase only if needed. INDRO history is project-scoped: running it from your home directory may return `No Git repository`. That means the working directory is wrong, not that history is missing.

CommandHUD run IDs and INDRO evidence UUIDs are different identifiers; use the matching retrieval command.

### Transcript — record a real terminal session

**When:** You need a faithful record of what happened.

```powershell
Start-Transcript -Path "$HOME\Desktop\session-transcript.txt"
# Work normally
Stop-Transcript
```

A transcript records the terminal session. Keep it separate from later reasoning notes or summaries. Do not reconstruct terminal output from memory and call it a transcript.

### HUDHistory — recover output without rerunning

**Documented in CommandHUD:** in `hud shell` / `hud tui`, slash commands can inspect retained runs:

- `/history [n]` — recent runs.
- `/head [run] [count]` and `/tail [run] [count]` — bounded output.
- `/find [run] pattern` and `/around [run] pattern [lines]` — find a relevant section.
- `/raw [run]` — complete output.
- `/diff run run` — compare two runs.
- `/copy` and `/context` — copy/recover the current context.

Use bounded output first. Full logs are available when needed.

### ClipboardHandoff — send the next person the useful context

A good handoff is not a giant chat dump. Include objective/status, repository root/branch/HEAD/dirty state, exact commands and exit codes, relevant source/tests, evidence IDs/logs/artifact paths, what is verified versus inferred, and the next useful action.

Keep original logs and artifacts; the handoff is an index and explanation, not a replacement for evidence.

---

## Files and artifacts: keep the thing you made

### KeepTheVideo — retain continuous video

A screenshot shows one instant. A video shows timing, movement, transitions, glitches, and whether behavior continues over time.

When those things matter, export and retain the actual MP4 (or other source format), not just a collection of screenshots. Keep useful still frames alongside it.

### CaptureSet — save the right evidence

| What you need to show | Useful artifact |
|---|---|
| One visual state | PNG / JPG |
| Motion or timing | MP4 / video |
| Sound or recording | WAV / audio |
| Animation or web graphic | GIF |
| Structured measurements | JSON / CSV |
| Reproducible work package | ZIP + manifest |
| Exact terminal session | Text transcript |
| Project that can be edited again | Native project file |

Avoid generating every possible artifact for every task. Keep the original editable project when future changes are likely.

### HashIt — identify an exported file

PowerShell:

```powershell
Get-FileHash "C:\path\to\artifact.zip" -Algorithm SHA256
```

macOS / Linux:

```bash
shasum -a 256 /path/to/artifact.zip
```

A matching SHA-256 checks whether two files have identical bytes. A hash alone does not prove where a file came from or whether its contents are correct.

### EvidenceZip — package a bounded handoff

Include only the files needed to understand or reproduce the result: source/configuration, commands, logs, test output, screenshots/video, manifest, and checksums.

PowerShell starting point:

```powershell
Compress-Archive -Path .\handoff\* -DestinationPath .\handoff.zip
Get-FileHash .\handoff.zip -Algorithm SHA256
```

Check the archive contents and file count. Do not include credentials, private keys, tokens, or unrelated personal files.

### SaveACopy — preserve the editable original

Before destructive edits, format conversion, or major changes, save a separate working copy. Exported PNG/MP4/WAV files are outputs; they do not replace an editable project such as an FLP, source tree, or native design file.

---

## Remote access and moving between devices

### TailscaleThenSSH — reach your own computer

**Tested setup:** Tailscale connects the devices; SSH provides a shell on the remote machine.

1. Check that the intended Tailscale peer is online.
2. Use its current Tailscale IP or DNS name; do not rely on an old address.
3. SSH to the intended machine/account.
4. Verify hostname, user, working directory, and repository before running commands.

A remembered IP, hostname, or machine name is not proof that it is still current.

### Reattach — continue a tmux session

**Tested setup:** after connecting to a remote Linux/WSL environment, attach to the existing tmux session rather than starting another copy of the work.

```bash
tmux ls
tmux attach -t development
```

To recover a bounded amount of terminal scrollback:

```bash
tmux capture-pane -p -S -1000
```

The session name is an example; use the session actually listed on that machine.

### Taildrop — send files to another device

Use Tailscale's Taildrop when both devices support it and are connected to the same tailnet. Check the current peer/device name and the received file location. Keep a separate copy of important originals.

### RemoteGUI — capture the real desktop

A headless SSH/WSL shell may not share the logged-in desktop session. If a screenshot or GUI action needs the interactive Windows desktop, use a capture/control mechanism running in that desktop session. Verify its completion result and the resulting image before claiming success.

---

## Applications and automation

### GUIScout — find the real application before automating it

A reusable GUI discovery sequence:

1. List running applications/windows.
2. Identify the intended window and process.
3. Capture a screenshot.
4. Inspect the accessibility/UI tree when available.
5. Identify controls by labels, roles, or stable identifiers.
6. Perform one bounded interaction.
7. Read back the resulting state or capture another screenshot.

Coordinates and guessed keyboard shortcuts are fallbacks. A window being visible does not prove that an automation layer can interact with it.

### UIPath — turn a repeated interaction into a recipe

When the same UI operation recurs, record the target window, control identity, action, required wait, and expected result. Replay it only after confirming the target application/window is the intended one. Keep a screenshot or state readback when the result matters.

### FLBridge — control FL Studio through its bridge

**Tested project workflow:** INDRO's FL bridge exposes inspection and parameter operations. Discover the live host before changing anything.

```powershell
indro fl doctor --json
indro fl channels list --json
indro fl plugins list --json
```

Inspect the target plugin and its parameters before reading or changing a value:

```powershell
indro fl plugin inspect --channel N --json
indro fl plugin params --channel N --json
indro fl plugin get --channel N --param N --json
```

A parameter write, where supported and intended:

```powershell
indro fl plugin set --channel N --param N --value 0.5 --verify --json
indro fl verify --latest --json
```

Replace `N` with the discovered channel/parameter. A successful host readback confirms state, not necessarily that the audio sounds correct. Listen to or record the actual output when sound is the thing being tested.

### AudioLoopback — record what the computer is playing

**Tested on one Windows workstation:** Python `soundcard` loopback successfully recorded the selected speaker endpoint to WAV. The exact endpoint varies by machine; enumerate devices first.

1. Identify the actual output endpoint.
2. Select a loopback-capable capture path.
3. Record a short sample.
4. Check duration, sample rate, channels, and file size.
5. Listen to it before treating the capture as valid.

Do not assume FFmpeg WASAPI is available in every installation. A microphone recording is not a substitute for desktop loopback.

---

## Development and debugging

### ReplayCapture — reproduce gameplay with evidence

**Tested INDRO workflow:**

```powershell
$env:INDRO_EVIDENCE_ROOT = "$HOME\Desktop\game-evidence"
indro godot replay --scenario smoke --capture --video
```

Use `--video` when continuous behavior matters. A replay can produce assertions, observations, logs, screenshots, video, a manifest, and a ZIP. Check the actual replay result and artifacts; exit zero alone is not proof that gameplay behaved correctly.

### VisualAblation — isolate a visual defect

Instead of changing several rendering variables at once:

1. Reproduce the same scene, camera, simulation state, and build.
2. Disable or isolate one light, shadow caster group, material family, or geometry group.
3. Capture the result.
4. Restore the baseline and test the next variable.
5. Combine only the factors that actually reproduce the defect.

For interaction effects, use a small parameter sweep or controlled combinations. Preserve the baseline capture. This is especially useful for shadow artifacts, lighting stains, missing shadows, and material glitches.

### FirstDivergence — find where two runs stop matching

For deterministic or parity tests, compare the earliest state/frame that differs. Later differences may be consequences of the first one. Inspect the changed fields and trace the code that owns them before changing the system.

### ProofReuse — don't rerun blindly

Before repeating a long verification, look for an existing successful run. Reuse it only if the relevant source, build graph, environment, dependencies, executable, inputs, and behavioral contract still match.

A cached/reused result saves work; it is not a fresh execution. If identity changed, run the verification again.

### CTestConfig — remember Visual Studio's configuration

For multi-configuration Visual Studio CMake builds, specify the configuration:

```powershell
ctest --test-dir .\build -C Release --output-on-failure
```

Without `-C Release` (or the configuration actually built), CTest may report tests as unavailable / not run. That is not the same as a test assertion failing.

### GitWorktree — compare versions without disturbing current work

When testing an older branch or radically different implementation, use a separate Git worktree and separate build directory. Keep the current dirty worktree intact. Compare the candidates with the same scenario and capture settings before deciding what to merge.

---

## Media and everyday computer tasks

### DownloadAudio — extract audio with yt-dlp

**Starting point; requires yt-dlp and FFmpeg:**

```powershell
yt-dlp -x --audio-format wav "https://www.youtube.com/watch?v=VIDEO_ID"
```

This extracts audio and converts it to WAV. Check the resulting file and listen to it. Respect the source's copyright and the platform's terms.

### ResizeImage — change dimensions without silently changing the image

First decide what “3000 × 3000” means:

- **Stretch:** exact dimensions, but changes proportions.
- **Fit inside:** preserves the whole image, but may produce smaller dimensions on one side.
- **Crop to square:** exact dimensions, but removes part of the image.
- **Pad to square:** exact dimensions, preserves all content, adds canvas around it.

Do not crop, stretch, recolor, sharpen, or otherwise edit the image when the request is only to resize it. State which method was used.

### QRandBarcode — encode text as a scannable image

Use a QR code for general text/URLs and a barcode format compatible with the target scanner. Verify by scanning the generated result and comparing the decoded text exactly with the input. Do not assume every barcode format supports arbitrary strings.

### AlbumArt — find the actual image behind a music page

When a music page exposes an album identifier, look for its image/API metadata rather than screenshotting the page. Check the image dimensions and whether the returned asset is the original or a resized thumbnail. Do not label an image “highest resolution” without checking.

### WebsiteAsset — replace a site asset through Git

For a site deployed from a Git-connected host such as Cloudflare Pages:

1. Identify the actual website repository and asset path.
2. Preserve the current asset.
3. Replace the intended file with the new export.
4. Check dimensions, transparency, animation, and file size.
5. Commit and push to the deployment branch.
6. Wait for the deployment to finish.
7. Open the live site and verify the actual served asset.

A successful Git push does not prove that deployment finished or that the browser is showing the new asset; cache may need to be considered.

---

## Recovery and troubleshooting

### ChromeHijack — diagnose search/new-tab redirects

**Tested investigation:** a Chrome hijack persisted after reinstall. The unwanted URL was found in both profile preference files and SQLite search-provider rows.

1. Identify the exact Chrome profile and close every Chrome process.
2. Back up Preferences, Secure Preferences, and Web Data.
3. Inspect default search/new-tab settings in both JSON preference files.
4. Inspect active provider rows in the Web Data `keywords` table.
5. Compare the same provider across stores; similar names/keywords can refer to different rows.
6. Make only the targeted repair after preserving a backup.
7. Relaunch Chrome and inspect again; close normally and inspect once more.

If the setting returns, find what rewrites it (policy, extension, process, or preference state). Repeating the same database edit is not a diagnosis.

### PrinterPath — separate the device from the application

**Tested DYMO LabelWriter 450 sequence:**

1. Inspect printer status and queued job IDs/states.
2. Test the physical feed button.
3. Inspect status and queue again.
4. Only then test the business application.

A working feed proves the mechanism can advance labels. It does not prove that postage, a postmark, or an application-generated label was printed.

### AudioSync — debug recording/playhead lag

When recorded audio and the playhead drift or disagree:

1. Identify the active FL Studio audio device and actual input/output endpoints.
2. Check buffer size and plugin latency on the recording path.
3. Compare a short recording with desktop loopback.
4. Bypass one plugin or change one audio-path variable at a time.
5. Check alignment over time, not only at the start.

Do not assume that a plugin is the cause until the timing comparison isolates it.

---

## Small habits that pay off

- **KeepTheOriginal:** save an editable copy before destructive changes.
- **NameTheOutput:** use a clear filename and a predictable output folder.
- **LatestFirst:** make the newest useful artifact easy to locate.
- **OneGoodReceipt:** copy a compact result, retain the full evidence separately.
- **CheckTheActualResult:** open the exported file, scan the QR, listen to the WAV, play the video, or inspect the live website.
- **UseTheRightMachine:** a path, process, device, or build from yesterday may not be today's active one.
- **RememberTheWorkingSequence:** when a multistep process succeeds, save the steps here while they are fresh.
- **ShareTheShortcut:** if a trick saves time more than once, write it down so somebody else can use it too.

---

## Add a trick

Use this small template:

```markdown
### ShortName — what it helps with

**When:** The situation where this is useful.

1. First step.
2. Next step.
3. Check the result.

**Example:**
```text
paste a working command here
```

**Status:** Tested / Setup-specific / Starting point.
**Notes:** Important limitation or dependency.
```


### DownloadAudio — choose the output folder

A tested Windows PowerShell example:

§§§powershell
yt-dlp -x --audio-format wav -P "$HOME\Downloads" "https://youtu.be/VIDEO_ID"
§§§

Requires FFmpeg. WAV is an uncompressed output container, but converting a compressed source to WAV does not restore information that was lost in the original compression.

### RemoteEvidencePacket — capture a desktop and its terminal together

**Tested cross-machine sequence:** when the task needs both a real Windows desktop screenshot and the remote terminal state:

1. Connect to the Windows machine over the trusted Tailscale/SSH path.
2. Recover the existing WSL/tmux session and capture bounded terminal scrollback with §tmux capture-pane -p -S -1000§.
3. Trigger the interactive desktop screenshot task in the logged-in Windows session. A headless SSH screenshot is not equivalent.
4. Wait for the task's completion result and session marker before using the image.
5. Create a small manifest describing the machine/session, timestamp, screenshot, terminal capture, and relevant run/evidence identity.
6. Validate the PNG, package screenshot + terminal + manifest into a timestamped ZIP, and calculate SHA-256.
7. Retrieve the actual ZIP over the approved SSH/SCP/SFTP path and verify its hash after transfer.

**Observed example:** a successful packet contained §desktop.png§, §terminal.txt§, and §manifest.txt§ in a timestamped ZIP; it was transferred to another PC. The interactive capture required a logged-in Windows session. Do not assume tmux survives WSL termination, Windows logout, or reboot merely because a detached session worked once.

### WebsiteDeploy — replace an asset on a Git-connected site

**Tested workflow with a Cloudflare-connected Git repository:**

1. Identify the exact website repository and the path of the currently deployed asset.
2. Preserve the old asset or its Git revision.
3. Put the replacement at the same intended path/name, checking transparency, dimensions, animation, and file size.
4. Inspect the diff to confirm only the intended asset changed.
5. Commit and push to the deployment branch.
6. Check the hosting provider's deployment result.
7. Open the live website and inspect the served asset; do not treat a successful push as deployment proof.

This workflow was used to replace a GitHub-style GIF on indrolend.com. The source asset was supplied locally; Cloudflare was already connected to Git. Exact repository paths and deployment branch are site-specific.

### FileTransfer — move a file between your own machines

Use the simplest transfer that fits:

- **Same Tailscale network:** Taildrop is convenient for direct device-to-device file transfer.
- **Remote shell available:** use SCP/SFTP for a known file or bounded evidence packet.
- **Chat attachment:** copy the actual file object, not a text path.
- **Large or repeatable handoff:** package a small manifest + artifacts, transfer it, then compare SHA-256 at the destination.

Always verify the received file exists and matches the source. A successful transfer command is not a substitute for checking the destination bytes.
