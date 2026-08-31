---
tip: XXXX
title: OTAK-NES: A Blockchain Security Architecture for Non-Exposure of Long-Term Authorization Keys
author: Ali (@alichatme)
status: Draft
type: Standards Track
category: Core
created: 2026-06-07
discussions-to: https://github.com/tronprotocol/tips/issues/890
---
# Summary

### OTAK-NES: One-Time Access Keys – Non-Exposure Security

### Long-Term Exposure of Published Signatures to Future Threats

Every account in a digital asset network, when transferring assets—even in very small amounts—signs a transaction using its private key and submits the generated signature to the network for validation.

This signature is exposed to various network components during transaction propagation and validation, including nodes. After the transaction is confirmed, the signature, along with transaction-related data, is permanently recorded in the blockchain history.

Consequently, each time a private key is used to sign a transaction, a new sample of cryptographic data associated with that key is made available to future adversaries. This data remains accessible for a very long period of time.

If an attacker were able to recover the private key corresponding to an account from a published signature and other related public data—using future quantum computers, Shor's algorithm, future advances in mathematics or computational power, cryptanalytic progress, implementation errors, or software bugs—the threat would not be limited to the past transaction or the transferred asset. Access to the private key would mean access to the account's signing authority and, therefore, to all assets held by that account.

In other words, the core problem is not merely the security of a transaction that has already occurred; rather, the exposure of a long-term private key can compromise the security of all assets that remain under the control of that same key.

This creates a critical characteristic in architectures based on long-term signing keys: a single key may be used over many years to sign numerous transactions, leaving the entire account's authorization dependent on one persistent key.

### OTAK-NES Architecture: Shifting the Threat Model from the Entire Account to a Single Transaction

OTAK-NES is designed to eliminate this dependency. In this architecture, keys that have access to account assets (Access Keys) are not used to sign operational transactions. Each transaction is signed by an independent one-time key (Child Key).

All Child Keys have private keys distinct from the Access Keys and from other Child Keys of the account, and each Child Key is permanently deactivated after a single use.

Therefore, even in a scenario where an operational Child Key is compromised in the future, the scope of that event is limited to that specific Child Key and its operational authority, and it does not extend to the Access Key or other Child Keys of the account.

### A Paradigm Shift in the Threat Model

This proposal introduces one-time access keys (or "Child Keys" for short) to the blockchain network.

A Child Key introduces a fundamental paradigm shift in blockchain security: it changes the threat model from the entire account and all assets under its control to a single transaction and the specific asset being transferred.

In conventional blockchain architectures, the exposure of a single operational signature from an Access Key can put all account assets at risk.

Under the OTAK-NES security architecture, long-term authorization keys of an account (Access Keys) do not directly produce operational transaction signatures on-chain.

Instead of allowing Access Keys to sign transactions directly, OTAK-NES introduces one-time authorization keys ("Child Keys") derived from an Access Key according to the TRC-102 hierarchical derivation model. Each Child Key is authorized through a Merkle Proof against a Merkle Root committed to during account creation and the Bootstrap process.

Each Child Key is valid for exactly one successful transaction. After that transaction is accepted by the network, the Child Key is permanently marked as consumed and cannot authorize another transaction.

This architecture achieves a critical security property: if a Child Key or its signature is compromised in the future, the potential impact at the network and account level is limited to that operational key, that transaction, and the specific asset being transferred—not to all account assets.

Consequently, the Access Keys of the account and the assets under their control are not exposed when a Child Key is compromised.

If, in the future, an attacker could recover the private key of a Child Key using advanced cryptographic methods or new computing technologies, because the Child Key's private key is distinct from all Access Keys of the account, the incident would only affect that specific consumed Child Key. Since that key has already been deactivated:

- It cannot be reused.
- It cannot sign a new transaction.
- No active operational permission is attached to it.
- The security of other Child Keys is not affected.
- It does not alone provide enough information for the attacker to recover the account's Access Keys.
- After the transaction is confirmed and the asset is transferred to the destination account, the used Child Key no longer controls the transferred asset and is immediately removed from the operational cycle and deactivated.

Consequently, the OTAK-NES architecture limits the impact of any possible security incident to a single Child Key, the specific asset being transferred (not all account assets), and prevents its spread to other components of the account.

This architecture is independent of the underlying signature algorithm. It can operate with existing signature algorithms as well as future post-quantum signature schemes. Its objective is not to replace signature algorithms, but to eliminate continuous exposure of authorization-key signatures and fundamentally change how the scope of a potential threat is contained on the blockchain.

### Non-Exposure of Access Key Signatures

The principle of non-exposure of signatures from keys with access to all assets of an account (Access Keys), although currently considered an optional aspect in some blockchain architectures, may become an important security requirement for blockchain architectures over time, as computational sciences advance and highly powerful computers, including quantum computers, emerge.

Beyond the above, there is another fundamental question:

Why would one refrain from limiting the potential scope of a compromise during transaction submission and signature publication—from the entire account and all assets under its control, to that specific transaction and the specific asset being transferred?

Even if we assume that a transaction signature alone provides complete cryptographic security and that publishing that signature creates no direct threat to the account's private key—although no signature algorithm, including post-quantum schemes, has ever made such a claim—the transaction may still be observed or processed by various network components before finalization and block inclusion, including:

- Mempool
- Network nodes
- Block producers or proposers
- Validators and other consensus participants
- Transaction propagation and relay infrastructure
- RPC and API providers
- and Intermediate services and infrastructure that receive, forward, or submit transactions for validation.

Therefore, even with a very strong signature algorithm, the exposure of a transaction and its signature in the public network environment represents an independent layer of exposure that cannot be eliminated solely by increasing the strength of the signature algorithm.

This raises another question:

Why should blockchain architectures not reduce the threat model as much as possible—from the entire account and all assets under its control—to the smallest operational unit, namely a specific operational key and a specific transaction?

The OTAK-NES security architecture is introduced precisely on this principle: reducing the potential scope of a compromise at the account level. It is designed for implementation on existing blockchain networks.

This approach can, in the long term, be regarded not only as a security feature, but as an architectural principle and a security standard for blockchains.

Blockchains that have adopted user security and asset protection as one of their fundamental principles can move toward evaluating and, subject to technical validation, implementing the OTAK-NES architecture.

### One-Time Access Key (Child Key)

A Child Key is a one-time authorization key derived from an account Access Key using the TRC-102 hierarchical derivation model, or similar models based on the same principles in other networks.

A Child Key is considered valid only if:

- It is derived from the account's registered authorization key
- Its membership in the committed Merkle Tree is proven through a valid Merkle Proof
- It has not previously been consumed.

Any transaction that does not satisfy these conditions MUST be rejected during transaction validation.

After a transaction is successfully accepted, the corresponding Child Key is permanently marked as consumed in the network state and can never be reused.

Only Child Key signatures appear in the blockchain transaction history. Long-term authorization keys never participate in normal operational transaction signing, thereby significantly reducing exposure to attacks that rely on analysis of long-lived signatures.

Consequently, even if the signature of a previously used Child Key is compromised in the future:

- Its impact at the architectural level is limited to that specific operational key and transaction
- If the asset transfer was successful, the asset associated with that transaction is no longer under the control of the source account and has been transferred to the destination account
- And since the Child Key has a private key independent of the account's Access Key, compromising one Child Key does not, by itself, compromise the Access Key of the account or other Child Keys.
___
___
# How OTAK-NES Architecture Defeats Signature Analysis Threats — Even When They Become Practical at the Network and Account Level, Including but Not Limited to Quantum Threats — at 4 Levels

First, the importance of the OTAK-NES security architecture and the non-exposure of the account's public key.

In all digital signature systems, whether classical (such as ECDSA) or post-quantum (such as ML-DSA, Falcon), transaction verification depends on network components having access to the signer's public key. In other words, for a node or validator to verify that a transaction was genuinely signed by the private key holder, it must have the corresponding public key. Therefore, the public key is either included directly in the transaction or made available to nodes in some form to complete the verification process. This principle is identical for both classical and post-quantum signature algorithms, and the network's signature algorithm does not change the fundamental requirement of "public key exposure."

### The Role of the Public Key in Signature Analysis Threats, Including Quantum Threats

Based on the latest published research on signature analysis threats — including but not limited to quantum threats — there is a critical difference in how an attacker can compromise a signature to analyze and recover the private key when both the public key and the signature are available:

- Digital Signature: In many scenarios, directly analyzing a signature without the corresponding public key makes the private key recovery problem significantly more difficult (because limited data is available for analysis).
- Public Key: When a digital signature is available along with its corresponding public key, the time required for a quantum computer to recover the account's private key is significantly reduced. For example, Shor's algorithm uses both the public key and the signature to derive the private key.

To better understand the importance of analyzing the threat to both the public key and the signature simultaneously, it is useful to review the account creation process in a blockchain:

1. Private Key: The wallet first generates the account's private key by producing a secure random value appropriate for the network's signature algorithm.

2. Public Key: The public key is then derived from the private key using a one-way function.

3. Digital Signature: The signature is generated using the signature algorithm and the private key, applied to the message or its hash.

As described, both the signature and the public key share a common characteristic: they are both derived from the private key.

Having both the signature and its corresponding public key provides an attacker with a significant advantage in signature analysis threats, including quantum threats. This gives the attacker two paths toward the same goal.

The public key is exposed along with the signature when a transaction is sent to the network. An attacker can use it immediately or store it for later analysis.

Access to signatures is always possible, as signatures are typically recorded permanently in the blockchain history.

### Analysis of the Critical Window

From the moment a transaction is sent to the network, its public key becomes accessible to various network components — such as the mempool, nodes, block producers, validators, and other network participants.

Any of these components, or any attacker with access to this data, can collect the public keys associated with Access Keys of accounts across the network and store them for future analysis and threats. This data can be used immediately or at any later time for targeted attacks.

Preventing the exposure of the Access Key's public key can create an independent defensive layer, because the public key of the Access Key — which controls account assets — is not published on the network under the OTAK-NES security architecture and therefore does not fall into the attacker's hands.

### The OTAK-NES Solution for Non-Exposure of Public Keys on the Network

OTAK-NES pursues as one of its main goals not only preventing the exposure of private keys, but also preventing the exposure of the public keys of keys that have access to account assets.

This architecture intelligently neutralizes this threat at the network level:

### 1. Key Separation: Access Keys and Child Keys Are Distinct

In OTAK-NES, two types of keys with completely different roles are defined:

- Access Key: The main, long-term key of the account that controls all assets. This key is never used for signing operational transactions, so its public key is never published on the network.
- Child Key: A one-time, temporary key derived from the Access Key according to TRC-102 under the OTAK-NES architecture. It has a public and private key pair that is completely different and independent from the account's Access Keys.

### 2. The Process of Non-Exposure of Access Key Public and Private Keys in the OTAK-NES Architecture

Under the OTAK-NES security architecture, the wallet creates two keys when the account is created:

- Owner Key
- Active Key

With the requirement that both keys have independent and distinct public/private key pairs.

After account creation, under the OTAK-NES architecture, the first transaction is limited to the Active Key transaction for registering the Owner Key's Merkle Root in Bootstrap. After the Owner Key's Merkle Root is successfully registered, the registering Active Key is deactivated by the network according to the OTAK-NES protocol. From that point on, all operational transactions — such as creating a new Active Key for the account or sending a transaction — are accepted and validated by the network **only and exclusively** through Child Keys. Any transaction sent with an Access Key is rejected with a `CHILD_KEY_REQUIRED` error.

Each time a new transaction is sent, a new Child Key is generated by the wallet from the Access Keys according to TRC-102, with public and private keys distinct from the main key. It signs the transaction and sends it to the network. After the transaction is confirmed, the Child Key is deactivated and then recorded in the network history.

Consequently, under the OTAK-NES architecture, when a transaction is sent, the Child Key's public key and signature are exposed to network components or potential attackers — **not** the Access Key's public key and signature.

