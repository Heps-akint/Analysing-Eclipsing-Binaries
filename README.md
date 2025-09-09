# Benchmarking Stars with TESS and APOGEE

## What was the project and why bother?

We wanted stars whose size and weight we know almost perfectly.  
Those *benchmark* stars are the yard-sticks that let astronomers test and tune their computer models of how stars live and age.  
If the yard-stick itself is fuzzy, every later result is fuzzy.  

Our target accuracy: **better than 2%** on both mass (how much stuff) and radius (how big).

---

## Who did what, where and when?

- **When?** January – August 2023 (my final BSc year at Keele).  
- **Where?** All work done in Python on Keele’s HPC plus a laptop; data came from two public archives.  
- **Who?** Four-person student team. I owned data cleaning, model-fitting, and error analysis for four candidate systems and advised on tooling for the others.

---

## The raw ingredients (data)

| Ingredient       | Source | What it gives | Why we picked it |
|------------------|--------|---------------|------------------|
| Light-curves (brightness vs. time) | **TESS** space telescope, Quick-Look Pipeline & SPOC archives | Shows one star passing in front of another (eclipses) | TESS covers almost the whole sky and measures brightness every 2–30 min—ideal for catching eclipses |
| Radial-velocities (Doppler shifts) | **APOGEE** infrared spectra, catalogue from Kounkel et al. 2021 | How fast each star moves toward/away from us at many times | Needed to weigh each star via Kepler’s laws. Only *double-lined spectroscopic binaries* give us two separate speed curves, so we filtered for those. |

---

## Step-by-step, from scratch

1. **Pick candidate systems**  
   - Start list: 791 binaries in APOGEE catalogue  
   - Filter: keep only those observed by TESS *and* showing deep, clean eclipses  
   - Result: 30 systems split among the team  

2. **Download & inspect light-curves (Lightkurve package)**  
   - SAP flux = raw sum of camera pixels  
   - PDC-SAP flux = same but instrument noise already corrected  
   - Always looked at both to spot problems  

3. **Clean the brightness data**  
   - Remove flagged bad points (cosmic rays, thruster firings)  
   - Flatten each segment (remove long drift & star-spot wiggles)  
   - Trim sector ends where Earth-shine hurts data  
   - Stitch sectors → one continuous series per star  

4. **Find the eclipses**  
   - Visual check + periodogram → confirm orbital period  

5. **Build a physics model (JKTEBOP)**  
   - Parameters: orbital period P, inclination i, fractional radii, temperature ratio, limb darkening, eccentricity e, argument of periastron ω  
   - Optimiser: Levenberg–Marquardt  
   - Errors: 1,000 Monte-Carlo trials (add noise → refit → distributions for each parameter)  

6. **Radial-velocity fitting**  
   - Fit sinusoids (or Kepler curve if eccentric) to find amplitudes K1, K2  
   - Combine with inclination i → absolute masses (M1, M2) and orbital separation a  
   - Multiply fractional radii by a → absolute radii (R1, R2)  

7. **Quality test: the 2% rule**  
   - Keep only systems with ≤2% error bars on every mass and radius  

8. **Cross-check with stellar-evolution tracks**  
   - Plot each star on a MIST isochrone diagram (radius vs. mass, coloured by age)  
   - If the point sits on a sensible age line → pipeline is sane  

---

## Typical result (one of my four systems)

**Star: FL Lyr**

- Masses: 1.153 ± 0.009 M☉ and 0.893 ± 0.007 M☉ (0.8% and 0.7%)  
- Radii: 1.184 ± 0.005 R☉ and 1.012 ± 0.005 R☉ (0.4% each)  
- χ² per data point ≈ 1.04 (good fit)  
- Isochrone age: primary ~5.6 Gyr, secondary consistent  

From my four systems: 2 passed the 2% threshold.  
Team-wide: 14 systems passed.

---

## Why each choice was made

- **TESS** → uninterrupted, space-quality light (no clouds, no day-night gaps)  
- **APOGEE** → infrared spectra of cooler stars → sharp lines, low RV errors  
- **Lightkurve** → automates downloads, filtering, flattening  
- **Flattening** → removes slow rolls in brightness (not eclipses), keeps depths true  
- **JKTEBOP** → fast, proven, assumes near-spherical stars (we excluded distorted ones)  
- **Monte-Carlo** → safer than covariance matrix, captures non-linear quirks  
- **2% cut-off** → from Torres (2010); below that, model errors dominate, so tighter has real value  

---

## What could trip us up (and how we handled it)

| Risk | Counter-measure |
|------|-----------------|
| Instrument drift or Earth-shine | Trim edges; compare SAP vs PDC-SAP |
| Star-spots mimicking shallow eclipses | Wide-window flattening; visual residual checks |
| Local-minimum fits | Run JKTEBOP multiple times with jittered starts |
| Circular orbit assumption when eccentric | Fit RVs first; if e > 0.05 rerun with fixed e |
| Over-confidence in errors | 1,000 Monte-Carlo iterations (not default 200) |

---

## Why the project matters (big picture)

1. **Stellar physics** – Precise benchmarks anchor age & composition scales across astrophysics (galaxy history, exoplanet atmospheres, etc.)  
2. **Methodology** – Shows how to wrangle huge, messy time-series into reliable numbers. Transferable to fields like smart-meters & battery data.  
3. **Open science** – All code notebooks and cleaned data are on GitHub for full reproducibility.

---

## Contents (for README navigation)

- [What was the project and why bother?](#what-was-the-project-and-why-bother)  
- [Who did what, where and when?](#who-did-what-where-and-when)  
- [The raw ingredients (data)](#the-raw-ingredients-data)  
- [Step-by-step, from scratch](#step-by-step-from-scratch)  
- [Typical result (one of my four systems)](#typical-result-one-of-my-four-systems)  
- [Why each choice was made](#why-each-choice-was-made)  
- [What could trip us up](#what-could-trip-us-up-and-how-we-handled-it)  
- [Why the project matters](#why-the-project-matters-big-picture)
