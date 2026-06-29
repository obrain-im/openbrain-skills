# Open Brain Wallet Reference

Use this reference before running Open Brain wallet commands or explaining wallet behavior.

## Overview

Open Brain wallet is an ERC-7715-based agent wallet. The Open Brain CLI maintains a local encrypted EVM crypto wallet under the Open Brain config directory; `obrain login` initializes or reuses this wallet. This wallet is fully controlled by the agent.

The agent wallet can sign messages and submit transfers, swaps, or other transactions from this wallet, but the wallet itself does not hold or custody user assets. Asset movement requires authorization from the user's main wallet. The agent can initiate an approve request through `obrain`; the user must then review and complete the approve action on the OpenBrain platform. A permission grants the agent wallet the ability to use up to a specified amount of an asset from the user's authorized wallet within a specified time window.

Within an active permission, the agent can transfer or swap assets from the user's authorized wallet. For transfers, the source address is the user's authorized wallet and the destination address is controlled by the user. For swaps, the input asset comes from the user's authorized wallet and the swapped output is returned to the user's wallet.

Before operating on assets, check whether an active permission exists for the asset and whether the remaining quota is sufficient. If no permission exists, or the quota is insufficient, request permission first. Only submit the transfer or swap after the permission is active and sufficient.

Describe permission approval as happening on the OpenBrain platform: the user reviews the pending approve record and completes the approve action there.

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

Check the balance of a user's wallet for a token or native asset:

```bash
obrain wallet balance eip155:56:0xYourWallet eip155:56:0xTokenAddress
obrain wallet balance eip155:56:0xYourWallet eip155:56:NATIVE
```

Submit a transfer using an approved wallet permission:

```bash
obrain wallet transfer <asset> <amount> <to> [--rpc-url <url>]
```

The recipient must be an EVM address chosen by the user. The source address is the user's authorized wallet, not the agent wallet. The command validates that the approved grant matches the local wallet, asset chain, asset type, token address, and approved amount before submitting the transaction.

Request a swap quote or submit a swap using an approved permission:

```bash
obrain wallet swap <fromAsset> <toToken> <amount> --dry-run [--slippage-bps <bps>] [--rpc-url <url>]
obrain wallet swap <fromAsset> <toToken> <amount> [--slippage-bps <bps>] [--rpc-url <url>]
```

Swap currently supports BSC assets (`eip155:56`). The input asset is spent from the user's authorized wallet, and the swap output is returned to the user's wallet. Use `--dry-run` first to inspect the quote, router, expected output, minimum output, price impact, and request ID. `--slippage-bps` defaults to `100` and must be between `0` and `10000`.

## Asset Operation Flow

For transfer or swap commands:

1. Identify the asset subject, chain ID, amount, recipient or output token, and intended operation.
2. Check the user's authorized wallet balance with `obrain wallet balance <wallet> <asset>`.
3. Check active wallet permissions with `obrain wallet permission list`.
4. If permission is missing or quota is insufficient, request authorization:

   ```bash
   obrain wallet permission request <subject> <quota> <hour|day|week|month> [--agent-id <agent-id>]
   ```

   Tell the user this starts an OpenBrain approve request for the specified asset, quota, and time window. The user should review the pending approve record on the OpenBrain platform and complete the approve action there.

5. Inspect the permission if needed:

   ```bash
   obrain wallet permission info <id>
   ```

6. Only submit the transfer or swap when the permission is active and sufficient.
7. For swaps, run `--dry-run` first to inspect the quote, route, output, and slippage.
8. Pass `--rpc-url` when the chain has no built-in RPC URL or the default RPC is unsuitable.

Wallet operations are funds-sensitive. Be conservative: verify asset, chain, amount, source wallet, recipient, quota, and slippage before submitting a transaction.

The agent's control of the wallet does not bypass permission requirements or user approval for moving user-authorized assets.

If a wallet command fails, read the printed error before retrying. Swap failures include the current slippage and may suggest retrying with a higher `--slippage-bps` when price limits or route movement are likely.
