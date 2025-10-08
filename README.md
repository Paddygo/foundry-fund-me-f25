# foundry-fund-me-f25



## FundMe https://sepolia.etherscan.io/address/0x405bf238df8ab7fdfc8afcb52ab5fc0767abb4a1#tokentxns


A minimal funding/donation contract where users send ETH with a USD-denominated minimum enforced via Chainlink price feeds.

### Features
- Minimum contribution: 5 USD (scaled to 18 decimals as `5e18`)
- USD conversion via Chainlink `AggregatorV3Interface`
- Owner-only withdrawals with two modes: `withdraw` and gas-optimized `cheaperWithdraw`
- Private state + explicit getters
- `receive`/`fallback` route ETH directly to `fund`

### How it works
- Price conversion uses `PriceConverter`:
  - `getLatestRoundData` fetches ETH/USD
  - Chainlink ETH/USD feeds typically have 8 decimals; the library scales to 18 so `MINIMUM_USD = 5e18` is comparable to `msg.value` in wei.
- Access control:
  - `i_owner` is set in the constructor and treated as immutable owner
  - `onlyOwner` uses custom error `NotOwner()`
- Gas optimization:
  - `cheaperWithdraw` caches `s_funders.length` in memory to avoid repeated storage reads during the loop, reducing gas (≈100 gas per extra read avoided, depending on context)

### Contract interface
- `constructor(address priceFeed)`:
  - `priceFeed`: Chainlink ETH/USD aggregator (e.g., Sepolia: see Chainlink docs)
- `fund()` payable:
  - Reverts unless `msg.value` in USD ≥ 5
- `withdraw()` onlyOwner:
  - Resets contributions and transfers full balance
- `cheaperWithdraw()` onlyOwner:
  - Same as `withdraw` but with reduced gas by caching array length
- Getters:
  - `getOwner()`, `getFunder(uint256)`, `getAddressToAmountFunded(address)`, `getVersion()`

### Fallback behavior
- `receive()` and `fallback()` both call `fund()`, so sending ETH directly enforces the USD minimum.

