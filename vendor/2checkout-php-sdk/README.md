2Checkout PHP SDK
=====================

This is the current 2Checkout PHP SDK providing developers with a simple set of bindings to the 2Checkout 6.0 REST API, IPN and Convert Plus Signature API.

To use, clone and require with composer or require the included autoloader.

```php
require_once('/path/to/2checkout-php-library/autoloader.php');
use Tco\TwocheckoutFacade;
```

All the features present in this library are used through TwocheckoutFacade instance which can be initialized with a configuration array.

```php
$config    = array(
            'sellerId'      => YOUR_MERCHANT_CODE, // REQUIRED
            'secretKey'     => YOUR_SECRET_KEY, // REQUIRED
            'buyLinkSecretWord'    => YOUR_SECRET_WORD,
            'jwtExpireTime' => 30,
            'curlVerifySsl' => 1
);

$tco = new TwocheckoutFacade($config);
```
##### - > All the required params in the $config can be found in [CPanel](https://secure.2checkout.com/cpanel/login.php) in Integrations -> Webhooks & API section

## Using the Api Core directly
Call any 2Checkout REST 6.0 endpoint like below:
```php
$result = $tco->apiCore()->call( '/orders/', $params, 'POST' );
```

## Place order helper
You can also use the provided Order helper object.

###### Dynamic Product:
```php
$dynamicOrderParams = array(
            'Country'           => 'US',
            'Currency'          => 'USD',
            'CustomerIP'        => '91.220.121.21',
            'ExternalReference' => 'CustOrd101',
            'Language'          => 'en',
            'Source'            => 'tcolib.local',
            'BillingDetails'    =>
                array(
                    'Address1'    => 'Street 1',
                    'City'        => 'Cleveland',
                    'State'       => 'Ohio',
                    'CountryCode' => 'US',
                    'Email'       => 'testcustomer@2Checkout.com',
                    'FirstName'   => 'John',
                    'LastName'    => 'Doe',
                    'Zip'         => '20034',
                ),
            'Items'             =>
                array(
                    0 =>
                        array(
                            'Name'         => 'Dynamic product2',
                            'Description'  => 'Product 2',
                            'Quantity'     => 3, //units
                            'IsDynamic'    => true,
                            'Tangible'     => false,
                            'PurchaseType' => 'PRODUCT',
                            'Price'        =>
                                array(
                                    'Amount' => 6, //value
                                    'Type'   => 'CUSTOM',
                                ),
                        ),
                    1 =>
                        array(
                            'Name'         => 'Dynamic product',
                            'Description'  => 'Test description',
                            'Quantity'     => 4,
                            'IsDynamic'    => true,
                            'Tangible'     => false,
                            'PurchaseType' => 'PRODUCT',
                            'Price'        =>
                                array(
                                    'Amount' => 1,
                                    'Type'   => 'CUSTOM',
                                )
                        ),
                ),
            'PaymentDetails'    =>
                array(
                    'Type'          => 'TEST', //'TEST' or 'EES_TOKEN_PAYMENT'
                    'Currency'      => 'USD',
                    'CustomerIP'    => '91.220.121.21',
                    'PaymentMethod' =>
                        array(
                            'RecurringEnabled' => false,
                            'HolderNameTime'   => 1,
                            'CardNumberTime'   => 1,
                        ),
                ),
        );
```
- We can get [Order](src/Tco/Source/Api/Rest/V6/Order.php) object by calling TwocheckoutFacade "order()" method:
```php
$order = $tco->order();
```
 - Order object has a method called "place" that generates a call to Api Core and returns a response: 
```php
$response = $order->place( $dynamicOrderParams );
```
 - To validate the response you first need to check for "refno" key in response and then do a validation API call for 
that "refno". Example:
```php
$orderData = $tco->order()->getOrder( array( 'RefNo' => $response['refNo'] ) );
```
Then validate "$orderData['Status']". For successfully placed payments this value is "AUTHRECEIVED" or "COMPLETE".

