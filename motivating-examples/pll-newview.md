# example terse code for PLL newview* functions

## terse code

    .sub newview(tip_case,
                 vl,
                 vr,
                 le, ri
                 num_states,
                 extEV,
                 fast_scaling,
                 wgt,
                 ex3,
                 sum_scale,
                 .out v)
        for i in .range(num_states):
            x1 = .sum(num_states, vl[_i]*le[i][_i])
            x2 = .sum(num_states, vr[_i]*ri[i][_i]);
            .on_each(v[:num_states], = x1*x2*extEV[i][_i])
        if tip_case in [PLL_TIP_INNER, PLL_INNER_INNER]:
            scale = .all(v[:num_states], .lazy(a): PLL_MINUSMINLIKELIHOOD < a < PLL_MINLIKELIHOOD)
            if scale:
                .on_each(v[:num_states], *= PLL_TWOTOTHE256)
                if fast_scaling:
                    sum_scale += wgt
                else:
                    ex3++

    .sub newviewi: .iter_transform({n, .help='site index'},
                                   {newview,
                                    _noiter=(tip_case, num_states, extEV, fast_scaling, sum_scale),
                                    _local=sum_scale})
        .pre:
            if fast_scaling:
              sum_scale = 0

## interpretation
The code above could be used to generate the 9 impls of see newview* (in 
PLL's 
[newviewGenericSpecial.c](https://www.assembla.com/code/phylogenetic-likelihood-library/git/nodes/master/src/newviewGenericSpecial.c))
if you fed the interpreter some description of how the data should be laid out in the argument.

The interpreter should be able to infer from newview the following:
  *  PLL_TIP_INNER, PLL_INNER_INNER: compile-time constants with == semantics
  * PLL_MINUSMINLIKELIHOOD, PLL_MINLIKELIHOOD, PLL_TWOTOTHE256: compile time numbers, PLL_MINUSMINLIKELIHOOD < PLL_MINLIKELIHOOD
  * vl and vr: read-only, integer-indexable collections of numbers of used_size >= num_states
  * `tip_case`: read-only scalar. => bool (in [PLL_TIP_INNER, PLL_INNER_INNER])
  * `le`, `ri`, `extEV`: read-only, 2D-integer -indexable collections of numbers used_size = num_states by num_states
  * `x1` and `x2`: numbers local to first for-loop
  *  `v` output (explicitly stated) overwritten.) integer-indexable collections of numbers of used_size >= num_states if tip_case in [PLL_TIP_INNER, PLL_INNER_INNER]
  * `fast_scaling`: read-only boolean
  * `wgt`: read-only. rhs addition. if bool(fast_scaling) (and runtime bool(scale))
  * `sum_scale`: incr by wgt, output. augmented. only used if bool(fast_scaling) (and runtime bool(scale))
  * `ex3`: Integral. output. augmented if bool(fast_scaling) (and runtime bool(scale))
  * `scale`: local bool

newviewi:
  * `tip_case`: loop invariant switch possible in loop over `n` increased integer dimension of `vl`, `vr`, `le`, `ri`, `v`
  *  if `fast_scaling`: `sum_scale` is overwritten.
  *  if `tip_case` not in [PLL_TIP_INNER, PLL_INNER_INNER], scaling blocks skippable.
