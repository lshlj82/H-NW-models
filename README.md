# The same graph, invented twice: Harary vs Newman–Watts

An interactive, single-file web demo accompanying

> S. Son, E. J. Choi, S. H. Lee, “Revisiting small-world network models: exploring technical realizations and the equivalence of the Newman–Watts and Harary models,” *Journal of the Korean Physical Society* **83**, 879–889 (2023). [doi:10.1007/s40042-023-00921-8](https://doi.org/10.1007/s40042-023-00921-8)

This demo was created by **Claude Opus 5.5** (Anthropic), working from the paper.

The paper makes three points, and the demo lets you check each one live in the browser:

1. The **Harary graph** (1962) is stochastic in its original form, but NetworkX's `hnm_harary_graph` places the leftover edges deterministically, which shrinks the range of reachable (clustering, path length) values.
2. The **Newman–Watts model** (1999) adds shortcuts between uniformly random node pairs, but NetworkX's `newman_watts_strogatz_graph` forces each shortcut to start at an endpoint of a ring edge. This changes the degree distribution and, slightly, the clustering. The original model's clustering is also non-monotonic in *p*, which the paper's Eq. (6) captures.
3. With an even connectivity *r*, the original Harary graph and the original Newman–Watts model are **the same random graph** under the mapping *k*<sub>init</sub> = *r*, *p* = 2*m*/(*nr*) − 1.

## Running it

There is nothing to build or install. Open `index.html` in a recent browser, or serve the folder with any static server:

```bash
python3 -m http.server
# then visit http://localhost:8000
```

To host it on **GitHub Pages**, push this repository and enable Pages under *Settings → Pages* with the root of the main branch as the source.

The page loads its two typefaces (Literata and Schibsted Grotesk) from Google Fonts and falls back to system fonts when offline. Formulas are written in MathML, which current versions of Chrome, Edge, Firefox and Safari render natively. There are no other external dependencies.

## What's on the page

| Section | Paper | What you can do |
|---|---|---|
| Harary's rule, with the corresponding Newman–Watts graphs | §2.1–2.2, §4 | Choose *n* and *m* and compare original Harary, NetworkX Harary, original Newman–Watts and NetworkX Newman–Watts graphs side by side, drawn on a circle with random edges highlighted. The Harary construction case (1a, 1b, 2, 3a, 3b) is identified and explained. |
| Randomness buys a wider range of graphs | Fig. 2 | Regenerate 1,760 NetworkX and 17,600 original Harary graphs at *n* = 64, *m* ∈ [256, 2015], and plot them in the (*C*, *L*) plane. |
| Two ways to add a shortcut | §3.1–3.3, Fig. 3b,d | Compare degree distributions of the two Newman–Watts versions for any *n*, *k*<sub>init</sub>, *p*, with standard deviations and the paper's *s* = 2/3 crossover prediction. |
| Clustering turns back up | §3.4, Figs. 3a,c and 4 | Simulated *C*<sub>*p*</sub>/*C*<sub>0</sub> (and optionally *L*<sub>*p*</sub>/*L*<sub>0</sub>) for both versions against Eq. (6), its one-shortcut truncation, and Newman's approximation (A1). The full formula is shown on the page. |
| The equivalence | §4, Fig. 5 | Relative clustering and path length of original Harary and original Newman–Watts graphs against *m*, for *r* ∈ {2, 4, 6, 8}. |

### The clustering formula

The analytic curve is Eq. (6) of the paper, normalized by its value at *p* = 0, *C*<sub>0</sub> = 3(*k*<sub>init</sub> − 2) / [4(*k*<sub>init</sub> − 1)]:

```
C_p = 3 · [ (n k/4)(k/2 − 1)
          + (k/2 + 1)(k/4) · k p/(n − 1 − k) · n
          + (k² p²/2)(k/n) · n
          + (k² p²/2)(1 − k/n) · k p/(n − 1 − k) · n ]
        / [ ½ n k (k − 1) + n k² p + ½ n k² p² ]          (k = k_init)
```

The numerator terms count triangles in the ring and triangles closed by one, two and three shortcuts. Keeping only the first term gives Newman's approximation, Eq. (A1); keeping the first two gives the one-shortcut curve.

## Implementation notes

All generators and measurements are in plain JavaScript inside `index.html`.

- **Original Harary** follows the case-by-case construction listed in §2.1 of the paper: build the maximally connected core deterministically from rings, diameters and near-diameters, then place the remaining edges uniformly at random among non-edges. In case 2 the core is randomly rotated, so which node has the extra degree varies (the graph is topologically deterministic).
- **NetworkX Harary** reproduces the logic of `hnm_harary_graph` in NetworkX 3.1.
- **Original Newman–Watts** adds Binomial(*n k*/2, *p*) shortcuts between uniformly random non-adjacent pairs. As in the paper, self-loops and multi-edges are disallowed so it can be compared fairly with the NetworkX version.
- **NetworkX Newman–Watts** reproduces the loop in `newman_watts_strogatz_graph` of NetworkX 3.1: for each ring edge (*u*, *v*), with probability *p*, connect *u* to a random node *w*, retrying until *w* is new.
- **Measures.** *C* is the average local clustering coefficient, with nodes of degree below 2 counted as zero (as in `networkx.average_clustering`). *L* is the average shortest-path length over reachable pairs. Adjacency is stored as bitsets, so triangle counting and breadth-first search run on 32-bit words, which is what makes tens of thousands of graphs feasible in the browser.
- Long simulations run in small time-sliced chunks so the page stays responsive, and they restart when you change a parameter.

