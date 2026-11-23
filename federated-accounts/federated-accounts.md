# Published Presentations on Federated Accounts

*A use-case document for wallet-anchored pseudonyms.*

Nathan Schepers — 2026-11-23

## 1. Use case

A room in a federated messaging system admits adults only. The accounts that ask to join are
hosted on many homeservers, and each homeserver has its own operator. Matrix is the example.

On a centralised platform, the platform checks a user's age once and keeps the result. Every
part of the service relies on that record. A federation has no such party. A homeserver that
states "this account belongs to an adult" gives its own word, and the room's server takes it
on trust.

The EUDI wallet allows another arrangement. The holder presents a proof of age once, and the
proof is published on the account. The room's server, and any other reader, verifies the
published proof against the trusted lists. The check is a computation over the published
record.

The published record states the account name, a pseudonym and the statement "over 18". The
pseudonym is a value that only the holder's wallet can produce for that account. It shows that
later records on the same account come from the same holder.

This document is an input to discussions on pseudonyms and on issuance. It asks for the two
things in section 7, which the Architecture and Reference Framework (ARF) [3] and the
`nym_auth` rulebook draft [5] lack for this use case. Section 8 discusses consent, the
retention rule of the age-verification specification [4], validity, the issuer in a record and
current implementations.

## 2. Publication

- **Each server checks for itself.** The record is evidence that every server can verify. The
  homeserver's statement about the account's age is replaced by a proof.
- **One presentation serves every room.** With a published record, a room's server reads and
  verifies. The holder presents in a browser session with one homeserver. That homeserver sees
  the holder's browser and network address, and no other server does. The alternative is one
  verification session per gated room.

Publication is the holder's choice for each account.

## 3. Assumptions

### Assumption 1: Pseudonym attestation

One of the holder's attestations is the source of the holder's pseudonyms. It includes a
commitment to a seed that the holder's wallet chose. The wallet is the only party that has
the seed. [1] sketches this. The `nym_auth` attestation has it as `private_seed` and its
commitment `psc`, the private seed commitment [5].

### Assumption 2: Pseudonym

The wallet derives a pseudonym from the seed, the homeserver's name and the account's local
name. The same pseudonym attestation and the same account give the same pseudonym. The
pseudonyms of two accounts are unlinkable without the wallet's seed. A seed of the issuer can
enter the derivation beside the wallet's seed. The issuer lacks the wallet's seed and cannot
compute the pseudonym.

In [1] the pseudonym is a function of the seed, a scope and an index. In [5] the two inputs
beside the seed are `verifier_id` and `verifier_context`. The homeserver's name is the scope
and the `verifier_id`. The account's local name is the index and the `verifier_context`. The
ARF has the same three inputs (AS-AP-53-001, formerly ZKP_04) and requires "multiple different
Pseudonyms at a given Relying Party" (AS-WP-11-004, formerly PA_04) [3].

### Assumption 3: Association

The wallet proves that the pseudonym attestation and a second attestation are bound to the
same wallet. [2] has this as its Assumption 4. The ARF calls the mechanism a cryptographic
binding of attestations [3]. The assumption names the wallet because the ARF states that the
binding "does not, by itself, guarantee that two attestations belong to the same User" [3].

### Assumption 4: Proof

The wallet proves in zero knowledge, in one proof or in several that are tied together, that:

- a trusted issuer signed each attestation, and each is valid at the stated time;
- the wallet signed this presentation session with the device key of each attestation;
- the requested statement holds for the second attestation's attributes;
- the pseudonym is derived from the seed that the pseudonym attestation commits to, the
  homeserver's name and the account's local name;
- the two attestations are bound to the same wallet.

The presentation reveals the statement, the pseudonym, the homeserver's name, the account's
local name, the issuers' certificates and the time. It is bound to the values of the
presentation session. Over the Digital Credentials API these are the page's origin, a nonce and
the thumbprint of the homeserver's encryption key [5].

### Assumption 5: Public verification

The presentation is verified from public values: the issuers' certificates, the time, the
values of the presentation session, the homeserver's name, the account's local name, the
statement and the pseudonym. Every party that verifies it with the same trusted lists reaches
the same result, on the day of the presentation and later.

## 4. Walk-through