### How a Quantum Attack on an Account Is Thwarted at Four Levels Under OTAK-NES, Even If a Child Key Is Compromised

Assume an attacker, having obtained a Child Key's transaction signature and public key, succeeds — in a very short time, such as a fraction of a second — in recovering the corresponding Child Key's private key.

(This is a security assumption for analysis and does not imply that quantum computers are capable of such speed.)

**Key Question:** With this private key, can the attacker send a new transaction to the network?

### Level 1: The Recovered Key Is Not the Access Key

Assuming the signature analysis succeeds, the quantum computer in this scenario calculates and obtains the Child Key's public and private key.

However, this public/private key pair **does not belong to the account's Access Key**. Therefore, recovering a Child Key's private key does not mean recovering the Access Key or gaining direct access to all account assets.

### Level 2: Creating a New Child Key

Even with a Child Key's private and public key, the attacker cannot use that same Child Key to create a new valid operational transaction.

Each Child Key in OTAK-NES is used for a single, specific transaction and is one-time-use only. After successful transaction confirmation, the Child Key is marked as consumed and can no longer be used to send a new or repeated transaction.

The private key of Child Key i does not, by itself, generate or reveal the private key of Child Key i+1.

### Level 3: The Asset Being Transferred Remains Safe Even If Signature Analysis Threats — Including but Not Limited to Quantum Threats — Become Practical

During the Child Key transaction confirmation process, two scenarios are possible if a signature analysis threat becomes practical:

**1. Transaction Confirmed:** The transferred asset is sent to the destination account before the attacker can complete the signature analysis and discover the private key.

- In this scenario, the asset has been transferred to the destination account, which is controlled by a different set of public/private keys.
- The Child Key has been marked as consumed and cannot be reused.

Therefore, recovering the Child Key's private key after transaction confirmation does not give the attacker new authority to access the sent asset or the assets controlled by the Access Keys.

**2. Transaction Not Confirmed:** The asset has not yet been transferred, and the transaction is still in the validation process.

If we assume the attacker could reconstruct the Child Key transaction using the same public and private key and resend it to the network in a fraction of a second, even in this hypothetical scenario, the key recovery time is never truly zero. Even under the most conservative assumption — treating this time as effectively zero — the reconstructed transaction would still enter the network's validation process after the original Child Key transaction.

Under the current design of the TRON network, it is not possible for an attacker to send a reconstructed transaction ahead of the original transaction.

As a result, the reconstructed transaction is placed in the transaction queue after the original transaction. Once the original transaction is confirmed, the reconstructed transaction, under the OTAK-NES architecture, encounters a `CHILD_KEY_CONSUMED` error.

Other blockchain networks can also add this independent defense layer by supporting OTAK-NES and, provided they enforce sequential transaction ordering.

### Level 4: The Account Continues Operation with the Same Level of Security

Assuming the Child Key has been recovered and its public and private keys are in the attacker's possession, the main Access Key remains **uncompromised**.

Child Keys are one-time-use, meaning they are deactivated after their first use. Additionally, Child Keys have public and private keys that are distinct from the Access Keys. Therefore, even if a Child Key's public and private keys are discovered, the account can continue normal operation on the network with the same level of security as before.

This security property under the OTAK-NES architecture is the strongest reason for its adoption.

### The Best Approach to Protecting a Network from Future Signature Analysis Threats, Including but Not Limited to Quantum Threats

The best approach for a network to remain protected from future signature analysis threats — including but not limited to quantum threats — is to use multiple independent defensive layers, rather than relying on a single cryptographic layer alone.

Using a post-quantum signature algorithm provides one strong defensive layer against such threats.

However, if one day this cryptographic layer is broken by an attacker using an analytical threat, the important question becomes:

**How will the network protect assets and transactions after the cryptographic layer is broken?**

The answer is clear:

- The network supports the OTAK-NES security architecture (one-time Child Keys).
- The network supports sequential transaction ordering.

Consequently, even if an attacker can recover a Child Key's private key, breaking the cryptographic layer alone is not sufficient to mount a successful attack.

### OTAK-NES as a Complementary Security Layer

OTAK-NES cannot and should not replace post-quantum signature algorithms on its own.

However, in combination with them, in addition to reducing the information available for attacking keys with direct access to account assets, and by prohibiting direct Access Key transaction submission and using one-time Child Keys, OTAK-NES creates a new architectural security layer.

More importantly, the OTAK-NES architecture attempts to answer the following question:

**If quantum threats one day become practical — even with a powerful post-quantum signature algorithm — what mechanism will the network have to maintain secure operation and defeat the attack?**

OTAK-NES answers that breaking one operational key should not mean breaking the account.

Even if an attacker can recover a Child Key's public and private key, the consumed Child Key should not be reusable for a new transaction, and the main Access Key should not be exposed to attack.

Therefore, if a network supports:

- The OTAK-NES security architecture
- Sequential transaction ordering

Breaking the primary cryptographic layer will not lead to account compromise or asset theft.
___
___
# Minimum Required Defense Layers for a Secure Network

### The Importance of Layered Security

Recent blockchain research and market trends indicate that users are increasingly choosing networks that meet minimum security standards and have a clear roadmap for future security enhancements.

Conversely, networks that lack these fundamental requirements or a transparent strategy for addressing future security threats may experience user migration toward more secure alternatives. This trend has intensified in response to security incidents—including various signature analysis threats (including but not limited to quantum attacks), digital asset thefts due to network security flaws, software bugs, and other vulnerabilities—demonstrating that "security" has become a key selection criterion for blockchain users.

### The Most Prominent Threat with High Publicity: Quantum Threats

Quantum security for a blockchain cannot be defined merely by selecting a post-quantum signature algorithm. A secure network must be able to protect users, keys, transactions, accounts, and the network itself across multiple independent layers. A weakness in one layer should not necessarily lead to the complete failure of other layers.

Under modern approaches, blockchain security can be viewed as a multi-layered defensive architecture. Each layer addresses a specific level of threats, and if one layer is weakened, other layers—depending on the threat model and architectural dependencies—can still protect assets and network functionality.

Even the best wallet and the best signature algorithm are insufficient for ensuring security in today's world. A network must have appropriate multi-layered defense mechanisms at the protocol level.

A secure network must be able to:

- Correctly validate signatures
- Control replay attacks
- Manage transaction lifetime and status
- Reject consumed keys
- Accurately enforce account permissions
- Establish and maintain logical transaction ordering during threats, preventing attackers from using mechanisms such as fee manipulation to push reconstructed transactions ahead of the original transaction
- During a security incident, reduce the impact scope from the entire account and all assets to smaller, more contained units—or neutralize the threat entirely even if it materializes
- Protect network operations and account assets even if a quantum threat becomes practical

### 1. First Defense Layer: Network Signature Algorithm

### Why Should Networks Migrate from ECDSA to Post-Quantum Signatures?

The security of digital signatures in blockchain networks depends on whether an attacker can derive the private key from public keys and published signatures, or forge a valid signature.

In ECDSA, security is based on the difficulty of the Elliptic Curve Discrete Logarithm Problem (ECDLP). This problem is sufficiently hard for current classical computers, but a powerful quantum computer running Shor's algorithm could solve it efficiently.

Therefore, if an attacker gains access to a sufficiently capable quantum computer, simply increasing ECDSA key sizes cannot address the fundamental problem. The real solution is migrating from ECDSA to a Post-Quantum Cryptography (PQC) signature algorithm—one whose security is based on a different mathematical problem that is resistant to known quantum attacks.

### Why Increasing ECDSA Key Size or Hardness Doesn't Provide Quantum Immunity

The difference lies in the type and structure of signature algorithms. Post-quantum signature algorithms are developed with fundamentally different designs compared to ECDSA, specifically to resist signature analysis threats including but not limited to quantum threats.

Increasing ECDSA key size—for example, from P-256 to P-384 or P-521—can only increase classical security. However, due to the inherent vulnerability of ECDSA to quantum attacks (especially Shor's algorithm), even making computations harder or keys larger cannot make it resistant to quantum threats.

### The Role of OTAK-NES on the ECDSA Signature Algorithm

Shor's algorithm and similar algorithms for signature recovery require both the signature and the account's public key. OTAK-NES not only prevents the exposure of private keys but also prevents the exposure of Access Key public keys, shrinking the threat scope from the entire account to the specific asset being transferred. If the network supports sequential transaction validation, OTAK-NES can defeat a quantum attack at four levels even if the attack succeeds.

### The Real Solution: Post-Quantum Signature Algorithms (PQC)

To counter signature analysis threats—including but not limited to quantum threats—post-quantum signature algorithms can be used, which are built on different mathematical foundations, including:

- ML-DSA (Dilithium): Based on lattice-based cryptography
- FN-DSA (Falcon): Based on lattice-based cryptography
- SLH-DSA (SPHINCS+): Based on hash-based cryptography

These algorithms, despite their post-quantum resistance, come with costs such as increased key and signature sizes. For example, the ML-DSA-44 signature size is approximately 2,420 bytes, while ECDSA signatures are typically much smaller.

Therefore, migrating from ECDSA to PQC is not merely about using "larger keys"; it means changing the security foundation of signatures from a problem that Shor's algorithm can solve to a structure designed to resist quantum threats.

The signature algorithm is one of the most important defense layers for a network. However, it should not be assumed that the signature algorithm is the first and last defense layer, and that migrating to post-quantum signatures alone provides all necessary protection.

A network should use an algorithm whose security level can be evaluated based on public analysis, established standards, and specific parameters. In the post-quantum domain, simply seeing the label "Post-Quantum" on an algorithm does not mean that all schemes are equal in terms of security, maturity, parameters, implementation complexity, or confidence levels.

NIST has finalized FIPS 204 for ML-DSA and FIPS 205 for SLH-DSA as post-quantum signature standards. FALCON is also being standardized as FN-DSA, and the PQC ecosystem continues to evolve.

Therefore, "post-quantum readiness" should be accompanied by several questions:

- What is the mathematical foundation of security?
- What is the threat model?
- What classical and quantum attacks have been analyzed?
- What is the security level of the selected signature type?
- What is the standardization status?
- What is the implementation complexity?
- What are the public key and signature sizes?
- What are the signature generation and verification costs?
- Is the implementation resistant to side-channel and fault attacks?

### Major Families of Post-Quantum Signature Schemes

At the architectural level, post-quantum signature schemes can be categorized based on their underlying mathematical assumptions:

**a) Lattice-based Schemes**

- Key example: ML-DSA
- This family is based on hard problems in lattice-based cryptography and is standardized in FIPS 204.
- The main advantage is a good balance between security, speed, and relatively manageable key and signature sizes.

**b) Hash-based Schemes**

- Key example: SLH-DSA
- SLH-DSA is based on hash function cryptography and is standardized in FIPS 205.
- Its security foundation differs from ML-DSA, and this diversity in security foundations can provide significant architectural defense value.
- However, signature sizes are typically larger, which can create overhead for high-volume blockchains.

**c) FALCON / FN-DSA**

- FALCON is also a lattice-based scheme selected for standardization under the name FN-DSA.
- It may offer advantages in signature size and performance, but implementation complexity and precision requirements must be equally considered.

### NIST Security Categories for Post-Quantum Signature Algorithms

NIST does not rank post-quantum algorithms from first to fifth; instead, it defines a Security Category for each parameter set.

These categories indicate the computational resources required to break a specific parameter set. In summary:

- Category 1: Approximately 128-bit security as a benchmark
- Category 2: Approximately 128-bit security
- Category 3: Approximately 192-bit security
- Category 4: Approximately 192-bit security
- Category 5: Approximately 256-bit security

However, an important note: these numbers do not mean that a Category 5 algorithm provides exactly "256 bits of quantum security." This categorization is a comparative metric for attack difficulty that NIST uses to evaluate different parameters.

