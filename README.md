# 50x.com - API Documentation

### Public methods:
#### 1. Current average quotes
```
GET https://rates.50x.com/market/
```
Obtain current average quotes with the base in selected currency or 1/median quotes in selected currency
**Parameters:**
|Name|Type|Mandatory|Description|
|---|---|:----------:|--------|
|base|String|No|Base currency symbol|
|coin|String|No|Currency symbol|
Note:
* Your query should contain parameter `base` or `coin`

**Sample query:**
```
GET https://rates.50x.com/market?base=BTC
```
**Sample reply:**
```
[{
    "STE": 
    {
        "vol": 15463,
        "rate": 4.183e-05, 
        "change": 0.0
    }, 
    "BCH": 
    {
        "vol": 463,
        "rate": 0.079074, 
        "change": -0.16
    }, 
    ... 
}]
```
**vol** - volume in BASE units or in COIN units (depends on query)
**rate** - rate
**change** - changes in % over the last 24 hours

#### 2. Orderbook
```
GET https://rates.50x.com/orderbook/
```
obtain orderbook for an indicated pair
**Parameters:**
|Name|Type|Mandatory|Description|
|---|---|:----------:|--------|
|pair|String|Yes|Pair symbol|
**Sample query:**
```
GET https://rates.50x.com/orderbook?pair=STE/ETH
```
**Sample reply:**
```
{
    "rate_med": 0.00108505, 
    "rate_bid": 0.00101011, 
    "rate_ask": 0.00115999, 
    "bid": [
        [0.00101011, 1085.55], 
        [0.0010101, 130.9999977], 
        ...
    ], 
    "ask": [
        [0.00115999, 676.24147622], 
        [0.00116, 1423.12] 
        ...
    ]
}
```
**rate_med** - average price between best bid and best ask= ( best bid + best ask ) / 2
**rate_bid** - best bid
**rate_ask** - best ask
**bid** - bids (buy orders) array in format [ price, volume ]
**ask** - asks (sell orders) array in format [ price, volume ]


#### 3. Chart
```
GET https://rates.50x.com/chart/
```
obtain median rates chart for a pair
**Parameters:**
|Name|Type|Mandatory|Description|
|---|---|:----------:|--------|
|pair|String|Yes|Pair symbol|
|period|String|Yes|Period symbol|
|begin|Timestamp|No|Beginning of the requested period|
|end|Timestamp|No|End of the requested period|
**Available periods:** M1, M5, M15, M30, H1, H2, H4, D, W
**Sample query:**
```
GET https://rates.50x.com/chart?pair=ETH/DASH&period=h1
```
**Sample reply:**
```
[{"close": 1.28261031,
  "date": 1536256800,
  "high": 1.29277708,
  "low": 1.27164355,
  "open": 1.28232894},
 {"close": 1.27752193,
  "date": 1536260400,
  "high": 1.30094927,
  "low": 1.2643695,
  "open": 1.28310266},
 {"close": 1.27696195,
  "date": 1536264000,
  "high": 1.28768982,
  "low": 1.27360386,
  "open": 1.27763158},
  ...
]
```

#### 4. List of last trades 
```
POST https://rates.50x.com/last_trades/
```
**Parameters:**
|Name|Type|Mandatory|Description|
|---|---|:----------:|--------|
|pair|String|No|Pair's name (obtain last trades for a pair)|
|sym|String|No|Currency's name (obtain last trades for a currency)|
**Sample reply:**
```
{"ok": true,
 "trades": [{"bs": "b",
             "pair": "ETH/BTC",
             "rate": 0.02993642,
             "ts": 1536678972.0,
             "vol1": 0.81505393,
             "vol2": 0.0243998},
            {"bs": "b",
             "pair": "ETH/BTC",
             "rate": 0.03,
             "ts": 1536677973.0,
             "vol1": 10.0,
             "vol2": 0.3},
             ...]
}
```


### Private methods:

All methods are called upon by POST queries to https://api.50x.com with the signature and API key in HTTP header
Attention! The device you are using to call private methods must be synchronized with the internet time and use correct timezone.
If your device system time varies by more than 4 seconds, you will get a 'login failed' message.

