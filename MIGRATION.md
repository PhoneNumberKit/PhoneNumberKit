# Migrating to PhoneNumberKit 5.0

PhoneNumberKit 5.0 moves to a new home and splits the UI components into a separate package.
The core parsing/formatting API is unchanged.

## 1. Change the package source

The library now lives at `github.com/PhoneNumberKit/PhoneNumberKit`.

```diff
- .package(url: "https://github.com/marmelroy/PhoneNumberKit", from: "4.0.0")
+ .package(url: "https://github.com/PhoneNumberKit/PhoneNumberKit", from: "5.0.0")
```

In Xcode: remove the old `marmelroy/PhoneNumberKit` package dependency and add
`https://github.com/PhoneNumberKit/PhoneNumberKit`.

If you used CocoaPods or Carthage: those transports are no longer published. Migrate to SPM.
The last CocoaPods release is `4.3.0` from the original repository (deprecated, frozen).

## 2. If you use the UI components

`PhoneNumberTextField` and the country-code picker moved to a new package,
**[PhoneNumberKitUI](https://github.com/PhoneNumberKit/PhoneNumberKitUI)**.

```diff
+ .package(url: "https://github.com/PhoneNumberKit/PhoneNumberKitUI", from: "1.0.0")
```

```diff
- import PhoneNumberKit
+ import PhoneNumberKit
+ import PhoneNumberKitUI   // for PhoneNumberTextField, CountryCodePicker, CountryCodePickerOptions
```

### `CountryCodePicker` configuration moved modules

The `CountryCodePicker` configuration type (`commonCountryCodes`, `forceModalPresentation`,
`alwaysShowsSearchBar`) now lives in `PhoneNumberKitUI`.

- Unqualified usage still works once you `import PhoneNumberKitUI`:

  ```swift
  CountryCodePicker.commonCountryCodes = ["US", "GB"]
  ```

- Module-qualified usage must change:

  ```diff
  - PhoneNumberKit.CountryCodePicker.commonCountryCodes = ["US", "GB"]
  + CountryCodePicker.commonCountryCodes = ["US", "GB"]
  ```

## 3. Platform minimums

The deployment floor was raised. If you target below these, stay on 4.x:

| Platform | Minimum |
|----------|---------|
| iOS      | 13.0    |
| macOS    | 10.15   |
| tvOS     | 13.0    |
| watchOS  | 6.0     |

## 4. No other source changes

`PhoneNumberUtility`, `PhoneNumber`, `PartialFormatter`, parsing, formatting, and validation
keep the same API. A straight URL swap is enough for core-only consumers.
