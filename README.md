# ISO-PERP Trading Protocol Specification
*Zero Bad-Debt Perpetual Swap with Deterministic Barrier Liquidations*

By: **Matthew Olsen** ([LinkedIn](https://www.linkedin.com/in/matthew-olsen-193203153/) • [GitHub](https://github.com/olsenmatthew) • [X](https://x.com/OlsenMatthew99))

---

## Executive Summary

**The Problem with Standard PERPs:**
Standard perpetual swaps suffer from fundamental risk management flaws: unpredictable liquidation prices due to slippage, bad debt risk requiring insurance funds, forced margin calls during volatility, and auto-deleveraging that closes profitable positions. These issues create operational complexity and capital inefficiency for quantitative traders.

**ISO-PERP Solution:**
ISO-PERP eliminates these problems through deterministic barrier liquidations. Traders know exact liquidation prices upfront, face zero counterparty credit risk, never receive margin calls, and avoid auto-deleveraging. The amount committed per order equals maximum loss. The system supports ultra-high leverage (up to reciprocal of relative tick size) with advanced risk controls for quantitative strategies.

---

## Product Specification

- **Ticker:** `BTC-USDC:ISOP`  
- **Reference:** Spot BTC aggregated index from major venues  
- **Duration:** Perpetual  
- **Payoff:** $\mathrm{PnL} = (P_{\rm mark} - P_0)\times Q\times L$
- **Liquidation:** Barrier knockout at pre-computed $P_{\rm liq}$ (no margin calls)
- **Funding:** Optional rate $f$ with barrier protection
- **Collateral:** Fixed amount per order equals maximum loss

---

## Core Mechanics

### Leverage and Tick-Size Constraints

- **Tick Size ($\Delta_{\rm tick}$)**: Minimum price increment (e.g., 0.0001 USD)  
- **Relative Tick:** $\mathrm{rel\_tick} = \frac{\Delta_{\rm tick}}{P}$
- **Maximum Leverage:** $L_{\max} = \frac{P}{\Delta_{\rm tick}}$
  
**Example (BTC-USDC:ISOP):** At $P=100{,}000$ USD, $\Delta_{\rm tick}=0.0001$ ⇒ $L_{\max}=10^9$

### Barrier Liquidation Pricing

**Long positions:** $P_{\rm liq} = P_0 \times \left(1 - \frac{1}{L}\right)$

**Short positions:** $P_{\rm liq} = P_0 \times \left(1 + \frac{1}{L}\right)$

**No Margin Calls:** The amount committed per order is the maximum possible loss. Liquidation occurs only at the predetermined barrier price.

### Matching Engine Architecture

1. **Leverage Bucket Sharding:** Orders segregated into discrete buckets (1×, 2×, 5×, 10×, 20×, 50×, 100×, 200×, 500×, 1,000×, 2,000×, 5,000×, 10,000×) where bucket $n$ contains orders with $L_n \leq L < L_{n+1}$.

2. **Counterparty Risk Filtering:** Orders specify `cp_min` and `cp_max` for acceptable counterparty leverage ranges.

3. **Priority Routing:** `high_first`/`low_first` flags determine liquidity access order.

4. **Barrier Liquidation Protocol:** On multi-tick price jumps, system identifies positions with $P_{\rm liq}$ between old and new mark price, liquidates both position holders and counterparties at barrier price, caps profits at liquidation barrier.

5. **Counterparty Tracking:** Bidirectional relationship management ensures liquidation events affect all connected parties simultaneously.

### Funding Rate Barriers

Optional funding with built-in protection:

- **Funding Rate:** $f$ applied per interval
- **Funding Barrier:** $f_{\rm bar} = \frac{1}{L}$
- **Pre-liquidation Rule:** If $f_{\rm scheduled} \geq f_{\rm bar}$, positions liquidate at $P_{\rm liq}$ before funding application

### Price Discovery Methodology

Three-tier price architecture prevents manipulation:

- **Index Price:** External reference from major venues
- **Mark Price:** Official price for valuations and liquidations  
- **Last Price:** Recent execution price (not used for liquidations)

**Mark Price Formula:** $\text{Mark Price} = 0.8 \times \text{Index Price} + 0.2 \times \text{Market Price}$

### Orderbook Fragmentation Effects

**Liquidity Characteristics:**
- Independent price discovery per leverage bucket
- Risk-adjusted pricing reflected in spread differentials
- Arbitrage constraints due to counterparty leverage filtering

**Cross-Bucket Dynamics:**
- Unified mark price across all buckets
- Volume-weighted trade aggregation for market price component
- Systematic arbitrage opportunities between buckets

### Position Exit Mechanics

**Unwinding Process:**
1. Opposite-direction orders to exit positions
2. Counterparty satisfaction chain maintains risk preferences
3. Four-party transactions (exiting trader, original counterparty, new trader, new counterparty)

**Cross-Bucket Risk Transfer:**
- Position exits create leverage bucket migration
- Example: 50× leverage positions transferred to 20× leverage counterparties

---

## Risk Management

| Risk Vector | Mitigation |
|-------------|------------|
| **Price Gaps** | Barrier capping at $P_{\rm liq}$ ensures full collateralization |
| **Funding Spikes** | Pre-liquidation at funding barrier |
| **Liquidity Manipulation** | Rate-limited auto-replace, per-bucket order caps |
| **Counterparty Risk** | Zero bad-debt guarantee through deterministic barriers |

---

## Pros & Cons Comparison

| Instrument | Pros | Cons | Best Use Case |
|------------|------|------|---------------|
| **ISO-PERP** | • Zero bad-debt risk<br>• No margin calls<br>• Deterministic liquidation prices<br>• Ultra-high leverage<br>• No time decay<br>• No counterparty risk | • Winners capped at barrier price<br>• Liquidity fragmented across buckets<br>• More complex than standard perps<br>• Limited to barrier knockout behavior | • High-leverage speculation<br>• Tail risk hedging<br>• Precise risk budgeting<br>• Algorithmic strategies<br>• Volatile market conditions |
| **Standard PERPs** | • Deep unified liquidity<br>• Simple mechanics<br>• Established infrastructure<br>• No knockout barriers<br>• Unlimited profit potential | • Bad debt risk (insurance fund)<br>• Frequent margin calls<br>• Auto-deleveraging risk<br>• Unpredictable liquidation slippage<br>• Maintenance margin complexity | • Large position sizes<br>• Long-term directional bets<br>• When maximum liquidity needed<br>• Traditional risk management |
| **Options** | • Defined risk (premium)<br>• Nonlinear payoffs<br>• Volatility exposure<br>• Multiple strategies<br>• No liquidation risk | • Up-front premium cost<br>• Time decay (theta)<br>• Limited duration<br>• Complex pricing<br>• Lower leverage | • Volatility trading<br>• Insurance/hedging<br>• Income strategies<br>• Complex payoff structures<br>• When theta acceptable |
| **Futures** | • Physical/cash settlement<br>• Standardized contracts<br>• Deep institutional liquidity<br>• Regulatory clarity<br>• Price discovery | • Expiration dates<br>• Rollover costs<br>• Margin requirements<br>• Contango/backwardation<br>• Less flexible leverage | • Institutional hedging<br>• Physical delivery needs<br>• Regulatory compliance<br>• Traditional derivatives |

---

## Comparative Analysis

| Feature | ISO-PERP | Standard Perps | Options |
|---------|----------|----------------|---------|
| **Premium** | Zero | Zero | Required |
| **Duration** | Infinite | Infinite | Finite + theta decay |
| **Payoff** | Linear to barrier | Linear to liquidation | Nonlinear convex |
| **Knockout** | Deterministic at $P_{\rm liq}$ | Insurance fund + ADL | Separate barrier products |
| **Funding** | Optional with barriers | Mandatory unbounded | N/A |
| **Leverage** | To tick/funding limits | To maintenance margin | Implicit via premium |
| **Bad Debt Risk** | Zero (guaranteed) | Exists (insurance fund) | Zero (premium paid) |
| **Margin Calls** | Never | Frequent during volatility | N/A |

---

## Quantitative Applications

**Statistical Arbitrage:**
- Basis trade ISO-PERP vs standard perps when divergence exceeds fees
- Cross-venue arbitrage using deterministic liquidation characteristics
- Exploit price discrepancies between leverage buckets

**Market Making:**
- Multi-bucket liquidity provision with guaranteed collateralization
- Spread capture with zero bad-debt guarantees
- Strategic counterparty leverage filtering

**High-Frequency Trading:**
- Auto-replace for continuous presence without client round-trips
- Microstructure strategies exploiting bucket fragmentation

**Risk Management:**
- High-leverage barrier knockouts for tail risk hedging
- Cross-asset hedging with zero counterparty credit risk
- Portfolio insurance using deterministic exit prices

**Volatility Trading:**
- Pure volatility exposure without funding rate uncertainty
- Barrier-based volatility strategies with known exit levels

---

## Order Parameters

**Required Fields:**
- `instrument`: Trading pair (e.g., "BTC-USDC:ISOP")
- `side`: BUY or SELL
- `price`: Limit price
- `quantity`: Order size
- `leverage`: Position leverage (determines bucket and liquidation price)
- `cp_min`: Minimum acceptable counterparty leverage
- `cp_max`: Maximum acceptable counterparty leverage  
- `auto_replace`: Continuous re-insertion on fill without liquidation
- `high_first`: Prioritize high-leverage counterparties

---

## Key Advantages for Quants

**Deterministic Risk:**
- Exact liquidation prices known at order placement
- No slippage risk on barrier events
- Zero bad-debt guarantees eliminate credit risk
- No margin calls: amount committed per order equals maximum loss

**Simplified Risk Management:**
- No maintenance margin requirements
- No forced position sizing due to margin calls
- Capital efficiency through precise loss calculation
- Eliminates margin call timing risk in volatile markets

**Advanced Order Routing:**
- Leverage bucket selection for counterparty risk control
- Priority flags for optimal liquidity access
- Auto-replace for algorithmic strategies

**Market Structure Arbitrage:**
- Cross-bucket price discrepancies
- Basis trading opportunities vs standard perps
- Funding rate arbitrage with barrier protection

**Portfolio Applications:**
- Precise risk budgeting with known max loss
- Correlation trading with zero funding noise
- Tail risk hedging with guaranteed exits
