# minimal-eth-burner

On Ethereum, it's common to "burn" (destroy) ETH by sending it to a "burner address" like [`0x000000000000000000000000000000000000dead`](https://etherscan.io/address/0x000000000000000000000000000000000000dead).

However, any ETH sent to burner addresses is theoretically recoverable if the private keys associated with the burner addresses are ever discovered. The only way to actually destroy ETH is by either:
- Burning it in [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) or [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) transaction base gas fees, or
- Sending ETH to a smart contract which calls `SELFDESTRUCT` and sends its ETH to itself.

This repository contains the minimal amount of code to create a new smart contract that immediately calls `SELFDESTRUCT` and sends its ETH to itself. All ETH sent to the smart contract creation transaction will be irreversibly destroyed.

## Usage

See example JavaScript below. Copy this code into the DevTools console in any web browser where you have an [EIP-1193](https://eips.ethereum.org/EIPS/eip-1193) wallet (e.g., MetaMask) installed.

> [!CAUTION]<br>
> All ETH sent in this transaction will be irreversibly destroyed.

```javascript
const addr = (await ethereum.request({ method: 'eth_requestAccounts' }))[0]; // Your current wallet address
const deployer = '0x696130ff5f526002601ef35f52600a601634f05f5f5f5f5f855af1'; // deployer.evm, compiled
const burn = '100000000000000000'; // Amount of ETH to burn (in wei)

// Irreversibly destroys all ETH sent
await ethereum.request({
  method: 'eth_sendTransaction',
  params: [{
    from: addr,
    data: deployer,
    value: `0x${BigInt(burn).toString(16)}`
  }]
});
```

## Multiple wallets

If you have multiple wallets installed, use the [EIP-6963](https://eips.ethereum.org/EIPS/eip-6963) `eip6963:requestProvider` event to select which wallet you want to use:

```javascript
const wallets = [];
window.addEventListener('eip6963:announceProvider', e => wallets.push(e.detail));
window.dispatchEvent(new Event('eip6963:requestProvider'));
console.table(wallets.map(w => w.info.name));
const { provider } = wallets[0]; // Replace 0 with the index of the wallet you want to use

// The remainder is the same as above except using `provider` instead of `ethereum`
const addr = (await provider.request({ method: 'eth_requestAccounts' }))[0]; // Your current wallet address
const deployer = '0x696130ff5f526002601ef35f52600a601634f05f5f5f5f5f855af1'; // deployer.evm, compiled
const burn = '100000000000000000'; // Amount of ETH to burn (in wei)

// Irreversibly destroys all ETH sent
await provider.request({
  method: 'eth_sendTransaction',
  params: [{
    from: addr,
    data: deployer,
    value: `0x${BigInt(burn).toString(16)}`
  }]
});
```

## EIP-6780

[EIP-6780](https://eips.ethereum.org/EIPS/eip-6780) (which went live in the [Dencun upgrade](https://eips.ethereum.org/EIPS/eip-7569) in March 2024) updates the behavior of the `SELFDESTRUCT` opcode so that a smart contract calling `SELFDESTRUCT` will only delete itself when it's called in the same transaction that created it.

Since the `deployer.evm` bytecode creates a new smart contract and then immediately calls it in the same transaction, its behavior is not changed by EIP-6780.

## References
- [Best way to burn ethers and other ethereum tokens?](https://ethereum.stackexchange.com/questions/16188/best-way-to-burn-ethers-and-other-ethereum-tokens/17617)
- [Burner.sol](https://etherscan.io/address/0xb69fba56b2e67e7dda61c8aa057886a8d1468575)
- [Burn.sol (from Optimism)](https://github.com/ethereum-optimism/optimism/blob/eb68d8395971bc4a125cd0fd07567547f5bc0c49/packages/contracts-bedrock/contracts/libraries/Burn.sol#L33-L42)
