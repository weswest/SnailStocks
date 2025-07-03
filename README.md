# Stock Market Game: Reinforcement Learning vs Deterministic Strategies

## Introduction and Problem Statement

This project analyzes optimal strategies for an unusual stock market game with the following rules:
- **Starting capital**: $300,000
- **6 stocks**: All starting at $100/share
- **7-day trading window**: Buy-only (no selling allowed)
- **Perfect one-day foresight**: You know with certainty whether each stock will be Bullish (+5-8%), Upward (+2-5%), Uncertain (-2% to +2%), Downward (-2% to -5%), or Bearish (-5% to -8%) tomorrow
- **Constraint**: Can only buy in 100-share lots

The twist that makes this game counterintuitive: despite perfect foresight, holding stocks has negative expected value due to volatility drag. When a stock goes up 20% then down 20%, you end at 96% of starting value. This creates an interesting optimization problem where buying too early is mathematically suboptimal.

## TL;DR: What Worked Best

After testing reinforcement learning and seven deterministic strategies, two approaches stood out:

1. **"Buy First Bullish"** (Highest EV): Wait for bullish stocks, then invest everything
   - Mean return: $19,888
   - Win rate: 75.5%
   - High volatility (σ = $30,500)

2. **"Peanut Butter Spread"** (Best risk-adjusted): Invest 1/n of remaining cash with n days left
   - Mean return: $17,564  
   - Win rate: 95.5%
   - Low volatility (σ = $10,420)
   - Sharpe ratio: 1.69 vs 0.65

## Reinforcement Learning: What Worked and What Failed

### Initial Approach
I implemented a Deep Q-Network (DQN) with several advanced features:
- Dueling architecture (separate value/advantage streams)
- N-step returns for better credit assignment
- Prioritized experience replay
- Custom reward shaping based on expected returns

### Why RL Failed

1. **The "Never Buy" Local Optimum**: Since the expected value of holding stocks is negative, the agent often converged to never buying anything—a safe but useless strategy.

2. **Sparse Reward Signal**: The agent only learns whether it made good decisions after 7 days, making credit assignment extremely difficult.

3. **Simple Optimal Strategy**: The mathematically optimal strategies (threshold-based buying after day 3-4) are straightforward enough that random exploration rarely discovers them.

4. **Implementation Complexity**: Multi-action spaces (6 stocks × buy/don't buy) combined with PyTorch tensor operations created numerous edge cases and debugging challenges.

### Technical Lessons Learned
- Start with simple baselines before adding complexity
- The game's structure (perfect information, simple optimal policy) makes it poorly suited for RL
- Sometimes domain-specific heuristics outperform general learning algorithms

## Deterministic Policies: Tradeoffs

I tested seven hand-crafted strategies across 1,000 Monte Carlo simulations each:

### Strategies Tested
1. **Buy First Bullish**: All-in on first bullish opportunity
2. **Threshold Day 4+**: Wait until day 4, then all-in
3. **Threshold Day 5+**: Wait until day 5, then all-in
4. **Simple Fractional**: Invest 1/n of remaining cash with n days left
5. **Fractional with Cutoff**: Fractional starting day 4
6. **Fractional + 50% Multi**: Fractional with 50% bonus for multiple bullish stocks
7. **Cutoff + 50% Multi**: Combines strategies 5 and 6

### Results Analysis

The key insight is the classic risk-return tradeoff:
- **Aggressive strategies** (buy early, all-in) maximize expected value but with high volatility
- **Conservative strategies** (fractional deployment) sacrifice ~10% expected return for ~66% volatility reduction

From behavioral economics, we know losses hurt roughly twice as much as equivalent gains feel good. This makes the fractional "peanut butter" strategy psychologically optimal for most players—you win 19 out of 20 weeks instead of experiencing wild swings.

### Implementation
The full implementation includes:
- Game engine with accurate probability distributions
- Strategy framework for testing different policies
- Monte Carlo simulator with confidence intervals
- Comprehensive visualization tools

See the included Jupyter notebooks for complete code and analysis.

## Detailed Results

### Strategy Performance (1,000 episodes each)

| Strategy | Mean Return | Std Dev | Sharpe Ratio | Win Rate | 95% CI Lower | 95% CI Upper |
|----------|-------------|---------|--------------|----------|--------------|--------------|
| Buy First Bullish | $19,888.43 | $30,500.51 | 0.652 | 75.50% | $17,994.78 | $21,782.07 |
| Threshold Day 4+ | $18,750.80 | $20,543.41 | 0.913 | 81.80% | $17,475.35 | $20,026.25 |
| Threshold Day 5+ | $17,915.04 | $16,656.78 | 1.080 | 85.00% | $16,880.89 | $18,949.19 |
| Simple Fractional | $17,564.05 | $10,420.66 | 1.690 | 95.50% | $16,917.08 | $18,211.03 |
| Fractional with Cutoff | $16,704.99 | $9,112.80 | 1.830 | 96.40% | $16,139.21 | $17,270.76 |
| Fractional + 50% Multi | $16,577.66 | $10,548.87 | 1.570 | 95.50% | $15,922.73 | $17,232.60 |
| Cutoff + 50% Multi | $15,681.99 | $9,090.36 | 1.730 | 96.00% | $15,117.61 | $16,246.37 |
