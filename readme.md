# KoinPal API - 2019-03-22
## General Information

- The base endpoint is: `https://ex.koinpal.com/api/`
- All endpoints return either a JSON object or array
- Data is return in ascending order. Older first, newest last.
- All time and timestamp related fields are in milliseconds.
- For `GET` endpoints, parameters must be sent as `query string`.
- For `POST`,`PUT`, and `DELETE` endpoints, the parameters may be sent as a `query string` or in the `request body` and `request body` if wish to do so.
- Parameters may be sent in any order.
- If a parameter sent in both the query string and request body, the query string parameter will be used.

## Endpoint security type

- Each endpoint has a security type that determines the how you will interact with it.
- API-keys are passed into Rest API via the `x-kpx-apikey` header. (Case sensitive)
- API-keys and secret-keys are case sensitive.
- API-keys can be configured to only access certain types of secure endpoints. For example, one API-key could be used for TRADE only, while another API-key can access everything except for TRADE routes.
- By default, API-keys can access all secure routes.

Security type | Description
---|---
NONE | Endpoint and be accessed freely
TRADE | Endpoint requiers sending a valid API-Key and signature
USER_DATA | Endpoint requiers sending a valid API-Key and signature
MARKET_DATA | Endpoint requiers sending a valid API-Key

## SIGNED (TRADE and USER_DATA) Endpoint security

- `SIGNED` endpoints require an additional parameter, `signature`, to be sent in the `query string` or `request body`.
- Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation. Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
- The signature is not case sensitive.
- totalParams is defined as the `query string` concatenated with the `request body`.


SIGNED Endpoint Examples for POST /api/v1/order
----------

Here is a step-by-step example of how to send a valid signed payload with PHP.

| Key       | Value                                                            |
| --------- | ---------------------------------------------------------------- |
| apiKey    | e2dd62985130cc84eed62d5ea06426fdbf4aa44ec73b24c49ddbe78588b9442c |
| secretKey | fa3d78ca65c168b02e625bfefcfc64b3418597c1f8ff3f26d02c292cc3143f02 |

| Parameter      | Value                           |
| -------------- | ------------------------------- |
| symbol         | BTCUSD                          |
| side           | BUY                             |
| order_type     | LIMIT                           |
| quantity       | 1                               |
| wallet_address | dsf7a0df7dafdsajfdfuasopfdsasd9 |

Example:
queryString symbol=btcusd&side=sell&quantity=1&order_type=market&wallet_address=dsd79asfd709asf78sasd7fa890

HMAC SHA256 signature

    hash_hmac('sha256', symbol=btcusd&side=sell&quantity=1&order_type=market&wallet_address=dsd79asfd709asf78sasd7fa890,fa3d78ca65c168b02e625bfefcfc64b3418597c1f8ff3f26d02c292cc3143f02)



Public API Endpoints

Terminology

- `base asset` refers to the asset that is the quantity of a symbol.
- `quote asset` refers to the asset that is the `price` of a symbol.

ENUM definitions
Symbol:

- BTCUSD
- ETHUSD
- LTCUSD
- BTCTWD
- ETHTWD
- LTCTWD

Order status

- NEW
- PENDING COMPLETION
- COMPLETED
- CANCELED
- PENDING_CANCEL
- REJECTED
- EXPIRED

Order types (orderTypes, type):

- MARKET

Order side (side):

- BUY
- SELL
General endpoints
Check server time
     GET /v1/time

Test connectivity to the Rest API and get the current server time.

Weight: 1
Parameters: NONE

Response:

    {
      "serverTime": 1499827319559
    }



Symbol price ticker
    GET /v1/ticker/price

Latest price for a symbol.

Weight: 1
Parameters:

| Name   | Type   | Mandatory | Description    |
| ------ | ------ | --------- | -------------- |
| symbol | STRING | YES       | btcusd, btctwd |

Response:

    {
      "symbol": "BTCTWD",
      "price": "118692.40551000"
    }



Account endpoints
New buy order by currency (TRADE)
    POST /v1/order  (HMAC SHA256)

Send in a new buy order by currency amount


New order (TRADE)
    POST /v1/order  (HMAC SHA256)

Send in a new order.

Parameters:

| Name           | Type    | Mandatory | Description                          |
| -------------- | ------- | --------- | ------------------------------------ |
| symbol         | STRING  | YES       |                                      |
| side           | ENUM    | YES       | Buy or Sell                          |
| order_type     | STRING  | YES       | MARKET                               |
| quantity       | DECIMAL | NO        | Buy or sell in term of token         |
| amount         | DECIMAL | NO        | Buy or sell in term of fiat currency |
| wallet_address | LONG    | YES       | Beneficiary wallet address           |

Response ACK

    {
        "order_id": 37
        "symbol": "btcusd",
        "side": "sell",
        "order_type": "market",
        "base": "usd",
        "order_price": 3860.3278, 
        "order_quantity": "10",
        "timestamp": "02-23-2019 16:49:03",
        "wallet_address": "4564safdsadf465678sadfasfdadsafa",
        "order_expiration": "02-23-2019 16:50:03",
    }


Confirm Order endpoints
    POST /v1/confirmOrder

Confirm an order

Weight: 1

| Name    | Type   | Mandatory | Description |
| ------- | ------ | --------- | ----------- |
| orderID | Int    | YES       |             |
| symbol  | String | YES       |             |

Response:

    {
      "status": true,
      "message": "Order confirmed"
    }



Query order (USER_DATA)
    GET /v1/order  (HMAC SHA256)

Check an order’s status.

Weight: 1

Parameters:

| Name    | Type | Mandatory | Description |
| ------- | ---- | --------- | ----------- |
| orderID | Int  | YES       |             |

Response:

    {
      "orderId": "27",
      "symbol": "btcusd",
      "side": "sel",
      "type": "market",
      "quantity": "10.00000000",
      "price": "3883.65180000",
      "time": "02-23-2019 12:07:30",
      "updateTime": "01-01-1970 08:00:00",
      "status": "Canceled"
    }



Cancel order (TRADE)
    DELETE /v1/order  (HMAC SHA256)

Cancel an active order.

Weight: 1

Parameters:

| Name    | Type   | Mandatory | Description |
| ------- | ------ | --------- | ----------- |
| symbol  | STRING | YES       |             |
| orderID | INT    | YES       |             |

Response:

    {
      "status": "success",
      "message": "Order has been canceled"
    }



Current open orders (USER_DATA)
    GET /v1/openOrders  (HMAC SHA256)

Get all open orders on a symbol. Careful when accessing this with no symbol.

Weight: 1 for a single symbol; 40 when the symbol parameter is omitted

Parameters:

| Name   | Type   | Madatory | Description |
| ------ | ------ | -------- | ----------- |
| symbol | STRING | NO       |             |

- If the symbol is not sent, orders for all symbols will be returned in an array
- When all symbols are returned, the number of requests counted against the rate limiter is equal to the number of symbols currently trading on the exchange.

Response:

    [
      {
        "id": "1",
        "symbol": "ethusd",
        "side": "buy",
        "order_type": "market",
        "order_quantity": "10.00000000",
        "order_price": "0.00000000",
        "timestamp": "1550821718",
        "wallet_address": "4564safdsadf465678sadfasfdadsafa",
        "order_status": "0",
        "order_expiration": null,
        "last_update": null
      }
    ]
