# Collection Requests

Collection Requests allow you to notify a customer to send funds to you. When you create a collection request, we send a notification to the customer, with insructions on how to fulfill the request. When they send the funds, a collection object shall be created, and matched with the collection request.

Creating a collection request prior to receiving an expected collection greatly improves the user experience for your subscribers by automatically matching transactions to your organization, even if the subscriber misses out some information or gets some of the information wrong, for example if they forget to include a reference code. Payments matching the amount, number and currency in the collection requests will be correctly assigned to you.

On some supported networks, collection requests actually "pull" the funds from the recipient's mobile money wallet, and all they have to do is enter their PIN code to approve the transaction. This greatly improves the customer experience.

The collection requests api endpoint is:
    <aside class="notice">https://app.beyonic.com/api/collectionrequests</aside>

## The Collection Request object

> Sample Collection Request Object (JSON):

```json
{
    "id": 3620,
    "organization": 1,
    "amount": "30",
    "currency": "BXC",
    "phone_number": "+80000000001",
    "created": "2014-11-22T20:57:04.017Z",
    "author": 15,
    "modified": "2014-11-22T20:57:04.018Z",
    "updated_by": 15
}
```

Field | Type | Description
----- | -----| ----
id | long integer | Unique object identifier
organization | long integer | The ID of the organization that owns this collection request (This is usually your organization ID)
amount | decimal | The collection request amount
currency | string | The 3 letter ISO currency code for the collection request. **Note:**: BXC is the Beyonic Test Currency code. See the "Testing" section for more information. Supported currency codes are BXC (Testing), UGX (Uganda), KES (Kenya)
phonenumber | string | The phone number that the collection request is intended for, in international format, starting with a +
reason | string or null | Internal description or reason for this collection request
metadata | hash | Any custom metadata that was added to the collection request when it was created
created | string | The date that the collection request was created, in the UTC timezone. Format: "YYYY-MM-DDTHH:MM:SSZ"
author | long integer | The ID of the user who created the collection request
modified | string | The date that the collection request was last modified, in the UTC timezone. Format: "YYYY-MM-DDTHH:MM:SSZ"
updated_by | string | The ID of the user who last updated the collection request
collection | long integer or null | The ID of the collection that fulfilled the collection request, if any
success_message | string or null | The confirmation message delivered to the customer upon successful completion of this payment request
instructions | string or null | Any custom instructions that were delivered to the customer
send_instructions | boolean | Whether or not Beyonic will attempt to send instructions to the user


## Creating a new Collection Request

> Sample Request:

```shell
curl https://app.beyonic.com/api/collectionrequests -H "Authorization: Token ab594c14986612f6167a975e1c369e71edab6900" \
-d phonenumber=+80000000001 \
-d currency=BXC \
-d amount=3000 \
-d metadata.my_id='123ASDAsd123'
-d send_instructions=True
```

```ruby
require 'beyonic'
Beyonic.api_key = 'ab594c14986612f6167a975e1c369e71edab6900'

collection_request = Beyonic::CollectionRequest.create(
    phonenumber: "+80000000001",
    amount: "100.2",
    currency: "BXC",
    metadata: {"my_id": "123ASDAsd123"},
    send_instructions: true
)

p collection_request  # Examine the returned object
```

```php
<?php
require_once('./lib/Beyonic.php');
Beyonic::setApiKey("ab594c14986612f6167a975e1c369e71edab6900");

$collection_request = Beyonic_Collection_Request::create(array(
  "phonenumber" => "+80000000001",
  "amount" => "100.2",
  "currency" => "BXC",
  "metadata" => array("my_id"=>"123ASDAsd123"),
  "send_instructions" => True
));

print_r($collection_request);  // Examine the returned object
?>
```

```python
import beyonic
beyonic.api_key = 'ab594c14986612f6167a975e1c369e71edab6900'


collection_request = beyonic.CollectionRequest.create(phonenumber='+80000000001',
                       amount='1200',
                       currency='BXC',
                       description='Per diem',
                       callback_url='https://my.website/payments/callback',
                       metadata={'my_id': '123ASDAsd123'},
                       send_instructions = True
                       )

print collection_request  # Examine the returned object
```