####2. Catalog Product
```php
$orderCatalogProductParams = array(
    'Country'           => 'br',
    'Currency'          => 'brl',
    'CustomerIP'        => '91.220.121.21',
    'ExternalReference' => 'CustOrderCatProd100',
    'Language'          => 'en',
    'BillingDetails'    =>
        array(
            'Address1'    => 'Test Address',
            'City'        => 'LA',
            'CountryCode' => 'BR',
            'Email'       => 'customer@2Checkout.com',
            'FirstName'   => 'Customer',
            'FiscalCode'  => '056.027.963-98',
            'LastName'    => '2Checkout',
            'Phone'       => '556133127400',
            'State'       => 'DF',
            'Zip'         => '70403-900',
        ),
    'Items'             =>
        array(
            0 =>
                array(
                    'Code'     => 'E377076E6A_COPY1', //Get the code from CPANEL at Setup->Products->Code 
                    column
                    'Quantity' => '1',
                ),
        ),
    'PaymentDetails'    =>
        array(
            'Type'          => 'TEST',
            'Currency'      => 'USD',
            'CustomerIP'    => '91.220.121.21',
            'PaymentMethod' =>
                array(
                    'CCID'               => '123',
                    'CardNumber'         => '4111111111111111',
                    'CardNumberTime'     => '12',
                    'CardType'           => 'VISA',
                    'ExpirationMonth'    => '12',
                    'ExpirationYear'     => '2023',
                    'HolderName'         => 'John Doe',
                    'HolderNameTime'     => '12',
                    'RecurringEnabled'   => true,
                    'Vendor3DSReturnURL' => 'www.test.com',
                    'Vendor3DSCancelURL' => 'www.test.com',
                ),
        ),
);
```
The rest of the flow is identical as for placing orders with dynamic products.

## Subscriptions helper
- For a payment to be recurring (generate a subscription) one needs to add to "item" (product) array structure the "RecurringOptions" field set.

###### An example of how this option can be configured:
```php
'RecurringOptions' =>
    array(
        'CycleLength'    => 1,
        'CycleUnit'      => 'MONTH',
        'CycleAmount'    => 3,
        'ContractLength' => 3,
        'ContractUnit'   => 'Year',
    ),
```

###### An example of an item having recurring option:
```php
'Items' =>
        array(
            0 =>
                array(
                    'Name'         => 'Dynamic product',
                    'Description'  => 'Product Description test',
                    'Quantity'     => 3, //units
                    'IsDynamic'    => true,
                    'Tangible'     => false,
                    'PurchaseType' => 'PRODUCT',
                    'Price'        =>
                        array(
                            'Amount' => 6, //amount in currency units
                            'Type'   => 'CUSTOM',
                        ),
                    //Dynamic product subscription.
                    'RecurringOptions' =>
                        array(
                            'CycleLength'    => 1,          //The length of the recurring billing cycle.
                            'CycleUnit'      => 'MONTH',    //Unit of measuring billing cycles (years, months).
                            'CycleAmount'    => 3,          //The amount to be billed on each renewal.
                            'ContractLength' => 3,          //The contact length for which the recurring option will apply.
                            'ContractUnit'   => 'Year',     //Unit of measuring contact length (years, months).
                        )
                )
        )
```

#### **Retrieving the subscriptions of a payment order** using an order number.
- In the following example we will get an array of all the subscriptions on an item:
```php
/**
 * This will return the following array data:
 * return an array having this structure:
 * array(
 *   'item_code' => array(
 *        0 => 'subscription1_code', 
 *        1=> 'Subscription2_code'
 *    )
 * )
**/
 
$productsRefWithSubscriptionRefArray = $tco->subscription()->getSubscriptionsByOrderRefNo( $orderData['RefNo'] );
```
- We can pick a subscription reference using the index it has in the retrieved array and then retrieve subscription details as in the following example:

