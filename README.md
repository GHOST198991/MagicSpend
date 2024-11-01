 contract address>,
  <UserOperation.sender and/or msg.sender of the withdraw call>,
  <chain ID>,
  withdrawRequest.asset,
  withdrawRequest.amount,
  withdrawRequest.nonce,
  withdrawRequest.expiry
)
```

MagicSpend is an [ERC-4337](https://eips.ethereum.org/EIPS/eip-4337) compliant paymaster (EntryPoint [v0.6](https://github.com/eth-infinitism/account-abstraction/releases/tag/v0.6.0)) and also enables withdraw requests with asset ETH (`address(0)`) to be used to pay transaction gas.

This contract is part of a broader MagicSpend product from Coinbase, which as a whole allows Coinbase users to seamlessly use their assets onchain.

<img width="661" alt="Diagram of Coinbase user making use of MagicSpend" src="https://github.com/coinbase/magic-spend/assets/6678357/42d3a8fc-a376-4139-9ea9-040cf094d74b">

We have started a [discussion](https://ethereum-magicians.org/t/proposed-json-rpc-method-wallet-getassetbalance/) around a possible new wallet RPC to enable apps to have better awareness of this "just in time" funding.

## Detailed Flows

When the withdrawing account is an ERC-4337 compliant smart contract (like [Smart Wallet](https://github.com/coinbase/smart-wallet)), there are three different ways the MagicSpend smart contract can be used

1. Pay gas only
2. Transfer funds during execution only
3. Pay gas and transfer funds during execution

### Pay gas only

<img width="901" alt="Pay gas only flow diagram" src="https://github.com/coinbase/magic-spend/assets/6678357/21274fb0-b901-4e20-bc1c-f320caa76e5b">

1. A ERC-4337 UserOperation is submitted to the bundler. The paymasterAndData field includes the MagicSpend address and the withdrawal request.
2. Bundler (EOA) calls EntryPoint smart contract.
3. Entrypoint first calls to `UserOperation.sender`, a smart contract wallet (SCW), to validate the user operation.
4. Entrypoint decrements the paymaster’s deposit in the Entrypoint. If the paymaster’s deposit is less than the gas cost, the transaction will revert.
5. EntryPoint calls the MagicSpend contract to run validations on the withdrawal, including checking the signature and ensuring withdraw.value is greater than transaction max gas cost.
6. Entrypoint calls to SCW with `UserOperation.calldata`
7. SCW does arbitrary operation, invoked by `UserOperation.calldata`.
8. Entrypoint makes post-op call to MagicSpend, with actual gas used in transaction.
9. MagicSpend sends the SCW any withdraw.value minus actual gas used.
10. Entrypoint refunds the paymaster if actual gas < estimated gas from (4.)
11. Entrypoint pays bundler for tx gas

### Transfer funds during execution only

<img width="600" alt="Diagram of 'Transfer funds during execution only' flow" src="https://github.com/coinbase/magic-spend/assets/6678357/124548ca-209d-41ac-844a 
```
