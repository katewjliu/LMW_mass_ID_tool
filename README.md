# LMW Fragment Mass Matcher (mAb / IgG)

A browser-based tool for proposing candidate identities of low-molecular-weight (LMW) fragment masses observed in intact-mass deconvolution of monoclonal antibodies (mAbs). Given the heavy-chain (HC) and light-chain (LC) sequences plus one or more deconvoluted masses, the tool enumerates plausible clip and chain-assembly candidates within a user-defined mass tolerance, ranks them by a confidence score, and reports the most likely interpretations.

**Live demo:** https://katewjliu.github.io/LMW_mass_ID_tool/

## What it does

For each query mass, the tool searches:

- **Single clips** — any contiguous HC or LC subsequence (full chain, terminal fragment, or internal fragment depending on the *Max cuts per chain* setting).
- **Multi-chain assemblies (1–4 chains, max 2 HC + 2 LC)** — held together by inter-chain disulfide bonds. Auto-detected disulfide topology is used to compute the exact number of S–S bonds per assembly and to reject combinations that aren't physically connected.

It applies common intact-mass-level modifications:

- Pyro-Glu (N-terminal Q→−17.027 Da, E→−18.011 Da)
- C-terminal Lys clipping (HC, 0/1 K loss)
- Fc N-glycoforms (G0F, G1F, G0, G2F, Man5, aglycosyl) at any N-X-S/T (X≠P) sequon
- Cysteinylation (+119.14 Da) and Glutathionylation (+305.31 Da) S–S adducts on free Cys

Results are grouped per query mass and ranked by a 0–100 confidence score that rewards canonical species (Full mAb, Half-mAb, Fab, F(ab')₂, Fc, Fc/2, HC, LC), cleavages at chemical-lability hotspots (D–P, N–G, D–G, N–S, D–X, X–D, G–G, G–L, L–G, S–G, A–S, L–S, X–C), and parsimonious interpretations.

## How to use

1. Paste the HC and LC sequences (one-letter code) into the two textareas, or click **Load trastuzumab example** to populate a known IgG1 reference.
2. Verify the auto-detected disulfide topology summary that appears below the textareas. It should report 4 HC intra-chain pairs, 2 LC intra-chain pairs, 1 HL inter-chain bond, and 2 HH inter-chain bonds for a standard IgG1.
3. Configure the modifications you want to consider (defaults: Pyro-Glu, K-loss, glycoforms G0F/G1F).
4. Pick the **Enzyme treatment** matching your sample preparation. **Untreated** is the default for stability or untreated CHO drug substance and applies only chemical-lability scoring. Selecting an enzyme (IdeS, papain, pepsin, GingisKHAN) adds bonus scoring at that enzyme's known cleavage site.
5. Choose **Sample state**: Non-reduced (default — searches multi-chain assemblies via topology) or Reduced (single-clip search only).
6. Set **Max cuts per chain**: 1 = terminal fragments only (default), 2 = include internal fragments, 0 = full chains only.
7. Paste your deconvoluted query masses (one per line) and click **Search**.

Results appear grouped by query mass. The Min score input (default 70) hides low-confidence candidates; lower it to inspect more options.

## Assumptions and limitations

**IgG1 disulfide topology only.** The auto-detected topology assumes a canonical IgG1 architecture: one CPPC hinge motif defines the inter-HC (HH) bonds, the last Cys before the hinge in HC pairs with the last Cys in LC for the inter-HL bond, and remaining Cys are paired sequentially within each chain as intra-chain bonds. **For IgG2, IgG3, IgG4, antibody fragments (Fabs, scFvs), bispecifics, fusion proteins, or any non-IgG1 construct, the detected topology will likely be incorrect** and assembly masses will be off. Verify the topology summary printed below the sequence textareas before trusting any results.

**Mass calculation.** Average masses are used by default (appropriate for typical intact-mass deconvolution at >5 kDa). Switch to monoisotopic for high-resolution sub-25-kDa fragments. Glycan masses are computed from monosaccharide compositions, not hardcoded.

**Hotspot bonuses are heuristic.** Chemical-lability bonuses (D–P, N–G, etc.) are based on published mAb fragmentation literature (Wang 2007, Vlasak 2011, Cordoba 2005, Yan 2009) but the exact weights are tunable defaults — edit `chemicalBonus()` in the source if your group has a different priority order.

**The matcher is for orientation, not validation.** Proposed clips need to be confirmed by peptide mapping, middle-down sequencing, or other orthogonal methods before being reported. A high score means "consistent with the mass and biologically plausible," not "this is what your peak is."

## How it works (technical overview)

- HC and LC are enumerated as contiguous subsequences subject to *Min clip length* and *Max cuts per chain*.
- For each clip, all combinations of selected modifications (pyro-Glu, K-loss, glycoforms, Cys/GSH adducts) are pre-computed.
- Single-clip search scans all clips for mass matches within tolerance.
- Multi-clip search (K=2,3,4) uses sorted-mass binary search and topology-derived exact disulfide counting for each candidate assembly; assemblies failing connectivity (e.g., 1 HC + 2 LC, where one LC has no inter-chain S–S partner) are rejected.
- Each match is classified (Full mAb / Half-mAb / Fab / F(ab')₂ / Fc / Fc/2 / HC / LC / etc.) and scored.

The tool runs entirely in the browser; no data leaves your machine.

## Files

- `index.html` — the entire tool. Single self-contained HTML file with embedded CSS and JS. No build step, no external dependencies.

## Disclaimer

Provided as-is, with no warranty of any kind. This is a research and exploration tool. Results should not be used as the sole basis for regulatory submissions, batch release decisions, or product disposition without orthogonal confirmation.

## License

Add your preferred license (MIT, Apache-2.0, etc.) here.
