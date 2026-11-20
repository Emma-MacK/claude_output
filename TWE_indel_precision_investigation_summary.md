# TWE (WES) indel precision investigation: summary

*Prepared 19 Nov 2026. Sources:*
- *Main source: `O-260716 TWE investigation run` (Confluence export).*
- *Background: `O-260703 WES Indels`, `O-TWE Indel impact`, `INDEL_inv_initial_dev_des.pptx`, `INDEL_Dev_des_2.pptx`, `INDEL_inv_rerun_analysis.pptx`, `INDEL summary.docx`.*
- *QC data: the verity database snapshot `verity_db_20261117_1311.tar.gz` (file-JFy551Q4JKFVPKkJ7bqJ497k, project 003_eggd_verity_dev), queried read-only.*

---

## Section 1: Investigation summary

### Problem
- hap.py indel precision for the TWE control (NA12878 / 9526) has been falling, and indel recall has become less consistent. TWE runs were put on hold because of the risk that a pathogenic indel would be missed or that a false indel would be reported.
- The trend was found with Westgard-style rolling and individual plots, using the first 30 runs as the baseline.
- Realigning older GRCh37 runs to GRCh38 lowers their precision slightly, but the downward trend is still there.
- Precision has gone from about 0.77–0.80 (2024) to about 0.70–0.72 (Apr–Nov 2026). The verity data in Section 2 confirms this.

### What was ruled out (background investigation)
| Candidate | Finding |
|---|---|
| Conductor config v2.5.0 / dias_single v2.14 | No change to calling. The testing-page precision of 0.806 came from a run that already had ~0.80. |
| Sentieon v5.1.0 | v5.1.0 vs v4.2.2 gave 100% concordance. Re-running hap.py on the v5.1.0 control gave the production value. |
| hap.py assets (truth VCF/BED, SDF, panel BED) | None changed since 2023–25. |
| UMI masking | Re-running 260319_A01295_0727 with the UMI-masking config made no difference to indel precision. |
| Post-capture PCR cycles | 8 cycles (251027_…0652) vs 11 cycles (251103_…0625) showed no obvious difference. |
| Control stock / extraction | Same control used throughout. The Chemagic go-live and pause did not line up with the drop. |

**Left open:**
- **Twist reagent lots.** The drops map to lot changes, especially probe lot 2500007516 (from 26-NGWES1, Jan 2026). One hybridisation reagent lot may have been held at the wrong temperature. Twist were contacted on 18/06/2026.
- **The TULIP unified library prep (CU-NG-SOP-32, live 17 Mar 2026).** It drops the WES 95 °C pre-denaturation before the 70 °C hybridisation hold. It also lists 11 enrichment PCR cycles, although CU-VVP-92 says WES stays at 8.
- **Why CEN shows no drop.** CEN shares the same harmonised wet lab and the same control, but its truth set has only about 6 indels, so it is not sensitive to this.

### Validation re-run (38_TWE-260715_0833)
- The original 2025 TWE samples were re-sequenced and compared sample-to-sample against their original VCFs with vcfeval/hap.py. Three regions were used: GMS PanelApp transcripts, Twist Comprehensive Exome targets and full chromosomes. **All comparisons passed.**
- **Workbook indels (run 250204_A01295_0476, 23 indels):**
  - All 3 real indels were still called in the validation VCFs.
  - None of the 7 indels with no comment (reported NMD) were called again.
  - 4 of the 13 artefact or "ignore" indels were called again.
  - So nothing reportable was lost.

### Control FN/FP review: the key finding
- hap.py FP and FN indel records were merged across all control comparisons and reviewed locus by locus. FN classification:
  - **273: wrong genotype.** The variant is present but has the wrong zygosity. Each one is counted as both an FP and an FN.
  - **47: different indel called at the same position.** Different length or allele.
  - **51: not called.** Low count, or no majority allele, e.g. a 1–4 bp deletion of mixed length.
- **Outcome (as reported):**
  - About 83% of FN are explained by a wrong GT or an alternate indel call, so the variant evidence is still in the data.
  - Of the remaining 61 FN, 43 come from one variant that the control misses consistently.
- **Per-case impact ("TWE Indel impact"; about 105 truth indels per case):**

