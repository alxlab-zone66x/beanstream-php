Bambora PHP API
==================

Composer ready PHP wrapper for [Bambora NA Payment API](https://dev.na.bambora.com/docs/references/).

This fork contains fixes for PHP 8 and up.

## Installation

The recommended way to install the library is using [Composer](https://getcomposer.org).

1) Add this json to your composer.json file:
```json
{
    "require": {
        "beanstream/beanstream": "dev-master"
    }
}
```

2) Next, run this from the command line:
```
composer install
```
3) Finally, add this line to your php file that will be using the SDK:
```
require 'vendor/autoload.php';
```

## Handling Exceptions

If the server returns an unexpected response or error, PHP API throws *\Beanstream\Exception*.

Positive error codes correspond to Beanstream API errors, see
[Bambora NA Payment API](https://dev.na.bambora.com/docs/references/)

Negative codes correspond to [cURL errors](http://curl.haxx.se/libcurl/c/libcurl-errors.html)
(original cURL error codes are positive, in *\Beanstream\Exception* those are just reversed).
Exception with zero error code are PHP API specific, e.g. *The curl extension is required* or
*Unexpected response format*.

Generally, any unsuccessful request, e.g. insufficient data or declined transaction, results in *\Beanstream\Exception*,
thus *try..catch* is recommended for intercepting and handling them, see example below.

## Your First Integration


```php
<?php

require 'vendor/autoload.php';

$merchant_id = ''; //INSERT MERCHANT ID (must be a 9 digit string)
$api_key = ''; //INSERT API ACCESS PASSCODE
$api_version = 'v1'; //default
$platform = 'api'; //default

//Create Beanstream Gateway
$beanstream = new \Beanstream\Gateway($merchant_id, $api_key, $platform, $api_version);

//Example Card Payment Data
$payment_data = array(
        'order_number' => 'a1b2c3',
        'amount' => 1.00,
        'payment_method' => 'card',
        'card' => array(
            'name' => 'Mr. Card Testerson',
            'number' => '4030000010001234',
            'expiry_month' => '07',
            'expiry_year' => '22',
            'cvd' => '123'
        )
);
$complete = TRUE; //set to FALSE for PA

//Try to submit a Card Payment
try {

	$result = $beanstream->payments()->makeCardPayment($payment_data, $complete);
    
    /*
     * Handle successful transaction, payment method returns
     * transaction details as result, so $result contains that data
     * in the form of associative array.
     */
} catch (\Beanstream\Exception $e) {
    /*
     * Handle transaction error, $e->code can be checked for a
     * specific error, e.g. 211 corresponds to transaction being
     * DECLINED, 314 - to missing or invalid payment information
     * etc.
     */
}
```

See examples.php for more examples.

## Tips

### Authentication

Beanstream defines separate API access passcodes for payment, profile and reporting API requests. It is possible 
to use the same value for all of them, and create a single instance of \Beanstream\Gateway. You can also 
initialize separate \Beanstream\Gateway instances for each type of request.  API access passcodes are 
configured via the Beanstream dashboard (See *administration -> account settings -> order settings* 
for payment and search passcodes, *configuration -> payment profile configuration* for profile passcode).


### Billing Address Province

Beanstream requires the *province* field submitted along with *billing* data to be a two-letter code. It only requires it when
the specified *country* is *US* or *CA*, for other country codes set it to *--* (two dashes) even if the corresponding country 
does have states or provinces.


### Backwards Compatibility
The default `$platform` value assigned is `'api'`, which sends your requests to the endpoint `'api.na.bambora.com'`. The ensure backwards compatibility, you may assign `$platform = 'www'` which will send your requests to the legacy endpoint `'www.beanstream.com/api'`.


### Ensuring TLS 1.2-Only Compatibility
If you would like to test compatibility with the LIVE TLS 1.2-only environment, assign `$platform = 'tls12-api'`, instead of `'api'`; This will point your requests to the endpoint `'tls12-api.na.bambora.com'`. Please be advised that this endpoint is provided for a limited time, and is intended for integration compatibility testing only, and is not intended for any type of load tests. More details can be found on the [Bambora North America Knowledge Base](https://help.na.bambora.com/hc/en-us/articles/115015460087-TLS-Upgrade-TLS-1-2).


