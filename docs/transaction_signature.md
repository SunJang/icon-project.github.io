# Introduction

When a user sends a transaction, they need to sign the data with their own private key. There are two major steps involved in signing the transaction, serializing the original transaction data and generating a signature with user's own private key. This document describes the process of generating a digital signature of transaction data.

- [Serialize transaction data](#Serialize-transaction-data)
  - [Precondition](#Precondition)
    - [Allowed types in JSON](#Allowed-types-in-JSON)
  - [Serialize](#Serialize)
    - [String type](#String-type)
    - [Dictionary type](#Dictionary-type)
    - [Array type](#Array-type)
    - [Null type](#Null-type)
  - [Example](#example)
    - [Normal send transaction](#Normal-send-transaction)
    - [SCORE API Invoke](#SCORE-API-Invoke)
- [Create transaction signature](#Create-transaction-signature)
  - [Required data](#Required-data)
    - [Private key](#Private-key)
    - [Transaction hash](#Transaction-hash)
  - [Create signature](#Create-signature)
  - [Example](#example-1)

# Serialize transaction data

Before signing the data, the data needs to be serialized as bytes.
This section describes how to serialize the transaction data.
This document does not describe how to make the transaction data itself. Transaction message specification is defined in [JSON-RPC API v3](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md).

## Precondition

Transaction data is in JSON format with some restrictions.

### Allowed types in JSON

| Type       | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| String     | Normal string without U+0000 (NULL) character. ex) “Value”   |
| Dictionary | Pairs of key and value. ex) {“key1”: “value1”, “key1”: “value2”} |
| Array      | Series of values. ex) [“value1”, “value2”]                   |
| Null       | null                                                         |

## Serialize

ICON follows the JSON-RPC v2.0 protocol spec. A signed transaction request looks like the following: 

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "hx5bfdb090f43a808005ffc27c25b213145e80b7cd",
        "value": "0xde0b6b3a7640000",
        "stepLimit": "0x12345",
        "timestamp": "0x563a6cf330136",
        "nonce": "0x1",
        "signature": "VAia7YZ2Ji6igKWzjR2YsGa2m53nKPrfK7uXYW78QLE+ATehAVZPC40szvAiA6NEU5gCYB4c4qaQzqDh2ugcHgA="
    }
}
```

Transaction data is serialized by concatenating key, value pairs in `params` with `.` as a delimiter. Our final goal is generating the `signature` of transaction data, therefore, the `signature` field shown above example is not part of the data to be serialized. Adding the method name, “icx_sendTransaction”, to the serialized string as a prefix completes the serialization process.
```
icx_sendTransaction.<key1>.<value1>....<keyN>.<valueN>
```

Make sure that all the special characters in string must be UTF-8 encoded.

### String type

Apply UTF-8 encoding to the text. Characters listed below should be escaped with `\`. String must not have any null character, U+0000.

| Name  | Backslash (REVERSE SOLIDUS) | Period (FULL STOP) | Left curly bracket | Right curly bracket | Left square bracket | Right square bracket |
| ----- | --------------------------- | ------------------ | ------------------ | ------------------- | ------------------- | -------------------- |
| Shape | *\\*                        | **.**              | **{**              | **}**               | **[**               | **]**                |
| UTF-8 | 0x5C                        | 0x2E               | 0x7B               | 0x7D                | 0x5B                | 0x5D                 |

### Dictionary type

Enclosed with `{` and `}`, and key/value pairs are separated with `.`. Every keys in dictionary are string type, therefore, the same encoding rules apply. The order of keys in the serialized data follows the natural ordering of the UTF-8 encoded byte comparison (it’s same as Unicode string order).

```
{<key1>.<value1>.<key2>.<value2>}
```

Example:

```json
{
       "version": "0x3",
       "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
       "to": "cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32",
       "value": "0xde0b6b3a7640000",
       "stepLimit": "0x12345",
       "timestamp": "0x563a6cf330136"
}
```

Serialized as

``` 
{from.hxbe258ceb872e08851f1f59694dac2558708ece11.stepLimit.0x12345.timestamp.0x563a6cf330136.to.cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32.value.0xde0b6b3a7640000.version.0x3}
```

### Array type

Enclosed with `[` and `]`. All values are separated with `.`.

``` 
[<value1>.<value2>.<value3>]
```

### Null type

Null will be represented as an escaped zero.

```
\0
```

## Example

### ICX transfer

Original JSON request:

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "hx5bfdb090f43a808005ffc27c25b213145e80b7cd",
        "value": "0xde0b6b3a7640000",
        "stepLimit": "0x12345",
        "timestamp": "0x563a6cf330136",
        "nonce": "0x1"
    }
}
```

Serialized params:

```
icx_sendTransaction.from.hxbe258ceb872e08851f1f59694dac2558708ece11.nonce.0x1.stepLimit.0x12345.timestamp.0x563a6cf330136.to.hx5bfdb090f43a808005ffc27c25b213145e80b7cd.value.0xde0b6b3a7640000.version.0x3
```

### SCORE API invocation

Original JSON request:

```json

{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32",
        "stepLimit": "0x12345",
        "timestamp": "0x563a6cf330136",
        "nonce": "0x1",
        "dataType": "call",
        "data": {
            "method": "transfer",
            "params": {
                "to": "hxab2d8215eab14bc6bdd8bfb2c8151257032ecd8b",
                "value": "0x1"
            }
        }
    }
}
```

Serialized params:

 ```
icx_sendTransaction.data.{method.transfer.params.{to.hxab2d8215eab14bc6bdd8bfb2c8151257032ecd8b.value.0x1}}.dataType.call.from.hxbe258ceb872e08851f1f59694dac2558708ece11.nonce.0x1.stepLimit.0x12345.timestamp.0x563a6cf330136.to.cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32.version.0x3
 ```

# Create transaction signature

This section describes how to create a valid signature for a transaction using the private key associated with a specific wallet address.

## Required data

### Private key

The private key of the `from` address from which the transaction request was made.

### Transaction hash 

32 bytes (256-bit) hash data which is created by hashing the serialized transaction data using SHA3_256.

## Create signature

The first step is making a serialized signature. Using the `secp256k1` library, create a recoverable ECDSA signature of a transaction hash. This ensures that the transaction was originated from the private key owner, and the transaction message received by the recipient is not compromised. The resulting output should be 64 bytes serialized signature (R, S) with 1 byte recovery id (V).

The final step is to encode the generated signature as a Base64-encoded string.

## Example 

Below is the example of creating a signature in python. 

Here is a sample transaction request message. We will create a signature for the transaction data.

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32",
        "value": "0xde0b6b3a7640000",
        "stepLimit": "0x12345",
        "timestamp": "0x563a6cf330136"
    }
}
```

Serialize transaction data.

```
icx_sendTransaction.from.hxbe258ceb872e08851f1f59694dac2558708ece11.stepLimit.0x12345.timestamp.0x563a6cf330136.to.cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32.value.0xde0b6b3a7640000.version.0x3
```

Hash the serialized transaction data and create a transaction signature.

```python
import base64
import hashlib
import secp256k1

serialized_transaction = "icx_sendTransaction.from.hxbe258ceb872e08851f1f59694dac2558708ece11.stepLimit.0x12345.timestamp.0x563a6cf330136.to.cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32.value.0xde0b6b3a7640000.version.0x3"

# transaction_hash
# result: b'\xc4\xa3\xa8\xae\xb5uH\x90\\\xfd\x9a1a\x9b\xe0\x05W\xf6\x03\x9a9\xac\xb8\xc5o\xce\x14\xcak\xae\x1f\x08'
msg_hash = hashlib.sha3_256(serialized_transaction.encode()).digest()

# prepare the private key (this private key is for example purpose)
private_key = b'\x870\x91*\xef\xedB\xac\x05\x8f\xd3\xf6\xfdvu8\x11\x04\xd49\xb3\xe1\x1f\x17\x1fTR\xd4\xf9\x19mL'

# create a private key object
private_key_object = secp256k1.PrivateKey(private_key)

# create a recoverable ECDSA signature
recoverable_signature = private_key_object.ecdsa_sign_recoverable(msg_hash, raw=True)

# convert the result from ecdsa_sign_recoverable to a tuple composed of 64 bytes and an integer denominated as recovery id.
signature, recovery_id = private_key_object.ecdsa_recoverable_serialize(recoverable_signature)
recoverable_sig = bytes(bytearray(signature) + recovery_id.to_bytes(1, 'big'))

# base64 encode
transaction_signature = base64.b64encode(recoverable_sig) 

# transaction_signature: b'a5fs7KC8Qw3Rpgyhx2b02WG7jghqdRT58dznUVb8qV12QhWx0zXi0YnIAmHHL2NF55ULn1RaEwrzQq2Fiq5W8wA='
```
