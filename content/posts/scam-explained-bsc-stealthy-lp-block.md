+++
title = "BSC-SCAM: Stealthy Liquidity Pool Block on PancakeSwap"
date = "2021-04-11T15:21:09+02:00"
author = "cryptot3ddybear"
authorTwitter = "cryptot3ddybear" #do not include @
cover = ""
tags = ["scam", "bsc", "pancakeswap", "solidity", "security", "blockchainsec"]
keywords = ["bsc", "scam", "lp", "security"]
description = ""
showFullContent = false
+++

A guide for crypto traders and a technical analysis of a popular 
Binance Smart Chain smart contract scam resulting in an inability to swap 
a token back due to `transferFrom` error on PancakeSwap.



Yesterday I received a message from TokenSniffer regarding another token
*SafeBinance* with swapping issues and a clue that there is a weird
instruction restricting somehow a `to` address in a `transfer` function.

As it turns out the token might be a part of a bigger scam campaign and
cool TokenSniffer\'s feature listed multiple similar tokens deployed
during the last few days. Here are a few examples:

-   SafeRocket
-   SafeCoinToken
-   FairSafeApe
-   SAFEMOUSE
-   SAFEROCKET
-   and many more (you can find TokenSniffer\'s similar token contracts
    feature, similarity score 0.9362 seems to match them quite well).

The initial \#SCAMToken target was \#SafeBinance, but when I checked the
message and similar tokens it turned out there was a newer one
(\#SafeRocker) released a few mins earlier and looked like it is still
active, so I decided to start with that one.

For quick info how to recognize this exact scam for non-tech people, go
to for traders section.

# Targets

-   initial target: \#SafeBinance
    -   <https://bscscan.com/token/0x612a8f4d3eacb6526d7a5e2c7c8c15c1c4dc5319>
    -   <https://tokensniffer.com/token/0x612a8f4d3eacb6526d7a5e2c7c8c15c1c4dc5319>
-   on-going scam similar to SafeBinance and analysed in this write-up:
    SafeRocket
    -   <https://bscscan.com/token/0x331bb194dc931c79351234733d69dc0f6b8b482a>

# TLDR;

The scammer uses a condition in `transferFrom` methods to block
transfers originating from PancakeSwap Liquidity Pool for the scam. The
address of the PancakeSwap LP is set during the very first interaction
of the pool with the contract via add liquidity at the beginning, so the
variable `newun` is set exactly to the address of a Liquidity Pool for
that token. When the variable is set swaps back are not working as
contract blocks transfers from Liquidity pool to the users, wht results
in `transferFrom` error message in PancakeSwap. The value of the coin
increases and due to the AMM mechanics the scammer to cash out pulls
back his LP token with a profit.

# For Traders

-   the SCAMToken has a total supply of 1,000,000,000,000,000 tokens and
    18 decimals and all of the similar ones have exactly the same total
    supply, but it\'s quite a weak indicator of this scam and can be
    easily modified by the scammers
-   nearly all transactions are from
    `PancakeSwap: <SCAM TOKEN Liquidity pool>` to the `victim address`
    (swapping something to SCAMToken) and there are very few where the
    recipient is different indicating that people are buying tokens only
    and nobody is actually swapping it back (also typical for fresh
    tokens where people tend to hold at the beginning)
-   **IMPORTANT** go to BSCScan and the contract of the token, for
    instance in this SCAMEToken
    `https://bscscan.com/address/0x331bb194dc931c79351234733d69dc0f6b8b482a#readContract`
    and look for `newun` variable or any other variable (if scammer
    renames it) which is set to the address of the PancakeSwap Liquidity
    Pool of that SCAMToken
-   In the transactions log it looks like:
    -   the very first (oldest) transaction is as usual generate coins
        from the `0x0` address to the contract\'s owner wallet
    -   the next one (second oldest) is a regular transfer of a big part
        of the supply to a dead address (burn)
    -   then the scammer adds liquidity to PancakeSwap and sets the
        `newun` variable
    -   from this moment users are able to swap to SCAMToken, but they
        are not able to swap-back, due to a blocked transfers from
        PancakeSwap Liquidity Pool
-   to cash out the scammer pulls the liquidity with a profit

# Technical Analysis

## Introduction

To understand the scam a brief high-level knoweldge of a swap
transaction performed by DEX like PancakeSwap is necessary. I\'m neither
solidity nor a blockchain developer, thus it\'s a pure analysis based on
a notepad, bscscan.com and google. I read my own swapping transactions
and the code of the contract and came up with a hypothesis how it works.
Naturally, solidity developers are more than welcome to point out
anything that is wrong in the proces I described. Just drop me a
message.

## PancakeSwap swap transactions

### Swap BNB-\>SomeToken

Basically when somebody wants to buy a token on PancakeSwap or a similar
decentralized exchange he or she needs to swap usually some BNB (in case
of the Binance Smart Chain) for the final token he or she wants.
Swapping different tokens than the main network\'s token are also
possible, but it complicates the process, so BNB-\>SomeToken is a basic
and the easiest example and will be the base for the analysis.

After a swap a new transaction should be present in the wallet and it
can be reviewed in BscScan. The transaction\'s details consists of this
the most crucial fields:

    From: <buyer wallet address>
    To  : <contract address of PancakeRouter>
    Tokens Transferred:
        From PancakeSwap Router 
               to PancakeSwap: SomeToken Liquidity Pool (BNB buyer pays)
        From PancakeSwap Some Token Liquidity Pool 
               to <buyer wallet address> (some number of SomeToken)

However, in details after a quick googling in the Logs tab in BscScan
there are more specific actions which are titled
`Transaction Reciept Event Logs` and in short it lists the following 5
events:

    1. Deposit BNB from buyer's wallet to PancakeSwap Router
    2. Transfer deposited BNB from PancakeSwap Router to PancakeSwap Liquidity Pool for SomeToken
    3. (IMPORTANT) Transfer form PancakeSwap Liquidity Pool for SomeToken to buyer's wallet
    4. PancakeSwap Liquidity Pool sync.
    5. PancakeSwap Liquidity Pool swap

Events 1 and 2 involve BNB contract (WBNB to be more precise in my
cases). Actions 3 involves SomeToken\'s contract (invoking
`transferFrom` method from SomeToken\'s contract and generating Transfer
event). Actions 4 and 5 are involving
`PancakeSwap Liquidity Pool for SomeToken` contract.

