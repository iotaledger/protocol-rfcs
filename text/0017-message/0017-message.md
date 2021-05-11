+ Feature name: `tangle-message`
+ Start date: 2020-07-28
+ RFC PR: [iotaledger/protocol-rfcs#0017](https://github.com/iotaledger/protocol-rfcs/pull/0017)

# Summary

A message is the object nodes gossip around in the network. It always references two other messages that are known as `parents`. It is stored as a vertex on the tangle data structure that the nodes maintain.

The messages will contain payloads. Some of them will be core payloads that will be processed by all nodes as part of the core protocol. Some of them will be community payloads that will enable the building of new functionalities on top of the Tangle. Some payloads may have other nested payloads embedded inside.
So upon parsing, it is done layer by layer.

# Motivation

To better understand this layered design, consider the internet protocol, for example: there is an Ethernet frame that contains an IP payload. This in turn contains a TCP packet that encapsulates an HTTP payload. Each layer has a certain responsibility and once this responsibility is completed, we move on to the next layer.

The same is true with how messages are parsed. The outer layer of the message enables the mapping of the message to a vertex in the Tangle and performing some basic validation. The next layer may be a transaction that mutates the ledger state, and the next layer may provide some extra functionality on the transactions to be used by applications.

By making it possible to add and exchange payloads, an architecture is being created that can be easily extended to accommodate future needs.

# Detailed design

### Data types

The following are data types that we will use when we specify fields in the message and payloads.

| Name   | Description   |
| ------ | ------------- |
| uint8  | An unsigned 8 bit integer encoded in Little Endian. |
| uint16  | An unsigned 16 bit integer encoded in Little Endian. |
| uint32  | An unsigned 32 bit integer encoded in Little Endian. |
| uint64  | An unsigned 64 bit integer encoded in Little Endian. |
| ByteArray[N] | A static size array of size N.   |
| ByteArray | A dynamically sized array. A uint32 denotes its length.   |
| string | A dynamically sized array of an UTF-8 encoded string. A uint16 denotes its length.   |


### Message ID
The message ID will be the `BLAKE2b-256` hash of the byte contents of the message. It should be used by the nodes to index the messages and by external APIs.


### Message structure

<table>
    <tr>
        <th>Name</th>
        <th>Type</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>NetworkID</td>
        <td>uint64</td>
        <td>Network identifier. This field will signify whether this message was meant for mainnet, testnet, or a private net. It also tells what protocol rules apply to the message. It is first 8 bytes of the `BLAKE2b-256` hash of the concatenation of the network type and the protocol version string.</td>
        </tr>
    <tr>
        <td> Parents' length </td>
        <td> uint8</td>
        <td> The number of messages we directly approve. Can be any value between 1-8.</td>
    </tr>
    <tr>
        <td>Parents </td>
        <td>ByteArray[32 * `parents length`]</td>
        <td>The Message IDs that are referenced.</td>
    </tr>
    <tr>
        <td>Payload Length</td>
        <td>uint32</td>
        <td> The length of the Payload. Since its type may be unknown to the node it must be declared in advanced. 0 length means no payload will be attached.</td>
    </tr>
    <tr>
        <td colspan="1">
            Payload
        </td>
        <td colspan="2">
            <details open="true">
                <summary>Generic Payload</summary>
                <blockquote>
                An outline of a general payload
                </blockquote>
                <table>
                    <tr>
                        <th>Name</th>
                        <th>Type</th>
                        <th>Description</th>
                    </tr>
                    <tr>
                        <td>Payload Type</td>
                        <td>uint32</td>
                        <td>
                            The type of the payload. It will instruct the node how to parse the fields that follow.
                        </td>
                    </tr>
                    <tr>
                        <td>Data Fields</td>
                        <td>ANY</td>
                        <td>A sequence of fields, where the structure depends on <code>payload type</code>.</td>
                    </tr>
                </table>
            </details>
    <tr>
        <td>Nonce</td>
        <td>uint64</td>
        <td>The nonce which lets this message fulfill the Proof-of-Work requirement.</td>
    </tr>
</table>

### Message validation

A message is considered valid, if the following syntactic rules are met:

1. The message size must not exceed 32 KiB (32 * 1024 bytes).
2. When parsing the message is complete, there should not be any trailing bytes left that were not parsed.
3. If the `payload type` is known to the node.
4. If the [Message PoW Hash](https://github.com/Wollac/protocol-rfcs/blob/message-pow/text/0024-message-pow/0024-message-pow.md) will contain at least the number of trailing 0 trits the node defines as required.
5. `Parents' length` must be between 1-8.


### Payloads

A message may contain a payload. The specification of the payloads is out of scope of this RFC. Below is a table of the currently specified core payloads with a link to their specifications. The `indexation payload` will be specified here as an example:

| Payload Name                              |   Type Value |
| ---------------------------------------   | -----------  | 
|  [Signed Transaction](https://github.com/luca-moser/protocol-rfcs/blob/signed-tx-payload/text/0000-signed-transaction-payload/0000-signed-transaction-payload.md)                       |     0        |
|  [Milestone Draft](https://github.com/jakubcech/protocol-rfcs/blob/jakubcech-milestonepayload/text/0019-milestone-payload/0019-milestone-payload.md)                                |     1        |
|  [Indexation Payload](#indexation-payload)  |     2        |

### Indexation payload

To be clear, the concept of the Payload allows the addition of an index to the encapsulating message, as well as some arbitrary data. Nodes will expose an API that will enable the querying of messages by the index.
Adding those capabilities may open nodes to DOS attack vectors:
1. Proliferation of index keys that may blow up the node's DB
2. Proliferation of messages associated with the same index

Node implementations may provide weak guarantees regarding the completion of indexes to address the above scenarios. 

Besides the index, the payload will also have a data field.
  A message that has been attached to the Tangle and approved by a milestone has several useful properties: verifying that the content of the data did not change and determining the approximate time it was published by checking the approving milestone. If the payload will be incorporated under
  the `signed transaction payload`, the content will be signed as well.


The structure of the payload is simple:

| Name             | Type          | Description               |
| --------         | -----------   | -----------               |
| Payload Type     | uint32        | Must be set to **2**      |
| Index            | ByteArray     | The index key of the message |
| Data             | ByteArray     | Data we are attaching    |

Note that `index` field should be 1 to 64 bytes long for the payload to be valid. The `data` may have a length of 0.


### Serialization Example

Below is a serialized valid message with the indexation payload. The index is the "SPAM" ASCII string and the message is the "Hello Iota"
ASCII string. The [Message PoW Hash](https://github.com/Wollac/protocol-rfcs/blob/message-pow/text/0024-message-pow/0024-message-pow.md) would have
  10 trailing zeroes for the given nonce in this example. Bytes are expressed as hexadecimal numbers.

[Version] **`01`** [Parent 1] `F532A53545103276B46876C473846D98648EE418468BCE76DF4868648DD73E5D` [Parent 2] `78D546B46AEC4557872139A48F66BC567687E8413578A14323548732358914A2` [Payload Length]
*`1100000000000000000000000000000000000000000000000000000000000000`*[Payload Type]**`02`**[Index] *`04`*`5350414D` [Data]*`0A`*`48656C6C6F20496F7461`[Nonce]`5d38333333333333`

# Rationale and alternatives

Instead of creating a layered approach, we could have simply created a flat `transaction message` that is tailored for mutating the ledger state, and try to fit all the use cases there. For example, with the unsigned data use-case, we could have filled some section of the transaction with the data. Then via a flag in the transaction, we could have instructed to not pass this transaction to the service that attempts to mutate the ledger state.

This approach seems less extensible. It might have made sense if we wanted to build a protocol that is just for ledger mutating transactions, but we want to be able to extend the protocol to do more than that.

# Unresolved questions

- What should be the maximum length of the message?
