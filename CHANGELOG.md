# Changelog

## 2026-10-27

### Photomeson production of nuclei: empirical model tables (PhotoPionProductionEmpirical)

#### Tables
- Added `tables/PhotoPionProductionEmpirical/` with the three photon-field-independent tables of the empirical photomeson model (Morejon et al., JCAP 11 (2019) 007, arXiv:1904.07999), exported from AstroPhoMes v0.4-15-g315809a (superheavy extension, A≤257, Z≤99):
  - `basis.txt`: 159-point eps_r grid with the cross-section basis vectors
  - `redistribution.txt`: SOPHIA secondary x-spectra (2312 rows, stored as truncated cumulative distributions, energy-renormalised)
  - `fragments.txt`: exclusive fragmentation channels (164,519 channels for 2008 mother nuclei; Fe-56 ⟨dA⟩ = 5.79)
- Added `tables/PhotoPionProductionEmpirical/README.txt` with the source and citation
- The table format is specified in `DATA_GENERATION.txt`; the exporter script (`export_tables.py`) is maintained in the AstroPhoMes repository

#### calc_all.py
- Added `empirical_photopion()`, called from `createCRPropaDefault()`, which copies the three tables unchanged into `data/PhotoPionProductionEmpirical`
- No `rate_<field>.txt` files are produced: the C++ module in the CRPropa trunk builds the interaction rate from the photon field at run time

#### Repository
- Removed the `lib/AstroPhoMes` git submodule (and `.gitmodules`), the `dataGeneration/` exporter and the `submodules: true` checkout option in the CI workflow; the tables are now committed directly, following the convention for other external models in `tables/`

## 2026-06-22

### Nuclear decay and mass tables: NuDat2 → NuDat3, extended to Pb-214

#### Tables
- Added `tables/decay_NuDat3.txt` — nuclear decay data from BNL NuDat3 (4000 entries, Z=0–82, A≤214)
- Added `tables/gamma_NuDat3.txt` — gamma emission data from BNL NuDat3 (65207 entries, Z=0–82)
- Removed `tables/decay_NuDat2.txt` and `tables/gamma_NuDat2.txt`

#### calc_decay.py
- Updated input files from `decay_NuDat2.txt` / `gamma_NuDat2.txt` to their NuDat3 counterparts
- Extended nuclear coverage from Fe-56 (Z≤26, N≤30) to Pb-214 (Z≤82, N≤132) via new constants `ZMAX=82` and `NMAX=132`; all loop ranges and table allocations updated accordingly
- Refactored `NuclearMassTable`: replaced flat array indexed by `Z*31+N` with a `dict` keyed by `(Z, N)`, removing the hard upper-bound on Z and N at lookup time
- Made excited-state detection robust: NuDat3 uses non-numeric labels (e.g. `'0.0+X'`) where NuDat2 used `'0+X'`; parsing now uses `try/except ValueError` instead of a string prefix check
- Removed hardcoded workarounds for Li-5 (redundant alpha row) and He-5 (redundant alpha row): NuDat3 already omits these physically redundant entries
- Removed hardcoded workaround for B-12 mode rename (B3A→B2A): NuDat3 already uses B2A
- Retained hardcoded workarounds for Fe-45 and K-46, which are still needed with NuDat3 data
- Added skip for spontaneous fission (SF) decay mode, which appears in NuDat3 but is not modelled in CRPropa
- Corrected comment for V-50: decay via electron capture (EC), not beta-/beta+

#### calc_mass.py
- Extended mass table from Z=0–26, N=0–30 to Z=0–82, N=0–132 (837 → 11,039 entries)
- Added NuDat3 mass excess (column 6, in keV) as a secondary mass source, filling isotopes not covered by NIST; priority order: NIST atomic mass > NuDat3 mass excess > A·amu approximation
- Output format now includes Z and N columns (`Z N mass`) instead of a positional flat array, making the table self-describing and compatible with the refactored `NuclearMassTable` lookup
- Added imports for `keV` and `c_squared` from `units.py`

#### Impact on output (nuclear_decay.txt, nuclear_mass.txt)
- Isotopes in decay table: 770 → 10,714
- Decay channels: 947 → 11,552
- 265 existing channels have updated lifetimes (e.g. neutron: 881.5 s → 885.7 s)
- 38 existing channels have updated branching ratios
- 2 decay modes newly present in NuDat3 with no handler: `B-P` (Be-11), `B5N` (Na-35) — currently skipped with "unknown decay mode" warning
