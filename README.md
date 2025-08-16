# Drosera-unique-trap
Trap for Drosera node, which is detecting exacr amount of incoming tokens and if it's equal to pre-defined, it will trigger event.

## How It Works

- `collect()` records the current balance of the target wallet.  
- `shouldRespond()` compares the latest balance with the previous one.  
- If the balance increased by `0.0000777 ETH`, the trap returns `true`.

## Trap Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface ITrap {
    function collect() external view returns (bytes memory);
    function shouldRespond(bytes[] calldata data) external pure returns (bool, bytes memory);
}

contract IncomingTransferTrap is ITrap {
    address public constant target = 0xYourWallet; // Wallet to monitor
    uint256 public constant minAmount = 0.0000777 ether; // Threshold for incoming transfer, you can set your own value

    function collect() external view override returns (bytes memory) {
        return abi.encode(target.balance);
    }

    function shouldRespond(bytes[] calldata data) external pure override returns (bool, bytes memory) {
        if (data.length < 2) return (false, "Insufficient data");

        uint256 current = abi.decode(data[0], (uint256));
        uint256 previous = abi.decode(data[1], (uint256));

        if (current > previous && (current - previous) >= minAmount) {
            return (true, "");
        }

        return (false, "");
    }
}

## Deployment

    Add the contract to your drosera.toml configuration under [[traps]].

    Compile the contract with drosera build.

    Apply the configuration with drosera apply.

    Start your operator and monitor the dashboard/logs.

When the target wallet receives 0.0000777 ETH, the trap will trigger and show a dark green block in the dashboard.