| | Best period (24-NGWES21 to 25-NGWES5) | Recent period (26-TULIP31 to 26-TULIP68) |
|---|---|---|
| FN/FP with wrong GT | 4.6 | 6.0 |
| FN/FP with wrong allele | 0.8 | 1.1 |
| FN missed entirely | 0.5 | 0.75 |
| FP not explained by GT/allele | 23.1 | 35.6 |

- No variant is missed or incorrect only in the current period. Every recurrent error locus was already present before the drop.

### Decision and actions
- Combined SNP + indel recall is above 99%, which is the WES validation requirement. Most FN are explained, so **TWE processing resumed.** Runs processed during the investigation had no reportable indels. A Sanger-confirmation note was added to EPIC for the affected run.
- **Actions:**
  1. Continue to monitor WES indel precision.
  2. Review how "FP due to GT" and FN counts are shown in MultiQC.
  3. Clinical scientists should treat indels in workbooks with suspicion, because the GT may be wrong. Handle them as they would a possible FP.

---

## Section 2: QC trends that correlate with indel precision (excluding hap.py)

### Method
- **Data:** a read-only copy of the verity snapshot. TWE38 (GRCh38) control samples, with one hap.py PASS record per run (latest MultiQC report). This gives **124 runs, Jun 2021 to Nov 2026**. The pre-2025-08 rows are the GRCh38 re-processing of older runs.
- **QC sources joined by sample:**
  - Picard HsMetrics (all 124)
  - samtools flagstat (all 124)
  - verifyBamID (all 124)
  - somalier (all 124)
  - vcf_qc het/hom (all 124)
  - FastQC (all 124)
  - Sentieon alignment and insert size (85)
  - Picard dup, insert size, alignment and variant-calling metrics (39). These only exist from config v2.5.0, so there are just 2 points before 2026.
- **Statistics:**
  - Spearman ρ against indel precision: across all runs, and within the pre-2026 runs only.
  - Spearman ρ against time.
  - Mann–Whitney test between P1 (pre-2026, n = 87) and P3 (TULIP period, Apr 2026 onwards, n = 32). P2 is Jan–Mar 2026 (n = 5).
- Analysis scripts and tables are in the session scratchpad (`build_table.py`, `analyse.py`, `plots.py`, `correlations.tsv`).

**hap.py context (P1 → P3 medians):**
- Precision 0.770 → 0.714. Recall 0.933 → 0.924.
- Indel FP 29 → 41. Of these, FP caused by GT mismatch went 6 → 7.5, so **most of the extra FP are not GT errors** (24 → 34).
- SNP precision and recall barely changed (0.992 → 0.991, 0.996 → 0.995).

### Metrics that correlate
| Metric (tool) | ρ vs precision (all) | ρ pre-2026 only | P1 median | P3 median | Change | MW p |
|---|---|---|---|---|---|---|
| **FREEMIX (verifyBamID)** | **−0.63** | −0.27 (p = 0.01) | 0.00080 | 0.00220 | **+164%, step change** | <1e-10 |
| % properly paired (samtools) | +0.50 | +0.16 (ns) | 98.71 | 98.07 | −0.65 pts | <1e-6 |
| PCT_EXC_MAPQ (Picard HS) | −0.48 | −0.14 (ns) | 0.0486 | 0.0523 | +7.6% | <1e-6 |
| GC_DROPOUT (Picard HS) | +0.36 | +0.14 (ns) | 0.57 | 0.43 | −25% | <1e-6 |
| PCT_EXC_OFF_TARGET (Picard HS) | +0.36 | +0.02 (ns) | 0.357 | 0.327 | −8.5% | <1e-6 |
| HS_PENALTY_20x / AT_DROPOUT (Picard HS) | +0.35 / +0.30 | +0.47 / +0.41 | 4.69 / 7.97 | 4.73 / 8.32 | ~0 | ns |
| HET_SNP_SENSITIVITY (Picard HS) | +0.29 | −0.02 (ns) | 0.9833 | 0.9823 | −0.1% | <1e-6 |
| Mean hom-alt AF (vcf_qc) | −0.28 | −0.03 (ns) | 0.9977 | 0.9982 | small | <1e-6 |
| Het/hom ratio (vcf_qc) | +0.20 | 0.00 | 1.905 | 1.895 | −0.5% | <1e-4 |
| % singletons (samtools) | −0.19 | +0.24 | 0.11 | 0.14 | +27% | <1e-6 |
| PCT_EXC_DUPE (Picard HS) / samtools dup rate | −0.20 | ~0 | 0.101 | 0.111 | +10% | 0.009 |
| Fold enrichment / zero-coverage targets (Picard HS) | −0.26 / −0.26 | ~0 | 39.8 / 1.30% | 41.3 / 1.39% | +4% / +7% | <1e-4 |

