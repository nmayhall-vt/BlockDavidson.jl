# BlockDavidson

[![Build Status](https://github.com/nmayhall-vt/BlockDavidson.jl/actions/workflows/CI.yml/badge.svg?branch=main)](https://github.com/nmayhall-vt/BlockDavidson.jl/actions/workflows/CI.yml?query=branch%3Amain)
[![Coverage](https://codecov.io/gh/nmayhall-vt/BlockDavidson.jl/branch/main/graph/badge.svg)](https://codecov.io/gh/nmayhall-vt/BlockDavidson.jl)

---
Simple code to solve for the eigenvectors/eigenvalues of a matrix or `LinearMap`. Preconditioner has not yet been implemented, so this is currently just a simple block lanczos code. 

## Example usage 

### Explicit matrix diagonalization

```julia
dav = Davidson(A)
e,v = eigs(dav)
```
to use the diagonal preconditioner specify as argument to eigs
```julia
e,v = eigs(dav, Adiag=diag(A))
```

### Matrix-free diagonalization 

We can also diagonalize an implicit matrix by defining a function `matvec` that algorithmically implments the action of `A` onto a vector or set of vectors. We can also specify several settings, including providing an initial guess `v_guess`:
```julia
using LinearMaps
function matvec(v)
    return A*v. # implement this however is appropriate
end


lmap = LinearMap(matvec)
dav = Davidson(lmap; max_iter=200, max_ss_vecs=8, tol=1e-6, nroots=6, v0=v_guess, lindep_thresh=1e-10)
e,v = eigs(dav)
```

Convergence issues:
- the implementation is still not as robust as it should be, with loss of orthogonality making it very difficult to converge tightly in some scenarios. If this is the case, try tightening `lindep_thresh`. Whenever the determinant of the subspace drops below this value, the algorithm is restarted. As such, if it's too loose, then the correction vectors aren't very accurate after orthogonalization. If it's too tight, then it restarts too frequently, slowing down convergence. 

