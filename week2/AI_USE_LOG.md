### AI was also used to make wording and formatting in logs better. It was just made to make it look more presentable.

### Prompt
I am auditing teaching_prices_messy.csv using pandas. The dataset has 97 rows and six columns: date, item, city, price, unit and data_label. All columns are currently stored as strings. Please suggest beginner-friendly pandas code for these problems, organized by P1–P5. Work on a copy called clean and leave raw unchanged. Explain every line. Do not invent replacement values or silently discard observations. Include before-and-after checks for row counts, missing values, data types and item counts. My initial diagnosis is:
P1: One missing price for Pulse C in City 2 on 2026-01-01. My planned decision is to retain the observation and flag the missing price.
P2: One exact duplicate, at dataframe indices 8 and 96. My planned decision is to keep the first occurrence and exclude the extra copy.
P3: Both "Oil B" and "oil b" appear as item labels. I want to investigate before standardizing them and check whether this creates conflicting records.
P4: Prices are stored as text. I need to investigate conversion problems before converting them to numbers.
P5: Dates are stored as text. I need to validate them before converting them to datetime values.

### AI Response
Run these in separate cells in **`2_work.ipynb`**. Comments explain each line. This is the proposed code for you to review in the Cleaning Code Court.

**Setup — load the original and make a copy**

```python
import pandas as pd  # Load pandas.

raw = pd.read_csv("teaching_prices_messy.csv")  # Read the original dataset.
raw_before = raw.copy(deep=True)  # Preserve a reference for the final check.
clean = raw.copy(deep=True)  # Make the working copy.
original_columns = raw.columns.tolist()  # Remember the six supplied columns.

print("Starting rows:", len(raw))  # Display the original record count.
```

**P1 — retain and flag the missing price**

```python
clean["missing_price"] = clean["price"].isna()  # Flag missing prices as True.

print("Missing prices:", clean["missing_price"].sum())  # Count flagged rows.
print(clean.loc[clean["missing_price"]])  # Display those rows.
```

This adds a flag. It does not fill the missing price or remove its row.

**P2 — exclude only the extra exact duplicate**

```python
duplicate_rows = clean.duplicated(
    subset=original_columns,
    keep="first"
)  # Compare all six original fields and mark additional identical copies.

excluded_duplicates = clean.loc[duplicate_rows].copy()  # Preserve excluded records.
print(excluded_duplicates)  # Show exactly what will be removed.

clean = clean.loc[~duplicate_rows].copy()  # Keep rows that are not extra copies.
print("Rows after removing exact duplicates:", len(clean))  # Check the count.
```

Using the original columns prevents newly added flags from affecting the duplicate test. `~` means “not.”

**P3 — investigate capitalization before changing the real labels**

```python
item_check = clean[["date", "item", "city", "price"]].copy()  # Make a test table.
item_check["item_original"] = item_check["item"]  # Preserve original labels.

item_check["item"] = item_check["item"].replace(
    {"oil b": "Oil B"}
)  # Test the proposed capitalization change in this separate table.

conflicts = item_check.duplicated(
    subset=["date", "item", "city"],
    keep=False
)  # Flag every record sharing a key after the proposed change.

print(item_check.loc[conflicts])  # Inspect records whose keys would conflict.

clean["item_label_review"] = clean["item"].eq("oil b")  # Flag the uncertain label.
clean["key_conflict_if_normalized"] = conflicts  # Record the conflict warning.
```

**This does not change `clean["item"]`.** If different prices share a key after normalization, investigate them before deciding how to standardize the label. Do not delete one automatically.

**P4 — inspect price formatting, then convert**

First, run:

```python
price_text = clean["price"].astype("string").str.strip()  # Prepare temporary text.
direct_numeric = pd.to_numeric(price_text, errors="coerce")  # Test conversion.

conversion_problem = price_text.notna() & direct_numeric.isna()  # Find failed nonmissing values.
print(clean.loc[conversion_problem, ["date", "item", "city", "price"]])  # Show the originals.
```

`errors="coerce"` is used **only for this diagnostic test**. It leaves `clean["price"]` unchanged.

**Inspect the output. Only run the following if you confirm that commas in the affected prices are thousands separators:**

```python
clean["price_original"] = clean["price"]  # Preserve the source values.

price_text = price_text.str.replace(",", "", regex=False)  # Remove verified thousands separators.
clean["price"] = pd.to_numeric(price_text, errors="raise")  # Convert, stopping on unrecognized values.

print(clean["price"].dtype)  # Check the resulting numeric type.
print("Missing prices after conversion:", clean["price"].isna().sum())  # Check for unexpected missing values.
```

Removing a thousands separator changes the representation, not the stated amount. It does **not** establish that the amount is plausible.

**P5 — validate dates before conversion**

```python
date_check = pd.to_datetime(
    clean["date"],
    format="%Y-%m-%d",
    errors="coerce"
)  # Test the expected date format using a temporary series.

clean["invalid_date"] = clean["date"].notna() & date_check.isna()  # Flag failed nonmissing dates.
print(clean.loc[clean["invalid_date"], ["date", "item", "city"]])  # Display failures.

if clean["invalid_date"].any():  # Check whether any dates failed.
    print("Investigate flagged dates; the date column has not been changed.")
else:
    clean["date_original"] = clean["date"]  # Preserve the original date strings.
    clean["date"] = date_check  # Convert only when validation succeeds.
    print("Date conversion completed.")
```

**Before-and-after checks**

```python
row_counts = pd.DataFrame(
    {"before": [len(raw)], "after": [len(clean)]},
    index=["rows"]
)  # Compare record counts.
print(row_counts)

missing_counts = pd.DataFrame({
    "before": raw[original_columns].isna().sum(),
    "after": clean[original_columns].isna().sum()
})  # Compare missing values in the original six columns.
print(missing_counts)

data_types = pd.DataFrame({
    "before": raw[original_columns].dtypes.astype(str),
    "after": clean[original_columns].dtypes.astype(str)
})  # Compare column types.
print(data_types)

item_counts = pd.DataFrame({
    "before": raw["item"].value_counts(dropna=False),
    "after": clean["item"].value_counts(dropna=False)
}).fillna(0).astype(int)  # Compare record counts for each item label.
print(item_counts)

print(clean["price"].describe())  # Inspect the numeric price summary after conversion.

pd.testing.assert_frame_equal(raw, raw_before)  # Verify that raw is unchanged.
assert len(raw) == len(clean) + len(excluded_duplicates)  # Account for every original row.
print("Original data unchanged; row counts reconcile.")
```

