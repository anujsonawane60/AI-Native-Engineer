---
id: m19-03-contracts-tokens
module: m19-frontier-electives
slug: smart-contracts-and-tokens
title: "Smart Contracts, Tokens, Wallets and Gas"
order: 3
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Describe a smart contract as public code with public state and metered execution
- Explain why every user action on a chain has a rupee cost attached
- State what a wallet actually holds, and why there is no password reset
- Name India's tax and legal position on virtual digital assets as of 2026
- Spot an AI design that hides an oracle, an indexer or a gas bill

## Lesson

### The pain

A team decides the canteen loyalty points should be a token. It sounds modern, the pitch deck looks sharper, and a mentor nods. Four weeks later they have numbers.

Every single point award is a transaction, and every transaction costs a fee paid in the chain's native currency. At a busy lunch that is 400 fee-paying writes an hour, each one costing somewhere between a fraction of a rupee and several rupees depending on how congested the network happens to be that afternoon — a number they do not control and cannot predict. Every student's full purchase history is now publicly readable by anyone with the wallet address, including the competitor across the road. Two students have lost their seed phrases and their points are gone with no recourse. One student discovered that the moment he converts points to anything of value, a 30% tax on gains and a 1% deduction at source enter his life. And a bug in the award logic, deployed to a live network, cannot be edited.

Meanwhile the canteen already accepts UPI, which costs nothing per transaction, settles in two seconds, and has a dispute process with a human at the end of it.

### The idea

**A smart contract is a program deployed to a ledger: its code is public, its data is public, it runs only when someone pays for each step, and once deployed you cannot change it.**

Those four properties are the whole chapter. Public code means anyone can read your business logic. Public state means anyone can read your balances. Metered execution means the chain charges per unit of computation — that charge is called gas — and the user pays it, in real money, win or lose. And immutability means the thing you deployed is the thing you have forever, unless you deliberately built in an admin who can swap it, which quietly puts a trusted referee back in the design you adopted to avoid trusted referees.

A token is nothing more exotic than a row in one such contract's state: a mapping of address to amount, with rules about who may change it.

### The analogy

The canteen installs a vending machine inside a sealed glass case. The rules are printed on the inside of the glass where everyone can read them, and so is the running total of every coin it has taken. There is no keyhole. You press a button and a coin is consumed by the mechanism whether or not the snack drops.

That is a smart contract. Readable by all, operated by all, changeable by none, and charging per press.

**Where the analogy breaks** in two important ways. First, a real vending machine can be unplugged, unlocked and refilled by the company that installed it — a deployed contract genuinely cannot be amended, and the only escape hatch is to deploy a new one and persuade everybody to use that instead. Second, a vending machine knows its own stock. A contract knows nothing about the outside world: it cannot read today's milk price, a fridge temperature or whether a thali was actually served. Anything from outside must be fed in by a trusted reporter called an oracle, and that oracle is a trusted referee with a different job title.

### Picture it

```
  Wallet                 Network                  Contract
┌────────────┐  signed  ┌───────────┐  included  ┌──────────────┐
│ private key│─────tx──▶│  mempool  │───in block▶│ code (public)│
│ + address  │          │ fee market│            │ state(public)│
└────────────┘          └───────────┘            └──────────────┘
      ▲                                                 │
      └──────── gas spent, success or failure ◀──────────┘
```

A wallet signs a transaction, the network charges gas to execute it against the contract's public code and public state, and the fee is spent whether the call succeeds or reverts.

```
  on-chain                     off-chain (still needed)
┌──────────────┐          ┌─────────────────────────────┐
│ contract code│◀──price──│ oracle: reports outside facts│
│ balances     │          ├─────────────────────────────┤
│ event log    │──events─▶│ indexer: makes it queryable  │
└──────────────┘          │ frontend, support, KYC       │
                          └─────────────────────────────┘
```

Any real product built on a contract still needs an oracle to bring outside facts in and an indexer to make on-chain history searchable, so "no backend required" is never true.

### Where it lives

Map it onto what you already have. The contract is your business logic and your database schema from M04, except both are world-readable and the schema can never be migrated. The wallet is authentication from M04, except the credential is unrecoverable and there is no password reset, no email OTP and no support team. The RPC endpoint you talk to is ordinary HTTP over TLS from M02, usually run by a company, which means your "decentralised" app typically has a single commercial dependency at the exact point users touch it. Gas is M17's unit economics with a new name, and unlike a cloud bill it is set by an open market you do not control.

