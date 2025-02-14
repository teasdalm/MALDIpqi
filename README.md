
<!-- README.md is generated from README.Rmd. Please edit that file -->

# MALDIpqi

<!-- badges: start -->

[![DOI](https://zenodo.org/badge/436219305.svg)](https://zenodo.org/badge/latestdoi/436219305)
<!-- badges: end -->

MALDIpqi calculates Parchment Glutamine Index from MALDI TOF ZooMS data.
It is a sample level measure of the glutamine deamidation from a list of
peptides.

For details in the data processing and mathematical method and models to
estimate PQI, refer to our publication Nair et al. (2022)

The method was initially developed to estimate parchment quality based
on deamidation, following the ideas from Wilson et al. (2012). However,
it can be applied in other tissues as in van Doorn et al. (2012) or
Brown et al. (2021)

## Installation

You can install the released version of MALDIpqi from github with:

``` r
install.packages('devtools')
install.github("ismaRP/MALDIpqi")
```

## Example

``` r
data_folder = "data/mzML"
results_folder = "results_test"
```

This is minimal workflow. It assumes the spectra are in data/mzML, in
mzML format and with file names in the form “samplename_replicate.ext”.
Where the replicate number is 1, 2 or 3 and ext is the extension of the
files.

``` r
library(MALDIpqi)

params = list(SNR=1.5, iterations=150, hws_smooth=8, halfWindowSize=20)

iso_peaks = getIsoPeaks(
  indir=data_folder, outdir=NULL, readf="mzml",
  peptides_user = NULL, nchunks = 50, ncores = 1, iocores = 1,
  n_isopeaks = 5, min_isopeaks = 4, iterations=150,
  smooth_method = "SavitzkyGolay", hws_smooth = 8, halfWindowSize = 20, SNR = 1.5)

q2e = wls_q2e(peptides_user = NULL, n_isopeaks = 5,
              data_list = iso_peaks, outdir=results_folder)

# EStimate free gamma model or fixed to -1/2
res_free_gamma = lme_pqi(q2e, outdir=results_folder,
                         logq = T, n_isopeaks=5, g="free")
res_fixed_gamma = lme_pqi(q2e, outdir=results_folder,
                          logq = T, n_isopeaks=5, g=-1/2)
```

If you want to run the samples using the inferred parameters from the
Orval Dataset. You can skip the preprocessing of this dataset, which is
the most time consuming and get the pre-computed isotopic peaks using
`get_ref_isopeaks()`. This has been done for different combinations of
parameters, that you can see with

``` get_ref_isopeaks(which_params)```

Then get the q2e and the linear mixed effect model


```r
# Extract reference iso_peaks calculated with parms
iso_peaks_orval = get_ref_isopeaks(params=params)

# Calculate q2e
q2e_orval = wls_q2e(peptides_user = NULL, n_isopeaks = 5,
                    data_list = iso_peaks_orval)
# Estimate LME model
pqi_orval = lme_pqi(q2e_orval, logq = T, g = -1/2, return_model = T)

# Get q2e from new data and log transform
q2e_new = q2e %>% mutate(resp=log(q))
pqi_ucc = predict_pqi(
  q2e_new, estimates = pqi_orval$estimates, model = pqi_orval$model,
  logq=T) # Here logq=T indicates that new data and model are already log transformed
```

## References

Nair, B. et al. (2022) ‘Parchment Glutamine Index (PQI): A novel method
to estimate glutamine deamidation levels in parchment collagen obtained
from low-quality MALDI-TOF data’, bioRxiv.
<doi:10.1101/2022.03.13.483627>.

Wilson, J., van Doorn, N.L. and Collins, M.J. (2012) ‘Assessing the
extent of bone degradation using glutamine deamidation in collagen’,
Analytical chemistry, 84(21), pp. 9041–9048.
<https://doi.org/10.1021/ac301333t>

van Doorn, N.L. et al. (2012) ‘Site-specific deamidation of glutamine: a
new marker of bone collagen deterioration’, Rapid communications in mass
spectrometry: RCM, 26(19), pp. 2319–2327.
<https://doi.org/10.1002/rcm.6351>

Brown, S. et al. (2021) ‘Examining collagen preservation through
glutamine deamidation at Denisova Cave’, Journal of archaeological
science, 133, p. 105454. <http://doi.org/10.1016/j.jas.2021.105454>

Bethencourt, J.H. et al. (2022) ‘Data from “A biocodicological analysis
of the medieval library and archive from Orval abbey, Belgium”’, Journal
of open archaeology data, 10(0). Available at:
<https://doi.org/10.5334/joad.89>.

Ruffini-Ronzani, N. et al. (2021) ‘A biocodicological analysis of the
medieval library and archive from Orval Abbey, Belgium’, Royal Society
Open Science, 8(6), p. 210210. <https://doi.org/10.1098/rsos.210210>