A higher Category does not automatically mean an algorithm is "absolutely better" for all use cases. Increasing security levels typically comes with increased key size, signature size, processing time, and storage and transmission costs.

Therefore, in a blockchain network, algorithm selection is not based solely on the highest Security Category; the required security level, key and signature sizes, generation and verification speed, bandwidth, transaction capacity, and storage costs must also be considered simultaneously.

### Why Does NIST Assign Security Levels to Post-Quantum Signatures?

When we say an algorithm is post-quantum (PQC), it does not mean the algorithm has "absolute immunity" or a fixed number like 100. The concept of PQC is that the algorithm is designed to resist known quantum attacks; however, no cryptographic algorithm can be declared immune forever and against all types of attacks with a fixed number.

For this reason, NIST uses Security Categories instead of saying an algorithm has "90% or 100% immunity."

### What Is the Floor and Ceiling of This Categorization?

In NIST's five-tier framework:

- Category 1 is the lowest level defined in this framework.
- Category 5 is the highest level defined in this framework.

However, this by no means implies that Category 5 represents the "highest possible security in the world" or a mathematical ceiling for cryptographic security. NIST defined these five levels solely for standardization and comparison.

Therefore, we should not say:

Category 5 = 100% quantum immunity

But rather:

Category 5 = A very high level of attack difficulty that, within NIST's framework, is comparable to the difficulty of a full key search against AES-256.

### Why Isn't a "100% Immunity" Signature Produced?

Because cryptographic security is not an absolute and fixed property. The security of an algorithm depends on factors such as:

- The best known attack against the algorithm
- The algorithm's parameter sizes
- The attacker's computational power
- Future advances in cryptography and cryptanalysis
- Advances in classical and quantum hardware
- How the algorithm is implemented and used

An algorithm that is highly secure against the best known attacks today could be broken by a new attack method discovered in the future. For this reason, cryptographic standards use security levels and specific security assumptions rather than claiming "100% immunity."

### If a Blockchain Wants Maximum Practical Security, What Should It Do?

The solution is not simply to push an algorithm to Category 5 and then claim the network is "100% quantum-safe." A blockchain must create a multi-layered defense architecture:

1. Use a standard PQC signature with an appropriate security level.
2. Use larger parameters for greater security margin when needed, considering the trade-offs in key size, signature size, processing, and transmission costs.
3. Support gradual migration and algorithm agility, so that if a serious weakness is discovered in one algorithm, the network can migrate to a new PQC algorithm.
4. Use hybrid signatures during transition periods, ensuring that an attacker must breach multiple layers to forge a transaction.
5. Protect the entire key lifecycle, including generation, storage, usage, rotation, and revocation.

### Conclusion

The real goal of a blockchain should not be to reach a hypothetical number like "100% immunity." The goal should be to create the greatest possible practical and provable security margin against the relevant threat model.

In simple terms:

PQC does not mean "absolute security"; PQC means shifting the security foundation to a structure designed to resist known quantum attacks.

And Category 5 is not the end of security; it is the highest level NIST has defined within this five-tier framework for comparison.

### Why Should a Network Still Adopt OTAK-NES Even If It Supports Post-Quantum Signatures?

- The OTAK-NES security architecture does not depend on any specific signature algorithm and can be deployed as an architectural layer regardless of the underlying signature scheme.
- A post-quantum signature algorithm alone cannot limit the threat scope from the entire account to a smaller component; OTAK-NES achieves this by limiting the lifetime and scope of operational key authority.
- In the defined OTAK-NES architecture scenario, the network supports Child Key mechanisms, so a successful attack on an operational Child Key signature does not necessarily compromise the entire account.
- A post-quantum signature algorithm alone does not prevent the repeated publication and use of long-term access keys with control over account assets.

### 2. Second Defense Layer: Sequential Transaction Validation

Sequential transaction validation can be an important security layer in network architecture.

In a model with ordered confirmation, the network can establish a specific sequential relationship between related transactions, ensuring that a later transaction cannot invalidate the result of a previous one simply by arriving at a node earlier or using mechanisms like fee manipulation to push a reconstructed transaction ahead of the original.

However, transaction ordering alone does not prevent all types of MEV or front-running attacks, as these depend on the mempool model, ordering policy, consensus, and transaction execution.

### Ordering Does Not Prevent Batch Processing

An important point: ordered confirmation does not necessarily mean "only one transaction at a time." A large exchange or service can prepare and send many transactions as a batch.

For example:

- Batch A consists of n transactions from one account sent to the network for validation.
- Then, Batch B consists of n transactions from the same account, and validation of Batch B does not begin until all transactions in Batch A are resolved.

In this model, the network treats that transaction batch as a single transaction for ordering purposes, maintaining transaction order regardless of whether transactions are submitted individually or in batches. What matters is maintaining the sequential validation relationship based on transaction arrival, regardless of whether they are submitted singly or in groups.

Sequential validation can be effective against a range of attacks, including:

- Replay attacks
- Double-spend attempts
- Race conditions
- Front-running attacks
- Transaction reordering
- Sandwich attacks
- Race attack prevention (eliminating concurrent submission, requiring execution permits)
- Transaction replacement prevention (eliminating mempool)
- Amount modification and replay prevention (parameter verification with canonical copy)
- Replay attacks with Permit ID + Validity Period stored in ledger
- Transaction-ordering dependence (deterministic order with a defined execution path)
- Transaction pinning (eliminating public mempool space, controlled admission)
- Censorship prevention (dynamic node selection, continuous monitoring)
- And other attacks based on rapid transaction reconstruction

Additionally, sequential validation in a network supporting the OTAK-NES architecture ensures that even if a quantum attack succeeds and an attacker reconstructs a transaction in a fraction of a second, they cannot insert the reconstructed transaction ahead of the original.

Furthermore, since each Child Key is only authorized for one transaction under OTAK-NES, after successful network validation, the Child Key is deactivated and then recorded in the network history. Therefore, a reconstructed transaction entering validation after the original transaction will be rejected with a `CHILD_KEY_CONSUMED` error.

### 3. Third Defense Layer: Wallet Security

A wallet is not just a user interface; it is responsible for key lifecycle management, signature generation, key selection, and transaction information display.

A secure wallet should, at account creation, not only create two keys with distinct public/private key pairs but also distribute operational permissions between them.

For example:

- Owner Key: The key with the highest level of authority, which should ideally not be used for routine transaction signing.
- Active Key: The key used for operational transaction signing. If the network does not yet support OTAK-NES, the Active Key should be periodically replaced.

The core principle is:

"The key used for long-term account recovery and control should not be the same key that signs everyday transactions."

In TRON, the Account Permission and Active Permission concepts allow for the separation of certain account authorities, making the signature and permission system part of the security model.

### Why Should the Wallet Periodically Change the Active Key If the Network Does Not Support OTAK-NES?

If an Active Key signs all transactions throughout an account's lifetime, a large number of signatures from that key will accumulate on-chain over time. Therefore, the wallet can implement an important defense policy:

"An account's operational key should not be repeatedly exposed beyond a specific time period or number of transactions."

The operational key of an account can be periodically replaced with a new key to prevent signature accumulation on the blockchain. In a proper design, the wallet should:

- Manage the operational key's lifetime
- Notify the user when it's time to rotate the key
- Keep the Owner Key as far as possible from routine operational transaction signing

### OTAK-NES Advantage in Preventing Access Key Signature Accumulation

Under the OTAK-NES security architecture, Access Keys with control over account assets are never permitted to send transactions directly to the network. All submitted transactions must use Child Keys derived from the Access Keys.

If a signature is sent directly using an Access Key, nodes will reject the transaction under the OTAK-NES architecture and return a `CHILD_KEY_REQUIRED` error to the wallet.

Therefore, with network support for OTAK-NES, the need to periodically rotate Access Keys to prevent signature accumulation is completely eliminated, as Access Keys are no longer authorized to sign network transactions under this architecture.

### 4. Fourth Defense Layer: Long-Term Digital Asset Storage

### An Account That Only Receives Assets Has a Different Attack Surface

To understand the importance of security architecture, we must consider a simple blockchain account. An account can be created, have an address, and receive assets without ever performing an outgoing transaction.

In this state, the blockchain only needs an account identifier or address to attribute incoming assets. The account is identified by an address, and a transfer transaction, after signing and validation, changes the account's asset status without requiring the destination account to be online.

The important security point:

"Receiving assets is not the same as spending assets."

As long as the account only receives assets, the only information published about the account on the blockchain is its address or identifier. Therefore, there is no data available on-chain to enable signature analysis threats or algorithms like Shor's algorithm.

This highlights a key principle: if an Access Key is not used in public or operational transactions, no signature history for that key exists for future analysis.

Everything changes with the first outgoing transaction. With the first transaction from the account, a signature is recorded in the network history, and the Access Key's public key is exposed for validation purposes.

### The Best Defense Layer for an Account with a Very High Balance

If an account is created to hold a large amount of assets for long-term storage, one of the strongest strategies to reduce the signature analysis attack surface is:

"Avoid sending any operational outgoing transactions from that account if at all possible."

The reason is simple: before the first outgoing transaction, only the account identifier is publicly available. With the first transaction, the necessary data for signature analysis—including the public key and signature—becomes accessible to attackers.

Each new transaction generates more cryptographic data, which is recorded as transaction history and becomes available to potential attackers.

This principle is very similar to the foundation of the OTAK-NES security architecture: non-exposure of Access Key signatures. In OTAK-NES, keys with access to account assets are not used to sign operational transactions. All operational transactions are performed using one-time Child Keys derived from the Access Keys.

### OTAK-NES: A Defense Layer for the Network and Account

OTAK-NES does not claim to replace:

- The user
- The wallet
- Seed/Backup
- The signature algorithm
- Consensus
- Or network policies

Rather, it is a complementary architectural layer that redesigns the signing authority lifecycle.

In the OTAK-NES architecture:

- The Access Key remains the long-term root of trust.
- The Access Key does not directly sign everyday operational transactions.
- Child Keys are used for operational transactions.
- Each Child Key has its own distinct public/private key pair.
- Each Child Key is valid for only one transaction.
- After successful validation, the Child Key is first deactivated, then its signature is recorded in the network history.
- Child Key membership in the authorized set can be verified via the Merkle Root.
- Access Key public and private keys are never published as signatures under the OTAK-NES architecture.

Therefore, OTAK-NES addresses precisely where other layers do not provide a complete answer or coverage:

- Why should a long-term key be able to produce signatures repeatedly when the threat is serious?
- If an Access Key is broken one day, why should that compromise mean the entire account and all its assets are lost?

In conventional architectures, a long-term key may sign hundreds, thousands, or even millions of transactions during an account's lifetime.

In the OTAK-NES security architecture, each transaction is submitted with a one-time Child Key. After successful network validation, the Child Key is deactivated, and only then is its signature recorded in the network history.

This change provides an important architectural advantage:

"If a Child Key is compromised for any reason, the attacker faces a temporary, disposable operational key, not a key that has produced thousands of public signatures over the years and has access to all account assets."

Post-quantum signature algorithms attempt to counter future threats by making cryptography harder. Meanwhile, OTAK-NES limits the attack impact by shrinking the threat scope and, in the presence of sequential transaction validation, attempts to defeat an attack even if it succeeds.

These are two different problems. They complement each other but do not overlap. Additionally, OTAK-NES answers a critical question:

### What If the Signature Algorithm Is Broken One Day?

If an attacker in the future can:

- Find an implementation vulnerability
- Discover a new cryptographic attack
- Exploit a flaw
- Perform a successful side-channel attack
- Or even gain access to a powerful quantum computer

In conventional architectures, compromising an Access Key can be extremely dangerous because the same key may have a long history of operational authorizations and could still be valid for future transactions.

In OTAK-NES, the architectural goal is that:

- Breaking a Child Key ≠ Breaking the Access Key
- Breaking one transaction ≠ Breaking the entire account and its assets
- Breaking one cryptographic layer ≠ Breaking all security layers
- A network's defense layers are not limited to post-quantum signatures alone

