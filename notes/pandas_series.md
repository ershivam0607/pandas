# Pandas Series — Complete Reference

---

## Table of Contents

1. [What is a Series?](#1-what-is-a-series)
2. [Creating a Series](#2-creating-a-series)
3. [Series Attributes](#3-series-attributes)
4. [Indexing & Slicing](#4-indexing--slicing)
5. [Vectorized Operations](#5-vectorized-operations)
6. [Boolean Masking & Filtering](#6-boolean-masking--filtering)
7. [Handling Missing Data (NaN)](#7-handling-missing-data-nan)
8. [String Operations (.str)](#8-string-operations-str)
9. [DateTime Operations (.dt)](#9-datetime-operations-dt)
10. [Aggregate & Statistical Methods](#10-aggregate--statistical-methods)
11. [Sorting & Ranking](#11-sorting--ranking)
12. [Mapping, Apply & Transform](#12-mapping-apply--transform)
13. [Value Counts & Unique Values](#13-value-counts--unique-values)
14. [Type Conversion](#14-type-conversion)
15. [Alignment & Reindexing](#15-alignment--reindexing)
16. [Combining Series](#16-combining-series)
17. [Series vs DataFrame Column](#17-series-vs-dataframe-column)
18. [Useful Tips & Gotchas](#18-useful-tips--gotchas)

---

## 1. What is a Series?

A **Pandas Series** is a one-dimensional labeled array capable of holding any data type (integers, floats, strings, Python objects, etc.).

```
Index   Value
  0  →   10
  1  →   20
  2  →   30
```

```python
import pandas as pd
import numpy as np
```

- Think of it as a column in a spreadsheet with a named index.
- Every Series has an **index** (labels) and **values** (the actual data).
- It is the building block of a `DataFrame`.

---

## 2. Creating a Series

### From a List

```python
s = pd.Series([10, 20, 30, 40])
# Default index: 0, 1, 2, 3
```

### From a List with Custom Index

```python
s = pd.Series([10, 20, 30], index=['a', 'b', 'c'])
```

### From a Dictionary

```python
data = {'apple': 100, 'banana': 200, 'cherry': 300}
s = pd.Series(data)
# Keys become the index
```

### From a Scalar (broadcasts)

```python
s = pd.Series(5, index=[0, 1, 2, 3])
# Result: 5, 5, 5, 5
```

### From a NumPy Array

```python
arr = np.array([1.1, 2.2, 3.3])
s = pd.Series(arr, index=['x', 'y', 'z'])
```

### From a Range

```python
s = pd.Series(range(1, 6))
```

### With Name

```python
s = pd.Series([10, 20, 30], name='scores')
```

---

## 3. Series Attributes

| Attribute         | Description                                      | Example                  |
|-------------------|--------------------------------------------------|--------------------------|
| `s.values`        | NumPy array of values                            | `array([10, 20, 30])`    |
| `s.index`         | Index labels                                     | `RangeIndex(start=0...)`|
| `s.dtype`         | Data type of values                              | `int64`                  |
| `s.shape`         | Tuple `(n,)`                                     | `(3,)`                   |
| `s.size`          | Total number of elements                         | `3`                      |
| `s.ndim`          | Number of dimensions (always 1)                  | `1`                      |
| `s.name`          | Name of the Series                               | `'scores'`               |
| `s.index.name`    | Name of the index                                | `'city'`                 |
| `s.nbytes`        | Total bytes consumed                             | `24`                     |
| `s.hasnans`       | Boolean — whether NaN values exist               | `True / False`           |
| `s.empty`         | Boolean — True if Series is empty                | `False`                  |
| `s.is_monotonic_increasing` | Check sorted order                   | `True / False`           |

```python
s = pd.Series([10, 20, 30], index=['a', 'b', 'c'], name='marks')

print(s.values)      # [10 20 30]
print(s.index)       # Index(['a', 'b', 'c'], dtype='object')
print(s.dtype)       # int64
print(s.shape)       # (3,)
print(s.name)        # marks
```

---

## 4. Indexing & Slicing

### Label-based Indexing — `.loc[]`

```python
s = pd.Series([10, 20, 30, 40], index=['a', 'b', 'c', 'd'])

s.loc['b']           # 20
s.loc['a':'c']       # a=10, b=20, c=30  (inclusive on both ends)
s.loc[['a', 'c']]    # a=10, c=30
```

### Position-based Indexing — `.iloc[]`

```python
s.iloc[1]            # 20
s.iloc[0:3]          # first 3 elements (exclusive end)
s.iloc[[0, 2]]       # elements at positions 0 and 2
s.iloc[-1]           # last element
```

### Direct Access (shorthand)

```python
s['b']               # 20  (label-based, same as .loc)
s[1]                 # 20  (integer position, same as .iloc — avoid ambiguity)
```

> **Best Practice**: Always use `.loc` or `.iloc` explicitly to avoid ambiguity, especially with integer indexes.

### Conditional / Boolean Indexing

```python
s[s > 20]            # elements where value > 20
s[(s > 10) & (s < 40)]
```

### Setting Values

```python
s.loc['a'] = 99
s.iloc[0] = 99
s[s < 0] = 0         # set all negatives to 0
```

---

## 5. Vectorized Operations

Operations are applied element-wise — no loops needed.

```python
s = pd.Series([1, 2, 3, 4])

s + 10          # [11, 12, 13, 14]
s * 2           # [2, 4, 6, 8]
s ** 2          # [1, 4, 9, 16]
s / 2           # [0.5, 1.0, 1.5, 2.0]
s // 2          # floor division
s % 3           # modulus

np.sqrt(s)      # element-wise square root
np.log(s)       # natural log
np.abs(s)       # absolute values
```

### Operations Between Two Series (Index-aligned)

```python
a = pd.Series([1, 2, 3], index=['x', 'y', 'z'])
b = pd.Series([10, 20, 30], index=['x', 'y', 'z'])

a + b           # [11, 22, 33]

# Mismatched index → NaN for missing
c = pd.Series([10, 20], index=['x', 'y'])
a + c           # x=11, y=22, z=NaN
```

---

## 6. Boolean Masking & Filtering

```python
s = pd.Series([5, 15, 25, 35, 45])

mask = s > 20
# 0    False
# 1    False
# 2     True
# 3     True
# 4     True

s[mask]                   # 25, 35, 45
s[s.between(10, 30)]      # 15, 25
s[s.isin([5, 35, 99])]   # 5, 35

# Negation
s[~mask]                  # 5, 15

# Multiple conditions
s[(s > 10) & (s < 40)]   # 15, 25, 35
s[(s < 10) | (s > 40)]   # 5, 45
```

---

## 7. Handling Missing Data (NaN)

```python
s = pd.Series([1, np.nan, 3, None, 5])

s.isna()         # True where NaN/None
s.notna()        # True where not NaN
s.isnull()       # alias for isna()
s.notnull()      # alias for notna()

s.dropna()                   # remove NaN rows
s.fillna(0)                  # fill NaN with 0
s.fillna(s.mean())           # fill with mean
s.fillna(method='ffill')     # forward fill (deprecated in newer pandas → use ffill())
s.ffill()                    # forward fill
s.bfill()                    # backward fill
s.fillna(method='bfill')     # backward fill (older syntax)
s.interpolate()              # linear interpolation

s.isna().sum()               # count of NaN values
s.isna().mean()              # proportion of NaN values
```

---

## 8. String Operations (`.str`)

Only works on `dtype=object` (string) Series.

```python
s = pd.Series(['Alice', 'Bob', 'charlie', '  Dave  ', 'eve123'])

s.str.upper()             # 'ALICE', 'BOB', ...
s.str.lower()             # 'alice', 'bob', ...
s.str.title()             # 'Alice', 'Bob', 'Charlie', ...
s.str.strip()             # remove leading/trailing whitespace
s.str.lstrip()
s.str.rstrip()

s.str.len()               # length of each string
s.str.contains('li')      # boolean mask
s.str.startswith('A')
s.str.endswith('e')

s.str.replace('e', 'E')   # string replace
s.str.replace(r'\d+', '', regex=True)  # regex replace

s.str.split(' ')          # split by space → list
s.str.split(' ', expand=True)  # → DataFrame

s.str[0]                  # first character of each string
s.str[:3]                 # first 3 characters

s.str.count('l')          # count occurrences
s.str.find('li')          # index of first occurrence
s.str.get(0)              # same as s.str[0]

s.str.pad(width=10, side='right', fillchar='-')  # padding
s.str.zfill(5)            # zero-pad on the left

s.str.cat(sep=', ')       # concatenate all into one string
```

---

## 9. DateTime Operations (`.dt`)

```python
s = pd.Series(pd.date_range('2024-01-01', periods=5, freq='D'))

s.dt.year           # [2024, 2024, ...]
s.dt.month          # [1, 1, ...]
s.dt.day
s.dt.hour
s.dt.minute
s.dt.second
s.dt.weekday        # 0=Monday, 6=Sunday
s.dt.day_name()     # 'Monday', 'Tuesday', ...
s.dt.month_name()
s.dt.quarter        # 1, 2, 3, or 4
s.dt.is_month_start
s.dt.is_month_end
s.dt.is_year_start
s.dt.is_year_end
s.dt.is_leap_year

s.dt.date           # date portion only
s.dt.time           # time portion only

s.dt.strftime('%Y-%m-%d')     # custom format string
s.dt.normalize()              # set time to midnight
s.dt.tz_localize('UTC')       # attach timezone
s.dt.tz_convert('Asia/Kolkata')  # convert timezone

# Timedelta
td = pd.Series(pd.to_timedelta(['1 days', '2 hours', '30 min']))
td.dt.days
td.dt.seconds
td.dt.total_seconds()
```

---

## 10. Aggregate & Statistical Methods

```python
s = pd.Series([4, 7, 2, 9, 1, 5, 8, 3, 6])

s.sum()           # total
s.mean()          # arithmetic mean
s.median()        # median
s.mode()          # mode (returns Series)
s.min()           # minimum
s.max()           # maximum
s.std()           # standard deviation (ddof=1 by default)
s.var()           # variance
s.sem()           # standard error of mean
s.skew()          # skewness
s.kurt()          # kurtosis

s.cumsum()        # cumulative sum
s.cumprod()       # cumulative product
s.cummin()        # cumulative minimum
s.cummax()        # cumulative maximum

s.pct_change()    # % change between consecutive elements
s.diff()          # first-order difference
s.diff(2)         # difference with lag=2

s.quantile(0.25)  # 25th percentile
s.quantile([0.25, 0.5, 0.75])  # multiple percentiles

s.describe()      # count, mean, std, min, 25%, 50%, 75%, max

s.abs()           # absolute values
s.round(2)        # round to 2 decimal places
s.clip(lower=3, upper=7)  # clip values to range [3, 7]
```

---

## 11. Sorting & Ranking

```python
s = pd.Series([30, 10, 40, 20, 50], index=['c', 'a', 'd', 'b', 'e'])

# Sort by values
s.sort_values()                       # ascending
s.sort_values(ascending=False)        # descending
s.sort_values(na_position='first')    # NaN at the top

# Sort by index
s.sort_index()                        # ascending index
s.sort_index(ascending=False)

# Ranking
s.rank()                              # average rank (default)
s.rank(method='min')                  # minimum rank for ties
s.rank(method='max')                  # maximum rank for ties
s.rank(method='first')               # first appearance wins
s.rank(method='dense')               # no gaps in rank
s.rank(ascending=False)              # rank from largest (1=largest)
s.rank(pct=True)                     # percentile rank (0 to 1)

# nlargest / nsmallest
s.nlargest(3)                        # top 3 values
s.nsmallest(3)                       # bottom 3 values
```

---

## 12. Mapping, Apply & Transform

### `.map()` — element-wise transformation

```python
s = pd.Series(['cat', 'dog', 'fish'])

# Map with a dictionary
s.map({'cat': 1, 'dog': 2, 'fish': 3})   # [1, 2, 3]

# Map with a function
s.map(str.upper)                          # ['CAT', 'DOG', 'FISH']

# Map with a lambda
s.map(lambda x: x + '!')                 # ['cat!', 'dog!', 'fish!']

# Note: .map() leaves unmatched values as NaN
s.map({'cat': 1})                         # [1, NaN, NaN]
```

### `.apply()` — apply any function

```python
s = pd.Series([1, 4, 9, 16])

s.apply(np.sqrt)                          # [1, 2, 3, 4]
s.apply(lambda x: x ** 2)
s.apply(lambda x: 'high' if x > 5 else 'low')
```

### `.transform()` — apply and preserve shape/index

```python
s.transform(lambda x: (x - x.mean()) / x.std())   # z-score normalization
s.transform(['mean', 'std'])                        # multiple functions
```

### Difference: `.map()` vs `.apply()`

| Feature              | `.map()`              | `.apply()`            |
|----------------------|-----------------------|-----------------------|
| Works with dict      | Yes                   | No                    |
| Works with function  | Yes                   | Yes                   |
| Unmatched → NaN      | Yes (dict mode)       | N/A                   |
| Primarily for        | Element replacement   | Custom functions      |

---

## 13. Value Counts & Unique Values

```python
s = pd.Series(['cat', 'dog', 'cat', 'fish', 'dog', 'cat'])

s.unique()            # array(['cat', 'dog', 'fish'])  — unique values
s.nunique()           # 3  — count of unique values
s.nunique(dropna=False)  # include NaN in count

s.value_counts()
# cat     3
# dog     2
# fish    1

s.value_counts(normalize=True)   # as proportions (0 to 1)
s.value_counts(ascending=True)   # least frequent first
s.value_counts(dropna=False)     # include NaN in count

s.duplicated()        # True for duplicated elements
s.drop_duplicates()   # keep only first occurrence
s.drop_duplicates(keep='last')   # keep last occurrence
s.drop_duplicates(keep=False)    # drop all duplicates
```

---

## 14. Type Conversion

```python
s = pd.Series(['1', '2', '3', '4'])

s.astype(int)              # convert to integer
s.astype(float)            # convert to float
s.astype(str)              # convert to string
s.astype('category')       # categorical type (memory efficient)
s.astype('bool')

# Safe conversion with error handling
pd.to_numeric(s)                       # convert to numeric
pd.to_numeric(s, errors='coerce')      # invalid → NaN
pd.to_numeric(s, errors='ignore')      # invalid → keep as-is

pd.to_datetime(s)                      # convert to datetime
pd.to_datetime(s, format='%Y-%m-%d')  # with explicit format
pd.to_datetime(s, errors='coerce')

# Categorical
s = s.astype('category')
s.cat.categories          # list of categories
s.cat.codes               # integer codes for each element
s.cat.add_categories(['new_cat'])
s.cat.remove_categories(['old_cat'])
s.cat.rename_categories({'old': 'new'})
s.cat.reorder_categories(['c', 'b', 'a'])
s.cat.as_ordered()        # make ordered categorical
```

---

## 15. Alignment & Reindexing

```python
s = pd.Series([10, 20, 30], index=['a', 'b', 'c'])

# Reindex — add/remove/reorder index labels
s.reindex(['a', 'c', 'e'])
# a    10.0
# c    30.0
# e     NaN      ← new label gets NaN

s.reindex(['a', 'c', 'e'], fill_value=0)   # fill with 0 instead
s.reindex(['a', 'c', 'e'], method='ffill') # forward fill

# Reset index (make index a column in a DataFrame)
s.reset_index()               # returns a DataFrame
s.reset_index(drop=True)      # returns a Series with 0,1,2 index

# Rename index labels
s.rename(index={'a': 'A', 'b': 'B'})

# Rename the Series itself
s.rename('new_name')

# Set index name
s.index.name = 'letters'
```

---

## 16. Combining Series

### `pd.concat()`

```python
s1 = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
s2 = pd.Series([4, 5, 6], index=['d', 'e', 'f'])

pd.concat([s1, s2])                # stack vertically
pd.concat([s1, s2], ignore_index=True)   # reset to 0,1,2...
pd.concat([s1, s2], keys=['first', 'second'])  # MultiIndex
```

### Arithmetic Alignment

```python
s1 = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
s2 = pd.Series([10, 20], index=['b', 'c'])

s1 + s2
# a     NaN
# b    22.0
# c    23.0

# Fill missing alignment with a value
s1.add(s2, fill_value=0)
# a     1.0
# b    22.0
# c    23.0
```

### `.combine_first()` — fill NaN from another Series

```python
s1 = pd.Series([1, np.nan, 3], index=['a', 'b', 'c'])
s2 = pd.Series([10, 20, 30], index=['a', 'b', 'd'])

s1.combine_first(s2)
# a     1.0   ← s1 wins (not NaN)
# b    20.0   ← s2 fills in s1's NaN
# c     3.0   ← s1 only
# d    30.0   ← s2 only
```

### `.update()` — in-place update with another Series

```python
s1.update(s2)   # modifies s1 in place where indexes overlap
```

---

## 17. Series vs DataFrame Column

```python
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})

col = df['A']          # returns a Series
type(col)              # pandas.core.series.Series

# Series → DataFrame
col.to_frame()                     # column name = Series name
col.to_frame(name='new_name')

# DataFrame column assignment
df['C'] = pd.Series([7, 8, 9])
df['D'] = df['A'] + df['B']       # vectorized

# Extract multiple columns → DataFrame (not Series)
df[['A', 'B']]         # returns a DataFrame
```

---

## 18. Useful Tips & Gotchas

### Copy vs View

```python
# .iloc and .loc may return a view or a copy — use .copy() to be safe
s2 = s.iloc[0:3].copy()    # always safe to modify
```

### Chained Assignment Warning

```python
# Bad (may not work as expected)
s[s > 5][0] = 99

# Good
mask = s > 5
s.loc[mask] = 99
```

### Integer Index Ambiguity

```python
s = pd.Series([10, 20, 30], index=[2, 1, 0])
s[0]          # returns 30 (label 0, not position 0)
s.iloc[0]     # returns 10 (always position-based)
s.loc[0]      # returns 30 (always label-based)
```

### `inplace=True` vs Reassignment

```python
s.sort_values(inplace=True)    # modifies in-place (not recommended)
s = s.sort_values()            # preferred — returns a new Series
```

### Memory Optimization

```python
s.astype('int32')              # use smaller dtype
s.astype('category')           # for low-cardinality string columns
s.memory_usage(deep=True)      # check memory in bytes
```

### Pipe for Method Chaining

```python
result = (
    s
    .dropna()
    .astype(float)
    .pipe(lambda x: x[x > 0])
    .reset_index(drop=True)
)
```

### Quick Summary Checklist

| Task                         | Method                         |
|------------------------------|--------------------------------|
| Create from list/dict/array  | `pd.Series(...)`               |
| Label-based access           | `.loc[]`                       |
| Position-based access        | `.iloc[]`                      |
| Filter by condition          | `s[s > x]`                     |
| Handle NaN                   | `.isna()`, `.fillna()`, `.dropna()` |
| Element-wise transform       | `.map()`, `.apply()`           |
| Aggregate                    | `.sum()`, `.mean()`, `.describe()` |
| Frequency count              | `.value_counts()`              |
| Sort values / index          | `.sort_values()`, `.sort_index()` |
| Type change                  | `.astype()`, `pd.to_numeric()` |
| Combine two Series           | `pd.concat()`, `.combine_first()` |
| String ops                   | `.str.*`                       |
| DateTime ops                 | `.dt.*`                        |
| Export to list/array         | `.tolist()`, `.to_numpy()`     |

---