```java
package com.beyonic.examples.collectionrequests;

import org.json.JSONException;
import org.json.JSONObject;

import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;

public class CreateCollectionRequestExample {

    private static final String API_ENDPOINT = "https://app.beyonic.com/api/collectionrequests";
    private static final String API_KEY = "ab594c14986612f6167a975e1c369e71edab6900";
    private static final String CHARSET = "UTF-8";
    private static final String PHONE_NUMBER = "+80000000001";
    private static final String CURRENCY = "BXC";
    private static final String DESCRIPTION = "Per Diem";
    private static final String AMOUNT = "1200";
    private static final String CALLBACK_URL = "https://my.website/payments/callback";
    private static final String SEND_INSTRUCTIONS = "True";

    public static void main(String[] args){
        URL url = null;
        try {
            url = new URL(API_ENDPOINT);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();

            conn.setRequestProperty("Content-Type", "application/json");
            conn.setRequestProperty("Accept", "application/json");
            conn.setRequestProperty("charset", CHARSET);
            conn.setRequestProperty("Authorization", "Token " + API_KEY);
            conn.setRequestMethod("POST");
            conn.setDoOutput(true);
            conn.setDoInput(true);

            JSONObject collectionRequestObject = createCollectionRequestObject();

            BufferedWriter out = new BufferedWriter(new OutputStreamWriter(conn.getOutputStream()));
            out.write(collectionRequestObject.toString());
            out.close();

            System.out.println(conn.getResponseCode() + " // " + conn.getResponseMessage());

            int beyID = 0;
            try {
                if (conn.getResponseCode() == 201) {
                    InputStream inputStream = conn.getInputStream();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                    String response = reader.readLine();
                    reader.close();

                    JSONObject obj = new JSONObject(response);
                    beyID = obj.getInt("id");
                    System.out.println("ID of created Collection Request: " + beyID);
                }
            } finally {
                conn.disconnect();
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static JSONObject createCollectionRequestObject() throws JSONException {
        JSONObject collectionRequest = new JSONObject();
        collectionRequest.put("phonenumber", PHONE_NUMBER);
        collectionRequest.put("amount", AMOUNT);
        collectionRequest.put("currency", CURRENCY);
        collectionRequest.put("description", DESCRIPTION);
        collectionRequest.put("callback_url", CALLBACK_URL);
        collectionRequest.put("send_instructions", SEND_INSTRUCTIONS);
        return collectionRequest;
    }
}
```

> Sample Response (JSON) - if you use one of the development libraries, this is automatically converted into a native object for you:

```json
{
    "id": 3620,
    "organization": 1,
    "amount": "30",
    "currency": "BXC",
    "phone_number": "+80000000001",
    "metadata": {
        "my_id": "123ASDAsd123"
    },
    "created": "2014-11-22T20:57:04.017Z",
    "author": 15,
    "modified": "2014-11-22T20:57:04.018Z",
    "updated_by": 15
}
```

To create a new collection request, make a POST to the endpoint above, with the attributes below.

Parameter | Required | Type | Example | Notes
--------- | -------- | ---- | ------- | -----
phonenumber | Yes | String | +80000000001 | Must be in international format
first_name | No | String | Luke | Optional subscriber first name - if omitted, the first name will be set to 'Anonymous'
last_name | No | String | Woods | Optional subscriber last name - if omitted, the last name will be set to 'Contact'
amount | Yes | String, Integer or Decimal | 3000 | Do not include thousands separators
currency | Yes | String | BXC | 3 letter ISO currency code. No currency conversion is done, so the currency must be valid for the selected phonenumber. You must have funded Beyonic an account in this currency. If your account for this currency has zero balance, your payment will fail. **Note:**: BXC is the Beyonic Test Currency code. See the "Testing" section for more information. Supported currency codes are BXC (Testing), UGX (Uganda), KES (Kenya)
reason | No | String | Transaction Fees | The reason for this collection request. This is used when sending automated instructions to the customer. If you omit this field, it will be set to your company name. Please keep this field below 20 characters because some networks may truncate it. You are also advised to include your company name so that it's clear who is requesting the funds.
metadata | No | JSON String | "{'my_id': '123ASDAsd123'}" | Custom attributes to store with this object. See the Metadata section for more information.
success_message | No | String (Max 140 characters) | "Thank you for your payment!" | *New in V2.* This message will be sent via SMS to the subscriber when they make a payment for this collection request. '-Powered by Beyonic' shall be appended to this message. If you leave this message out, a default message shall be sent by Beyonic. You can include {amount} and {customer} placeholders - these will be replaced with the amount and customer name or number details before the message is sent.
send_instructions | No | Boolean | False | *New in V2.* Defaults to False. Indicates whether we should send payment instructions to the subscriber via SMS. Note that this field defaults to False, so if you want the collection request to actually be sent out, you must set this field to True. Omitting the field will prevent collection requests from being sent out.
instructions | No | String (Max 140 characters) | "Use #1234 as the reference" | *New in V2.* Allows overriding of the default instructions sent to the subscriber. If omitted, default network-specific instructions will be sent to the subscriber via SMS. If you want to skip sending ANY sms instructions and turn off even the default instructions, set this parameter to "skip" (instructions = "skip")

