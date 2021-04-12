+++
title = "SCAM-HOWTO: How to recognize Stealthy Liquidity Pool Block on PancakeSwap"
date = "2021-04-12T14:45:13+02:00"
author = "cryptot3ddybear"
authorTwitter = "cryptot3ddybear" #do not include @
cover = ""
tags = ["scam", "howto", "bsc", "pancakeswap", "solidity", "security", "blockchainsec"]
keywords = ["bsc", "howto", "scam", "lp", "security"]
description = ""
showFullContent = false
+++

A easy guide for traders to check whether a contract is another instance of
SafeBinance family scam as it turns out multiple scams token from this family 
are created every day. A full analysis can be found here [BSC-SCAM: Stealthy Liquidity Pool Block on PancakeSwap](/posts/scam-explained-bsc-stealthy-lp-block/). It would not save you in every case, 
but maybe it might increase your probability to not lose your money in certain cases.


As there are multiple tokens popping up daily, belonging to this scam family, 
here is a quick guideline how to recognize another instance without any technical 
skills, only using your browsers and a few clicks. 

The tutorial is based on MarsBank (MARSBANK) token which 
seems to belong to the same family, but by the time of writing this article was not yet flagged as a scam.

```javascript
0x9d69c6138e819d366ab07a1091d2ab623031e0e8
```

## Step 1 -- You got an address. Great. Go to TokenSniffer

TokenSniffer does an excellent job showing similar tokens. However, as new 
tokens are created every day it might happen that some are not flagged as a scams. 
As a trader, make sure to check if the token was already analysed by TokenSniffer. 
Go to the following URL to check the token you want to buy and check if it was 
analysed:

<https://tokensniffer.com/token/0x9d69c6138e819d366ab07a1091d2ab623031e0e8>

{{< image src="/img/bsc-scam-lpblock/howto/step-01.png" alt="Check token on TokenSniffer" position="center" style="border-radius: 8px;" >}}

As presented in the screenshot, there are multiple tokens with a very similar contract. 
Even though the main token is not flagged as a scam, check some highlighted tokens, as they 
might have already been reported. Keep in mind the Score closer to 1 means the contract 
is more similar to a given token. View can be clicked to see the differences, as usually it's 
only a different name.

{{< image src="/img/bsc-scam-lpblock/howto/step-01-2.png" alt="Check similar tokens" position="center" style="border-radius: 8px;" >}}

If it is very similar to other token that is marked as a scam, this token will be most probably 
a scam also. You can already do not bother to check it further, however to be sure you can 
check other steps.

> There might be a case where there is no info in TokenSniffer about the token you are 
> checking. It means it was not analysed and you simply need to wait and lose a chance of 
> early buy or risk. Do your own research and remember those are your money. #bsSAFU!

## Step 2 -- Go to BSCSCAN to the details of the token

To go deeper, go to BSCSCAN.com. The website shows all the transactions, address of the contract, and allows you to see some properties of the contract behind the token. 

<https://bscscan.com/token/0x9d69C6138E819D366AB07A1091D2ab623031E0E8#readContract>

{{< image src="/img/bsc-scam-lpblock/howto/step-02.png" alt="Check details of the token in BscScan" position="center" style="border-radius: 8px;" >}}

I would not talk about others indicators of a potential scam token, but let's focus on 
the most imporatant check and go to **Read Contract** tab.

> You can see the **Read Contract** tab at the bottom of the screenshot above.

Next stage is to check whether there is a variable `newun` and what is its value. More details why it is this variable and what it does you can find in the previous blog post about the analysis of this scam (link at the top).

{{< image src="/img/bsc-scam-lpblock/howto/step-02-2.png" alt="Read Contract > newun variable" position="center" style="border-radius: 8px;" >}}

As presented in the screenshot the variable `newun` is there and is set to an address. 

> I folded irrelevant functions, so simply scroll down after selecting **Read Contract** if `newun` is not at the top. 

The address is set to `0x3edccf0d87f7c430c4deacf4a2ed1e7bfb73b1ee` and when the link is clicked 
it leads to the address of the PancakeSwap Liquidity Pool of (in this case): MarksBank 3 token.

{{< image src="/img/bsc-scam-lpblock/howto/step-02-3.png" alt="Newun leads to PancakeSwap Liquidity Pool" position="center" style="border-radius: 8px;" >}}

Naturally, there might be some legit contracts using this variable, however each contract I checked and were highlighted as similar and not yet flagged as a scam on TokenSniffer were a copy-paste and a potential scam.

## Step 3 -- Final chart confirmation

If you want to be sure the token is a scam or you simply bought a token, you got a `transferFrom` error 
in PancakeSwap and cannot get back your money, you can check the last indicator -- a *only green chart*.

{{< image src="/img/bsc-scam-lpblock/howto/step-03.png" alt="Check the address of newun, pointing to PancakeSwap" position="center" style="border-radius: 8px;" >}}

In this case it is clear that all people who bought are unable to swap it back. However, keep in mind that 
scammers can fake some swap-back transactions to show here some red candles. However, if it is green only 
and you are unable to swap it was unfortunately a scam.


# Final thoughts

- msg [@cryptot3ddybear](https://twitter.com/cryptot3ddybear) on Twitter if you found a scam and want to know how it works 
or increase your chances to not fell for it again
- for more details about this scam family go to previous posts

# Buy a coffee

Found it useful? Learnt something? Saved your moeny? Buy a coffee:

    MetaMask Wallet address (Etherum, BSC)
    0xD9Da5Fd5B049Da96e8C51738Ea93501f2e414EE6


{{< image src="/img/cryptoteddybear-wallet-Etherum_BSC.png" alt="Coffee & Scam Analysis Fund" position="center" style="border-radius: 8px;" >}}
