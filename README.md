# Crypto Triangular Arbitrage Bot

A Python-based cryptocurrency triangular arbitrage detector and paper trading bot built in Google Colab. The project uses live exchange rate data from the CoinGecko API and executes simulated trades through Alpaca's paper trading environment — no real money involved.

---

## What Is Triangular Arbitrage?

Triangular arbitrage exploits pricing inefficiencies across three or more assets. In a perfect market, if you convert Currency A → B → C → A, you should end up with exactly what you started with. When the product of exchange rates along a path deviates from 1.0 (a "disequilibrium factor"), a profit opportunity exists.

This project:
1. Fetches real-time cross-rates between 13 cryptocurrencies
2. Models them as a weighted directed graph
3. Finds all simple paths between every currency pair
4. Computes a **disequilibrium factor** (forward_weight × reverse_weight) for each path
5. Identifies the most and least efficient trading loops
6. (Planned) Submits paper trade orders via Alpaca for the best opportunity found

A factor **greater than 1.0** indicates a potential arbitrage opportunity. A factor **less than 1.0** indicates a loss on the round trip.

---

## Cryptocurrencies Tracked

| Ticker | Name          |
|--------|---------------|
| BTC    | Bitcoin       |
| ETH    | Ethereum      |
| XRP    | Ripple        |
| ADA    | Cardano       |
| BCH    | Bitcoin Cash  |
| EOS    | EOS           |
| LTC    | Litecoin      |
| XLM    | Stellar       |
| DOGE   | Dogecoin      |
| SOL    | Solana        |
| DOT    | Polkadot      |
| AVAX   | Avalanche     |
| CELO   | Celo          |

> **Note:** ADA (Cardano) is excluded from reverse path calculations because CoinGecko does not provide reverse ADA trading pairs, making round-trip arbitrage through ADA non-computable.

---

## Tech Stack

| Tool / Library | Purpose |
|----------------|---------|
| Python 3        | Core language |
| Google Colab    | Development and execution environment |
| [CoinGecko API](https://www.coingecko.com/en/api) | Live cryptocurrency exchange rate data (free, no key required) |
| [NetworkX](https://networkx.org/) | Directed graph construction and path traversal |
| [Alpaca](https://alpaca.markets/) | Paper trading execution (simulated trades, no real funds) |
| `requests`      | HTTP calls to the CoinGecko API |
| `itertools`     | Generating all currency pair permutations |

---

## How It Works

### 1. Fetch Exchange Rates
Live cross-rates are pulled from the CoinGecko `/simple/price` endpoint. Each coin is priced against all other 12 coins simultaneously, giving a 13×13 rate matrix.

### 2. Build a Directed Graph
Each currency is a **node**. Each exchange rate is a **directed weighted edge** (e.g., BTC → ETH with weight 0.059 means 1 BTC = 0.059 ETH). This is modeled as a `networkx.DiGraph`.

### 3. Find All Simple Paths
For every ordered pair of currencies `(c1, c2)`, `nx.all_simple_paths` finds every non-repeating route through the graph.

### 4. Compute Disequilibrium Factor
For each path:
- **Forward weight** = product of edge weights along the path (c1 → ... → c2)
- **Reverse weight** = product of edge weights along the reversed path (c2 → ... → c1)
- **Disequilibrium factor** = `forward_weight × reverse_weight`

If the factor is not exactly 1.0, a pricing inefficiency exists. The path with the **maximum factor** represents the best arbitrage candidate.

### 5. Report Results
The script prints:
- All paths and their weights for every currency pair
- The path with the **smallest** disequilibrium factor (most undervalued round trip)
- The path with the **greatest** disequilibrium factor (best arbitrage opportunity)

---

## Setup

### Running in Google Colab (Recommended)

1. Open [Google Colab](https://colab.research.google.com/)
2. Upload `FinalProjectCrypto.ipynb` or open it directly from GitHub via `File > Open notebook > GitHub`
3. Run all cells — no additional setup required, Colab pre-installs `requests` and `networkx`

### Running Locally

**Prerequisites:** Python 3.8+

```bash
git clone https://github.com/varvarley/Crypto-Project.git
cd Crypto-Project
pip install -r requirements.txt
jupyter notebook FinalProjectCrypto.ipynb
```

### Alpaca Paper Trading Setup

To enable live paper trade execution (planned feature), you will need a free Alpaca account:

1. Sign up at [alpaca.markets](https://alpaca.markets/)
2. Navigate to **Paper Trading** and generate an API key and secret
3. Set your credentials as environment variables or add them directly to the notebook:

```python
ALPACA_API_KEY = "your_key_here"
ALPACA_SECRET_KEY = "your_secret_here"
BASE_URL = "https://paper-api.alpaca.markets"
```

> The `BASE_URL` above points to Alpaca's **paper trading** endpoint. No real funds are ever used.

---

## Project Structure

```
Crypto-Project/
├── FinalProjectCrypto.ipynb   # Main notebook
├── requirements.txt           # Python dependencies
└── README.md                  # This file
```

---

## Known Limitations

- **ADA excluded from reverse paths** — CoinGecko does not return ADA as a base currency in its pricing data, so round trips involving ADA in reverse cannot be computed.
- **No fee modeling** — Exchange/transaction fees are not currently factored into the disequilibrium calculation. Real arbitrage profitability depends heavily on fees.
- **Snapshot pricing** — Prices are fetched once per run. In live trading, rates change in milliseconds; a detected opportunity may vanish before an order executes.
- **CoinGecko rate limits** — The free API tier has rate limits. High-frequency polling will result in 429 errors.
- **`all_simple_paths` performance** — With 13 nodes and all permutations, the number of paths grows factorially. This is manageable at 13 coins but will slow significantly if more coins are added.

---

## Planned Improvements

- [ ] Integrate Alpaca paper trade order submission for the best detected path
- [ ] Save exchange rate snapshots to CSV in a `/data` folder for historical analysis
- [ ] Use USD as a base/anchor currency for paths that don't share a direct trading pair
- [ ] Add transaction fee modeling to filter out non-profitable opportunities
- [ ] Implement continuous polling with configurable interval (using `time.sleep`)
- [ ] Optimize path search — consider Bellman-Ford for negative cycle detection (log-transformed weights) instead of brute-force permutations
- [ ] Add visualization of the currency graph using NetworkX drawing tools

---

## License

See [LICENSE](LICENSE) for details.