## Retrieving a single Collection Request

> Sample Request:

```shell
curl https://app.beyonic.com/api/collectionrequests/230 -H "Authorization: Token ab594c14986612f6167a975e1c369e71edab6900"
```

```ruby
require 'beyonic'
Beyonic.api_key = 'ab594c14986612f6167a975e1c369e71edab6900'

collection_request = Beyonic::CollectionRequest.get(23)

p collection_request  # Examine the returned object
```

```php
<?php
require_once('./lib/Beyonic.php');
Beyonic::setApiKey("ab594c14986612f6167a975e1c369e71edab6900");

$collection_request = Beyonic_Collection_Request::get(23);

print_r($collection_request);  // Examine the returned object
?>
```

```python
import beyonic
beyonic.api_key = 'ab594c14986612f6167a975e1c369e71edab6900'

collection_request = beyonic.CollectionRequest.get(23)

print collection_request  # Examine the returned object
```

```java
package com.beyonic.examples.collectionrequests;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;


public class SingleCollectionRequestExample {

    private static final String API_ENDPOINT = "https://app.beyonic.com/api/collectionrequests";
    private static final String API_KEY = "ab594c14986612f6167a975e1c369e71edab6900";
    private static final String CHARSET = "UTF-8";

    public static void main(String[] args){
        URL url = null;
        try {
            url = new URL(API_ENDPOINT + "/1");
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();

            conn.setRequestProperty("Content-Type", "application/json");
            conn.setRequestProperty("Accept", "application/json");
            conn.setRequestProperty("charset", CHARSET);
            conn.setRequestProperty("Authorization", "Token " + API_KEY);

            System.out.println(conn.getResponseCode() + " // " + conn.getResponseMessage());

            try {
                if (conn.getResponseCode() == 200) {
                    InputStream inputStream = conn.getInputStream();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                    String response = reader.readLine();
                    reader.close();

                    System.out.println(response);
                }
            } finally {
                conn.disconnect();
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

> Sample Response (JSON) - if you use one of the development libraries, this is automatically converted into a native object for you:

```json
{
    "id": 230,
    "organization": 1,
    "amount": "30",
    "currency": "BXC",
    "phone_number": "+80000000001",
    "created": "2014-11-22T20:57:04.017Z",
    "author": 15,
    "modified": "2014-11-22T20:57:04.018Z",
    "updated_by": 15
}
```

To retrieve a single collection request, provide the collection request id and a collection request object will be returned.

Parameter | Required | Type | Example | Notes
--------- | -------- | ---- | ------- | -----
id | Yes | Integer | 23 | The id of the collection you want to retrieve

## Listing all Collection Requests

> Sample Request:

```shell
curl https://app.beyonic.com/api/collectionrequests -H "Authorization: Token ab594c14986612f6167a975e1c369e71edab6900"
```

```ruby
require 'beyonic'
Beyonic.api_key = 'ab594c14986612f6167a975e1c369e71edab6900'

collection_requests = Beyonic::CollectionRequest.list

p collection_requests  # Examine the returned objects
```

```php
<?php
require_once('./lib/Beyonic.php');
Beyonic::setApiKey("ab594c14986612f6167a975e1c369e71edab6900");

$collection_requests = Beyonic_Collection_Request::getAll();

print_r($collection_requests);  // Examine the returned objects
?>
```

```python
import beyonic
beyonic.api_key = 'ab594c14986612f6167a975e1c369e71edab6900'

collection_requests = beyonic.CollectionRequest.list()

