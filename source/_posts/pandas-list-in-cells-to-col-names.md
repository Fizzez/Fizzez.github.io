---
title: "Pandas: Conver slash-separated string contents to column names with default value"
date: 2019-09-12 17:21:08
tags: 
- Python
- Pandas
- English
---

# Description

Got a pandas DataFrame that has a column with slash-separated properties. I want to conver the properties to column-separated  `True` or `False` values.

Like the following example. The DataFrame contains 5 persons' information. The 'characters' column, in which slash-sepatared characters are contained, describing the personalities, is the target.

```plain
   id gender                characters
0   0      F            active/earnest
1   1      M                     funny
2   2      F  active/dedicated/earnest
3   3      F     dedicated/disciplined
4   4      M        active/disciplined
```

The result should look like the following. The slash-separated characters are firstly gathered to form the new columns, as a new view to personality, then appended right next to previous DataFrame. If the person has a specific character describe in 'characters' column, a `True` will be given in the corresponding new columns, else `False`.

```plain
   id gender                characters  active  earnest  funny  dedicated  disciplined
0   0      F            active/earnest    True     True  False      False        False
1   1      M                     funny   False    False   True      False        False
2   2      F  active/dedicated/earnest    True     True  False       True        False
3   3      F     dedicated/disciplined   False    False  False       True         True
4   4      M        active/disciplined    True    False  False      False         True
```

# Solution

1. Split the 'slash-separated properities' to a list of properties.

   ```python
   series_characters = df['characters'].str.split('/')
   ```

   ```plain
   series_characters
   
   >>> 0               [active, earnest]
   >>> 1                         [funny]
   >>> 2    [active, dedicated, earnest]
   >>> 3        [dedicated, disciplined]
   >>> 4           [active, disciplined]
   >>> Name: characters, dtype: object
   ```

2. Convert the list to pd.Series with default value.

   ```python
   series_characters = series_characters.apply(lambda lst: pd.Series(dict.fromkeys(lst, True)))
   ```

   ```plain
   series_characters
   
   >>>   active earnest funny dedicated disciplined
   >>> 0   True    True   NaN       NaN         NaN
   >>> 1    NaN     NaN  True       NaN         NaN
   >>> 2   True    True   NaN      True         NaN
   >>> 3    NaN     NaN   NaN      True        True
   >>> 4   True     NaN   NaN       NaN        True
   ```

3. Concat the result to original DataFrame and fill `nan`.

   ```python
   df_new = pd.concat([df, series_characters], axis=1).fillna(False)
   ```

   ```plain
   df_new
   
   >>>    id gender                characters  active  earnest  funny  dedicated  disciplined
   >>> 0   0      F            active/earnest    True     True  False      False        False
   >>> 1   1      M                     funny   False    False   True      False        False
   >>> 2   2      F  active/dedicated/earnest    True     True  False       True        False
   >>> 3   3      F     dedicated/disciplined   False    False  False       True         True
   >>> 4   4      M        active/disciplined    True    False  False      False         True
   ```

# Notebook Link

A runnable Jupyter Notebook that contains the example above.

> https://github.com/Fizzez/playground/blob/master/python/pandas-list-in-cells-to-col-names.ipynb