
# Q&A and Common Issues in Advanced SQLite3 Exercises

This document addresses common questions and issues students may encounter while working through the advanced SQLite3 exercises. It also provides troubleshooting tips and solutions for common errors, including those related to `pandas` and plotting graphs.

## Table of Contents

1. [General Questions](#general-questions)
2. [Common Issues with SQLite3](#common-issues-with-sqlite3)
3. [Common Errors in Python Code](#common-errors-in-python-code)
4. [Common Issues with Pandas](#common-issues-with-pandas)
5. [Common Issues with Plotting Graphs](#common-issues-with-plotting-graphs)
6. [Troubleshooting Tips](#troubleshooting-tips)

---

## General Questions

### Q1: How do I connect to an SQLite database in Python?

**A1:** You can connect to an SQLite database using the `sqlite3` library in Python. Here’s a basic example:

```python
import sqlite3

# Connect to the database (or create it if it doesn't exist)
conexion = sqlite3.connect('database_name.db')

# Create a cursor object to execute SQL commands
cursor = conexion.cursor()
```

### Q2: What is the purpose of using `Faker` in these exercises?

**A2:** The `Faker` library is used to generate realistic, synthetic data for testing and practicing with databases. It helps create data that mimics real-world entries, such as names, dates, and other text fields, allowing students to practice SQL and Python with more realistic datasets.

---

## Common Issues with SQLite3

### Q3: Why am I getting a "database is locked" error?

**A3:** The "database is locked" error typically occurs when another connection to the SQLite database is not closed properly, and the database is still in use. To fix this, make sure that all connections are closed using `conexion.close()` and avoid opening multiple connections to the same database simultaneously.

### Q4: How can I ensure that foreign key constraints are enforced?

**A4:** SQLite requires that foreign key support be explicitly enabled. To enforce foreign key constraints, add the following line after establishing the database connection:

```python
cursor.execute("PRAGMA foreign_keys = ON;")
```

This ensures that foreign key constraints are checked and enforced during database operations.

---

## Common Errors in Python Code

### Q5: I'm getting a "TypeError: 'NoneType' object is not iterable" error. What does this mean?

**A5:** This error occurs when you try to iterate over an object that is `None`. It often happens when a function or query returns `None` instead of the expected iterable object (like a list or tuple). To fix this, ensure that the variable you're iterating over has been properly initialized and contains the expected data. For example, check your SQL queries to ensure they are correctly returning data:

```python
resultados = cursor.fetchall()
if resultados:  # Ensure 'resultados' is not None
    for resultado in resultados:
        print(resultado)
```

### Q6: Why am I getting an "OperationalError: no such table" when querying a table I just created?

**A6:** This error occurs if the table name is misspelled, the table wasn't created successfully, or the database connection was reset (for example, if the program was restarted). Double-check the table name and ensure that the table creation command was executed correctly before attempting to query it.

### Q7: I'm getting a "ValueError: SQL logic error or missing database" when trying to insert data. What should I do?

**A7:** This error can occur if there is an issue with the SQL statement or the database itself. Common causes include:

- **Syntax errors in the SQL command:** Double-check your SQL syntax.
- **Foreign key constraints:** If you're inserting data into a table with foreign keys, ensure that the referenced records exist in the parent table.
- **Database corruption:** If the database file is corrupted, try creating a new database and re-running your scripts.

---

## Common Issues with Pandas

### Q8: Why am I getting a "KeyError" when accessing a column in a DataFrame?

**A8:** A `KeyError` occurs when you try to access a column that doesn't exist in the DataFrame. This can happen if the column name is misspelled or if the DataFrame was not created properly. Double-check the column names and ensure that the DataFrame has been populated with the correct data:

```python
# Correct way to access a column
df['column_name']
```

### Q9: My DataFrame is empty after querying the database. What went wrong?

**A9:** If your DataFrame is empty, it could be due to an incorrect SQL query or no matching data in the database. Ensure that your query is correct and that the database contains the expected data. You can also print the query to verify its correctness:

```python
query = "SELECT * FROM alumnos WHERE edad > 20"
df = pd.read_sql_query(query, conexion)
print(df)
```

### Q10: How do I handle missing data in a DataFrame?

**A10:** `pandas` provides several methods for handling missing data:

- **Drop missing data:** Remove rows or columns with missing values using `dropna()`.

  ```python
  df.dropna(inplace=True)
  ```

- **Fill missing data:** Replace missing values with a specified value using `fillna()`.

  ```python
  df['column_name'].fillna(value=0, inplace=True)
  ```

- **Interpolate missing data:** Estimate missing values using interpolation.

  ```python
  df['column_name'].interpolate(inplace=True)
  ```

---

## Common Issues with Plotting Graphs

### Q11: My plot is not displaying in Jupyter Notebook or my Python environment. What should I do?

**A11:** If your plot is not displaying, ensure that you have correctly called `plt.show()` to render the plot:

```python
import matplotlib.pyplot as plt

# Your plotting code
plt.show()
```

If you're using Jupyter Notebook, ensure that `%matplotlib inline` is included at the beginning of your notebook:

```python
%matplotlib inline
```

### Q12: I'm getting an error when trying to plot a graph. What could be the issue?

**A12:** Common issues when plotting graphs include:

- **Missing or incorrect data:** Ensure that the data being plotted is not `None` or missing.
- **Incorrect syntax:** Double-check the syntax of your plotting code, especially for parameters and function names.
- **Incompatible data types:** Ensure that the data types used for plotting (e.g., numeric data) are appropriate for the type of plot you're creating.

### Q13: My graph looks distorted or unreadable. How can I fix it?

**A13:** If your graph appears distorted, consider the following:

- **Adjust the figure size:** Use `plt.figure(figsize=(width, height))` to change the size of the plot.

  ```python
  plt.figure(figsize=(10, 6))
  ```

- **Modify axis limits:** Use `plt.xlim()` and `plt.ylim()` to set appropriate limits for the axes.

  ```python
  plt.xlim(0, 100)
  plt.ylim(0, 10)
  ```

- **Rotate axis labels:** Use `plt.xticks(rotation=45)` to rotate the x-axis labels for better readability.

  ```python
  plt.xticks(rotation=45)
  ```

---

## Troubleshooting Tips

### T1: How can I debug SQL queries in Python?

**Tip:** To debug SQL queries, you can print the query string before executing it:

```python
query = "SELECT * FROM alumnos WHERE edad > ?"
print(query)
cursor.execute(query, (20,))
```

This allows you to see the exact query being executed, helping you identify any syntax errors or issues with placeholders.

### T2: What should I do if my queries are returning empty results?

**Tip:** If your queries are returning no results, check the following:

- **Correct table and column names:** Ensure you're querying the correct tables and columns.
- **Data types:** Ensure that the data types in your WHERE clause match the data types in the database.
- **Query logic:** Verify that your query logic (e.g., WHERE conditions) is correct and should return results.

### T3: How do I handle NULL values in queries?

**Tip:** When dealing with `NULL` values, use the `IS NULL` or `IS NOT NULL` clauses in your queries. For example:

```python
cursor.execute("SELECT * FROM alumnos WHERE edad IS NULL")
```

This will select all records where the `edad` column has `NULL` values.

---

## Conclusion

By addressing these common issues and errors, students can more effectively troubleshoot problems they encounter while working through the exercises. Understanding these pitfalls will help deepen your knowledge of SQLite, Python, `pandas`, and data visualization, making you more proficient in debugging and problem-solving.
