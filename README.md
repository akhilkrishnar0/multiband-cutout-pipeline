# Multiband Cutout Pipeline

Prepare FUV, NUV, g, r and z cutouts using explicit galaxy centres. This initial package preserves the uploaded scientific code unchanged.

## Install
```python
%pip install git+https://github.com/akhilkrishnar0/multiband-cutout-pipeline.git
```
Restart the notebook kernel. An older local module with the same name can shadow the installed package.

## Run
```python
import pandas as pd
from multiband_cutout_pipeline import MultiBandCutoutPipeline
df_run = pd.DataFrame({"galaxy": galaxies_new, "distance": distances_new, "size": sizes_new}).merge(centers, on="galaxy", how="left", validate="many_to_one")
missing = df_run.loc[df_run[["ra_deg", "dec_deg"]].isna().any(axis=1), "galaxy"].tolist()
if missing:
    raise ValueError(f"Missing centres: {missing}")
pipe = MultiBandCutoutPipeline()
pipe.run(galaxies=df_run["galaxy"].tolist(), ras=df_run["ra_deg"].tolist(), decs=df_run["dec_deg"].tolist(), distances=df_run["distance"].tolist(), sizes=df_run["size"].tolist(), do_plot=True)
```

Provide the galaxy, distance and size lists and a centres table with galaxy, ra_deg and dec_deg columns. Coordinates are ICRS degrees, distances are Mpc, and sizes are square side lengths in arcseconds. Distances are used only for scale bars. The explicit-coordinate call does not use SIMBAD.

## Inputs
- `downloaded_galex/<name>/cropped/FUV_stacked.fits`
- `downloaded_galex/<name>/cropped/NUV_stacked.fits`
- `decals_data/outputs/images/<name>_g.fits` (also r/z)

Spaces are removed from names when forming file paths. Paths can be changed using galex_dir, decals_dir and out_dir constructor arguments.

## Outputs
`cutouts/<name>/` contains FUV.fits, NUV.fits, g.fits, r.fits, z.fits and, when plotting succeeds, <name>_multiband.png and <name>_RGB.png. Existing outputs are overwritten. Cropping uses Cutout2D with partial coverage padded by NaN; it does not reproject or PSF-match science images.

## Known limitations of the preserved source
- RGB images combine arrays directly without WCS alignment checks or reprojection; matching angular crops do not guarantee pixel registration.
- Response/exposure, coverage and inverse-variance maps are not cropped.
- Scale bars assume 1.5 arcsec/pixel for GALEX and 0.262 arcsec/pixel for DECaLS instead of reading the WCS.
- Missing bands and some failures are printed and skipped; stale prior cutouts may remain and be used in plots.
- Header handling replaces selected WCS keywords manually; additional distortion or higher-axis metadata may need attention.
- The optional SIMBAD resolver expects legacy uppercase RA/DEC fields and may need updating for newer astroquery responses.
- FUV is multiplied by five for the UV RGB display; display scaling is not a physical colour measurement.

Source inspected only; no tests or pipeline runs performed. This release packages the uploaded workflow; it does not claim these limitations are corrected or scientific outputs validated. No license has been selected.
