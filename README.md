# Really Nice Mixer

Really Nice Mixer is Ethereum mixing service without any trusted 3rd party.

The goal of the project is to implement a most simple and cheap way to introduce transaction anonymity for Ethereum.

While there's many ideas how to implement Zero Knowledge Proofs for Ether and ERC-20 transactions, all of them are 
expensive to operate and inpractical. Really Nice Mixer is following a more simple approach.

## Concept

In short:

1. User starts a mixing batch (Mixing Event)
- other people join the batch
- by sending exactly same amount (say 1 Ether)
- a user has a message box associated with the batch
2. Once the batch is filled
- user send a multihop encrypted message to another user
- it goes through other participants, only last one can decrypt
- message contains a target address
- target addresses are added as output to the mix
3. Once outputs are filled
- users can confirm or dismiss the mixing event
- if all participants confirm, it can be withdrawn to the outputs
- otherwise event is closed and all coins are retuned to the original addresses
4. Mixing event is closed

As a results coins are redistributed to new addresses without association to source addresses. Operation could 
be easily automated on client side and anonymize any amount of ether (or tokens) to new addresses.

## Motivation

All balances and transaction history is public on Ethereum blockchain, which is an issue when you send transaction to a
stranger. Recipient will see whole history of your transactions and balances, learn about associated services, exchanges,
other addresses, etc. People tend to use exchanges as an intermediary just to hide their original address, or use
darkweb mixers risking to lose all of their coins or receive coins associated with criminal activities.

By having a trustless onchain smart contract based mixer most of such issues and risk can be avoided and will help
people to keep their investments more private.

Please be advised that Really Nice Mixer is not supposed and not allowed to be used to launder money received from
any illicit activities.

## Details

### Mixing Event

A mixing event is an individual batch of users exchanging their coins. Each participant provides same initial amount, 
and receive same amount to a new address.

Anyone can start a new mixing event, and it will be either finished with accepted status and coins will be distributed to 
new addresses, or will be rejected and coins will
return back.

Mixing Event ogranizer defines:

- time limit for the event actions
- amount of required participants
- mixing amount per user

### Joining a mixing event

When a new participant joins the event she send matched amount of Ether, public key to receive coordination messages 
and communication channel details (see below)

### Commit to the new addresses

It's obvious that participants can't reveal their final address, so instead of revealing it as is, a participant send
it as encrypted message to another random participant. The message is encrypted for recipient public key, when decrypted
it either has a final address or a next participant with another encrypted message. Eventually a last
recipient can decrypt final message with withdrawal address. You need multiple intermediaries to relay message to hide tracks.

Basically a message is a multilayered encrypted address, when a recipient can decrypt only a next hop. For better security
users will send fake messages with zero destination, only to add additional noise which supposed to make it harder to
associate addresses.

> User Alice -> user X -> user Y - User Bob

User Bob receives message containing a withdrawal address, but he will be unable to associate it with Alice, unless he
is also cooperating with _both_ X and Y. Alice can choose more that two relay users between her and Bob.

After receiving the message Bob adds received address to the list of withdrawal addresses of the mixing event. There is
no verification at this point.

### Commitment to withdrawal set

Once set of destination addresses is filled participants should confirm or reject the withdrawal set of addresses. If
any of the participants rejected it (or didn't confirm it during specified timeframe) then the mixing event is cancelled
and all deposits are returned to original addresses. Only if every participant confirmed destination set it can
be distributed to target addresses.

User vote for whole list, not for a particular addresses, therefore it's impossible to guess who own which address.

If Bob cheated and have added own address instead of address proposed by Alice, then Alice will reject the list. At this case
Bob may guess that address he received belongs to Alice, but because Dapp will automatically generate new random address
per each new mix (i.e. never be reused), this information will be useless.

### Messaging

Big part of the process is transferring encrypted messages containing target address. If we want 2 intermediaries, which
seems to be a minimum, it will be a least 3*n messages (where n is the number of participants).

Messages could be sent through open channel, including an onchain implementation on smart contract. Onchain communication
could be costly, but should not be very expensive, as it's just a payment for bytes being transferred. For the first version
of the Really Nice Mixer a simple smart-contract based communication will be implemented. Another option would be
running a Tor endpoint for communication, that type of communication may be added in a future version.

### Withdrawal

Withdrawal is done from contract, anyone can call withdrawal function (depending on provided gas amount), so coins will be
sent to a predefined addresses. Because it sent from contract and anyone can initiate sending, it doesn't need any balance
on the receiving address. This is opposite to schemas where recipient must initiate withdrawal by providing a secret. Latter
requires an initial amount to pay for a gas, i.e. you need to transfer small amount to target address prior to the
mixing, such operation would actually help to deanonymize target address.

_Our schema gives ability to generate fresh address per mixing event, and doesn't require to transfer a small amount to 
target address prior mixing._

## Roadmap

The current work is focused on Proof-of-Concept smart contract and basic UI. It will be designed with onchain messaging and 
support only Ether itself.

Next goal would be adding support for any ERC-20 token, Tor messaging, desktop app, etc.

## Support development

The project requires a lot of work to make it safe and usable, it will require extensive development, not even
counting initial proof-of-concept. Security issues are especially critical for such kind of smart contracts, so it will
need a 3rd party audit.

You can help the project by joining as a contributing developer (JS/Solidity).

Or you can help the project by donating to `0x4E3B1752ee6397b242A5ef326030A99B9D484686`

Feel free to contact by trumpmacron@protonmail.com if you have any questions.

## License

WTFPL
