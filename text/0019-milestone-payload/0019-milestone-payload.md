+ Feature name: `milestone-payload`
+ Start date: 2020-07-28
+ RFC PR: [iotaledger/protocol-rfcs#0019](https://github.com/iotaledger/protocol-rfcs/pull/19)
+ Author: Angelo Capossele

# Summary

In IOTA, nodes use the milestones issued by the Coordinator to reach a consensus on which transactions are confirmed. This RFC proposes a milestone payload for the messages described in [Draft RFC-17](https://github.com/GalRogozinski/protocol-rfcs/blob/message/text/0017-message/0017-message.md). It uses Edwards-curve Digital Signature Algorithm (EdDSA) to authenticate the milestones.

# Motivation

In the current IOTA protocol, milestones are authenticated using a ternary Merkle signature scheme. With [Chrysalis](https://roadmap.iota.org/chrysalis), ternary transactions will be replaced with binary messages containing different payload types. In order to address these new requirements, this RFC proposes the use of a dedicated payload type for milestones. It contains the same essential data fields that were previously included in the milestone bundle. Additionally, this document also describes how Ed25519 signatures are used to assure authenticity of the issued milestones. In order to make the management and security of the used private keys easier, simple multisignature features with support for key rotation have been added.

# Detailed design

The _Milestone Essence_, consisting of the actual milestone information (like its index number or position in the tangle), is signed using the Ed25519 signature scheme as described in the IRTF [RFC 8032](https://tools.ietf.org/html/rfc8032). It uses keys of 32 bytes, while the generated signatures are 64 bytes.

To increase the security of the design, a milestone can (optionally) be independently signed by multiple keys at once. These keys should be operated by detached signature provider services running on independent infrastructure elements. This assist in mitigating the risk of an attacker having access to all the key material necessary for forging milestones. While the Coordinator takes responsibility for forming Milestone Payload Messages, it delegates signing in to these providers through an ad-hoc RPC connector. Mutual authentication should be enforced between the Coordinator and the signature providers: a [client-authenticated TLS handshake](https://en.wikipedia.org/wiki/Transport_Layer_Security#Client-authenticated_TLS_handshake) scheme is advisable. To increase the flexibility of the mechanism, nodes can be configured to require a quorum of valid signatures to consider a milestone as genuine.

In addition, a key rotation policy can also be enforced by limiting key validity to certain milestone intervals. Accordingly, nodes need to know which public keys are applicable for which milestone index. This can be provided by configuring a list of entries consisting of the following fields:
- _Index Range_ providing the interval of milestone indices for which this entry is valid. The interval must not overlap with any other entry.
- _Applicable Public Keys_ defining the set of valid public keys.
- _Signature Threshold_ specifying the minimum number of valid signatures. Must be at least one and not greater than the number of _Applicable Public Keys_.

## Structure

All values are serialized in little-endian encoding. The serialized form of the milestone is deterministic, meaning the same logical milestone always results in the same serialized byte sequence.

The following table structure describes the entirety of a _Milestone Payload_ in its serialized form ([Data Type Notation](https://github.com/GalRogozinski/protocol-rfcs/blob/message/text/0017-message/0017-message.md#data-types)):

<table>
  <tr>
    <th>Name</th>
    <th>Type</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Payload Type</td>
    <td>uint32</td>
    <td>Set to <strong>value 1</strong> to denote a <i>Milestone Payload</i>.</td>
  </tr>
  <tr>
    <td valign="top">Essence <code>oneOf</code></td>
    <td colspan="2">
      <details open="true">
        <summary>Milestone Essence</summary>
        <blockquote>Describes the signed part of a <i>Milestone Payload</i>.</blockquote>
        <table>
          <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
          </tr>
          <tr>
            <td>Index Number</td>
            <td>uint32</td>
            <td>The index number of the milestone.</td>
          </tr>
          <tr>
            <td>Timestamp</td>
            <td>uint64</td>
            <td>The Unix timestamp at which the milestone was issued. The unix timestamp is specified in seconds.</td>
          </tr>
          <tr>
            <td>Parents' Length</td>
            <td>uint8</td>
            <td>The number of messages we directly approve. Can be any value between 1-8.</td>
          </tr>
          <tr>
            <td>Parents</td>
            <td>ByteArray[32 * Parents' Length]</td>
            <td>The Message IDs of the <i>Messages</i> referenced by the milestone.</td>
          </tr>
          <tr>
            <td>Inclusion Merkle Root</td>
            <td>ByteArray[32]</td>
            <td>256-bit hash based on the message IDs of all the not-ignored state-mutating transactions referenced by the milestone. (<a href="https://github.com/iotaledger/protocol-rfcs/blob/milestone-merkle-validation-chrysalis-pt-2/text/0012-milestone-merkle-validation/0012-milestone-merkle-validation.md">Update RFC-0012</a>)</td>
          </tr>
          <tr>
            <td>Next PoW Score</td>
            <td>uint32</td>
            <td>The new PoW score all messages should adhere to. If 0 then the PoW score should not change. See <a href="https://github.com/Wollac/protocol-rfcs/blob/message-pow/text/0024-message-pow/0024-message-pow.md">RFC-0024</a>.</td>
          </tr>
          <tr>
          <td>Next PoW Score Milestone Index</td>
            <td>uint32</td>
            <td>The index of the first milestone that will require a new minimal pow score for applying transactions. This field comes into effect only if the `Next PoW Score` field is non 0.</td>
          </tr>
          <tr>
            <td>Keys Count</td>
            <td>uint8</td>
            <td>Number of public keys entries.</td>
          </tr>
          <tr>
            <td>Public Keys</td>
            <td>ByteArray[32 * Keys Count]</td>
            <td>An array of public keys to validate the signatures. The keys must be in lexicographical order.</td>
          </tr>
        </table>
      </details>
    </td>
  </tr>
  <tr>
    <td>Signatures Count</td>
    <td>uint8</td>
    <td>Number of signature entries. The number must match the field <code>Keys Count</code>.</td>
  </tr>
  <tr>
    <td>Signatures</td>
    <td>Array&lt;ByteArray[64]&gt;</td>
    <td>An array of signatures signing the serialized <i>Milestone Essence</i>. The signatures must be in the same order as the specified public keys.</td>
  </tr>
</table>

## Generation

- Generate a new _Milestone Essence_ corresponding to the Coordinator milestone.
- Transmit the serialized _Milestone Essence_ to the corresponding number of signature service providers.
  - The signature provider service will sign the received serialized bytes as-is.
  - The signature provider will serialize the signature bytes and return them to the Coordinator.
- Fill the `Signatures` field of the milestone payload with the received signature bytes.
- Generate a *Message* as defined in [RFC-0017 (draft)](https://github.com/GalRogozinski/protocol-rfcs/blob/message/text/0017-message/0017-message.md) using the same `Parents` for the created _Milestone Payload_.

## Syntactical validation

- `Parents` of the payload must match `Parents` of the encapsulating _Message_.
- `Next PoW Score Milestone Index` should be larger than the current milestone index if `Next Pow Score` is different than 0. Else, it should be 0.
- `Keys Count` must be at least the _Signature Threshold_ and at most the number of _Applicable Public Keys_ for the current milestone index.
- `Public keys`:
  - The provided keys must form a subset of the _Applicable Public Keys_ for the current milestone index.
  - The keys must be unique.
  - The keys must be in lexicographical order.
- `Signatures Count` must match the amount of public keys. 
- All `Signatures` must be valid.
- Given the type and length information, the _Milestone Payload_ must consume the entire byte array of the `Payload` field of the _Message_.

# Rationale and alternatives

- Instead of using EdDSA we could have chosen ECDSA. Both algorithms are well supported and widespread. However, signing with ECDSA requires fresh randomness while EdDSA does not. Especially in the case of milestones where essences are signed many times using the same key, this is a crucial property.
- Due to the layered design of messages and payloads, it is practically not possible to prevent reattachments of milestone payloads. Hence, this payload has been designed in a way to be independent from the message it is contained in. A milestone should be considered as a virtual marker (referencing `Parents`) rather than an actual message in the Tangle. This concept is compatible with reattachments and supports a cleaner separation of the message layers.

# Unresolved questions

- Forcing matching `Parents` in the _Milestone Payload_ and its _Message_ makes it impossible to reattach the same payload at different positions in the Tangle. While this does not prevent reattachments in general (a different, valid `Nonce`, for example would lead to a new Message ID), this still simplifies milestone processing. However, it violates a clear separation of payload and message. As such, it might still be desirable to slightly complicate the milestone processing by allowing arbitrary `Parents` field. This separates the two layers completely and should not have any impact on the actual milestone properties.
