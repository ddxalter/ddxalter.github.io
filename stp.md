---
layout: default
title: Short-Term Plasticity
description: generalized calculation of short-term plasticity
---

<script type="text/x-mathjax-config">MathJax.Hub.Config({tex2jax:{inlineMath:[['\$','\$'],['\\(','\\)']],processEscapes:true},CommonHTML: {matchFontHeight:false}});</script>
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# Short-Term Plasticity

A simple model of short-term plasticity is as follows [[1]](#1),
$$
\tau \frac{d P_\mathrm{rel}(t)}{d t} = P_0 - P_\mathrm{rel}(t)
$$
If there is no pre-synaptic spike, $P_\mathrm{rel}(t)$ exponentially approaches to $P_0$ with time constant $\tau$.

In short-term fascilitation case, when a pre-synaptic spike comes $P_\mathrm{rel}(t)$ is immediately updated as follows,
$$
P_\mathrm{rel}(t) \leftarrow P_\mathrm{rel}(t) + f_F (1 - P_\mathrm{rel}(t)).
$$
This rule means that $P_\mathrm{rel}(t)$ exponentially approaches to $1$ every time a spike comes with the coefficient $1 - f_F~(0 < f_F < 1)$.

Also in depression case, $P_\mathrm{rel}(t)$ approaches to $0$ with the coefficient $0 < f_D < 1$.
$$
P_\mathrm{rel}(t) \leftarrow f_D P_\mathrm{rel}(t).
$$
$P_\mathrm{rel}(t)$ approaches to $0$ every time a spike comes.

This update rule is very simple, but a bit complicated to implement as a program.

# Generalized Calculation of Short-Term Plasticity

The generalized short-term plasticity is written as follows
$$
\tau \frac{d P_\mathrm{rel}(t)}{d t} = P_0 - P_\mathrm{rel}(t) \\
P_\mathrm{rel}(t) \leftarrow P_\mathrm{rel}(t) + f_G (P_1 - P_\mathrm{rel}(t)),
$$
or more mathematicaly,
$$
\frac{d P_\mathrm{rel}(t)}{d t} = \frac{P_0 - P_\mathrm{rel}(t)}{\tau} + f_G (P_1 - P_\mathrm{rel}(t)) \sum_{f} \delta (t-t^f).
$$
where new term $P_1$ is a target that $P_\mathrm{rel}$ approaches to it when spike comes, and $f_G$ is generalized coefficient which determines <em>"amount of plasticity per spike"</em>.

These constants are easily converted from conventional constans.
$$
P_1 = \begin{cases} 1 & (\text{fascilitation}) \\ 0 & (\text{depression})\end{cases} \\ 
f_G = \begin{cases} f_F & (\text{fascilitation}) \\ 1 - f_D & (\text{depression}) \end{cases} \\
$$

Finally, we obtain updating program from above.
For example, the C-style code can be written as follows.
```C
prel = prel + dt * (p0 - prel) / tau + spike * f_g * (p1 - prel)
```

## Sample Code

```C
void stp_step(double* prel, const int spike, const double dt const double p0, const double p1, const double f_g)
{
    *prel += dt * (p0 - *prel) / tau + spike * f_g * (p1 - *prel)
}

void stf_step(double* prel, const int spike, const double dt, const double p0, const double fF)
{
    stp_step(prel, spike, dt, p0, 1.0, f)
}


void std_step(double* prel, const int spike, const double dt, const double p0, const double fD)
{
    stp_step(prel, spike, dt, p0, 0.0, 1.0 - fD)
}
```


## References
<a id="1">[1]</a> 
Dayan, Peter, and L. F. Abbott. 2001.
Theoretical neuroscience: computational and mathematical modeling of neural systems.
Cambridge, Mass: Massachusetts Institute of Technology Press.
