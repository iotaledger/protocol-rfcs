+ Feature name: dust-protection
+ Start date: 2020-12-07
+ RFC PR: [iotaledger/protocol-rfcs#0032](https://github.com/iotaledger/protocol-rfcs/pull/0032)


# Summary
In the UTXO model, each node in the network needs to keep track of all the currently unspent outputs. When the number of outputs gets too large, this can cause performance and memory issues.
This RFC proposes a new protocol rule regarding the processing of outputs that transfer a very small amount of IOTA, so-called dust outputs: Dust outputs are only allowed when they are backed up by a certain deposit on the receiving address. This limits the amount of dust outputs, thus making it expensive to proliferate dust. Since a receiver must make a deposit, the protocol makes receiving dust an opt-in feature.

# Motivation

An attacker, or even honest users, can proliferate the UTXO set with outputs holding a tiny amount of iotas. This can cause the UTXO set to grow to a prohibitively large size. 
Nodes may freeze or crash due to the increasing amount of memory and computational resources needed.

In order to protect nodes from such attacks, one possible solution is to make accumulating dust outputs expensive. Since IOTA does not have any fees that might limit the feasibility of issuing many dust transactions, deposits pose a valid alternative to achieve a similar effect.

When an address is supposed to receive micro transactions, it must have an unspent output of a special type as a deposit. This deposit cannot be consumed by any transaction as long as the dust outputs remain unspent.

An additional benefit of this rule is that it makes a mass of privacy violating [forced address reuse attacks](https://en.bitcoin.it/wiki/Privacy#Forced_address_reuse) more expensive to carry out.


# Detailed design

### Definitions

*Dust output*: A transaction output that has an amount smaller than 1 Mi

*SigLockedDustAllowanceOutput*: A new output type for deposits that enables an address to receive dust outputs. It can be consumed as an input like a regular `SigLockedSingleOutput`.

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
    <td valign="top">Address</td>
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
              Set to <strong>value 1</strong> to denote an <i>Ed25519 Address</i>.
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
    <td>The amount of tokens to deposit with this <i>SigLockedDustAllowanceOutput</i> output.</td>
  </tr>
</table>


### Validation

Let A be the address that should hold the dust outputs' balances. Let S be the sum of all the amounts of all unspent `SigLockedDustAllowanceOutputs` on A. Then, the maximum number of allowed dust outputs on A is S divided by 100,000 and rounded down, i.e. 10 outputs for each 1 Mi deposited.
However, regardless of S, the number of dust outputs must never exceed 100 per address.

The amount of a `SigLockedDustAllowanceOutput` must be at least 1 Mi. Apart from this, `SigLockedDustAllowanceOutputs` are processed identical to `SigLockedSingleOutput`. The transaction validation as defined in [Draft RFC-18](https://github.com/luca-moser/protocol-rfcs/blob/signed-tx-payload/text/0000-transaction-payload/0000-transaction-payload.md), however, needs to be adapted.

_Syntactical validation_ for `SigLockedDustAllowanceOutput`:
- The `Address` must be unique in the set of `SigLockedDustAllowanceOutputs` in one transaction T. However, there can be one `SigLockedSingleOutput` and one `SigLockedDustAllowanceOutputs` T.
- The `Amount` must be ≥ 1,000,000.

The _semantic validation_ remains unchanged and are checked for both `SigLockedSingleOutputs` and `SigLockedDustAllowanceOutput`, but this RFC introduces one additional criterion:

A transaction T
  - consuming a `SigLockedDustAllowanceOutput` on address A **or**
  - creating a dust output with address A,

is only semantically valid, if, after T is booked, the number of unspent dust outputs on A does not exceed the allowed threshold of min(S / 100000, 100).

# Drawbacks

- There can no longer be addresses holding less than 1 Mi.
- A service receiving micropayments may fail receiving them, if it did not consolidate dust outputs or raised the deposit for the receiving address.

# Rationale and alternatives

The rationale for creating a special `SigLockedDustAllowanceOutput` rather than rely on the default `SigLockedSingleOutputs` is to prevent attackers from polluting arbitrary addresses that happen to hold
a large amount of funds with dust.

One may note that an attacker can deposit a dust allowance on 3rd party address outside his control and pollute that address with dust.
From a security perspective this is better than an attacker depositing a dust allowance on addresses under his control.
This is because the receiving party might later choose to consolidate the dust outputs and hence relief UTXO memory consumption.
The receiving party is also unlikely to be displeased from obtaining more funds, small as they may be.

There are potential alternatives to introducing dust allowance deposits:

- *Burning dust*: Allow dust outputs to exists only for a limited amount of time in the ledger. After this, they are removed completely and the associated funds are invalidated.
- *Sweeping dust into Merkle trees*: Instead of burning dust outputs after some time, they are instead compressed into a Merkle tree and only the tree root is kept. In order to spend one of these compressed outputs, the corresponding Merkle audit path needs to be supplied in addition to a regular signature.

The first option can cause issues, when dust outputs were burned before users could consolidate them. Also, changing the supply can be controversial.

The second option is much more complicated as it introduces a completely new unlock mechanisms and requires the nodes to store the Merkle tree roots indefinitely.


# Unresolved questions

- An attacker can send microtransactions to an address with a `SigLockedDustAllowanceOutput` in order to fill the allowed threshold and block honest senders of microtransactions. The owner of the address can mitigate this by simply consolidating the attacker's dust and collecting it for profit. The problem is that the cost of doing PoW too often may exceed the profit made by collecting dust. Perhaps it is a good idea to limit "tiny" (1 i) dust outputs more than larger dust outputs? In the original discussion each dust class was supposed to be capped to a certain fixed amount. But perhaps we should give weights to different dust classes. So "tiny" dust will weigh more and fill the cap more quickly than larger dust?
