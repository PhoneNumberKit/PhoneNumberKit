# PhoneNumberKit

![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/PhoneNumberKit/PhoneNumberKit/pr.yml?branch=main&label=tests&style=for-the-badge)

Swift framework for parsing, formatting and validating international phone numbers.
Inspired by Google's libphonenumber.

> **New home.** PhoneNumberKit is now maintained at
> [github.com/PhoneNumberKit/PhoneNumberKit](https://github.com/PhoneNumberKit/PhoneNumberKit).
> The original repository at `marmelroy/PhoneNumberKit` is frozen at `4.2.13`/`4.3.0` and no
> longer receives metadata updates. If you are upgrading, see [MIGRATION.md](MIGRATION.md).
>
> Looking for the `PhoneNumberTextField` and the country-code picker? Those UI components now
> live in a separate package: **[PhoneNumberKitUI](https://github.com/PhoneNumberKit/PhoneNumberKitUI)**.

## Features

|                  | Features                                                                                    |
| ---------------- | ------------------------------------------------------------------------------------------- |
| :phone:          | Validate, normalize and extract the elements of any phone number string.                    |
| :100:            | Simple Swift syntax and a lightweight readable codebase.                                     |
| :checkered_flag: | Fast. 1000 parses -> ~0.4 seconds.                                                          |
| :books:          | Best-in-class metadata from Google's libPhoneNumber project.                                |
| :trophy:         | Fully tested to match the accuracy of Google's libPhoneNumber.                              |
| :us:             | Convert country codes to country names and vice versa.                                       |

## Installation

PhoneNumberKit is distributed via the [Swift Package Manager](https://swift.org/package-manager/).

Add it to your `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/PhoneNumberKit/PhoneNumberKit", from: "5.0.0")
]
```

Or in Xcode: **File > Add Package Dependencies…** and enter
`https://github.com/PhoneNumberKit/PhoneNumberKit`.

## Usage

```swift
import PhoneNumberKit
```

All interactions happen through a `PhoneNumberUtility` object. It is relatively expensive to
allocate (it parses metadata into memory for its lifetime), so allocate it once and reuse it.

```swift
let phoneNumberUtility = PhoneNumberUtility()
```

Parse a string. The region is computed automatically but can be overridden. A hard type
validation runs by default; it can be skipped with `ignoreType` for performance.

```swift
do {
    let phoneNumber = try phoneNumberUtility.parse("+33 6 89 017383")
    let custom = try phoneNumberUtility.parse("+44 20 7031 3000", withRegion: "GB", ignoreType: true)
} catch {
    print("Generic parser error")
}
```

For large batches, use the array parser (invalid numbers are dropped):

```swift
let raw = ["0291 12345678", "+49 291 12345678", "04134 1234", "09123 12345"]
let numbers = phoneNumberUtility.parse(raw)
let numbersDE = phoneNumberUtility.parse(raw, withRegion: "DE", ignoreType: true)
```

`PhoneNumber` is an immutable struct:

```swift
phoneNumber.numberString
phoneNumber.countryCode
phoneNumber.nationalNumber
phoneNumber.numberExtension
phoneNumber.type // e.g. .mobile or .fixedLine
```

Formatting:

```swift
phoneNumberUtility.format(phoneNumber, toType: .e164)          // +61236618300
phoneNumberUtility.format(phoneNumber, toType: .international)  // +61 2 3661 8300
phoneNumberUtility.format(phoneNumber, toType: .national)      // (02) 3661 8300
```

Country/dialing-code queries:

```swift
phoneNumberUtility.countries(withCode: 33)
phoneNumberUtility.countryCode(for: "FR")
```

As-you-type formatting without UIKit:

```swift
PartialFormatter().formatPartial("+336895555") // +33 6 89 55 55
```

## UI components

`PhoneNumberTextField` and the customizable country-code picker now live in
**[PhoneNumberKitUI](https://github.com/PhoneNumberKit/PhoneNumberKitUI)** (iOS only).

## Acknowledgements

PhoneNumberKit was originally created by **Roy Marmelstein** and stewarded for years. It is
currently maintained by **Bruno Guidolim** and contributors under the PhoneNumberKit
organization. Metadata is sourced from Google's [libphonenumber](https://github.com/google/libphonenumber).

## License

MIT. See [LICENSE](LICENSE).
