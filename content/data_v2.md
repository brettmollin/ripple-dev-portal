Ripple Data API v2
==================

The Ripple Data API v2 provides access to raw and processed information about changes in the Ripple Consensus Ledger. This information is stored in a database for easy access, which frees `rippled` servers to maintain fewer historical ledger versions. Additionally, the Data API v2 acts as a source of processed analytic data to applications such as [Ripple Charts](https://www.ripplecharts.com/) and [ripple.com](https://www.ripple.com).

Ripple Labs provides a live instance of the `rippled` Historical Database API with as complete a transaction record as possible at the following address:

`https://data.ripple.com`

The Ripple Data API v2 is an evolution of the [Historical Database v1](historical_data.html) and the [Charts API](charts_api.html). The [rippled Historical Database source code](https://github.com/ripple/rippled-historical-database) is also available under an open-source license, so you can use, install, and contribute back to the project.

## API Methods ##

The Data API v2 provides a REST API with the following methods:

General Methods:

* [Get Ledgers - `GET /v2/ledgers/{:ledger_identifier}`](#get-ledgers)
* [Get Transaction - `GET /v2/transactions/{:hash}`](#get-transaction)
* [Get Transactions - `GET /v2/transactions/`](#get-transactions)
* [Get Exchanges - `GET /v2/exchanges/:base/:counter`](#get-exchanges)
* [Get Daily Summary - `GET /v2/reports/`](#get-daily-summary)
* [Get Stats - `GET /v2/stats/`](#get-stats)

Account Methods:

* [Get Account - `GET /v2/accounts/{:address}`](#get-account)
* [Get Accounts - `GET /v2/accounts`](#get-accounts)
* [Get Account Balances - `GET /v2/accounts/{:address}/balances`](#get-account-balances)
* [Get Account Transaction History - `GET /v2/accounts/{:address}/transactions`](#get-account-transaction-history)
* [Get Transaction By Account and Sequence - `GET /v2/accounts/{:address}/transactions/{:sequence}`](#get-transaction-by-account-and-sequence)
* [Get Account Payments - `GET /v2/accounts/{:address}/payments`](#get-account-payments)
* [Get Account Exchanges - `GET /v2/accounts/{:address}/exchanges`](#get-account-exchanges)
* [Get Account Balance Changes - `GET /v2/accounts/{:address}/balance_changes`](#get-account-balance-changes)
* [Get Account Reports - `GET /v2/accounts/{:address}/reports`](#get-account-reports)



# API Objects #

## Transaction Objects ##

Transactions have two formats - a compact "binary" format where the defining fields of the transaction are encoded as strings of hex, and an expanded format where the defining fields of the transaction are nested as complete JSON objects.

### Full JSON Format ###

| Field | Value | Description |
|-------|-------|-------------|
| hash  | String - Transaction Hash | An identifying SHA-512 hash unique to this transaction, as a hex string. |
| date  | String - IS0 8601 UTC Timestamp | The time when this transaction was included in a validated ledger. |
| ledger_index | Number - Ledger Index | The sequence number of the ledger that included this ledger. |
| tx    | Object | The fields of this transaction object, as defined by the [Transaction Format](https://ripple.com/build/transactions) |
| meta  | Object | Metadata about the results of this transaction. |

### Binary Format ###

| Field | Value | Description |
|-------|-------|-------------|
| hash  | String - Transaction Hash | An identifying SHA-512 hash unique to this transaction, as a hex string. |
| date  | String - IS0 8601 UTC Timestamp | The time when this transaction was included in a validated ledger. |
| ledger_index | Number - Ledger Index | The sequence number of the ledger that included this ledger. |
| tx    | String | The binary data that represents this transaction, as a hex string. |
| meta  | Object | The binary data that represents this transaction's metadata, as a hex string. |

## Ledger Objects ##

A "ledger" is one version of the shared global ledger. Each ledger object has the following fields:

| Field        | Value | Description |
|--------------|-------|-------------|
| ledger_hash  | String - Transaction Hash | An identifying hash unique to this ledger, as a hex string. |
| ledger_index | Number (Unsigned Integer) - Ledger Index | The sequence number of the ledger. Each new ledger has a ledger index 1 higher than the ledger that came before it. |
| parent_hash  | String - Transaction Hash | The identifying hash of the previous ledger. |
| total_coins  | Unsigned Integer | The total number of drops of XRP still in existence at the time of the ledger. (Each "drop" is 100,000 XRP.) |
| close\_time\_res | Number | Approximate number of seconds between closing one ledger version and closing the next one. |
| accounts\_hash | String - Hash | Hash of the account information contained in this ledger, as hex. |
| transactions\_hash | String - Hash | Hash of the transaction information contained in this ledger, as hex. |
| close_time | Unsigned Integer - UNIX time | The time at which this ledger was closed. |
| close\_time\_human | String - IS0 8601 UTC Timestamp | The time at which this ledger was closed. |

### Genesis Ledger ###

Every ledger has a sequence number, also known as a Ledger Index, which is an unsigned integer. The very first ledger was sequence number 1, and each subsequent ledger is 1 higher than the sequence number of the ledger immediately before it.

However, due to a mishap early in Ripple's history, ledgers 1 through 32569 were lost. Thus, ledger #32570 is the earliest ledger available anywhere. For purposes of the Data API v2, ledger #32570 is considered the _genesis ledger_.

## Account Creation Objects ##

An account creation object represents the action of creating an account in the Ripple Consensus Ledger. There are two variations, depending on whether the account was already present in ledger 32570, the earliest ledger available. Accounts that were already present in ledger 32570 are termed _genesis accounts_.

| Field | Value | Description |
|-------|-------|-------------|
| address | String - Address | The identifying address of this account, in base-58. |
| inception | String - ISO 8601 UTC Timestamp | The UTC timestamp that the account was created. For genesis accounts, this is the timestamp of ledger 32570. |
| ledger\_index | Number (Unsigned Integer) - Ledger Index | The sequence number of the ledger when the account was created, or 32570 for genesis accounts. |
| parent | String - Address | (Omitted for genesis accounts) The identifying address of the account that provided the initial funding for this account. |
| tx_hash | String - Hash | (Omitted for genesis accounts) The identifying hash of the transaction that funded this account. |
| initial\_balance | String - Decimal number | (Omitted for genesis accounts) The amount of XRP that funded this account. |
| genesis\_balance | Number ?? | (Genesis accounts only) The amount of XRP this account held as of ledger #32570. |
| genesis\_index | Number (Unsigned Integer) | (Genesis accounts only) The transaction sequence number of the account as of ledger #32570. |


## Exchange Objects ##

An exchange object represents an actual exchange of currency, which can occur in the Ripple Consensus Ledger as the result of executing either an OfferCreate transaction or a Payment transaction. In order for currency to actually change hands, there must be a previously-unfilled Offer previously placed in the ledger with an OfferCreate transaction.

A single transaction can cause several exchanges to occur. In this case, the sender of the transaction is the taker for all the exchanges, but each exchange will have a different provider, currency pair, or both.

| Field | Value | Description |
|-------|-------|-------------|
| base\_amount | Number | The amount of the base currency that was traded |
| counter\_amount | Number | The amount of the counter currency that was traded |
| rate | Number | The amount of the counter currency acquired per 1 unit of the base currency |
| buyer | String - Address | The account that acquired the base currency |
| executed\_time | String - ISO 8601 UTC timestamp | The time the exchange occurred |
| ledger\_index | Number - Ledger Index | The sequence number of the ledger that included this transaction |
| offer\_sequence | Number - Transaction Sequence | The sequence number of the `provider`'s existing offer in the ledger. |
| provider | String - Address | The account that had an existing Offer in the ledger |
| seller | String - Address | The account that acquired the counter currency |
| taker | String - Address | The account that sent the transaction which executed this exchange |
| tx\_hash | String - Hash | The identifying hash of the transaction that executed this exchange. (**Note:** This exchange may be one of several executed in a single transaction.) |
| tx\_type | String | The type of transaction that executed this exchange, either `Payment` or `OfferCreate`. |


## Reports Objects ##

Reports objects show the activity of a given account over a specific interval of time, typically a day. Reports have the following fields:

| Field | Value | Description |
|-------|-------|-------------|
| account | String - Address | One account worth of activity |
| date | String - ISO8601 UTC timestamp | 
| high\_value\_received | Number | Largest amount received in a single transaction. <span class='draft-comment'>(Payments only, or offers, too? Normalized?)</span> |
| high\_value\_sent | Number | 
| payments\_received | Number | <span class='draft-comment'>(Including transactions to which this account is involved as neither sender nor receiver?)</span> |
| payments\_sent | Number | 
| receiving\_counterparties | Number | <span class='draft-comment'>(Need a good definition of who counts as a counterparty in a complex tx)</span> |
| sending\_counterparties | Number | 
| total\_value | Number | Sum of total value received and sent. |
| total\_value\_received | Number | Sum of all credits to the `account`.
| total\_value\_sent | Number | 


## Payment Objects ##

In the Data API, a Payment Object represents an event where one account sent value to another account. This mostly lines up with Ripple transactions of the `Payment` transaction type, except that the Data API does not consider a transaction to be a payment if the sending `Account` and the `Destination` account are the same, or if the transaction failed.

Payment objects have the following fields:

| Field | Value | Description |
|-------|-------|-------------|
| amount | String - Number | The amount of the destination `currency` that the transaction was instructed to send. In the case of Partial Payments, this is a "maximum" amount. |
| delivered\_amount | String - Number | The amount of the destination `currency` actually received by the destination account. |
| destination\_balance\_changes | Array | Array of [balance change objects][], indicating all changes made to the `destination` account's balances. |
| source\_balance\_changes | Array | Array of [balance change objects][], indicating all changes to the `source` account's balances (except the XRP transaction cost). |
| fee | String - Number | The amount of XRP spent by the `source` account on the transaction cost. |
| currency | String - Currency Code | The currency that the `destination` account received. |
| destination | String - Address | The account that received the payment. |
| executed\_time | String - ISO 8601 UTC Timestamp | The time the ledger that included this payment closed. |
| ledger\_index | Number - Ledger Index | The sequence number of the ledger that included this payment. |
| source | String - Address | The account that sent the payment. |
| source\_currency | String - Currency Code | The currency that the `source` account spent. |
| tx\_hash | String - Hash | The identifying hash of the transaction that caused the payment. |

[balance change objects]: #balance-change-objects

### Balance Change Objects ###

Balance change objects represent a single balance change that occurs in transaction execution. Every balance change object is associated with a Ripple account. A single Ripple transaction may cause changes to balances with several counterparties, as well as changes to XRP.

Balance Change Objects have the following fields:

| Field | Value | Description |
|-------|-------|-------------|
| counterparty | String - Address | The counterparty, or issuer, of the `currency`. In the case of XRP, this is an empty string. |
| currency | String - Currency Code | The currency for which this balance changed. |
| value | String - Number | The amount of the `currency` that the associated account gained or lost. This value can be positive (for amounts gained) or negative (for amounts lost). |




# API Reference #

## Get Ledgers ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/getLedger.js "Source")

Retrieve a specific Ledger by hash, index, date, or latest validated.

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/ledgers/{:identifier}
```

</div>

The following URL parameters are required by this API endpoint:

| Field | Value | Description |
|-------|-------|-------------|
| ledger_identifier | Ledger Hash, Sequence Number, or ISO 8601 UTC timestamp (YYYY-MM-DDThh:mmZ) | (Optional) An identifier for the ledger to retrieve: either the full hash in hex, an integer sequence number, or a date-time. If a date-time is provided, retrieve the ledger that was most recently closed at that time. If omitted, retrieve the latest validated ledger. |

Optionally, you can also include the following query parameters:

| Field | Value | Description |
|-------|-------|-------------|
| transactions | Boolean | If `true`, include the identifying hashes of all transactions that are part of this ledger. |
| binary | Boolean | If `true`, include all transactions from this ledger as hex-formatted binary data. (If provided, overrides `transactions`.) |
| expand | Boolean | If `true`, include all transactions from this ledger as nested JSON objects. (If provided, overrides `binary` and `transactions`.) |

#### Response Format ####

A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| ledger | [Ledger object](#ledger-objects) | The requested ledger |

#### Example ####

Request:

```
GET /v2/ledgers/3170DA37CE2B7F045F889594CBC323D88686D2E90E8FFD2BBCD9BAD12E416DB5
```

Response:

```
200 OK
{
    "result": "success",
    "ledger": {
        "ledger_hash": "3170da37ce2b7f045f889594cbc323d88686d2e90e8ffd2bbcd9bad12e416db5",
        "ledger_index": 8317037,
        "parent_hash": "aff6e04f07f441abc6b4133f8c50c65935b817a85b895f06dba098b3fbc1be90",
        "total_coins": 99999980165594400,
        "close_time_res": 10,
        "accounts_hash": "8ad73e49a34d8b9c31bc13b8a97c56981e45ee70225ef4892e8b198fec5a1f7d",
        "transactions_hash": "33e0b9c5fd7766343e67854aed4222f5ed9c9507e0ec0d7ae7d54d0f17adb98e",
        "close_time": 1408047740,
        "close_time_human": "2014-08-14T20:22:20+00:00"
    }
}
```



## Get Transaction ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/getTransactions.js "Source")

Retrieve a specific transaction by its identifying hash.

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/transactions/{:hash}
```

</div>

The following URL parameters are required by this API endpoint:

| Field | Value | Description |
|-------|-------|-------------|
| hash  | Transaction Hash | The identifying (SHA-512) hash of the transaction, as hex. |

Optionally, you can also include the following query parameters:

| Field | Value | Description |
|-------|-------|-------------|
| binary | Boolean | If `true`, return transaction data in binary format, as a hex string. Otherwise, return transaction data as nested JSON. Defaults to false. |

#### Response Format ####

A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| transaction | [Transaction object](#transaction-objects) | The requested transaction |

#### Example ####

Request:

```
GET /v2/transactions/03EDF724397D2DEE70E49D512AECD619E9EA536BE6CFD48ED167AE2596055C9A
```

Response:

```js
200 OK
{
    "result": "success",
    "transaction": {
        "ledger_index": 8317037,
        "date": "2014-08-14T20:22:20+00:00",
        "hash": "03EDF724397D2DEE70E49D512AECD619E9EA536BE6CFD48ED167AE2596055C9A",
        "tx": {
            "TransactionType": "OfferCreate",
            "Flags": 131072,
            "Sequence": 159244,
            "TakerPays": {
                "value": "0.001567373",
                "currency": "BTC",
                "issuer": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B"
            },
            "TakerGets": "146348921",
            "Fee": "64",
            "SigningPubKey": "02279DDA900BC53575FC5DFA217113A5B21C1ACB2BB2AEFDD60EA478A074E9E264",
            "TxnSignature": "3045022100D81FFECC36A3DEF0922EB5D16F1AA5AA0804C30A18ED3B512093A75E87C81AD602206B221E22A4E3158785C109E7508624AD3DE5C0E06108D34FA709FCC9575C9441",
            "Account": "r2d2iZiCcJmNL6vhUGFjs8U8BuUq6BnmT"
        },
        "meta": {
            "TransactionIndex": 0,
            "AffectedNodes": [
                {
                    "ModifiedNode": {
                        "LedgerEntryType": "AccountRoot",
                        "PreviousTxnLgrSeq": 8317036,
                        "PreviousTxnID": "A56793D47925BED682BFF754806121E3C0281E63C24B62ADF7078EF86CC2AA53",
                        "LedgerIndex": "2880A9B4FB90A306B576C2D532BFE390AB3904642647DCF739492AA244EF46D1",
                        "PreviousFields": {
                            "Balance": "275716601760"
                        },
                        "FinalFields": {
                            "Flags": 0,
                            "Sequence": 326323,
                            "OwnerCount": 27,
                            "Balance": "275862935331",
                            "Account": "rfCFLzNJYvvnoGHWQYACmJpTgkLUaugLEw",
                            "RegularKey": "rfYqosNivHQFJ6KpArouxoci3QE3huKNYe"
                        }
                    }
                },
                {
                    "ModifiedNode": {
                        "LedgerEntryType": "AccountRoot",
                        "PreviousTxnLgrSeq": 8317030,
                        "PreviousTxnID": "F8E33A40A481F037BA788231421737AF2AD13161928B15A14F6ABC5007D6A2B7",
                        "LedgerIndex": "4F83A2CF7E70F77F79A307E6A472BFC2585B806A70833CCD1C26105BAE0D6E05",
                        "PreviousFields": {
                            "OwnerCount": 18,
                            "Balance": "213169802117"
                        },
                        "FinalFields": {
                            "Flags": 0,
                            "Sequence": 1405,
                            "OwnerCount": 17,
                            "Balance": "213169802799",
                            "Account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59"
                        }
                    }
                },
                {
                    "ModifiedNode": {
                        "LedgerEntryType": "Offer",
                        "PreviousTxnLgrSeq": 8317036,
                        "PreviousTxnID": "B7256723E74A38A17C9F4F5CB938BD45581C57767E44BF26DAC8BA9FF2D58021",
                        "LedgerIndex": "55D0954D7C190006BF905D3B3D269A07016CB9F97B1E37ABE7CC8275DBBDE3DA",
                        "PreviousFields": {
                            "TakerPays": "8016312417",
                            "TakerGets": {
                                "value": "0.085862",
                                "currency": "BTC",
                                "issuer": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B"
                            }
                        },
                        "FinalFields": {
                            "Flags": 0,
                            "Sequence": 326320,
                            "Expiration": 461364125,
                            "BookNode": "0000000000000000",
                            "OwnerNode": "0000000000000003",
                            "BookDirectory": "7B73A610A009249B0CC0D4311E8BA7927B5A34D86634581C5F212B4AE944DDE4",
                            "TakerPays": "7869978846",
                            "TakerGets": {
                                "value": "0.084294634308",
                                "currency": "BTC",
                                "issuer": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B"
                            },
                            "Account": "rfCFLzNJYvvnoGHWQYACmJpTgkLUaugLEw"
                        }
                    }
                },
                {
                    "ModifiedNode": {
                        "LedgerEntryType": "RippleState",
                        "PreviousTxnLgrSeq": 8317014,
                        "PreviousTxnID": "AB356934159ECD02D27089C0AA97EF2B3DA4B3A5A3A9EC522474CADDF276B307",
                        "LedgerIndex": "5DC222A1BAF75AE489F9C78F772AEF6AF00C66660B4C28D93DC05FF4A4124D27",
                        "PreviousFields": {
                            "Balance": {
                                "value": "-2.254933867206176",
                                "currency": "BTC",
                                "issuer": "rrrrrrrrrrrrrrrrrrrrBZbvji"
                            }
                        },
                        "FinalFields": {
                            "Flags": 2228224,
                            "LowNode": "0000000000000221",
                            "HighNode": "0000000000000000",
                            "Balance": {
                                "value": "-2.253363366782792",
                                "currency": "BTC",
                                "issuer": "rrrrrrrrrrrrrrrrrrrrBZbvji"
                            },
                            "LowLimit": {
                                "value": "0",
                                "currency": "BTC",
                                "issuer": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B"
                            },
                            "HighLimit": {
                                "value": "1000",
                                "currency": "BTC",
                                "issuer": "rfCFLzNJYvvnoGHWQYACmJpTgkLUaugLEw"
                            }
                        }
                    }
                },
                {
                    "ModifiedNode": {
                        "LedgerEntryType": "RippleState",
                        "PreviousTxnLgrSeq": 8317030,
                        "PreviousTxnID": "F8E33A40A481F037BA788231421737AF2AD13161928B15A14F6ABC5007D6A2B7",
                        "LedgerIndex": "767C12AF647CDF5FEB9019B37018748A79C50EDAF87E8D4C7F39F78AA7CA9765",
                        "PreviousFields": {
                            "Balance": {
                                "value": "-0.000000007322616",
                                "currency": "BTC",
                                "issuer": "rrrrrrrrrrrrrrrrrrrrBZbvji"
                            }
                        },
                        "FinalFields": {
                            "Flags": 131072,
                            "LowNode": "0000000000000043",
                            "HighNode": "0000000000000000",
                            "Balance": {
                                "value": "0",
                                "currency": "BTC",
                                "issuer": "rrrrrrrrrrrrrrrrrrrrBZbvji"
                            },
                            "LowLimit": {
                                "value": "0",
                                "currency": "BTC",
                                "issuer": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B"
                            },
                            "HighLimit": {
                                "value": "3",
                                "currency": "BTC",
                                "issuer": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59"
                            }
                        }
                    }
                },
                {
                    "DeletedNode": {
                        "LedgerEntryType": "DirectoryNode",
                        "LedgerIndex": "7B73A610A009249B0CC0D4311E8BA7927B5A34D86634581C5F211CEE1E0697A0",
                        "FinalFields": {
                            "Flags": 0,
                            "ExchangeRate": "5F211CEE1E0697A0",
                            "RootIndex": "7B73A610A009249B0CC0D4311E8BA7927B5A34D86634581C5F211CEE1E0697A0",
                            "TakerPaysCurrency": "0000000000000000000000000000000000000000",
                            "TakerPaysIssuer": "0000000000000000000000000000000000000000",
                            "TakerGetsCurrency": "0000000000000000000000004254430000000000",
                            "TakerGetsIssuer": "0A20B3C85F482532A9578DBB3950B85CA06594D1"
                        }
                    }
                },
                {
                    "DeletedNode": {
                        "LedgerEntryType": "Offer",
                        "LedgerIndex": "AF3C702057C9C47DB9E809FD8C76CD22521012C5CC7AE95D914EC9E226F1D7E5",
                        "PreviousFields": {
                            "TakerPays": "182677152",
                            "TakerGets": {
                                "value": "0.001959953669835",
                                "currency": "BTC",
                                "issuer": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B"
                            }
                        },
                        "FinalFields": {
                            "Flags": 131072,
                            "Sequence": 1404,
                            "PreviousTxnLgrSeq": 8317030,
                            "BookNode": "0000000000000000",
                            "OwnerNode": "0000000000000000",
                            "PreviousTxnID": "F8E33A40A481F037BA788231421737AF2AD13161928B15A14F6ABC5007D6A2B7",
                            "BookDirectory": "7B73A610A009249B0CC0D4311E8BA7927B5A34D86634581C5F211CEE1E0697A0",
                            "TakerPays": "182676470",
                            "TakerGets": {
                                "value": "0.001959946361835",
                                "currency": "BTC",
                                "issuer": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B"
                            },
                            "Account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59"
                        }
                    }
                },
                {
                    "ModifiedNode": {
                        "LedgerEntryType": "AccountRoot",
                        "PreviousTxnLgrSeq": 8317036,
                        "PreviousTxnID": "049D5872010AE9144F8FE6A8A92937B6A21C065C2FA9A0233C9CAB7F5485ADC5",
                        "LedgerIndex": "B4C12A5134DCFE012CCC035F62D5903179DE5CABE74B228D84C7E4905BECC032",
                        "PreviousFields": {
                            "Sequence": 159244,
                            "Balance": "201007563226"
                        },
                        "FinalFields": {
                            "Flags": 0,
                            "Sequence": 159245,
                            "OwnerCount": 19,
                            "Balance": "200861228909",
                            "Account": "r2d2iZiCcJmNL6vhUGFjs8U8BuUq6BnmT"
                        }
                    }
                },
                {
                    "ModifiedNode": {
                        "LedgerEntryType": "RippleState",
                        "PreviousTxnLgrSeq": 8317036,
                        "PreviousTxnID": "03229A664FF41A8767818DC8B236E59279078BB33C440996CB0A0093F213C173",
                        "LedgerIndex": "F16445094C00CD3A38CE7851BD6ECA77CA2F281019BE800AB984CDF57D2F2631",
                        "PreviousFields": {
                            "Balance": {
                                "value": "0",
                                "currency": "BTC",
                                "issuer": "rrrrrrrrrrrrrrrrrrrrBZbvji"
                            }
                        },
                        "FinalFields": {
                            "Flags": 1114112,
                            "LowNode": "0000000000000000",
                            "HighNode": "0000000000000233",
                            "Balance": {
                                "value": "0.001567373",
                                "currency": "BTC",
                                "issuer": "rrrrrrrrrrrrrrrrrrrrBZbvji"
                            },
                            "LowLimit": {
                                "value": "0",
                                "currency": "BTC",
                                "issuer": "r2d2iZiCcJmNL6vhUGFjs8U8BuUq6BnmT"
                            },
                            "HighLimit": {
                                "value": "0",
                                "currency": "BTC",
                                "issuer": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B"
                            }
                        }
                    }
                },
                {
                    "ModifiedNode": {
                        "LedgerEntryType": "DirectoryNode",
                        "LedgerIndex": "F60ADF645E78B69857D2E4AEC8B7742FEABC8431BD8611D099B428C3E816DF93",
                        "FinalFields": {
                            "Flags": 0,
                            "RootIndex": "F60ADF645E78B69857D2E4AEC8B7742FEABC8431BD8611D099B428C3E816DF93",
                            "Owner": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59"
                        }
                    }
                }
            ],
            "TransactionResult": "tesSUCCESS"
        }
    }
}
```





## Get Transactions ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/getTransactions.js "Source")

Retrieve transactions by time

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/transactions/
```

</div>

Optionally, you can include the following query parameters:

| Field      | Value | Description |
|------------|-------|-------------|
| start      | String | UTC start time of query range |
| end        | String | UTC end time of query range |
| descending | Boolean | reverse chronological order |
| type       | String | filter transactions for a specific transaction type |
| result     | String | filter transactions for a specific transaction result |
| binary     | Boolean | return transactions in binary form |
| limit      | Integer | max results per page (defaults to 20) |
| marker     | String | The pagination marker from a previous response |

#### Example ####

Request:

```
GET https://data.ripple.com/v2/transactions/?result=tecPATH_DRY&limit=2&type=Payment
```

Response:

```
{
  "result": "success",
  "count": 2,
  "marker": "20130106022000|000000053869|00000",
  "transactions": [
    {
      "hash": "B8E4335A94438EC8209135A4E861A4C88F988C651B819DDAF2E8C55F9B41E589",
      "date": "2013-01-02T20:13:40+00:00",
      "ledger_index": 40752,
      "ledger_hash": "55A900C2BA9483DC83F8FC065DE7789570662365BDE98EB75C5F4CE4F9B43214",
      "tx": {
        "TransactionType": "Payment",
        "Flags": 0,
        "Sequence": 61,
        "Amount": {
          "value": "96",
          "currency": "USD",
          "issuer": "rJ6VE6L87yaVmdyxa9jZFXSAdEFSoTGPbE"
        },
        "Fee": "10",
        "SigningPubKey": "02082622E4DA1DC6EA6B38A48956D816881E000ACF0C5F5B52863B9F698799D474",
        "TxnSignature": "304402200A0746192EBC7BC3C1B9D657F42B6345A49D75FE23EF340CB6F0427254C139D00220446BF9169C94AEDC87F56D01DB011866E2A67E2AADDCC45C4D11422550D044CB",
        "Account": "rB5TihdPbKgMrkFqrqUC3yLdE8hhv4BdeY",
        "Destination": "rJ6VE6L87yaVmdyxa9jZFXSAdEFSoTGPbE"
      },
      "meta": {
        "TransactionIndex": 0,
        "AffectedNodes": [
          {
            "ModifiedNode": {
              "LedgerEntryType": "AccountRoot",
              "PreviousTxnLgrSeq": 40212,
              "PreviousTxnID": "F491DC8B5E51045D4420297293199039D5AE1EA0C6D62CAD9A973E3C89E40CD6",
              "LedgerIndex": "9B242A0D59328CE964FFFBFF7D3BBF8B024F9CB1A212923727B42F24ADC93930",
              "PreviousFields": {
                "Sequence": 61,
                "Balance": "8178999999999400"
              },
              "FinalFields": {
                "Flags": 0,
                "Sequence": 62,
                "OwnerCount": 6,
                "Balance": "8178999999999390",
                "Account": "rB5TihdPbKgMrkFqrqUC3yLdE8hhv4BdeY"
              }
            }
          }
        ],
        "TransactionResult": "tecPATH_DRY"
      }
    },
    {
      "hash": "1E1C14BF5E61682F3DC9D035D9908816497B8E8843E05C0EE98E06DFDDDAE920",
      "date": "2013-01-05T08:43:10+00:00",
      "ledger_index": 51819,
      "ledger_hash": "88ED10E4E31FC7580285CF173B264690B0E8688A3FC9F5F9C62F1A295B96269D",
      "tx": {
        "TransactionType": "Payment",
        "Flags": 0,
        "Sequence": 10,
        "Amount": {
          "value": "2",
          "currency": "EUR",
          "issuer": "rfitr7nL7MX85LLKJce7E3ATQjSiyUPDfj"
        },
        "Fee": "10",
        "SigningPubKey": "03FDDCD97668B686100E60653FD1E5210A8310616669AACB3A1FCC6D2C090CCB32",
        "TxnSignature": "304402204F9BB7E37C14A3A3762E2A7DADB9A28D1AFFB3797521229B6FB98BA666B5491B02204F69AAEAFAC8FA473E52042FF06035AB3618A54E0B76C9852766D55184E98598",
        "Account": "rhdAw3LiEfWWmSrbnZG3udsN7PoWKT56Qo",
        "Destination": "rfitr7nL7MX85LLKJce7E3ATQjSiyUPDfj"
      },
      "meta": {
        "TransactionIndex": 0,
        "AffectedNodes": [
          {
            "ModifiedNode": {
              "LedgerEntryType": "AccountRoot",
              "PreviousTxnLgrSeq": 51814,
              "PreviousTxnID": "5EC1C179996BD87E2EB11FE60A37ADD0FB2229ADC7D13B204FAB04FABED8A38D",
              "LedgerIndex": "AC1B67084F84839A3158A4E38618218BF9016047B1EE435AECD4B02226AB2105",
              "PreviousFields": {
                "Sequence": 10,
                "Balance": "10000999910"
              },
              "FinalFields": {
                "Flags": 0,
                "Sequence": 11,
                "OwnerCount": 2,
                "Balance": "10000999900",
                "Account": "rhdAw3LiEfWWmSrbnZG3udsN7PoWKT56Qo"
              }
            }
          }
        ],
        "TransactionResult": "tecPATH_DRY"
      }
    }
  ]
}
```

#### Response Format ####
A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| count | Integer | Number of Transactions returned. |
| marker | String | Pagination marker |
| transactions | Array of [Transaction object](#transaction-objects) | The requested transactions |




## Get Exchanges ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/getExchanges.js "Source")

Retrieve Exchanges for a given currency pair over time.  Results can be returned as individual exchanges or aggregated to a specific list of intervals

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/exchanges/{:base}/{:counter}
```

</div>

This method requires the following URL parameters:

| Field | Value | Description |
|-------|-------|-------------|
| base  | String | base currency of the pair in the format currency[+issuer] (required) |
| counter | String | counter currency of the pair in the format currency[+issuer] (required) |

Optionally, you can also include the following query parameters:

| Field       | Value | Description |
|-------------|-------|-------------|
| start       | String | UTC start time of query range |
| end         | String | UTC end time of query range |
| interval    | String | Aggregation interval: `1minute`, `5minute`, `15minute`, `30minute`, `1hour`, `2hour`, `4hour`, `1day`, `3day`, `7day`, or `1month` |
| descending  | Boolean | reverse chronological order |
| reduce      | Boolean | aggregate all individual results |
| limit       | Integer | max results per page (defaults to 200) |
| marker      | String | pagination key from previously returned response |
| autobridged | Boolean | return only results from autobridged exchanges |
| format      | String | format of returned results: `csv`,`json` defaults to `json` |

#### Example ####

Request:

```
https://data.ripple.com/v2/exchanges/USD+rMwjYedjc7qqtKYVLiAccJSmCwih4LnE2q/XRP?descending=true&limit=3&result=tesSUCCESS&type=OfferCreate
```

Response:

```
{
  "result": "success",
  "count": 3,
  "marker": "USD|rMwjYedjc7qqtKYVLiAccJSmCwih4LnE2q|XRP||20150820230940|000015383279|00002|00000",
  "exchanges": [
    {
      "base_amount": 1.5868807911938,
      "counter_amount": 207.598995,
      "rate": 130.8220479774132,
      "buyer": "rsCC4p8HGz9R7yHiNjW3WPn9R6sKQv2jbT",
      "executed_time": "2015-08-20T23:09:50",
      "ledger_index": 15383280,
      "offer_sequence": 2269,
      "provider": "rsCC4p8HGz9R7yHiNjW3WPn9R6sKQv2jbT",
      "seller": "rLMJ4db4uwHcd6NHg6jvTaYb8sH5Gy4tg5",
      "taker": "rLMJ4db4uwHcd6NHg6jvTaYb8sH5Gy4tg5",
      "tx_hash": "881A3704CF3DA1212955F9768D5A7A817669F2003046B83C32E96422103A7B2F",
      "tx_type": "Payment"
    },
    {
      "base_amount": 22.5,
      "counter_amount": 2946.032801,
      "rate": 130.9347911555556,
      "buyer": "rJoom9YSbpxnYcQ47SZzdoDE46iPxtZUA7",
      "executed_time": "2015-08-20T23:09:50",
      "ledger_index": 15383280,
      "offer_sequence": 2760,
      "provider": "rJoom9YSbpxnYcQ47SZzdoDE46iPxtZUA7",
      "seller": "rLMJ4db4uwHcd6NHg6jvTaYb8sH5Gy4tg5",
      "taker": "rLMJ4db4uwHcd6NHg6jvTaYb8sH5Gy4tg5",
      "tx_hash": "881A3704CF3DA1212955F9768D5A7A817669F2003046B83C32E96422103A7B2F",
      "tx_type": "Payment"
    },
    {
      "base_amount": 90.6280759869664,
      "counter_amount": 11866.368204,
      "rate": 130.934791171968,
      "buyer": "rMToPrWnbtX8wAKacb8KvV7Uq7oXqVePtm",
      "executed_time": "2015-08-20T23:09:50",
      "ledger_index": 15383280,
      "offer_sequence": 2757,
      "provider": "rMToPrWnbtX8wAKacb8KvV7Uq7oXqVePtm",
      "seller": "rLMJ4db4uwHcd6NHg6jvTaYb8sH5Gy4tg5",
      "taker": "rLMJ4db4uwHcd6NHg6jvTaYb8sH5Gy4tg5",
      "tx_hash": "881A3704CF3DA1212955F9768D5A7A817669F2003046B83C32E96422103A7B2F",
      "tx_type": "Payment"
    }
  ]
}
```

#### Response Format ####
A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| count | Integer | Number of Transactions returned. |
| marker | String | Pagination marker |
| exchanges | Array of <span class='draft-comment'>exchange objects</span> | The requested exchanges |




## Get Daily Summary ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/reports.js "Source")

Retrieve per account per day aggregated payment summaries

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/reports/{:date}
```

</div>

This method requires the following URL parameters:

| Field | Value  | Description |
|-------|--------|-------------|
| date  | String | UTC query date (defaults to today) |

Optionally, you can also include the following query parameters:

| Field    | Value   | Description |
|----------|---------|-------------|
| accounts | Boolean | Include lists of counterparty accounts |
| payments | Boolean | Include lists of individual payments |
| format   | String  | Format of returned results: `csv` or `json`. Defaults to `json`. |


#### Response Format ####

A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| count | Integer | Number of reports returned. |
| marker | String | Pagination marker |
| reports | Array of <span class='draft-comment'>report objects</span> | The requested reports |




## Get Stats ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/stats.js "Source")

Retrieve statistics about transaction activity in the Ripple Consensus Ledger, divided into intervals of time.

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/stats
```

</div>


Optionally, you can also include the following query parameters:

| Field      | Value   | Description |
|------------|---------|-------------|
| family     | String  | If provided, filter results to a single family of stats: `type`, `result`, or `metric`. By default, provides all stats from all families. |
| metrics    | String  | Filter results to one or more metrics (in a comma-separated list). Requires the `family` of the metrics to be specified. By default, provides all metrics in the family. |
| start      | String  | UTC start time of query range |
| end        | String  | UTC end time of query range |
| interval   | String  | Aggregation interval (`hour`,`day`,`week`, defaults to `day`) |
| limit      | Integer | Max results per page (defaults to 200) |
| marker     | String  | Pagination key from previously returned response |
| descending | Boolean | Reverse chronological order |
| format     | String  | Format of returned results: `csv`,`json` defaults to `json` |

##### Families and Metrics #####

The `family` and `metrics` query parameters provide a way to filter results to a specific subset of all metrics available for transactions in any given interval. Each metric is tied to a specific family, as follows:

| Family | Included Metrics | Meaning |
|--------|------------------|---------|
| type | All Ripple transaction types, including `Payment`, `AccountSet`, `SetRegularKey`, `OfferCreate`, `OfferCancel`, `TrustSet`. | Number of transactions of the given type that occurred during the interval. |
| result | All [transaction result codes](transactions.html#transaction-results) (string codes, not the numeric codes), including `tesSUCCESS`, `tecPATH_DRY`, and many others. | Number of transactions that resulted in the given code during the interval. |
| metric | Data-API defined [transaction metrics](#transaction-metrics). | (Varies) |

#### Response Format ####
A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| count | Integer | Number of reports returned. |
| marker | String | Pagination marker |
| stats | Array of stats objects | The requested stats. Omits metrics with a value of 0, and intervals that have no nonzero metrics. |


#### Transaction Metrics ####

The Data API automatically calculates the following metrics for every interval:

| Field  | Value | Description |
|--------|-------|-------------|
| accounts\_created | Number | The number of new accounts funded during this interval. |
| exchanges\_count | Number | The number of currency exchanges that occurred during this interval. |
| ledger\_count | Number | The number of ledgers closed during this interval. |
| ledger\_interval | Number | The average number of seconds between ledgers closing during this interval. |
| payments\_count | Number | The number of payments from one account to another during this interval. |
| tx\_per\_ledger | Number | The average number of transactions per ledger in this interval. |

If any of the metrics have a value of 0, they are omitted from the results.


## Get Accounts ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/accounts.js "Source")

Get funded ripple network accounts

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/accounts
```

</div>

Optionally, you can include the following query parameters:

| Field      | Value   | Description |
|------------|---------|-------------|
| start      | String  | UTC start time of query range |
| end        | String  | UTC end time of query range |
| interval   | String  | Aggregation interval (`hour`,`day`,`week`, defaults to `day`) |
| limit      | Integer | Max results per page (defaults to 200) |
| marker     | String  | Pagination key from previously returned response |
| descending | Boolean | Reverse chronological order |
| parent     | String  | Limit results to specified parent account |
| reduce     | Boolean | Return a count of results only |
| format     | String  | Format of returned results: `csv`,`json` defaults to `json` |

#### Response Format ####
A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| count  | Integer | Number of reports returned. |
| marker | String | Pagination marker |
| stats  | Array of [account creation objects](#account-creation-objects) | The requested accounts |



## Get Account ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/getaccount.js "Source")

Get creation info for a specific ripple account

#### Request Format ####

```
GET /v2/accounts/{:address}
```


This method requires the following URL parameters:

| Field   | Value  | Description |
|---------|--------|-------------|
| address | String | Ripple address to query |

#### Response Format ####

A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| account | Object - [Account Creation](#account-creation-objects) | The requested account |





## Get Account Balances ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/accountBalances.js "Source")

Get balances for a specific ripple account

<div class='multicode'>

*REST*

```
GET /v2/accounts/{:address}/balances
```

</div>

This method requires the following URL parameters:

| Field    | Value  | Description |
|----------|--------|-------------|
| address | String | Ripple address to query |

Optionally, you can also include the following query parameters:

| Field        | Value   | Description |
|--------------|---------|-------------|
| ledger_index | Integer | Index of ledger for historical balances |
| ledger_hash  | String  | Ledger hash for historical balances |
| date         | String  | UTC date for historical balances |
| currency     | String  | Restrict results to specified currency |
| issuer       | String  | Restrict results to specified counterparty/issuer |
| limit        | Integer | Max results per page (defaults to 200) |
| marker       | String  | Pagination key from previously returned response |
| format       | String  | Format of returned results: `csv`,`json` defaults to `json` |



#### Response Format ####
A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| ledger_index | Integer | ledger index for balances query |
| close_time | String | close time of the ledger |
| limit | String | number of results returned, if limit was exceeded |
| marker | String | Pagination marker |
| balances | Array of balance objects | The requested balances |




## Get Account Transaction History ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/accountTransactions.js "Source")

Retrieve a history of transactions that affected a specific account. This includes all transactions the account sent, payments the account received, and payments that rippled through the account.

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/accounts/{:address}/transactions
```

</div>

This method requires the following URL parameters:

| Field    | Value  | Description |
|----------|--------|-------------|
| :address | String | Ripple address to query |


Optionally, you can also include the following query parameters:

| Field        | Value   | Description |
|--------------|---------|-------------|
| start        | String  | UTC start time of query range |
| end          | String  | UTC end time of query range |
| min_sequence | String  | Minimum sequence number to query |
| max_sequence | String  | Max sequence number to query |
| type         | String  | Restrict results to a specified transaction type |
| result       | String  | Restrict results to specified transaction result |
| binary       | Boolean | Return results in binary format |
| descending   | Boolean | Reverse chronological order |
| limit        | Integer | Max results per page (defaults to 20) |
| marker       | String  | Pagination key from previously returned response |
| format       | String  | Format of returned results: `csv`,`json` defaults to `json` |


#### Response Format ####

A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| count  | Integer | The number of objects contained in the `transactions` field. |
| marker | String | Pagination marker |
| transactions | Array of [transaction objects](#transaction-objects) | All transactions matching the request. |




## Get Transaction By Account And Sequence ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/accountTxSeq.js "Source")

Retrieve a specifc transaction originating from a specified account

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/accounts/{:address}/transactions/{:sequence}
```

</div>

This method requires the following URL parameters:

| Field     | Value   | Description |
|-----------|---------|-------------|
| :address  | String  | Ripple address to query |
| :sequence | Integer | Transaction sequence |


Optionally, you can also include the following query parameters:

| Field  | Value   | Description |
|--------|---------|-------------|
| binary | Boolean | Return transaction in binary format |


#### Response Format ####

A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| transaction | [transaction object](#transaction-objects) | requested transaction |




## Get Account Payments ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/accountPayments.js "Source")

Retrieve a payments for a specified account

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/accounts/{:address}/payments
```

</div>

This method requires the following URL parameters:

| Field    | Value  | Description |
|----------|--------|-------------|
| :address | String | Ripple address to query |


Optionally, you can also include the following query parameters:

| Field      | Value   | Description |
|------------|---------|-------------|
| start      | String  | UTC start time of query range |
| end        | String  | UTC end time of query range |
| type       | String  | Type of payment - `sent` or `received` |
| currency   | String  | Restrict results to specified currency |
| issuer     | String  | Restrict results to specified issuer |
| descending | Boolean | Reverse chronological order |
| limit      | Integer | Max results per page (defaults to 20) |
| marker     | String  | Pagination key from previously returned response |
| format     | String  | Format of returned results: `csv`,`json` defaults to `json` |


#### Response Format ####

A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| count  | Integer | The number of objects contained in the `payments` field. |
| marker | String | Pagination marker |
| payments | Array of payment objects | All payments matching the request. |




## Get Account Exchanges ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/accountExchanges.js "Source")

Retrieve Exchanges for a given account over time.

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/accounts/{:address}/exchanges/
GET /v2/accounts/{:address}/exchanges/{:base}/{:counter}
```

</div>

This method requires the following URL parameters:

| Field    | Value  | Description |
|----------|--------|-------------|
| :address | String | Ripple address to query |
| :base    | String | Base currency of the pair in the format currency[+issuer] |
| :counter | String | Counter currency of the pair in the format currency[+issuer] |


Optionally, you can also include the following query parameters:

| Field      | Value   | Description |
|------------|---------|-------------|
| start      | String  | UTC start time of query range |
| end        | String  | UTC end time of query range |
| descending | Boolean | Reverse chronological order |
| limit      | Integer | Max results per page (defaults to 200) |
| marker     | String  | Pagination key from previously returned response |
| format     | String  | Format of returned results: `csv`,`json` defaults to `json` |


#### Response Format ####
A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| count | Integer | Number of exchanges returned. |
| marker | String | Pagination marker |
| exchanges | Array of exchange objects | The requested exchanges |



## Get Account Balance Changes ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/accountBalanceChanges.js "Source")

Retrieve Balance changes for a given account over time.

#### Request Format ####

<div class='multicode'>

*REST*

```
GET /v2/accounts/{:address}/balance_changes/
```

</div>

This method requires the following URL parameters:

| Field    | Value  | Description |
|----------|--------|-------------|
| :address | String | Ripple address to query |


Optionally, you can also include the following query parameters:

| Field      | Value   | Description |
|------------|---------|-------------|
| currency   | String  | Restrict results to specified currency |
| issuer     | String  | Restrict results to specified counterparty/issuer |
| start      | String  | UTC start time of query range |
| end        | String  | UTC end time of query range |
| descending | Boolean | Reverse chronological order |
| limit      | Integer | Max results per page (defaults to 200) |
| marker     | String  | Pagination key from previously returned response |
| format     | String  | Format of returned results: `csv`,`json` defaults to `json` |


#### Response Format ####
A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| count | Integer | Number of balance changes returned. |
| marker | String | Pagination marker |
| exchanges | Array of balance change objects | The requested balance changes |




## Get Account Reports ##
[[Source]<br>](https://github.com/ripple/rippled-historical-database/blob/develop/api/routesV2/accountReports.js "Source")

Retrieve daily summaries of payment activity for an account.

<div class='multicode'>

*REST*

```
GET /v2/accounts/{:address}/reports/
GET /v2/accounts/{:address}/reports/{:date}
```

</div>

This method requires the following URL parameters:

| Field    | Value  | Description |
|----------|--------|-------------|
| :address | String | Ripple address to query |
| :date    | String | UTC date for single report |


Optionally, you can also include the following query parameters:

| Field      | Value   | Description |
|------------|---------|-------------|
| start      | String  | UTC start time of query range |
| end        | String  | UTC end time of query range |
| accounts   | Boolean | Include lists of counterparty accounts |
| payments   | Boolean | Include lists of individual payments |
| descending | Boolean | Reverse chronological order |
| format     | String  | Format of returned results: `csv`,`json` defaults to `json` |


#### Response Format ####
A successful response uses the HTTP code **200 OK** and has a JSON body with the following:

| Field  | Value | Description |
|--------|-------|-------------|
| result | `success` | Indicates that the body represents a successful response. |
| count | Integer | Number of reports returned. |
| reports | Array of report objects | The requested reports |





# Running the Historical Database #

You can also serve the Data API v2 from your own instance of the Historical Database software, and populate it with transactions from your own `rippled` instance. This is useful if you do not want to depend on Ripple Labs to operate the historical database indefinitely, or you want access to historical transactions from within your own intranet.

## Installation ##

### Dependencies ###

The Historical Database requires the following software installed first:

* [HBase](http://hbase.apache.org/) (required for v2),
* [Node.js](http://nodejs.org/)
* [npm](https://www.npmjs.org/)
* [git](http://git-scm.com/) (optional) for installation and updating.

### Installation Process ###

For v2 (hbase):

  1. Set up an hbase cluster
  2. Clone the rippled Historical Database Git Repository:
    `git clone https://github.com/ripple/rippled-historical-database.git`
    (You can also download and extract a zipped release instead.)
  3. Use npm to install additional modules:
    `cd rippled-historical-database`
    `npm install`
    The install script will also create the required config files: `config/api.config.json` and `config/import.config.json`
  4. Modify the API and import config files as needed. If you only wish to run the v2 endpoints, remove the `postgres` section from the api config file.

At this point, the rippled Historical Database is installed. See [Services](#services) for the different components that you can run.

### Services ###

The `rippled` Historical Database consists of several processes that can be run separately.

* [Live Ledger Importer](#live-ledger-importer) - Monitors `rippled` for newly-validated ledgers.
    Command: `node import/live`
* [Backfiller](#backfiller) - Populates the database with older ledgers from a `rippled` instance.
    Command: `node import/postgres/backfill`
* API Server - Provides [REST API access](#usage) to the data.
    Command:  `npm start` (restarts the server automatically when source files change),
    or `node api/server.js` (simple start)

# Importing Data #

In order to retrieve data from the `rippled` Historical Database, you must first populate it with data. Broadly speaking, there are two ways this can happen:

* Connect to a `rippled` server that has the historical ledgers, and retrieve them. (Later, you can reconfigure the `rippled` server not to maintain history older than what you have in your Historical Database.)
    * You can choose to retrieve only new ledgers as they are validated, or you can retrieve old ledgers, too.
* Or, you can load a dump from a database that already has the historical ledger data. (At this time, there are no publicly-available database dumps of historical data.) Use the standard process for your database.

In all cases, keep in mind that the integrity of the data is only as good as the original source. If you retrieve data from a public server, you are assuming that the operator of that server is trustworthy. If you load from a database dump, you are assuming that the provider of the dump has not corrupted or tampered with the data.

## Live Ledger Importer ##

The Live Ledger Importer is a service that connects to a `rippled` server using the WebSocket API, and listens for ledger close events. Each time a new ledger is closed, the Importer requests the latest validated ledger. Although this process has some fault tolerance built in to prevent ledgers from being skipped, it is still possible that the Importer may miss ledgers.

The Live Ledger Importer includes a secondary process that runs periodically to validate the data already imported and check for gaps in the ledger history.

The Live Ledger Importer can import to one or more different data stores concurrently. If you have configured the historical database to use another storage scheme, you can use the `--type` parameter to specify the database type or types to use. If not specified, the `rippled` Historical Database defaults to PostgreSQL.

Here are some examples:

```
// defaults to Hbase:
$ node import/live

// Use Postgres instead:
$ node import/live --type postgres

// Use PostgreSQL and Hbase simultaneously:
$ node import/live --type postgres,hbase
```

## Backfiller ##

The Backfiller retrieves old ledgers from a `rippled` instance by moving backwards in time. You can optionally provide start and stop indexes to retrieve a specific range of ledgers, by their sequence number.

The `--startIndex` parameter defines the most-recent ledger to retrieve. The Backfiller retrieves this ledger first and then continues retrieving progressively older ledgers. If this parameter is omitted, the Backfiller begins with the newest validated ledger.

The `--stopIndex` parameter defines the oldest ledger to retrieve. The Backfiller stops after it retrieves this ledger. If omitted, the Backfiller continues as far back as possible. Because backfilling goes from most recent to least recent, the stop index should be a smaller than the start index.

**Warning:** The Backfiller is best for filling in relatively short histories of transactions. Importing a complete history of all Ripple transactions using the Backfiller could take weeks. If you want a full history, we recommend acquiring a database dump with early transctions, and importing it directly. Ripple Labs used the internal SQLite database from an offline `rippled` to populate its historical databases with the early transactions, then used the Backfiller to catch up to date after the import finished.

Here are some examples:

```
// retrieve everything to PostgreSQL
node import/postgres/backfill

// get ledgers #1,000,000 to #2,000,000 (inclusive) and store in hbase
node import/hbase/backfill --startIndex 2000000 --stopIndex 1000000
```