### Deviations and caveats

- Eq. (A1) as printed in the paper gives the last denominator term as ½ *k*² *p*², without the factor *n* that appears in Eq. (6). This looks like a typo, so the demo uses Eq. (6)'s denominator for all three analytic curves.
- The dotted "one-shortcut (Jo-type)" curve is Eq. (6) truncated after the one-shortcut term. This matches the paper's description of Jo's correction, but it has not been checked against the formula in Jo's thesis itself.
- Simulation sample sizes are smaller than the paper's in some panels (for example 30 graphs per point instead of 500 for the clustering curves) to keep the page fast. Expect somewhat noisier points.

## References

1. S. Son, E. J. Choi, S. H. Lee, *J. Korean Phys. Soc.* **83**, 879 (2023). [doi:10.1007/s40042-023-00921-8](https://doi.org/10.1007/s40042-023-00921-8)
2. F. Harary, The maximum connectivity of a graph, *Proc. Natl. Acad. Sci. USA* **48**, 1142 (1962).
3. M. E. J. Newman, D. J. Watts, Renormalization group analysis of the small-world network model, *Phys. Lett. A* **263**, 341 (1999).
4. M. E. J. Newman, *Networks: An Introduction* (Oxford University Press, Oxford, 2010). The clustering derivation used here is omitted from the 2nd edition (2018).
5. H.-H. Jo, *Sandpiles on small-world networks*, Master's thesis, Korea Advanced Institute of Science and Technology (2001). [koasas.kaist.ac.kr/handle/10203/48565](https://koasas.kaist.ac.kr/handle/10203/48565)
6. A. A. Hagberg, D. A. Schult, P. J. Swart, Exploring network structure, dynamics, and function using NetworkX, in *Proc. 7th Python in Science Conference (SciPy 2008)*, pp. 11–15. [networkx.org](https://networkx.org)

### Citing the paper

```bibtex
@article{Son2023Revisiting,
  author  = {Son, Seora and Choi, Eun Ji and Lee, Sang Hoon},
  title   = {Revisiting small-world network models: exploring technical realizations
             and the equivalence of the {Newman--Watts} and {Harary} models},
  journal = {Journal of the Korean Physical Society},
  volume  = {83},
  pages   = {879--889},
  year    = {2023},
  doi     = {10.1007/s40042-023-00921-8}
}
```

## Credits

The demo code and this README were created by Claude Opus 5.5 (Anthropic). The models, formulas and findings are from Son, Choi and Lee (2023) and the works they cite.

## License

Add a license of your choice (for example MIT) before publishing. The paper itself is © The Korean Physical Society and is not included in this repository.