### Swap SomeToken-\>BNB

The swap back process looks very similar, but it first requires an
Approval action, which takes only network\'s fee and allows to swap
SomeToken to other token. After confirming that it will be visible in
wallet\'s transactions list, however details or approval are irrelevant
for this analysis. Then the actual swap can be done and when it is
performed it will be also listed in wallet\'s transactions list as
following (the most crucial fields):

```
    From: <buyer's wallet address>
    Interacted With (To): <contract address of PancakeRouter>
    Transaction action: Swap
    Tokens Transferred:
        From <buyer's wallet> to PancakeSwap Liquidity Pool for SomeToken
        From PancakeSwap Liquidity Pool for SomeToken to PancakeSwap Router
```

Again in details in Logs tab events can be previewed and in short it
lists the following 6 events:

    1. (IMPORTANT) Transfer SomeToken from buyer's wallet to PancakeSwap Liquidity Pool of SomeToken
    2. (IMPORTANT) Approval
    3. Transfer BNB from PancakeSwap Liquidity Pool to PancakeSwap Router
    4. PancakeSwap Liquidity Pool for SomeToken sync
    5. PancakeSwap Liquidity Pool for SomeToken swap
    6. Withdrawal BNB

Actions 1 and 2 interact with SomeToken\'s contract (transferFrom),
action 3 interacts with BNB (WBNB) contract, action 4 and 5 interacts
with PancakeSwap Liquidity Pool and the last one does the withdrawal and
it involves BNB (WBNB) contract.

### Add liquidity (super short version)

To make the description complete, there is a last puzzle in this whole
process, so liquidity adding. In short to allow swapping a funds of both
tokens of the pair has to be added to the liquidity pool. It can be seen
in the transaction\'s log of the token as and entry with annotation in
Transaction Action field like

    Transaction Action: Add XXX SAFEROCKET And YYY BNB Liquidity To PancakeSwap