The Indian regulatory position matters and you should be able to state it. Virtual digital assets are legal to hold and trade but are not legal tender, and no Indian regulator will help you if a counterparty vanishes. Gains are taxed at a flat 30% with no deduction of expenses other than cost of acquisition, losses cannot be set off against anything, and a 1% tax is deducted at source on transfers above the threshold. Exchanges operating in India are reporting entities under anti-money-laundering rules. Marketing a token to Indian retail users is therefore a compliance project, not a feature, and this is the chapter where a founder should notice that before the pitch and not after.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Smart contract | A program whose code and state live on a ledger | A token contract |
| Gas | The per-step charge for executing a transaction | Paid in the chain's own coin |
| Gas limit | The most computation you authorise for one call | Exceed it and the call reverts, fee kept |
| Wallet | Software holding a private key and an address | A browser extension or phone app |
| Seed phrase | Words that regenerate the private key | Twelve or twenty-four English words |
| Address | The public identifier derived from the key | Where tokens are credited |
| Token | A balance tracked inside a contract | Loyalty points, a stablecoin |
| Fungible / non-fungible | Interchangeable units, versus unique items | Points, versus a numbered certificate |
| Testnet | A free parallel network for development | Where your experiments belong |
| Faucet | A tap that gives free testnet coins | No real money involved |
| Oracle | A trusted reporter of outside facts | Today's milk price |
| Indexer | A service that makes on-chain history queryable | How a frontend lists past awards |
| Approval | Permission for a contract to move your tokens | The most-abused feature in the space |

### Tradeoffs and gotchas

**Everything is public, forever.** Balances, timings, counterparties and your logic. Competitors read it, and so does anyone profiling a user. Personal data on a chain conflicts directly with the DPDP Act's expectation that you can erase it.

**A failed transaction still costs money.** Users who are not already crypto users find this unforgivable, and they are right to.

**No password reset, ever.** A lost seed phrase is a lost account. Custodial wallets fix this by holding the key for the user, which reintroduces exactly the trusted intermediary the architecture was chosen to avoid.

**Approvals are the main way people lose funds.** Granting a contract permission to move your tokens is a standing authorisation that most users never revoke, and a malicious or compromised contract drains the lot. This is M07's excessive-agency problem with money attached.

**Upgradeability is a trust decision in disguise.** The patterns exist, they work, and they all mean somebody holds a key that can replace the code under your users. That is fine — but then say honestly that your system has an administrator.

**Gas prices are a market.** A design whose unit economics work at a quiet hour can be unusable during congestion, and you have no lever.

### Verify it

**Red flag: no gas cost per user action.** Ask for a table: every action a user takes, how many transactions it needs, and the cost in rupees at a low and a high gas price. If the AI will not produce numbers, it has not designed anything.

**Red flag: "no backend needed".** Ask where the frontend gets a user's history, where outside facts come from, and who runs the RPC endpoint. Every honest answer names an indexer, an oracle and a provider.

**Red flag: personal data written on chain.** Names, phone numbers, Aadhaar numbers, even an email hash that can be checked against a guess. Ask what happens when a user exercises a deletion right. The answer is that you cannot comply.

**Red flag: silence on the irreversible cases.** A correct design document says what happens when a user sends to the wrong address, loses a seed phrase, or when a bug is found after deployment. If those three are absent, the design has not met reality.

**Red flag: no mention of Indian tax or legal status** in a product aimed at Indian users. 30% on gains, 1% deducted at source, losses not set off, not legal tender. An AI that proposes a consumer token for an Indian campus without raising this is giving you an American answer.

## Walkthrough
1. Install a free browser wallet in a throwaway browser profile. Write the seed phrase on paper. Note that the app refuses to show it again casually, and write down why.
2. Switch the wallet to a public testnet. Confirm that the balance is zero and that nothing here involves real money.
3. Use a free faucet to receive testnet coins. Note how long the transfer took to appear.
4. Open a public block explorer for the same testnet and paste your address. Observe that your entire history is visible to anyone who has the address.
5. Send a small testnet amount to a classmate's address. Before confirming, screenshot the fee estimate and note both the gas price and the total.
6. Find your transaction on the explorer. Record the gas used, the fee paid, and the block number it landed in.
7. Open the explorer page for any well-known token contract on that testnet. Read its public code tab and its public state. Write down one thing about the logic you could learn as a competitor.
8. Deliberately send a transaction with the gas limit set far too low. Record what happens to the transaction and to the fee.
9. Find the approvals or permissions screen in your wallet. List what a granted approval would allow a contract to do without asking you again.
10. Convert your recorded fee into rupees at today's rate, multiply by 400 lunchtime awards, and write the daily cost next to the cost of 400 UPI transfers. Keep both numbers for chapter 4.