The account is `@user1:hs.example`. The room is on a second homeserver, `hs2.example`. The
age attestation is the Proof of Age attestation of the age-verification specification [4],
which states `age_over_18` as an attribute. The pseudonym attestation is the `nym_auth`
attestation of [5].

### Step 1: Account

`@user1:hs.example` is created on `hs.example` as any account is.

*Assumptions: none.*

### Step 2: Age disclosure

The holder is logged in to the account and asks `hs.example` to add a proof of age to it.
`hs.example` requests from the wallet the statement "over 18" from an age attestation and the
pseudonym for `@user1:hs.example`. The request states that the result is published, and the
wallet shows this to the holder. The holder approves.

The wallet returns the statement, the pseudonym and the proof, encrypted to `hs.example`.
`hs.example` decrypts the response, verifies the proof, records the pseudonym as the account's
pseudonym, and publishes the record on the account. The record is the decrypted presentation
with the session values needed to verify it. Its date is the time in the proof.

*Assumptions: 1, 2, 3, 4.*

### Step 3: Verification by a reader

`@user1:hs.example` asks to join the room. `hs2.example` fetches the account's record and
makes these checks:

- The issuers' certificates are on the trusted lists.
- The proof verifies for the statement "over 18", the pseudonym, the homeserver's name
  `hs.example`, the account's local name `user1` and the session values in the record.
- The record's date is within the period the room accepts.

A record copied to another account fails the second check, because the pseudonym and the
proof are for `@user1:hs.example`.

`hs2.example` admits the account. Any other party with the record and the trusted lists
makes the same checks and reaches the same result.

*Assumptions: 4, 5.*

### Step 4: Refresh

A Proof of Age attestation is recommended to be valid for at most three months [4]. The
wallet obtains a new one, and the holder presents again as in step 2. The new proof gives the
same pseudonym, because the pseudonym comes from the pseudonym attestation and the same
account.

`hs.example` accepts a new record only if its pseudonym equals the account's pseudonym, and
publishes it beside the first. A presentation by another holder gives a different pseudonym
and is refused. A reader of the two records knows that one wallet is behind both.

*Assumptions: 1, 2, 3, 4.*

## 5. Properties

- **A reader learns the statement and the issuers.** The holder of the account's pseudonym had
  a valid Proof of Age attestation on the record's date. The record names the issuers of the
  two attestations.
- **Pseudonyms of one holder are unlinkable.** Two accounts have unrelated pseudonyms. The
  issuer has the commitment and a reader has the pseudonyms. Relating one to the other requires
  the wallet's seed. The issuers named in a record and the timing of records are outside this
  property.
- **The issuer's part ends at issuance.** Presentation and verification use the attestations,
  the trusted lists and the circuit.
- **The holder decides what is published.** The choice is made for each account and for each
  statement.
- **Any party can hold, serve and verify the records.** A copy verifies as the original does.
- **A record is permanent.** A published record verifies for as long as a copy and the
  circuit exist and its issuers are on the trusted lists.
- **The records of an account come from one wallet.** The first proof for an account fixes its
  pseudonym. Every later record needs the same pseudonym attestation and the same seed.

The costs:

- **Sybil resistance is given up.** One pseudonym attestation supports any number of accounts,
  so a count of verified accounts is not a count of people. Handling abuse stays with server
  operators, as it is today. Where a deployment wants a limit, [1] bounds the number of
  pseudonyms per scope.
- **A new issuer can mean a new pseudonym.** In [5] a new issuer "means a new `psc` and a new
  pseudonym". The account is then established again.
- **Withdrawal leaves copies.** Withdrawal removes a record from the account. Copies made
  before the withdrawal still verify. [2] discusses this for the directory.
- **Timing can link.** An issuer that logs when it issued and a reader that sees when a record
  appeared can compare the two. The specification makes the related point that timestamps
  inside an attestation "can provide linkability clues" [4].

## 6. Other statements

The walk-through publishes one statement. The same steps publish others.

- **Phone number.** The statement is a phone number disclosed from a carrier attestation. An
  index of such records by number is the directory of [2].
- **Other attributes.** Residency, a professional licence and a membership are examples. Each
  needs the association of assumption 3. A statement that a named attribute has a given value
  is within the deployed Longfellow circuit for mdocs [7], and other statements need a circuit
  for them. Records from two attestations under one pseudonym show a reader that one holder has
  both.
