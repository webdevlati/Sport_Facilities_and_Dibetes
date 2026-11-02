# Sports Facilities and Diabetes Clinic Encounters in Saudi Arabia

A Python data analytics case study testing whether regions with more sports facilities have fewer (or more) diabetes clinic encounters.

**Short answer:** Using raw counts, the two appear positively correlated. That is a population-size effect. Once both measures are adjusted for population, the correlation falls to near zero and is not statistically significant. **No association is detectable in this data.**

---

## Table of contents

1. [Research question](#research-question)
2. [Data sources](#data-sources)
3. [Methodology](#methodology)
4. [Key findings](#key-findings)
5. [Visualizations](#visualizations)
6. [Limitations](#limitations)
7. [Possible next steps](#possible-next-steps)
8. [How to run](#how-to-run)
9. [Tech stack](#tech-stack)

---

## Research question

Is there a correlation between the availability of sports facilities in a region and the number of diabetes clinic visits in that region?

The analysis is at the level of Saudi Arabia's **13 administrative regions**.

## Data sources

| Dataset | Source | Content | Raw size |
|---|---|---|---|
| Encounters to Diabetic Clinics, MOH Hospitals | Ministry of Health (MOH) | Encounters by health region, split by Saudi/non-Saudi and gender, with a total | 20 health regions + 1 total row |
| Sports facilities (`data.xlsx`) | Ministry of Sport (MOS) | One row per sub-facility (hall, pitch, track...) with facility, city and region names, all in Arabic | 773 rows |
| Population by administrative region, 2022 | GASTAT, Saudi Census 2022 | Total population (Saudi + non-Saudi) per region | 13 regions, 32,175,224 total |

The census figures were taken from the GASTAT tables as republished by the Gulf Labour Markets, Migration and Population (GLMM) programme. The 13 regional values sum to the national total.

## Methodology

### 1. Cleaning
- **MOH:** standardized column names, dropped the grand-total row (so it isn't counted as a region), and confirmed that the four demographic columns add up to the `Total` column for every row.
- **MOS:** removed 3 fully duplicated rows (773 to 770 sub-facilities).

### 2. Harmonizing regions
The two datasets use different regional systems. MOH reports **20 health regions**, MOS reports **13 administrative regions**. The 20 MOH regions were mapped to the 13 administrative regions, and the Arabic MOS region names were mapped to the same English names.

| Administrative region | MOH health regions merged into it |
|---|---|
| Makkah | The Holy Capital, Jeddah, Ta'if, Qunfudah |
| Eastern | Eastern, Al-Ahsa, Hafr Al-Baten |
| Aseer | Aseer, Bishah |
| Al-Jouf | Al-Jouf, Qurrayat |
| Riyadh, Madinah, Qassim, Tabuk, Hail, Northern Borders, Jazan, Najran, Al-Baha | One-to-one |

Checks after the merge: no unmapped names in either dataset, all 13 regions matched on both sides, and total encounters still equal the original MOH total of **617,377**.

### 3. Aggregation
- **Encounters:** summed per administrative region.
- **Facilities:** two measures per region: the number of sub-facilities (rows) and the number of distinct facilities.

### 4. Normalizing by population
Raw counts mostly reflect how many people live in a region. Two per-capita measures were created:
- `encounters_per_1000` = diabetes encounters / population x 1,000
- `facilities_per_100k` = sports sub-facilities / population x 100,000

### 5. Statistical tests
Both **Pearson** (linear relationship, sensitive to outliers) and **Spearman** (rank-based, more robust) correlations were calculated. Tests were run on raw counts, on per-capita rates, and on per-capita rates with Makkah and Madinah excluded as a robustness check.

## Key findings

### Finding 1: Raw counts show a positive correlation

| Comparison | Pearson r (p) | Spearman rho (p) |
|---|---|---|
| Sub-facilities vs encounters | 0.56 (0.046) | 0.67 (0.012) |
| Distinct facilities vs encounters | 0.49 (0.092) | 0.71 (0.007) |

On the surface, regions with more facilities also have more diabetes visits, the opposite of "more facilities, less diabetes".

### Finding 2: Population size explains this

Both variables rise strongly with population (Spearman rho against population: 0.75 for encounters, 0.77 for sub-facilities). Large regions have more people, more facilities and more clinics. The raw correlation was confounded by region size.

### Finding 3: After adjusting for population, the correlation disappears

| Analysis (13 regions unless stated) | Pearson r (p) | Spearman rho (p) |
|---|---|---|
| Per capita | 0.31 (0.302) | 0.13 (0.681) |
| Per capita, excluding Makkah and Madinah (n = 11) | 0.20 (0.548) | 0.09 (0.790) |

Neither result is statistically significant, and the Spearman values are close to zero.

### Finding 4: The result is robust to excluding Makkah and Madinah

These two regions were checked because their clinic counts could include visits by Hajj and Umrah pilgrims, who are not in the census population. Excluding them leaves the conclusion unchanged. In fact, both have low per-capita encounter rates (16.8 and 11.0 per 1,000), so they were not inflated in this data.

### Finding 5: Per-capita diabetes encounters vary widely between regions

| Region | Encounters per 1,000 | Sub-facilities per 100,000 |
|---|---|---|
| Al-Baha | 62.0 | 5.3 |
| Al-Jouf | 55.4 | 1.8 |
| Aseer | 47.7 | 1.6 |
| Northern Borders | 43.1 | 2.4 |
| Qassim | 38.1 | 5.2 |
| Jazan | 35.8 | 1.2 |
| Hail | 35.8 | 5.1 |
| Najran | 35.6 | 3.5 |
| Makkah | 16.8 | 1.6 |
| Tabuk | 14.4 | 1.7 |
| Madinah | 11.0 | 2.0 |
| Eastern | 9.9 | 3.7 |
| Riyadh | 9.4 | 2.0 |

The highest rates are in smaller regions (Al-Baha, Al-Jouf, Aseer), and the lowest are in the largest regions (Riyadh, Eastern). The facility rate does not line up with this pattern (for example, Al-Baha and Qassim have both high facility rates and high encounter rates). Differences in clinic access, reporting or care-seeking between regions probably matter far more than facility counts, but this dataset can't test that.

### Conclusion

**No detectable association between sports facility availability and diabetes clinic encounters at the regional level.** This does not prove that no relationship exists. With only 13 regions, the analysis can only detect fairly strong correlations (roughly |r| above 0.55), so a weak or moderate real effect could be hidden.

## Visualizations

The notebook includes seven charts, in order:

1. MOH encounters by health region (20 regions)
2. Raw encounters and facilities by administrative region
3. Encounters and facilities against population (shows the confounding)
4. Per-capita encounters and facilities by region
5. **Main result:** per-capita scatter plot with trend line (bubble size = population)
6. Robustness check: scatter plot with Makkah and Madinah highlighted and excluded from the trend line
7. Summary bar chart: raw vs per-capita Spearman correlation

## Limitations

- **Small sample:** only 13 regions, so low statistical power.
- **Encounters are not prevalence.** They depend on how many MOH clinics and doctors a region has, and they exclude private-sector care.
- **Facility counts are crude.** The data says nothing about facility size, quality, cost or public access, and counting sub-facilities treats a small hall and a main stadium pitch the same way.
- **Ecological fallacy:** a region-level result can't show how individuals behave.
- **Correlation is not causation.** Even a clear result couldn't show that facilities prevent or cause diabetes.
- **Time mismatch (to check):** population is from the 2022 census. The year covered by the MOH encounters data should be confirmed and stated here: `[add year]`.
- **Region mapping:** MOH health regions were mapped to administrative regions by geography. Boundaries may not match exactly.
- **Population definition:** census figures count residents and exclude Hajj, Umrah and tourist visitors.
- **No control variables:** age structure, urbanization, income, clinic capacity and other factors were not controlled for.

## Possible next steps

- Use city- or governorate-level data for many more data points.
- Add diabetes prevalence from national health surveys instead of clinic encounters.
- Control for age structure, urbanization and clinic capacity with a multiple regression.
- Weight facilities by type or size, or measure public access.
- Use the Saudi/non-Saudi and gender splits already in the MOH data for subgroup analysis.

## How to run

1. Put the notebook and the two Excel files in the same folder (or edit the paths in the first code cell).
2. Install the dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn scipy openpyxl
   ```
3. Open `Sport_Facilities_and_Dibetes.ipynb` in Jupyter and run all cells from top to bottom.

**Files**

| File | Purpose |
|---|---|
| `Sport_Facilities_and_Dibetes.ipynb` | Full analysis notebook |
| `Encounters to Diabetic Clinics MOH Hospitals.xlsx` | MOH data |
| `data.xlsx` | MOS sports facilities data |

## Tech stack

Python, pandas, NumPy, SciPy (Pearson and Spearman correlation), Matplotlib, Seaborn, Jupyter Notebook.

## Author

`[Your name]`
