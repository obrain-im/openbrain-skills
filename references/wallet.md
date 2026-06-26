# Open Brain Wallet Reference

Use this reference before running Open Brain wallet commands or explaining wallet behavior.

## Overview

Open Brain wallet is an ERC-7715-based agent wallet. The Open Brain CLI maintains a local encrypted EVM crypto wallet under the Open Brain config directory; `obrain login` initializes or reuses this wallet.

The agent wallet can sign messages and submit approved transfers, swaps, or other transactions, but it does not custody user assets by itself. Asset movement requires authorization from the user's main wallet. A permission grants the agent wallet the ability to use up to a specified amount of an asset within a specified time window.

Before operating on assets, check whether an active permission exists for the asset and whether the remaining quota is sufficient. If no permission exists, or the quota is insufficient, request permission first. Only submit the transfer or swap after the permission is active and sufficient.

## Commands

Show the local wallet address:

```bash
obrain wallet info
```

Sign a message and record the wallet activity remotely:

```bash
obrain wallet sign-message "hello" [--agent-id <agent-id>]
obrain wallet sign-message --file ./message.txt [--agent-id <agent-id>]
```

Provide either a message argument or `--file`, not both.

Request, inspect, or list wallet execution permissions:

```bash
obrain wallet permission request <subject> <quota> <hour|day|week|month> [--agent-id <agent-id>]
obrain wallet permission info <id>
obrain wallet permission list [--agent-id <agent-id>] [--limit <1-100>]
```

Use asset subjects like:

```text
eip155:<chainId>:NATIVE
eip155:<chainId>:0xTokenAddress
```

Submit a transfer using an approved wallet permission:

```bash
obrain wallet transfer <asset> <amount> <to> [--rpc-url <url>]
```

The recipient must be an EVM address. The command validates that the approved grant matches the local wallet, asset chain, asset type, token address, and approved amount before submitting the transaction.

Request a swap quote or submit a swap using an approved permission:

```bash
obrain wallet swap <fromAsset> <toToken> <amount> --dry-run [--slippage-bps <bps>] [--rpc-url <url>]
obrain wallet swap <fromAsset> <toToken> <amount> [--slippage-bps <bps>] [--rpc-url <url>]
```

Swap currently supports BSC assets (`eip155:56`). Use `--dry-run` first to inspect the quote, router, expected output, minimum output, price impact, and request ID. `--slippage-bps` defaults to `100` and must be between `0` and `10000`.

## Asset Operation Flow

For transfer or swap commands:

1. Identify the asset subject, chain ID, amount, recipient or output token, and intended operation.
2. Check active wallet permissions with `obrain wallet permission list`.
3. If permission is missing or quota is insufficient, request authorization:

   ```bash
   obrain wallet permission request <subject> <quota> <hour|day|week|month> [--agent-id <agent-id>]
   ```

4. Inspect the permission if needed:

   ```bash
   obrain wallet permission info <id>
   ```

5. Only submit the transfer or swap when the permission is active and sufficient.
6. For swaps, run `--dry-run` first to inspect the quote, route, output, and slippage.
7. Pass `--rpc-url` when the chain has no built-in RPC URL or the default RPC is unsuitable.

Wallet operations are funds-sensitive. Be conservative: verify asset, chain, amount, recipient, quota, and slippage before submitting a transaction.

If a wallet command fails, read the printed error before retrying. Swap failures include the current slippage and may suggest retrying with a higher `--slippage-bps` when price limits or route movement are likely.