**No meaningful relationship** with:
- Q30 or mean Q-score (bcl2fastq)
- Mismatch, HQ error or indel rate (Sentieon/Picard alignment)
- Mean target coverage, % ≥20x or % usable bases on target
- Insert size
- Estimated library size
- FastQC GC or deduplication
- somalier allele balance (stored at 2 d.p., too coarse to use)
- Mean het allele fraction (0.444 vs 0.444)

**Sequencing quality and depth did not change.**

![Indel precision vs FREEMIX](figures/fig1_precision_vs_freemix.png)

*Figure 1. TWE control indel precision (top) and verifyBamID FREEMIX (bottom). Dashed lines: (1) TWE GRCh38 production, Aug 2025; (2) first use of probe lot 2500007516, Jan 2026; (3) config v3.1.0 and TULIP go-live, 17–25 Mar 2026.*

![FREEMIX by assay](figures/fig2_freemix_by_assay.png)

*Figure 2. Monthly median FREEMIX in patient samples. TWE, CEN and MYE all step up at the end of March / April 2026.*

![Other QC metrics](figures/fig3_other_qc_metrics.png)

*Figure 3. Other non-hap.py metrics that change in step with the April 2026 precision drop.*

### Interpretation
1. **FREEMIX is the strongest correlate, and it is a step change, not a drift.**
   - TWE control FREEMIX sits at 0.0008–0.0010 through to 19 Mar 2026 (26-NGWES5). From the first TULIP run (7 Apr 2026, 26-TULIP13) it sits at 0.0019–0.0027.
   - The same step appears in **all TWE patient samples** and in **CEN**. CEN goes from about 0.0013 to 0.0021, starting with runs dated 25 Mar 2026. MYE also rises a little.
   - The cause is therefore a change that affects all harmonised assays, not something specific to the control or to TWE.
   - Within the TULIP period, FREEMIX does not track precision (ρ = −0.07, n = 32). It marks the process change rather than acting as a dose-response predictor.
   - The absolute level (about 0.2%) is far below any contamination threshold.
2. **Read-pairing and mapping quality got worse at the same step.** Properly paired reads fell, singletons rose, and reads excluded for low MAPQ rose. This points to more chimeric or improperly paired fragments, but not more base-calling error (mismatch rate is unchanged).
3. **Capture behaviour changed.**
   - GC dropout fell sharply, off-target fell and fold enrichment rose. Duplication rose about 10%.
   - This fits a change to the hybridisation and enrichment PCR conditions: the TULIP hyb programme without the 95 °C step, and possibly 11 PCR cycles.
4. **Genotype-level QC moved slightly in the direction of the GT errors.** Het SNP sensitivity fell a little, mean hom-alt allele fraction rose, and the het/hom ratio fell. The effect sizes are small.
5. **Main confounder: the timing.** The step lines up with *both* TULIP go-live (17 Mar) and dias config v3.1.0 (live 25 Mar). Verity alone cannot separate them.
   - **Recommended check:** compare FREEMIX, properly-paired % and PCT_EXC_MAPQ in `38_TWE-260702_1210` (all TWE controls re-processed with the new pipeline) against the original runs.
   - If old runs re-processed with v3.1.0 keep FREEMIX at about 0.0009, the step is wet-lab (TULIP).
6. **The decline began before TULIP, and it does have a QC correlate: library complexity.** The first production run with the drop, 260218_A01295_0701, came before harmonisation and multiplexing. It shows a clear rise in duplicates and fall in library size. See Section 2b.

### 2b. Run 260218_A01295_0701 and later low-precision runs

