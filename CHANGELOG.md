# Changelog

All notable changes are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project uses
[Semantic Versioning](https://semver.org/).

## [Unreleased]

### Fixed

- Windows: profiles open again with newer Claude Desktop versions, which move their data to
  `%LOCALAPPDATA%\Claude-Data` and then ignore `--user-data-dir`. Profiles now start with
  `LOCALAPPDATA` written as a `\\?\` path, which makes Claude skip that move.
- Windows: Claude in Chrome pairing finds the regular Claude's manifest in its new locations.

### Changed

- `Data` is a reserved profile name, like `3p`: Claude uses the `Claude-Data` folder itself.

## [0.1.0] - 2026-10-13

### Added

- `multi-claude add | list | open | remove | doctor | uninstall` on Windows, macOS (beta) and Linux
  (experimental).
- Profiles: the official Claude Desktop with its own `--user-data-dir`, so each keeps its own sign-in.
- Icons recolored from the installed Claude logo, in seven named colors or any hex value.
- Windows: each profile gets its own taskbar button through AppUserModelID tagging.
- Claude in Chrome stays paired with the regular Claude (Chrome, Edge, Brave, Chromium) while
  profiles run; `doctor` shows the pairing per browser.
- Installers that download a versioned release and verify it against the SHA-256 written into them,
  with build provenance attestations, and offer to add a profile with a color picker.

[Unreleased]: https://github.com/thomasog/multi-claude/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/thomasog/multi-claude/releases/tag/v0.1.0
