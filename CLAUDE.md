# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PhoneNumberKit is a Swift framework for parsing, formatting, and validating international phone
numbers, inspired by Google's libphonenumber. Supports iOS, macOS, tvOS, watchOS, and Linux.

This is the **foundation** package (core parsing/formatting/validation + metadata). It is the
continuation of the original `marmelroy/PhoneNumberKit`, now maintained under the PhoneNumberKit
organization. UI components (`PhoneNumberTextField`, country-code picker) live in a **separate
package**: [PhoneNumberKitUI](https://github.com/PhoneNumberKit/PhoneNumberKitUI), which depends
on this one. Distribution is **Swift Package Manager only** (CocoaPods/Carthage were dropped in 5.0).

## Common Commands

This is a pure Swift package — use the `swift` toolchain only (no `.xcodeproj`, no `xcodebuild`).

```bash
# Test (works on Linux and macOS)
swift test

# Build
swift build

# Update metadata from Google's libphonenumber (requires Python 3.12+ and xmljson)
cd Sources/PhoneNumberKit/Resources && sh ./update_metadata.sh
```

## Architecture

Source lives under `Sources/PhoneNumberKit/`, tests under `Tests/PhoneNumberKitTests/`.

### Core Components
- **PhoneNumberUtility** (`Sources/PhoneNumberKit/PhoneNumberUtility.swift`) — main entry point for
  all operations. Expensive to allocate (parses metadata into memory); instantiate once and reuse.
- **PhoneNumber** (`Sources/PhoneNumberKit/PhoneNumber.swift`) — immutable, `Sendable` struct:
  numberString, countryCode, leadingZero, nationalNumber, numberExtension, type, regionID.
  Equatable, Hashable, Codable.
- **MetadataManager** (`Sources/PhoneNumberKit/MetadataManager.swift`) — loads metadata from the
  bundled JSON; O(1) lookup by country code / region ID; handles shared calling codes (e.g. NANPA).
- **PhoneNumberParser** (`Sources/PhoneNumberKit/PhoneNumberParser.swift`) — internal normalization,
  prefix/extension stripping, type determination.
- **ParseManager** (`Sources/PhoneNumberKit/ParseManager.swift`) — coordinates parser + metadata;
  single and batch parsing.

### Formatting
- **Formatter** (`Sources/PhoneNumberKit/Formatter.swift`) — `internal`. Formats to E164,
  international, national.
- **PartialFormatter** (`Sources/PhoneNumberKit/PartialFormatter.swift`) — `public` as-you-type
  formatter. Gated `#if canImport(ObjectiveC)` — **not available on Linux**. Consumed by
  PhoneNumberKitUI's text field, but lives here.

## Key Design Patterns

- **Metadata loading**: loaded once at `PhoneNumberUtility` init via a callback; default reads the
  bundled `PhoneNumberMetadata.json` from `Bundle.module`. Inject a custom source via `metadataCallback`.
- **Type validation**: parser matches metadata patterns per type; skip with `ignoreType` for
  performance. Hierarchy: specific types (mobile, pager…) → fixedOrMobile → unknown.
- **Region detection**: `CNContactsUserDefaults` country code, falling back to `Locale.current`;
  handles macCatalyst edge cases.
- **Performance**: batch array parsing; regex cached in RegexManager; O(1) metadata dictionaries.

## Platform / Packaging

- `swift-tools-version: 5.9`. Platforms: iOS 13, macOS 10.15, tvOS 13, watchOS 6 (+ Linux).
- Products: `PhoneNumberKit` (auto), `PhoneNumberKit-Static`, `PhoneNumberKit-Dynamic`.
- Target `PhoneNumberKit` at `Sources/PhoneNumberKit`; resources: `PhoneNumberMetadata.json`
  (processed), `PrivacyInfo.xcprivacy` (copied). `Resources/Original` (source XML) and the
  update script are excluded from the target.
- Public value types conform to `Sendable` (PhoneNumber, the Metadata* structs, enums in Constants).

## Metadata Updates

- Source: Google's libphonenumber. Script: `Sources/PhoneNumberKit/Resources/update_metadata.sh`
  (note its `cd ../../..` returns to repo root from the Resources dir — tied to the Sources layout).
- Workflow `.github/workflows/update_metadata.yml` runs every 12h (and on demand), opens a PR.
- `.github/workflows/release.yml` cuts a patch release (tag + GitHub release) when
  `.metadata-version` changes; the version is derived from the latest semver git tag (no podspec).

## CI

- `pr.yml` runs three parallel jobs: `swift test` on Linux, `swift test` on macOS, and an
  iOS compile guarantee (`xcodebuild -scheme PhoneNumberKit -destination generic/platform=iOS build`
  against `Package.swift` — `swift build` can't target iOS). macOS covers iOS's source paths (no
  iOS-only code); Linux differs via the `canImport(ObjectiveC)`/`!os(Linux)` guards, so it's tested
  separately.
- All CI shells use `set -o pipefail`.

## Important Notes

- Create `PhoneNumber` via `PhoneNumberUtility`, not directly (deprecated initializers fail).
- Reuse `PhoneNumberUtility` instances — they're expensive to allocate.
- `PartialFormatter` requires ObjectiveC (not on Linux).
- Use `ignoreType` and the array parsing method for performance-sensitive paths.
- UI work belongs in PhoneNumberKitUI, not here. Keep this package UI-free (no UIKit imports).
