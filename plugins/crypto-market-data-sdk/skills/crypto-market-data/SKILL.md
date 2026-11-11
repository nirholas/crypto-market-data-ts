---
name: crypto-market-data
description: Write TypeScript or JavaScript that fetches crypto prices, trending coins, global market stats, DeFi TVL and the Fear and Greed Index with the @nirholas/crypto-market-data library. Use when a project needs cached, rate-limited CoinGecko and DeFiLlama data, including in Edge Runtime.
---

# @nirholas/crypto-market-data

A zero-dependency client over the public CoinGecko, DeFiLlama and Fear and Greed APIs with built-in caching and rate limiting. No API key is required. Node 18 or newer (it uses global `fetch`), ESM and CommonJS builds.

```
npm install @nirholas/crypto-market-data
```

## Usage

A default instance `marketData` is exported, along with one function per method (for example `getTopCoins`). Create your own client to change limits:

```ts
import { MarketDataClient, MarketDataError, formatPrice } from '@nirholas/crypto-market-data';

const client = new MarketDataClient({
  rateLimitWindow: 60000,     // ms, default 60000
  maxRequestsPerWindow: 25,   // default 25
  timeout: 10000,             // ms, default 10000
  userAgent: 'my-app/1.0',
});

try {
  const top = await client.getTopCoins(10);
  for (const c of top) console.log(c.symbol, formatPrice(c.current_price), c.price_change_percentage_24h);
} catch (err) {
  if (err instanceof MarketDataError && err.isRateLimited) {
    console.log('rate limited, status', err.statusCode);
  }
}
```

## Methods

- Prices: `getSimplePrices()` (bitcoin, ethereum, solana in USD with 24h change), `getTopCoins(limit = 50)`, `getTrending()`, `getCoinDetails(coinId)`, `searchCoins(query)`, `compareCoins(coinIds[])`, `getCoinsList()`.
- History: `getHistoricalPrices(coinId, days | 'max', interval?)` where interval is `'minutely' | 'hourly' | 'daily'`, `getOHLC(coinId, days)`, `getHistoricalPrice(coinId, date)`.
- Market: `getGlobalMarketData()`, `getMarketOverview()`, `getFearGreedIndex()` (`value`, `value_classification`), `getGlobalDeFiData()`, `getDerivativesTickers()`.
- DeFi: `getTopProtocols(limit = 20)`, `getTopChains(limit = 20)`.
- Exchanges and categories: `getExchanges(perPage, page)`, `getExchangeDetails(id)`, `getCoinTickers(coinId, page)`, `getCategories()`, `getCategoryCoins(categoryId, perPage, page)`.
- Community: `getCoinDeveloperData(coinId)`, `getCoinCommunityData(coinId)`.
- Client control: `clearCache()`, `getCacheStats()`, `getRateLimitStatus()` returning `remaining`, `windowResetAt`, `isBlocked`.
- Formatting helpers: `formatPrice`, `formatNumber`, `formatPercent`, `getFearGreedColor`, `getFearGreedBgColor`.

Coin IDs are CoinGecko IDs (`bitcoin`, `solana`), not ticker symbols; use `searchCoins` to resolve a symbol.

## Notes

- Some methods return `null` on failure (`getFearGreedIndex`, `getGlobalMarketData`, `getCoinDetails`), so null-check them.
- The free CoinGecko tier is limited; keep the default rate limit unless you have measured headroom. Repeated calls inside the cache window are served from cache.
- To verify before writing code, read `src/index.ts` in the repo; every method above is defined there.