- **Statements shown to readers.** A statement on a profile is read on any server at any time.
  The holder cannot present to each reader. A published record gives every reader a proof.

A holder with two accounts can publish different statements on each. The accounts stay
unlinkable as far as the statements themselves allow.

## 7. Gaps

The following is missing for the walk-through above.

### Pseudonym for an account

- `account-recovery-ssc`, a pseudonym configuration in [5], gives a pseudonym that "is distinct
  per `verifier_id` and `verifier_context`", for "the account that `verifier_context` names".
  It uses the issuer's `secret_seed`, and the issuer "can map the pseudonym to an identity".
- Needed: the same configuration with the wallet's seed in the derivation.

### Association of two attestations

- The `nym_auth` circuits prove a pseudonym and an age range over the `nym_auth` attestation
  [5]. The Proof of Age attestation is a separate attestation with the attribute
  `age_over_18` [4].
- In [5] "The Wallet Unit samples a `private_seed` for each Credential Issuer". Attestations
  from two issuers then commit to different seeds.
- The pull request of [5] includes a draft, Credential Auxiliary Data, in which "The Wallet
  computes a commitment. The Credential Issuer signs it into the Credential." The draft
  defines this for any credential configuration.
- The implementing regulation has a wallet provide a pseudonym "either standalone or in
  combination with any person identification data or electronic attribute attestation"
  (Article 14(2)) [6]. The specification it names for pseudonyms is WebAuthn (Annex V). The
  ARF's discussion paper on pseudonyms notes for WebAuthn credentials "No cryptographic
  binding to attributes in the wallet" [3].
- The ARF requires that a relying party "SHALL be able to verify that the same User performed
  both actions" when a pseudonym is registered together with an attestation (AS-WP-11-013,
  formerly PA_12). Its discussion paper on pseudonyms states that "there is no mechanism
  specified for providing cryptographic binding between a pseudonym and (presented) attributes"
  [3].
- [1] recommends "every issued credential to include a fresh commitment with a new random
  nonce". With one seed committed in both attestations, one proof covers both.
- [2] assumes that the pseudonym, the credential it is derived from and the disclosed
  attribute are "bound as one wallet's act of presentation".
- The ARF's discussion paper on combined presentation describes a "Cryptographic Binding of
  attestations", a proof "that two or more public keys, each embedded in a separate
  device-bound attestation, are managed by the same WSCD" (ACP_11). It "MAY be implemented
  using a Zero-Knowledge Proof mechanism" (ACP_12) [3].
- The deployed Longfellow circuit for mdocs proves that named attributes have given values
  [7]. [8] is a design for one circuit whose statements come from a published list of queries.
- Needed: a way to show that a pseudonym and a statement from another attestation come from
  one wallet.

## 8. Discussion

### Consent

- In step 2 the holder approves publication. The wallet's response is encrypted to the
  homeserver. The homeserver decrypts it and publishes the presentation.
- Registration of a relying party includes "the intended use" (Article 5b(2)(c)) [6].
- The consent text for a presentation that is published is an open question.

### Retention

- The age-verification specification requires, for its Proof of Age attestation, that
  "Verifiers must not retain proof bytes after the verification decision" [4]. A published
  record is a retained proof, and no text exempts a presentation that the holder asks to have
  published. This document reads the rule, in the light of the ARF as a whole, as not applying
  to a zero-knowledge presentation that the holder asks to have published. The next three
  bullets give the reasons.
- The reason the specification gives is an issuer that kept copies or hashes of the proofs it
  issued and obtains the proofs a verifier received. Such an issuer "could match them in the
  plain-mDoc flow". Of zero-knowledge presentations the same entry states that "there is
  nothing on the verifier side to match against issuance records" [4].
- The ARF requires a relying party to "discard the values of all unique elements" of an
  attestation and to pass them to no other party (AS-RP-01-002, formerly OIA_16) [3]. The
  elements it names are the salts, the attribute hashes, the attestation identifier, the device
  public key and the issuer's signature value. A zero-knowledge presentation from the circuits
  of [5] and [7] reveals none of them. The specification states that "a ZKP reveals no reusable
  byte pattern" [4].