```php
$subscriptionSearch['SubscriptionReference'] = reset($productsRefWithSubscriptionRefArray)[0];
$subscriptionData = $tco->subscription()->searchSubscriptions($subscriptionSearch);
```
For more parameters by which subscriptions can be searched, check the API documentation [Here](https://app.swaggerhub.com/apis-docs/2Checkout-API/api-rest_documentation/6.0#/), and also check the Subscription class [Subscription](src/Tco/Source/Api/Rest/V6/Subscription.php)

## Buy link signature generation
- To generate a buy link signature, create an array with the following structure (Details [Here](https://knowledgecenter.2checkout.com/Documentation/07Commerce/2Checkout-ConvertPlus/ConvertPlus_URL_parameters))

```php
$buyLinkParameters = ‌array (
                       'address' => 'Street 1',
                       'city' => 'Cleveland',
                       'country' => 'US',
                       'name' => 'John Doe',
                       'phone' => '0018143519403',
                       'zip' => '20034',
                       'email' => 'testcustomer@2Checkout.com',
                       'company-name' => 'Verifone',
                       'state' => 'Ohio',
                       'ship-name' => 'John Doe',
                       'ship-address' => 'Street 1',
                       'ship-city' => 'Cleveland',
                       'ship-country' => 'US',
                       'ship-email' => 'testcustomer@2Checkout.com',
                       'ship-state' => 'Ohio',
                       'prod' => 'Colored Pencil',
                       'price' => 2,
                       'qty' => 1,
                       'type' => 'PRODUCT',
                       'tangible' => 0, // int 0 or 1
                       'src' => 'phpLibrary',
                       'return-url' => 'http://tcolib.local/Examples/Controllers/BuyLink/paymentCallback.php',
                       'return-type' => 'redirect',
                       'expiration' => 1617117946, // this is a dynamic unixtimestamp value
                       'order-ext-ref' => 'CustOrd101',
                       'item-ext-ref' => '20210330102546',
                       'customer-ext-ref' => 'testcustomer@2Checkout.com',
                       'currency' => 'usd',
                       'language' => 'en',
                       'test' => 1, //int (0 or 1)
                       'merchant' => '', //one needs a valid merchant id
                       'dynamic' => 1, //always int (0 or 1)
                       'recurrence' => '1:MONTH',
                       'duration' => '12:MONTH',
                       'renewal-price' => 2,
                     );

$buyLinkSignature = $tco->getBuyLinkSignature($buyLinkParameters);
$buyLinkParameters['signature'] = $buyLinkSignature;
$redirectTo = 'https://secure.2checkout.com/checkout/buy/?' . ( http_build_query( $buyLinkParameters ) );
//Now one can just redirect to the generated url.
```

##Refunds
Full refunds can be initiated using the library. Details about 2Checkout refunds [Here](https://knowledgecenter.2checkout.com/Documentation/27Refunds_and_chargebacks/Refunds)
```php
$orderData = $tco->order()->getOrder( array( 'RefNo' => $this->lastRefNo ) );
//validate if requested order is valid

// Constructing FULL Refund Details
$refundData = array(
    'RefNo'        => $orderData['RefNo'], //Required
    'refundParams' => array(
        "amount"  => $orderData["GrossPrice"], //Required
        "comment" => 'REFUND Example',
        "reason"  => 'Other' //Required, default reason is "Other"
    )
);

$response = $tco->order()->issueRefund( $refundData );
```

## Processing IPNs (Instant Payment Notification)
IPNs are webhooks from 2Checkout which can be used to update payment statuses asynchronously. Instant Payment Notification (IPN) works as a message service generating automatic order/transaction notifications for your 2Checkout account. Use the notifications to process order data into your own management systems by synchronizing it with 2Checkout account events. 

More details [Here](https://verifone.cloud/docs/2checkout/API-Integration/Webhooks/06Instant_Payment_Notification_IPN)

####1. Validate Ipn Hash:
```php
$params = $_POST;
    if ( ! isset( $params['REFNOEXT'] ) && ( ! isset( $params['REFNO'] ) && empty( $params['REFNO'] ) ) ) {
        throw new TcoException( 'Cannot identify order in Ipn request.' );
    }
$validIpn = $tco->validateIpnResponse($params);    
```
####2. Generate Ipn response:
```php
$responseToken = $tco->generateIpnResponse($params);
echo $responseToken;
```


## Exceptions
The library will throw a `TcoException` if an error response is returned. Here are some of the most known & encountered exceptions: 

#####When one has a curl exception
 - 'Exception ApiCore response: ' + MESSAGE
#####When BuyLink signature generator does not return the signature. 
 - 'Api response is missing Signature parameter: ' + MESSAGE
#####The reason above + some reasons regarding configuration or JwtSignature generation
 - 'Exception generating buy link signature: ' + MESSAGE
#####When retrieving a subscription by order reference number 
 - 'Exception getting subscriptions by order RefNo.'
#####When one or more subscription search parameter
 - 'Exception! Some subscription search parameters are not accepted: ' + MESSAGE
#####When some search parameters for order search are not accepted
 - Exception! Some order search parameters are not accepted: ' + MESSAGE
#####If refunds fail this exception is raised
 - 'Error when placing refund request: ' + MESSAGE
#####Missing Secret word value for Buy Link signature generation
 - 'Buy link Secret Word is not set in Tco Config! First set it and then use it.'
#####When TcoConfig receives not accepted parameters or sellerId && secretKey values are missing 
 - 'Some configuration keys are not accepted or not set!'
#####Something went wrong when generating buy link signature in facade
 - 'Exception getting buy link signature! Details: ' + MESSAGE
#####When validating IPN request fails
 - 'Cannot validate Ipn Request. Details:' + MESSAGE
#####When generating IPN response fails
 - 'Cannot generate Ipn Response. Details:' + MESSAGE