This is the philosophy of multi-layered defense in a network with the help of the OTAK-NES architecture.
___
___
# Identified Signature-Related Attacks in Blockchain Systems

Throughout the history of blockchain systems, numerous attacks have exploited weaknesses in digital signature generation, cryptographic implementations, or signature verification mechanisms. The following summarizes the most relevant categories of signature-related attacks.

### 1. Nonce Reuse Attacks

In ECDSA, if two signatures are generated using the same nonce (k), the corresponding private key can be recovered mathematically.
Several academic studies have demonstrated that nonce reuse has exposed thousands of private keys across multiple blockchain ecosystems, resulting in severe financial losses.

### 2. Replay Attacks

Replay attacks occur when a valid signature intended for one transaction or context is reused in another context.
Without sufficient contextual binding, an attacker may reuse previously valid signatures to authorize unintended operations.

### 3. Signature Malleability

ECDSA signatures are inherently malleable.
An attacker may generate an alternative valid representation of the same signature without knowing the private key, potentially causing replay scenarios or transaction consistency problems if protocols do not properly normalize signatures.

### 4. Fault Injection Attacks

Fault injection attacks intentionally disturb the signing process through hardware manipulation, voltage glitches, clock modifications, laser injection, or similar techniques.
Such attacks may reveal sensitive information or cause abnormal signature generation.

### 5. Side-Channel Attacks

Side-channel attacks attempt to recover secret information by observing physical characteristics of the signing process, including timing behavior, power consumption, cache activity, or electromagnetic emissions.
These attacks are particularly relevant for hardware wallets and dedicated signing devices.

### 6. Signature Forgery

Signature forgery attacks attempt to construct signatures that satisfy protocol verification without possessing the legitimate private key.
Although modern signature algorithms are designed to prevent such attacks, implementation flaws or protocol weaknesses may increase practical risk.

### 7. Quantum Cryptanalysis

Future large-scale quantum computers executing algorithms such as Shor's algorithm may threaten classical public-key cryptography, including ECDSA, by making recovery of private keys from exposed public keys theoretically possible.
OTAK-NESA is not a post-quantum signature algorithm and does not replace post-quantum cryptography.

Instead, its objective is architectural:

- minimize long-term exposure of authorization keys;
- ensure that publicly exposed signatures belong only to disposable Child Keys;
- confine any future cryptanalytic breakthrough to temporary authorization keys rather than permanent account authorization keys.

For complete quantum resistance, OTAK-NESA is intended to complement future post-quantum signature algorithms rather than replace them.

### 8. Examples of Recent Incidents

Recent blockchain security reports continue to demonstrate that signature-related vulnerabilities remain an important practical concern.

Incidents involving weak nonce generation, compromised signing infrastructure, replay vulnerabilities, and exposed signing keys illustrate that long-term signature exposure continues to represent an important attack surface across blockchain ecosystems.

### How OTAK-NESA Mitigates These Threats

Rather than modifying the underlying signature algorithm, OTAK-NESA changes the authorization architecture.

Long-term authorization keys never participate in routine transaction signing. Instead, every transaction is authorized using a disposable Child Key whose membership is verified through a Merkle Proof.

- Nonce Reuse:
A nonce reuse event can only affect the corresponding one-time Child Key. Long-term authorization keys remain isolated from transaction signing.
- Replay:
Replay opportunities are reduced by binding each Child Key to a unique authorization state and one-time consumption rules.
- Signature Malleability:
Signature malleability may still exist at the algorithm level, but its practical impact is limited because Child Keys cannot be reused after successful authorization.
- Fault Injection:
Any successful attack is confined to a disposable Child Key rather than exposing long-term authorization keys.
- Side-Channel Attacks:
Information leakage from a Child Key affects only that temporary authorization key and does not directly expose long-term authorization keys.
- Signature Forgery:
An attacker must simultaneously satisfy signature verification and produce a valid Merkle Proof corresponding to the authorized key hierarchy.
- Quantum Cryptanalysis:
OTAK-NESA minimizes long-term authorization-key exposure and is designed to operate alongside future post-quantum signature algorithms.

### Conclusion

OTAK-NESA does not attempt to eliminate every cryptographic attack at the signature algorithm level.

Instead, it introduces a security architecture in which permanent authorization keys remain isolated from routine transaction signing while disposable Child Keys are used for transaction authorization.

As a result, the impact of many well-known signature-related attacks can be significantly reduced and confined to temporary authorization keys, while the long-term exposure of permanent authorization keys is minimized.

This architectural approach is independent of the underlying signature algorithm and can therefore operate alongside both existing classical signature schemes and future post-quantum signature algorithms.
___
___
# Why Merkle Trees?
 
Merkle Trees have become one of the fundamental cryptographic building blocks of modern blockchain systems. Their primary purpose is to provide efficient, verifiable, and scalable authentication of large collections of data without requiring every participant to process the entire dataset.
Rather than serving only as a storage optimization mechanism, Merkle Trees enable decentralized systems to verify integrity, detect unauthorized modification, and authenticate individual elements using compact cryptographic proofs.
 
### Adoption Across Blockchain Systems
The widespread adoption of Merkle Trees across major blockchain platforms demonstrates their importance as a general-purpose cryptographic authentication mechanism.
 

Bitcoin
 
Bitcoin uses a Merkle Tree to summarize every transaction contained within a block into a single **Merkle Root**, which is stored in the block header.
This design allows Simplified Payment Verification (SPV) clients to verify the inclusion of individual transactions using only a Merkle Proof instead of downloading the entire block, significantly reducing bandwidth and storage requirements.


Ethereum
 
Ethereum extends the concept by employing Merkle Patricia Tries to authenticate the complete blockchain state, including accounts, balances, storage, and smart contract data.
State transitions are verified by computing new authenticated state roots after transaction execution, allowing every node to independently verify network consistency.


Layer-2 Scaling Systems
 
Modern Layer-2 protocols, including optimistic and zero-knowledge rollups, extensively use Merkle Trees to authenticate large batches of off-chain transactions while publishing only compact commitments to the base blockchain.
This dramatically improves scalability without sacrificing cryptographic verifiability.
 

Data Availability Systems
 
Projects focused on modular blockchain architectures and data availability also rely on Merkle Trees to allow nodes to verify data integrity without downloading complete datasets.
  
### Security Properties of Merkle Trees
Merkle Trees provide several important security properties that have made them a standard component of blockchain architecture.


Data Integrity
 
Any modification to a single leaf changes every hash along the authentication path up to the Merkle Root.
Consequently, unauthorized modification becomes immediately detectable.
 

Efficient Verification
 
Membership verification requires only a compact Merkle Proof rather than the complete dataset.
Verification complexity grows logarithmically with the number of leaves, making the mechanism highly scalable.
 

Decentralized Verification
 
Every node can independently verify cryptographic proofs without relying on trusted third parties or centralized databases.
 

Scalability
 
Merkle Trees allow authentication of extremely large datasets while maintaining relatively small proof sizes, making them practical for blockchain systems with millions of authenticated objects.
  
### Security Considerations
 
Like every cryptographic construction, Merkle Trees inherit the security assumptions of their underlying hash functions.
Their practical security therefore depends primarily upon:
 
- collision resistance;
- second-preimage resistance;
- secure implementation;
- correct tree construction.

Modern blockchain implementations mitigate known implementation risks through standardized tree construction rules and well-established cryptographic hash functions.
  
### Why OTAK-NESA Uses Merkle Trees
 
OTAK-NESA adopts Merkle Trees because they provide an efficient and decentralized mechanism for authenticating large sets of disposable authorization keys without requiring every key to be stored or transmitted during transaction validation.
Instead of registering every Child Key individually, OTAK-NESA commits the complete authorization set through a single Merkle Root.
Each transaction carries only:
 
- the Child Key;
- its Merkle Proof;
- the transaction signature.

Network nodes verify membership by comparing the supplied proof against the committed Merkle Root stored in the account state.

This approach provides several architectural advantages:
 
- compact representation of potentially millions of Child Keys;
- logarithmic verification complexity;
- decentralized verification without trusted intermediaries;
- efficient bandwidth and storage usage;
- cryptographic integrity of the authorization hierarchy;
- compatibility with large-scale blockchain systems.
 
### Merkle Trees within the OTAK-NESA Architecture
 
Within OTAK-NESA, the Merkle Tree is not merely a data structure.

It serves as the cryptographic commitment representing the entire authorization hierarchy.
 
Long-term authorization keys remain isolated from routine transaction signing, while temporary Child Keys are authenticated through Merkle Proofs.
 
This architecture enables scalable one-time authorization without exposing permanent authorization keys during normal blockchain operation.
___
___
# Auditability
 
### 1. Auditability in Conventional Blockchain Systems
 
One of the fundamental properties of blockchain systems is that every transaction remains permanently verifiable.
 
Each transaction contains a digital signature that is permanently recorded on-chain. As a result:

- Any participant can independently verify any historical transaction at any time
- The cryptographic relationship between the transaction, its signature, and the corresponding public key remains publicly verifiable
- Auditors, validators, exchanges, regulators, and users can independently validate historical transactions without requiring access to private keys or trusted third parties

This permanent public verifiability is one of the foundations of blockchain transparency and decentralization.
  
### 2. How OTAK-NES Preserves Auditability
 
OTAK-NES does not change the auditability model of blockchain systems.
 
Instead, it changes **which keys are exposed through transaction signatures.**

Under OTAK-NES:

- Every transaction is signed by a one-time Child Key.
- Child Keys are deterministically derived from long-term authorization keys using the blockchain's supported hierarchical deterministic (HD) derivation mechanism
- The Child Key signature is permanently recorded on-chain according to the normal blockchain protocol
- The Child Key public key remains publicly available for signature verification.
- A Merkle Proof included in the transaction proves that the Child Key belongs to the account's authorized key hierarchy
- The corresponding Merkle Root is stored in the account state and remains publicly verifiable

Consequently, any participant can independently verify:

1. the digital signature

2. the validity of the Merkle Proof

3. the inclusion of the Child Key in the authorized hierarchy

4. that the Child Key had not been consumed before execution

5. the complete transaction history of the account

No trusted third party is introduced, and no additional off-chain information is required.

### 3. Auditability After Child Key Consumption

After a transaction is finalized, the Child Key is marked as **Consumed**.

This state transition prevents future reuse of that Child Key.

Importantly:

- The transaction signature remains permanently stored on-chain.
- Historical verification remains fully possible.
- Merkle Proof validation remains valid indefinitely.
- The consumed status only affects future authorization, not historical auditability

Therefore, OTAK-NES separates  authorization validity  from  historical verifiability.
  
### 4. Preservation of Decentralization
 
OTAK-NES preserves all core decentralization principles.

Specifically:

- Every transaction remains publicly verifiable
- Every cryptographic proof can be independently validated
- No trusted coordinator exists
- No centralized key registry is required
- Every validator reaches identical verification results using only publicly available blockchain data.

The Merkle Root stored in the account state acts as the immutable commitment to the authorized Child Key hierarchy, while Merkle Proofs provide efficient membership verification.

### 5. Difference from Conventional Blockchain Authorization

In conventional blockchain systems:

- Long-term authorization keys repeatedly sign transactions throughout the lifetime of an account.
- Every signature contributes additional publicly available cryptographic material associated with the same long-term authorization key

In OTAK-NES:

- Long-term authorization keys are reserved exclusively for authorization management
- Daily transactions are performed exclusively by one-time Child Keys
- Every Child Key authorizes only a single transaction before becoming permanently consumed

As a result, long-term authorization keys no longer accumulate large numbers of publicly exposed transaction signatures over the lifetime of the account.

The transparency, auditability, and decentralized verification properties of blockchain remain unchanged.

Only the authorization architecture changes.

### 6. Conclusion

OTAK-NES preserves every property required for public blockchain auditability.