**Context:**
- 260218_A01295_0701 (26-NGWES3, control precision 0.721, FN 10, recall 0.905) is the first production run to show the drop.
- It came before TULIP harmonisation (17–25 Mar 2026). It also came before multiplexing in production: the first TWE flow cell shared with CEN/MYE in verity is 260417_A01295_0751 (17 Apr 2026).
- The multiplexing validation (`DV-Multiplexing S1 flow cell QC assessment`, approved 12 Mar 2026) found that FREEMIX roughly doubles (about 0.1% → 0.2%) when compared with production. The rise was mostly already present in the harmonisation (masked-UMI) runs, so the document attributes it to harmonisation rather than multiplexing. Insert size fell by 13–25 bp for TWE. Fewer reads were mapped for TWE (−16.5%), but this stayed above thresholds.

**Figures 4–6 method:**
- They cover the GRCh38 production period (Aug 2025 – Nov 2026) in the same style as Figure 1.
- Each panel shows two series: the per-run **median of all patient samples** (blue) and the **control** (orange).
- Run 0701 is shaded.
- Runs whose control precision is below the pre-2026 baseline −2 SD (0.708) are ringed. There are 13 of them: 0727, 0742, 0751, 0769, 0783, 0788, 0808, 0824, 0847, 0849, 0864, 0870 and 0867.
- Per-run patient medians are in `twe38_run_patient_medians.tsv` (scratchpad, `run_level.py`, `plots2.py`).

![Library complexity](figures/fig4_library_complexity.png)

*Figure 4. Library complexity. Run 0701 has the highest duplicate fraction and the lowest estimated library size of any pre-TULIP production run, in both the control and the patient samples. The run before it, 260205_0690 (precision 0.731), starts the same pattern.*

![Harmonisation markers](figures/fig5_harmonisation_markers.png)

*Figure 5. FREEMIX, properly paired % and PCT_EXC_MAPQ. These are normal on 0701 and step up only with TULIP harmonisation (line 2), before multiplexing started (line 3).*

![Capture and insert size](figures/fig6_capture_and_insert.png)

*Figure 6. GC/AT dropout and insert size. Run 0701 has unusually low AT dropout (control 4.0, patient median 2.3) and the longest median insert size of the period (239 bp). After TULIP, GC dropout falls, AT dropout rises and insert size gets shorter.*

**Two separate changes are visible:**

1. **Feb–Mar 2026 (pre-harmonisation, pre-multiplexing): runs 0690, 0701 and 0727 show a loss of library complexity.**

   | | Production mean (Aug 2025 – Jan 2026) | Run 0701 |
   |---|---|---|
   | Control duplicate fraction | 0.092 | **0.152** (z = +4.8) |
   | Patient median duplicate fraction | 0.080–0.115 | **0.150** |
   | Patient median library size | 2.0–2.9 × 10⁸ | **1.5 × 10⁸** |

   - Usable on-target bases and AT dropout also fall.
   - Across the 16 pre-TULIP production runs, duplicate fraction correlates with control precision (control ρ = −0.62, p = 0.011; patient median ρ = −0.57, p = 0.022). Patient-median usable on-target bases correlates even more strongly (ρ = +0.76, p = 0.001).
   - Because the patient samples on the same run show the same shift, this is a **batch-level library-prep effect**, not a property of the control DNA.
   - It starts with the first runs after probe lot 2500007516 came in (26-NGWES1, 19 Jan 2026, still looked normal). FREEMIX, properly paired % and MAPQ exclusion are all normal on 0701.
2. **Late Mar / Apr 2026 onwards: the harmonisation signature, layered on top.**
   - FREEMIX steps up by about 2.4×, properly paired % falls, PCT_EXC_MAPQ rises, GC dropout falls and insert size shortens.
   - This happens in every run after line 2, including those before the first multiplexed flow cell (0742, 0748, 0754). That matches the multiplexing validation's conclusion that the contamination rise comes from harmonisation.
   - Within this period, run-to-run precision is not explained by any single QC metric (all |ρ| < 0.4). The low-precision runs (ringed) are spread through the period rather than grouped at QC extremes. Several do sit at high duplication (0769: 0.146; 0870: 0.157; 0867: 0.148), but others do not (0751: 0.042).

