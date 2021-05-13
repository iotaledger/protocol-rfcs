+ Feature name: transaction_payload
+ Start date: 2020-07-10
+ RFC PR: [iotaledger/protocol-rfcs#18](https://github.com/iotaledger/protocol-rfcs/pull/18)

# Summary

This RFC defines a new transaction structure for Chrysalis Phase 2, which replaces the current notion of bundles. Specifically, this RFC describes the transaction payload to be embedded into a [message](https://github.com/iotaledger/protocol-rfcs/pull/17).

# Motivation

The current IOTA protocol uses **transactions** (which are vertices in the Tangle), where each transaction defines either an input or output. A grouping of those input/output transaction vertices make up a **bundle** which transfers the given values as an atomic unit (the entire bundle is applied or none of it). The input transactions define the funds to consume and create the deposits onto the output transactions target addresses. Additionally, to accommodate the larger WOTS signatures, additional transaction vertices might be part of the bundle to carry parts of the signature which do not fit into one transaction vertex.

The bundle concept has proven to be time consuming, with several issues as well:
* Since the data making up the bundle is split across multiple vertices, it complicates the validation of the entire transfer. Instead of being able to immediately tell whether a bundle is valid or not, a node implementation must first collect all parts of the bundle before any actual validation can happen. This increases the complexity of the node implementation.
* Reattaching the tail transaction of a bundle causes the entire transfer to be reapplied.
* Due to the split across multiple transaction vertices and having to do PoW for each of them, a bundle might already be lazy in terms of where it attaches, reducing its chances to be confirmed.

To fix the problems mentioned above and to create a more flexible transaction structure, the goal is to achieve a self-contained transaction structure defining the data of the entire transfer as a payload to be embedded into a message.

The new transaction structure should fulfil the following criteria:
* Support for Ed25519 (and thus reusable addresses).
* Support for adding new types of signature schemes, addresses, inputs, and outputs as part of protocol upgrades.
* Self-contained, as in being able to validate the transaction immediately after receiving it.
* Enable unspent transaction outputs (UTXO) as inputs instead of an account based model (UTXO enables easier double-spend detection).

# Detailed design

## UTXO

The *unspent transaction output* (UTXO) model defines a ledger state where balances are not directly associated to addresses but to the outputs of transactions. In this model, transactions specify the outputs of previous transactions as inputs, which are consumed to create new outputs. A transaction must consume the entirety of the specified inputs.

Using an UTXO based model provides several benefits:
* Parallel validation of transactions.
* Easier double-spend detection, since conflicting transactions would reference the same UTXO.
* Replay-protection which is important when having reusable addresses. Replaying the same transaction would manifest itself as already being applied or existent and thus not have any impact.
* Technically seen, balances may no longer be associated to addresses which raises the level of abstraction and thus enables other types of outputs. Consider, for example, a type of output which specifies the balance to be unlocked by a transaction which must fulfil a Proof-of-Work difficulty or supply some other unlock criteria, etc.

Within a transaction using UTXOs, inputs and outputs make up the to-be-signed data of the transaction. The section unlocking the inputs is called *unlock block*. An unlock block may contain a signature proving ownership of a given input's address and/or other unlock criteria.

The following image depicts the flow of funds using UTXO:

![UTXO flow](https://i.imgur.com/h3uxf6N.png)

The way UTXOs are referenced is further described in the <i>Structure</i> section of this RFC.

## Structure

### Serialized Layout

A <i>Transaction Payload</i> payload is made up of two parts:
1. The <i>The Transaction Essence</i> part which contains the inputs, outputs and an optional embedded payload.
2. The <i>Unlock Blocks</i> which unlock the <i>Transaction Essence</i>'s inputs. In case the unlock block contains a signature, it signs the entire <i>Transaction Essence</i> part.

All values are serialized in little-endian encoding. The serialized form of the transaction is deterministic, meaning the same logical transaction always results in the same serialized byte sequence.

A `Blake2b-256` hash of the entire serialized data makes up <i>Transaction Payload</i>'s ID.

Following table structure describes the entirety of a <i>Transaction Payload</i>'s serialized form:
* [Data Type Notation](https://github.com/GalRogozinski/protocol-rfcs/blob/message/text/0017-message/0017-message.md#data-types)
* <details>
    <summary>Subschema Notation</summary>
    <table>
        <tr>
            <th>Name</th>
            <th>Description</th>
        </tr>
        <tr>
            <td><code>oneOf</code></td>
            <td>One of the listed subschemas.</td>
        </tr>
        <tr>
            <td><code>optOneOf</code></td>
            <td>Optionally one of the listed subschemas.</td>
        </tr>
        <tr>
            <td><code>anyOf</code></td>
            <td>Any (one or more) of the listed subschemas.</td>
        </tr>
    </table>
</details>

<p></p>

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
        Set to <strong>value 0</strong> to denote a <i>Transaction Payload</i>.
        </td>
    </tr>
    <tr>
        <td valign="top">Essence <code>oneOf</code></td>
        <td colspan="2">
            <details open="true">
                <summary>Transaction Essence</summary>
                <blockquote>
                Describes the essence data making up a transaction by defining its inputs and outputs and an optional payload.
                </blockquote>
                <table>
                    <tr>
                        <td><b>Name</b></td>
                        <td><b>Type</b></td>
                        <td><b>Description</b></td>
                    </tr>
                    <tr>
                        <td>Transaction Type</td>
                        <td>uint8</td>
                        <td>
                        Set to <strong>value 0</strong> to denote a <i>Transaction Essence</i>.
                        </td>
                    </tr>
                   <tr>
                        <td>Inputs Count</td>
                        <td>uint16</td>
                        <td>The amount of inputs proceeding.</td>
                    </tr>
                   <tr>
                        <td valign="top">Inputs <code>anyOf</code></td>
                        <td colspan="2">
                            <details>
                                <summary>UTXO Input</summary>
                                <blockquote>
                                Describes an input which references an unspent transaction output to consume.
                                </blockquote>
                                <table>
                                    <tr>
                                        <td><b>Name</b></td>
                                        <td><b>Type</b></td>
                                        <td><b>Description</b></td>
                                    </tr>
                                    <tr>
                                        <td>Input Type</td>
                                        <td>uint8</td>
                                        <td>
                                            Set to <strong>value 0</strong> to denote an <i>UTXO Input</i>.
                                        </td>
                                    </tr>
                                    <tr>
                                        <td>Transaction ID</td>
                                        <td>Array&lt;byte&gt;[32]</td>
                                        <td>The BLAKE2b-256 hash of the transaction from which the UTXO comes from.</td>
                                    </tr>
                                    <tr>
                                        <td>Transaction Output Index</td>
                                        <td>uint16</td>
                                        <td>The index of the output on the referenced transaction to consume.</td>
                                    </tr>
                                </table>
                            </details>
                        </td>
                    </tr>
                   <tr>
                        <td>Outputs Count</td>
                        <td>uint16</td>
                        <td>The amount of outputs proceeding.</td>
                    </tr>
                   <tr>
                        <td valign="top">Outputs <code>anyOf</code></td>
                        <td colspan="2">
                            <details>
                                <summary>SigLockedSingleOutput</summary>
                                <blockquote>
                                Describes a deposit to a single address which is unlocked via a signature.
                                </blockquote>
                                <table>
                                    <tr>
                                        <td><b>Name</b></td>
                                        <td><b>Type</b></td>
                                        <td><b>Description</b></td>
                                    </tr>
                                    <tr>
                                        <td>Output Type</td>
                                        <td>uint8</td>
                                        <td>
                                            Set to <strong>value 0</strong> to denote a <i>SigLockedSingleOutput</i>.
                                        </td>
                                    </tr>
                                    <tr>
                                        <td valign="top">Address <code>oneOf</code></td>
                                        <td colspan="2">
                                            <details>
                                                <summary>Ed25519 Address</summary>
                                                <table>
                                                    <tr>
                                                        <td><b>Name</b></td>
                                                        <td><b>Type</b></td>
                                                        <td><b>Description</b></td>
                                                    </tr>
                                                    <tr>
                                                        <td>Address Type</td>
                                                        <td>uint8</td>
                                                        <td>
                                                            Set to <strong>value 0</strong> to denote an <i>Ed25519 Address</i>.
                                                        </td>
                                                    </tr>
                                                    <tr>
                                                        <td>Address</td>
                                                        <td>Array&lt;byte&gt;[32]</td>
                                                        <td>The raw bytes of the Ed25519 address which is a BLAKE2b-256 hash of the Ed25519 public key.</td>
                                                    </tr>
                                                </table>
                                            </details>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td>Amount</td>
                                        <td>uint64</td>
                                        <td>The amount of tokens to deposit.</td>
                                    </tr>
                                </table>
                            </details>
                            <details>
                                <summary>SigLockedDustAllowanceOutput</summary>
                                <blockquote>
                                Describes a deposit which as a special property also alters the dust allowance of the target address.
                                </blockquote>
                                <table>
                                    <tr>
                                        <td><b>Name</b></td>
                                        <td><b>Type</b></td>
                                        <td><b>Description</b></td>
                                    </tr>
                                    <tr>
                                        <td>Output Type</td>
                                        <td>uint8</td>
                                        <td>
                                            Set to <strong>value 1</strong> to denote a <i>SigLockedDustAllowanceOutput</i>.
                                        </td>
                                    </tr>
                                    <tr>
                                        <td valign="top">Address <code>oneOf</code></td>
                                        <td colspan="2">
                                            <details>
                                                <summary>Ed25519 Address</summary>
                                                <table>
                                                    <tr>
                                                        <td><b>Name</b></td>
                                                        <td><b>Type</b></td>
                                                        <td><b>Description</b></td>
                                                    </tr>
                                                    <tr>
                                                        <td>Address Type</td>
                                                        <td>uint8</td>
                                                        <td>
                                                            Set to <strong>value 0</strong> to denote an <i>Ed25519 Address</i>.
                                                        </td>
                                                    </tr>
                                                    <tr>
                                                        <td>Address</td>
                                                        <td>Array&lt;byte&gt;[32]</td>
                                                        <td>The raw bytes of the Ed25519 address which is a BLAKE2b-256 hash of the Ed25519 public key.</td>
                                                    </tr>
                                                </table>
                                            </details>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td>Amount</td>
                                        <td>uint64</td>
                                        <td>The amount of tokens to deposit.</td>
                                    </tr>
                                </table>
                            </details>
                        </td>
                    </tr>
                    <tr>
                        <td>Payload Length</td>
                        <td>uint32</td>
                        <td>The length in bytes of the optional payload.</td>
                    </tr>
                   <tr>
                        <td valign="top">Payload <code>optOneOf</code></td>
                        <td colspan="2">
                            <details>
                                <summary>Indexation Payload</summary>
                            </details>
                        </td>
                    </tr>
                </table>
            </details>
        </td>
    </tr>
    <tr>
        <td>Unlock Blocks Count</td>
        <td>uint16</td>
        <td>The count of unlock blocks proceeding. Must match count of inputs specified.</td>
    </tr>
    <tr>
        <td valign="top">Unlock Blocks <code>anyOf</code></td>
        <td colspan="2">
            <details open="true">
                <summary>Signature Unlock Block</summary>
                <blockquote>
                Defines an unlock block containing signature(s) unlocking input(s).
                </blockquote>
                <table>
                    <tr>
                        <th>Name</th>
                        <th>Type</th>
                        <th>Description</th>
                    </tr>
                    <tr>
                        <td>Unlock Type</td>
                        <td>uint8</td>
                        <td>
                            Set to <strong>value 0</strong> to denote a <i>Signature Unlock Block</i>.
                        </td>
                    </tr>
                    <tr>
                        <td valign="top">Signature <code>oneOf</code></td>
                        <td colspan="2">
                             <details>
                                <summary>Ed25519 Signature</summary>
                                <table>
                                    <tr>
                                        <th>Name</th>
                                        <th>Type</th>
                                        <th>Description</th>
                                    </tr>
                                    <tr>
                                        <td>Signature Type</td>
                                        <td>uint8</td>
                                        <td>
                                            Set to <strong>value 0</strong> to denote an <i>Ed25519 Signature</i>.
                                        </td>
                                    </tr>
                                    <tr>
                                        <td>Public key</td>
                                        <td>Array&lt;byte&gt;[32]</td>
                                        <td>The public key of the Ed25519 keypair which is used to verify the signature.</td>
                                    </tr>
                                    <tr>
                                        <td>Signature</td>
                                        <td>Array&lt;byte&gt;[64]</td>
                                        <td>The signature signing the Blake2b-256 hash of the serialized <i>Transaction Essence</i>.</td>
                                    </tr>
                                </table>
                            </details>
                        </td>
                    </tr>
                </table>
            </details>
            <details open="true">
                <summary>Reference Unlock Block</summary>
                <blockquote>
                References a previous unlock block in order to substitute the duplication of the same unlock block data for inputs which unlock through the same data.
                </blockquote>
                <table>
                    <tr>
                        <th>Name</th>
                        <th>Type</th>
                        <th>Description</th>
                    </tr>
                    <tr>
                        <td>Unlock Type</td>
                        <td>uint8</td>
                        <td>
                            Set to <strong>value 1</strong> to denote a <i>Reference Unlock Block</i>.
                        </td>
                    </tr>
                    <tr>
                        <td>Reference</td>
                        <td>uint16</td>
                        <td>Represents the index of a previous unlock block.</td>
                    </tr>
                </table>
            </details>
        </td>
    </tr>
</table>

### Transaction Parts

In general, all parts of a <i>Transaction Payload</i> begin with a byte describing the type of the given part to keep the flexibility to introduce new types/versions of the given part in the future.

#### Transaction Essence Data

As described, the <i>Transaction Essence</i> of a <i>Transaction Payload</i> carries the inputs, outputs, and an optional payload. The <i>Transaction Essence</i> is an explicit type and therefore starts with its own <i>Transaction Essence Type</i> byte which is of value 0.

A <i>Transaction Essence</i> must contain at least one input and output.

##### Inputs

The <i>Inputs</i> part holds the inputs to consume, respectively, to fund the outputs of the <i>Transaction Essence</i>. There is only one type of input as of now, the <i>UTXO Input</i>. In the future, more types of inputs may be specified as part of protocol upgrades.

Each defined input must be accompanied by a corresponding <i>Unlock Block</i> at the same index in the <i>Unlock Blocks</i> part of the <i>Transaction Payload</i>.

If multiple inputs can be unlocked through the same <i>Unlock Block</i>, then the given <i>Unlock Block</i> only needs to be specified at the index of the first input which gets unlocked by it.

Subsequent inputs which are unlocked through the same data must have a <i>Reference Unlock Block</i> pointing to the index of a previous <i>Unlock Block</i>.
This ensures that no duplicate data needs to occur in the same transaction.

###### UTXO Input

An <i>UTXO Input</i> is an input which references an output of a previous transaction by using the given transaction's BLAKE2b-256 hash + the index of the output on that transaction. An <i>UTXO Input</i> must be accompanied by an <i>Unlock Block</i> for the corresponding type of output the <i>UTXO Input</i> is referencing.

Example: If the output the input references outputs to an Ed25519 address, then the corresponding unlock block must be of type <i>Signature Unlock Block</i> holding an Ed25519 signature.

##### Outputs

The <i>Outputs</i> part holds the outputs to create with this <i>Transaction Payload</i>.

###### SigLockedSingleOutput

The <i>SigLockedSingleOutput</i> defines an output (with a certain amount) to a single target address which is unlocked via a signature proving ownership over the given address. This output can hold addresses of different types.

###### SigLockedDustAllowanceOutput

The <i>SigLockedDustAllowanceOutput</i> works the same as a <i>SigLockedSingleOutput</i> but additionally controls the dust allowance on the target address. See [Dust Protection RFC](https://github.com/iotaledger/protocol-rfcs/pull/32) for further information.


##### Payload

The payload part of a <i>Transaction Essence</i> can hold an optional payload. This payload does not affect the validity of the <i>Transaction Essence</i>. If the transaction is not syntactically valid, then the payload must also be discarded.

Supported payload types to be embedded into a <i>Transaction Essence</i>:

| Name                  | Type Value |
| --------------------- | ---------- |
| Indexation Payload    | 2          |

#### Unlock Blocks

The <i>Unlock Blocks</i> part holds the unlock blocks unlocking inputs within an <i>Transaction Essence</i>.

There are different types of <i>Unlock Blocks</i>:
<table>
    <tr>
        <td><b>Name</b></td>
        <td><b>Value</b></td>
        <td><b>Description</b></td>
    </tr>
    <tr>
        <td>Signature Unlock Block</td>
        <td>0</td>
        <td>An unlock block holding one or more signatures unlocking one or more inputs.</td>
    </tr>
<tr>
        <td>Reference Unlock Block</td>
        <td>1</td>
        <td>An unlock block which must reference a previous unlock block which unlocks also the input at the same index as this <i>Reference Unlock Block</i>.</td>
    </tr>
</table>

##### Signature Unlock Block

A <i>Signature Unlock Block</i> defines an <i>Unlock Block</i> which holds one or more signatures signing the Blake2b-256 hash of the <i>Transaction Essence</i> (including the optional payload).

##### Reference Unlock block

A <i>Reference Unlock Block</i> defines an <i>Unlock Block</i> which references a previous <i>Unlock Block</i> (which must not be another <i>Reference Unlock Block</i>). It must be used if multiple inputs can be unlocked through the same origin <i>Unlock Block</i>.

Example:
Consider an <i>Transaction Essence</i> containing <i>UTXO Inputs</i> A, B and C, where A and C are both spending the UTXOs originating from the same Ed25519 address. The <i>Unlock Block</i> part must thereby have following structure:

| Index | Must Contain                                                                                                         |
| ----- | ---------------------------------------------------------------------------------------------------------------- |
| 0     | A <i>Signature Unlock Block</i> holding the corresponding Ed25519 signature to unlock A and C.                    |
| 1     | A <i>Signature Unlock Block</i> which unlocks B.                                                                 |
| 2     | A <i>Reference Unlock Block</i> which references index 0, since C also gets unlocked by the same signature as A. |

## Validation

A <i>Transaction Payload</i> payload has different validation stages, since some validation steps can only be executed at the point when certain information has (or has not) been received. We therefore distinguish between syntactical- and semantic validation.

### Syntactical Validation

This validation can commence as soon as the transaction data has been received in its entirety. It validates the structure but not the signatures of the transaction. If the transaction does not pass this stage, it must not be broadcasted further and can be discarded right away.

The following criteria defines whether the transaction passes the syntactical validation:
* `Transaction Essence Type` value must be 0, denoting an `Transaction Essence`.
* Inputs:
  * `Inputs Count` must be 0 < x ≤ 127.
  * At least one input must be specified.
  * `Input Type` value must be 0, denoting an `UTXO Input`.
  * `UTXO Input`:
    * `Transaction Output Index` must be 0 ≤ x < 127.
    * Every combination of `Transaction ID` + `Transaction Output Index` must be unique in the inputs set.
  * Inputs must be in lexicographical order of their serialized form.<sup>1</sup>
* Outputs:
  * `Outputs Count` must be 0 < x ≤ 127.
  * At least one output must be specified.
  * `Output Type` must denote a `SigLockedSingleOutput` or `SigLockedDustAllowanceOutput`.
  * `SigLockedSingleOutput`/`SigLockedDustAllowanceOutput`:
    * `Address Type` must be 0, denoting an `Ed25519` address.
    * The `Address` must be unique in the set of `SigLockedSingleOutputs`/`SigLockedDustAllowanceOutputs` (two separate sets).
    * `Amount` must be > 0.
  * Outputs must be in lexicographical order by their serialized form.<sup>1</sup>
  * Accumulated output balance must not exceed the total supply of tokens `2'779'530'283'277'761`.
* `Payload Length` must be 0 (to indicate that there's no payload) or be valid for the specified payload type.
* `Payload Type` must be one of the supported payload types if `Payload Length` is not 0.
* `Unlock Blocks Count` must match the amount of inputs. Must be 0 < x ≤ 127.
* `Unlock Block Type` must either be 0 or 1, denoting a `Signature Unlock Block` or `Reference Unlock block`.
* `Signature Unlock Blocks` must define a `Ed25519 Signature`.
* A `Signature Unlock Block` unlocking multiple inputs must only appear once (be unique) and be positioned at the same index of the first input it unlocks. All other inputs unlocked by the same `Signature Unlock Block` must have a companion `Reference Unlock Block` at the same index as the corresponding input which points to the origin `Signature Unlock Block`.
* `Reference Unlock Blocks` must specify a previous `Unlock Block` which is not of type `Reference Unlock Block`. The reference index must therefore be < the index of the `Reference Unlock Block`.
* Given the type and length of the information, the <i>Transaction Payload</i> must consume the entire byte array for the `Payload Length` field in the <i>Message</i> it defines.

<sup>1</sup> ensures that serialization of the transaction becomes deterministic, meaning that libraries always produce the same bytes given the logical transaction.

### Semantic Validation

Semantic validation starts when a message that is part of a confirmation cone of a milestone is processed in the [White-Flag ordering](https://github.com/iotaledger/protocol-rfcs/blob/master/text/0005-white-flag/0005-white-flag.md#deterministically-ordering-the-tangle). Semantics are only validated during White-Flag confirmations to enforce an ordering that can be understood by all the nodes (i.e. milestone cones), no matter the order the transactions are received. Otherwise, if semantic validation would occur as soon as a transaction would be received, it could potentially lead to nodes having different views of the UTXOs available to spend.


Processing transactions in the White-Flag ordering enables users to spend UTXOs which are in the same milestone confirmation cone, if their transaction comes after the funding transaction in the mentioned White-Flag ordering. It is recommended that users spending unconfirmed UTXOs attach their message directly onto the message containing the source transaction.

The following criteria defines whether the transaction passes the semantic validation:
1. The UTXOs the transaction references must be known (booked) and unspent.
1. The transaction is spending the entirety of the funds of the referenced UTXOs to the outputs.
1. The address type of the referenced UTXO must match the signature type contained in the corresponding <i>Signature Unlock Block</i>.
1. The <i>Signature Unlock Blocks</i> are valid, i.e. the signatures prove ownership over the addresses of the referenced UTXOs.

If a transaction passes the semantic validation, its referenced UTXOs must be marked as spent and the corresponding new outputs must be booked/specified in the ledger. The booked transaction then also becomes part of the White-Flag Merkle tree inclusion set.

Transactions which do not pass semantic validation are ignored. Their UTXOs are not marked as spent and neither are their outputs booked into the ledger.

## Miscellaneous

### Absent transaction timestamp

A transaction timestamp whether signed or not, does not actually tell when the transaction was issued. Therefore, the timestamp has been left out from the transaction structure. The correct way to determine the issuance time is to use a combination of the solidification and confirmation timestamps of the message embedding the transaction.

### How to compute the balance

Since the ledger is no longer account based, meaning that balances are directly mapped to addresses, computing the balance involves iterating over all UTXOs where their destination address is the address in question and then accumulating their amounts.

### Reusing the same address with Ed25519

While creating multiple signatures with Ed25519 does not reduce security, repeatedly reusing the same address not only drastically reduces the privacy of users but also all other people in the UTXO chain of the moved funds. Applications and services are then instructed to create new addresses per deposit, to circumvent the privacy issues stemming from address reuse.
In essence, Ed25519 support allows for smaller transaction sizes and to safely spend funds which were sent to an already used deposit address.
Ed25519 addresses are not meant to be used like an e-mail address. See this [Bitcoin wiki entry](https://en.bitcoin.it/wiki/Address_reuse#:~:text=The%20most%20private%20and%20secure,a%20brand%20new%20bitcoin%20address.) for further information on how address reuse reduces privacy and this [article](https://en.bitcoin.it/wiki/Receiving_donations_with_bitcoin) on why the same should be applied to donation addresses.

# Drawbacks

The new transaction format is the core data type within the IOTA ecosystem. Changing it means that all projects need to accommodate for it, including client libraries, blueprints, PoC, and applications using IOTA in general. There is no way to keep the changes backwards compatible. Additionally, these changes are breaking, meaning that all nodes must upgrade to further participate in the network.

Additionally, local snapshots can no longer be represented by a list of addresses and their balances, since the ledger is now made up of the UTXOs on which the actual funds reside on. Therefore, local snapshot file schemes have to be adjusted to incorporate the transaction hashes, output indices, and then the destination addresses including the balances.

# Rationale and alternatives

Introducing this new transaction structure allows for further extensions in the future, to accommodate new requirements. With the support for Ed25519 addresses/signatures, transaction size is drastically reduced and allows for safe re-signing in case funds appear to be deposited onto a previous generated address. Due to the switch to a complete binary transaction, size is further reduced, saving network bandwidth and processing time.

Other transaction structures have been considered but they would have misused existing transaction fields to accommodate for new features, instead of putting them into a proper descriptive structure. Additionally, those ideas would not have been safe against replay attacks, which deems reusing the old transaction structure, for example for Ed25519 addresses/signatures, as infeasible.

Not switching to the new transaction structure described in this RFC leads to people being open to loss of funds because of WOTS address re-use and not being able to properly extend the protocol further down the line.

# Unresolved questions

- What parts of the design do you expect to resolve through the RFC process
  before this gets merged?
- What parts of the design do you expect to resolve through the implementation
  of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be
  addressed in the future independently of the solution that comes out of this
  RFC?
