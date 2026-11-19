# Domeniq

**A private home ledger that files your paperwork for you.** Photograph a receipt, a bill, a warranty card or a car document with your phone; Domeniq reads it, works out what it is, and files it. Everything stays on your own computer at home.

This repository holds the releases only: the Windows installer, the signed release feed that installed copies check for updates, and the compose file for the Docker image (see [Docker](#docker-linux-server)). Each release carries the installer as `DomenIQ-Setup.exe` (the stable name, so that the link [`releases/latest/download/DomenIQ-Setup.exe`](https://github.com/StasEdward/domeniq-releases/releases/latest/download/DomenIQ-Setup.exe) always gives the newest) and as `DomenIQ-Setup-<version>.exe`, the same file under its versioned name (what installed copies download when they update), with its SHA-256.

## What you need

- A Windows 10 (version 1809 or later) or Windows 11 PC, 64-bit, that stays on while you use Domeniq. No graphics card is needed.
- An internet connection while installing: the installer downloads the Hebrew text recognition models (about 180 MB) once.
- A model that reads the documents: Domeniq connects its own hosted model by itself (25 documents are free to try), or you can use your own Anthropic, OpenAI-compatible or local model.
- A phone or another computer on the same home network, to photograph and upload documents.

## Install

1. Download [`DomenIQ-Setup.exe`](https://github.com/StasEdward/domeniq-releases/releases/latest/download/DomenIQ-Setup.exe) (or `DomenIQ-Setup-<version>.exe`, the same installer, from the [latest release](https://github.com/StasEdward/domeniq-releases/releases/latest)).
2. Run it. The installer is not code-signed yet, so Windows SmartScreen shows "Windows protected your PC": click **More info**, then **Run anyway**. If you want to be sure the file is the published one, run `Get-FileHash .\DomenIQ-Setup.exe` in PowerShell and compare the hash with the one in the release notes.
3. Allow the administrator prompt. Setup asks nothing else: it puts the program in `C:\Program Files\DomenIQ`, picks a free port (80 when it is free, else 8080 or the next free one), adds a Domeniq icon to the Start menu and the desktop and uses your Windows language for the screens. It shows one more page only when you ran Domeniq before from a console: **Existing household** moves that household into the service's folder (close the old Domeniq window first). Only the data moves: a llama.cpp model the console node used does not carry over, so connect a model on **Settings > AI model** afterwards, or set it again under `"Ai": { "Local": { "Endpoint": "http://<host>:8081/v1" } }` in the `DomenIQ` section of `C:\Program Files\DomenIQ\node\appsettings.Production.json` and run `Restart-Service DomenIQ`.
4. Leave **Open Domeniq** checked and click **Finish**. The browser opens the setup page.

If Setup says that `C:\ProgramData\DomenIQ` already exists and belongs to another account, that folder was not made by Domeniq's installer (any user can create folders there). Rename or delete it as an administrator and run Setup again. If Setup says the folder holds a Domeniq database, it may be your household (copied there by hand, say): if you know it is yours, take ownership of the folder and everything in it as an administrator (Properties > Security > Advanced > Owner: Administrators, replacing the owner on subcontainers and objects), or contact support, then run Setup again; never take over a folder you did not make.

## First run

1. Name the household and choose the owner password. Under **Who reads the documents** leave the Domeniq model chosen: Domeniq asks clients.domeniq.app for a free trial key by itself (25 documents, 30 days; one trial per computer), and documents are read from your first upload. A banner asks you to save the recovery phrase (**Save as file** or **Print**): it is the only way to restore a backup on another computer.
2. When the trial is used up or ran out of time (or was refused), the Overview offers **Get a plan**; after you subscribe and the payment is confirmed, the paid key reaches Domeniq by itself.
3. On your phone: click the Domeniq icon in the notification area, choose **Phone address**, and open that address on the phone (or scan the QR code). Sign in with the owner password and upload a photo of a receipt.

If the phone cannot reach Domeniq, check that the phone is on the same Wi-Fi and that Windows treats this network as **Private** (Settings > Network and internet > your network > Network profile type): the installer opens the port for private networks only.

## What is installed

- `C:\Program Files\DomenIQ`: the program (`node`), the tray icon (`tray`) and the uninstaller.
- The Windows service **Domeniq home node** (`DomenIQ`), started automatically with Windows and restarted if it stops.
- `C:\ProgramData\DomenIQ`: your household (the database, the encrypted documents, the household key, the text recognition models). Only administrators and Windows itself can open this folder.
- A Windows Firewall rule **Domeniq** for the port, on private networks.
- The tray icon, started when you sign in: green when Domeniq runs, amber while it starts or updates, grey when it is stopped.

## Update

Domeniq checks this repository once a day. When a new version is out, the Overview and **Settings** show "Version X is available". Click **Update** in Settings and type your password: Domeniq checks the release feed's signature, downloads the installer, checks its size and SHA-256 against the signed feed, and installs it; the page reconnects by itself after a minute or so. This works from the phone too.

You can also download the new installer and run it over the old one: your household and settings stay.

Before an update changes the database, Domeniq keeps a copy of it in `C:\ProgramData\DomenIQ\backups`. If a new version does not start, see "Restoring after a failed update" below.

## Back up

**Settings > Backup** downloads an encrypted backup of the whole household. For automatic backups, add `"Backup": { "Directory": "D:\\DomeniqBackups" }` (a folder on another drive that exists) to the `DomenIQ` section of `C:\Program Files\DomenIQ\node\appsettings.Production.json` and restart the service (`Restart-Service DomenIQ`). Keep the recovery phrase: a backup cannot be opened without it.

## Restoring after a failed update

If Domeniq does not start after an update (the tray icon stays grey), in PowerShell as administrator:

1. Run the installer of the previous version from the [releases page](https://github.com/StasEdward/domeniq-releases/releases).
2. `Stop-Service DomenIQ`
3. In `C:\ProgramData\DomenIQ`, delete `homevault.db-wal` and `homevault.db-shm`, then copy the file of the version you updated to from `backups` (`before-<version you updated to>-<date>.db`, one per version) over `homevault.db`.
4. `Start-Service DomenIQ`

What was recorded after the update started is lost.

## Uninstall

Windows Settings > Apps > Installed apps > **Domeniq** > Uninstall. The uninstaller asks whether to **also delete your household data**; the answer is No unless you choose Yes. With No, `C:\ProgramData\DomenIQ` stays and a later install finds your household again. Download a backup first if you are not sure.

## Docker (Linux server)

For a Linux server, x86-64 or ARM64 (a NAS, a mini PC, a Raspberry Pi 5), with Docker and its Compose plugin. Each release publishes the image `ghcr.io/stasedward/domeniq` (tags `<version>`, `<major>.<minor>` and `latest`) and attaches its compose file as `docker-compose.yml`.

```bash
mkdir domeniq && cd domeniq
curl -fsSLO https://github.com/StasEdward/domeniq-releases/releases/latest/download/docker-compose.yml
docker compose up -d
```

Then open `http://<server>/` from a computer on the home network and set Domeniq up at once: until a household exists, the first visitor sets it up. The rest is as on Windows (see First run).

- The container uses the server's own network (host networking), so Domeniq sees the server's address on the home network, gets its secure name `<label>.n.domeniq.app` and listens on the server's ports 80 (HTTP) and 443 (HTTPS). Open both in the server's firewall for the home network. Other ports, a fixed version or the first screen's language go into a `.env` file next to the compose file; its comments list them. Host networking needs a Linux host: Docker Desktop on Windows or macOS is not supported, use the Windows installer there.
- Your household lives in the Docker volume `domeniq_domeniq-data`. **Settings > Backup** and the recovery phrase work as on Windows.
- Update: when a new version is out, the Overview and **Settings** say so (Domeniq checks this repository once a day), and **Settings** shows the command: `docker compose pull && docker compose up -d`. The container does not update itself; the volume stays, and Domeniq upgrades the database when it starts.
- The image is signed. To check it: `cosign verify ghcr.io/stasedward/domeniq:<version> --certificate-identity-regexp '^https://github.com/StasEdward/HomeVault/' --certificate-oidc-issuer https://token.actions.githubusercontent.com`.

## Licenses

Domeniq includes, under their own licenses:

- .NET, ASP.NET Core and Entity Framework Core (MIT), SQLite (public domain), ONNX Runtime (MIT), SkiaSharp (MIT), QRCoder (MIT), PdfPig (Apache-2.0).
- [HebrewOcrNet](https://github.com/StasEdward/HebrewOcrNet), on top of [RapidOcrNet](https://github.com/BobLd/RapidOcrNet) (Apache-2.0).
- The PaddleOCR Hebrew models of [paddleocr-hebrew](https://github.com/RivoksLab/paddleocr-hebrew) by Rivok Labs (Apache-2.0), downloaded by the installer from this repository's `paddle-models-*` release, which carries their LICENSE and NOTICE; the Docker image carries the same files and the LICENSE in `/opt/domeniq/models/paddle`. The image also includes Ubuntu packages, among them [Tesseract](https://github.com/tesseract-ocr/tesseract) (Apache-2.0), each with its copyright file in `/usr/share/doc`. (Maintainers: such a release is never marked latest, `gh release create ... --latest=false`, so that `releases/latest/download/releases.json` stays the newest version's feed.)

The license of Domeniq itself has not been chosen yet.