**Implication:** the drop seems to have two stages. First, a loss of library complexity that starts at run 0701, before harmonisation and multiplexing. Second, a step down at harmonisation, linked to the masked-UMI / TULIP workflow, which pushes most later runs below the baseline −2 SD. Multiplexing itself does not add a further visible change.

---

## Section 3: Proposed mechanism and possible root causes

### Mechanism: allele-fraction distortion at indel sites
A diploid caller assigns a genotype from the fraction of reads supporting each allele: about 0.5 for het, about 1.0 for hom-alt. Anything that systematically moves the observed allele fraction (AF) at an indel causes the GT errors seen in the controls (het called as hom, hom called as het, or a different allele length). The evidence above suggests three steps in NGS library preparation that can act together:

1. **Fragments carrying an indel are captured less efficiently than reference fragments (reference bias at hybridisation).**
   - Twist probes match the reference sequence. A fragment with an indel forms a probe–target duplex with a bulge or gap, which is less stable. Its share of captured fragments then depends heavily on how stringent the hybridisation is.
   - Taking out the 95 °C pre-denaturation means double-stranded library fragments and secondary structure are not fully melted when hybridisation starts. A probe lot with weaker binding (or a hybridisation reagent stored at the wrong temperature) would have a similar effect.
   - **Effect:** a het indel's AF drops below the caller's het window, so it is called hom-ref (FN). Or the share of indel-carrying fragments becomes more variable from fragment to fragment.
   - The drop in GC dropout and the change in on/off-target show that capture conditions did change.
2. **Polymerase slippage during PCR in homopolymers and short tandem repeats.**
   - Most recurrent FP/FN loci sit in repeats: e.g. chr8:99832341 poly-T, chr7:33177459 CT, chr5:256319, FANCD2 chr3:10046723/5, INSR chr19:7184640. A different indel at the same position, and "no majority allele" (1–4 bp deletions of mixed length), are the typical signature of stutter.
   - Each extra PCR cycle (8 → 11), and lower library complexity (duplication +10%), adds stutter reads of ±1 unit. Stutter reads are split between neighbouring lengths. That lowers the AF of the true hom-alt allele, so it is called **het** (FP_GT plus an FN), and it creates new low-AF indels (the rise in non-GT FP).
3. **Template switching (chimeras) and cross-sample molecules from pooled, multiplexed capture.**
   - Under harmonisation, libraries are pooled (8 per capture) before hybridisation and amplified together after capture.
   - Template switching during post-capture PCR makes chimeric molecules. Sample-to-sample molecules and index misassignment add a small amount of foreign or recombined sequence.
   - This matches the step in FREEMIX (verifyBamID measures excess non-genotype alleles at known SNPs, so it is a direct proxy for allele-fraction "leak"), the fall in properly paired reads, and the rise in singletons and low-MAPQ reads.
   - At 0.2% this cannot flip a genotype on its own. In repeats, though, where local mapping is already uncertain and stutter has already moved the AF towards the het/hom boundary, it adds to the distortion. Chimeric reads that end inside a repeat are also soft-clipped or mis-placed, so they under-count the indel allele.

In short, **process changes widened the spread of observed AF at indels, especially in repeats.** That pushes borderline sites across the caller's het/hom thresholds and produces low-AF stutter calls. SNPs (precision 0.992 → 0.991) and non-repeat sequence are largely unaffected.

