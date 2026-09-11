# mac-spotlight-doctor

[![Release](https://img.shields.io/github/v/release/zhuhroscar-tech/mac-spotlight-doctor?include_prereleases&label=release)](https://github.com/zhuhroscar-tech/mac-spotlight-doctor/releases/tag/v0.1.0)

A tiny macOS CLI utility that helps users investigate **why an external volume
isn't safe to eject because Spotlight or other processes keep handles open**.

## Simple explanation

If an external drive won't eject and macOS mentions Spotlight or `mds_stores`,
this tool checks whether Spotlight is actively indexing that drive and shows
you what's holding it open, in plain language. It can also turn Spotlight
indexing off for just that drive if you want to eject it faster — it never
force-quits a process or touches indexing anywhere else.

```text
$ mac-spotlight-doctor /Volumes/YourDrive
Spotlight indexing: active on /Volumes/YourDrive
Open handles: mds_stores (pid 61), mdworker_shared (pid 4021)
Suggestion: run with --spotlight off to pause indexing on this volume, then
retry eject.
```

It is intentionally conservative: first shows who is holding the volume and
whether Spotlight indexing is active for that path, then optionally lets you
toggle Spotlight indexing off/on for the target.

## Why this exists

Demand appears repeatedly across English, Chinese, and Japanese communities:

- People report drives staying "in use" when macOS blocks ejection and often see
  `_mds_stores`/Spotlight activity.
- Chinese community traffic around `Dec 13, 2025` explicitly mentions adding the
  external drive to Spotlight exclusion as a workaround.
- Japanese resources document adding external media to Spotlight privacy/exclusion
  when ejection and indexing behavior is problematic.

Built-in tooling already exists (`diskutil`, `lsof`, `mdutil`), but command output is
noisy and fragmented. This tool turns it into a small, consistent diagnostic flow.

## Features

- Detect open file handles for a path/volume with `lsof`.
- Read disk metadata via `diskutil` when available.
- Detect Spotlight indexing state for the target with `mdutil -s`.
- JSON output (`--json`) for scripts.
- Toggle Spotlight indexing at that mount path only (`--spotlight on|off`).
- Optional status-only query (`--spotlight status`).

## Install (developer)

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -e .
mac-spotlight-doctor --help
```

## Install from release artifact

```bash
curl -LO https://github.com/zhuhroscar-tech/mac-spotlight-doctor/releases/download/v0.1.0/mac_spotlight_doctor-0.1.0-py3-none-any.whl
curl -LO https://github.com/zhuhroscar-tech/mac-spotlight-doctor/releases/download/v0.1.0/SHA256SUMS.txt
shasum -a 256 -c SHA256SUMS.txt
python3 -m pip install --user --force-reinstall mac_spotlight_doctor-0.1.0-py3-none-any.whl
```

## Usage

```bash
# Read-only diagnosis
mac-spotlight-doctor /Volumes/YourDrive

# JSON for scripts
mac-spotlight-doctor --json /Volumes/YourDrive

# Turn indexing off temporarily (no force kill), then re-check
mac-spotlight-doctor --spotlight off /Volumes/YourDrive
mac-spotlight-doctor --spotlight status /Volumes/YourDrive
```

## Safety / privacy

- No telemetry.
- No network calls.
- No automatic process termination.
- Commands are local and OS-level only.

## Limits

- Requires `lsof`, `diskutil`, and `mdutil` (macOS standard tools).
- It can suggest likely causes, but does not replace Disk Arbitration or full storage maintenance tooling.
- Not signed/notarized; this is a source/packaged Python artifact.

## Uninstall

```bash
pip uninstall mac-spotlight-doctor
```

## License

MIT.
