# PaymoneyWubby VOD Archive Downloader

A simple PowerShell script that bulk-downloads the [PaymoneyWubby VOD archive](https://archive.wubby.tv/) and organizes every video into a clean, dated folder structure on your own drive.

The archive serves videos through short-lived signed links, so you can't just point a normal downloader at the page URLs (you'll end up with a bunch of useless ~12 KB HTML files). This script handles that for you: for each video it requests a fresh signed download link and saves the real `.mp4` to the right place.

## What you get

Every VOD, sorted into folders by year and month, with filenames based on the date the stream aired:

```
<SaveRoot>\
├── 2023\
│   ├── 01-January\
│   ├── 02-February\
│   └── ...
├── 2024\
├── 2025\
└── 2026\
    └── 05-May\
        ├── 2026-05-26.mp4
        ├── 2026-05-27_part01.mp4
        └── 2026-05-27_part02.mp4
```

**As of the last update, that's roughly 1,000 videos totaling about 7.5 TB.** Make sure you have the space (an 8 TB drive is comfortable; 4 TB is not enough for the full set).

## Naming convention

- **Untitled streams** (shown as "no-title" on the site) are named by date: `YYYY-MM-DD.mp4`
- **Titled streams** add the title after the date: `YYYY-MM-DD_The Stream Title.mp4`
- **Multiple streams on the same day** get a sequence suffix in air order: `_part01`, `_part02`, etc.

Titles are cleaned of characters Windows doesn't allow in filenames and trimmed to a safe length so paths stay within Windows limits.

## Requirements

- **Windows 10 or 11** — `curl.exe` ships with both, so there's nothing to install.
- Enough free disk space for whatever you're downloading (up to ~7.5 TB for everything).
- An internet connection. A wired connection is recommended for a download this large.

## Setup & usage

1. Download `Download-WubbyArchive.ps1`.
2. Open it in any text editor (Notepad is fine) and change **one line** near the top:

```powershell
   $SaveRoot = "Y:\Paymoneywubby_VOD_archive"   # <-- set this to wherever you want the videos
```

   That's the **only** thing you need to edit. For example: `"D:\Wubby"` or `"C:\Users\You\Videos\Wubby"`.

3. Save the file, then run it from PowerShell:

```powershell
   powershell -ExecutionPolicy Bypass -File ".\Download-WubbyArchive.ps1"
```

   (Or right-click the file in Explorer → **Run with PowerShell**.)

The script will create the folders automatically and start downloading. It prints progress as `[N/1000] Downloading -> ...` and shows a summary when it finishes.

## Tips

- **It's safe to stop and resume.** Files that are already fully downloaded are skipped, so you can close it and run it again later to pick up where you left off. Running it overnight (or over several sessions) is a good idea given the size.
- **Want just a portion?** It downloads everything by default. If you only want certain years, you can delete the lines you don't want from the `$jobs = @( ... )` list before running.
- **Retrying failures.** If any files fail (network blips, etc.), the script reports the count at the end. Just run it again — it retries anything that's missing or incomplete.

## How it works

The archive's videos live on Cloudflare R2 storage and are only accessible via signed URLs that expire after 24 hours. For each video, the script:

1. Sends the video's key to the archive's link endpoint (`archive.wubby.tv/rpc/getPresignedUrl`).
2. Receives a fresh, signed download URL.
3. Downloads the real `.mp4` with `curl` straight to its destination folder.

Because each link is generated right before it's used, expiring URLs are never a problem — even across long or interrupted runs.

## Troubleshooting

- **"running scripts is disabled on this system"** — Use the `-ExecutionPolicy Bypass` flag shown in the usage command above.
- **Files are tiny (a few KB) and won't play** — That means a normal page URL was downloaded instead of the video. This script avoids that by using the signed-link endpoint; tiny leftover files from other tools (under 1 MB) are treated as incomplete and re-downloaded automatically.
- **Downloads are slow** — This is a very large dataset; speed depends on your connection and the server. Let it run in the background.

## Disclaimer

This is a community convenience tool for downloading content that the archive already makes publicly available. Please be respectful of the host's bandwidth, and only download what you actually intend to keep. This project is not affiliated with or endorsed by PaymoneyWubby or the archive's operators.
