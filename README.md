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
scoop update <app>
```

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

- The nightly release tag (`nightly-build`) has a fixed `published_at`, so the
  manifest derives the version from the Windows asset's `updated_at` instead.
- The Windows archive name includes the Qt version
  (`chatterino-windows-x86-64-Qt-6.8.3.zip`). When Chatterino upgrades Qt, the
  `url` in both `architecture.64bit` and `autoupdate` must be bumped.
- The nightly build has no published checksums. If no `hash` is present under
  `autoupdate`, Scoop downloads the archive and hashes it locally.

## How the manifests work

- `url` points directly at the release archive. Scoop downloads it for you.
- `extract_dir` selects the app folder inside the archive, so no manual
  extraction is needed. The Chatterino zips contain a `Chatterino2/` folder.
- `bin` / `shortcuts` point at the executable inside that folder.
- `persist` keeps settings, logs, themes etc. across updates.
- `hash` pins the downloaded archive; Scoop refuses to install on a mismatch.

## Maintaining this bucket

Check for a new version:

```powershell
& "$env:USERPROFILE\scoop\apps\scoop\current\bin\checkver.ps1" <app>
```

Update the manifest automatically:

```powershell
& "$env:USERPROFILE\scoop\apps\scoop\current\bin\checkver.ps1" <app> -u
```

Test the result with:

```powershell
scoop install .\<app>.json
```
