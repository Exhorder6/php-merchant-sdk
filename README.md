# COINQVEST Merchant SDK (PHP)

Official COINQVEST Merchant API SDK for PHP by www.coinqvest.com

This SDK implements the REST API documented at https://www.coinqvest.com/en/api-docs

For SDKs in different languages, see https://www.coinqvest.com/en/api-docs#sdks

Read our Merchant API [development guide](https://www.coinqvest.com/en/blog/guide-mastering-cryptocurrency-checkouts-with-coinqvest-merchant-apis-321ac139ce15) and the examples below to help you get started.

Requirements
------------
* PHP >=5.3.0
* cURL extension for PHP
* OpenSSL extension for PHP

Installation as Drop-In
-----------------------
Copy the contents of `src` into the include path of your project.

**Usage Client**
```php
include('CQMerchantClient.class.php');

$client = new CQMerchantClient('YOUR-API-KEY', 'YOUR-API-SECRET', '/var/log/coinqvest.log');
```

Get your API key and secret here: https://www.coinqvest.com/en/api-settings

## Examples

**Creating a Customer** (https://www.coinqvest.com/en/api-docs#post-customer)

Creates a customer object, which can be associated with checkouts, payments and invoices. Checkouts associated with a customer generate more transaction details, help with your accounting and can automatically create invoices for your customer and yourself.

```php
$response = $client->post('/customer', array(
    'customer' => array(
        'email' => 'john@doe.com',
        'firstname' => 'John',
        'lastname' => 'Doe',
        'company' => 'ACME Inc.',
        'adr1' => '810 Beach St',
        'adr2' => 'Finance Department',
        'zip' => 'CA 94133',
        'city' => 'San Francisco',
        'countrycode' => 'US'
    )
));

if ($response->httpStatusCode == 200) {   
    $data = json_decode($response->responseBody, true);
    $customerId = $data['customerId']; // use this to associate a checkout with this customer
}
```

**Creating a Hosted Checkout** (https://www.coinqvest.com/en/api-docs#post-checkout-hosted)

Hosted checkouts are the simplest form of getting paid using the COINQVEST platform. 

Using this endpoint, your server submits a set of parameters, such as the payment details including optional tax items, customer information, and settlement currency. Then your server receives a checkout URL in return, which can then be displayed back to your customer. 

Upon visiting the URL, your customer is presented with a checkout page hosted on COINQVEST servers. This page displays all the information the customer needs to complete payment.

```php
$response = $client->post('/checkout/hosted', array(
    'charge' => array(
        'customerId' => $customerId, // associates this charge with a customer
        'currency' => 'USD', // specifies the billing currency
        'lineItems' => array( // a list of line items included in this charge
            array(
                'description' => 'T-Shirt',
                'netAmount' => 10, // denominated in the currency specified above
                'quantity' => 1
            )
        ),
        'discountItems' => array( // an optional list of discounts
            array(
                'description' => 'Loyalty Discount',
                'netAmount' => '0.5'
            )
        ),
        'shippingCostItems' => array( // any shipping costs?
            array(
                'description' => 'Shipping and Handling',
                'netAmount' => '3.99',
                'taxable' => false // sometimes shipping costs are taxable
            )
        ),
        'taxItems' => array( // any taxes?
            array(
                'name' => 'CA Sales Tax',
                'percent' => '0.0825' // 8.25% CA sales tax
            )
        )
    ),
    'settlementCurrency' => 'EUR' // specifies in which currency you want to settle 
));

if ($response->httpStatusCode == 200) {   
    $data = json_decode($response->responseBody, true);
    $checkoutId = $data['checkoutId']; // store this persistently in your database
    $url = $data['url']; // redirect your customer to this URL to complete the payment
}
```

**Monitoring Payment State** (https://www.coinqvest.com/en/api-docs#get-checkout)

Once the payment is captured we notify you via email, [WEBHOOK /payment](https://www.coinqvest.com/en/api-docs#webhook-payment), or you can poll [GET /checkout](https://www.coinqvest.com/en/api-docs#get-checkout) for payment status updates:

```php
$response = $client->get('/checkout', array('id' => $checkoutId));

if ($response->httpStatusCode == 200) {   
    $data = json_decode($response->responseBody, true);
    $state = $data['checkout']['state'];
    if (in_array($state, array('COMPLETED', 'DELAYED_COMPLETED', 'RESOLVED')) {
        echo "The payment has completed and your account was credited. You can now ship the goods."
    } else {
        // try again in 30 seconds or so...
    }
}
```

**Querying USD Wallet** (https://www.coinqvest.com/en/api-docs#get-wallet)
```php
$response = $client->get('/wallet', array('assetCode' => 'USD'));
```

**Querying all Wallets** (https://www.coinqvest.com/en/api-docs#get-wallets)
```php
$response = $client->get('/wallets');
```

**Updating a Customer** (https://www.coinqvest.com/en/api-docs#put-customer)
```php
$response = $client->post('/customer', array(
    'customer' => array(
        'id' => 'fd4f47a50c7f'
        'email' => 'new@email-address.com'
    )
));
```

**Deleting a Customer** (https://www.coinqvest.com/en/api-docs#delete-customer)
```php
$response = $client->delete('/customer', array(
    'customer' => array(
        'id' => 'fd4f47a50c7f'
    )
));
```

Please inspect https://www.coinqvest.com/en/api-docs for detailed API documentation or send us an email to service@coinqvest.com.

Support and Feedback
--------------------
Your feedback is appreciated! If you have specific problems or bugs with this SDK, please file an issue on Github. For general feedback and support requests, send an email to service@coinqvest.com.

Contributing
------------

1. Fork it ( https://github.com/COINQVEST/php-merchant-sdk/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
