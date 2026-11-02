# scoop-bucket

Personal [Scoop](https://scoop.sh) bucket.

## Usage

### Add this bucket

```powershell
scoop bucket add scoop-bucket https://github.com/hyzor/scoop-bucket.git
```

The first argument is whatever you want to call the bucket locally; `scoop-bucket`
matches the repository name.

### Install an app

```powershell
scoop install <app>
```

Without adding the bucket, you can install straight from a manifest file:

```powershell
scoop install .\<app>.json
```

### Update

```powershell
scoop update
scoop bucket update scoop-bucket
scoop update *
scoop update <app>
```

`scoop update` on its own refreshes the bucket clones, which is what pulls new
manifests down. `scoop update *` only updates installed apps, and it re-pulls
buckets only when Scoop itself has an update, so refresh the bucket first or use
`scoop bucket update scoop-bucket`.

### Uninstall

```powershell
scoop uninstall <app>
```

## Apps

### chatterino-nightly

[Chatterino](https://chatterino.com) Twitch chat client, nightly build.

```powershell
scoop install chatterino-nightly
```

For the stable build, use the official bucket instead:
`scoop install extras/chatterino`.

Notes:

- The manifest uses Scoop's built-in nightly support (`"version": "nightly"`).
  Scoop stamps every install with the date it was made (`nightly-YYYYMMDD`) and
  skips hash verification for it, so a new build is picked up without editing or
  committing anything. Scoop never runs `checkver` during `scoop update`, which
  is why the previous date-in-`version` scheme never noticed new builds.
- Scoop only treats nightly apps as out of date when nightly comparison is
  enabled. Enable it once:

  ```powershell
  scoop config UPDATE_NIGHTLY $true
  ```

  This is a global setting: every `"version": "nightly"` app from any bucket is
  then reported as outdated once per calendar day.
- Versions are compared per calendar day, so a build published later on the same
  day only shows up with the next day's update. To grab the current build right
  away, force it:

  ```powershell
  scoop update chatterino-nightly --force
  ```
- The Windows archive name includes the Qt version
  (`chatterino-windows-x86-64-Qt-6.8.3.zip`). When Chatterino upgrades Qt, the
  `url` must be updated to the new asset name.
- The nightly archive is rebuilt in place and has no published checksums, so
  the manifest carries no `hash` and Scoop does not verify the download.

#### Taskbar pin keeps launching an old build

Pinning Chatterino to the taskbar stores the folder the exe really lives in,
e.g. `...\apps\chatterino-nightly\nightly-20261022\chatterino.exe`, not the
`current` junction. Each nightly update installs into a new dated folder and
moves `current`, so the pin keeps launching the folder it was made from (old
folders stay on disk until `scoop cleanup` removes them).

Point the pin at the junction instead; it then follows every update:

```powershell
$pin = "$env:APPDATA\Microsoft\Internet Explorer\Quick Launch\User Pinned\TaskBar\Chatterino.lnk"
$shell = New-Object -ComObject WScript.Shell
$s = $shell.CreateShortcut($pin)
$s.TargetPath = "$env:USERPROFILE\scoop\apps\chatterino-nightly\current\chatterino.exe"
$s.WorkingDirectory = "$env:USERPROFILE\scoop\apps\chatterino-nightly\current"
$s.Save()
```

If the taskbar still starts the old build afterwards, restart Explorer or sign
out and back in so it drops its cached pinned item.

### deskflow-continuous

[Deskflow](https://deskflow.org) keyboard and mouse sharing, tracking the
rolling `continuous` pre-release.

```powershell
scoop install deskflow-continuous
```

For the stable release, use the official bucket instead:
`scoop install extras/deskflow`.

Notes:

- This follows the `continuous` pre-release, not the latest stable tag. It uses
  Scoop's nightly support (`"version": "nightly"`), so installs are stamped
  `nightly-YYYYMMDD`, hashes are skipped, and new builds are picked up without
  editing the manifest. `UPDATE_NIGHTLY` and the daily comparison behave as
  described for `chatterino-nightly` above.
- Because comparison is per calendar day, `scoop update` re-downloads once a day
  while nightly updates are enabled, even if Deskflow has not published a new
  continuous build that day. Use `--force` to grab the current asset immediately.
- The manifest uses the portable archive. The portable build omits Deskflow's
  background service, so the client does not work at UAC prompts or the login
  screen. If you need those, install the official MSI from the release page or
  use `winget install Deskflow.Deskflow`.
- The portable build stores its settings in `settings/Deskflow.conf`; `persist`
  keeps that folder across updates in
  `%USERPROFILE%\scoop\persist\deskflow-continuous\settings`.
- The asset name carries no version
  (`deskflow-continuous-win-x64-portable.zip`), so there is no URL to update
  when Deskflow changes its packaging.
- `bin` exposes `deskflow-core.exe`; the GUI is launched from the Start Menu
  shortcut.

## How the manifests work

- `url` points directly at the release archive. Scoop downloads it for you.
- `extract_dir` selects the app folder inside the archive, so no manual
  extraction is needed. The Chatterino zip contains a `Chatterino2/` folder; the
  Deskflow zip contains a `deskflow-continuous-win-x64-portable/` folder.
- `bin` / `shortcuts` point at the executable inside that folder.
- `persist` keeps settings, logs, themes etc. across updates.
- `hash` pins the downloaded archive; Scoop refuses to install on a mismatch.
  Both manifests use nightly versioning, so they carry no hash and Scoop skips
  the check for them.

## Maintaining this bucket

`chatterino-nightly` and `deskflow-continuous` deliberately have no
`checkver`/`autoupdate`: their version is the literal string `nightly` and Scoop
fills in the date itself. Running `bin\checkver.ps1 -u` would replace
`"version": "nightly"` with a fixed date and break automatic updates, so do not
run it on these manifests.

Deskflow's asset names are stable, so its `url` needs no maintenance. The only
thing that can go stale is Chatterino's Qt-versioned asset name. Check the
current Chatterino nightly assets with:

```powershell
(Invoke-RestMethod 'https://api.github.com/repos/Chatterino/chatterino2/releases/tags/nightly-build').assets.name
```

If the Windows zip is no longer named `chatterino-windows-x86-64-Qt-6.8.3.zip`,
update `url` to match. The Deskflow asset names can be checked with:

```powershell
(Invoke-RestMethod 'https://api.github.com/repos/deskflow/deskflow/releases/tags/continuous').assets.name
```

Test a manifest change with:

```powershell
scoop install .\<app>.json
scoop update <app>
```
