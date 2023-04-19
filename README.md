# dLocal Direct SDK for Android

The dLocal Direct SDK allows easy integration with dLocal API to perform card tokenization and also 
includes card utility methods, like card detection, validation and formatting.

## Table of Contents

1. [Requirements](#requirements)
2. [Installation](#installation)
3. [Getting Started](#getting-started)
    - [Initialize the SDK](#initialize-the-sdk)
    - [Testing the integration](#testing-the-integration)
4. [Tokenize Card](#tokenize-card)
    - [Create a Card Token](#create-a-card-token)
    - [Get Bin Information](#get-bin-information)
    - [Create Installments Plan](#create-installments-plan)
5. [Brand Detection](#brand-detection)
    - [Browse cards brands](#browse-cards-brands)
    - [Detect brands from card number](#detect-brands-from-card-number)
    - [Card image for the detected brand](#card-image-for-the-detected-brand)
6. [Fields Validation](#fields-validation)
    - [Validate holder name](#validate-holder-name)
    - [Validate a card number](#validate-a-card-number)
    - [Validate expiration date](#validate-expiration-date)
    - [Validate security code](#validate-security-code)
7. [Input Formatting](#input-formatting)
    - [Format a card number](#format-a-card-number)
    - [Format card number with last numbers](#format-card-number-with-last-numbers)
    - [Format expiration date](#format-expiration-date)
    - [Format expiration month and year (separated fields)](#format-expiration-month-and-year-(separated-fields))
    - [Format security code](#format-security-code)
8. [Sample App](#sample-app)
9. [Report Issues](#report-issues)
10. [License](#license)

## Requirements

- Supports API versions from 23 (Android 6.0 Marshmallow) and higher.
- App permission `android.permission.INTERNET`

## Installation

New releases of the dLocal Direct SDK are published via [Maven Repository](https://mvnrepository.com/artifact/com.dlocal.android/dlocal-direct).  
The latest version is available via `mavenCentral()`.

Add `mavenCentral()` to the project level [build.gradle]() file's repositories section, if you don't have it already:

```groovy
repositories {
   mavenCentral()
   ...
}
```

Add dLocal Direct SDK dependency to the application's [build.gradle]() file:

```groovy
dependencies {
   ... 
   implementation 'com.dlocal.android:dlocal-direct:0.4.1' 
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

See the [SampleApplication]() for a detailed example.

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
    binNumber = "BIN-NUMBER", 
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
    binNumber = "BIN-NUMBER",
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

For those countries that we don't have data for, you can still use a global expert as follows:

```kotlin
import com.dlocal.direct.DLCardExpert

val cardExpert = DLCardExpert.global
val supportedBrands = cardExpert.allBrands.map { it.niceName }
println("Globally accepted brands: $supportedBrands")
```

This global expert contains globally accepted card brands like Visa and Mastercard among others. 

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
cardExpert.detectBrand(binNumber = "4242 42") // returns [Visa]

// Supports both formatted and raw card numbers
cardExpert.detectBrand(binNumber = "424242") // returns [Visa]

// Detect card brand from an incomplete card number
// In this case multiple card brands start with number "4", in order to find out which card is being entered user will need to enter additional numbers
cardExpert.detectBrand(binNumber = "4") // returns [Visa, Visa Débito, Maestro]

// Cards that are not supported in Uruguay will return an empty collection
// For example, American Express cards are not supported in Uruguay and "377400111111115" is a valid Amex card number
cardExpert.detectBrand(binNumber = "377400") // returns []

// Invalid values will return an empty collection
cardExpert.detectBrand(binNumber = "HELLO") // returns []

// An empty card number will return all brands supported in this Uruguay
cardExpert.detectBrand(binNumber = "") // returns [Visa, Oca, Mastercard, Diners, Lider, Visa Débito, Mastercard Débito, Maestro]
```

### Card image for the detected brand

You can access various image URLs for different sizes and resolutions using the `cardImage` method in the detected brand:

```kotlin
val brand = cardExpert.detectBrand(binNumber = "4242 4242 4242 4242")

// Image URL of size Small, XHDPI resolution and background
val xhdpiSmall = brand.cardImage(density = DLImageDensity.XHDPI, size = DLImageSize.SMALL, background = true)

// Image URL of size Small, HDPI resolution and no background
val hdpiSmall = brand.cardImage(density = DLImageDensity.HDPI, size = DLImageSize.SMALL, background = false)

// Image URL of size Medium, LDPI resolution and background
val ldpiMedium = brand.cardImage(density = DLImageDensity.LDPI, size = DLImageSize.MEDIUM, background = true)
```

## Fields Validation

### Validate holder name

```kotlin
cardExpert.validateName(holderName = "John Doe") // true
cardExpert.validateName(holderName = "") // false
cardExpert.validateName(holderName = " ") // false
cardExpert.validateName(holderName = "John Doe 2") // false
cardExpert.validateName(holderName = "John Doe!") // false
```


### Validate a card number

```kotlin
cardExpert.validateCardNumber(cardNumber = "") // false
cardExpert.validateCardNumber(cardNumber = "4") // false
cardExpert.validateCardNumber(cardNumber = "4242 4242 4242 4242") // true
cardExpert.validateCardNumber(cardNumber = "4242424242424242") // true

cardExpert.validateCardNumber(cardNumber = "4242424242424241") // false (fails luhn check)
cardExpert.validateCardNumber(cardNumber = "4A") // false
```

### Validate expiration date

```kotlin
cardExpert.validateExpirationDate(expirationDate = "") // false
cardExpert.validateExpirationDate(expirationDate = "0") // false
cardExpert.validateExpirationDate(expirationDate = "02") // false
cardExpert.validateExpirationDate(expirationDate = "02/") // false
cardExpert.validateExpirationDate(expirationDate = "02/2") // false
cardExpert.validateExpirationDate(expirationDate = "02/26") // true
cardExpert.validateExpirationDate(expirationDate = "2/26") // true

cardExpert.validateExpirationDate(expirationDate = "13") // false
cardExpert.validateExpirationDate(expirationDate = "7/22") // false (expired)
cardExpert.validateExpirationDate(expirationDate = "8/2022") // false
```

### Validate security code

The following will validate the security code using standard security code rules:

```kotlin
val brands = cardExpert.detectBrand(binNumber = "4242 42") // returns [Visa]

cardExpert.validateSecurityCode(securityCode = "", brands = brands) // false
cardExpert.validateSecurityCode(securityCode = "1", brands = brands) // false
cardExpert.validateSecurityCode(securityCode = "12", brands = brands) // false
cardExpert.validateSecurityCode(securityCode = "123", brands = brands) // true

cardExpert.validateSecurityCode(securityCode = "1234", brands = brands) // false, security code for Visa cards are always of length three
cardExpert.validateSecurityCode(securityCode = "12A", brands = brands) // false, security codes for Visa cards can only contain numbers
```

If you want for instance to allow input of security codes for any card that is valid in Uruguay:

```kotlin
val uruguayBrands = cardExpert.allBrands

cardExpert.validateSecurityCode(securityCode = "123", brands = uruguayBrands) // true
cardExpert.validateSecurityCode(securityCode = "1234", brands = uruguayBrands) // false as there is no supported card in Uruguay that allows security code length of four digits
```

If your brands list is empty or null, the validation will use the default Brand which has a security code of only digits with minimum length of 3 and maximum of 4.
```kotlin
cardExpert.validateSecurityCode(securityCode = "123", brands = null) // true
cardExpert.validateSecurityCode(securityCode = "12", brands = null) // false, minimum length is 3
cardExpert.validateSecurityCode(securityCode = "1234", brands = emptyList()) // true
cardExpert.validateSecurityCode(securityCode = "12345", brands = emptyList()) // false, maximum length is 4
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
cardExpert.formatExpirationDate(date = "02") // returns "02/" (automatically formats into month "02" and inserts "/")
cardExpert.formatExpirationDate(date = "45") // returns "04/5" (formats "4" into "04")
cardExpert.formatExpirationDate(date = "2/A") // returns "2" (disallows entering any non numeric character)
cardExpert.formatExpirationDate(date = "22") // returns "02/2"
cardExpert.formatExpirationDate(date = "220") // returns "02/20"
cardExpert.formatExpirationDate(date = "2205") // returns "02/20"
cardExpert.formatExpirationDate(date = "1222") // returns "12/22"
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
cardExpert.formatExpirationMonth(month = "13") // returns "01" (this is an invalid month so we take only the first digit)
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

If you pass `null` brand, the formatter will use the default Brand which has a security code of only digits with minimum length of 3 and maximum of 4.

```kotlin
cardExpert.formatSecurityCode(code = "1A234B56", brand = null) // returns "1234"
cardExpert.formatSecurityCode(code = "00 1 3", brand = null) // returns "0013"
```

## Sample App

In this repository there's a [sample app]() to showcase how to use the SDK, please refer to the code for more detailed examples.

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