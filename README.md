# Precipitation Variability and Extreme Events in the Western Mediterranean

A regional climate analysis of precipitation variability and extremes in the Western Mediterranean (35°N-47°N, 10°W-15°E) using CHIRPS v2.0 daily precipitation (1981-2023), ETCCDI extreme indices, NAO circulation analysis, non-stationary GEV models with a GMST covariate, and CMIP6 historical-vs-hist-nat attribution with the IPSL-CM6A-LR ensemble.

## The question

Are extreme precipitation events in the Western Mediterranean intensifying under anthropogenic forcing, and can that signal be separated from the region's dominant internal circulation variability (NAO, ENSO)?

## Key findings

- **Global context**: global mean land precipitation shows a significant increasing trend of +0.031 mm/day per decade (1981–2023, Mann-Kendall p = 0.001), consistent with Clausius-Clapeyron thermodynamic intensification under ~0.85°C of observed warming.
- **Regional seasonality**: a pronounced Mediterranean regime, minimum in July (1.01 mm/day), maximum in November (2.82 mm/day). Extreme events (> p95) concentrate in autumn (SON: 42.4%) and winter (DJF: 33.6%), with almost none in summer (JJA: 2.6%).
- **ETCCDI indices** (Rx1day, R95p, SDII, CDD): no statistically significant trend in any index at the regional-mean scale over 1981–2023, a statement about signal-to-noise given high NAO/ENSO/AMO-driven interannual variability, not about the absence of forcing.
- **NAO linkage**: DJF NAO explains 39.5% of DJF precipitation variance (r = −0.628, p < 0.0001). The annual-mean NAO explains only 3.8% of annual variance, the NAO's control on Mediterranean precipitation is a winter-specific phenomenon. The NAO itself shows no significant trend, ruling it out as the driver of the post-2005 dry-year clustering (6 of the 10 driest years on record occur after 2005).
- **Attribution analysis** - four methods, one coherent picture:

| Method | Result | Interpretation |
|---|---|---|
| Non-stationary GEV (observations) | μ₁ = −0.039 mm/day/°C, LR p = 0.980 | No detectable forced trend in observed Rx1day |
| Dynamical adjustment (NAO-residual) | Residual MK p = 0.786 | No thermodynamic signal after removing NAO |
| Early vs. late distribution shift | σ: 0.121 → 0.214, KS p = 0.538 | Directional increase in variability, not significant at n ≈ 21–22/period |
| CMIP6 historical vs. hist-nat (IPSL-CM6A-LR) | PR = 0.94 (median) → 1.22 (95th pct) | Competing mechanisms: drying at moderate intensities, intensification at extremes |

The CMIP6 result is the project's central finding: a **probability ratio gradient** from below 1 at moderate precipitation intensities to above 1 at the 95th percentile (FAR = 18.2%), reflecting two competing mechanisms, Hadley cell expansion and jet weakening suppressing moderate events, against Clausius-Clapeyron intensification of the events that do occur. The crossover happens between the median and 90th percentile of the historical distribution. A 95th-percentile event's return period drops from 19.7 years (counterfactual) to 16.1 years (factual), a 22% increase in recurrence frequency.

**Caveat**: the CMIP6 Rx1day values are computed from monthly (Amon) output and are therefore ~4× lower in magnitude than the daily CHIRPS Rx1day used in the observational sections, they are not on the same intensity scale and should be read as the model's internal historical-vs-hist-nat shift, not a direct confirmation of observational PR magnitudes. The qualitative PR gradient (below 1 → above 1 with increasing intensity) is the robust result; the precise PR values are conditional on the monthly proxy.

## Repository structure

```
.
├── Precipitation_Variability_Western_Mediterranean_final.ipynb
├── data/                          # not included - see Data below
│   ├── chirps-v2.0.1981.days_p25.nc
│   ├── chirps-v2.0.1982.days_p25.nc
│   ├── ...
│   ├── nao_monthly.txt             # auto-downloaded on first run
│   └── GISTEMP_global_annual.csv   # auto-downloaded on first run
├── figures/                        # written by the notebook on first run
│   ├── 01_data_check.png
│   ├── ...
│   └── 13_CMIP6_attribution.png
├── requirements.txt
└── README.md
```

## Data

Four sources are used; three are fetched automatically by the notebook, one needs to be downloaded manually.

