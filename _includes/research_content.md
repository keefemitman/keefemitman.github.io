# Welcome to my research page!

Below you can find some of the research topics that I've worked on in the past,
am working on now, or would like to work on in the future! If you have any questions, feel free to reach out!

## Contents

<style>
ul.no-bullets {
   list-style-type: none;
   padding-left: 0;
   margin: 0;
}
  
li.custom-bullet {
   list-style: none; 
   padding-left: 35px;
   background: url('/assets/images/waveform.png') no-repeat left top;
   background-size: 30px 30px;
}
</style>

<ul class='no-bullet'>
  <li class='custom-bullet'><a href="#numerical-relativity">Numerical relativity</a></li>
  <li class='custom-bullet'><a href="#gravitational-wave-memory">Gravitational wave memory</a></li>
  <li class='custom-bullet'><a href="#gravitational-wave-coordinate-freedoms-the-bms-group">Gravitational wave coordinate freedoms: the BMS group</a></li>
  <li class='custom-bullet'><a href="#gravitational-wave-hybridizations">Gravitational wave hybridizations</a></li>
  <li class='custom-bullet'><a href="#gravitational-wave-surrogate-models">Gravitational wave surrogate models</a></li>
  <li class='custom-bullet'><a href="#binary-black-hole-ringdowns">Binary black hole ringdowns</a></li>
  <li class='custom-bullet'><a href="#active-galactic-nuclei-agns">Active galactic nuclei (AGNs)</a></li>
</ul>

---

## Numerical relativity

As one may imagine, while solving Einstein's equations analytically for a single black hole is relatively simple (thanks Roy Kerr!),
solving Einstein's equations analytically for *two* black holes is effectively impossible and instead one solves them numerically
by running a numerical simulation. A priori, however, Einstein's equations aren't set up in a form that's easy to "evolve". Consequently,
to remedy this, we often write Einstein's equations in what's called "3+1 form", i.e., specifying three spatial dimensions and one time dimension,
with which we will evolve some space-like initial data. If one works in an appropriate *gauge*, i.e., a well-behaved coordinate system,
then evolving this initial data becomes a well-posed problem (i.e., a solution will exist and be unique).

While this grossly oversimplifies the amount of work required to run a successful numerical relativity simulation, this is the main gist.
And at the end of the day, if the code is written correctly, you can obtain some pretty remarkable simulations like this:

<video width="640" height="360" controls>
  <source src="/assets/images/simulation.mp4" type="video/mp4">
</video>

In the above simulation, color represents the lapse $\alpha$, which effectively describes how slow time flows due to the spacetime's curvature.
At the bottom there's also a waveform representing the emitted gravitational wave, which is what we actually observe! But how do we compute that?

Earlier I mentioned that when simulating Einstein's equations, it's particularly important to understand and control the coordinate system
that you're using. And this is part of what makes numerical relativity so challenging: there's an infinite number of coordinates to choose from!
This is the general *diffeomorphism invariance* of general relativity. As a result, we can't simply provide the LIGO-Virgo-KAGRA Collaboration with
the spacetime metric on some worldline in our simulation because it would be highly dependent on the arbitrary coordinate system we chose.
Instead, we need a way to limit how much we need to worry about coordinates. One way to do this is to look to some region of spacetime that has
some kind of structure that we cannot simply change arbitrary. But where to look? If we assume that the spacetime we're simulating is *asymptotically flat*, i.e.,
that it has zero curvature infinitely far away from the origin, then we've naturally focused in on a region of spacetime with structure! While this boundary
of spacetime can be partitioned into components, the one we will be interested in is called *future null infinity*---the final destination of
outgoing null (i.e., light-like) radiation such as gravitational waves. Future null infinity, or $\mathcal{I}^{+}$ for short, has a particular structure
that restricts what type of coordinate systems are valid. Furthermore, the metric takes on a particularly nice form known as the Bondi-Sachs metric:

$$
\begin{align}
ds^{2}&=-du^{2}-2dudr+2r^{2}\gamma_{z\bar{z}}dzd\bar{z}\nonumber\\
&\phantom{=.}+\frac{2m_{B}}{r}du^{2}+rC_{zz}dz^{2}+rC_{\bar{z}\bar{z}}d\bar{z}^{2}+D^{z}C_{zz}dudz+D^{\bar{z}}C_{\bar{z}\bar{z}}dud\bar{z}\nonumber\\
&\phantom{=.}+\frac{1}{r}(\frac{4}{3}(N_{z}+u\partial_{z}m_{B})-\frac{1}{4}D_{z}(C_{zz}C^{zz}))dudz+\mathrm{c.c.}+\cdots,
\end{align}
$$

where "c.c." stands for complex conjugation, $D_{z}$ is the covariant derivative with respect to the round metric on the two-sphere $\gamma_{z\bar{z}}$, $m_{B}$, $C_{zz}$, and $N_{z}$ are functions of $(u,z,\bar{z})$, and ellipsis represent subleading corrections. While "nice" may sound ironic after seeing the
equation for the metric, it really is nice because $C_{zz}$ is exactly the gravitational wave that we observe in our gravitational wave detectors!
So if we want to compute the gravitational wave in our simulations, then we simply need to solve Einstein's equations in this Bondi-Sachs gauge.

<img align="right" src="/assets/images/foliation.png" alt="drawing" width="200"/>

