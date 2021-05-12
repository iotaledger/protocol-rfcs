+ Feature name: `colored-coins`
+ Start date: 2021-02-18
+ RFC PR: [iotaledger/protocol-rfcs#0034](https://github.com/iotaledger/protocol-rfcs/pull/0034)

# Summary

Colored coins are IOTA coins associated with a special color, that is a globally unique collection of bytes within the Tangle.

 - Regular IOTA coins are spent to mint colored coins. 1 IOTA corresponds to 1 newly minted Colored Coin. The amount of
   coins to mint is defined in the minting output(s) of a transaction. Once minted, the color can't be changed, and no
   more new coins can be minted with the same color.
 - Colored coins can be transferred just like IOTA coins, without transaction fees.
 - Colored coins can be destroyed  (uncolored), which essentially means that the coins lose their unique color
   attribute and become regular IOTA coins.

# Motivation

The possibility to mark a set of coins with certain information is a consequence of the UTXO model introduced in
Chrysalis Phase 2. In a UTXO output, next to the amount of coins, additional tagging information can be embedded, such
as color.

Tagging coins makes it possible to associate certain IOTA coins with real world assets, like bonds, stocks, property
rights or loyalty points. Allowing real world assets to be represented by digital tokens that are secure, transferable,
feeless and fast, opens up new use cases for the IOTA Protocol.

## Use Cases

The sections below describe some, but far from all possible use cases of IOTA colored coins. It will be up to the
community and industry to find further innovative ways of utilizing the new features of the protocol.

### Utility Tokens
A utility token provides benefit to its owner by allowing them to access goods, services or products via the use of a
token. A paper based concert ticket, gift cards one collects at a coffee shop or a monthly public transportation pass
are all examples of tokens. They let one verify that they indeed have the right to claim their benefits.

Colored coins can provide the same functionality in a secure, digital world with added perks:
- Tokens can't be counterfeited.
- Tokens are held in the digital wallet of the user, the IOTA Tangle provides a decentralized administrative framework
  to distribute, transfer and manage tokens.
- Anyone can create, transfer or hold tokens in a permissionless way.
- Colored coins inherit the feeless nature of IOTA, transferring a colored coin has no costs.
- Issuers must bear the cost of acquiring the IOTA tokens to be colored. With current market price of $1.30 of MIOTA
  (17/02/2021), an issuer must spend $1.30 to buy IOTA coins for minting 1 million colored coins.
- Utility tokens can be converted to regular IOTA tokens. This is beneficial in 2 ways:
    - Once users exchange the utility tokens for a service, the service provider can easily convert the tokens to IOTA
      or re-color them to be used in another service.
    - Users holding the utility tokens can also convert them to IOTA at any point in time, although this is only
      rational behavior if the face value of the utility token in IOTA is more, than the service they are redeemable for.
- Utility tokens can be easily integrated into Firefly wallet.
- The issuer can easily monitor the distribution, usage and available supply of the utility token through examining the
  IOTA Tangle.

To summarize, utility tokens on IOTA provide a secure, convenient and cheap way to sell services, organize loyalty
programs and digitize paper based token systems.

### Security Tokens
Security tokens refer to digital tokens that represent financial assets that fall under the category of
[securities](https://en.wikipedia.org/wiki/Security_(finance)). These are tradable financial instruments that are
heavily regulated, under varying conditions based on the jurisdiction of the issuer's country.

There is a huge potential in the industry of security tokens, because they address the shortcomings of traditional securities:
- Regulatory compliance (KYC, AML, etc.) can be enforced through underlying protocol.
- Both regulators and issuers have real time data on how securities are allocated and traded in the market.
- Traditionally illiquid assets such as real estate or art can be tokenized, enabling better price discovery,
  fractional ownership and more liquidity.
- The cost of managing securities falls drastically, as the protocol can act as the managing entity.
- A security token can be traded on secondary markets 24/7.
- Security tokens can reduce the entry barriers for smaller or retail investors.

It is important to note, that a security token itself is just a claim to the underlying security, it is not the asset itself.

One of the biggest technological challenges of security token frameworks built on top of distributed ledgers is how to
enforce regulatory compliance. Smart contract platforms are well suited to solve this problem by employing autonomous
agents (smart contracts) on the base layer to manage the life cycle of security tokens.

IOTA has its smart contract solution (ISCP) on layer 2, because the base layer of the IOTA Protocol has to remain
lightweight enough to cater to the needs of the machine economy.
ISCP is suitable for advanced security tokenization, where all features of a security token can be programmed and
enforced by the smart contract itself. An issuer can control supply, regulate trading, or might even reverse transactions
if demanded by law. Of course, this is not possible on layer 1, as it would add too much complexity to the protocol.

Although, there is another way to carry out regulatory compliant security tokenization on layer 1, with the help of
colored coins. An issuer might mint and distribute security tokens as colored coins to multi-party signature addresses.
- To unlock funds, both the issuer and the user has to sign the transaction.
- Transfers to addresses that are not verified would fail to gather the signature of the issuer.
- The issuer can halt trading and enforce compliance rules since it knows the verified identities of users.
- The security tokens are still handled in a secure, decentralized and feeless infrastructure, the IOTA Tangle.

In conclusion, colored coins provide the opportunity for security tokenization on the base layer with trusted issuer
platforms acting as guides, but also enable layer 2 solutions like ISCP that support full scale, customizable,
self-enforcing rules to be implemented as autonomous agents.

### Non-fungible Tokens

Digital tokens can be classified into two categories based on their nature of fungibility:
- **Fungible Tokens** are interchangeable with other tokens of the same type, as they represent the same amount of
  value. Cash or IOTA is fungible, because a $10 bill can be exchanged for two $5 bills, and 10 MIOTA is equivalent
  to 5 MIOTA + 5 MIOTA.
- **Non-fungible Tokens** on the other hand are not interchangeable, because each token has unique properties that give
  them their value.

A Non-fungible Token (NFT) can represent an artwork, a collectible or special edition item. It is essentially a digital
property right that can be traded. Some exciting use cases are:
- Crypto collectibles such as crypto kitties, fantasy football league cards, etc.
- In-game items that can be traded on secondary markets.
- Digital artwork recorded on the ledger.
- NFTs can also be used as collateral in decentralized finance.

Colored coins support on the IOTA Protocol would finally introduce the capability of creating NFTs on layer 1. A color
is a globally unique property of a minted token, therefore, if there is only one token ever minted with that particular
color, it becomes an NFT. Of course, it is fungible in the sense that it can be uncolored and used as a regular IOTA
token, but no other coin can be created with the same color, so it is a true NFT until the owner decides to destroy it.

An NFT can also be viewed as a "global identity token" that may have data associated with it.
A non-forkable (non-copyable) chain of data states can be built by attaching metadata to the transfers of the NFT on
layer 1. Therefore, it is perfectly suited for use cases such as Access, Streams or DID.

It is important to note, that a colored coin used as an NFT has only one unique property: its color. Therefore,
advanced use cases, where a myriad of unique properties have to be associated to a single token are not supported on
layer 1, rather via ISCP on layer 2. However, if the NFT represents a property right, the original attributes of the
property could still be stored off-tangle, in a distributed registry, or in a registry smart contract.

# Detailed design

To represent colored coins in the IOTA Protocol, all output types must be able to deal with them. Colored coins therefore
are not represented as a distinct output type, rather, they are part of all output types.
The current `SigLockedSingleOutput` shall be transformed into `SigLockedOutput`:

   <summary>SigLockedOutput</summary>
    <blockquote>
    Describes a transfer to a single address which is unlocked via a signature.
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
                Set to <strong>value 0</strong> to denote a <i>SigLockedOutput</i>.
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
                            <td>Array&lt;byte&gt;\[32\]</td>
                            <td>The raw bytes of the Ed25519 address which is a BLAKE2b-256 hash of the Ed25519 public key.</td>
                        </tr>
                    </table>
                </details>
            </td>
        </tr>
        <tr>
            <td>Balances Count</td>
            <td>uint8</td>
            <td>
                The number of <i>ColoredBalances</i> in a <i>SigLockedOutput</i>.
            </td>
        </tr>
        <tr>
            <td valign="top">Balances <code>anyOf</code></td>
            <td colspan="2">
				<details>
                    <summary>ColoredBalance</summary>
                    <table>
                        <tr>
                            <td><b>Name</b></td>
                            <td><b>Type</b></td>
                            <td><b>Description</b></td>
                        </tr>
                        <tr>
                            <td>Color</td>
                            <td>Array&lt;byte&gt;\[32\]</td>
                            <td>
                                The color of the tokens. `ColorIOTA` denotes IOTA tokens.
                            </td>
                        </tr>
                        <tr>
                            <td>Amount</td>
                            <td>uint64</td>
                            <td> The amount of tokens of this <i>ColoredBalance</i> output.</td>
                        </tr>
                    </table>
                </details>
            </td>
    </table>

<p></p>

There are couple new concepts as compared to a `SigLockedSingleOutput`.

## Color

Colors are globally unique and are assigned to tokens upon minting. The value of the color field is generated from the
ID of the output that minted it. The `outputID` consists of 34 bytes: 32 bytes of `TransactionID`, and 2 bytes to
denote the index of the output within the transaction. An `OutputID` therefore is a unique value in the tangle.
When minting colored coins with an output, the color of the tokens is defined as the BLAKE2b-256 hash of the `OutputID`.

The zero value of a color (32 bytes that are all zero) has a distinct name, it is called `ColorIOTA`, and defines the
color of an IOTA token.

A special color, `ColorMint` indicates that an output wishes to create new colored coins. `ColorMint` is defined as 32
bytes that are all maxed out, 256 bits that are all 1s.

## Colored Balance

The balance of an output is encoded as an ordered unique collection of Colored Balances. A Colored Balance describes the
color and the amount of the tokens. Colored Balances are sorted lexicographically based on their color value.
Therefore, in any output, `ColorIOTA` balance is always the first if present, and `ColorMint` balance is always the
last if present.

Each color can only be present in `Balances` once, duplicates are not allowed, even if they describe different amounts.

## Transaction Validation

Previously, the only thing a transaction could do was to move tokens from an address to another address. With the
introduction of colored coins, a transaction might mint colored coins or might uncolor them. Transaction validation
rules have to be adjusted to implicitly handle the two new operations.

### Syntactical Validation

 - `Balances Count` defines the number of Colored Balances in the output. There should be no trailing bytes left in the
   serialized output after parsing `Balances Count` `Colored Balances`. If the bytes of serialized output are exhausted
   before parsing `Balances Count` `Colored Balances`, the output and the transaction is invalid.
- `Balances` must be sorted lexicographically based on their color, otherwise the output and the transaction is invalid.
- A `Colored Balance` must have `Amount` greater than zero.

### Semantic Validation

Unlocking doesn't change, the new output type is also locked by a signature.
The same unlock rules apply to a `SigLockedOutput` as to a `SigLockedSingleOutput`.

Before colored coins, a transaction was considered invalid if it spent more funds in the created outputs, than it consumed
from the unlocked inputs. With the introduction of colors and colored balances however, the amount of different colors
and their operation also has to be taken into account.

The two sides of the transaction (inputs and outputs) define a balance sheet with positive and negative sides for
different "colors". Consuming an input adds to the positive side, while creating a new output incurs a negative balance.
The sum of the two side of the balance sheet must be zero.

**Semantic Rules for consuming inputs (previously created outputs):**
- An unlocked `SigLockedOutput` adds all of its `Colored Balances` to the positive side of the balance sheet.

**Semantic Rules for creating new outputs:**
- A newly created `SigLockedOutput` adds all of its `Colored Balances` to the negative side of the balance sheet.

If the transaction minted or uncolored tokens, the balance sheet is no longer balanced, because either new colors
were created or colors were destroyed, so they are only present on one side. The following rules apply to the balance
sheet:
 - To mint new colors, the created output has a `Colored Balance` entry with `color=ColorMint`. There is a surplus of
   `ColorMint` tokens on the negative side of the balance sheet, which can be balanced by any remaining balances in the positive
   side. This means that new colored coins can not only be created from `ColorIOTA` tokens, but from any colored coins.
   The latter case is considered re-coloring.
 - To uncolor coins, the newly created output has a `Colored Balance` entry with `color=ColorIOTA`. There is a surplus
   of `ColorIOTA` tokens on the negative side of the balance sheet, which can be balanced by remaining balances of
   the uncolored color on the positive side.


Pseudo-code for semantic transaction balances validation:
```
consumedInputsBalances: map Color -> Amount

for all consumed inputs in tx:
    for all colored balances in input:
        consumedInputsBalances[color] += amount

recoloredCoins = 0

for all outputs in tx:
    for all colored balances in output:
        if color is ColorMint or ColorIOTA:
            recoloredCoins += amount
        else if color not present in consumedInputsBalances:
            return TransactionBalancesInvalid
        else:
            consumedInputsBalances[color] -= amount

unspentCoins = 0

for all remaining colored balances in consumedInputsBalances:
    unspentCoins += balance

if recoloredCoins does not equal unspentCoins, transaction balances are invalid
```

Implementation of above algorithm should pay great attention to buffer overflows.


### Post Processing

Previously, post processing only meant calculating the `OutputID` of the newly created outputs by taking the
`Transaction ID` + `index of the output in the transaction`, and booking them into the ledger.

With colored coins, an additional step has to be introduced for newly minted colors in outputs. The minting operation is
carried out by putting the special `ColorMint` colored balance into the output. This colored balance is replaced with
the actual color before booking the output into the ledger. The color is calculated as the BLAKE2b-256 hash of the
`OutputID`.

### Notes
- Outputs in the serialized transaction must be in lexicographic order, therefore there can be no two transactions that
  define the same UTXO mutations while having different transaction IDs.
- One transaction might mint several coins with unique colors. The maximum amount of different colors that can be
  minted in a single transaction is exactly the maximum allowed number of outputs, that is `127`.
- One transaction might uncolor several colored coins. The theoretical maximum amount of different colors that can be
  uncolored in a single transaction is the maximum allowed number of inputs times the maximum number of balances.
  If all balances in all consumed outputs hold different colors, 127 * 255 = 32385 different colors could be uncolored,
  without taking into account the maximum allowed transaction size. The size of one output with 255 colored balances is
  1 + 33 + 1 + 255 * 40 = 10235 bytes. 4 of such outputs is already [too big to fit in a message](https://github.com/GalRogozinski/protocol-rfcs/blob/message/text/0017-message/0017-message.md#message-validation).
- It is not possible to mint a color in a transaction and uncolor it in the same transaction, since uncoloring can only
  happen if coins with that particular color are present in list of unlocked inputs.
- The size of a `SigLockedOutput` with 1 `ColorIOTA` balance is exactly `75 bytes`, compared to the `42 bytes` size of a
  `SigLockedSingleOutput`.

From the last note it seems that `SigLockedOutput` is less efficient in encoding normal IOTAs. To spare most importantly
bandwidth, the serialized form of a transaction can encode colors into a dictionary that is part of the transaction
essence. Whenever a color is referenced in an output of the transaction, instead of the 32 bytes of the color, and index
is used to reference the color from the color dictionary. `ColorIOTA` and/or `ColorMint` could have their reserved
indices in the dictionary, so they can be encoded in less, than 32 bytes.

### Dust protection

Dust protection refers to preventing the unnecessary increase of the ledger database by splitting up funds into several,
smaller outputs. Chrysalis Phase 2 introduced a new dust protection mechanism, that defines on a protocol level when
such small outputs can be accepted.
Colored coins are just tagged IOTA tokens on protocol level. Therefore, any dust protection solution for IOTA tokens
shall be the same as for colored coins.
Dust protection doesn't change with the introduction of colored coins, a `SigLockedOutput` is considered to be
dust if it has a cumulative balance that is less, than `1 Mi`.

Note, that it is possible to mint colored coins into outputs that have other balances as well, so an NFT minted into an
output that has already `1Mi` `ColorIOTA` balance is not considered dust. It is also possible to send the NFT together
with the `1Mi` to a new owner, so it is not considered as dust.

A business selling digital goods as colored coins can for example require the buyers to pay extra `1Mi` for the service,
which will be returned to them in the transfer that sends the buyers their colored coins. The user then doesn't have to
set up a dust allowance output. If the colored coin is distributed from a smart contract in ISCP, it can automatically
make sure to only accept requests that include this dust prevention component.

# Drawbacks

- Increases the complexity of the transaction validating logic, however, it does not introduce additional signature
  validation overhead.
- Adds additional post processing step for minting operations.
- All client libraries have to be modified to handle colored coins and the new output type.

# Rationale and alternatives

#### What other designs have been considered and what is the rationale for not choosing them?
The alternative ways of supporting colored coins is to build a scripting framework into the protocol, like bitcoin does,
or try to design an [Extended UTXO](https://iohk.io/en/research/library/papers/the-extended-utxo-model/) model. Both of
them can support a lot more functionality on protocol level, but also further increase the transaction validation logic.
The IOTA protocol has to remain lightweight enough to be the backbone of the machine economy.
### Why is this design the best in the space of possible designs?
Due to the implicit minting and uncoloring operations, colored coins become orthogonal to the output type. Any future
output type can support colored coins by having its balance encoded as Colored Balances.
The NFT capabilities of single supply colored coins makes it possible to register non-forkable data states on protocol
level.
### What is the impact of not doing this?
Digital assets, NFTs and utility tokens wil not be supported, hence the protocol prevents possible future use cases.

# Unresolved questions

 - Color dictionary would require another RFC.
 - `SigLockedDustAllowance`  output shall be updated with Colored Balances.
