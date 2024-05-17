## In the function PrizePool.setDrawManager(), anyone can frontrun it and become the drawManager
Reading the documentation of the Prize Pool contract, the following is specified: `The Prize Pool allows a 'draw manager' contract to complete the Draw and withdraw tokens from the reserve.` In the code, on [line 296](https://github.com/GenerationSoftware/pt-v5-prize-pool/blob/4bc8a12b857856828c018510b5500d722b79ca3a/src/PrizePool.sol#L296), it is specified that the PrizePool.setDrawManager() function `Allows a caller to set the DrawManager if not already set.` This function is not protected in cases where a malicious attacker wants to front-run and take control of the draw manager permissions.

### PROOF OF CONCEPT
[PoolTogether docs link :](https://dev.pooltogether.com/protocol/next/design/prize-pool#incentivized-draws)
```text
The Prize Pool allows a "draw manager" contract to complete the Draw and withdraw tokens from the reserve. 
```

### Mittigation 
We recommend adding access control to the function or directly setting the drawManager in the constructor and removing this function.
