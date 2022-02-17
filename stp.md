---
layout: default
title: Short-Term Plasticity
description: generalized calculation of short-term plasticity
---

<script type="text/x-mathjax-config">MathJax.Hub.Config({tex2jax:{inlineMath:[['\$','\$'],['\\(','\\)']],processEscapes:true},CommonHTML: {matchFontHeight:false}});</script>
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# Short-Term Plasticity

A simple model of short-term plasticity [[1]](#1) is as follows,

$$
\tau \frac{d P_\mathrm{rel}(t)}{d t} = P_0 - P_\mathrm{rel}(t)
$$

If there is no pre-synaptic spike, $P_\mathrm{rel}(t)$ exponentially approaches to $P_0$ with time constant $\tau$.

In short-term facilitation case, when a pre-synaptic spike comes $P_\mathrm{rel}(t)$ is immediately updated as follows,

$$
P_\mathrm{rel}(t) \leftarrow P_\mathrm{rel}(t) + f_F (1 - P_\mathrm{rel}(t)).
$$

This rule means that $P_\mathrm{rel}(t)$ exponentially approaches to $1$ every time a spike comes with the coefficient $1 - f_F~(0 < f_F < 1)$.

Also in depression case, $P_\mathrm{rel}(t)$ approaches to $0$ with the coefficient $0 < f_D < 1$.

$$
P_\mathrm{rel}(t) \leftarrow f_D P_\mathrm{rel}(t).
$$

$P_\mathrm{rel}(t)$ approaches to $0$ every time a spike comes.

We can write these equations more mathematically [[2]](#2) as follows,

$$
\frac{d P_\mathrm{rel}(t)}{d t} = -\frac{P_\mathrm{rel}(t) - P_0}{\tau} + f_F (1 - P_\mathrm{rel}(t)) \sum_{f} \delta (t-t^f), \\

\frac{d P_\mathrm{rel}(t)}{d t} = -\frac{P_\mathrm{rel}(t) - P_0}{\tau} - f_D P_\mathrm{rel}(t) \sum_{f} \delta (t-t^f),
$$

where $\delta$ is Dirac's delta function, $f$ is pre-synaptic spike index, and $t^f$ is the time spike $f$ occured.

This update rule is very simple, but a bit complicated to implement as a program.

# Generalized Calculation of Short-Term Plasticity

The generalized short-term plasticity is written as follows,

$$
\tau \frac{d P_\mathrm{rel}(t)}{d t} = P_0 - P_\mathrm{rel}(t) \\
P_\mathrm{rel}(t) \leftarrow P_\mathrm{rel}(t) + f_G (P_1 - P_\mathrm{rel}(t)),
$$

or more mathematically,

$$
\frac{d P_\mathrm{rel}(t)}{d t} = \frac{P_0 - P_\mathrm{rel}(t)}{\tau} + f_G (P_1 - P_\mathrm{rel}(t)) \sum_{f} \delta (t-t^f).
$$

where new term $P_1$ is a target that $P_\mathrm{rel}$ approaches to it when spike comes, and $f_G$ is generalized coefficient which determines <em>"amount of plasticity per spike"</em>.

These constants are easily converted from conventional constans.

$$
P_1 = \begin{cases} 1 & (\text{facilitation}) \\ 0 & (\text{depression})\end{cases} \\ 
f_G = \begin{cases} f_F & (\text{facilitation}) \\ 1 - f_D & (\text{depression}) \end{cases} \\
$$

We can not only use binary $P_1 \in \{0, 1\}$, but also continuous $0 \leq P_1 \leq 1$. In this case, $P_1$ regulates maximum facilitation or minimum depression.

Finally, we obtain updating program from above.
For example, the C-style code can be written as follows.

```C
prel = prel + dt * (p0 - prel) / tau + f_G * (p1 - prel) * spike;
```

## Sample Code

```C
void stp_step(double* prel, const int spike, const double dt const double p0, const double p1, const double f_G)
{
    *prel += dt * (p0 - *prel) / tau + f_G * (p1 - *prel) * spike;
}

void stf_step(double* prel, const int spike, const double dt, const double p0, const double f_F)
{
    stp_step(prel, spike, dt, p0, 1.0, f_F);
}


void std_step(double* prel, const int spike, const double dt, const double p0, const double f_D)
{
    stp_step(prel, spike, dt, p0, 0.0, 1.0 - f_D);
}

int main(void)
{
    const double dt = 0.1; // msec 
    const int nstep = 10000;

    const double p0 = 0.5;
    const double f_F = 0.1;
    const double f_D = 0.9;
    const double firingrate = 100; // hz
    const double firingprob = dt * firingrate / 1000.0;

    double std_prel = p0;
    double std_prel = p0;
    for(int step = 0; step < nstep; ++step)
    {
        double t = dt * step;
        int spike = (rand() / (double) RAND_MAX) < firingprob ? 1 : 0;

        stf_step(&stf_prel, spike, dt, p0, f_D);        
        std_step(&stf_prel, spike, dt, p0, f_F);

        printf("%f %d %f %f\n", t, spike, stf_prel, std_prel);
    }
}
```


## References
<a id="1">[1]</a> 
Dayan, Peter, and L. F. Abbott. 2001.
Theoretical neuroscience: computational and mathematical modeling of neural systems.
Cambridge, Mass: Massachusetts Institute of Technology Press.

<a id="2">[2]</a> 
Gerstner, W., Kistler, W., Naud, R., & Paninski, L. (2014). Neuronal Dynamics: From Single Neurons to Networks and Models of Cognition. Cambridge: Cambridge University Press. doi:10.1017/CBO9781107447615

