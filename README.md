
<!-- README.md is generated from README.Rmd. Please edit that file -->

# MALDIpqi

<!-- badges: start -->
<!-- badges: end -->

MALDIpqi calculates Parchment Glutamine Index from MALDI TOF ZooMS data.
It is a sample level measure of the glutamine deamidation from a list of
peptides.

For details in the data processing and mathematical method and models to
estimate PQI, refer to our publication Nair et al. (2022)

The method was initially developed to estimate parchment quality based
on deamidation, following some ideas from Wilson et al. (2012). However,
it can be applied in other tissues as in van Doorn et al. (2012) or
Brown et al. (2021)

## Installation

You can install the released version of MALDIpqi from github with:

``` r
install.packages('devtools')
install.github("ismaRP/MALDIpqi")
```

## References

Nair, B. et al. (2022) ‘Parchment Glutamine Index (PQI): A novel method
to estimate glutamine deamidation levels in parchment collagen obtained
from low-quality MALDI-TOF data’, bioRxiv.
<doi:10.1101/2022.03.13.483627>.

Wilson, J., van Doorn, N.L. and Collins, M.J. (2012) ‘Assessing the
extent of bone degradation using glutamine deamidation in collagen’,
Analytical chemistry, 84(21), pp. 9041–9048.

van Doorn, N.L. et al. (2012) ‘Site-specific deamidation of glutamine: a
new marker of bone collagen deterioration’, Rapid communications in mass
spectrometry: RCM, 26(19), pp. 2319–2327.

Brown, S. et al. (2021) ‘Examining collagen preservation through
glutamine deamidation at Denisova Cave’, Journal of archaeological
science, 133, p. 105454.

## Example

``` r
data_folder = "data/mzML"
results_folder = "results_test"
```

This is minimal example of the workflow. It assumes the spectra are in
data/mzML, in mzML format and with file names in the form
“samplename_replicate.ext”. Where the replicate number is 1, 2 or 3 and
ext is the extension of the files.

``` r
library(MALDIpqi)

iso_peaks = getIsoPeaks(
  indir=data_folder, outdir=NULL, readf="mzml",
  peptides = peptides, chunks = 50, n_isopeaks = 5, min_isopeaks = 4,
  smooth_method = "SavitzkyGolay", hws_smooth = 8, halfWindowSize = 20, SNR = 1.5)


q2e = wls_q2e(peptides = peptides, n_isopeaks = 5,
              data_list = iso_peaks, outdir=results_folder)

res_free_gamma = lme_pqi(q2e, peptides=peptides, outdir=results_folder,
                         n_isopeaks=5, g="free")
res_fixed_gamma = lme_pqi(q2e, peptides=peptides, outdir=results_folder,
                          n_isopeaks=5, g=-1/2)
```
