# Cleaning Log and Reflection

## Code Review

Each executable statement below has a verdict. Multiline statements count as one
operation. The locations refer to the code cells and line numbers in `2_work.ipynb`.

| Line / location | Code | Verdict | Reason |
|---|---|---|---|
| L01 (cell 1, line 1) | `import pandas as pd` | **KEEP** | Loads pandas; no dataset values are changed. |
| L02 (cell 1, line 3) | `raw = pd.read_csv("C:\\Users\\twizz\\Downloads\\teaching_prices_messy.csv")` | **KEEP** | Reads 97 records from the supplied CSV. The path is specific to this PC. |
| L03 (cell 1, line 4) | `raw_before = raw.copy(deep=True)` | **KEEP** | Preserves the initial dataframe for the final equality check. |
| L04 (cell 1, line 5) | `clean = raw.copy(deep=True)` | **KEEP** | Makes a separate copy. |
| L05 (cell 1, line 6) | `original_columns = raw.columns.tolist()` | **KEEP** | Restricts duplicate comparisons to the original six fields. |
| L06 (cell 1, line 8) | `print("Starting rows:", len(raw))` | **KEEP** | Displays the evidence or status; does not change observations. |
| L07 (cell 2, line 1) | `clean["missing_price"] = clean["price"].isna()` | **KEEP** | Flags index 5; its price remains missing and the record remains present. |
| L08 (cell 2, line 3) | `print("Missing prices:", clean["missing_price"].sum())` | **KEEP** | Displays the evidence or status; does not change observations. |
| L09 (cell 2, line 4) | `print(clean.loc[clean["missing_price"]])` | **KEEP** | Displays the evidence or status; does not change observations. |
| L10 (cell 3, line 1) | `duplicate_rows = clean.duplicated( subset=original_columns, keep="first" )` | **KEEP** | Marks additional exact matches across all six original fields; only index 96 is True. |
| L11 (cell 3, line 6) | `excluded_duplicates = clean.loc[duplicate_rows].copy()` | **KEEP** | Keeps the excluded record in memory for inspection and row reconciliation. |
| L12 (cell 3, line 7) | `print(excluded_duplicates)` | **KEEP** | Displays the evidence or status; does not change observations. |
| L13 (cell 3, line 9) | `clean = clean.loc[~duplicate_rows].copy()` | **KEEP** | Excludes only index 96; all other original indices remain. |
| L14 (cell 3, line 10) | `print("Rows after removing exact duplicates:", len(clean))` | **KEEP** | Displays the evidence or status; does not change observations. |
| L15 (cell 4, line 1) | `item_check = clean[["date", "item", "city", "price"]].copy()` | **KEEP** | Creates a separate test table so the actual item labels are not overwritten. |
| L16 (cell 4, line 2) | `item_check["item_original"] = item_check["item"]` | **KEEP** | Preserves the original label beside the proposed label. |
| L17 (cell 4, line 4) | `item_check["item"] = item_check["item"].replace( {"oil b": "Oil B"} )` | **KEEP** | Tests the explicit oil b to Oil B mapping in the temporary table only. |
| L18 (cell 4, line 8) | `conflicts = item_check.duplicated( subset=["date", "item", "city"], keep=False )` | **KEEP** | Flags both indices 40 and 42 after the hypothetical mapping; prices differ. |
| L19 (cell 4, line 13) | `print(item_check.loc[conflicts])` | **KEEP** | Displays the evidence or status; does not change observations. |
| L20 (cell 4, line 15) | `clean["item_label_review"] = clean["item"].eq("oil b")` | **KEEP** | Flags index 40 for investigation without recoding its item. |
| L21 (cell 4, line 16) | `clean["key_conflict_if_normalized"] = conflicts` | **KEEP** | Preserves the warning on both affected observations. |
| L22 (cell 5, line 1) | `price_text = clean["price"].astype("string").str.strip()` | **KEEP** | Prepares temporary text without overwriting source prices. |
| L23 (cell 5, line 2) | `direct_numeric = pd.to_numeric(price_text, errors="coerce")` | **KEEP** | Uses coercion only for diagnosis; the original price column is not replaced by this temporary result. |
| L24 (cell 5, line 4) | `conversion_problem = price_text.notna() & direct_numeric.isna()` | **KEEP** | Distinguishes a failed observed value at index 17 from the already missing value at index 5. |
| L25 (cell 5, line 5) | `print(clean.loc[conversion_problem, ["date", "item", "city", "price"]])` | **KEEP** | Displays the evidence or status; does not change observations. |
| L26 (cell 6, line 1) | `clean["price_original"] = clean["price"]` | **KEEP** | Retains the original price strings, including 1,250. |
| L27 (cell 6, line 3) | `price_text = price_text.str.replace(",", "", regex=False)` | **KEEP** | Removes the comma from the observed 1,250 value. Keep for this file; inspect and validate grouping before reuse on other comma formats. |
| L28 (cell 6, line 4) | `clean["price"] = pd.to_numeric(price_text, errors="raise")` | **KEEP** | Strict conversion succeeds; 1,250 becomes 1250, the missing count stays 1, and unrecognized values would raise an error. |
| L29 (cell 6, line 6) | `print(clean["price"].dtype)` | **KEEP** | Displays the evidence or status; does not change observations. |
| L30 (cell 6, line 7) | `print("Missing prices after conversion:", clean["price"].isna().sum())` | **KEEP** | Displays the evidence or status; does not change observations. |
| L31 (cell 7, line 1) | `date_check = pd.to_datetime( clean["date"], format="%Y-%m-%d", errors="coerce" )` | **KEEP** | Tests the expected ISO format in a temporary series; it does not silently replace source dates. |
| L32 (cell 7, line 7) | `clean["invalid_date"] = clean["date"].notna() & date_check.isna()` | **KEEP** | Flags failed nonmissing dates; no failures were found. |
| L33 (cell 7, line 8) | `print(clean.loc[clean["invalid_date"], ["date", "item", "city"]])` | **KEEP** | Displays the evidence or status; does not change observations. |
| L34 (cell 7, line 10) | `if clean['invalid_date'].any():` | **KEEP** | Blocks date-column replacement if any nonmissing date fails validation; zero failures were found. |
| L35 (cell 7, line 11) | `print("Investigate flagged dates; the date column has not been changed.")` | **KEEP** | Displays the evidence or status; does not change observations. |
| L36 (cell 7, line 13) | `clean["date_original"] = clean["date"]` | **KEEP** | Preserves the original date strings before changing the working dtype. |
| L37 (cell 7, line 14) | `clean["date"] = date_check` | **KEEP** | Runs only after successful validation; all date values retain the same calendar dates. |
| L38 (cell 7, line 15) | `print("Date conversion completed.")` | **KEEP** | Displays the evidence or status; does not change observations. |
| L39 (cell 8, line 1) | `row_counts = pd.DataFrame( {"before": [len(raw)], "after": [len(clean)]}, index=["rows"] )` | **KEEP** | Records the actual change from 97 rows to 96. |
| L40 (cell 8, line 5) | `print(row_counts)` | **KEEP** | Displays the evidence or status; does not change observations. |
| L41 (cell 8, line 7) | `missing_counts = pd.DataFrame({ "before": raw[original_columns].isna().sum(), "after": clean[original_columns].isna().sum() })` | **KEEP** | Compares only the six supplied fields; price has 1 missing value before and after. |
| L42 (cell 8, line 11) | `print(missing_counts)` | **KEEP** | Displays the evidence or status; does not change observations. |
| L43 (cell 8, line 13) | `data_types = pd.DataFrame({ "before": raw[original_columns].dtypes.astype(str), "after": clean[original_columns].dtypes.astype(str) })` | **KEEP** | Shows price changing to Float64 and date to datetime; the other fields remain text. |
| L44 (cell 8, line 17) | `print(data_types)` | **KEEP** | Displays the evidence or status; does not change observations. |
| L45 (cell 8, line 19) | `item_counts = pd.DataFrame({ "before": raw["item"].value_counts(dropna=False), "after": clean["item"].value_counts(dropna=False) }).fillna(0).astype(int)` | **KEEP** | Shows actual label counts rather than forcing equal counts or hiding oil b. |
| L46 (cell 8, line 23) | `print(item_counts)` | **KEEP** | Displays the evidence or status; does not change observations. |
| L47 (cell 8, line 25) | `print(clean["price"].describe())` | **KEEP** | Displays the evidence or status; does not change observations. |
| L48 (cell 8, line 27) | `pd.testing.assert_frame_equal(raw, raw_before)` | **KEEP** | Passes: the raw dataframe is identical to its initial copy. |
| L49 (cell 8, line 28) | `assert len(raw) == len(clean) + len(excluded_duplicates)` | **KEEP** | Passes: 97 original records = 96 retained + 1 excluded. |
| L50 (cell 8, line 29) | `print("Original data unchanged; row counts reconcile.")` | **KEEP** | Displays the evidence or status; does not change observations. |

## Cleaning Log

- **P1:** Kept the missing price and added a flag. There was no confirmed replacement.
- **P2:** Excluded the extra duplicate at index 96 and kept the first copy at index 8.
- **P3:** Left the actual item labels unchanged. The test showed a conflict between indices 40 and 42, with prices of 108.5 and 318.0.
- **P4:** Converted prices to numbers after checking the comma-formatted value. Original prices were preserved.
- **P5:** Converted dates after validation. No invalid dates were found.

## Before and After

| Check | Before | After |
|---|---|---|
| Rows | 97 | 96 |
| Missing prices | 1 | 1 |
| Price type | Text | Numeric |
| Date type | Text | Datetime |

The original dataframe stayed unchanged, and the row-count check passed.

After cleaning, Staple A had 23 records, Oil B, Pulse C and Sweetener D had 24 each, and `oil b` had one. The item counts are therefore still unresolved. Prices such as 1250 and 9999 also need investigation and were not replaced.

## Reflection

The code mostly followed my initial plan. The extra check for conflicting item records was useful because changing capitalization looked simple but could hide a bigger problem.

Checking only the total number of rows would not have been enough. There were 96 rows after removing the duplicate, but the item counts still needed attention. Keeping the original values and flagging uncertain records made the changes easier to check.