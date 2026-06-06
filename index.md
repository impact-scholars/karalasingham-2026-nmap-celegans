---
title: "Neuro-Modulating Architecture Priors (NMAP) for Context-Dependent Reconfiguration of Locomotory Circuits in Caenorhabditis elegans"
short_title: "NMAP for C. elegans Locomotory Circuits"
options:
    breakable_figures: true
abstract: |
    *Caenorhabditis elegans* switches between distinct crawling and swimming gaits via extrasynaptic dopamine and serotonin signalling, yet existing neuro-inspired architectures such as Neural Circuit Architectural Priors (NCAP) encode only the hardwired synaptic layer with fixed weights, precluding context-dependent reconfiguration. We introduce **Neuro-Modulating Architecture Priors (NMAP)**: a three-layer extension that pairs the NCAP central pattern generator with a GRU context encoder emitting a dopamine/serotonin-analogue modulatory vector and a hierarchical manager that infers substrate from proprioceptive phase-lag alone. Trained with PPO in a progressive water–land MuJoCo environment, NMAP produces a bistable amplitude–frequency separation, four discrete oscillator-period clusters tracking the curriculum, and qualitatively distinct kymographs per substrate. NCAP, under the same curriculum, collapses toward a single compromise gait. Neuromodulatory priors are a necessary architectural extension beyond hardwired connectivity for context-adaptive embodied AI.
acknowledgments: |
    This work was supported by the Neuromatch Impact Scholar Program. We thank Dr. Srikanth Ramaswamy (Neural Circuits Laboratory, Newcastle University) for mentorship, and the program sponsors and teaching assistants whose contributions do not meet the criteria of any authorship role.
---

## Background

Biological locomotion adapts across mechanical contexts through neuromodulation. In *C. elegans*, dopamine is necessary for crawling and serotonin for swimming [@vidalgadea2011]; dopamine-deficient mutants exhibit unstable locomotion rates [@omura2012]. Whereas hardwired synapses support point-to-point communication, neuromodulators act as wireless broadcasts that dynamically reconfigure entire network states [@randi2023] — a distinction that integrated neuromechanical models alone cannot capture [@boyle2012]. Neural Circuit Architectural Priors (NCAP) embed sparse connectivity, sign constraints, and intrinsic dynamics as differentiable priors for embodied control [@bhattasali2022]. While effective for single-substrate locomotion, NCAP captures only the hardwired synaptic layer with fixed weights and cannot reconfigure motor output across mechanical contexts. We close this architectural gap with **Neuro-Modulating Architecture Priors (NMAP)**, in which a learned modulatory layer sits on top of NCAP and a hierarchical manager selects gait from proprioception alone.

## Methods

**Simulation.** All experiments used the `dm_control` physics suite [@tunyasuvunakool2020] with a six-link, five-joint swimmer modelled on *C. elegans*. We used two substrate regimes: an aquatic baseline at viscosity $\mu = 0.001$, and a progressive mixed substrate where circular land zones at $\mu = 0.05$ were introduced in a four-phase curriculum (pure water, then one, two, and four islands). The agent received no explicit zone-detection signal; effective viscosity updated each step from head position relative to zone boundaries.

**NMAP architecture.** Layer 1 preserves the standard NCAP central pattern generator (CPG) unchanged. Layer 2 adds a *context encoder* — a GRU that consumes only the inter-joint phase-lag signal (consistent with the absence of viscosity sensors in *C. elegans*) and emits a two-dimensional context vector analogous to dopamine and serotonin signalling, which produces per-joint gain and bias parameters applied at each CPG step. The effective oscillator period is modulated as a sigmoid function of the context vector, enabling smooth swim-to-crawl transitions. Layer 3 introduces an HRL *manager* that observes accumulated mechanical context and issues a continuous mode signal to the encoder, implementing a timescale separation between gait selection and locomotion.

**Training.** All models were trained with PPO for $3 \times 10^{6}$ environment steps. The base reward is a tolerance function on the swimmer head's forward velocity. Experiment 3 (NMAP) added a transition bonus on confirmed substrate crossings and a mismatch penalty for applying the wrong gait, both annealed from zero over the first 30 % of training. We compared (1) NCAP on a single aquatic substrate, (2) NCAP on the progressive curriculum, and (3) NMAP on the progressive curriculum.

## Results

NMAP produces measurable bistable gait switching that matches *C. elegans* kinematics; NCAP, under identical curriculum and reward, fails to differentiate motor output by substrate across every diagnostic.

