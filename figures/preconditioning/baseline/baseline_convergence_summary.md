# 1bpos baseline convergence summary

The 1bpos reconstruction is effectively converged by the end of the 1000-epoch
run:

| quantity | value |
|---|---:|
| Final objective gap / total decrease | `8.8 × 10⁻⁹` |
| Final PET iterate velocity | `7.3 × 10⁻⁸` per epoch |
| Relaxed step size | `0.22` |
| Epoch reaching 99.9% of objective decrease | `45` |

The objective plateau is therefore genuine convergence rather than step-size
starvation: the iterate is barely moving even though the relaxed step remains
substantial.