Every transaction remains:

- publicly verifiable
- cryptographically provable
- permanently auditable,
- independently validated by every node

The architecture simply replaces repeated exposure of long-term authorization keys with independently verifiable one-time authorization keys.

This preserves blockchain transparency while significantly reducing the long-term exposure of high-authority authorization keys without sacrificing decentralization or auditability.
___
___
# Easy Implementation (Non‑mandatory)

One of the advantages of OTAK‑NES is its ability to be deployed easily and incrementally. Rather than requiring a disruptive, one‑time network‑wide migration, this model proposes a gradual adoption path that allows blockchain networks to introduce the new architecture while preserving backward compatibility throughout the transition.

This section describes one possible deployment strategy. It is provided as an implementation recommendation, not as a protocol requirement.

### Minimal Protocol Requirements

OTAK‑NES does not require:

- a new digital signature algorithm
- modifications to the consensus mechanism
- changes to block production
- changes to transaction ordering
- trusted third parties
- or centralised coordination

Instead, it builds on existing cryptographic capabilities already widely used in modern blockchain ecosystems, including:

- Hierarchical Deterministic (HD) key derivation (BIP32/BIP44, or network‑specific standards such as TRC‑102)
- Merkle Trees
- standard digital signatures
- and existing account‑permission models

As a result, OTAK‑NES introduces a new authorisation architecture rather than replacing the underlying cryptographic infrastructure.

### Phase 1 — Bootstrap Infrastructure

The first deployment stage introduces only the supporting infrastructure required by OTAK‑NES.

At this stage, the network becomes capable of:

- storing the account's Merkle Root
- recording Bootstrap completion
- maintaining the account's Child Key state
- and tracking the last authorised Child Key index

No changes are made to normal transaction authorisation.

All existing transactions continue to be signed using the conventional authorisation keys.

From a user's perspective, the network behaves exactly as before.

### Phase 2 — Wallet Support

Once the Bootstrap infrastructure is in place, wallet software can begin supporting OTAK‑NES.

Wallets become capable of:

- deriving Child Keys
- constructing Merkle Proofs
- creating OTAK‑NES transactions
- and performing Bootstrap initialisation

At this stage, the blockchain can support both authorisation models:

- traditional Access‑Key signatures
- and OTAK‑NES Child‑Key signatures

This enables gradual ecosystem adoption without disrupting existing users.

### Phase 3 — Dual Validation in a Specific Domain

In this model, a specific domain is first selected. During the transition period, transactions in that domain can be validated using either authorisation model.

If a transaction in the chosen domain is signed using the conventional authorisation model, validation proceeds exactly as it does today.

If the transaction is authorised using OTAK‑NES, nodes additionally verify:

- the Child Key signature
- the Merkle Proof
- Child Key membership
- Child Key consumption status
- and sequential Child Key usage

Both authorisation models remain usable within the chosen domain during this phase. This allows wallets, infrastructure providers, and the network to migrate safely to the new architecture.

The dual‑validation period is limited and temporary — it is not permanent. Once the network is confident in the stability and correct functioning of OTAK‑NES in that domain, it should make OTAK‑NES enforcement mandatory for that domain.

### Phase 4 — Domain‑by‑Domain Expansion

Under this proposal, migration to the OTAK‑NES architecture is performed domain by domain, rather than all at once.

Each domain goes through the following cycle:

1. Child Key Support

   The designated domain supports authorisation based on OTAK‑NES Child Keys.

2. Dual‑Validation Period

   During this period, transactions in the domain can be validated using either authorisation model:

- Access Keys
- or OTAK‑NES Child Keys

   This period lasts until the network is satisfied that OTAK‑NES is stable and ready for enforcement in that domain.

3. **OTAK‑NES Enforcement in the Domain**

   After the transition period ends, the network establishes OTAK‑NES as the sole authorisation model for that domain.

   In this state, transactions that do not satisfy OTAK‑NES requirements are rejected with a `CHILD_KEY_REQUIRED` error.

   Consequently, after a domain is fully migrated, Access Keys are no longer used for operational transactions in that domain.

4. **Expansion to the Next Domain**

   Once migration of the previous domain is complete, the next domain enters its dual‑validation period.

   Thus, at any given time, it is possible that:

   - Domain A → OTAK‑NES‑only
   - Domain B → Dual Validation

   This process continues for subsequent domains.

5. **Full OTAK‑NES Coverage**

   This model continues until all domains of the network have migrated to OTAK‑NES. Ultimately, as the architecture expands to the entire network, OTAK‑NES becomes the sole operational authorisation model.

   Domains such as transfers of a specific asset type, token transfers, staking, governance voting, and smart‑contract interactions can each enter this process according to their own readiness and requirements.

### Phase 5 — Full OTAK‑NES Authorisation

After a domain has been fully migrated, and ultimately after all target domains have been covered and OTAK‑NES is deployed network‑wide, network nodes accept only transactions that satisfy OTAK‑NES validation requirements.

Specifically, an OTAK‑NES transaction must:

- be signed by a valid Child Key
- include a valid Merkle Proof
- reference the next valid unconsumed Child Key according to the account state
- and satisfy all network authorisation validation rules

Following successful Bootstrap completion and full migration of a domain to OTAK‑NES, Access Keys no longer function as operational transaction‑signing identities for that domain.

Operational transactions in that domain are authorised exclusively through valid Child Keys derived from the corresponding Access Key.

The only exception is the initial Bootstrap transaction in networks that do not support Zero‑Knowledge proofs for Merkle Root registration. In such blockchain networks, wallets create two keys with different private keys simultaneously during account creation:

- Owner Key
- Active Key

The Active Key then registers the Merkle Root corresponding to the Owner Key, and after successful Bootstrap completion, that Active Key is automatically deactivated by the network in accordance with OTAK‑NES requirements.

After each domain is fully migrated, long‑term Access Keys no longer produce operational on‑chain signatures for that domain.

In migrated domains, all subsequent account operations — including permission updates, Merkle Tree rotation, and multisignature transactions — are authorised according to OTAK‑NES rules, exclusively through valid Child Keys derived from the Owner Key or Active Keys, in line with the network's protocols.

### Advantages of Incremental Deployment

The staged deployment model offers several important advantages:

- No disruptive network‑wide migration.
- Backward compatibility throughout the transition.
- Phased adoption by wallets, exchanges, validators, and infrastructure providers, allowing safe migration.
- Minimal operational risk.
- Progressive activation as the ecosystem matures.
- No dependency on immediate protocol‑wide adoption.

Most importantly, blockchain networks may introduce OTAK‑NES gradually without interrupting existing operations or forcing all participants to migrate immediately.

### Architectural Flexibility

OTAK‑NES intentionally separates the architecture from its deployment strategy.

The security model remains identical regardless of whether a blockchain adopts OTAK‑NES immediately or through multiple deployment phases.

Consequently, each blockchain ecosystem may choose a migration schedule that best fits its governance model, infrastructure maturity, operational constraints, and security objectives.

This approach provides the flexibility needed for gradual migration to OTAK‑NES and does not require networks to adopt the architecture all at once.
___
___
# Secure Bootstrap Protocol

### Native OTAK-NES Approach Compared with Zero-Knowledge-Based Designs

From a theoretical perspective, one of the most ideal methods for proving ownership without disclosing sensitive information is the use of Zero-Knowledge Proof systems. This technology can prove the validity of a claim to the network without revealing confidential information, making it one of the most advanced tools in modern cryptography.

However, integrating a complete Zero-Knowledge infrastructure directly into a Layer-1 blockchain requires new cryptographic components, including specialized proving systems, verification logic, and significant protocol-level engineering changes. Furthermore, generating and verifying Zero-Knowledge proofs in many applications entails considerable computational cost and implementation complexity.

OTAK-NES follows a different architectural approach. The objective of this architecture is not to compete with or replace Zero-Knowledge technologies, but to provide a native and innovative solution that is simpler and more compatible with existing blockchain architectures, capable of delivering a significant portion of the desired security benefits without relying on Zero-Knowledge infrastructure.

To achieve this goal, OTAK-NES leverages existing network capabilities, including the Permission model, public derivation, Merkle Trees, one-time Child Keys, and minimal account-state extensions. This approach seeks to prevent long-term exposure of signatures from keys with the highest level of authority while maintaining compatibility with the current network architecture, thereby substantially reducing the attack surface.

Should networks in the future adopt complete Zero-Knowledge infrastructure, OTAK-NES can serve as a complementary security layer alongside them. Thus, these two approaches are not competitors; they can complement each other in future architectures.

### The Merkle Root Registration Challenge in Bootstrap

The core mission of OTAK-NES is to prevent the disclosure of Access Key signatures. However, the network cannot register a Merkle Root without a signature from an Access Key, and without the Merkle Root, Child Key validation is impossible.

### OTAK-NES Solution for Networks That Do Not Yet Support Zero-Knowledge: Deactivating the Active Key After Merkle Root Registration in Bootstrap

In blockchain networks such as TRON, wallets can create two keys simultaneously during account creation, each with its own independent private key:

- Owner Key: the key with the highest level of authority.
- Active Key: an operational authorization key.

Although some implementations may allow both permissions to share the same private key, OTAK-NES requires the Owner Key and Active Key to be generated from independent private keys in order to preserve their security independence.

This capability already exists in TRON (based on TIP-16) and is technically feasible in other blockchain networks as well.

**Solution:** Instead of using the Owner Key to register the Owner Key's Merkle Root, OTAK-NES uses the Active Key created alongside the Owner Key during account creation. After the Owner Key's Merkle Root is successfully registered in Bootstrap, this Active Key is immediately and automatically deactivated by the network according to the OTAK-NES architecture.

**What does deactivation mean?** It means the network removes the Active Key that registered the Owner Key's Merkle Root from the account's permitted list (Permission.keys). Therefore, even if a future threat targets this signature or an algorithm succeeds in recovering the private key of this Active Key, it cannot be used to sign any valid transaction because the network no longer recognises that key as a valid signer.

**Why does this solution work?**

- Active Keys are independent:** Each Active Key has a private key distinct from the Owner Key and other Active Keys. Deactivating one Active Key does not affect the Owner Key or other Active Keys.
- Owner Key security is preserved:** The Owner Key is never used to sign blockchain transactions, so no operational signature from it is ever recorded on-chain.
- Other Active Keys and the Owner Key remain fully independent: The account can continue using other Active Keys independently if required.
- Compatibility with existing permission models: This method is fully compatible with permission structures such as TIP-16 and TIP-105 in TRON and requires no redesign of the permission model.

**Complete process:**

1. The user creates an account consisting of an Owner Key and an Active Key with independent private keys.
2. Using the Active Key, the user signs the Bootstrap transaction and registers the Merkle Root corresponding to the Owner Key.
3. After successful confirmation, the network stores the Merkle Root in Bootstrap and, according to OTAK-NES architectural requirements, automatically deactivates the Active Key that registered the Owner Key's Merkle Root.
4. After this Active Key is deactivated, it can no longer sign any transaction.
5. All subsequent transactions are authorized exclusively by Child Keys derived from the Owner Key.
6. New Active Keys may be added to the account using Child Keys derived from the Owner Key, each with its own separate Merkle Tree registered in Bootstrap.

**Security outcome:** This method cannot completely eliminate the publication of an Access Key signature, but it fully neutralizes the long-term operational impact of that signature and preserves the long-term security model of the account.

### Step-by-Step Description

### Step 1: Wallet

At account creation, the wallet creates an Owner Key and an Active Key with independent private keys.

The wallet derives a predefined set of Child Keys from the Owner Key according to a hierarchical deterministic derivation standard (e.g., TRC-102).

The wallet constructs the Owner Key's Merkle Tree.

The wallet computes the corresponding Merkle Root.

The wallet submits the Bootstrap Request for the Owner Key using the account's Active Key, not the Owner Key.

The Bootstrap Request includes:

- Account address
- Owner Key Merkle Root
- OTAK-NES security parameters
- Initial ownership proof data

### Step 2: Bootstrap Validation

Network nodes verify:

- Initial ownership proof
- Submitted security parameters
- Merkle Root validity
- Protocol compliance

### Step 3: Bootstrap Result Recording

After successful consensus:

- The Owner Key's Merkle Root is stored
- `BootstrapCompleted = true` is set
- The Active Key used for Bootstrap is automatically deactivated by the network according to the OTAK-NES architecture after successful completion

### Transaction Validation

For every operational transaction:

**Wallet:**

- Selects the next unused Child Key.
- Generates the corresponding Merkle Proof.
- Signs the transaction.

**Network nodes verify:**

- The Child Key signature is valid.
- The Merkle Proof is valid.
- The Child Key belongs to the registered Merkle Root.
- The Child Key index matches `last_used_child_index + 1`.

**After successful confirmation:**

- The transaction is executed.
- `last_used_child_index` is updated.
- The signature is permanently recorded according to normal blockchain rules.
- That Child Key can never authorize another transaction again.

### Account Recovery

In OTAK-NES, Access Keys and Child Keys are deterministically derived from the user's Seed.

The network stores neither:

- Private keys
- Seed phrases
- Derivation secrets

During recovery, the wallet:

- Regenerates the Access Keys.
- Regenerates the Child Key hierarchy.
- Rebuilds the Merkle Tree.
- Recomputes the Merkle Root.
- Verifies that it matches the value stored on-chain.

Previously consumed Child Keys cannot be reused after recovery because network validation always relies on `last_used_child_index`.

### Required Protocol Changes

To support OTAK-NES, the protocol requires:

- Add a `BootstrapCompleted` field.
- Store a Merkle Root for every Access Key — each Access Key has its own independent Merkle Tree.
- Add Merkle Proof validation.
- Maintain `last_used_child_index` for sequential Child Key validation.
- Automatically deactivate the Bootstrap Active Key after successful Merkle Root registration.
- Reject operational transactions: after full network migration to OTAK-NES (not phased migration), and after Bootstrap completion, only Child Keys are authorized to submit transactions. Therefore, nodes must reject any transaction signed directly by Access Keys and return a `CHILD_KEY_REQUIRED` error to the wallet.

### Security Outcome

After successful Bootstrap completion, all operational signatures published on-chain belong exclusively to Child Keys.

**Consequently:**

- Access Keys never appear as operational signing identities.
- No long-term signature history from Access Keys accumulates on-chain.
- Every transaction uses an independent cryptographic identity.
- Every Child Key becomes permanently unusable after successful execution.
- Compromising one Child Key does not compromise Access Keys or other Child Keys.
- The impact of any security incident remains limited to a single transaction.
- The account establishes its complete OTAK-NES security model before engaging in normal economic activity.
___
___
# OTAK-NES Architecture

### 1. Access Key Restriction After Bootstrap Completion

In the OTAK-NES architecture, if the network does not support Zero-Knowledge proofs for the initial Merkle Root registration during Bootstrap, the wallet creates an account with both an Owner Key and an Active Key simultaneously during account creation, each with its own independent private key.

Although some implementations or wallets may allow assigning a shared private key to both the Owner Key and the Active Key, OTAK-NES does not permit this model. It defines the creation of the Owner Key and Active Key with independent private keys as an architectural requirement, to ensure that the independent security of these two keys is fully maintained.

The first transaction, according to the OTAK-NES security architecture, is the Bootstrap transaction, signed using the Active Key created at account creation. This transaction is exclusively used to register the Merkle Root of the Owner Key, thereby authorising transactions signed by Child Keys derived from that Owner Key.

After the Owner Key's Merkle Root is successfully registered in Bootstrap, the network deactivates the Active Key.

This transaction is identified at the network level as an initialisation transaction, not as a regular fee-based transaction.

