+++
title = "BSC-SCAM: PancakeSwap NoApproval"
date = "2021-04-08T22:17:00+00:00"
author = "cryptot3ddybear"
authorTwitter = "cryptot3ddybear" #do not include @
cover = ""
tags = ["scam", "bsc", "pancakeswap", "solidity", "security", "blockchainsec"]
keywords = ["bsc", "scam", "approval", "security"]
description = ""
showFullContent = false
+++

A guide for crypto traders and a technical analysis of a popular 
Binance Smart Chain smart contract scam resulting in an inability 
to Approve swapping on PancakeSwap.


> Originaly posted as gist [LINK](https://gist.github.com/cryptoteddybear/9f771ec284c3cbf70486cc2087b4deb7), but migrated to a new blog.


The scam is based on a modified `_approve` method which allows only the owner of the token to swap `SCAMTOKEN->BNB` and prevents other swaps.

# Example and analysis is based on

- token: Ape
  + https://bscscan.com/token/0x4ec0a6637c8b81593a2a893d10754bd3cb5254e7
- contract
  + https://bscscan.com/address/0x4ec0a6637c8b81593a2a893d10754bd3cb5254e7
- scammer's wallet:
  + https://bscscan.com/address/0x8dcf2b0c9a75b565e451a7f76067714cd96f03a1

# Other similar scam coins

- Eclipse - https://tokensniffer.com/token/0xe50b465ae2356cae271c464fa440e06dd8e0cb04
- Shining Star - https://tokensniffer.com/token/0x45be63857c9df528542d3e72355e193a8dc0896f
- SafeGravity - https://tokensniffer.com/token/0x563a930333dd3934ecc4ab0006ddd8d05eca5873

# TLDR;

- First transaction is a transfer of nearly all funds from `0x0000000000000000000000000000000000000000` to the owner of the contract
- Second transaction is add liquidity of all transfered funds and BNB to Pancake LP
- People are swapping `BNB->SCAMTOKEN`, but nobody can swap back, due to no way to `_approve`
- Scammer can from time to time to a swap back to make it look more legit as he is the only one who can swap `SCAMTOKEN->BNB`
- Scammer pulls out his LP token

# For traders

- code in a contract with IF clause and fixed address in `_approve` method
- big liquidity is added at the very beginning
- no locked liquidity (like for most of the fresh tokens)
- swaps back (`SCAMTOKEN->BNB`) are only from a fixed wallet or maybe multiple fixed wallets in future, who knows how scammers modify that scam
- chart is nearly green all the time as people swap and fall into a trap, but to see such chart some people need to fell into the trap already ;(


# Analysis

## STEP 1 -- Create token and the contract

The scammer in step one creates a contract with malicious `_approve` method:

https://bscscan.com/address/0x4ec0a6637c8b81593a2a893d10754bd3cb5254e7#code

```javascript
  function _approve(address owner, address spender, uint256 amount) internal {
    require(owner != address(0), "BEP20: approve from the zero address");
    require(spender != address(0), "BEP20: approve to the zero address");
    
    if (owner == address(0x8Dcf2B0C9A75b565e451a7F76067714cd96F03a1)) {
        _allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    } else {
        _allowances[owner][spender] = 0;
        emit Approval(owner, spender, 0);
    }
  }
```

The if clause emits approval only when the owner of the contract is doing approving.

The following is the main part:

```javascript
    if (owner == address(0x8Dcf2B0C9A75b565e451a7F76067714cd96F03a1)) {
        _allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    } 
```

## STEP 2 -- Add liquidity

The scamer next adds a lot of liquidity.

https://bscscan.com/tx/0xf24a7fe95dcffe4c7c6c782668f96ec2be52b34d996bc452a8847a1dd7352387

According to Transaction Action.

```
Add 95,000 APE And 11 BNB Liquidity To PancakeSwap
```

## STEP 3 -- Waiting for victims and swaps

People are swapping, but nobody is actually swapping it back as it is not possible due to the malicious `_approve` function.
   
On a chart it can be seen that the chart is only green and constantly increasing, however as you can see in APE example, the scammer did swap back at some point, to pretend it is possible to swap back and for a moment the chart had some red candles.

https://bscscan.com/tx/0x10ebd9e978d60ccecb08edbd8574fed206842e3a625c022346f50cd4513d0422

Sample swap back transaction from scammer's wallet (owner of the contract)

```
From 0x8dcf2b0c9a75b565e451a7f76067714cd96f03a1To PancakeSwap: APE 21 For 100 Ape (APE)
From PancakeSwap: APE 21To PancakeSwap: Router For 0.012071609240905323 ($5.05) Wrapped BNB (WBNB)
```

## STEP 4 -- Cash-out

When scammer wants to cash out, he/she simply takes out the LP token and BNB is increased, while the scam token is decreased, due to AMM and liquidity pools mechanics. People after all were only able to deposit BNB and buy SCAMToken.

https://bscscan.com/tx/0x548d263b4c3197e5be7c3d6615c87d558984e8404f815701c8e38dcb6452c731

## Results

In this example with APE token it was not so smooth as the token has not got a traction and the scammer has not earned a lot.

```
Remove 92,317.1 APE And 11.3202162031125056 BNB Liquidity FromPancakeSwap
```

compared to what was put into the liquidity pool

```
Add 95,000 APE And 11 BNB Liquidity ToPancakeSwap
```

So only ~0.32 BNB earned, this time. In other cases it was way more around 11 BNB.


# Final thoughts

- msg [me@Twitter](https://twitter.com/cryptot3ddybear) if you found a scam and want to know how it works 
or increase your chances to not fell for it again

# Buy a coffee

Found it useful? Learnt something? Saved your moeny? Buy a coffee:

    MetaMask Wallet address (Etherum, BSC)
    0xD9Da5Fd5B049Da96e8C51738Ea93501f2e414EE6


{{< image src="/img/cryptoteddybear-wallet-Etherum_BSC.png" alt="Coffee & Scam Analysis Fund" position="center" style="border-radius: 8px;" >}}

