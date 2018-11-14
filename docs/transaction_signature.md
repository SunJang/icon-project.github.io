# Introduction

This document describes how to generate a transaction signature. There are two major operations for generating the signature, serialize the original transaction data and sign the serialized data with user's own private key.

- [Serialize transaction data](#Serialize-transaction-data)
  - [Pre-condition](#Pre-condition)
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

When a user sends a transaction, he needs to sign the data with his own private key.
Before signing the data, it needs to be serialized as bytes for generating a hash.
This section describes how to serialize the transaction data.
But it doesn't describe how to make the transaction data itself. It is described in [JSON-RPC API v3](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md) documents.

## Pre-condition

Transaction data is written in JSON with some restrictions.

### Allowed types in JSON

| Type       | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| String     | Normal string without U+0000 (NULL) character. ex) “Value”   |
| Dictionary | Pairs of key and value. ex) {“key1”: “value1”, “key1”: “value2”} |
| Array      | Series of values. ex) [“value1”, “value2”]                   |
| Null       | null                                                         |

## Serialize

ICON uses JSON-RPC v2.0. And the transaction request looks like the following:

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

Serialization is applied on `params` field value, a dictionary. Of course, the `signature` field is ignored. It uses the same rule of Dictionary except that it doesn't have any brace for each end. And it will have a method name (“icx_sendTransaction”) as the prefix of serialized data. It uses `.` for the separator between name and object data itself.

```
icx_sendTransaction.<key1>.<value1>.<key2>.<value2>
```

You may refer UTF-8 encoded byte for all special characters in String type.

### String type

Apply UTF-8 encoding to the text. Characters in the list below should be escaped with `\`. Key and string must not have any null character, U+0000.

| Name  | Backslash (REVERSE SOLIDUS) | Period (FULL STOP) | Left curly bracket | Right curly bracket | Left square bracket | Right square bracket |
| ----- | --------------------------- | ------------------ | ------------------ | ------------------- | ------------------- | -------------------- |
| Shape | *\\*                        | **.**              | **{**              | **}**               | **[**               | **]**                |
| UTF-8 | 0x5C                        | 0x2E               | 0x7B               | 0x7D                | 0x5B                | 0x5D                 |

### Dictionary type

Enclosed with `{` and `}`, and key/value pairs are separated with `.`. The encoding of key follows encoding of String type. The order of keys follows normal “UTF-8” encoding byte comparison (it’s same as Unicode string order).

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

### Normal ICX transfer

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

### SCORE API Invoke

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

For a transaction to be executed successfully, a user needs to create and present the signature with which user can prove the ownership of the address. 

This section describes how to create a valid signature using address's private key and serialized transaction data.

## Required data

### Private key

The private key of `from` address which was used for generating a transaction data.

### Transaction hash 

32 bytes (256-bit) hash data which is created by hashing the serialized transaction data using SHA3_256.

## Create signature

The first step is making a serialized signature. Using the `secp256k1` library, create recoverable ECDSA signature of a transaction. At this point, transaction hash is used as message data (used for validation of transaction). The result data should be 64 bytes serialized signature (R, S) + 1 byte recovery id (V).  

The final step is encoding serialized signature. Based on Base64, encode serialized signature. 

## Example 

Below is the example of creating a signature in python. 

Here is the transaction data. We will create a transaction signature using these data.

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

Hash the serialized transaction data and create transaction signature.

```python
import base64
import hashlib
import secp256k1

serialized_transaction = "icx_sendTransaction.from.hxbe258ceb872e08851f1f59694dac2558708ece11.stepLimit.0x12345.timestamp.0x563a6cf330136.to.cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32.value.0xde0b6b3a7640000.version.0x3"

# transaction_hash
# result: b'\xc4\xa3\xa8\xae\xb5uH\x90\\\xfd\x9a1a\x9b\xe0\x05W\xf6\x03\x9a9\xac\xb8\xc5o\xce\x14\xcak\xae\x1f\x08'
msg_hash = hashlib.sha3_256(serialized_transaction.encode()).digest()

# prepare the private key (this private key is just used for example)
private_key = b'\xbd\xf1o \xef\x8b\xe1\x08\x9f\x81\xd1\xc35\xfcf\xd9\xaa\xb8\t\xc0\xba>\xbcl\x08\xb1\xb8\xf0Q\xde\x7f\xaa'

# create a private key object
private_key_object = secp256k1.PrivateKey(private_key)

# create a recoverable ECDSA signature
recoverable_signature = private_key_object.ecdsa_sign_recoverable(msg_hash, raw=True)

# convert the result from ecdsa_sign_recoverable to a tuple composed of 65 bytes and an integer denominated as recovery id.
signature, recovery_id = private_key_object.ecdsa_recoverable_serialize(recoverable_signature)
recoverable_sig = bytes(bytearray(signature) + recovery_id.to_bytes(1, 'big'))

# base64 encode
transaction_signature = base64.b64encode(recoverable_sig) 

# transaction_signature: b'h4ZD76kjLyaUM4P7OtnhFXPVf2lrJ/9qDZM1/lYJlEIDHhwwPtOAfZ5LBBemNzoJM5r6mXmB8NXZAsVBwGFwWgE='
```