Therefore, after account creation, nodes reject any transaction signed by Access Keys (other than the Active Key Bootstrap transaction for registering the Owner Key's Merkle Root) as an unsuccessful transaction, returning it to the account.

From this point forward:

All operational transactions must be signed with Child Keys and submitted to the network. Consequently, network nodes return any transaction signed using Access Keys (other than Child Keys) as unsuccessful.

Thus, after the successful registration of the Owner Key's Merkle Root by the Active Key and its subsequent deactivation:

- Only transactions authorised by valid, unused Child Keys are accepted.
- Each Child Key must provide a valid signature, a valid Merkle Proof, and proof of membership in the account's registered Merkle Root when submitting a transaction.
- After successful execution, the Child Key is permanently marked as consumed and cannot submit another transaction.

Therefore, after full migration to OTAK-NES, Access Keys no longer function as operational transaction signing identities. Their role is limited to maintaining the root of trust and deriving authorised Child Keys.

This architectural restriction ensures that long-term authorisation keys do not continuously produce transaction signatures on-chain, thereby eliminating the accumulation of long-term signature exposure while preserving full blockchain transparency, auditability, and decentralised verification.

In a phased deployment model, networks may temporarily support both Access-Key-based signatures and Child-Key-based signatures derived from Access Keys during the migration period, and can plan the gradual migration based on domain or transaction type. However, after full OTAK-NES activation, direct transaction signing by Access Keys for operational activities is permanently disabled, and any transaction submitted without a Child Key signature is returned to the wallet as unsuccessful.

OTAK-NES is founded on the principle of **non-exposure of Access Key signatures**.

In this architecture, keys with account authority, including the **Owner Key** and **Active Keys**, are never directly used to sign operational on-chain transactions.

These keys act solely as the **Root of Trust** and provide the basis for hierarchically deriving Child Keys in the wallet.

After successful Bootstrap completion, all account operations are performed exclusively through **Child Keys**.

If a transaction signed directly by an Access Key is submitted, network nodes must return that transaction as unsuccessful.

### 2. Child Key Generation

The wallet derives a set of Child Keys from the Access Keys using a hierarchical deterministic derivation standard (such as TRC-102 on TRON).

Each Child Key:

- Has an independent public/private key pair.
- Is cryptographically independent of other Child Keys.
- Is valid for only one transaction.
- Cannot be used to recover the Access Key due to the one-way nature of the derivation process.
- After successful transaction confirmation, its identifier is recorded as **Consumed** in the network state.
- Only after this status is recorded are the transaction and its signature published according to normal blockchain rules.

### 3. Merkle Tree Structure

In the OTAK-NES architecture, Child Keys are organised into a **Merkle Tree**.

The wallet, locally and offline:

- Generates the Child Keys.
- Builds the Merkle Tree.
- Computes the Merkle Root.

During the Bootstrap process:

- Only the Merkle Root is stored in the account state.
- The Child Keys themselves are not stored on the network.
- The network does not retain any version of Child Key public or private keys.

The Merkle Root serves as the cryptographic root of trust for validating all Child Keys of that account.

### 4. Information Stored in Account State

The network does not need to store a list of Child Keys.

The OTAK-NES-related state for each account includes only the following minimal information:

- merkle_root
- bootstrap_completed
- last_used_child_index
- and other information required based on each network's protocols.

### 5. Transaction Submission Process

For each transaction:

- The wallet selects the first unused Child Key.
- Signs the transaction with it.
- Generates the corresponding Merkle Proof.
- Submits the transaction, signature, and Merkle Proof to the network.

### 6. On-Network Validation

Upon receiving a transaction, network nodes:

- Verify the digital signature.
- Validate the Merkle Proof.
- Check its correspondence with the Merkle Root stored in the account.
- Confirm that Bootstrap has been completed.
- Verify that the Child Key has not been previously consumed.
- Verify that the Child Key index exactly matches `last_used_child_index + 1`.

After successful confirmation:

- The transaction is executed.
- last_used_child_index  is updated.
- The transaction signature is permanently recorded according to network protocols.
- That Child Key can never be used again.

In this process, nodes do not need to reproduce the derivation process or verify the cryptographic relationship between the Access Key and the Child Key.

### 7. Child Key One-Time-Use Model

Each Child Key can be used only once.

After successful confirmation:

- The transaction is executed.
- The Child Key is considered consumed.
- The possibility of reusing it is permanently eliminated.

If an attempt is made to reuse a consumed Child Key:

- The transaction is rejected.
- A `CHILD_KEY_CONSUMED` error is returned.

### 8. Child Key Management in the Wallet

All Child Keys are generated completely deterministically.

Consequently:

- All Child Keys can be recovered from a single Seed.
- The wallet can locally generate millions of Child Keys.
- There is no need to register keys separately on the network.
- No additional backup is required.
- Full recovery of the account structure is possible using only the master Seed.

### 9. Child Key Tracking Model

OTAK-NES does not require maintaining a set of consumed keys, a bitmap, or a list of used Child Keys.

Instead, the network only maintains the `last_used_child_index` value.

This model:

- Prevents account state growth.
- Keeps the state size of each account constant.
- Does not require pruning or archiving of consumed Child Keys.
- Naturally prevents replay attacks.

### 10. Rotation Window

To prevent the exhaustion of Child Keys in a Merkle Tree, OTAK-NES introduces the concept of a **Rotation Window**.

The Rotation Window is a portion at the end of each active Merkle Tree that is unusable for ordinary transactions and is reserved exclusively for Merkle Tree replacement.

When the wallet enters the Rotation Window, it automatically:

- Generates a new set of Child Keys.
- Builds a new Merkle Tree.
- Computes the new Merkle Root.
- Submits a Rotation transaction using one of the authorised Child Keys located in the Rotation Window.

After confirmation:

- The new Merkle Root replaces the previous one.
- last_used_child_index  is reset to zero.
- The new Merkle Tree becomes active.
- The previous Merkle Tree is permanently deactivated.
- All unused Child Keys in the previous Merkle Tree become permanently invalid.

### 11. Account Recovery

All Access Keys, Child Keys, and Merkle Trees are deterministically derived from the user's Seed.

The network never stores:

- The Seed
- Private keys
- Derivation secrets

During recovery, the wallet:

- Regenerates the key structure.
- Rebuilds the active Merkle Tree.
- Synchronises it with the `merkle_root` and `last_used_child_index` stored in the network state.

### 12. Shifting the Threat Model from the Entire Account and All Assets Under Its Control to the Specific Transaction and Asset Being Transferred

OTAK-NES provides a fundamental change in the blockchain threat model. Instead of allowing the exposure of an operational signature to potentially threaten the entire account and all assets under its control, the architecture limits the potential impact to the specific transaction and the specific asset being transferred. This capability is now available to the community.

OTAK-NES enables a reduction in the risk of exposing assets controlled by private keys by preventing the routine public exposure of signatures produced by those keys. The architecture is designed for practical implementation on existing blockchain networks.

Protection of submitted transactions during the network's internal validation process can be approached through different methods:

1. Post-Quantum Signatures: This approach increases computational resistance against quantum attacks, but by itself cannot reduce the threat model from the entire account to a specific transaction.

2. OTAK-NES Architecture: Beyond limiting the scope of the threat from the level of all controlled assets to the level of the asset being transferred, this architecture provides an important contribution to blockchain security. Key contributions of the architecture toward reducing the risk of public signature exposure include:

- The use of one-time Child Keys derived from Access Keys, so that each operational signature is produced independently of the account's long-term authorization keys.
- Ensuring that exposure of a Child Key, by itself, does not expose the Access Key or other Child Keys.
- Preventing replay attacks and Child Key reuse through mandatory sequential indexing of Child Keys.
- Reducing the risk associated with operational-key-level Nonce reuse by using each Child Key exactly once.
- Limiting the impact of any potential exposure to the level of a specific key and transaction, so that exposure of an operational key, by itself, does not imply exposure of the private key controlling all assets of the account.


Security Properties

The OTAK-NES architecture provides several important security advantages:

- Access Keys never sign operational transactions.
- No long-term history of Access Key signatures is created on-chain.
- Each transaction is signed by an independent cryptographic identity.
- Each Child Key can be used only once.
- If a Child Key is compromised, the impact of the attack is limited to that single transaction.
- This incident does not affect Access Keys, other Child Keys, or future transactions.
- The state size of each account remains constant regardless of the number of transactions.
- The account can continue operating indefinitely through successive rotations.

### 13. Multisig Account Support Through Independent Merkle Trees

OTAK-NES fully supports multisignature accounts while preserving its fundamental security principle:  Access Key signatures must never be exposed during normal account operation.

In a multisignature account, a Permission may contain multiple Access Keys, each with its own weight and signing authority. Rather than sharing a single Merkle Tree across all Access Keys, OTAK-NES assigns an **independent Merkle Tree** to every Access Key.

Each Access Key therefore maintains its own:

- Merkle Root
- Child Key hierarchy
- last_used_child_index
- Child Key lifecycle

This separation allows every Access Key to operate independently while remaining fully compatible with the existing Permission model.

Bootstrapping Additional Access Keys

During account creation, the Owner Key and its initial Active Key follow the Bootstrap procedure defined by OTAK-NES.

When an additional Active Key is later added to the account, the process follows the same architectural principle:

1. A valid Child Key derived from an already-authorized Access Key authorizes the creation of the new Active Key.
2. During this operation, the wallet generates a dedicated Merkle Tree for the new Active Key.
3. The corresponding Merkle Root is registered in the account state.
4. From that point forward, the new Active Key authorizes transactions exclusively through Child Keys derived from its own Merkle Tree.

As a result, every Access Key owns an independent authorization domain without affecting the operation of other keys.


Architectural Advantages

Independent State Management

Each Access Key advances through its own Child Key sequence independently. Consumption of Child Keys associated with one Access Key has no effect on the state of any other Access Key.

Security Isolation

If an individual Access Key or one of its Child Keys is ever compromised, the impact is confined to that Access Key's own authorization domain. Other Access Keys, their Merkle Trees, and their Child Key hierarchies remain unaffected.

This compartmentalization is a fundamental security property of OTAK-NES and significantly reduces the scope of any individual security incident.
 
**Simplified Verification**

Each signer submits:

- a transaction signature,
- a Child Key, and
- the corresponding Merkle Proof.

Network nodes validate each proof only against the Merkle Root registered for that specific Access Key. No cross-tree verification or coordination between different Merkle Trees is required.

**Scalable Permission Architecture**

Additional Access Keys can be introduced without modifying existing Merkle Trees or Child Key hierarchies. Every newly added Access Key simply receives its own independent Merkle Tree and begins operating within its own authorization domain.

This enables multisignature accounts to scale naturally while preserving deterministic validation.
 
**Algorithm Independence**

Because every Access Key owns an independent Merkle Tree, different Access Keys may employ different digital signature algorithms if supported by the network.

For example, one Access Key may authorize Child Keys using ECDSA while another uses a future post-quantum signature algorithm such as ML-DSA or Falcon.

This flexibility allows gradual migration to newer signature schemes without requiring changes to the overall OTAK-NES architecture.
 
Multisignature Transaction Flow

For a multisignature transaction:

1. Each signer derives an unused Child Key from its own Access Key.
2. Each signer signs the transaction using that Child Key.
3. Each signer includes the corresponding Merkle Proof.
4. Network nodes validate every signature against the Merkle Root associated with the corresponding Access Key.
5. Once the required signature threshold defined by the Permission is satisfied, the transaction is executed.
6. Every participating Child Key is permanently marked as consumed.
 
Security Outcome

Using an independent Merkle Tree for every Access Key preserves all core OTAK-NES security guarantees in multisignature accounts:
 
- Access Keys never directly sign operational transactions.
- Each Access Key maintains an isolated Child Key hierarchy.
- Compromise of one authorization domain does not affect others.
- Existing Permission models remain fully supported.
- Mixed signature algorithms can coexist within the same multisignature account.
- Multisignature accounts remain scalable, deterministic, and fully compatible with future cryptographic evolution.

This architecture extends the OTAK-NES security model to multisignature accounts without altering the underlying Permission framework while maintaining the protocol's core objective of eliminating long-term operational exposure of Access Key signatures.

### 14. A Distinct Approach to Security Architecture

Core Principle:
Non-Exposure of Long-Term Authorization Keys

The OTAK-NES architecture is built on a fundamental security principle: **long-term authorization keys should never be repeatedly exposed through operational transaction signing.**

Rather than focusing on making a single cryptographic algorithm harder to break, OTAK-NES eliminates the primary long-term attack surface by ensuring that Access Keys — the keys with the highest level of authority over an account — never publish signatures on the blockchain during routine operations.

Beyond Post-Quantum:
A Broader Security Framework

While post-quantum signature algorithms (such as ML-DSA, Falcon, or SLH-DSA) aim to resist quantum attacks by increasing cryptographic complexity — with larger signatures and higher verification costs — OTAK-NES takes a fundamentally different path.

Post-quantum signatures address quantum threats by hardening cryptography. OTAK-NES addresses the broader problem of long-term signature exposure by changing the authorization architecture itself.

This architectural separation provides multiple security benefits:

- Protection against known and emerging threats: The architecture helps protect against known quantum attacks (such as Shor's algorithm), while also reducing exposure to future cryptanalytic advances, implementation vulnerabilities, and side-channel attacks — without requiring any change to the underlying signature algorithm.
- Prevention of replay attacks: Because each Child Key is bound to a unique index and consumed immediately after use, replay attacks become impossible.
- Elimination of nonce reuse vulnerabilities: Each Child Key is used exactly once, eliminating the risk of nonce reuse and private-key recovery through signature analysis.
- Compartmentalization of security incidents: If any individual Child Key is compromised, the impact is confined to that single transaction. The attack does not propagate to Access Keys, other Child Keys, or future transactions.
- Defense against implementation vulnerabilities: Even if a vulnerability exists in the signature algorithm implementation, the exposure is limited to one-time Child Keys rather than long-term authorization keys.
- Protection against side-channel and fault attacks: These attacks typically target the signing process of a specific key. With one-time Child Keys, any successful attack is confined to a disposable key that becomes immediately invalid after use.

OTAK-NES as a Foundational Security Layer

OTAK-NES is not a new signature algorithm. It is a **key lifecycle architecture** that can operate alongside any signature scheme — whether classical ECDSA, post-quantum Falcon, or any future cryptographic algorithm.

This architecture provides a foundational security layer that reduces long-term exposure, limits the impact of security incidents, and preserves blockchain transparency, auditability, and decentralization.

In simplified terms:

- Post-quantum signature algorithms answer: How should a signature remain secure against quantum attacks?
- OTAK-NES answers: How should signing authority be distributed, delegated, limited, and retired throughout the lifetime of an account to minimize long-term exposure?

Because these operate at different architectural layers, they reinforce rather than replace one another — they do not overlap or compete. Combining strong cryptographic algorithms with a compartmentalized key lifecycle architecture provides a more resilient security model than relying on either approach alone.

Conclusion

OTAK-NES offers a comprehensive security architecture that:

- Eliminates long-term exposure of Access Key signatures on the blockchain.
- Protects against quantum threats without requiring changes to the signature algorithm.
- Prevents replay attacks through mandatory sequential key consumption.
- Eliminates nonce reuse vulnerabilities by using each Child Key exactly once.
- Limits the impact of any security incident to a single transaction.
- Provides a foundation for defense-in-depth that remains valuable regardless of future cryptographic developments.

Whether a blockchain uses classical ECDSA, post-quantum signatures, or any future algorithm, OTAK-NES provides an independent security layer that reduces the long-term exposure of its most critical authorization keys while preserving the transparency, auditability, and decentralization that make blockchains trustworthy.
___
___
# Threat Model

The OTAK-NES architecture is designed under the assumption that the long-term security of blockchain accounts depends not only on the strength of the underlying digital signature algorithm, but also on the duration of signing authority exposure and the **size and scope of a threat**.

Rather than assuming that attacks on a private key will not occur, or relying on the permanent security of a specific signature type or cryptographic primitive, or assuming that a transaction signature alone provides complete cryptographic security for an account and that publishing it creates no direct threat to the account's private key—although no signature algorithm, including post-quantum schemes, has ever made such a claim and in practice such a claim has never been absolutely proven—OTAK-NES seeks to minimize public exposure and repeated use of long-term authorization keys, and to **reduce the scope of a potential security threat from the entire account and all its assets to the specific transaction and the specific amount of assets being transferred—not to all assets of an account.** This is achieved by distributing operational signing authority across one-time keys (Child Keys) through the OTAK-NES architecture.

This approach introduces a fundamental innovation in blockchain security architecture. To the extent that a review of prominent blockchain architectures has been conducted, no other common architecture has limited this threat model from the entire account level to a specific transaction and asset in this manner.

Accordingly, the architecture considers the following threat classes.

### Shift in Threat Model: From All Account Assets to a Specific Transaction and Its Corresponding Asset

In conventional architectures, the exposure of a single operational signature can threaten the entire account and all assets under its control.

OTAK-NES limits the scope of this threat from the entire account to a specific transaction and the asset associated with that transaction—not to all assets of an account. This is enabled through one-time Child Keys derived from Access Keys, based on the OTAK-NES architecture:

- Each operational signature is performed with an independent key.
- The exposure of a single Child Key does not, by itself, provide a direct path to exposing the Access Key or other Child Keys.
- The impact of any potential exposure is, at the architectural level, confined to a single operational key and a specific transaction.
- The exposure of one operational key does not, by itself, compromise the private key with access to the entire account's assets.
- After successful transaction confirmation and transfer of assets to the destination account, the asset is no longer under the control of the source account, and the used Child Key no longer holds active operational authority over that asset or other assets of the source account, and is removed from the operational cycle and deactivated.

### Non-Exposure of Access Key Signatures

The principle of non-exposure of signatures from keys with access to all assets of an account (Access Keys), although currently considered an optional aspect in some blockchain architectures, may become an important security requirement for blockchain architectures over time, as computational sciences advance and highly powerful computers, including quantum computers, emerge.

Beyond the above, there is another fundamental question:

**Why would one refrain from limiting the potential scope of a compromise during transaction submission and signature publication—from the entire account and all assets under its control, to that specific transaction and the specific asset being transferred?**

When the exposure of an account signature can, in the presence of an exploitable weakness in the algorithm, implementation, or other security conditions, become a threat to the private key and consequently to all account assets, we must also recognize that an account signature is not only visible after being recorded on the blockchain; rather, there are multiple points along the path to transaction finality where it can be observed, processed, and exposed.

A transaction, from creation and propagation to finalization and permanent block recording, may be received, relayed, temporarily stored, or processed by various network components and related infrastructure, including:

- Mempool
- Network nodes
- Block producers or proposers
- Validators and other consensus participants
- Transaction propagation and relay infrastructure
- RPC and API providers
-and Intermediate services and infrastructure that receive, forward, or submit transactions for validation.

Therefore, a transaction signature may be visible and subject to analysis at multiple points along the transaction propagation and processing path, before the transaction is finalized and permanently recorded in the blockchain history.

This further highlights the importance of the OTAK-NES architecture, because its objective is not merely to reduce the duration of signature exposure, but to eliminate the use of Access Keys as transaction signing keys and transfer this role to independent, one-time Child Keys.

In this architecture, even if an operational Child Key is exposed during a transaction, that key only authorizes that specific transaction and is permanently deactivated after confirmation, while the Access Key is never used to sign operational transactions and no signature from it is ever published on the network.

Therefore, even with a very strong signature algorithm, the exposure of a transaction and its signature in the public network environment represents an independent layer of exposure that cannot be eliminated solely by increasing the strength of the signature algorithm.

This raises another question:

**Why should blockchain architectures not reduce the threat model as much as possible—from the entire account and all assets under its control—to the smallest operational unit, namely a specific operational key and a specific transaction?**

The OTAK-NES security architecture is introduced precisely on this principle: reducing the potential scope of a compromise at the account level. It is designed for implementation on existing blockchain networks.

This approach can, in the long term, be regarded not only as a security feature, but as an architectural principle and a security standard for blockchains.

Blockchains that have adopted user security and asset protection as one of their fundamental principles can move toward evaluating and, subject to technical validation, implementing the OTAK-NES architecture.

### Long-Term Signature Exposure

In conventional blockchain architectures, long-term Access Keys may repeatedly sign operational transactions over many years; as a result, signatures generated by the same authorization key are permanently recorded in the network history.

This creates an accumulating set of cryptographic data tied to the account's private key, with each transaction providing additional analytical information to attackers.

OTAK-NES counters this growing analytical exposure by ensuring that Access Keys do not directly sign routine operational transactions after successful Bootstrap. In this architecture, operational transactions are authorized through independent, one-time Child Keys.

This limits the scope and magnitude of a potential signature exposure event from an authorization key with access to account assets and all account assets, to a temporary key and the specific transaction associated with it.

### Implementation Vulnerabilities

Security flaws may arise from weaknesses in the mathematical design of a digital signature algorithm, from implementation errors, from reliance on network components for transaction visibility and processing, from unauthorized leakage or copying, or from other factors.

Examples include:

- Incorrect randomness generation
- Nonce generation errors
- Software implementation bugs
- Protocol integration errors
- and Wallet implementation errors.

No signature algorithm has ever claimed to be resistant to all known and unknown threats or to be prepared for all future attack vectors. Although signature algorithms—especially post-quantum signature schemes—are designed according to rigorous security standards and requirements, their security remains dependent on cryptographic assumptions, design, implementation, and both current and future threat landscapes.

OTAK-NES does not attempt to completely eliminate such vulnerabilities.

Instead, by ensuring that operational transactions are authorized through independent, one-time Child Keys, it seeks to limit the potential impact of such vulnerabilities from the entire account and all its assets to the specific transaction and the specific asset being transferred.

## Side-Channel and Physical Attacks

Practical attacks such as:

- Power analysis
- Timing analysis
- Electromagnetic leakage
- Cache attacks
- Fault injection
- and Hardware implementation attacks

may target implementations of digital signature algorithms.

If such an attack compromises a specific Child Key, OTAK-NES is designed to limit its operational impact to the related authorization event and the specific transaction, rather than to the private key of the entire account and all its assets. This is because Access Keys with control over all assets are not used to sign operational transactions in this architecture. Additionally, each Child Key is used only once for a specific transaction and becomes unusable for subsequent operational authorization.

Protection against specific side-channel attacks remains dependent on the security of the wallet implementation and the signature algorithm in use.

### Future Cryptanalytic Advances

Future advances in cryptanalysis may reveal weaknesses that are currently unknown.

These advances may affect:

- Existing classical signature schemes
- Post-quantum signature schemes
- and Future signature algorithms.

OTAK-NES does not depend on predicting which algorithms may become vulnerable in the future.

Instead, by distributing operational signing authority across independent, one-time Child Keys rather than centralizing routine transaction signing on a single long-term operational key, it reduces long-term signature exposure from the entire account to a smaller component, namely a specific transaction with limited associated assets.

### Quantum Computing

Large-scale quantum computers may in the future threaten classical signature cryptography, and continued development and evaluation of post-quantum signature schemes will remain essential to addressing future threats.

OTAK-NES is not a replacement for post-quantum signature algorithms and does not by itself provide post-quantum cryptographic security. At the same time, the architecture can support stronger and more secure signature algorithms against future threats, including quantum threats.

OTAK-NES provides an independent architectural security layer that reduces risk by minimizing exposure to Access Key signatures and by using Child Keys derived from the account's master keys.

Accordingly, OTAK-NES is designed to be usable alongside any signature algorithm, whether classical or post-quantum.

When Child Keys use a suitable post-quantum signature scheme, the two mechanisms can provide complementary protections: the signature scheme provides cryptographic resistance against quantum attacks, while OTAK-NES provides key isolation, separation of authorization authority, and reduction of the threat scope.

### Unknown Future Attack Techniques

No security architecture can assume that all future attack vectors are already known or that it can fully address all future threats.

New attack methods may emerge through advances in:

- Cryptanalysis
- Advances in mathematics and discovery of new analytical algorithms
- Hardware capabilities
- Implementation research
- Protocol analysis
- or Techniques not currently foreseen.

OTAK-NES adopts a defensive architectural approach rather than relying on assumptions about specific attacks.

By minimizing long-term exposure of authorization keys and isolating operational authority through one-time Child Keys, the architecture seeks to reduce the potential consequences of both known and unknown future threats.

### Security Assumptions

OTAK-NES is based on the following assumptions:

- The user's master Seed remains confidential
- Hierarchical key derivation operates correctly
- The digital signature algorithm in use provides its intended level of security
- and Merkle Tree construction and verification are correctly implemented.

OTAK-NES does not replace these mechanisms.

Instead, it introduces an additional architectural security layer that, independently of the signature algorithm in use, reduces long-term exposure and reuse of transaction signing authority.

### Security Objective

The primary objective of OTAK-NES is not to make digital signatures cryptographically stronger.

Its objective is to minimize long-term exposure of high-authority authorization keys and to separate operational authorization authority through one-time Child Keys.

By separating authorization authority from operational signing authority, OTAK-NES seeks to reduce the scope of a potential compromise from a long-term authorization key and the assets under its control, to the level of a specific operational key, a specific transaction, and the specific asset being transferred.

This principle limits the potential consequences of compromising an operational key, while maintaining architectural compatibility with both current and future digital signature schemes.

In essence, this is not merely a change in how keys are managed—it is a fundamental change in how the threat itself is defined, contained, and reduced in scope on the blockchain.
___
___
# Advantages of OTAK-NES in:

- Simplified Migration to New Signature Algorithms
- Simultaneous Support for Multiple Signature Algorithms
- Layer-2 Compatibility

One of the architectural advantages of OTAK-NES is that it separates key lifecycle management from the underlying digital signature algorithm.

OTAK-NES does not define a new signature algorithm. Instead, it defines how Access Keys, Child Keys, Merkle Trees, and transaction authorization are managed. Consequently, the same architecture can operate with different signature algorithms without changing the OTAK-NES security model.

### Simplified Migration to New Signature Algorithms

Blockchain networks inevitably evolve. As new digital signature algorithms become available, networks may wish to migrate from existing schemes to newer ones.

In conventional architectures, migrating to a new signature algorithm often requires replacing the long-term signing authority used for every operational transaction.

Under OTAK-NES, however, each Access Key maintains its own independent Merkle Tree and its own hierarchy of Child Keys.

As a result, a newly introduced Access Key may use a different signature algorithm while existing Access Keys continue operating with their original algorithm until migration is complete.

This enables gradual migration rather than requiring every account to transition simultaneously.

### Simultaneous Support for Multiple Signature Algorithms

Because each Access Key is associated with its own Merkle Tree, different Access Keys within the same account — or across different accounts — may use different digital signature algorithms.

For example, one Access Key may derive Child Keys using ECDSA, while another Access Key derives Child Keys using a post-quantum signature scheme.

During transaction validation, network nodes simply verify the signature according to the algorithm associated with the corresponding Access Key and validate the supplied Merkle Proof against that key's registered Merkle Root.

This architecture naturally supports heterogeneous cryptographic environments without introducing conflicts between different signature schemes.

### Incremental Network Deployment

OTAK-NES also enables phased deployment at the network level.

Rather than requiring immediate migration of every transaction type, a network may progressively expand support for the new signature algorithm based on transaction categories.

For example, support may initially be introduced for a limited category of transactions before being extended to additional transaction types as wallets, infrastructure providers, and network software adopt the architecture.

Similarly, networks may temporarily support multiple signature algorithms simultaneously during migration until older algorithms are eventually retired.

### Layer-2 Compatibility

OTAK-NES is compatible with Layer-2 architectures because its trust model is based on cryptographic authorization rather than a specific execution environment.

Whether transactions are executed directly on Layer-1 or processed through Layer-2 protocols, Child Keys, Merkle Proofs, and Access Key separation remain applicable provided that the corresponding protocol supports transaction authorization using this architectural model.

Consequently, OTAK-NES can serve as a common authorization architecture across Layer-1 and Layer-2 systems while remaining independent of the underlying execution model and signature scheme.

### Architectural Benefit

By separating authorization architecture from the signature algorithm itself, OTAK-NES provides long-term flexibility.

Networks may adopt stronger signature algorithms, support multiple algorithms simultaneously, deploy migrations incrementally, and extend the same authorization model to Layer-2 systems without fundamentally redesigning the lifecycle of account authorization.
___
___
# OTAK-NES Approach for Reducing Signature Exposure in Existing Accounts Prior to Full Deployment

For accounts created before the full deployment of OTAK-NES that already have a history of outgoing transactions, several adoption paths can be defined, each providing a different level of reduction in long-term signature exposure.

A fundamental characteristic of blockchain systems is that signatures published on-chain are permanent and cannot be removed after confirmation. Consequently, any account that has already produced on-chain signatures before the activation of OTAK-NES will permanently retain part of its historical cryptographic footprint.

OTAK-NES enables a significant reduction in the future exposure of such accounts. However, the degree of reduction depends on the adoption path selected.

### Option 1: Create a New Account with a New Seed

In this approach, the user generates a completely new Seed and creates a new account.

After completing the Bootstrap process and activating the OTAK-NES security model, all assets are transferred from the original account to the new account, and operational use of the original account is discontinued.

This provides the highest possible reduction in long-term signature exposure because the new account begins its operational life without any previous on-chain signature history. Consequently, the assets become completely separated from the cryptographic history of the original account.

The only cost of this approach is the need to securely manage and back up a new Seed.

### Option 2: Create a New Account Derived from the Same Seed

In this approach, the user keeps the existing Seed but creates a new account using a different derivation path defined by the TRC-102 hierarchical deterministic derivation standard.

Example:

"m/44'/195'/0'/0/1"

The derived account has an independent public/private key pair while remaining recoverable from the same Seed.

After creating the new account and completing Bootstrap, assets are transferred from the original account to the newly derived account.

This approach avoids generating a new Seed while providing a completely new set of Access Keys and Child Keys for future operations.

If the derived account has no previous on-chain signature history before entering operational use and its Bootstrap process is completed according to the OTAK-NES architecture, its long-term signature exposure will be significantly lower than that of the original account.

### Option 3: Preserve the Existing Account Identity

In this approach, the account address remains unchanged while a new set of Access Keys is assigned to the existing account.

After the account operates under the OTAK-NES architecture, the permission structure can be updated through an "AccountPermissionUpdateContract" transaction signed by a valid Child Key together with its corresponding Merkle Proof.

After successful confirmation:

- The previous Access Keys are replaced with a new set of Access Keys.
- A new Merkle Root corresponding to the updated security structure is stored in the account state.
- All future transactions are validated using the new Access Keys and their corresponding Child Keys.

This approach is particularly suitable for accounts whose address, operational history, public identity, or external integrations must be preserved.

However, it should be noted that:

- Previously published signatures remain permanently recorded on the blockchain.
- Replacing Access Keys does not erase the account's historical cryptographic records.
- This approach reduces future signature exposure but does not make the account equivalent to a newly created account with no prior on-chain signature history.

### Summary

All three approaches are fully compatible with the OTAK-NES architecture, although they provide different levels of reduction in long-term signature exposure.

- Option 1 provides the greatest reduction because assets are moved to an account with no previous signature history.
- Option 2 also provides substantial reduction while preserving the existing Seed.
- Option 3 offers the greatest operational compatibility by preserving the existing account identity. However, because historical signatures remain permanently recorded on-chain, its reduction in long-term signature exposure is inherently lower than that achieved by the first two approaches.
___
___