**Example: (python 2.7)**
```
from urllib import urlencode
import hashlib
import hmac
import requests
import json
import time


API_50x_SRV = 'https://api.50x.com/'
API_50x_KEY = 'REPLACE TO YOUR API KEY'
API_50x_SEC = 'REPLACE TO YOUR API SEC'


def API_50x_call( method, pars, api_key, api_sec ) :
  headers = {}
  pars.update({'timestamp': int(time.time() * 1000)})
  query_string = urlencode(pars)
  m = hmac.new(api_sec.encode('utf-8'), query_string.encode('utf-8'), hashlib.sha256)        
  sign = m.hexdigest()
  pars['signature'] = sign
  query_string = urlencode(pars)
  headers.update({
            'Accept': 'application/json',
            'User-Agent': 'binance/python',
            'X-MBX-APIKEY': api_key,
            'ORIGIN-PARS': query_string.encode('utf-8')
  })
  sr = requests.post(API_50x_SRV+method+'/', data=pars, headers=headers).text
  rj = json.loads( sr )
  return rj
  

print API_50x_call( 'json.userinfo', {}, API_50x_KEY, API_50x_SEC )
print API_50x_call( 'json.place_order', {'pair':'STE/ETH','ot':'l','v':200,'r':0.001,'bs':'b'}, API_50x_KEY, API_50x_SEC )
print API_50x_call( 'json.orderslist', {'pair':'STE/ETH'}, API_50x_KEY, API_50x_SEC )
```

All methods return JSON. 
In case of success, JSON will NOT contain error field.
In case of an error, method will return JSON with `{error:"error description"}`
If method is designed to be called upon by an authorized user, but is called without authorization, JSON will contain `{error:"Login required"}`
#### 1. User information:
```
POST https://api.50x.com/json.userinfo/
```
After authorization, user information and balances are shown 
**Parameters:** No
**Sample reply** (subsequently, additional fields may appear):
```
{
    "email_confirmed": true, 
    "last_auth_ip": "10.1.0.254", 
    "lang": "en", 
    "first_name": "Nick", 
    "ip_restriction_list": "", 
    "blocked": 0, 
    "prev_auth_ip": "10.1.0.254", 
    "email": "nick3@mail.com", 
    "username": "nick3@mail.com", 
    "ip_restriction_enabled": false, 
    "fa2_auth_enabled": true, 
    "last_name": "", 
    "actions": "", 
    "balances": [
        ["A2A(B)", 0.074999889323, 0.0],
        ["A2A", 0.0, 0.0], 
        ["BTC", 0.8909556407, 0.28180805], 
        ["ETH", 13.08039, 7.614], 
        ["LTC", 0.11994, 0.0], 
        ["PCT", 0.0, 0.0],
        ["DASH", 0.0, 0.0],
        ["USDT", 0.0, 0.0], 
        ["BTG", 0.0, 0.0], 
        ["BCH", 0.0, 0.0], 
        ["TRX", 1839.7, 100.2], 
        ["GSXC", 0.0, 0.0]
    ]
}
``` 
"balances" format: ["COIN-SYMBOL", TOTAL_BALANCE, RESERVED_BALANCE]

