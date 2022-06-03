# us-faculty-hiring-networks
This repository contains the data described in `Quantifying hierarchy and dynamics in U.S. faculty hiring`.

There are three datasets included here:

1. edge lists for all faculty networks (found at `data/edge_lists.csv`)
2. ranks of institutions in all faculty networks (found at `data/ranks.csv`)
3. miscellaneous stats, like Gini coefficients, for institutions for all faculty networks (found at `data/stats.csv`)

## Using the data

Because all of our analyses were done at three scopes (across academia; per domain, Natural Sciences for example; and per field, Computer Science for example), all of the csvs above have two columns (plus other columns that differ from dataset to dataset): 

1. `TaxonomyLevel`: values in this column indicate the level of scope (academia/domain/field) for a given row.
2. `TaxonmyValue`: values in this column indicate the subset of that scope (e.g., 'Computer Science') for a given row.

So, for example, if you want to see some information for, say, the field of Computer science, you would take the data (assuming you've read it into a pandas dataframe) and do this:

```
df = df[
  (df['TaxonomyLevel'] == 'Field')
  &
  (df['TaxonomyValue'] == 'Computer Science')
]
```

If, instead, you wanted to see information academia, you would take the data (again assuming you've read it into a pandas dataframe) and do this:

```
df = df[
  (df['TaxonomyLevel'] == 'Academia')
  &
  (df['TaxonomyValue'] == 'Academia')
]
```

# edge lists

The csv containing the faculty hiring network edge lists has the following columns:

- `Total`: Integer. The total number of faculty employed at `InstitutionName` who received their degree from `DegreeInstitutionName` within the particular the particular subset `TaxonomyValue` of the scope `TaxonomyLevel`. The sum of `Women` and `Men` for the given row.
- `Women`: Integer. Same as `Total`, but instead of being the count of all faculty, this is the count of women faculty.
- `Men`: Integer. Same as `Total`, but instead of being the count of all faculty, this is the count of men faculty.
- `DegreeInstitutionId`: Integer. A unique identifer for the institution that produced the faculty.
- `DegreeInstitutionName`: String. The name of the institution that produced the facutly.
- `InstitutionId`: Integer. A unique identifer for the institution that employed the faculty.
- `InstitutionName`: String. The name of the institution that employed the facutly.
- `TaxonomyValue`: String. The subscope of the faculty hiring network that this edge is a part of.
- `TaxonomyLevel`: String. The scope of the faculty hiring network that this edge is a part of.

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

If you wanted to look get the faculty hiring network for, say, the field of Ecology, you would do something like this (assuming your working directory is the root of this repostory):
```
import pandas as pd

df = pd.read_csv('data/edge_lists.csv')
df = df[
  (df['TaxonomyLevel'] == 'Field')
  &
  (df['TaxonomyValue'] == 'Ecology')
]
```

# institution stats

The csv containing university-level information can be found at `data/institution-stats.csv`, and has the following columns:

- `TaxonomyValue`: String. The subscope of the faculty hiring network that this edge is a part of.
- `TaxonomyLevel`: String. The scope of the faculty hiring network that this edge is a part of.
- `InstitutionId`: Integer. A unique identifer for the institution that employed the faculty.
- `InstitutionName`: String. The name of the institution that employed the facutly.
- `NonAttritionEvents`: Integer. the number of faculty who graduated from the institution who did not leave the institution (across all years)
- `AttritionEvents`: Integer. the number of faculty who graduated from the institution who left the institution (across all years)
- `ProductionRank`: Integer. the ordinal rank of the institution, in terms of how many faculty they produced
- `PrestigeRank`: Float. the SpringRank of the institution, scaled from 0-1. A rank of 0 indicates high prestige; a rank of 1 indicates low prestige.
- `OrdinalPrestigeRank`: Integer. the ordinal version of `PrestigeRank`.

Institutions that did not met our criteria for inclusion in a given faculty hiring network are not ranked, though it is possible that they nonetheless have a program within the bounds of that particular faculty hiring network. For more information on our inclusion criteria for rankings, see the supplement.

For example, for a row with this information:

- `OrdinalPrestigeRank`: 0
- `InstitutionId`: 1
- `InstitutionName`: "Puma University"
- `TaxonomyValue`: "Felinology"
- `TaxonomyLevel`: "Field"

means that Puma University is the most prestigious institution in the field of Felinology.

If you wanted to see the 5 most prestigious institutions across academia, you would do something like this (assuming your working directory is the root of this repostory):

```
import pandas as pd

df = pd.read_csv('data/ranks.csv')

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

- `TaxonomyValue`: String. The subscope of the faculty hiring network that this edge is a part of.
- `TaxonomyLevel`: String. The scope of the faculty hiring network that this edge is a part of.
- `Year`: Integer. The year of the stat.
- `GiniCoefficient`: Float. The Gini coefficient for that year.
- `FractionWomen`: Float. The fraction of women in that year.

# academia/domain/field-level stats

The csv containing the miscellaneous stats at the academia/domain/field-level has the following columns:

- `TaxonomyValue`: String. The subscope of the faculty hiring network that this edge is a part of.
- `TaxonomyLevel`: String. The scope of the faculty hiring network that this edge is a part of.
- `NoDoctorate`: Integer. The count of faculty without a doctorate.
- `Doctorate (US)`: Integer. The count of faculty with a US doctorate.
- `Doctorate (Non-US)`: Integer. The count of faculty with a non-US doctorate.
- `Doctorate (Canada/UK)`: Integer. The count of faculty with a doctorate from Canada/the UK.
- `Doctorate (Africa)`: Integer. The count of faculty with a doctorate from Africa.
- `Doctorate (Asia)`: Integer. The count of faculty with a doctorate from Asia.
- `Doctorate (Europe)`: Integer. The count of faculty with a doctorate from Europe.
- `Doctorate (North America)`: Integer. The count of faculty with a doctorate from North America.
- `Doctorate (Oceania)`: Integer. The count of faculty with a doctorate from Oceania.
- `Doctorate (South America)`: Integer. The count of faculty with a doctorate from South America.
- `NonAttritionEvents (US)`: Integer. The count of faculty who did not leave academia in a given year, across all years, for faculty with a US doctorate.
- `AttritionEvents (US)`: Integer. The count of faculty who left academia in a given year, across all years.
- `NonAttritionEvents (Canada/UK)`: Same as `NonAttritionEvents (US)`, but for faculty with a doctorate from Canada/UK
- `AttritionEvents (Canada/UK)`: Same as `AttritionEvents (US)`, but for faculty with a doctorate from Canada/UK
- `NonAttritionEvents (non US/Canada/UK)`: Same as `NonAttritionEvents (US)`, but for faculty with a doctorate from somewhere other than the US/Canada/UK
- `AttritionEvents (non US/Canada/UK)`: Same as `AttritionEvents (US)`, but for faculty with a doctorate from somewhere other than the US/Canada/UK
- `NewFaculty`: Integer. The count of facutly who started within the sample frame and earned their doctorate within 3 years of their first year in the dataset.
- `ExistingFaculty`: Integer. The count of faculty who started before the sample frame or earned their doctorate more than 3 years before their first year in the dataset.
- `GiniCoefficient`: Float. The Gini coefficient.
- `GiniCoefficient (NewFaculty)`: Float. The Gini coefficient for new faculty.
- `GiniCoefficient (ExistingFaculty)`: Float. The Gini coefficient for existing faculty.
- `Men`: Integer. The number of men in the dataset.
- `Women`: Integer. The number of women in the dataset.
- `FractionWomen`: Float. The fraction of women in the dataset.
- `FractionWomen (new faculty)`: Float. The fraction of women among new faculty.
- `FractionWomen (existing faculty)`: Float. The fraction of women among existing faculty.
- `SelfHires`: Integer. The number of faculty who were self-hired.
- `SelfHires (Men)`: Integer. The number of men who were self-hired.
- `SelfHires (Women)`: Integer. The number of women who were self-hired.
- `NonAttritionEvents (non-self-hires)`: Same as `NonAttritionEvents (US)`, but for faculty who were not self-hired.
- `AttritionEvents (non-self-hires)`: Same as `AttritionEvents (US)`, but for faculty who were not self-hired.
- `NonAttritionEvents (self-hires)`: Same as `NonAttritionEvents (US)`, but for faculty who were self-hired.
- `AttritionEvents (self-hires)`: Same as `AttritionEvents (US)`, but for faculty who were self-hired.
- `FractionUpHierarchyHires`: Float. The fraction of faculty who are employed at an institution that is more prestigious than the institution they received their degree at, within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`.
- `FractionDownHierarchyHires`: Float. The fraction of faculty who are employed at an institution that is less prestigious than the institution they received their degree at, within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`.
- `FractionUpHierarchyHires (null model)`: Float. The fraction of faculty expected by the null model to be employed at an institution that is less prestigious than the institution they received their degree at, within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`.
- `NullModelMoreHierarchicalThanEmpiricalCount`: Integer. The number of times a null model was more hierarchical than the empirical data. 1000 trials were run.

If you wanted to get statistics for, say, the domain of Humanities, you would do something like this (assuming your working directory is the root of this repostory):

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
