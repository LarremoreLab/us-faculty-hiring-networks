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

# ranks

The csv containing the ranks of institutions in the faculty hiring networks has the following columns:

- `Rank`: Integer. The ordinal rank of the institution `InstitutionName` within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`. An `OrdinalRank` of 0 indicates the highest prestige.
- `InstitutionId`: Integer. A unique identifer for the institution that employed the faculty.
- `InstitutionName`: String. The name of the institution that employed the facutly.
- `TaxonomyValue`: String. The subscope of the faculty hiring network that this edge is a part of.
- `TaxonomyLevel`: String. The scope of the faculty hiring network that this edge is a part of.

Institutions that did not met our criteria for inclusion in a given faculty hiring network are not ranked, though it is possible that they nonetheless have a program within the bounds of that particular faculty hiring network. For more information on our inclusion criteria for rankings, see the supplement.

For example, for a row with this information:

- `Rank`: 0
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

df = df.sort_values(by=['Rank'])
print(df.head())
```

# Miscellaneous stats

The csv containing the miscellaneous stats on the faculty hiring networks has the following columns:

- `GiniCoefficient`: Float. The [Gini coefficient](https://en.wikipedia.org/wiki/Gini_coefficient) of faculty production within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`. Values range from 0 to 1. A value of 0 indicates maximum equality (production is equally distributed across producers). A value of 1 indicates that a single producer produces everything.
- `FractionWomen`: Float. The fraction of women faculty within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`.
- `PercentUpHierarchyHires`: Float. The fraction of faculty who are employed at an institution that is more prestigious than the institution they received their degree at, within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`.
- `PercentSelfHires`: Float. The fraction of faculty who are employed at the institution they received their degree at, within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`.
- `PercentDownHierarchyHires`: Float. The fraction of faculty who are employed at an institution that is less prestigious than the institution they received their degree at, within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`.
- `PercentNonUSPhD`: Float. The fraction of faculty who received their degree from an institution outside the U.S., within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`.
- `PercentUSPhDInField`: Float. The fraction of faculty who received their degree from an institution that employs faculty in the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`, within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`. Note: this is an *upper bound* on the number of faculty who were trained in-field. For example, a faculty member employed in Aerospace Engineering who earned their degree in Mechanical Engineering from an institution that also has an Aerospace Engineering department would be considered to work in-field.
- `PercentUSPhDOutOfField`: Float. The fraction of faculty who received their degree from an institution that does not employ faculty in the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`, within the subscope `TaxonomyValue` at the scope of `TaxonomyLevel`. Note: this is an *lower bound* on the number of faculty who were trained out of field. For example, a faculty member employed in Biology is considered to have been trained out of field *only* if the institution they received their does not have a Biology department; whether that faculty member's degree is in Biology or not is (unfortunately) not considered.
- `TaxonomyValue`: String. The subscope of the faculty hiring network that this edge is a part of.
- `TaxonomyLevel`: String. The scope of the faculty hiring network that this edge is a part of.

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