## Talk to AI
```prompt
CONTEXT:      I am a student founder in India. I am being told to put my
              product's loyalty points on a public blockchain as a token. I
              do not write code; I need to judge the proposal. My users are
              college students on budget Android phones who already use UPI.

GOAL:         A costed, honest assessment of what a token would mean for
              this product, including what it forces on my users.

REQUIREMENTS: 1. A table of every user action - earn, check balance, spend,
                 transfer - showing how many on-chain transactions it needs
                 and the rupee cost at a low and a high gas price, with the
                 gas price you assumed stated explicitly
              2. A list of everything about my users that becomes publicly
                 readable, and who could use it
              3. The off-chain services I would still need: oracle, indexer,
                 RPC provider, support, and what each one costs me in trust
              4. What happens in four failure cases: wrong address, lost
                 seed phrase, failed transaction, bug found after deployment
              5. India's current tax and legal position on virtual digital
                 assets for both me and my users, with the caveat that rules
                 change and I must check the current position

CONSTRAINTS:  No code. Plain English. Assume I cannot ask users to buy
              anything or to understand gas. Compare every cost against
              UPI, which my users already have.

NON-GOALS:    Do not recommend a specific chain. Do not discuss token price,
              fundraising or investment. Do not write a contract.

ACCEPTANCE:   - Every cost row has a number and a stated assumption
              - You name at least one off-chain service I cannot avoid
              - You state plainly whether a plain database would serve this
                product better, and why
              - You flag that tax and regulatory details must be verified
                against the current rules

ASK:          Give me the cost table first, then the privacy list, then your
              verdict in three sentences. Do not soften the verdict.
```

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, what it means that a program's code and all of its data are visible to everybody and can never be changed after it is switched on. Use one everyday object as the comparison, keep it under 200 words, and end with the one consequence I should remember.
```

```prompt
I do not understand gas. Walk me through one transaction step by step and tell me who pays, when they pay, what happens to the payment if the transaction fails, and who decides the price. Then ask me what happens if I set the gas limit too low, and wait for my answer before correcting me.
```

```prompt
Give me five short product ideas aimed at Indian college students where someone has proposed a token. For each one, ask me to name who the token is for and what it does that a database column cannot. Do not give me your own opinion until I have answered all five.
```

## Quiz

### Q1 · L2
What does gas pay for?
- (a) Storing the contract's source code permanently
- (b) Each step of computation and storage a transaction performs ✅
- (c) The electricity used by the user's phone
- (d) The right to read the contract's public state
> Gas meters execution. Reading public state costs nothing, and the fee is charged per step of work the network performs on your behalf.

### Q2 · L3
A transaction runs out of gas and reverts. What happens?
- (a) The fee is refunded because nothing changed
- (b) The state change is undone but the fee is kept ✅
- (c) The transaction is retried automatically at a higher price
- (d) The contract is marked as faulty
> The network did the work up to the point of failure and charges for it. A user pays for a failure, which is one of the hardest parts of the model to explain to ordinary users.

### Q3 · L4
An AI proposes a loyalty-token app and says no backend is required. What is the strongest objection?
- (a) Contracts cannot store balances
- (b) A frontend needs an indexer for history, an oracle for outside facts, and someone's RPC endpoint ✅
- (c) Browsers cannot talk to a blockchain
- (d) Tokens require a database to mint
> The contract holds state but cannot be queried flexibly, cannot learn outside facts, and is reached through a provider's server. Every real product carries all three dependencies.

### Q4 · L4
Why is putting a user's phone number into contract state a serious design error for an Indian product?
- (a) Contract storage is too expensive for text
- (b) It is publicly readable and can never be deleted, which conflicts with data-deletion obligations ✅
- (c) Phone numbers cannot be stored as contract data types
- (d) It would make the contract non-upgradeable
> On-chain data is public and permanent by design. Under the DPDP Act you are expected to be able to erase personal data on request, and here you simply cannot.

### Q5 · L5
Your team adds an admin key so the contract can be upgraded if a bug is found. What has changed about the system's trust model?
- (a) Nothing, because upgrades are a standard pattern
- (b) The system now has a trusted administrator who can replace the rules under its users ✅
- (c) The contract is no longer public
- (d) Gas costs are now paid by the admin
> Upgradeability is a legitimate choice, but it reinstates exactly the trusted party the architecture was adopted to remove. Say so out loud in your decision log rather than claiming trustlessness.

## Go deeper
- [Ethereum docs: Gas and fees](https://ethereum.org/en/developers/docs/gas/)
- [Ethereum docs: Networks, testnets and faucets](https://ethereum.org/en/developers/docs/networks/)
