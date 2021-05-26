+ Feature name:`ed25519-validation`
+ Start date: 2020-10-30
+ RFC PR: [iotaledger/protocol-rfcs#0028](https://github.com/iotaledger/protocol-rfcs/pull/28)

# Summary

The IOTA protocol uses Ed25519 signatures to assure the authenticity of transactions in Chrysalis. However, although Ed25519 is standardized in [RFC 8032](https://tools.ietf.org/html/rfc8032), it does not define strict validation criteria. As a result, compatible implementations do not need to agree on whether a particular signature is valid or not. While this might be acceptable for classical message signing, it is unacceptable in the context of consensus critical applications like IOTA.

This RFC proposes to adopt [ZIP-215](https://zips.z.cash/zip-0215) to explicitly define validation criteria. This mainly involves the following sections of the Ed25519 spec:
- decoding of elliptic curve points as described in [Section 5.1.3](https://tools.ietf.org/html/rfc8032#section-5.1.3)
- validation itself as described in [Section 5.1.7](https://tools.ietf.org/html/rfc8032#section-5.1.7)

# Motivation

Based on [research done by Henry de Valence](https://hdevalence.ca/blog/2020-10-04-its-25519am) we know that: 
1. Not all implementations follow the decoding rules defined in RFC 8032, but instead accept non-canonically encoded inputs.
2. The Ed25519 RFC provides two alternative verification equations, whereas one is stronger than the other. Different implementations use different equations and therefore validation results vary even across implementations that follow the RFC 8032. 

This lack of consistent validation behavior is especially critical for IOTA as they can cause a breach of consensus across node implementations! For example, one node implementation may consider a particular transaction valid and mutate the ledger state accordingly, while a different implementation may discard the same transaction due to invalidity. This would result in a network fork and could only be resolved outside of the protocol. Therefore, an explicit and unambiguous definition of validation criteria, such as ZIP-215, is necessary.

Furthermore, it is important to note that the holder of the secret key can produce more than one valid distinct signature. Such transactions with the same essence but different signatures are considered as double spends by the consensus protocol and handled accordingly. While this does not pose a problem for the core protocol, it may be a problem for 2nd layer solutions, similar to how [transaction malleability in bitcoin presented an issue for the lightning network](https://en.bitcoinwiki.org/wiki/Transaction_Malleability#How_Does_Transaction_Malleability_Affect_The_Lightning_Network.3F).

# Detailed Design

In order to have consistent validation of Ed25519 signatures for all edge cases and throughout different implementations, this RFC proposes explicit validation criteria. These three criteria **must** be checked to evaluate whether a signature is valid.

Using the notation and Ed25519 parameters as described in the RFC 8032, the criteria are defined as follows:

1. Accept non-canonical encodings of A and R.
2. Reject values for S that are greater or equal than L.
3. Use the equation [8][S]B = [8]R + [8][k]A' for validation.

In the following, we will explain each of these in more detail.

## Decoding

The Curve25519 is defined over the finite field of order p=2<sup>255</sup>−19. A curve point (x,y) is encoded into its compressed 32-byte representation, namely by the 255-bit encoding of the field element y followed by a single sign bit that is 1 for negative x and 0 otherwise. This approach provides a unique encoding for each valid point. However, there are two classes of edge cases representing non-canonical encodings of valid points:
- encoding a y-coordinate as y + p
- encoding a curve point (0,y) with the sign bit set to 1

In contrast to the Section [Decoding](https://tools.ietf.org/html/rfc8032#section-5.1.3) of RFC 8032, it is _not_ required that the encodings of A and R are canonical. As long as the corresponding (x,y) is a valid curve point, any of such edge cases will be accepted.

It is worth noting that due to allowing different encodings of the same point, one cannot check point equality by doing byte per byte comparisons.

## Validation 

The RFC 8032 mentions two alternative verification equations:
1. [8][S]B = [8]R + [8][k]A'
2. [S]B = R + [k]A'

Each honestly generated signature following RFC 8032 satisfies the second, cofactor-less equation and thus, also the first equation. However, the opposite is not true: There are solutions only satisfying the first but not the latter.<br> This ambiguity in RFC 8032 has led to the current situation in which different implementations rely on different verification equations. 

In order to be consistent with batched verification, the group equation [8][S]B = [8]R + [8][k]A' _must_ be used for validations instead of [S]B = R + [k]A'.

## Malleability

The non-negative integer S is encoded into 32 bytes as part of the signature. However, a third party could replace S with S' = S + n·L for any natural n with S' < 2<sup>256</sup> and the modified signature R || S' would still pass verification. Requiring a value less than L resolves this malleability issue. Unfortunately, this check is not present in all common Ed25519 implementations.

Analogous to RFC 8032, the encoding of S _must_ represent an integer less than L.

It is not possible for an external party to mutate R and still pass verification. The owner of the secret key, however, can create many different signatures for the same content: While Ed25519 defines a deterministic method of calculating the integer scalar r from the private key and the message, it is impossible to tell during signature verification if the point R = [r]B was created properly or any other scalar has been used.<br> As a result, there is a practically countless amount of different valid signatures corresponding to a certain message and public key.

# Drawbacks

- Allowing non-canonical encodings is a direct contradiction of RFC 8032 and rather unintuitive. Furthermore, it introduces alternative encodings for a handful of points on the curve. Even though such points will, for all practical purposes, never occur in honest signatures, it still theoretically introduces an external party malleability vector.
- The cofactor validation is computationally slightly more expensive than the cofactor-less version since it requires a multiplication by 8.

# Rationale and alternatives

While the malleability of S poses serious issues and thus, must be prevented, the other two validation criteria, namely non-canonical encodings as well as the cofactor-less validation equation, could be relaxed without introducing attack vectors.

Unfortunately, the Ed25519 `ref10` reference implementation as well as other implementations accept non-canonical points. As such, rejecting those inputs now would introduce a breaking change. While this might be acceptable for the IOTA protocol itself, since no Ed25519 signatures have been added to the ledger prior to this RFC, other consensus-critical applications require this backward compatibility with previously accepted signatures. Due to these considerations, the criterion was included in ZIP-215 to allow a seamless transition for existing consensus-critical contexts. This RFC aims to rather follow the existing ZIP-215 specification for compatibility and maintainability than to create a new standard.

Using the cofactor-less validation poses a similar breaking change since signatures accepted by implementations using the cofactor validation would then be rejected. More importantly, however, in order to be able to use the much faster batch verification, the cofactor version is required. 

# Unresolved questions

- Are there issues resulting from the malleability introduced by non-canonical encodings of specially prepared public keys? 
- Accepting non-canonical encodings of R allows 3rd party malleability of signatures, which is something that was previously explicitly prevented. Even though this should not effect honest signatures, is that really acceptable? 