| Source | How it's obtained | Manual step needed? |
|---|---|---|
| CHIRPS v2.0 daily precipitation (1981–2023, 0.25°) | Pre-downloaded NetCDF files read from `./data/` | **Yes** - see below |
| NOAA CPC monthly NAO index | Auto-downloaded via `pooch` from `cpc.ncep.noaa.gov` | No |
| NASA GISTEMP v4 global mean temperature | Auto-downloaded via `requests` from `data.giss.nasa.gov` | No |
| IPSL-CM6A-LR CMIP6 (historical & hist-nat) | Streamed directly from the public Pangeo Cloud Zarr catalog (Google Cloud Storage, anonymous access) | No |

**CHIRPS setup** (the one manual step):

1. Create a `data/` folder next to the notebook.
2. Download the annual daily 0.25° NetCDF files from the [CHIRPS v2.0 global daily archive](https://data.chc.ucsb.edu/products/CHIRPS-2.0/global_daily/netcdf/p25/), one file per year, 1981–2023. Files are named `chirps-v2.0.<YEAR>.days_p25.nc`.
3. Place them all directly in `data/` — the notebook locates them with `glob.glob("data/chirps-v2.0.*.days_p25.nc")`, so the naming convention above must match exactly.

A quick bash loop to grab them all:

```bash
mkdir -p data && cd data
for year in $(seq 1981 2023); do
  wget "https://data.chc.ucsb.edu/products/CHIRPS-2.0/global_daily/netcdf/p25/chirps-v2.0.${year}.days_p25.nc"
done
```

(~43 files, several hundred MB total — `data/` should stay out of version control; see `.gitignore` below.)

## Setup

Requires Python ≥ 3.9.

```
numpy
pandas
xarray
matplotlib
cartopy
scipy
pymannkendall
pooch
requests
s3fs
gcsfs
intake
intake-esm
zarr
dask
netCDF4
tqdm
```

```bash
pip install -r requirements.txt
```

`intake-esm` registers the `esm_datastore` driver used to query the Pangeo CMIP6 catalog, it's required even though only `intake` is imported directly in the notebook.

## Running

1. Populate `data/` with the CHIRPS files (see Data above) — everything else downloads automatically on first run.
2. Launch Jupyter and open `Precipitation_Variability_Western_Mediterranean_final.ipynb`.
3. `Kernel → Restart & Run All`.

The CMIP6 section streams several ensemble members directly from cloud storage, so it needs a working internet connection and will be the slowest part of a full run.

## .gitignore

```
data/
.ipynb_checkpoints/
__pycache__/
*.pyc
```

(`figures/` can stay tracked if you want the rendered PNGs visible on GitHub, or be ignored if you'd rather keep the repo lean and let the notebook's inline outputs speak for themselves.)

## Methodology

- **Region**: 35°N–47°N, 10°W–15°E, area-weighted (cosine latitude) regional mean.
- **ETCCDI indices**: Rx1day, R95p, SDII, CDD, computed with Mann-Kendall trend tests.
- **NAO analysis**: DJF and annual Pearson correlation between NAO index and Mediterranean precipitation; dynamical-adjustment residual trend analysis to isolate the circulation-independent signal.
- **Non-stationary GEV**: location parameter as a linear function of GMST, fit by maximum likelihood; likelihood-ratio test against the stationary null.
- **Early/late distribution shift**: GEV fits to 1981–2001 vs. 2002–2023 subperiods, compared via Kolmogorov–Smirnov test.
- **CMIP6 attribution**: IPSL-CM6A-LR historical vs. hist-nat ensembles (10 common members), regional Rx1day pooled across members, probability ratios computed at the model's own median/90th/95th percentile thresholds.

## Limitations

- Spatial averaging over a large domain suppresses localized extreme signals; point-scale attribution would likely show larger, more interpretable signals.
- 43 years is a short record for separating a forced drying trend from comparable-magnitude decadal internal variability.
- CDD computed on the regional-mean series measures domain-average dry spells, not station-scale drought duration, not directly comparable to standard CDD usage.
- The CMIP6 analysis uses monthly (Amon) resolution, not daily, making its Rx1day values ~4× smaller than the observational ones and limiting direct comparability.
- Single-model (IPSL-CM6A-LR), a multi-model ensemble would be needed to assess whether the PR crossover from below 1 to above 1 is robust across models.

## References

- Allen, M. R. (2003). Liability for climate change. *Nature*.
- Hurrell, J. W. (1995). Decadal trends in the North Atlantic Oscillation. *Science*.
- Philip, S. Y., et al. (2020). A protocol for probabilistic extreme event attribution analyses. *Advances in Statistical Climatology, Meteorology and Oceanography*.
- Shepherd, T. G. (2016). A common framework for approaches to extreme event attribution. *Current Climate Change Reports*.

## Author

Yassine Moudene - Applied Mathematics & Statistics (Economic Risk & Data Science), University of Bordeaux.
