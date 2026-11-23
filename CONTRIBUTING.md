# Add your chain to Aria

This repository is the published Orchestra Labs chain registry. The Aria wallets read it, and so does anyone using
`https://registry.orchestralabs.org/chains`. A daily refresh rebuilds it from public registries, so the files at the top
level (`chains.json`, `chains-by-type/`, `ibc.json`, …) are generated: don't edit them.

There are two ways to add a chain.

## 1. Upstream (preferred)

The refresh reads these registries every day. A chain merged into one of them reaches this registry at the next refresh,
with no PR here.

| Chain family | Registry |
|---|---|
| Cosmos SDK | [cosmos/chain-registry](https://github.com/cosmos/chain-registry) or [chainapsis/keplr-chain-registry](https://github.com/chainapsis/keplr-chain-registry) |
| EVM | [ethereum-lists/chains](https://github.com/ethereum-lists/chains) |

## 2. A pull request here

If your chain isn't upstream, open a PR that adds one folder:

```
community/<mainnet|testnet>/<chain_type>/<chain_id>/chain.json
community/<mainnet|testnet>/<chain_type>/<chain_id>/assets.json
```

`chain_type` is `cosmos` or `eip155` for now. Other chain families need wallet support before the registry can list them.
`chain_id` is the Cosmos chain ID (`mychain-1`) or the EVM chain ID in decimal (`424242`). Touch nothing outside your
folder. A PR can't change a chain the registry already lists: for that, open an issue or a PR to its upstream registry.

### chain.json

```json
{
  "chain_name": "mychain",
  "chain_id": "mychain-1",
  "chain_type": "cosmos",
  "network_level": "mainnet",
  "pretty_name": "My Chain",
  "website": "https://mychain.example",
  "logo_uri": "https://mychain.example/logo.png",
  "bech32_prefix": "my",
  "slip44": 118,
  "key_algos": ["secp256k1"],
  "fees": [{ "denom": "umy", "gasPriceStep": { "low": 0.01, "average": 0.025, "high": 0.04 } }],
  "staking_denoms": ["umy"],
  "rpc_uris": [{ "address": "https://rpc.mychain.example", "provider": "My Chain Foundation" }],
  "rest_uris": [{ "address": "https://api.mychain.example", "provider": "My Chain Foundation" }],
  "explorers": [{ "kind": "My Explorer", "url": "https://explorer.mychain.example", "tx_page": "https://explorer.mychain.example/tx/${txHash}" }]
}
```

- Required: `chain_name` (lowercase letters, digits, `-`, `_`), `chain_id`, `chain_type`, `network_level` (matching the
  folder names), `pretty_name`, `fees` and `rpc_uris`.
- Cosmos chains also need `bech32_prefix`, `slip44` and `rest_uris`. `key_algos` is optional: `secp256k1`,
  `ethsecp256k1`, `ed25519` or `sr25519`.
- An EVM chain's fee denom is its smallest unit, usually `wei`.
- Every URL is https. List your own public endpoints first: the check and the daily probe try the first ten.
- Leave out `status` and `verification`: the registry sets them.
- Keys starting with `_` are notes for reviewers (sources, contacts) and are never published.

### assets.json

Each token by its denom, with at least the fee token:

```json
{
  "umy": { "symbol": "MY", "name": "My Chain", "exponent": 6, "logo": "https://mychain.example/my.png" }
}
```

## What the PR check tests

Every PR runs the same checks the daily refresh applies:

1. **Shape**: the fields above, and that the PR changes only `community/` chain folders.
2. **Not already listed**: the chain ID isn't in the registry already, for that chain type.
3. **Mainnet or testnet**: for EVM chains, Chainlink's and Hyperlane's chain lists and CoinGecko decide when they know the
   chain ID, then its names. For Cosmos chains, its chain ID and names: a mainnet whose names say testnet, devnet or a
   known test network is refused.
4. **It's live and it's the chain you say**: one of your first ten RPCs must report a block from the last 30 days. For
   EVM, `eth_chainId` must return your chain ID and `eth_sendRawTransaction` must exist. For Cosmos, `/status` must report
   your chain ID.

A maintainer reviews the PR as well and merges it. The next daily refresh publishes your chain.

## After it's listed

- Your chain is marked as a community submission: its `verification` reads `"by": ["community-pr"]`, and becomes
  `["community-pr", "rpc-probe"]` for the chain type once the daily probe confirms it. Wallets can show that the chain
  was listed by its team, not checked by Orchestra Labs.
- The probe runs every day. If none of your RPCs reports a block for 30 days, or they answer for another chain, the
  chain's `status` becomes `killed` and the wallets hide it. It comes back when the probe passes again.
- If your chain later appears upstream, the upstream entry replaces yours.
- To update your chain, open a PR that changes your folder. To remove it, open an issue.
