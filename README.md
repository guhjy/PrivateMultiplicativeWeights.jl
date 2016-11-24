# PrivateMultiplicativeWeights.jl

A simple and practical algorithm for differentially private data release.

MIT Licensed. See `LICENSE.md`.

## Installation

Open a Julia prompt and call: `Pkg.clone("https://github.com/mrtzh/PrivateMultiplicativeWeights.jl.git")`

## Main Features

* Differentially private synthetic data preserving lower order marginals of an input data set
* Optimized in-memory implementation for small number of data attributes
* Scalable heuristic for large number of data attributes
* Easy-to-use interfaces for custom query sets and data representations

## Example
For the sake of illustration, we create a random data set with hidden correlations. Columns correspond to data points.
```
d, n = 20, 1000
data_matrix = rand(0:1, d ,n)
data_matrix[3, :] = data_matrix[1, :] .* data_matrix[2, :]
```

We can run MWEM to produce synthetic data accurate for 1st, 2nd, 3rd order marginals of the source data.
```
using PrivateMultiplicativeWeights
mw = mwem(Parities(d, 3), Tabular(data_matrix))
```
This will convert the data to its explicit histogram representation of size 2^d
and may not be useful when d is large. See section on factored histograms
for an alternative when the dimension d is large.

### Convert histograms to matrices

We can convert synthetic data in histogram representation to a tabular 
(matrix) representation.
```
table = Tabular(mw.synthetic, n)
```

### Compute error of approximation
Compute error achieved by MWEM:
```
maximum_error(mw), mean_squared_error(mw)
```
Note that these statistics are *not* differentially private.

## Parameters

We can set various parameters:
```
mw = mwem(Parities(d, 3),
          Tabular(data_matrix),
          epsilon=1.0,
          iterations=10,
          repetitions=10,
          verbose=false,
          noisy_init=false)
```
Parameters:

| Name | Default | Description |
| ---- | ------- | ----------- |
| `epsilon` | `1.0` | Privacy parameter per iteration. Each iteration of MWEM is `epsilon`-differentially private. Total privacy guarantees follow via composition theorems. |
| `iterations` | `10` | Number of iterations of MWEM. Each iteration corresponds to selecting one query via the exponential mechanism, evaluating the query on the data, and updating the internal state. |
| `repetitions`| `10` | Number of times MWEM cycles through previously measured queries per iteration. This has no additional privacy cost. |
| `noisy_init` | `false` | Histogram initalization through noise addition. When `noisy_init` is set to false, the initialization is uniform. |
| `verbose` | `false` | print timing and error statistics per iteration (information is not differentially private)


## Custom query sets

There are two ways to define custom query sets.

### Query matrices

You can define custom query workloads by using `HistogramQueries(query_matrix)`
instead of `Parities(d, 3)`. Here `query matrix` is an `N x k` matrix specifying
the query set in its Histogram representation, `N` is the histogram length and
`k` is the `k` is the number of queries.

### Custom query types

To build query sets with your own implicit representations, sub-type
`Query` and `Queries`. Implement the functions specified in `src/interface.jl`.

See `src/parities.jl` for an example.

## Factored histograms
When the histogram representation is too large, try using factored histograms.
Factored histograms maintain a product distribution over clusters of attributes
of the data. Each component is represented using a single histogram. Components
are merged as it becomes necessary. This often allows to scale up MWEM by orders
of magnitude.  
```
d, n = 100, 1000
data_matrix = rand(0:1, d, n)
data_matrix[3, :] = data_matrix[1, :] .* data_matrix[2, :]
mw = mwem(FactorParities(d, 3), Tabular(data_matrix))
```

Also see `examples.jl`.

## Citing this package

The MWEM algorithm was presented in the following paper:
```
@inproceedings{HLM12,
	author = "Moritz Hardt and Katrina Ligett and Frank McSherry",
	title = "A simple and practical algorithm for differentially-private data release",
    title = {Proc.\ 26th Neural Information Processing Systems (NIPS)},
    booktitle = {Proc.\ 26th Neural Information Processing Systems (NIPS)},
    year = {2012},
}
```
