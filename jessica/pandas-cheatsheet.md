# Pandas Cheat Sheet

## Table of Contents
1. [Data Structures](#data-structures)
2. [Reading and Writing Data](#reading-and-writing-data)
3. [Data Inspection](#data-inspection)
4. [Data Selection](#data-selection)
5. [Data Cleaning](#data-cleaning)
6. [Data Manipulation](#data-manipulation)
7. [Grouping and Aggregation](#grouping-and-aggregation)
8. [Merging and Joining](#merging-and-joining)
9. [Time Series](#time-series)
10. [Plotting](#plotting)

## Data Structures
```python
import pandas as pd

# Series
s = pd.Series([1, 2, 3, 4])

# DataFrame
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})
```

## Reading and Writing Data
```python
# CSV
df = pd.read_csv('file.csv')
df.to_csv('file.csv', index=False)

# Excel
df = pd.read_excel('file.xlsx', sheet_name='Sheet1')
df.to_excel('file.xlsx', sheet_name='Sheet1', index=False)

# JSON
df = pd.read_json('file.json')
df.to_json('file.json')

# SQL
import sqlite3
conn = sqlite3.connect('database.db')
df = pd.read_sql('SELECT * FROM table', conn)
df.to_sql('table', conn, if_exists='replace', index=False)
```

## Data Inspection
```python
df.head()  # First 5 rows
df.tail()  # Last 5 rows
df.info()  # Concise summary
df.describe()  # Statistical summary
df.shape  # (rows, columns)
df.columns  # Column names
df.dtypes  # Data types of columns
```

## Data Selection
```python
# Selecting columns
df['A']  # Single column
df[['A', 'B']]  # Multiple columns

# Selecting rows
df.loc[0]  # Single row by label
df.iloc[0]  # Single row by position
df.loc[0:2]  # Multiple rows by label
df.iloc[0:2]  # Multiple rows by position

# Boolean indexing
df[df['A'] > 2]
```

## Data Cleaning
```python
# Handling missing values
df.dropna()  # Drop rows with any missing values
df.fillna(value=0)  # Fill missing values with 0

# Removing duplicates
df.drop_duplicates()

# Renaming columns
df.rename(columns={'old_name': 'new_name'})

# Changing data types
df['A'] = df['A'].astype('int64')
```

## Data Manipulation
```python
# Adding new columns
df['C'] = df['A'] + df['B']

# Applying functions
df['A'].apply(lambda x: x*2)

# Sorting
df.sort_values('A', ascending=False)

# Value counts
df['A'].value_counts()
```

## Grouping and Aggregation
```python
# Grouping
grouped = df.groupby('A')

# Aggregation
grouped.agg({'B': 'mean', 'C': 'sum'})

# Pivot tables
pd.pivot_table(df, values='D', index=['A', 'B'], columns=['C'])
```

## Merging and Joining
```python
# Merge
pd.merge(df1, df2, on='key')

# Concatenate
pd.concat([df1, df2], axis=0)  # Vertically
pd.concat([df1, df2], axis=1)  # Horizontally
```

## Time Series
```python
# Creating date range