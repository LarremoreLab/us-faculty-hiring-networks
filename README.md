# us-faculty-hiring-networks
This repository contains the data described in [*Quantifying hierarchy and dynamics in U.S. faculty hiring and retention*](https://doi.org/10.1038/s41586-022-05222-x) (Wapman, Zhang, Clauset & Larremore, *Nature*, 2022; see also the [Author Correction](https://doi.org/10.1038/s41586-023-06379-9), *Nature*, 2023).

There are four datasets included here:

1. edge lists for all faculty hiring networks (`data/edge-lists.csv`)
2. prestige ranks, production ranks, and attrition counts for the institutions in all faculty hiring networks (`data/institution-stats.csv`)
3. miscellaneous stats, like Gini coefficients, for all faculty hiring networks (`data/stats.csv`)
4. yearly stats (Gini coefficients and fraction of women, by year) for all faculty hiring networks (`data/yearly-stats.csv`)

> **August 2026 update.** All four files were regenerated from the original analysis pipeline to fix several bugs in the *export* step — most importantly, ranked institutions were missing from `institution-stats.csv` (our thanks to Todd Jones and Alex Bell for reporting and diagnosing this). The underlying data, sample, and ranking procedure are unchanged. See the [changelog](#changelog) at the bottom of this file.

## Using the data

Because all of our analyses were done at three scopes (across academia; per domain, Natural Sciences for example; and per field, Computer Science for example), all of the csvs above have two columns (plus other columns that differ from dataset to dataset):

1. `TaxonomyLevel`: values in this column indicate the level of scope (`Academia`/`Domain`/`Field`) for a given row.
2. `TaxonomyValue`: values in this column indicate the subset of that scope (e.g., 'Computer Science') for a given row.

So, for example, if you want to see some information for, say, the field of Computer Science, you would take the data (assuming you've read it into a pandas dataframe) and do this:

```
df = df[
  (df['TaxonomyLevel'] == 'Field')
  &
  (df['TaxonomyValue'] == 'Computer Science')
]
```

If, instead, you wanted to see information for academia as a whole, you would take the data (again assuming you've read it into a pandas dataframe) and do this:

```
df = df[
  (df['TaxonomyLevel'] == 'Academia')
  &
  (df['TaxonomyValue'] == 'Academia')
]
```

Two things to keep in mind when aggregating across scopes:

- A department that belongs to two fields (e.g., a Department of Physics and Astronomy) contributes its faculty to both fields' networks, but only once to the enclosing domain's network and once to academia's. Some departments also belong to "fields" that were too small or incoherent to be analyzed as fields; their faculty still appear at the domain and academia levels. So field-level totals sum to more than domain-level totals, which sum to more than academia-level totals.
- The data are resolved to the institution, not the department: if an institution has several departments in the same field, they are combined.

# edge lists

The csv containing the faculty hiring network edge lists (`data/edge-lists.csv`) has one row per (employing institution, doctoral institution) pair within each faculty hiring network, with the following columns:

- `TaxonomyLevel`: String. The scope of the faculty hiring network that this edge is a part of.
- `TaxonomyValue`: String. The subscope of the faculty hiring network that this edge is a part of.
- `InstitutionId`: Integer. A unique identifier for the institution that employs the faculty.
- `InstitutionName`: String. The name of the institution that employs the faculty.
- `DegreeInstitutionId`: Integer. A unique identifier for the institution that granted the faculty's doctorates.
- `DegreeInstitutionName`: String. The name of the institution that granted the faculty's doctorates.
- `Total`: Integer. The number of faculty employed at `InstitutionName` who received their doctorate from `DegreeInstitutionName`, within the particular subset `TaxonomyValue` of the scope `TaxonomyLevel`.
- `Men`: Integer. Same as `Total`, but counting only men.
- `Women`: Integer. Same as `Total`, but counting only women. Faculty whose gender could not be determined are included in `Total` but in neither `Men` nor `Women`, so `Total` can exceed `Men + Women`.

Notes:

- The edge lists include every faculty member in the sample who has a recorded doctoral institution — including faculty whose doctorate is from a non-US institution (such institutions appear only as `DegreeInstitution`s). The prestige rankings in `institution-stats.csv` are computed on the sub-network of US PhD-granting employing institutions (see below), so many `DegreeInstitution`s in this file are not ranked.
- Faculty with no recorded doctoral institution are not in this file (they are counted in the `NoDoctorate` column of `stats.csv`).

For example, for a row with this information:

- `Total`: 3
- `Women`: 2
- `Men`: 1
- `DegreeInstitutionId`: 1
- `DegreeInstitutionName`: "University of New Mexico"
- `InstitutionId`: 2
- `InstitutionName`: "University of Colorado, Boulder"
- `TaxonomyValue`: "Computer Science"
- `TaxonomyLevel`: "Field"

means that the University of Colorado, Boulder employs 3 faculty who received their PhD from the University of New Mexico in the field of Computer Science, and that 1 of those faculty is a man, and 2 are women.

If you wanted to get the faculty hiring network for, say, the field of Ecology, you would do something like this (assuming your working directory is the root of this repository):
```
import pandas as pd

df = pd.read_csv('data/edge-lists.csv')
df = df[
  (df['TaxonomyLevel'] == 'Field')
  &
  (df['TaxonomyValue'] == 'Ecology')
]
```

# institution stats

The csv containing institution-level information can be found at `data/institution-stats.csv`. It has one row for every institution that is ranked in a given faculty hiring network (12,669 rows across 116 networks: academia, 8 domains, and 107 fields), with the following columns:

- `InstitutionId`: Integer. A unique identifier for the institution (the same ids as in `edge-lists.csv`).
- `InstitutionName`: String. The name of the institution.
- `TaxonomyLevel`: String. The scope of the faculty hiring network.
- `TaxonomyValue`: String. The subscope of the faculty hiring network.
- `PrestigeRank`: Float. The institution's SpringRank score in this network, divided by the maximum score in the network, so that it lies between 0 and 1. **1 indicates the most prestigious institution in the network; 0 the least prestigious.**
- `OrdinalPrestigeRank`: Integer. The ordinal version of `PrestigeRank`: 0 is the most prestigious institution in the network, and n−1 the least prestigious in a network of n ranked institutions. Contiguous within each network.
- `ProductionRank`: Integer. The ordinal rank of the institution (0 = most) by the number of faculty it produced who are employed in this network. This ranking is computed among *all* institutions that produced at least one faculty member employed in the network — including institutions that are not themselves in the prestige ranking (e.g., non-US institutions) — so the values that appear in this file are not contiguous. It is blank for ranked institutions that produced no faculty employed in the network (institutions that only employ).
- `NonAttritedFaculty`: Integer, **unit: faculty**. The number of faculty at risk (see *Units* below) who received their doctorate from the institution and are employed in this network, and who did **not** leave the sample.
- `AttritedFaculty`: Integer, **unit: faculty**. The number of those faculty who left the sample.
- `NonAttritionEvents`: Integer, **unit: faculty-years**. The number of faculty-years at risk, among those same faculty, in which the faculty member did **not** leave.
- `AttritionEvents`: Integer, **unit: faculty-years**. The number of faculty-years at risk in which the faculty member left. Because a faculty member leaves at most once, `AttritionEvents` always equals `AttritedFaculty`; both are given so that each unit is complete on its own.

**Units.** Attrition is reported in two units, following the paper's Methods. Whether a faculty member left in a given year is identified by their absence in the following year, so every year a faculty member is observed — except the final year of the sample (2020) — is one *faculty-year at risk*; a faculty member is *at risk* if they have at least one such year (faculty first observed in 2020 are therefore not counted). A faculty member *attrited* if their (last) departure from the sample happened while they were employed in this network; faculty who change disciplines are not counted as attritions from the discipline they left. Hence:

- `NonAttritedFaculty + AttritedFaculty` is the number of faculty at risk, and `AttritedFaculty / (NonAttritedFaculty + AttritedFaculty)` is the share of those faculty who left during 2011–2020;
- `NonAttritionEvents + AttritionEvents` is the number of faculty-years at risk, and `AttritionEvents / (NonAttritionEvents + AttritionEvents)` is the **average annual attrition risk**, as defined in the paper.

For example, MIT-trained faculty across academia: 4,827 faculty at risk (3,548 stayed, 1,279 left: a 26.5% share over the decade) contributed 31,537 faculty-years at risk (30,258 without attrition, 1,279 with: a 4.1% annual risk).

Attrition is counted on the sample used for the paper's attrition and retention analyses — faculty in departments that are observed in every year of the sample — which is a subset of the sample used to compute the rankings. A ranked institution can therefore have no faculty at risk in a network (all four columns 0).

Institutions that did not meet our criteria for inclusion in a given faculty hiring network are not ranked, though it is possible that they nonetheless have a program within the bounds of that particular faculty hiring network. Briefly: a network's nodes are the US PhD-granting institutions that employ faculty in the network, after iteratively removing doctoral institutions that do not themselves employ faculty in the network. For more information on our inclusion criteria for rankings, see the supplement.

For example, for a row with this information:

- `OrdinalPrestigeRank`: 0
- `InstitutionId`: 1
- `InstitutionName`: "Puma University"
- `TaxonomyValue`: "Felinology"
- `TaxonomyLevel`: "Field"

means that Puma University is the most prestigious institution in the field of Felinology.

If you wanted to see the 5 most prestigious institutions across academia, you would do something like this (assuming your working directory is the root of this repository):

```
import pandas as pd

df = pd.read_csv('data/institution-stats.csv')

df = df[
  (df['TaxonomyLevel'] == 'Academia')
  &
  (df['TaxonomyValue'] == 'Academia')
]

df = df.sort_values(by=['OrdinalPrestigeRank'])
print(df.head())
```
# yearly stats

The csv containing stats computed for multiple years can be found at `data/yearly-stats.csv`, and has the following columns:

- `TaxonomyLevel`: String. The scope of the faculty hiring network.
- `TaxonomyValue`: String. The subscope of the faculty hiring network.
- `Year`: Integer. The year of the stat.
- `GiniCoefficient`: Float. The Gini coefficient of faculty production for that year.
- `FractionFemale`: Float. The fraction of faculty who are women in that year.

# academia/domain/field-level stats

The csv containing the miscellaneous stats at the academia/domain/field-level (`data/stats.csv`) has the following columns:

- `TaxonomyLevel`: String. The scope of the faculty hiring network.
- `TaxonomyValue`: String. The subscope of the faculty hiring network.
- `NoDoctorate`: Integer. The count of faculty with no recorded doctoral institution.
- `Doctorate (US)`: Integer. The count of faculty with a US doctorate.
- `Doctorate (Non-US)`: Integer. The count of faculty with a non-US doctorate.
- `Doctorate (Canada/UK)`: Integer. The count of faculty with a doctorate from Canada or the UK.
- `Doctorate (Africa)`, `Doctorate (Asia)`, `Doctorate (Europe)`, `Doctorate (North America)`, `Doctorate (Oceania)`, `Doctorate (South America)`: Integer. The count of faculty with a doctorate from an institution on that continent.
- `NonAttritedFaculty (US)`, `AttritedFaculty (US)`, `NonAttritionEvents (US)`, `AttritionEvents (US)`: Integer. Attrition counts for faculty with a US doctorate, in the two units described under [institution stats](#institution-stats): faculty (`…Faculty`) and faculty-years at risk (`…Events`); `AttritedFaculty` and `AttritionEvents` are the same number.
- `NonAttritedFaculty (Canada/UK)`, `AttritedFaculty (Canada/UK)`, `NonAttritionEvents (Canada/UK)`, `AttritionEvents (Canada/UK)`: Same, for faculty with a doctorate from Canada or the UK.
- `NonAttritedFaculty (non US/Canada/UK)`, `AttritedFaculty (non US/Canada/UK)`, `NonAttritionEvents (non US/Canada/UK)`, `AttritionEvents (non US/Canada/UK)`: Same, for faculty with a doctorate from somewhere other than the US, Canada, or the UK. Faculty with no recorded doctoral institution are in none of these three groups.
- `NewFaculty`: Integer. The count of faculty labelled as new hires: those who earned their degree within 4 years of their first recorded employment as faculty (i.e., first year of employment in the sample ≤ doctoral year + 4; this is the definition used for the paper's analyses, as stated in the [Author Correction](https://doi.org/10.1038/s41586-023-06379-9)).
- `ExistingFaculty`: Integer. The count of all other faculty.
- `GiniCoefficient`: Float. The Gini coefficient of faculty production.
- `GiniCoefficient (NewFaculty)`, `GiniCoefficient (ExistingFaculty)`: Float. The Gini coefficient of faculty production among new / existing faculty.
- `Men`: Integer. The number of men.
- `Women`: Integer. The number of women.
- `FractionWomen`: Float. The fraction of faculty who are women.
- `FractionWomen (NewFaculty)`, `FractionWomen (ExistingFaculty)`: Float. The fraction of new / existing faculty who are women.
- `SelfHires`: Integer. The number of faculty employed by the institution that granted their doctorate.
- `SelfHires (Men)`, `SelfHires (Women)`: Integer. The number of men / women who are self-hires.
- `NonAttritedFaculty (SelfHires)`, `AttritedFaculty (SelfHires)`, `NonAttritionEvents (SelfHires)`, `AttritionEvents (SelfHires)`: Same, for faculty who are self-hires (faculty-years are attributed to the faculty member's self-hire status in that year, attritions to the status in the year they left).
- `NonAttritedFaculty (non-SelfHires)`, `AttritedFaculty (non-SelfHires)`, `NonAttritionEvents (non-SelfHires)`, `AttritionEvents (non-SelfHires)`: Same, for faculty who are not self-hires.
- `FractionUpHierarchyHires`: Float. The fraction of faculty employed at an institution that is more prestigious than the institution that granted their doctorate.
- `FractionDownHierarchyHires`: Float. The fraction of faculty employed at an institution that is less prestigious than the institution that granted their doctorate. (The remainder, `1 - FractionUpHierarchyHires - FractionDownHierarchyHires`, are self-hires.)
- `FractionUpHierarchyHires (null model)`: Float. The fraction of faculty expected by the null model to be employed at an institution more prestigious than the institution that granted their doctorate.
- `NullModelMoreHierarchicalThanEmpiricalCount`: Integer. The number of null-model draws (out of 1000) that were more hierarchical than the empirical data.

If you wanted to get statistics for, say, the domain of Humanities, you would do something like this (assuming your working directory is the root of this repository):

```
import pandas as pd

df = pd.read_csv('data/stats.csv')

df = df[
  (df['TaxonomyLevel'] == 'Domain')
  &
  (df['TaxonomyValue'] == 'Humanities')
]

print(df.head())
```

# Changelog

## August 2026

All four files were regenerated with the original analysis pipeline ([`us-faculty-hiring-and-retention-code`](https://github.com/LarremoreLab/us-faculty-hiring-and-retention-code)) from the same underlying data, sample, and rankings used for the paper and for the previous release of these files. Regenerating with the *unmodified* export code reproduced the previously published files exactly; the changes below are fixes to the export step only.

We are grateful to Todd Jones (Mississippi State University) and Alex Bell (Georgia State University), who noticed that institutions were missing from `institution-stats.csv`, traced the omission to the export code, and prompted this update.

- **`institution-stats.csv`: every ranked institution is now included** (12,669 rows, previously 9,832). The export used to inner-join the rankings against an attrition table that has no row for institutions with zero attrition events, which silently dropped 2,837 ranked institution–network rows and left gaps in `OrdinalPrestigeRank` (e.g., Accounting had 90 of 152 ranked institutions). `OrdinalPrestigeRank` is now contiguous (0…n−1) in every network. `PrestigeRank`, `OrdinalPrestigeRank`, and `ProductionRank` are unchanged for every previously published row.
- **Attrition is now reported in two explicit units** — faculty (`NonAttritedFaculty`, `AttritedFaculty`) and faculty-years (`NonAttritionEvents`, `AttritionEvents`) — in `institution-stats.csv` and in the `(US)`, `(Canada/UK)`, `(non US/Canada/UK)`, `(SelfHires)`, and `(non-SelfHires)` groups of `stats.csv`, following the paper's definitions (final year not at risk; attritions attributed to the discipline/status at departure; see *Units* under [institution stats](#institution-stats)). Previously the two columns were per-person counts under event names, and `NonAttritionEvents` held the *total* number of faculty (attrited + not). Relative to those previously published per-person counts: `AttritedFaculty` equals the old `AttritionEvents` for every academia row and is lower on about 0.5% of attritions in domain/field rows (faculty who changed discipline before leaving are no longer counted as attritions from the discipline they left); faculty at risk is about 3.6% lower than the old total because faculty first observed in 2020 were never at risk. The `(non US/Canada/UK)` columns previously also included faculty with no recorded doctoral institution; they no longer do.
- **`stats.csv`: the empirical `FractionUpHierarchyHires` is restored.** Previously the null-model value overwrote it and the file had two columns named `FractionUpHierarchyHires`, both holding the null-model value. The null-model value is now in `FractionUpHierarchyHires (null model)`.
- **`edge-lists.csv`: 6,134 artifact rows removed.** They had no `DegreeInstitutionId`, a blank `Total`, and `Men = Women = 0` (faculty with no recorded doctoral institution, which cannot be placed on an edge). The remaining 405,410 rows are unchanged.
- `ProductionRank` is blank for ranked institutions that produced no faculty employed in the network (these rows are all new; see its definition above).
- Integer columns (ids and counts) are written as integers rather than floats; rows are sorted by `TaxonomyLevel`, `TaxonomyValue`, and then rank / ids / year.
- README: corrected file names, the direction of `PrestigeRank` (1 = most prestigious), column names, and several column definitions.