print collection_requests  # Examine the returned objects
```

```java
package com.beyonic.examples.collectionrequests;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class ListAllCollectionRequestsExample {

    private static final String API_ENDPOINT = "https://app.beyonic.com/api/collectionrequests";
    private static final String API_KEY = "ab594c14986612f6167a975e1c369e71edab6900";
    private static final String CHARSET = "UTF-8";

    public static void main(String[] args){
        URL url = null;
        try {
            url = new URL(API_ENDPOINT);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();

            conn.setRequestProperty("Content-Type", "application/json");
            conn.setRequestProperty("Accept", "application/json");
            conn.setRequestProperty("charset", CHARSET);
            conn.setRequestProperty("Authorization", "Token " + API_KEY);

            System.out.println(conn.getResponseCode() + " // " + conn.getResponseMessage());

            try {
                if (conn.getResponseCode() == 200) {
                    InputStream inputStream = conn.getInputStream();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                    String response = reader.readLine();
                    reader.close();

                    System.out.println(response);
                }
            } finally {
                conn.disconnect();
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

> Sample Response (JSON) - if you use one of the development libraries, this is automatically converted into a native object for you:

```json
{
    "count": 3,
    "next": "http://localhost:8000/api/collectionrequests?offset=10",
    "previous": null,
    "results": [
        {
            "id": 99,
            "organization": "Beyonic",
            "amount": "3000.0000",
            "currency": "BXC",
            "phonenumber": "+80000000001",
            "metadata": null,
            "created": "2015-08-10T16:10:01Z",
            "author": 42,
            "modified": "2015-08-10T16:10:01Z",
            "updated_by": 42
        },
        {
            "id": 100,
            "organization": "Beyonic",
            "amount": "3000.0000",
            "currency": "BXC",
            "phonenumber": "+80000000001",
            "metadata": null,
            "created": "2015-08-10T16:10:01Z",
            "author": 42,
            "modified": "2015-08-10T16:10:01Z",
            "updated_by": 42
        },
        {
            "id": 101,
            "organization": "Beyonic",
            "amount": "3000.0000",
            "currency": "BXC",
            "phonenumber": "+80000000001",
            "metadata": null,
            "created": "2015-08-10T16:10:01Z",
            "author": 42,
            "modified": "2015-08-10T16:10:01Z",
            "updated_by": 42
        }
    ]
}
```

To retrieve a list of all collections, make a GET request to the collections endpoint. This will return a list of collection objects.

## Filtering Collection Requests

> Sample Request:

```shell
curl https://app.beyonic.com/api/collectionrequests?remote_transaction_id=12345&amount=500 -H "Authorization: Token ab594c14986612f6167a975e1c369e71edab6900"
```

```ruby
require 'beyonic'
Beyonic.api_key = 'ab594c14986612f6167a975e1c369e71edab6900'

collection_requests = Beyonic::CollectionRequest.list(
  remote_transaction_id: '12345',
  amount: 500
)

p collection_requests  # Examine the returned objects
```

```php
<?php
require_once('./lib/Beyonic.php');
Beyonic::setApiKey("ab594c14986612f6167a975e1c369e71edab6900");

$collection_requests = Beyonic_Collection_Request::getAll(array(
  "remote_transaction_id" => "12345",
  "amount" => 500
));

print_r($collection_requests);  // Examine the returned objects
?>
```

```python
import beyonic
beyonic.api_key = 'ab594c14986612f6167a975e1c369e71edab6900'

collection_requests = beyonic.CollectionRequest.list(remote_transaction_id='12345', amount=500)

print collection_requests  # Examine the returned objects
```

```java
package com.beyonic.examples.collectionrequests;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class FilterCollectionRequestsExample {

    private static final String API_ENDPOINT = "https://app.beyonic.com/api/collections";
    private static final String API_KEY = "ab594c14986612f6167a975e1c369e71edab6900";
    private static final String CHARSET = "UTF-8";

    public static void main(String[] args){
        URL url = null;
        try {
            url = new URL(API_ENDPOINT + "?remote_transaction_id=SS12312&amount=500");
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();

            conn.setRequestProperty("Content-Type", "application/json");
            conn.setRequestProperty("Accept", "application/json");
            conn.setRequestProperty("charset", CHARSET);
            conn.setRequestProperty("Authorization", "Token " + API_KEY);

            System.out.println(conn.getResponseCode() + " // " + conn.getResponseMessage());

            try {
                if (conn.getResponseCode() == 200) {
                    InputStream inputStream = conn.getInputStream();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                    String response = reader.readLine();
                    reader.close();

                    System.out.println(response);
                }
            } finally {
                conn.disconnect();
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

You can search or filter collection requests on the following fields. Simply add them to your request as shown in the examples. You can combine multiple filters. Note that filters return exact matches only. (The phonenumber field is treated differently - see below).

* amount - the transaction amount
* currency - the currency code
* collection - the ID of the collection that fulfilled or matched this collection request
* phonenumber - the phonenumber that the collection request was intended for. Note that the phonenumber will be matched in international format, starting with a '+' sign. If the '+' sign isn't included in your request, it will be appended before attempting to match your request.
* remote_transaction_id - the transaction id or transaction reference of the collection on the mobile network operator's side
