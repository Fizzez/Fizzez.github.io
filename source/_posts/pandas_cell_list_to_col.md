## Pandas: How to convert list in cells to columns

#### Description:

pd.DataFrame with string list

#### Steps:

1. Convert cell content to list

   ```python
   df['col_name'].str.split(' / ')
   ```

2. Convert list to dict with default value

   ```python
   df['col_name'].apply(lambda lst: dict.fromkeys(lst, DefaultVal))
   ```

3. Coonvert dict to pd.Series

   ```python
   df['col_name'].apply(pd.Series)
   ```

4. Concat pd.Series and pd.DataFrame

