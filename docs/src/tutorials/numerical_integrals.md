# Numerically Solving Integrals

For basic multidimensional quadrature we can construct and solve a `IntegralProblem`:

``` @example integrate1
using Integrals
f(x,p) = sum(sin.(x))
prob = IntegralProblem(f,ones(2),3ones(2))
sol = solve(prob,HCubatureJL(),reltol=1e-3,abstol=1e-3)
sol.u
```

If we would like to parallelize the computation, we can use the batch interface
to compute multiple points at once. For example, here we do allocation-free
multithreading with Cubature.jl:

``` @example integrate2
using Integrals, IntegralsCubature, Base.Threads
function f(dx,x,p)
  Threads.@threads for i in 1:size(x,2)
    dx[i] = sum(sin.(@view(x[:,i])))
  end
end
prob = IntegralProblem(f,ones(2),3ones(2),batch=2)
sol = solve(prob,CubatureJLh(),reltol=1e-3,abstol=1e-3)
sol.u
```

If we would like to compare the results against Cuba.jl's `Cuhre` method, then
the change is a one-argument change:

``` @example integrate2
using IntegralsCuba
sol = solve(prob,CubaCuhre(),reltol=1e-3,abstol=1e-3)
sol.u
```

Integrals.jl also has specific solvers for integrals in a single dimension, such as `QuadGKLJ`.
For example we can create our own sine function by integrating the cosine function from 0 to x.

``` @example integrate3
using Integrals
my_sin(x) = solve(IntegralProblem((x,p)->cos(x), 0.0, x), QuadGKJL()).u
x = 0:0.1:2*pi
@. my_sin(x) ≈ sin(x)
```