# Changes (March 9, 2025)
* Changed peak calling behavior.
    * Using relative proportion of dips is **intuitive but can be difficult to explain**.
        * Switch to z-scores, the distance from the mean in standard deviations.
        * We assume reads are normally distribution in evaluated regions.
    * The relative proportion also had issues as it **required determining a variable, relative height**.
        * Depending on `bp_edges` and the deviation in methylation, this relative height could be inflated.
    * Operating per bin and using a standardized threshold across the entire region resolves many of these issues.
* Add 1D sequence similarity as track and a method to reduce false-positives.
    * Previous iterations of CDR-Finder could achieve ~90% TP rate.
    * The remaining false-positives resided in pericentromeric regions with high variability in methylation and repeat type.
        * Mostly due to transposable elements interspersed in alpha-satellite repeats.
    * Because of this variability, sequence identity drops rapidly and can be used to restrict evaluated ALR regions.

# Changes (August 18, 2024)
* Added basic test files with Git LFS
* Added plot cdr R script.
* Added output, log, and benchmark dir.
* Remove singularity for rules.
* Rewrote `calculate_windows.py` as unbearably slow.
    * Use `intervaltree` library for faster overlap detection.
    * Remove `pandas` and `numpy`.
    * Remove slide argument as unnecessary and leads to undefined behavior

    ```bash
    time python workflow/scripts/calculate_windows_test.py         --target_bed test/bed/CHM13_cen_500kbp.bed         --methylation_tsv results/bed/CHM13_subset.bed         --window_size 5000      -p 1 > /dev/null
    ```
    ```
    real    0m12.428s
    user    0m10.869s
    sys     0m0.208s
    ```

    ```bash
    time python tests/scripts/calculate_windows.py         --target_bed test/bed/CHM13_cen_500kbp.bed         --methylation_tsv results/bed/CHM13_subset.bed         --window_size 5000         --slide 5000  > /dev/null
    ```
    ```
    real    22m59.106s
    user    22m34.406s
    sys     0m21.116s
    ```
* Use `modkit` instead of `modbam2bed`.
    * `modkit` produces identical results and is actively supported by ONT.
* Rework CDR detection script to not use custom valley detection algorithm and rely on pre-existing well-tested libraries. Using the [`scipy.signals`](https://docs.scipy.org/doc/scipy/reference/signal.html) library, we can achieve similar results out of the box.
    * Remove confidence.
    * Changed parameters so more robust thresholds. Rather than a static `low_threshold` of `39`, two params are used:
        * `thr_height_perc_valley`
            * Threshold percent of the average methylation percentage needed as the minimal height of a valley from the median.
                * ex. 0.1 allows valleys with a height from the median equal to 10% of the median.
            * *How deep should this valley be in the context of the median methylation in this region?*
        * `thr_prom_perc_valley`
            * Threshold percent of the average methylation percentage needed as the minimal prominence of a valley from the median.
                * ex. 0.1 allows valleys with a prominence from the median equal to 10% of the median.
            * See https://en.wikipedia.org/wiki/Topographic_prominence.
            * *How prominent is this valley in the context of the median methylation in this region?*
            * Helps in removing low-confidence CDRs at the edges.
