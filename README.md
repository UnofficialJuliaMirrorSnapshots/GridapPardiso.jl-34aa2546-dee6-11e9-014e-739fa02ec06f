# GridapPardiso

[![Stable](https://img.shields.io/badge/docs-stable-blue.svg)](https://gridap.github.io/GridapPardiso.jl/stable)
[![Dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://gridap.github.io/GridapPardiso.jl/dev)
[![Build Status](https://travis-ci.com/gridap/GridapPardiso.jl.svg?branch=master)](https://travis-ci.com/gridap/GridapPardiso.jl)
[![Codecov](https://codecov.io/gh/gridap/GridapPardiso.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/gridap/GridapPardiso.jl)

[Gridap](https://github.com/gridap/Gridap.jl) (Grid-based approximation of partial differential equations in Julia) plugin to use the [Intel Pardiso MKL direct sparse solver](https://software.intel.com/en-us/mkl-developer-reference-fortran-intel-mkl-pardiso-parallel-direct-sparse-solver-interface).

## Basic Usage

```julia
using Gridap
using GridapPardiso
A = sparse([1,2,3,4,5],[1,2,3,4,5],[1.0,2.0,3.0,4.0,5.0])
b = ones(A.n)
x = similar(b)
msglvl = 1
ps = PardisoSolver(GridapPardiso.MTYPE_REAL_NON_SYMMETRIC, new_iparm(A), msglvl)
ss = symbolic_setup(ps, A)
ns = numerical_setup(ss, A)
solve!(x, ns, b)
```

## Usage in a Finite Element computation

```julia
using Gridap
using GridapPardiso

# Define the FE problem
# -Δu = x*y in (0,1)^3, u = 0 on the boundary.

model = CartesianDiscreteModel(
  domain=(0,1,0,1,0,1), partition=(10,10,10))

fespace = FESpace(
  reffe=:Lagrangian, order=1, valuetype=Float64,
  conformity=:H1, model=model, diritags="boundary")

V = TestFESpace(fespace)
U = TrialFESpace(fespace,0.0)

trian = Triangulation(model)
quad = CellQuadrature(trian,degree=2)

t_Ω = AffineFETerm(
  (v,u) -> inner(∇(v),∇(u)),
  (v) -> inner(v, (x) -> x[1]*x[2] ),
  trian, quad)

op = LinearFEOperator(V,U,t_Ω)

# Use Pardiso to solve the problem

ls = PardisoSolver() # Pardiso with default values
solver = LinearFESolver(ls)
uh = solve(solver,op)

```

## Installation

**GridPardiso** itself is installed when you add and use it into another project.

Please, ensure that your system fulfill the requirements.

To include into your project form Julia REPL, use the following commands:

```
pkg> add GridapPardiso
julia> using GridapPardiso
```

If, for any reason, you need to manually build the project, write down the following commands in Julia REPL:
```
pkg> add GridapPardiso
pkg> build GridPardiso
julia> using GridapPardiso
```

### Requirements

- `MKLROOT` environment variable must be set pointing to the MKL installation root directory.
- `gcc` compiler must be installed and accessible in your system.
- `OpenMP` library (`libgomp1` in linux OS) must be installed and accesible in your system.

**GridapPardiso** relies on [Intel Pardiso MKL direct sparse solver](https://software.intel.com/en-us/mkl-developer-reference-fortran-intel-mkl-pardiso-parallel-direct-sparse-solver-interface). So, you need it in order to be able to use **GridPardiso**.

[Intel MKL](https://software.intel.com/en-us/mkl) includes `/opt/intel/mkl/bin/mklvars.sh` script to setup the correct environment to use it. We strongly recommend to run this script as follows:

```
$ source /opt/intel/mkl/bin/mklvars.sh intel64
```

This script setup the `MKLROOT` environment variable required by **GridapPardiso** to build it correctly.

In addition, please make sure that [OpenMP](https://www.openmp.org/) is installed. We use the default distribution package that is installed together with `GCC` compilers in Linux environments.

To fullfil this requirements, in a debian-based OS, we recommend install the following packages:

```
$ apt-get update
$ apt-get install -y gcc ligomp1
```

## Notes

Currently **GridapPardiso** only works with `SparseMatrixCSC` (from [SparseArrays](https://docs.julialang.org/en/v1/stdlib/SparseArrays/)), `SparseMatrixCSR` and `SymSparseMatrixCSR` (from [SparseMatricesCSR](https://gridap.github.io/SparseMatricesCSR.jl/stable/)) matrices. Any other `AbstractMatrix{Float64}` matrix is converted to `SparseMatrixCSC{Float64,Integer}`.