In practice this works via the following. First, solve Einstein's equations for a gravitational system in 3+1 form on a series of
Cauchy (i.e., space-like) slices, i.e., the blue foliations shown on the right. Then, at some radius, write the metric
(and it's derivatives) to file to produce a "worldtube" of metric data. Once this evolution is finished, one can then proceed to do a
*second* evolution that includes future null infinity on the computational grid by *compactifying* the radial coordinate, i.e.,
redifining one's coordinates so that instead of working with $r\in[r_{\mathrm{worldtube}},\infty]$ one works with finite coordinates, say,
$y\equiv1-\frac{2r_{\mathrm{worldtube}}}{r}\in[-1,1]$.

<img align="right" src="/assets/images/CCE_cartoon.png" alt="drawing" width="200"/>

This evolution is called Cauchy-characteristic evolution (CCE), and instead evolves Einstein's equations on null slices (rather than Cauchy slices).
Furthermore, because the radial coordinate is compactified, future null infinity is included on the computational grid so one can
easily compute the gravitational wave that current gravitational wave detectors observe! In fact, by performing a Cauchy evolution of
a binary black hole coalescence that closely resembles the first gravitational wave detection, GW150914, using the SXS Collaboration's code $\texttt{SpEC}$
followed by a Cauchy-characteristic evolution executed with the code $\texttt{SpECTRE}$, then one obtains the following solution to Einstein's equations:

![](/assets/images/GW150914.png)

But what's with the offset at the end? (hint: keep reading to find out)

Image Credit:
- SXS Collaboration, <a href="https://www.youtube.com/watch?v=c-2XIuNFgD0">YouTube Channel</a>
- Gourgoulhon, <a href="https://arxiv.org/abs/gr-qc/0703035">arXiv:gr-qc/0703035</a>
- Moxon *et al.*, <a href="https://arxiv.org/abs/2110.08635">arXiv:gr-qc/2110.08635</a>

---

## Gravitational wave memory

<img align="right" src="/assets/images/memory.gif" alt="drawing" width="200"/>

Above we saw that unlike what's commonly presented in the literature, gravitational waves typically do not decay to zero at late times!



This phenomenon is known as the *gravitational wave memory effect* and corresponds to the fact that two freely-falling observers
will experience a net displacement relative to each other due to a gravitational wave passing through the spacetime between them.
This is illustrated through the gif shown on the right. While time progresses, the spacetime between the observers (the points) oscillates
with the peaks and the troughs of the gravitational wave. But, since the gravitational wave does not decay to zero at late times
(due to the memory effect) the observers remain permanently displaced relative to their initial circular configuration!
While this effect has yet to be observed, we can resolve it in numerical relativity simulations. And, in the next five-ish years,
we expect to be able to claim a detection within the population of events observed by the LVK collaboration.

To do so, however, necessitates that one can separate out the memory contribution to the gravitational wave strain from the
usual oscillatory contribution. So, in [arXiv:2007.11562](https://arxiv.org/abs/2007.11562) and [arXiv:2011.01309](https://arxiv.org/abs/2011.01309)
(or see [arXiv:2405.08868](https://arxiv.org/abs/2405.08868) for a review)
we showed exactly how to do this with numerical relativity simulations! It turns out that because memory is
intimately related to the symmetries of asymptotic infinity
(see [Gravitational wave coordinate freedoms: the BMS group](#gravitational-wave-coordinate-freedoms-the-bms-group))
one can utilizes Noether's theorem to construct balance laws which naturally decompose the strain into a
component which looks like the usual oscillatory contribution and a component which looks like the step-like memory contribution.
Consequently, understanding whether or not we've observed memory in a gravitatianal wave observation is trivial
(provided that we can observe a binary black hole coalescence with a high enough signal-to-noise ratio!)

---

## Gravitational wave coordinate freedoms: the BMS group

In [Numerical Relativity](#numerical-relativity) we explained why it is much easier to study gravitational radiation from
gravitational systems at future null infinity. In particular, the added structure of the boundary of asymptotically flat spacetimes
makes it so that the coordinate freedoms one must control are not abitrary diffeomorphisms, but rather a more restricted set of symmetries.
While one may naively expect the symmetries of future null infinity to simply be the 10 Poincar&eacute; symmetries---four spacetime translations,
three Lorentz rotations, and three Lorentz boosts---it turns out that there is actually an infinite number of symmetries,
which are contained in a particular extension of the Poincar&eacute; group known as the *BMS group*, named after
Bondi, van der Burg, Metzner, and Sachs.

These additional symmetries of the BMS group come from the fact that future null infinity is built from an infinite number of null generators:
one for each point on the celestial two-sphere (mathematicall we have $I^{+}\cong\mathbb{R}\times S^{2}$). Physically, a way to think of this is the following.
Imagine you have a group of observers watching some event, say a supernovae. If these observers know their relative positions, then they can change
their personal clocks so that they all receive the same spherically-symmetric information at the same time. However, if you push these observers to
asymptotic infinity, then they become causally disconnected from one another, so there is no way for them to synchronize their personal clocks to begin with,
meaning these "direction-dependent" time translations, a.k.a., "supertranslations", are a symmetry of the system.

Consequently, whenever one computes a gravitational wavefrom at future null infinity, it is subject to the infinite number of BMS freedoms,
which must be controlled in some well-defined manner! Across [arXiv:2105.02300](https://arxiv.org/abs/2105.02300) and [arXiv:2208.04356](https://arxiv.org/abs/2208.04356) (with a review in [arXiv:2405.08868](https://arxiv.org/abs/2405.08868)), we describe exactly how this can be done to compare NR waveforms from different resolutions,
or NR waveform to other waveform approximants, like post-Newtonian waveforms! (see below!)

---

## Gravitational wave hybridizations

In developement

---

## Gravitational wave surrogate models

In developement

---

## Binary black hole ringdowns

In developement

---

## Active galactic nuclei (AGNs)

In developement