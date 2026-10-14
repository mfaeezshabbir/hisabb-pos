# Hisabb Releases

Public update feed for **Hisabb** installers.

This repository holds **release binaries only**. It is intentionally public so installed copies of Hisabb can check for updates without a GitHub login. Source code, signing keys, and shop licences live elsewhere and are **never** published here.

| Related | URL |
|--------|-----|
| Source (private) | Managed separately — not this repo |
| Default app feed | `github:mfaeezshabbir/hisabb-releases` |

---

## What belongs here

Each GitHub Release (`vX.Y.Z`) should contain:

| Asset | Purpose |
|-------|---------|
| `HisabbSetup-X.Y.Z.exe` | Generic installer (no shop licence embedded) |
| `HisabbSetup-X.Y.Z.exe.sha256` | SHA-256 checksum the app verifies before installing |

Release notes (English) go in the GitHub Release description.

## What must never go here

- Source code or project files
- Private signing keys (`*.pem`)
- Shop licence files (`*.lic`)
- Per-shop installers (those embed a licence for one customer)
- Databases, tokens, or customer data

---

## How releases are managed

You do **not** commit installers to `main`. Releases are cut from the private Hisabb build machine:

```powershell
# From the private Hisabb repo (build machine with .NET MAUI + Inno Setup)
.\build\publish.ps1 -Version 1.1.0 -NotesEn "What changed" -GitHubRelease
```

That command:

1. Builds a self-contained Windows app  
2. Compiles the Inno Setup installer  
3. Writes `HisabbSetup-1.1.0.exe.sha256`  
4. Creates GitHub Release `v1.1.0` on **this** repo with both assets  

Requirements on the build machine:

- Logged in as the owner: `gh auth login` (account that can push releases here)
- Inno Setup 6 (`winget install JRSoftware.InnoSetup`)
- **No** `-Shop` flag — per-shop builds are refused for GitHub publish

### Day-to-day workflow

```
Private source repo  →  publish.ps1 -GitHubRelease  →  this public repo (Releases)
                                                              ↓
                                              Hisabb app checks latest release
                                              downloads installer + verifies SHA-256
```

| Task | How |
|------|-----|
| Publish a new version | `.\build\publish.ps1 -Version X.Y.Z -NotesEn "…" -GitHubRelease` |
| Fix a bad release | Delete that release on GitHub, then republish the same or a new version |
| Yank an update | Mark the release as draft / delete it; apps only read the latest non-draft release |
| Per-shop installer | Build with `-Shop "…"` and send the `.exe` / `.lic` privately — never attach to this feed |
| Override feed in a shop | Settings → Updates (LAN `update.json` or another URL still works) |

---

## How the app uses this repo

1. Hisabb calls the GitHub API for the **latest** release on `mfaeezshabbir/hisabb-releases`.
2. It looks for `HisabbSetup-*.exe` and a matching SHA-256 (digest or `.sha256` asset).
3. If the version is newer than the installed app (and the shop licence still allows updates), an **Update** chip appears.
4. On install: download → hash check → local DB backup → silent run of the installer.

If nothing has been published yet, the app reports that no release is available.

---

## Licence vs updates

Licensing is separate from this feed:

- Shops load a signed `.lic` (machine-bound / shop-bound as you issued it).
- The app re-checks the licence at launch, sign-in, update check, and periodically at runtime.
- A shop whose licence has an *updates-until* date will not be offered releases published after that date.
- Replacement licences are issued offline with `hisabb-license` and sent to the shop — they are not hosted here.

---

## Safety notes

- Keep this repo **public** (anonymous read of Releases) and the source repo **private**.
- Prefer Authenticode-signing `HisabbSetup-*.exe` so Windows SmartScreen trusts downloads.
- Losing the offline private signing key means you cannot issue new licences; leaking it means anyone can.

---

*Installers only. Not the Hisabb source repository.*
