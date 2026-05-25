# Changelog

## 5.0.0

First release under the PhoneNumberKit organization
(`github.com/PhoneNumberKit/PhoneNumberKit`). Continues the lineage from
`marmelroy/PhoneNumberKit` 4.2.13.

### Breaking
- **UI components removed.** `PhoneNumberTextField` and the country-code picker moved to a
  separate package, [PhoneNumberKitUI](https://github.com/PhoneNumberKit/PhoneNumberKitUI).
  See [MIGRATION.md](MIGRATION.md).
- **Raised platform minimums** to iOS 13 / macOS 10.15 / tvOS 13 / watchOS 6.
- The `CountryCodePicker` configuration type moved to `PhoneNumberKitUI`.

### Changed
- `swift-tools-version` bumped to 5.9.
- Added `Sendable` conformances to value types where applicable.
- Distribution is now Swift Package Manager only (CocoaPods and Carthage dropped).

### Notes
- Core parsing, formatting, and validation APIs are unchanged.
- Metadata continues to track Google's libphonenumber, updated automatically.
