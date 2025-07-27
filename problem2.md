# Mathematical Framework for Optimal Execution to Minimize Temporary Impact

## Problem Statement

We need to determine the optimal execution quantities $x_i$ at each time period $t_i$ for $i = 1, 2, \ldots, N$ (where $N = 390$ minutes in a trading day), such that:
1. We execute exactly $S$ total shares: $\sum_{i=1}^{N} x_i = S$
2. All execution quantities are non-negative: $x_i \geq 0$ for all $i$
3. We minimize the total temporary impact cost: $\min \sum_{i=1}^{N} g_{t_i}(x_i)$

where $g_{t_i}(x_i)$ represents the temporary impact function at time $t_i$ for executing $x_i$ shares.

## Modeling the Temporary Impact Function

Based on empirical evidence and market microstructure theory, we model the temporary impact function as a power-law:

$$g_{t_i}(x_i) = \eta_{t_i} \cdot x_i^\delta$$

where:
- $\eta_{t_i}$ is the time-varying impact coefficient at time $t_i$
- $\delta$ is the power-law exponent (typically $\delta \approx 0.5$ based on the square-root law of market impact)
- $x_i$ is the number of shares executed at time $t_i$

The coefficient $\eta_{t_i}$ captures the time-varying liquidity conditions throughout the trading day, with higher values indicating lower liquidity (and thus higher impact per share).

## Mathematical Formulation of the Optimization Problem

Our optimization problem can be formulated as:

$$\min_{\mathbf{x}} \sum_{i=1}^{N} \eta_{t_i} \cdot x_i^\delta$$

subject to:

$$\sum_{i=1}^{N} x_i = S$$

$$x_i \geq 0 \quad \forall i = 1, 2, \ldots, N$$

This is a constrained convex optimization problem when $\delta \geq 1$. However, with $\delta = 0.5$, the objective function is concave, making the problem more challenging.

## Analytical Solution via Lagrange Multipliers

For the special case where $\delta > 0$, we can derive an analytical solution using Lagrange multipliers.

$$x_i = \frac{S \cdot \eta_{t_i}^2}{\sum_{j=1}^{N} \eta_{t_j}^2}$$

This is the optimal allocation when $\delta = 0.5$, assuming all $x_i \geq 0$ (which is satisfied since $\eta_{t_i} > 0$).


## Numerical Solution: Projected Gradient Descent Algorithm

For general values of $\delta$ and to handle additional constraints or more complex impact models, we can use projected gradient descent:

1. **Initialize**: Start with a feasible solution, e.g., $x_i^{(0)} = \frac{S}{N}$ for all $i$

2. **Gradient Step**: At iteration $k$, compute the gradient of the objective function:
   $$\nabla f(\mathbf{x}^{(k)})_i = \delta \cdot \eta_{t_i} \cdot (x_i^{(k)})^{\delta-1}$$

3. **Update**: Take a step in the negative gradient direction:
   $$\mathbf{y}^{(k+1)} = \mathbf{x}^{(k)} - \alpha_k \nabla f(\mathbf{x}^{(k)})$$
   where $\alpha_k$ is the step size at iteration $k$

4. **Project**: Project $\mathbf{y}^{(k+1)}$ onto the constraint set $\{\mathbf{x} : \sum_i x_i = S, x_i \geq 0\}$:
   $$\mathbf{x}^{(k+1)} = \Pi_{\mathcal{C}}(\mathbf{y}^{(k+1)})$$

5. **Check Convergence**: If $\|\mathbf{x}^{(k+1)} - \mathbf{x}^{(k)}\| < \epsilon$ or maximum iterations reached, stop; otherwise, return to step 2

The projection onto the simplex constraint $\{\mathbf{x} : \sum_i x_i = S, x_i \geq 0\}$ can be efficiently computed using algorithms such as the one proposed by Wang and Carreira-Perpiñán (2013).

## Practical Algorithm Implementation

Here's a step-by-step algorithm to determine the optimal execution quantities $x_i$ at each time $t_i$:

1. **Data Collection and Preprocessing**:
   - Collect historical order book data for the target security
   - Aggregate data into minute-level intervals
   - Calculate relevant features (spread, depth, etc.)

2. **Impact Model Estimation**:
   - For each time period $t_i$, estimate the impact coefficient $\eta_{t_i}$ by:
     - Fitting a power-law model to historical trade data
     - Using log-log regression to estimate $\delta$ and $\eta_{t_i}$
     - If data is sparse, use smoothing techniques or time-of-day patterns

3. **Optimization**:
   - If using $\delta = 0.5$ (square-root impact), apply the analytical solution:
     $$x_i = \frac{S \cdot \eta_{t_i}^2}{\sum_{j=1}^{N} \eta_{t_j}^2}$$
   - For other values of $\delta$ or more complex models, use projected gradient descent as described above

4. **Execution Schedule Generation**:
   - Round the continuous solution to practical execution quantities if necessary
   - Ensure the constraint $\sum_i x_i = S$ is still satisfied after rounding
   - Generate a minute-by-minute execution schedule

5. **Adaptation** (optional):
   - As the day progresses, update impact estimates based on observed market conditions
   - Re-optimize the remaining execution schedule accordingly

## Conclusion

This mathematical framework provides a rigorous approach to determining optimal execution quantities $x_i$ at each time $t_i$ while minimizing total temporary impact. The key insights are:

1. The power-law model $g_{t_i}(x_i) = \eta_{t_i} \cdot x_i^\delta$ with $\delta \approx 0.5$ accurately captures the temporary impact based on empirical evidence

2. For $\delta = 0.5$, we have a closed-form solution that allocates more shares to periods with lower impact coefficients

3. For general cases, projected gradient descent provides a robust numerical solution

4. The framework can be extended to incorporate risk aversion, permanent impact, and other practical considerations

This approach balances theoretical rigor with practical implementability, providing a solid foundation for optimal execution strategies in financial markets.