#### 2. Incoming transactions
```
POST https://api.50x.com/json.incoming_transactions/
```
shows list of incoming transactions (deposits) 
**Parameters:**
|Name|Type|Mandatory|Description|
|---|---|:----------:|--------|
|pn|String|No|Page number starting from 1 (one page = 30 items)|
|sym|String|No|currency symbol (for example, ETH,BTC,LTC...)|
|state|String|No|"wait" or "ok" (to be deposited or already deposited)|
**Sample reply:**
```
{
    "ok": true, 
    "orders": [
        {"addr": "0x4687d453980f7da22ce8d2960c55045480edcf99", 
         "rec_time": 1522572619.0, 
         "txh": "0x9a8bdc9677dd33580738e0d0b2cfce260d68dae93a12d401c05889cd731e2468", "
         symbol": "ETH", 
         "amount": 0.1, 
         "state": "ok", 
         "confirmations": 98}, 
        {"addr": "0x4687d453980f7da22ce8d2960c55045480edcf99",
         "rec_time": 1522146010.0, 
         "txh": "0x3fbf75c3e54ab072486c9c708a545fe6123dc76aa0418826e09dac0ce060f6b2", 
         "symbol": "ETH", 
         "amount": 0.002,
         "state": "ok",
         "confirmations": 5697}, 
        {"addr": "0x4687d453980f7da22ce8d2960c55045480edcf99", 
         "rec_time": 1522146010.0, 
         "txh": "0x300fb44b9782d1d977472148d306297de8f887662a2677364dabde84044e6230", 
         "symbol": "ETH",
         "amount": 0.0003, 
         "state": "ok", 
         "confirmations": 5765}
    ]
}
```
#### 3. List of withdrawal orders  
```
POST https://api.50x.com/json.withdraw_orders/
```
List of withdrawal orders
**Parameters:**
|Name|Type|Mandatory|Description|
|---|---|:----------:|--------|
|pn|String|No|page number starting form 1 (one page = 30 items)|
|sym|String|No|Currency symbol (for example, ETH,BTC,LTC...)|
|state|String|No|"new", "confirmed", "done" (new, confirmed, completed)|
**Sample reply:**
```
{
    "ok": true, 
    "orders": [
        {"symbol": "DOGE", 
         "state": "done", 
         "time_payout": 1522220735.0, 
         "time_confirm": 1522220553.0, 
         "to_tag": null, 
         "time_create": 1522195200.0, 
         "out_txh": "4c024b9a56da913194850b3339f5c4ae03d7b0cfc3d8d7c86d70f2f97ee4f99b", 
         "to_addr": "DKJ4uJWKtP5oZtTbfuyzNTPfuVC444eSZr", 
         "amount": 98.0
         "memo": "Any string up to 255 characters long for your own referance"}, 
        {"symbol": "DASH", 
         "state": "done", 
         "time_payout": 1522218636.0,
         "time_confirm": 1522218603.0,
         "to_tag": null, 
         "time_create": 1522195200.0, 
         "out_txh": "acb3582a3238ee3722e83a900988de6f994dfa982fd628eb6bdb1b64f147f718",
         "to_addr": "XnJ44u6EAyc3ePTZcxkcW1dFrVFvfc4ggh",
         "amount": 0.00797,
         "memo": "Any string up to 255 characters long for your own referance"}
    ]
}
```
#### 4. Placing an order
```
POST https://api.50x.com/json.place_order/
```
Placing a trading order
**Parameters:**
|Name|Type|Mandatory|Description|
|---|---|:----------:|--------|
|pair|String|Yes|Trading pair (for example, ETH/BTC, ETH/DASH, LTC/ETH)|
|v|Float|Yes|Volume|
|ot|String|Yes|Order type (l - limit, m - market)|
|r|Float|Yes|Order execution rate|
|bs|String|Yes|If BUY, then b, if SELL, then s|
|lifetime|Float|No|Number of seconds of order's lifetime from placement (optional)|
|expire_time|Float|No|time when order is automatically cancelled (optional, lifetime has the priority)|
|memo|String|No|Any string up to 255 characters long for your own referance|
**Sample reply:**
```
{"ok": true,
 "order": {"bs": "b",
           "expire_time": 0,
           "filled_vol": 0,
           "oid": 151871,
           "pair": "ETH/BTC",
           "rate": 0.001,
           "state": 0,
           "sym1": "ETH",
           "sym2": "BTC",
           "time_cancel": 0,
           "time_complete": 0,
           "time_create": 1536856649.0,
           "time_last_fill": 0,
           "type": "l",
           "vol": 1.0,
           "memo": "Any string up to 255 characters long for your own referance",}
}
```
#### 5. Order cancellation
```
POST https://api.50x.com/json.cancel_order/
```
Cancels open orders
**Parameters:**
|Name|Type|Mandatory|Description|
|---|---|:----------:|--------|
|oid|String|Yes|id orders|
**Sample reply:**
```
{"oid": 151871, "ok": true}
```
#### 6. Changing orders
```
POST https://api.50x.com/json.change_order/
```
Change an open order
**Parameters:**
|Name|Type|Mandatory|Description|
|---|---|:----------:|--------|
|oid|String|Yes|id orders|
|amount|Float|No|order's new volume (optional, if not included or = 0, then volume shall remain unchanged)|
|rate|Float|No|order's new rate (optional, if not included or = 0, then rate shall remain unchanged)|
|expire_time|Float|No|order's expiration time, unix timestamp (when it is automatically cancelled, optional, if not inlucded, shall remain unchanged)|
|memo|String|No|Any string up to 255 characters long for your own referance|
**Sample reply:**
```
{"ok": true}
```
#### 7. Getting the lists of Open or Closed orders 
```
POST https://api.50x.com/json.orderslist/
```
Returns the list of open orders.
If history=1 parameter is included, returns the list of closed orders.
**Parameters:**
|Name|Type|Mandatory|Description|
|---|---|:----------:|--------|
|history|String|No|if included, shows history, and not open orders|
|pn|Float|No|page number starting from 1 (one page = 50 orders)|
|pair|String|No|selection based on the indicated pair (for example, ETH/BTC, ETH/DASH, LTC/ETH)|
|sym1|String|No|selection based on the pair's first symbol (for example, for ETH/BTC, the first symbol is ETH)|
|sym2|String|No|selection based on the pair's second symbol (for example, for ETH/BTC, the second symbol is BTC)|
|ot|String|No|selection based on the order's type (l,m,c - see place_order description)|
|bs|String|No|selection based on BUY or SELL, b and s values respectively|
|state|Float|No|selection based on the trading order's state|
**list of possible states of trading orders:**
`0 - Waiting`, `1 - Placed`, `2 - Partial filled`, `3 - Full filled`, `50 - Wait for cancel`, `51 - Cancelling`, `52 - Cancelled`
**Sample reply:**
```
{"ok": true, "orders": [
    {"time_last_fill": 0, 
     "time_cancel": 0, 
     "filled_vol": 0.0, 
     "rate": 0.07598567,
     "commission": 0.2,
     "vol": 1.0,
     "time_create": 1525244149.0, 
     "oid": 101011, 
     "sym1": "ETH", 
     "time_complete": 0, 
     "condition_details": null, 
     "sym2": "BTC", 
     "state": 0, 
     "condition_type": null, 
     "bs": "b", 
     "pair": "ETH/BTC", 
     "memo": "Any string up to 255 characters long for your own referance",
     "type": "l"},
     ...
]}
```
#### 8. Blotter - cashflow
```
POST https://api.50x.com/json.blotter/
```
Obtain all operations with the selected currency on the account 
**Parameters:**
|Name|Type|Mandatory|Description|
|---|---|:----------:|--------|
|sym|String|Yes|Currency symbol|
**Sample reply:**
```
{"ok": true, 
 "blotter": [{"addr": "",
              "amount": 1.0,
              "at": 1536857914.0,
              "data_id": 151907,
              "o_bs": "b",
              "o_pair": "ETH/BTC",
              "o_rate": 0.03247101,
              "o_t": "m",
              "o_vol": 1.0,
              "sym": "ETH",
              "t": "tc",
              "tag": "",
              "txh": ""}],
}
```
**Reply common values:**
"at" - Timestamp of the transaction
"t" - transaction source: indicates which module initiated the transaction, possible states:
`tc - Trading Core`, `com - Commission`, `wd - Withdrawal`, `in - Incomming transaction (deposit)`, `fix - Error correction`
"data_id"- transaction ID, or order ID for trades (if "t"="tc")
"amount"- Balance change for selected asset 
"sym"- Asset Symbol
**Reply deposit/withdrawal values:**
  "txh" - TxHash for withdrawals
  "tag" - Transaction tag for withdrawals for thecoins requiring tags like  XRP, IOTA, XLM
  "addr" - External blockchain address for deposits and withdrawals
**Reply trade/commission/fix values:**
  "o_bs" - buy/sell for trades: `b - buy`, `s - sell`
  "o_t" - Order Type for trades: `l - limit`, `m - market`
  "o_pair" - Order Type for trades
  "o_vol" - Order Volume for trades
  "o_rate" - Order Rate for trades
