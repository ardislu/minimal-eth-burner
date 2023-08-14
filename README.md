# minimal-eth-burner

In Ethereum, it's common to send ETH to a "burner address" like [`0x000000000000000000000000000000000000dead`](https://etherscan.io/address/0x000000000000000000000000000000000000dead) to effectively "destroy" ETH.

However, this ETH is theoretically recoverable if the private key for this public address is ever discovered. The only way to actually destroy ETH is by sending ETH to a smart contract which calls `SELFDESTRUCT` and sends its ETH to itself.

This repository contains the minimal amount of code to create a new smart contract that immediately calls `SELFDESTRUCT` and sends its ETH to itself. All ETH sent to the smart contract creation transaction will be irreversibly destroyed.

## Usage

See example JavaScript below. Copy this code into the DevTools console in any web browser where you have a wallet (e.g., MetaMask) installed.

**WARNING: all ETH sent in this transaction will be irreversibly destroyed.**

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
