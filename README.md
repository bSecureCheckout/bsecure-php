bSecure 
=========================
bSecure is a utility library for two-click checkout custom integration. bSecure library simplifies communication between builder and bSecure server and processing tasks for builder's ease.

### About bSecure Checkout ##

It gives you an option to enable *universal-login*, *two-click checkout* and accept multiple payment method for your customers, as well as run your e-commerce store hassle free.\
It is built for *desktop*, *tablet*, and *mobile devices* and is continuously tested and updated to offer a frictionless payment experience for your e-commerce store.


### Manual Installation
If you do not wish to use Composer, you can download the latest release. Then, to use the bindings, include the init.php file.

`` require_once('/path/to/bSecure-php/init.php')``

**Prerequisites** 

The bindings require the following extensions in order to work properly:
* curl

If you install manually, you'll want to make sure that these extensions are available.

### Documentation
Visit our Site  [bSecure](https://www.bsecure.pk/) to read the documentation and get support.

### Configuration Setup

By following a few simple steps, you can set up your **bSecure Checkout** and **Single-Sign-On**. 

#### Getting Your Credentials

1. Go to [Builder's Portal](https://builder.bsecure.pk/)
2. [App Integration](https://builder.bsecure.pk/integration-sandbox) >> Sandbox / Live
3. Select Environment Type (Custom Integration)
4. Fill following fields:\
    a. *Store URL* its required in any case\
    b. *Login Redirect URL* Required for feature **Login with bSecure**\
    c. *Checkout Redirect URL* Required for feature **Pay with bSecure**\
    d. *Checkout Order Status webhook* Required for feature **Pay with bSecure**
5. Save your client credentials ('<YOUR-CLIENT-ID> and <YOUR-CLIENT-SECRET>')
6. Please make sure to keep credentials at safe place in your code.


## Client Authentication
To call below mentioned functions of bSecure, you first have to authenticate your client.
Following function will be used to authenticate your client:

```
    \bSecure\bSecure::setClientId('<YOUR-CLIENT-ID>');
    \bSecure\bSecure::setClientSecret('<YOUR-CLIENT-SECRET>');
    \bSecure\bSecure::setAppEnvironment('<YOUR-APP-ENVIRONMENT>');
    \bSecure\bSecure::getAuthToken();
```
``
<YOUR-CLIENT-ID> and <YOUR-CLIENT-SECRET> can be obtained from Builder's Portal for your application.
``

We suggest to call this function before each of the following to keep authentication updated:
* create_order
* order_status
* get_customer_profile

  
## bSecure Checkout

#### Create Order
To create an order you should have an order_id, customer, charges and products object parameters that are to be set before creating an order.
##### Create Order Request Params:

###### Product Object:

Products object should be in below mentioned format:

```
'products' => 
      array (
        0 => 
            array (
              'id' => 'product-id',
              'name' => 'product-name',
              'sku' => 'product-sku',
              'quantity' => 0,
              'price' => 0,
              'sale_price' => 0,
              'image' => 'product-image',
              'description' => 'product-description',
              'short_description' => 'product-short-description',
            ),
      ),
```

###### Shipment Object

Shipment object should be in below mentioned format:

>1- If the merchant want his pre-specified shipment method then he should pass shipment method detail in below mentioned format:  
```
'shipment' => 
      array (
        'charges' => 'numeric',
        'method_name' => 'string'
      ),
```

###### Customer Object

Customer object should be in below mentioned format:

>1- If the customer has already signed-in via bSecure into your system and you have auth-code for the customer you can
just pass that code in the customer object no need for the rest of the fields.

>2- Since all the fields in Customer object are optional, if you don’t have any information about customer just pass the
empty object, or if you have customer details then your customer object should be in below mentioned format:
```
'customer' => 
      array (
        'name' => 'string',
        'email' => 'string',
        'country_code' => 'string',
        'phone_number' => 'string',
      ),
```
###### Charges Object

Charges object should be in below mentioned format:

```
'charges' => 
      array (
        'sub_total' => 'float',
        'discount' =>' float',
        'total' => 'float',
      ),
```

#### Create Order
```
\bSecure\Order::setOrderId('<YOUR-ORDER-ID>');
\bSecure\Order::setCustomer($customer);
\bSecure\Order::setShipmentDetails($shipment);
\bSecure\Order::setCartItems($products);
\bSecure\Order::setCharges($charges);
$result = \bSecure\Order::createOrder();
return $result;
```

In response createOrder(), will return order expiry, checkout_url, order_reference and merchant_order_id.
```
array (
  'expiry' => '2020-11-27 10:55:14',
  'checkout_url' => 'bSecure-checkout-url',
  'order_reference' => 'bsecure-reference',
  'merchant_order_id' => '<YOUR-ORDER-ID>',
) 
```
>If you are using a web-solution then simply redirect the user to checkout_url
```
if(!empty($result['checkout_url']))
return redirect($result['checkout_url']); 
```
>If you have Android or IOS SDK then initialize your sdk and provide order_reference to it
```
if(!empty($result']))
return $result; 
```
When order is created successfully on bSecure, you will be redirected to bSecure SDK or bSecure checkout app where you will process your checkout.


#### Callback on Order Placement
Once the order is successfully placed, bSecure will redirect the customer to the url you mentioned in “Checkout
redirect url” in your [environment settings](https://builder.bsecure.pk/) in [Builder's Portal](https://builder.bsecure.pk/), with one additional param **order_ref** in the query
string.

#### Order Updates
By using order_ref you received in the "**[Callback on Order Placement](#callback-on-order-placement)**" you can call below method to get order details.

```
$order_ref = $order->order_ref;

$result =  \bSecure\Order::orderStatus($order_ref);
return $result;
```

#### Order Status Change Webhook
Whenever there is any change in order status or payment status, bSecure will send you an update with complete
order details (contents will be the same as response of *[Order Updates](#order-updates)* on the URL you mentioned in *Checkout Order Status webhook* in your environment settings in [Builder's Portal](https://builder.bsecure.pk/). (your webhook must be able to accept POST request).

## bSecure Single Sign On (SSO)

### Authenticate Client
You can authenticate your client by calling below mentioned function

Save the provided **state** as you will receive the same state in the successful authorization callback.
```
$client = bSecure\SSO::clientAuthenticate($state);
return redirect($client['redirect_url']); 

```
In response clientAuthenticate(), will return a redirect_url, on which you have to redirect your customer.
```
array (
  "redirect_url": "<SSO-REDIRECT-LINK>",
)
```

### Client Authorization
On Successful Authorization,\
bSecure will redirect to <LOGIN-REDIRECT-LINK> you provided when setting up environment in Builders portal, along
with two parameters in query string: **code** and **state**
```
eg: https://my-store.com/sso-callback?code=abc&state=xyz
```
code received in above callback is customer's auth_code which will be further used to get customer profile.

#### Verify Callback
Verify the state you received in the callback by matching it to the value you stored in DB before sending the client authentication
request, you should only proceed if and only if both values match.

### Get Customer Profile
Auth_code recieved from **[Client Authorization](#client-authorization)** should be passed to method below to get customer profile. 


```
return bSecure\SSO::customerProfile('<AUTH-CODE>');
```

In response, it will return customer name, email, phone_number, country_code, address book.
```
array (
    'name' => 'customer-name',
    'email' => 'customer-email',
    'phone_number' => 'customer-phone-number',
    'country_code' => customer-phone-code,
    'address' => 
        array (
          'country' => '',
          'state' => '',
          'city' => '',
          'area' => '',
          'address' => '',
          'postal_code' => '',
        ),
)
```
### Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

## Contributions

**"bSecure – Your Universal Checkout"** is open source software.