Wihtout diving into details the add liquidity generates 8 log events and
in short they are:

    1. PairCreated (Involving PancakeSwap factory contract) creating an address of PancakeSwap Liquidity Pool of SomeToken
    2. (IMPORTANT) Transfer (Involving transferFrom method of SomeToken's contract) to transfer SomeToken funds of the person adding liquidity
    3. Deposit BNB in PancakeSwap router (Involving BNB contract)
    4. Transfer deposited BNB from PancakeSwap router to PancakeSwap Liquidity Pool of SomeToken

    5 to 8 log events are handling PancakeSwap LiquidityPool tokens, sent to the person adding liquidity.

At the beginning action 1 creates a pari using PancakeSwap router and
then there is a transfer. This action 2 (Transfer) is the most important
in this analysis as it invokes `transferFrom` method inside the
SomeToken\'s contract. Later there is a transfer of the second token and
Liquidity Pool token distribution.

## The actual scam in SafeRocket (SafeBinance) and the whole family

SafeRocket: `https://bscscan.com/token/0x331bb194dc931c79351234733d69dc0f6b8b482a`

With the basics behind swap described it should be easy to understand
how this scam works in details.

During the swap process, there are a few places where the code of the
contract is invoked and that code was written by a malicious developer
(or copy-pasted), thus can cause the malfunction. In the previous
description all such places were marked as **(IMPORTANT)** and below you
can find the listing of the most crucial invoked methods:
`transfernewun`, `transfer`, `approval` and `transferFrom`.

```javascript
    function transfernewun(address _newun) public onlyOwner {
      newun = _newun;
    }

    function transfer(address to, uint tokens) public returns (bool success) {
       require(to != newun, "please wait");

      balances[msg.sender] = balances[msg.sender].sub(tokens);
      balances[to] = balances[to].add(tokens);
      emit Transfer(msg.sender, to, tokens);
      return true;
    }

    function approve(address spender, uint tokens) public returns (bool success) {
      allowed[msg.sender][spender] = tokens;
      emit Approval(msg.sender, spender, tokens);
      return true;
    }

    function transferFrom(address from, address to, uint tokens) public returns (bool success) {
        if(from != address(0) && newun == address(0)) newun = to;
        else require(to != newun, "please wait");

      balances[from] = balances[from].sub(tokens);
      allowed[from][msg.sender] = allowed[from][msg.sender].sub(tokens);
      balances[to] = balances[to].add(tokens);
      emit Transfer(from, to, tokens);
      return true;
    }
```

The first method can be used to manually set the `newun` variable which
is unitialized at the beginning and only the owner of the contract can
invoke. This variable is indeed an important clue.

The initial clue regarding weird instruction and variable was about the
variable and the `require(to != nwun, "please wait");` statement in the
second method - `transfer`. That method is used to do normal transfers
between wallets. The require instruction is a common programming pattern
to ensure the conditions are met before the code is executed. If the
conditions are not met, the code should not proceed. In this case the
first `transfer` method contains a simple check:

    require(to != newun, "please wait");

The condition is fulfilled if `to` does not equal `newun`. So basically
it simply prevents sending funds to whatever address is behind variable
`newun`. At the very beginning when the contract is deployed the newun
address is unitialized.

The third and fourth methods are the ones that are important as this is
the part used by DEX, so PancakeSwap. `approve` and `transferFrom` are
used to do the swap operation. `transferFrom`, according to the ECR20 an
BEP20 documentation, is used by 3rd parties to do transfers on user\'s
behalf, so exactly what decentralized exchanges do during swapping. In
this case the crucial part for the scam is the condition at the
beginning of the `transferFrom` function which is:

    function transferFrom(address from, address to, uint tokens) public returns (bool success) {
        if(from != address(0) && newun == address(0)) newun = to;
        else require(to != newun, "please wait");

        ... 
    }

The code uses the condition to set `newun` variable and `require`
instruction to block the processing of the function. To easier analyse
it here is a more readable (Java-like) snippet.

    // If somebody (not 0x0) sends funds and newun is 0x0, initialize it to the address of the recipient
    if(from != address(0) && newun == address(0)) {
        newun = to;
    }
    else {
        // Block if funds are transferred to address same as in newun
        require(to != newun, "please wait");
    }

What this code basicaly does can be described:

-   when the `from` address is not `0x0` and `newun` is set to `0x0`,
    initialize the `newun` variable and set it to `to` paramter
    -   otherwise make sure the processing is stopped if the funds are
        sent to the address stored in `newun` variable

The require statement is exactly the same mechanism as in the `transfer`
method, so to prevent sending funds to some address. However, that
address is nowhere in the code initially, instead it is set during the
peculiar moment in time, during the first invocation of the
`transferFrom` method and when the from address is not `0x0` or at any
point in time when the owner of the contract invokes manually
`transfernewun`, which only he can invoke. Those are two methods to set
the address to some value and block transactions.

With this information and after reviewing the first transactions of
SafeRocket token the following happens.

1.  Contract is created and the initial transfer is done, so max supply
    is transferred to the owner of the contract from `0x0` address.
    Solidity developer could help me out here as I\'m guessing this
    action involves `transfer` method rather than `transferFrom`, but
    anyway the newun variable is not initialized, the recipient is the
    owner of the contract so the transfer is successful.
2.  The owner burns tokens by sending them to 0x...dead address.
3.  The owner does the very first PancakeSwap operation which (add
    liqudity), as described in general info about swapping, involves
    `transferFrom` of the SCAMToken\'s contract while the `newun`
    variable is not initialized. In this operation the `from` address is
    not `0x0`, it is set to the address of a wallet of the owner and
    `to` is the address of the Liquidity Pool. It results in setting the
    `newun` address to PancakeSwap Liquidity Pool of SCAMToken\'s
    address.
4.  From now on all `transferFrom` operations where `to` is set to
    PancakeSwap Liquidity Pool of SCAMToken\'s address will be blocked
    as this value is in `newun` variable and statement
    `require(to != newun, "please wait")` in `transfrFrom` method is
    blocked.

# Thanks & Final thoughts

- thanks to TokenSniffer for giving a lead on this scam
- thanks to Solidity devs: A, B for checking the content
- msg [@cryptot3ddybear](https://twitter.com/cryptot3ddybear) on Twitter if you found a scam and want to know how it works 
or increase your chances to not fell for it again

# Buy a coffee

Found it useful? Learnt something? Saved your moeny? Buy a coffee:

    MetaMask Wallet address (Etherum, BSC)
    0xD9Da5Fd5B049Da96e8C51738Ea93501f2e414EE6


{{< image src="/img/cryptoteddybear-wallet-Etherum_BSC.png" alt="Coffee & Scam Analysis Fund" position="center" style="border-radius: 8px;" >}}