### Possible root causes (most to least likely)
| # | Root cause | Evidence for | Evidence against / caveats | Test to confirm or exclude |
|---|---|---|---|---|
| 0 | **Loss of library complexity from Feb 2026** (pre-harmonisation): reagent lot (probe, hyb or library-prep kit), input DNA amount/quality, or over-amplification | Run 0701 and neighbouring runs: duplicate fraction 0.15 (z = +4.8), library size −37%, lower usable on-target bases, in patients and control alike. Correlates with precision before TULIP (ρ = −0.57 to −0.76). Low complexity drives the PCR-jackpotting and stutter mechanisms above. | Complexity partly recovers after April while precision stays low, so it does not explain the later step. | Pull the lot numbers (library-prep kit, binding/purification beads, probes, hyb reagent) and DNA input for 0690/0701/0727. Re-prep the 0701 control with the previous lots. Add duplicate fraction and library size to TWE trend monitoring. |
| 1 | **TULIP unified library prep (SOP-32):** hyb programme without the 95 °C denaturation, enrichment PCR cycles (8 vs 11), harmonised workflow | Step in precision and in FREEMIX, pairing, MAPQ, GC dropout and duplication exactly at TULIP go-live. Also seen in CEN and MYE. | Earlier PCR-cycle comparison (0652 vs 0625) found no difference. Lines up with config v3.1.0. | Process one control DNA by SOP-14 and SOP-32 side by side, same lot and run. Confirm how many enrichment cycles are actually run for WES. |
| 2 | **Multiplexed (8-plex) pooled capture:** template switching or chimeras in post-capture PCR, cross-sample molecules, index misassignment | FREEMIX ×2.6 in all assays, more singletons and improper pairs, more low-MAPQ reads | Absolute contamination is tiny. Within-period FREEMIX does not track precision. **The precision drop starts at 0701, before multiplexing in production. The FREEMIX step comes with harmonisation, before the first multiplexed flow cell (0751). The multiplexing validation also found no extra FREEMIX change beyond harmonisation.** Now considered less likely. | Capture the control alone vs in an 8-plex pool. Check unknown/undetermined barcodes and index-hop rates (bcl-convert). Check the chimera fraction in Picard alignment metrics. |
| 3 | **Twist reagent lot or storage:** probe lot 2500007516, hyb reagent lot possibly held at the wrong temperature | Precision falls before TULIP (0690, 0701, 0727). **These runs show higher duplication and lower library size in both patients and the control (Section 2b)**, which fits a reagent-driven loss of capture or library efficiency. The lab's lot timeline matches the drops. | The first run with the new probe lot (0676) looked normal. Lots overlap with process changes. | Twist stability and freeze–thaw data. Run the same library with old and new probe/hyb lots side by side. |
| 4 | **dias config v3.1.0 (25 Mar 2026)** changed alignment, duplicate/UMI handling or QC tools | Same dates as the step. FREEMIX, MAPQ and pairing are all computed downstream of the pipeline. | Earlier pipeline changes were tested with no effect. The CEN step starts on runs sequenced after TULIP go-live. | Compare QC from `38_TWE-260702_1210` (old runs on the new pipeline) with the originals. Diff v3.1.0 against v2.5.x for the BWA, markdup/UMI and verifyBamID settings. |
| 5 | **UMI handling** (masking vs consensus grouping) | Masking UMIs raised validation-run precision to 0.694–0.733. Notes suggest grouping is better still. | Masking had no effect on 260319_…0727. | Re-run recent controls with UMI-aware consensus (e.g. Sentieon UMI) and compare FP_GT and stutter FP. |
| 6 | **Built-in repeat sensitivity** (GRCh38 move, small truth set) | GRCh38 realignment lowers precision. About 105 truth indels per case, so 1 FP ≈ 0.25% precision. All recurrent loci were present before the drop. | Explains the starting level and noise, not the step. | Stratify FP/FN by GIAB homopolymer/STR stratifications before and after. Report precision outside repeats. |
| 7 | **Control DNA degradation** (age, freeze–thaw) | A gradual decline would fit. | The same FREEMIX/QC step appears in patient samples, and nothing changes in insert size. | Make a fresh control aliquot or extraction and run it alongside the current stock. |
| 8 | **Sequencer / flow cell / base quality** | (none) | Both NovaSeqs are affected. Q30, mismatch and error rates are flat. | No further action. Keep monitoring. |

**Suggested priorities:**
1. Use the existing re-processed data to separate pipeline from wet lab (cause 4). This is cheap.
2. Check reagent lots and DNA input for 0690/0701/0727, where the library-complexity loss starts (cause 0). Then run one controlled wet-lab experiment on one control DNA that varies the SOP (with and without 95 °C, 8 vs 11 cycles), with pooling (single vs 8-plex) as a lower-priority arm (causes 1–2).
3. Add duplicate fraction, HS library size, FREEMIX, properly-paired % and FP_GT/FP to the TWE trend monitoring, as early-warning indicators alongside hap.py precision.
