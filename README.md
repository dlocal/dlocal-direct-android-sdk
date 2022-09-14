# dLocal Direct SDK for Android

The dLocal Direct SDK allows easy integration with dLocal API to perform card tokenization and also 
includes card utility methods, like card detection, validation and formatting.

## Table of Contents

1. [ Requirements ](#markdown-header-requirements)
2. [ Installation ](#markdown-header-installation)
3. [ Getting Started ](#markdown-header-getting-started)
    - [ Initialize the SDK ](#markdown-header-initialize-the-sdk)
    - [ Testing the integration ](#markdown-header-testing-the-integration)
4. [ Tokenize Card ](#markdown-header-tokenize-card)
    - [ Create a Card Token ](#markdown-header-create-a-card-token)
    - [ Get Bin Information ](#markdown-header-get-bin-information)
    - [ Create Installments Plan ](#markdown-header-sample-app)
5. [ Brand Detection ](#markdown-header-brand-detection)
    - [ Browse cards brands ](#markdown-header-browse-cards-brands)
    - [ Detect brands from card number ](#markdown-header-detect-brands-from-card-number)
6. [ Fields Validation ](#markdown-header-fields-validation)
    - [ Possible results ](#markdown-header-possible-results)
    - [ Validate a card number ](#markdown-header-validate-a-card-number)
    - [ Validate expiration date ](#markdown-header-validate-expiration-date)
    - [ Validate security code ](#markdown-header--alidate-security-code)
7. [ Input Formatting ](#markdown-header-input-formatting)
    - [ Format a card number ](#markdown-header-format-a-card-number)
    - [ Format card number with last numbers ](#markdown-header-format-card-number-with-last-numbers)
    - [ Format expiration date ](#markdown-header-format-expiration-date)
    - [ Format expiration month and year (separated fields) ](#markdown-header-format-expiration-month-and-year-(separated-fields))
    - [ Format security code ](#markdown-header-format-security-code)
8. [ Sample App ](#markdown-header-sample-app)
9. [ Report Issues ](#markdown-header-report-issues)
10. [ License ](#markdown-header-license)

## Requirements

- Supports API versions from 23 (Android 6.0 Marshmallow) and higher.
- App permission `android.permission.INTERNET`

## Installation

New releases of the dLocal Direct SDK are published via [Maven Repository](https://mvnrepository.com/artifact/com.dlocal.android/dlocal-direct).  
The latest version is available via `mavenCentral()`.

Add `mavenCentral()` to the project level [build.gradle](https://bitbucket.org/dlocal-public/dlocal-direct-android-sdk/src/master/build.gradle#lines-5) file's repositories section, if you don't have it already:

```groovy
repositories {
   mavenCentral()
   ...
}
```

Add dLocal Direct SDK dependency to the application's [build.gradle](https://bitbucket.org/dlocal-public/dlocal-direct-android-sdk/src/master/app/build.gradle#lines-38) file:

```groovy
dependencies {
   ... 
   implementation 'com.dlocal.android:dlocal-direct:0.0.1' 
   ...
}    
```  

## Getting started

### Initialize the SDK

Create an instance of dLocal Direct passing your API key and the two letter [ISO 3166](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country code:

```kotlin  
import com.dlocal.direct.DLCardTokenizer  
  
val dlocal = DLCardTokenizer(apiKey = "API_KEY", countryCode = "US", testMode = true)  
```  

> NOTE: To initialize the dLocal Direct instance you will need an API Key. Please contact your Technical Account Manager to obtain one.

### Testing the integration

We strongly **recommend that you use the `SANDBOX` environment when testing**, and only use `PRODUCTION` in production ready builds.

You can specify to use a different environment with the `testMode` param, which is false by default, i.e:

```kotlin
import com.dlocal.direct.DLCardTokenizer  

val dlocal = if (BuildConfig.DEBUG) {
    DLCardTokenizer(apiKey = "SBX API KEY", country = "US", testMode = true)
} else {
	DLCardTokenizer(apiKey = "PROD API KEY", country = "US", testMode = false)
}
```

Replacing the `apiKey` with yours for each environment.

See the [SampleApplication](https://bitbucket.org/dlocal-public/dlocal-direct-android-sdk/src/master/app/src/main/java/com/dlocal/sampleapp/SampleApplication.kt) for a detailed example.

## Tokenize Card

### Create a Card Token

```kotlin
import com.dlocal.direct.DLCardData
import com.dlocal.direct.DLCardTokenizer

val cardData = DLCardData(
    holderName = "HOLDER-NAME",
    cardNumber = "CARD-NUMBER",
    cvv = "CVV",
    expirationMonth = 12,
    expirationYear = 2025
)

dlocal.tokenizeCard(cardData, onSuccess = { token ->
    println("Successfully tokenized card: ${token.value}")
}, onError = { error ->
    println(error.debugMessage)
})
```

### Get Bin Information

```kotlin
import com.dlocal.direct.DLCardTokenizer

dlocal.getBinInformation(
    cardNumber = "CARD-NUMBER", 
    onSuccess = { binInfo ->
        println("Successfully obtained bin information: ${binInfo.bin}")
    }, onError = { error ->
        println("Failed to obtain bin information: ${error.debugMessage}") 
    }
)
```  

### Create Installments Plan

```kotlin
import com.dlocal.direct.DLCardTokenizer

dlocal.createInstallmentsPlan(
    cardNumber = "CARD-NUMBER",
    currencyCode = "CURRENCY-CODE",
    amount = 500.0,
    onSuccess = { installments ->
        println("Successfully created installments plan: ${installments.id}")
    },
    onError = { error ->
        println("Failed to create installments plan: ${error.debugMessage}")
    }
)
```

## Brand Detection

All interactions are done through one instance of the `DLCardExpert` class which you can create as follows:

```kotlin
import com.dlocal.direct.DLCardExpert

val cardExpert = DLCardExpert(countryCode = "UY")
```

Card data is specific for the country defined at the initialization step of the sdk instance. In below examples we will assume that the sdk was initialized with country code for Uruguay ("UY").

### Browse cards brands

```kotlin
cardExpert.allBrands // Use this to access the list of all card brands supported in the country

cardExpert.brand(withIdentifier = DLCardBrandIdentifier.VISA)?.let { visa ->
    // Visa is supported in Uruguay
    println(visa.identifier) // "visa"
    println(visa.niceName) // "Visa"
    println(visa.length) // [16]
    println(visa.image) // https://static.dlocal.com/fields/input-icons/visa.svg
} ?: run {
    println("Visa is not supported in Uruguay")
}
```

### Detect brands from card number

```kotlin
val cardExpert = DLCardExpert(countryCode = "UY")

// Detect card brand from a complete card number
cardExpert.detectBrand(cardNumber = "4242 4242 4242 4242") // returns [Visa]

// Supports both formatted and raw card numbers
cardExpert.detectBrand(cardNumber = "4242424242424242") // returns [Visa]

// Detect card brand from an incomplete card number
// In this case multiple card brands start with number "4", in order to find out which card is being entered user will need to enter additional numbers
cardExpert.detectBrand(cardNumber = "4") // returns [Visa, Visa Débito, Maestro]

// Cards that are not supported in Uruguay will return an empty collection
// For example, American Express cards are not supported in Uruguay and "377400111111115" is a valid Amex card number
cardExpert.detectBrand(cardNumber = "377400111111115") // returns []

// Invalid values will return an empty collection
cardExpert.detectBrand(cardNumber = "HELLO") // returns []

// An empty card number will return all brands supported in this Uruguay
cardExpert.detectBrand(cardNumber = "") // returns [Visa, Oca, Mastercard, Diners, Lider, Visa Débito, Mastercard Débito, Maestro]
```

## Fields Validation

### Possible results

Validation of card fields can result in one of the following `DLValidationResult` type:

- `PotentiallyValid` meaning that the value entered is potentially valid but not yet valid, this means that by adding additional characters it could become valid
- `Valid` meaning that the value entered is valid
- `Invalid` meaning that the value entered is invalid and will never be valid by adding additional characters to it

### Validate a card number

```kotlin
cardExpert.validateCardNumber(cardNumber = "") // returns PotentiallyValid
cardExpert.validateCardNumber(cardNumber = "4") // returns PotentiallyValid
cardExpert.validateCardNumber(cardNumber = "4242 4242 4242 4242") // returns Valid
cardExpert.validateCardNumber(cardNumber = "4242424242424242") // returns Valid

cardExpert.validateCardNumber(cardNumber = "4242424242424241") // returns Invalid (fails luhn check)
cardExpert.validateCardNumber(cardNumber = "4A") // returns Invalid
```

### Validate expiration date

```kotlin
cardExpert.validateExpirationDate(expirationDate = "") // returns PotentiallyValid
cardExpert.validateExpirationDate(expirationDate = "0") // returns PotentiallyValid
cardExpert.validateExpirationDate(expirationDate = "02") // returns PotentiallyValid
cardExpert.validateExpirationDate(expirationDate = "02/") // returns PotentiallyValid
cardExpert.validateExpirationDate(expirationDate = "02/2") // returns PotentiallyValid
cardExpert.validateExpirationDate(expirationDate = "02/26") // returns Valid
cardExpert.validateExpirationDate(expirationDate = "2/26") // returns Valid

cardExpert.validateExpirationDate(expirationDate = "13") // returns Invalid
cardExpert.validateExpirationDate(expirationDate = "7/22") // returns Invalid (expired)
cardExpert.validateExpirationDate(expirationDate = "8/2022") // returns Invalid
```

### Validate security code

Different card brands have different rules for validating security codes, this is why this function will ask you to pass a card brand as parameter.

```kotlin
val brands = cardExpert.detectBrand(cardNumber = "4242 4242 4242 4242") // returns [Visa]

cardExpert.validateSecurityCode(securityCode = "", brands = brands) // returns PotentiallyValid
cardExpert.validateSecurityCode(securityCode = "1", brands = brands) // returns PotentiallyValid
cardExpert.validateSecurityCode(securityCode = "12", brands = brands) // returns PotentiallyValid
cardExpert.validateSecurityCode(securityCode = "123", brands = brands) // returns Valid

cardExpert.validateSecurityCode(securityCode = "1234", brands = brands) // returns Invalid, security code for Visa cards are always of length three
cardExpert.validateSecurityCode(securityCode = "12A", brands = brands) // returns Invalid, security codes for Visa cards can only contain numbers
```

If you want for instance to allow input of security codes for any card that is valid in Uruguay:

```kotlin
val uruguayBrands = cardExpert.allBrands

cardExpert.validateSecurityCode(securityCode = "123", brands = uruguayBrands) // returns Valid
cardExpert.validateSecurityCode(securityCode = "1234", brands = uruguayBrands) // returns Invalid as there is no supported card in Uruguay that allows security code length of four digits
```

## Input Formatting

### Format a card number

Use this when users are typing in to the card field to facilitate the input of the card.

```kotlin
cardExpert.formatCardNumber("") // returns ""
cardExpert.formatCardNumber("4") // returns "4"
cardExpert.formatCardNumber("42") // returns "42"
cardExpert.formatCardNumber("4242") // returns "4242" (Visa detected at this point)
cardExpert.formatCardNumber("42424") // returns "4242 4"
cardExpert.formatCardNumber("424242") // returns "4242 42"
cardExpert.formatCardNumber("4242424242424242") // returns "4242 4242 4242 4242"
cardExpert.formatCardNumber("42424242424242425") // returns "42424242424242425" (notice that extra "5" entered which makes input match no card brand)
cardExpert.formatCardNumber("42 42424242424242") // returns "4242 4242 4242 4242"
cardExpert.formatCardNumber("42 4242  42 42 4 2424    2") // returns "4242 4242 4242 4242"
```

If you want to use a specific brand to format the number, you can pass that brand to the function as follows:

```kotlin
val diners = cardExpert.brand(withIdentifier = DLCardBrandIdentifier.DINERS_CLUB)
cardExpert.format(cardNumber = "12345678901234", brand = diners) // returns "1234 567890 1234"
```

### Format card number with last numbers

If you want to display a card identified by last numbers, you can use this option as follows:

```kotlin
val diners = cardExpert.brand(withIdentifier = DLCardBrandIdentifier.DINERS_CLUB)

cardExpert.formatCardEndingWith(ending = "1234", brand = diners) // returns "**** ****** 1234"
cardExpert.formatCardEndingWith(ending = "234", brand = diners) // returns "**** ****** *234"
cardExpert.formatCardEndingWith(ending = "34", brand = diners) // returns "**** ****** **34"
cardExpert.formatCardEndingWith(ending = "4", brand = diners) // returns "**** ****** ***4"
cardExpert.formatCardEndingWith(ending = "", brand = diners) // returns "**** ****** ****"
```

### Format expiration date

If your form contains a single field for expiration date then this is the function you want to use to format the input. Formatting includes normalization to help users with input of the expiration date, like automatically inserting a "/" to separate month and year.

```kotlin
cardExpert.formatExpirationDate(date = "") // returns ""
cardExpert.formatExpirationDate(date = "1") // returns "1" (expecting a second number to understand whether this is month "01" or "11" or "12")
cardExpert.formatExpirationDate(date = "11") // returns "11/" (automatically inserts "/" character so user only has to enter numbers)
cardExpert.formatExpirationDate(date = "2") // returns "02/" (automatically formats into month "02" and inserts "/")
cardExpert.formatExpirationDate(date = "2/") // returns "02/" (formats "2" into "02")
cardExpert.formatExpirationDate(date = "2/A") // returns "02/" (disallows entering any non numeric character)
cardExpert.formatExpirationDate(date = "2/2") // returns "02/2"
cardExpert.formatExpirationDate(date = "2/20") // returns "02/20"
cardExpert.formatExpirationDate(date = "2/202") // returns "02/202"
cardExpert.formatExpirationDate(date = "2/2022") // returns "02/22" (shortens year)
```

### Format expiration month and year (separated fields)

If your form has separated fields for month and year, you will want to use these methods for formatting the input. Formatting includes normalization of values to match card formatting rules (e.g. turning "2" into "02") and some validation (e.g. disallow input of non numeric characters, disallow input of more than two characters for months, etc).

```kotlin
cardExpert.formatExpirationMonth(month = "") // returns ""
cardExpert.formatExpirationMonth(month = "0") // returns "0"
cardExpert.formatExpirationMonth(month = "1") // returns "1" (as this could be "01", "11" or "12")
cardExpert.formatExpirationMonth(month = "2") // returns "02"
cardExpert.formatExpirationMonth(month = "3") // returns "03"
cardExpert.formatExpirationMonth(month = "4") // returns "04"
cardExpert.formatExpirationMonth(month = "5") // returns "05"
cardExpert.formatExpirationMonth(month = "6") // returns "06"
cardExpert.formatExpirationMonth(month = "7") // returns "07"
cardExpert.formatExpirationMonth(month = "8") // returns "08"
cardExpert.formatExpirationMonth(month = "9") // returns "09"
cardExpert.formatExpirationMonth(month = "10") // returns "10"
cardExpert.formatExpirationMonth(month = "11") // returns "11"
cardExpert.formatExpirationMonth(month = "12") // returns "12"

cardExpert.formatExpirationMonth(month = "121") // returns "12" (disallows entering a third character)
cardExpert.formatExpirationMonth(month = "13") // returns "13" (this is an invalid month so we cannot format it)
cardExpert.formatExpirationMonth(month = "1A") // returns "1" (does not allow entering non numeric characters)
cardExpert.formatExpirationMonth(month = "1 ") // returns "1" (does not allow spaces)
```

```kotlin
cardExpert.formatExpirationYear(year = "") // returns ""
cardExpert.formatExpirationYear(year = "0") // returns "" (does not allow first character to be a zero)
cardExpert.formatExpirationYear(year = "1") // returns "" (does not allow first character to be a one)
cardExpert.formatExpirationYear(year = "2") // returns "2"
cardExpert.formatExpirationYear(year = "20") // returns "20"
cardExpert.formatExpirationYear(year = "202") // returns "202" (allowed as user seems to be constructing four character year)
cardExpert.formatExpirationYear(year = "2022") // returns "22" (shortens length to match standard card expiration year formatting)
cardExpert.formatExpirationYear(year = "20225") // returns "22" (ignores everything coming after the fifth character)
cardExpert.formatExpirationYear(year = "202A") // returns "202" (does not allow entering non numeric characters)
```

### Format security code

Different card brands have different rules for security code validation. This is why we ask you to input the brand for which you are formatting the code.

```kotlin
val diners = cardExpert.brand(withIdentifier = DLCardBrandIdentifier.DINERS_CLUB)
        
cardExpert.formatSecurityCode(code = "", brand = diners) // returns ""
cardExpert.formatSecurityCode(code = "1", brand = diners) // returns "1"
cardExpert.formatSecurityCode(code = "12", brand = diners) // returns "12"
cardExpert.formatSecurityCode(code = "123", brand = diners) // returns "123"
cardExpert.formatSecurityCode(code = "1234", brand = diners) // returns "123" (security code limited to 3 characters for this brand)
cardExpert.formatSecurityCode(code = "12 ", brand = diners) // returns "12" (spaces are removed)
cardExpert.formatSecurityCode(code = "12A", brand = diners) // returns "12" (non numeric characters are removed)
```

If you don't yet know the brand you can instead use a `DLCardCodeFormatter` as follows:

```kotlin
val securityCodeFormatter = DLCardCodeFormatter(sizes = listOf(3, 4)) // Allow security codes of three or four numbers
cardExpert.formatSecurityCode(code = "1234", formatter = securityCodeFormatter) // returns "1234"
```

## Sample App

In this repository there's a [sample app](https://bitbucket.org/dlocal-public/dlocal-direct-android-sdk/src/master/app/) to showcase how to use the SDK, please refer to the code for more detailed examples.

## Report Issues

If you have a problem or find an issue with the SDK please contact us at [mobile-dev@dlocal.com](mailto:mobile-dev@dlocal.com).

## License

```text
    MIT License

    Copyright (c) 2022 DLOCAL

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
```