- The regulation forbids intermediaries to "store data about the content of the transaction"
  (Article 5b(10)) [6]. In the walk-through the homeserver is the relying party and requests
  the presentation for its own account.
- A leaked presentation and a published record have the same values: the statement, the
  pseudonym, the issuers' certificates and the time. See "Issuer in the record".

### Validity

- The specification recommends a validity of at most three months and states that
  "Attestation revocation is not required for age verification purposes" [4]. The circuits
  check validity dates and no revocation.
- The validity period does the work of revocation. After a compromise of an issuer's signing
  key, "forged proofs could circulate for up to 3 months in deployments without a status
  list", and "a compromised issuer can be removed from the Trusted List, after which verifiers
  reject its signatures" [4].
- A proof shows that the attestation was valid at the record's time. A reader checks the
  issuers against the trusted lists as they stand on the day it reads, and chooses the age of
  record it accepts (step 3).
- For the directory, [2] has each reader check revocation data fetched in bulk, and a record
  "validates against external trust roots until the underlying credential expires or is
  revoked". For an attestation with a short validity and no revocation, the validity period
  takes that place.

### Issuer in the record

- The deployed circuit takes the issuer's public key as a public value, so a record names its
  issuer [7]. A record made from two attestations names both issuers.
- The specification states of the plain presentation that the issuer's identity is visible,
  "typically revealing the user's Member State" [4]. A verifier has the issuer today. A
  published record shows it to every reader.
- Hiding the issuer within a set is under discussion. [5] allows a system to "hide which key
  of the set signed". The ARF's discussion paper on zero-knowledge proofs describes a proof
  that an attestation was issued by a provider on a trusted list "without disclosing any
  additional information about the specific PID Provider or Attestation Provider" [3]. The
  specification names "possible future multi-issuer trust anchors" [4].

### Requests for an account on another homeserver

- In [1] a scope belongs to the party that requests it: "one RP cannot request an scp produced
  by another RP". In the walk-through the account's own homeserver makes the request. A room's
  server that takes a presentation for an account on another homeserver is outside that rule.

### Wallets and issuers

- An issue on the rulebook's repository reports the commitment from the wallet to the issuer
  implemented "in the Kotlin wallet SDK, the wallet backend engine and go-wmp", and an issuer
  that "adds the per-subject `psc` record" [9].
- The seed has to move to a new device with the holder. [1] discusses backup and transfer.

## 9. References

1. René Mayrhofer, Anja Lehmann, abhi shelat, "A Brief Note on Cryptographic Pseudonyms for
   Anonymous Credentials", arXiv 2510.05419v1, 6 October 2025.
   https://arxiv.org/pdf/2510.05419v1
2. Nathan Schepers, "A Privacy-Preserving Phone Book for Federated Messaging", 2026-05-21.
   [phone-book/phone-book.md](../phone-book/phone-book.md)
3. Architecture and Reference Framework, read at `f5789a7` (2026-10-02): Annex 2, high-level
   requirements AS-AP-53-001, AS-RP-01-002, AS-WP-11-004 and AS-WP-11-013; discussion paper E
   on pseudonyms (re-review); discussion paper G on zero-knowledge proofs; discussion paper K
   on combined presentation of attestations.
   https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework
4. Age Verification technical specification, read at `8b97287` (2026-09-02): sections 3.4.1 to
   3.4.3 and `AV-threat-model.md`.
   https://github.com/eu-digital-identity-wallet/av-doc-technical-specification
5. `nym_auth` rulebook, version 0.7, pull request 12 at `8fafd9c` (2026-11-05).
   https://github.com/sirosfoundation/nym_auth_rulebook/pull/12
6. Regulation (EU) No 910/2014, consolidated version of 2024-10-18, Article 5b. Commission
   Implementing Regulation (EU) 2024/2979, Article 14 and Annex V.
7. longfellow-zk, read at `1424f50` (2026-11-19): `lib/circuits/mdoc/mdoc_zk.h`.
   https://github.com/google/longfellow-zk
8. Nathan Schepers, predicate-circuit. https://github.com/pipe23-org/predicate-circuit
9. `nym_auth` rulebook, issue 13, "Credential Auxiliary Data: three points from an
   implementation", read 2026-11-23.
   https://github.com/sirosfoundation/nym_auth_rulebook/issues/13
