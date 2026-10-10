# TradePilot — AI-Assisted Paper Trading Terminal

TradePilot is a full-stack web-based paper-trading terminal for Indian equities. It combines live market data, portfolio management, technical-indicator analysis, an AI-style strategy assistant, and a simulated order-execution engine in one interface.

> **Important:** This is a paper-trading / educational system. It does not connect to a real broker or place real-money orders.

## What it does

In simple terms, TradePilot lets a user:

- Create an account and receive **₹1,00,000 virtual capital**.
- Search for Indian stocks and add them to a personal watchlist.
- View current quotes, price change, OHLC data, volume and historical charts.
- Track holdings, invested value, cash balance, net worth and unrealized P&L.
- Place simulated **Market, Limit and Stop-Loss (SL)** orders.
- Keep limit/SL orders open until their price condition is met.
- Cancel open orders.
- Use an AI-style assistant to analyze stocks using **RSI and 20-period SMA**.
- View the terminal on both desktop and mobile layouts.

## How the system works

```text
User
  │
  ├── Login / Register
  │
  ▼
Next.js Trading Terminal
  │
  ├── Dashboard / Portfolio
  ├── Watchlist & Stock Search
  ├── Stock Analysis + Charts
  ├── Trade Panel
  └── TradePilot AI
  │
  ▼
Next.js API Routes
  │
  ├── Quote / History / Search ──► Yahoo Finance market data
  ├── Authentication ────────────► NextAuth + bcrypt
  ├── Portfolio / Orders ────────► Prisma
  └── Trade Execution ───────────► Paper-trading engine
  │
  ▼
PostgreSQL Database
```

## Main features

### 1. Authentication

Users can register and log in using email/password credentials. Passwords are hashed with bcrypt and authenticated sessions use NextAuth JWT sessions.

New accounts start with ₹1,00,000 of virtual capital.

### 2. Live market data

The application uses Yahoo Finance endpoints through server-side API routes for quotes and historical prices. The quote service supports multiple symbols in one request and includes:

- Current price
- Previous close
- Percentage change
- Day high / low
- Open
- Volume
- Currency

The quote route also has a short in-memory cache, request deduplication and retry/timeout handling to reduce unnecessary upstream requests.

### 3. Market-aware terminal

The application contains logic for the Indian market schedule:

- NSE/BSE trading window: **9:15 AM – 3:30 PM IST**
- Monday–Friday trading days
- Configured 2026 market holidays

Trading is disabled when the application considers the market closed.

### 4. Portfolio management

The portfolio layer calculates values from holdings and current quotes:

- Cash balance
- Invested amount
- Current valuation
- Net worth
- Unrealized P&L
- Unrealized P&L percentage

Holdings are stored per user and per symbol with quantity and average cost.

### 5. Watchlist

Users can search stocks and add/remove symbols from a watchlist. Watchlist data is persisted in the database and also cached locally in the browser for a smoother UI experience.

### 6. Paper-trading engine

The trade API validates the market state, user, password, funds/holdings and current market price before recording a trade.

Supported order types:

| Order | Behaviour |
|---|---|
| **MARKET** | Immediately simulated at the fetched market price |
| **LIMIT** | Executes when BUY price ≤ limit or SELL price ≥ limit; otherwise remains open |
| **SL** | Creates a pending stop-loss order; the current implementation primarily handles SELL stop conditions |

Transactions update the user's virtual balance and portfolio atomically using Prisma transactions.

Approximate trading charges are also simulated using STT and other charges.

### 7. Open-order execution

Open orders are periodically checked against current prices. When a limit/stop condition is satisfied, the order is marked `EXECUTED` and the portfolio is updated.

Users can also cancel open orders. BUY-order funds reserved for an open order are returned when the order is cancelled.

### 8. Technical analysis assistant

The **TradePilot AI** interface is currently an application-side strategy engine rather than a remote LLM service.

It calculates:

- **RSI (14-period)**
- **SMA (20-period)**
- Price vs. SMA trend relationship

The resulting rule-based bias can produce signals such as:

- `BUY / ACCUMULATE`
- `BUY DIP`
- `HOLD`
- `SELL / WAIT`
- `SELL RALLY`

It can also answer basic portfolio, RSI, SMA and trading-navigation questions.

### 9. Market sentiment

The dashboard derives a simple bullish/bearish sentiment score from the NIFTY's percentage movement. This is a heuristic indicator, not a machine-learning sentiment model.

### 10. Responsive terminal

The project has separate desktop and mobile terminal experiences.

Desktop includes:

- Header/navigation
- Portfolio summary
- Sidebar
- Watchlist
- Open orders
- Holdings
- Analysis view
- Trade panel
- AI planner

Mobile includes a bottom navigation experience for Home, Trade, AI and Wallet/Assets.

## Technology stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Next.js 16 |
| Backend | Next.js App Router API routes |
| Authentication | NextAuth.js + bcryptjs |
| Database ORM | Prisma |
| Database | PostgreSQL |
| Market data | Yahoo Finance endpoints |
| Charts | Recharts / custom chart components |
| Language | JavaScript / JSX |
| Styling | CSS + component-level styling |

