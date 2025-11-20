# CMA-ES & L-BFGS-B

A high-performance Rust optimization library featuring two complementary state-of-the-art algorithms: **CMA-ES** (Covariance Matrix Adaptation Evolution Strategy) and **L-BFGS-B** (Limited-memory Broyden-Fletcher-Goldfarb-Shanno with Box constraints).

Originally developed for [options surface calibration](https://github.com/gnsqd/surface) in quantitative finance, this library provides robust, production-ready implementations suitable for any optimization problem requiring either gradient-free evolutionary optimization or efficient quasi-Newton methods with bounds.

## Features

- **CMA-ES (Covariance Matrix Adaptation Evolution Strategy)**: Advanced evolutionary algorithm with adaptive covariance matrix
  - IPOP (Increasing Population) restart strategy
  - BIPOP (Bi-population) restart strategy  
  - Parallel evaluation support
  - Bounds handling via mirroring
  
- **L-BFGS-B**: Limited-memory Broyden-Fletcher-Goldfarb-Shanno algorithm with box constraints
  - Strong Wolfe line search
  - Adaptive finite differences
  - Numerically stable implementation
  - Memory-efficient for high-dimensional problems

## Installation

Add this to your `Cargo.toml`:

```toml
[dependencies]
cmaes-lbfgsb = "0.1.0"
```

## Quick Start

### CMA-ES Example

```rust
use cmaes_lbfgsb::cmaes::{canonical_cmaes_optimize, CmaesCanonicalConfig};

// Define objective function (minimize)
let objective = |x: &[f64]| {
    // Sphere function: f(x) = sum(x_i^2)
    x.iter().map(|&xi| xi * xi).sum::<f64>()
};

// Set bounds for each parameter
let bounds = vec![(-5.0, 5.0); 3]; // 3D problem, each param in [-5, 5]

// Configure CMA-ES
let config = CmaesCanonicalConfig {
    population_size: 12,
    max_generations: 100,
    seed: 42,
    verbosity: 1,
    ..Default::default()
};

// Optimize
let result = canonical_cmaes_optimize(objective, &bounds, config, None);

println!("Best solution: {:?}", result.best_solution);
println!("Generations used: {}", result.generations_used);
```

### L-BFGS-B Example

```rust
use cmaes_lbfgsb::lbfgsb_optimize::{lbfgsb_optimize, LbfgsbConfig};

// Define objective function
let objective = |x: &[f64]| {
    // Rosenbrock function
    let mut sum = 0.0;
    for i in 0..x.len()-1 {
        let a = 1.0 - x[i];
        let b = x[i+1] - x[i]*x[i];
        sum += a*a + 100.0*b*b;
    }
    sum
};

// Initial guess
let mut x = vec![-1.0, 1.0];

// Set bounds
let bounds = vec![(-2.0, 2.0), (-2.0, 2.0)];

// Configure L-BFGS-B
let config = Some(LbfgsbConfig {
    memory_size: 10,
    obj_tol: 1e-6,
    ..Default::default()
});

// Optimize
let result = lbfgsb_optimize(
    &mut x,
    &bounds,
    &objective,
    1000, // max iterations
    1e-5, // gradient tolerance
    None, // no callback
    config
);

match result {
    Ok((best_obj, best_params)) => {
        println!("Best objective: {}", best_obj);
        println!("Best parameters: {:?}", best_params);
    }
    Err(e) => println!("Optimization failed: {}", e),
}
```

## Algorithm Details

### CMA-ES (Covariance Matrix Adaptation Evolution Strategy)

**CMA-ES** is a state-of-the-art evolutionary algorithm that excels in **gradient-free optimization**. It's particularly powerful for complex, real-world optimization problems.

**What makes CMA-ES special:**
- **Self-adaptive**: Automatically adjusts its search distribution based on the optimization landscape
- **Covariance matrix learning**: Discovers correlations between parameters and adapts the search ellipsoid accordingly  
- **Robust to noise**: Works well even with noisy or discontinuous objective functions
- **Scale-invariant**: Performance doesn't depend on parameter scaling
- **Multimodal capability**: Can escape local minima through restart strategies

**Ideal for:**
- **Options surface calibration**: Handle complex, non-convex pricing model parameter spaces
- Non-differentiable functions (e.g., simulation-based objectives)
- Expensive function evaluations where gradients are costly/unavailable
- Multimodal landscapes with many local optima
- Problems with unknown parameter interactions

**Technical highlights:**
- IPOP (Increasing Population) and BIPOP (Bi-population) restart strategies
- Advanced evolution path tracking for faster convergence
- Parallel population evaluation
- Sophisticated termination criteria

### L-BFGS-B (Limited-memory BFGS with Box constraints)

**L-BFGS-B** is a quasi-Newton method that provides fast convergence for smooth optimization problems. It's the gold standard for **large-scale constrained optimization**.

**What makes L-BFGS-B special:**
- **Quasi-Newton efficiency**: Approximates second-order information without computing full Hessian
- **Memory efficient**: Stores only the last few gradient vectors (typically 3-20)
- **Box constraints**: Native support for parameter bounds without penalty methods
- **Superlinear convergence**: Very fast near the optimum for smooth functions

**Ideal for:**
- **Options model fitting**: Fast calibration when gradients are available/approximable
- Large-scale problems (1000+ parameters)
- Smooth objective functions (pricing models, likelihood functions)
- Problems requiring high precision solutions
- When function evaluations are expensive but gradients are cheap

**Technical highlights:**
- Strong Wolfe line search for robust step selection
- Adaptive finite difference gradients with numerical stability
- Active set method for bound constraint handling
- Kahan summation for numerical precision

## Configuration Options

Both algorithms provide extensive configuration options for fine-tuning performance. Here are the key parameters:

### CMA-ES Configuration

```rust
let config = CmaesCanonicalConfig {
    population_size: 50,           // Population size (0 = auto)
    max_generations: 1000,         // Maximum generations
    seed: 12345,                   // Random seed
    parallel_eval: true,           // Enable parallel evaluation
    verbosity: 1,                  // Output level (0-2)
    ipop_restarts: 3,              // IPOP restart count
    bipop_restarts: 0,             // BIPOP restart count (overrides IPOP)
    total_evals_budget: 50000,     // Total function evaluation budget
    use_subrun_budgeting: true,    // Advanced budget allocation
    // ... 15+ additional parameters available
    ..Default::default()
};
```

### L-BFGS-B Configuration

```rust
let config = LbfgsbConfig {
    memory_size: 10,               // L-BFGS memory size
    obj_tol: 1e-8,                 // Objective tolerance
    step_size_tol: 1e-9,           // Step size tolerance
    c1: 1e-4,                      // Armijo parameter
    c2: 0.9,                       // Curvature parameter
    fd_epsilon: 1e-8,              // Finite difference step
    max_line_search_iters: 20,     // Max line search iterations
    // ... additional parameters available
    ..Default::default()
};
```

### 📖 **Complete Configuration Documentation**

For comprehensive documentation of all parameters including:
- **Default values and typical ranges**
- **Trade-offs and performance implications** 
- **Guidelines for different problem types**
- **Mathematical background and theory**
- **Code examples for common scenarios**

See the full documentation in the source code:
- **CMA-ES**: [`CmaesCanonicalConfig`](src/cmaes.rs) - 25+ parameters with detailed explanations
- **L-BFGS-B**: [`LbfgsbConfig`](src/lbfgsb_optimize.rs) - 10+ parameters with comprehensive guidance

Or generate the documentation locally:
```bash
cargo doc --open
```

## Origins: Options Surface Calibration

This library was originally developed to solve a challenging problem in **quantitative finance**: calibrating complex options pricing models to market data.

You can see usage of this library in the calibration process of [our Surface lib](https://github.com/gnsqd/surface/).

**The Challenge:**
- **Volatility surfaces** are highly non-linear and often exhibit multiple local minima
- **Model parameters** (volatility, mean reversion, jump intensities) are tightly coupled
- **Market data** is noisy and may contain arbitrage-free constraints
- **Speed matters** in production trading systems

**Why Two Algorithms:**
- **CMA-ES** handles the initial "global search" phase, finding promising parameter regions despite noise and local minima
- **L-BFGS-B** provides fast "local refinement", polishing solutions to high precision with proper constraints

**Beyond Finance:**
While designed for options pricing, these implementations excel at any optimization problem with similar characteristics:
- Parameter estimation in complex models
- Machine learning hyperparameter tuning
- Engineering design optimization
- Scientific model fitting

## License

This project is licensed under the dual License - see the [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
