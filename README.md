# Uniswap V2 Core
This is just for the core repo only. For the v2 periphery repo, please go to my forked periphery repo.
## Report
Like v1, v2 also use a factory to create pairs(exchanges in v1). Instead of using ETH for each currency, v2 create an exchange for a pair of erc-tokens. To transfer from currency to ETH, it makes use of WETH, which provide a erc-20 token inteface and can transact to ETH in 1:1 ratio. This reduces the double charing of 0.3% from both exchange in v1. Since ETH is not the middle man anymore, to swap from one token to another token, one needs to use router.

To provide price oracle a way to calculate average prices, Uniswap v2 provides functions and stores reserves of both tokens, priceCumulativeLast for both tokens and blockTimestampLast. They are recalculated everytime _update is called. Reserves may not the same as balances when someone just transfer tokens withhout swaping/ changing liquidity with the contracts. This is intended as explained in the last paragraph of section 2.2 in the white paper (https://uniswap.org/whitepaper.pdf).

To provide charge fee for the protocol, kLast and reserves are used to calculate the accumulated fee, which in the end provided 1/6 of it to the feeTo address. The fee is collected everytime liquidity is changed, instead of every transaction to reduce gas fee for transactions.

To provide flash swap, the swap function transfer both token0 and token1 to the "to" address first, and pass data to the "to" address if there are data. This allows the contract in "to" to use the tokens to do other stuff first in this transaction. After it, the swap function checks whether the promised invariant still holds. If it does, everything is fine no one is owing money. Otherwise, it reverts the process like nothings has happened (except the user still has to pay the gas fee).

Also, V2 has chosen to use other contracts (in the periphery repo) to interact with the pair contract. This allows different people use their own contracts, like oracles, own router contracts to interact with the pair contracts. This makes the contracts more modular.

The burn function requires the sender to send the tokens to "this" address before calling the burn function. It just burns all the liquidity tokens it has. This means that if someone sends liquidity tokens to the contract without burning it. The next one who burn the liquidity token can get more tokens. (This probably wouldn't be a security problem as no one would do such thing).

The permit functions allow users to let others to transfer their liquidity tokens after signing to show that they permit the transaction.

### What I have learnt
1. Importance to checking overflow and underflow. This repo uses safeMath and UQ112x112 libraries to handle the overflow issues. It also uses skim() to avoid the reserves to overflow as they are represented in uint112.
2. Use of the revert mechanism. Using require to safe guard to check the condition afterwards allows the user to do more stuff in a single transaction, e.g. flash swap.
## Unsolved Questions
1. Protocol Fee 
Why is the accumulated fee =1-sqrt(k1)/sqrt(k2)? I think the square root should be related to the initialization of liquidity token supply 
s_minted = sqrt(x_deposited · y_deposited)
but doesn’t this equation no longer hold true after swapping? I would like to know how the formula of the accumulated fee is derived.
2. Sync()
I don’t really get when sync would be called. I don’t get what "in the case that a token asynchronously deflates the balance of a pair" really means. I would love to have an example of it.

## TODO
- read about abi and ecrecover
- read eip-712