**Amplitude–frequency separation (@figure-main A,B).** These scatter plots characterise locomotion kinematics by plotting joint amplitude (rad) against undulation frequency (Hz) across all locomotion bouts; in *C. elegans*, swim and crawl gaits occupy distinct, non-overlapping regions of this space. NCAP on the progressive curriculum (A) produces a broad, diffuse cloud with no coherent clustering — its fixed CPG collapses under mixed-terrain pressure into states committed to neither gait. NMAP (B) is the only model producing two clearly separated clusters — a high-amplitude, low-frequency group (crawl) and a lower-amplitude group (swim) — with a measurable bistable gap at 1.1–1.5 Hz, the discrete two-state structure characteristic of real *C. elegans*.

**Substrate-specific forward speed (@figure-main C,D).** Mean forward speed is measured separately on water (blue) and land (orange) across the training phases. NCAP (C) lets water and land speeds converge toward a shared compromise by Phase 3, with heavily overlapping error bars — a single intermediate gait adequate on both substrates but excelling at neither. NMAP (D) maintains a persistent, directional water–land gap ($\sim$0.007–0.020 m/s, water faster), reflecting genuine dual specialisation rather than compromise.

**Body-wave morphology (@figure-main E,F).** Kymographs plot joint position (anterior to posterior, vertical) against time (horizontal); colour encodes the direction of joint deflection, and diagonal stripe spacing encodes wave speed and period. NCAP (E) produces near-identical, swim-like stripe patterns on water and land across every phase — no terrain adaptation. NMAP (F) produces fast, tightly spaced diagonals in water and visibly slower, broader bands on land, with the contrast sharpening from Phase 1 to Phase 3 — the most direct visual evidence of genuine gait switching.

**CPG period commitment** (described here, not figured). Swim and crawl require different body-wave frequencies: fast short-period oscillations in low-viscosity water, slow long-period oscillations on high-viscosity land. NCAP on a single substrate locks near its nominal 60-step period (mean 58.9 steps); on the progressive curriculum it collapses to a broad unimodal distribution (mean 19.8 steps) with no discrete structure. NMAP instead shows discrete clusters spanning short ($\sim$10–15 step, swim) and long ($\sim$55–65 step, crawl) periods — the CPG commits to substrate-appropriate frequencies rather than settling on an intermediate.

## Ablations

We isolated the contribution of each architectural component with three ablations of NMAP under identical training conditions. **Removing neuromodulation** (gain = 1, bias = 0) contracts the land-crawl wave period toward swim frequency across all phases; switching degrades but is not eliminated, because the HRL manager retains residual modulation capacity. **Disabling the bistability regulariser** allows context vectors to adopt continuous intermediate values rather than committing to discrete attractors; discrete gait commitment collapses, the reward profile reverts to an NCAP-like pattern, and water performance drops by 16 % — making this the single most critical component. **Removing anisotropic drag** reduces substrate contrast to viscosity alone; switching persists via viscosity contrast, but the crawl cluster over-amplifies, degrading biological correspondence without eliminating gait differentiation.

```{figure} figure.svg
:name: figure-main
:alt: Six-panel vector figure contrasting NCAP and NMAP on the progressive water-land curriculum.

NCAP (left of each pair) versus NMAP (right) on the progressive water–land curriculum. Throughout, blue denotes the water substrate and orange the land substrate.
\
**A.** NCAP amplitude–frequency (joint amplitude vs undulation frequency): a broad, diffuse scatter with no swim/crawl separation.
\
**B.** NMAP amplitude–frequency: two separated clusters — high-amplitude, low-frequency (crawl) and lower-amplitude (swim) — with a bistable gap at $\sim$1.1–1.5 Hz.
\
**C.** NCAP forward speed by curriculum phase (blue = water, orange = land): the water–land gap collapses toward a shared compromise.
\
**D.** NMAP forward speed by phase: a stable, directional water–land speed gap is preserved across the curriculum.
\
**E.** NCAP kymographs across phases (blue = water column, orange = land column): near-identical swim-like body waves on both substrates.
\
**F.** NMAP kymographs across phases: fast short-period swim waves in water; slow long-period crawl waves on land.
```

## Conclusion

NMAP demonstrates that a learned dopamine/serotonin-analogue modulatory layer, coupled with a hierarchical manager that infers substrate from proprioceptive phase-lag alone, is sufficient to produce bistable swim–crawl transitions that quantitatively match *C. elegans* kinematics. NCAP, despite an identical CPG and curriculum, cannot. Ablations identify the bistability regulariser as the single most critical component, with neuromodulation and anisotropic drag providing complementary contributions to switching precision. **Neuromodulatory priors are a necessary architectural extension beyond hardwired connectivity for context-adaptive neuro-inspired AI.**

## Code availability

The code used in this study is publicly available at <https://github.com/ShirodkarTejas/nma_nai_on> and <https://github.com/sagthi/NMAP>. Published via [Impact Scholars](https://github.com/impact-scholars/karalasingham-2026-nmap-celegans).
