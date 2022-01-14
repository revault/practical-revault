# Revault: A Multi-Party Bitcoin Vault Architecture

The secure storage and transaction of funds is a big challenge to Bitcoin users and especially to companies managing a substantial amount of bitcoins. Traditional controls of expenses are hard to set-up with Bitcoin due to its features: censorship-resistance, eventual transaction finality, and pseudonymous addresses.

Revault is a self-managed custody solution for multi-party holders (such as a company or their third party custodian) with strong theft mitigation from external parties and internal fund managers. Revault participants will have one or both roles as a *stakeholder* or *manager*. Revault has several novel features;

- On-chain tiered access control, where *stakeholders* have primary control and an ability to safely delegate funds to *managers*.
- Ability to cancel managers' withdrawals from a vault (automatically based on pre-configured policy, or manually)
- Ability to restrict managers' spending behaviours based on arbitrary policies
- Physical coercion mitigation using a trapdoor mechanism to move all funds to a preset *emergency descriptor*

Revault can be tailor-fit to many organisational structures with custom multi-signature thresholds, time-lock periods, and feature sets. Users can set policies in-line with business logic. Users can start with a simple deployment and scale infrastructure as their security budget increases. 

Contents:

1. [Introduction to Revault](#introduction-to-revault)
2. [Process and Revault Transaction Types](#process-and-revault-transaction-types)
3. [Infrastructure and Feature Choice](#infrastructure-and-feature-choice)
4. [Threat Model](#threat-model)

## Introduction to Revault

Revault is a custody solution with tiered access control. The bitcoins are co-owned by N stakeholders and are accessible with an N-of-N multi-signature. Portions of these bitcoins are delegated to M fund managers as-needed and are accessible with a K-of-M multi-signature but are subject to a set of _restrictive policies_. In practice, this delegation only allows the managers to follow pre-approved unvaulting and spending policies, thus retaining control at the stakeholder level. 

The single-tiered multi-signature approach common today is subject to a trade-off between security (large number of signatures required) and usability (low number of parties required). By comparison, Revault achieves practical usability through safe delegation to managers who handle the spending process without sacrificing on the security of a large multi-signature.

In addition, Revault comes with an (optional) deterrent feature for emergency scenarios. Before deploying Revault, stakeholders set-up an _emergency descriptor_. At any time stakeholders can trigger their panic button and unilaterally transfer all bitcoin in custody to the emergency descriptor. For example if 2 of the 3 stakeholders are abducted or physically threatened, the remaining stakeholder can save funds from theft. This feature drastically reduces the likelihood that a coordinated attack on the stakeholders will be profitable and thus acts as a credible deterrent from physical coercion.

## Process and Revault Transaction Types

![Revault Transaction Graph. Note that only one of the Cancel, Unvault Emergency or Spend TXs can consume a specific UTXO, and that the Spend TX has lowest priority as it must wait for the unvault time-lock to expire.](tx_diagram.jpg)

Funds enter custody through Deposit transactions (TXs) as unspent-transaction-outputs (UTXOs) locked to the *[deposit output descriptor](transactions.md#out)*, an N-of-N multi-signature among stakeholders. After receiving a deposit, the stakeholders exchange signatures for a set of 4 transactions which are _not broadcast_. These "pre-signed" transactions are used by stakeholders to control the flow of funds and enable the delegation and emergency deterrent features of Revault. As seen in the diagram, they include; Emergency, Unvault, Cancel and Unvault Emergency TXs.

During the set-up, stakeholders prepare an *[emergency output descriptor](transactions.md#emergency_txs)* which is a Bitcoin script template that is harder to unlock than *deposit output descriptor*. Keeping the *emergency output descriptor* private among stakeholders strengthens the emergency deterrent feature as attackers don't know how the emergency wallet is protected before it is spent from.

The stakeholders exchange signatures for the Cancel, Emergency and Unvault Emergency TXs:

- The Cancel TX would spend the Unvault TX to pay back to the *deposit output descriptor*.
- The Emergency TX would spend a deposit and pay to the *emergency output descriptor*.
- The Unvault Emergency TX would spend the Unvault TX to pay to the *emergency output descriptor*.

This way, funds are protected _before_ delegation to fund managers. Eventually, the Unvault TX may be signed (to restrictively delegate the UTXO to managers). It spends from the *deposit output descriptor* and pays to the *[unvault output descriptor](transactions.md#out-1)*, and is accessible with either:

- the managers' K-of-M multi-signature + time-lock of X blocks + J cosigning servers (where J = 0 if deploying without co-signers or J = N if deploying with co-signers), or
- the stakeholders' N-of-N multi-signature (with no delay).

Managers broadcast the relevant Unvault TX and are forced to wait for the unvault time-lock to expire before they can broadcast a Spend TX which pays to external wallets. During that delay period, a Cancel or Unvault Emergency TX can be broadcasted which will revoke the spend attempt.

Please check the complete specification of [transactions](transactions.md) for more details.

## Infrastructure and Feature Choice

Users can choose features to suit their needs. Different features require different infrastructure to operate. 

First, there are base requirements common to all Revault deployments. Each stakeholder and manager controls a wallet and a signing device. An untrusted Coordinator server is used to route messages and thus enable wallets to update and synchronise their state.  

- **Revault Wallet**: an open-source software running with a bitcoind back-end used to keep track of the users' co-owned coins, transaction signatures, transaction history and to connect with a signing device and infrastructure servers.

- **Signing Device**: a hardware module used to store private keys and wallet descriptors and to handle signing operations. Given information of the UTXO locked to the *[deposit output descriptor](transactions.md#out)*, this device can construct the set of Revault TXs itself based on its configuration data. This alleviates the need for human-verification of the TXs when signing.

- **Coordinator server**: An always-online untrusted server used for efficient message routing among participants and other servers. Its primary use is for exchanging signatures of pre-signed transactions and Spend TXs.

With these in place, stakeholders are able to delegate bitcoin to fund managers and manually broadcast Cancel, Emergency and Unvault-Emergency TXs. Fund managers will also be able to manually broadcast Cancel TXs to recover from mistakes. 

Next, users who want additional features will require additional infrastructure components. Each stakeholder may control one or more watchtowers and/ or a co-signing server.

- **Watchtower**: An always-online server that monitors the Bitcoin network and chain, and depending on its responsibilities (which could include: automated unvault policy enforcement, automated spend policy enforcement, emergency deterrent) it will respond to relevant events.

- **Co-signing Server**: An always-online server that acts as a 'stupidly simple' anti-replay signing oracle: it will accept to sign any Spend TX (consuming an Unvault UTXO), *but only once*. This is necessary to stop the managers from changing the Spend TX after the Unvault TX is broadcast.

Automatic unvault policy enforcement; requires a Watchtower. By default the Watchtower will broadcast a Cancel TX any time it detects an Unvault TX unless it adheres to a specific pre-configured unvault policy.

Spend policy enforcement; requires a Watchtower and Co-signing Server. By default the Watchtower will broadcast a Cancel TX any time it detects the Spend TX's parent Unvault TX unless it is made aware of the Spend TX in advance and that Spend TX adheres to a pre-configured spend policy. The co-signing server is a means to stop the managers from spending from the Unvault TX with a malicious TX which replaces the Spend TX after it was announced to the watchtowers (which must therefore be done *before* broadcasting the Unvault TX). 

Automatic Emergency Deterrent: requires a *Watchtower* which monitors the Bitcoin network and maintains multiple direct communication channels with the stakeholder. The trigger can be a panic button or dead-man-switch (or otherwise). When triggered the Watchtower which will broadcast all of its Emergency TXs and/or Unvault Emergency TXs. To further reduce the incentive of a theft attempt the broadcast of any Emergency TX associated with any vault triggers the broadcast of all Emergency TXs associated with all the remaining vaults.

Please see [here](https://github.com/revault/practical-revault/blob/master/messages.md#processes) for a more detailed look at the processes in Revault. 

The flexibility brought by the active defense mechanism allows any unvault or spending policy to be set and enforced by watchtowers. This therefore largely extends the scope of the possibilities compared to on-chain enforcement and is more in-phase with typical business policies (enforcement of a 2FA, of business hours, etc..).


## Risk

While Revault has been designed to fix (some of) the threats on existing custody protocols, it introduces new challenges and different assumptions. Revault aims to prevent theft with up to N-1 stakeholders' signing devices being compromised. Here we highlight different attack scenarios, trying to identify weaknesses in the design and finish with a set of security assumptions introduced when using Revault. For more detailed research, check the [Operational Risk Framework](https://github.com/revault/research/blob/master/risk_analysis/paper.pdf).

### Emergency Deterrent Risk

While the Emergency TX is the strongest mitigation in the Revault architecture, it introduces a potential risk of blackmail or losses derived from lack of business continuity (e.g. a block-and-short attack). An attacker (internal or external) getting hold of a single Emergency TX can threaten to push it to the Bitcoin network, triggering a push of all Emergency TXs by the watchtowers. The funds are not at risk in this scenario but getting the funds back online would be long and difficult. On the other hand, an important part of the security model (the deterrent) is enforced by the pre-signed Emergency TXs being accessible.

### Delegation Risk - Unvault Policies

An "active defence" mechanism is used to restrict the behaviour of fund managers. Contrary to commonly deployed multi-signature wallets, it takes managers two transactions to spend funds. The first step encodes an on-chain time-lock. During this time, only a single honest watchtower is required to cancel given a breach of the unvault policy. It is expected and encouraged to deploy many watchtowers (at least one per stakeholder). 

### Delegation Risk - Spend Policies

To by-pass the spend policy enforcement, an attacker would need to compromise all co-signing servers *or* to compromise all watchtowers. Note that due to the former, there are more attack vectors here compared with unvault policies and so spend policy enforcement is relatively weaker. In the former case, an attacker could alter the Spend TX after the Unvault time-lock expires. In the latter case, a malicious Spend TX would not trigger the broadcast of a Cancel TX.


### Denial-of-service Risk

As a consequence of the N-1 resilience model, there are multiple attacks against the practical usage of the funds:

- signing refusal by a stakeholder
- unjustified Cancel TX push
- unjustified Emergency TX push
- taking down a co-signing server
- taking down (all) the coordinator(s)
- pinning Unvault TXs

In addition, if the network becomes highly congested, high feerates would trigger most Bitcoin network nodes' to increase their mempool's minimum accepted feerate (those who didn't manually increase the mempool limits). Unvault TXs that are prepared with an initial 24 sat/vb feerate could fail to be relayed if the minimum accepted feerate increases sufficiently. Managers would not be able to use funds delegated to them until the backlog of transactions is processed by the Bitcoin network (mined).


### Key-tree Risk

The use of BIP32 unhardened derivation introduces even more reliance on the signing devices' security. Since the chaincodes are exchanged and stored on the wallets (running on untrusted online devices) of each participant, leaking a single child private key would be fatal to a participant's entire key-tree. Therefore the leak of a single child private key of all N stakeholders would be equivalent to the loss of all the funds. Using unhardened derivation implies that there is no partial loss of funds in case of a key leak by every stakeholder.


### Transaction Fee Risk

A fundamental issue arises with pre-signed transactions commiting to the feerate well in advance of the time of broadcast. The required feerate is unpredictable and a transaction with a low feerate may not be included in a block in time.

[Pinning attacks](https://bitcoinops.org/en/topics/transaction-pinning/) are possible when using either RBF or CPFP to dynamically allocate fees to transactions. These attacks exploit vulnerabilities that can cause transactions to become "stuck" in the mempool for long times. For protocols like Revault which require timely transaction confirmation, these attacks can be used to defeat theft-mitigation features (e.g. pinning a Cancel TX for longer than the Unvault TX's time-lock, and stealing funds).

The Unvault TXs have an additional output that allows dynamically allocating fees through child-pays-for-parent (CPFP). This feature helps managers optimise withdrawal times. The possibility of pinning Unvault TXs doesn't impact the theft-mitigation features of Revault but can be used as a denial-of-service attack by comrpomised managers. This is acceptable since managers already have multiple ways to launch denial-of-service attacks. For the Cancel, Emergency and Unvault Emergency TXs it is critical to avoid transaction pinning attacks. Unfortunately, we are not aware of a method to dynamically allocate fees to these transactions without the risk of pinning attacks (at least, without any soft-forks to Bitcoin). A more detailed discussion can be found [here](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-November/019614.html).

Emergency and Unvault Emergency TXs are simply pre-signed with exorbitant fees as they are intended as a deterrent, and not a regular expense.
