Documentation
===

## Short description

VisitMe™ is one of the biggest food ordering websites in Hungary. This document is a developer guideline for restaurants to be able to check out the orders in their own systems.

## Process and registration

Two type of information has to be stored in our system:
1. You have to provide the URI (Callback URI) on which you would like the notifications of the orders to be received
2. You have to provide a Secret Key for security purposes. This can be selected by the user, but it's highly recommended to consist of at least 32 characters

The order notifications happen through REST API, as described in the following process graph:

![image flow](images/flow.jpg)

1. The order notifications are sent via an HTTP request to the external software. This is always done with the help of the HEAD method. The order identifier can be found in the HTTP header: __X-Visitme-Order-Id__
2. If everything goes well and we receive a 200 response code, __the order status will change to 'seen' in the system! From this point on, it is not our responsibility if the employees of the restaurant do not check the order.__ However, it will be your responsibility to take care of the internal notifications.
3. The details of the order can be retrieved with the GET method. The identifier of the order must be passed as a GET parameter, and the Secret Key must be passed in the __X-Visitme-Auth__ HTTP header.
4. [OPTIONAL, but recommended] The order can be either set to successful or cancelled. If you do not set the order to cancelled within 13 hours, it will be automatically set to completed by the system. __From this point on, the order cannot be modified.__

## Implementation

The public API endpoints (YAML formátumban) OpenAPI specification can be found here:

[OpenAPI documentation](openapi/api-documentation.yaml).

## After implementation

When the implementation is completed in the desired format, please send us an e-mail to info[ a t ]visitme.hu, with the following details:

1. the callback URI (only HTTPS protocol can be accepted)
2. the security key that will be used [preferably in UUID format, or it can be in any other randomly generated format, it can be a 32 character (min.) string as well]
3. the IP address(es) from which the API requests are initiated


### Firewall settings

It is natural to set a firewall to filter the unnecessary connections to a given endpoint. Currently, VisitMe sends notification from the following IP address(es):

* 92.119.123.42

If you are using a WAF or a similar firewall software, please make sure these IP addresses are not blocked.

### Secret key in case of multiple restaurants

If multiple restaurants are using the same software, the Secret Key can be the same if that helps with the authentication.

### Verification of orders that are not visible

Although the availability of our systems is high, there can be unseen network or server issues, as a result of which restaurants do not receive the order notification. After placing the order, the meta information of the order (payment method, order ID, time of order placement) are stored in a secondary and temporary database, from which the frequently done operations are performed. This database is called order manager. When an order is placed, the order manager automatically calls the restaurant after a certain time (6 mins), and it also sends an e-mail to the VisitMe support team after 8 and 16 mins, who take over control from there manually.

If the upload to the external system fails (we do not get a 200 response code), the order manager repeats the request every minute for 13 hours. However, it’s important to highlight that human intervention happens sooner.

Regardless of all the above, there is an API endpoint from which all of the identifiers of the unseen orders can be retrieved.