## Project structure

```text
Trading_Agent/
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   ├── auth/       # NextAuth credentials authentication
│   │   │   ├── history/    # Historical market data
│   │   │   ├── orders/     # Open-order checks and cancellation
│   │   │   ├── quote/      # Current market quotes
│   │   │   ├── register/   # User registration
│   │   │   ├── search/     # Stock search
│   │   │   ├── trade/      # Paper-trade execution
│   │   │   ├── user/       # Portfolio/user data
│   │   │   └── watchlist/  # Watchlist operations
│   │   ├── login/          # Login / registration UI
│   │   └── profile/        # User profile and trading stats
│   │
│   ├── components/
│   │   ├── tabs/           # Home, analysis and assets views
│   │   ├── AIChat.js       # Rule-based AI strategy assistant
│   │   ├── AnalysisTab.js  # Stock analysis UI
│   │   ├── TradePanel.js   # Order entry UI
│   │   ├── PriceContext.js  # Centralized quote polling/state
│   │   └── Providers.js    # Session, portfolio and price providers
│   │
│   ├── hooks/
│   │   ├── useLivePrices.js
│   │   └── usePortfolioData.js
│   │
│   └── lib/
│       ├── auth.js         # Authentication configuration
│       ├── indicators.js   # RSI / SMA / EMA calculations
│       └── market.js       # Indian market hours/holiday logic
│
├── prisma/
│   └── schema.prisma       # Database schema
│
├── __tests__/              # Order execution tests
├── package.json
└── README.md
```

## Database model

The Prisma schema contains five main entities:

- **User** — account, password hash, virtual balance and onboarding state.
- **Portfolio** — current holdings and average cost.
- **Trade** — market/limit/SL orders, execution status and charges.
- **Watchlist** — symbols saved by each user.
- **History** — portfolio performance history used for statistics.

Relationships are user-centric, and `(userId, symbol)` is unique for both portfolio and watchlist records.

## Getting started

### Prerequisites

- Node.js
- npm
- PostgreSQL database

### 1. Clone

```bash
git clone https://github.com/RonitS07/FXTrade.git
cd FXTrade
```

### 2. Install dependencies

```bash
npm install
```

### 3. Configure environment variables

Create a `.env` file:

```env
DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DATABASE"
NEXTAUTH_SECRET="your-long-random-secret"
NEXTAUTH_URL="http://localhost:3000"
```

Do not commit `.env` or other secrets to GitHub.

### 4. Set up Prisma

```bash
npx prisma generate
npx prisma migrate dev
```

### 5. Run the application

```bash
npm run dev
```

Open:

```text
http://localhost:3000
```

### Production build

```bash
npm run build
npm start
```

## Testing

The repository contains tests for core limit-order matching behaviour in `__tests__/tradeLogic.test.js`.

The tests cover:

- BUY limit execution
- BUY limit rejection when the price is too high
- SELL limit execution
- Multiple orders in a batch

The current `package.json` does not expose a dedicated test script, so the test setup may need to be wired into the project's preferred Jest configuration before running it as part of CI.

## Architecture notes

A useful design choice in the project is the centralized `PriceProvider`. Components subscribe to symbols, while a shared polling loop batches quote requests and distributes results through context. The quote API adds caching and request deduplication on top of that, reducing repeated upstream calls.

Portfolio calculations are derived from the stored holdings plus the latest available quotes, while trade and order changes are persisted through Prisma database transactions.

## Current limitations

This project is a **paper-trading simulator**, not a production brokerage system. In particular:

- Yahoo Finance is used as the market-data source rather than a broker/exchange feed.
- The live-price implementation uses polling rather than an exchange WebSocket.
- The AI assistant is rule-based and currently focuses on RSI/SMA rather than using an LLM or trained predictive model.
- Stop-loss handling is simplified.
- Trading charges are approximate.
- The sentiment score is heuristic.
- The order-execution worker is implemented through periodic application requests rather than a dedicated background job/queue.
- Production deployment would require stronger authorization, validation, observability, rate limiting, concurrency controls and a proper broker/exchange integration.

## Roadmap

Potential next steps:

- Add a real LLM agent for natural-language market research and planning.
- Introduce a proper agent/tool architecture for market data, portfolio analysis and order planning.
- Add technical indicators such as MACD, Bollinger Bands, ATR and VWAP.
- Build a dedicated background order-execution worker.
- Add WebSocket-based market data.
- Add backtesting and strategy evaluation.
- Add risk management and position-sizing logic.
- Add comprehensive integration/E2E tests and CI.
- Replace simulator execution with a broker sandbox API before considering live trading.

## Disclaimer

TradePilot is intended for education, experimentation and paper trading. Market data can be delayed or unavailable, simulated execution does not represent real exchange execution, and the strategy assistant is not financial advice. Do not use this application as the sole basis for real investment decisions.

## Author

**Ronit S07**

GitHub: https://github.com/RonitS07
