# Initial Diagnosis

## Dataset Overview

Loaded the dataset using pandas and checked the first five rows with df.head() to confirm if the file was loaded in our environment correctly. I used df.shape, df.columns.tolist() and df.info() and found that the dataset contains 97 rows and six columns: date, item, city, price, unit and data_label.

## Problem diagnosis

| ID     | Finding                                                                        | Code that revealed it                                       | Planned decision                         | Reason 
|--------|--------------------------------------------------------------------------------|--- ------------------------------------------- ------------ |------------------------------------------|-----------------------------------------------------------------------------------------
| P1 | Missing price for Pulse C, City 2, 2026-01-01, at index 5.                     | `df.isna().sum()` and `df[df.isna().any(axis=1)]`           | Retain and flag                      | The remaining information is useful, and there is no verified replacement price. 

| P2 | Exact duplicate at indices 8 and 96: Staple A, City 1, 2026-01-08, price 99.5. | `df.duplicated().sum()` and `df[df.duplicated(keep=False)]` | Exclude the extra copy               | All fields match. Keep first occurrence to avoid double-counting. 

| P3 | Both `Oil B` and `oil b` appear as item labels.                                | `df["item"].unique()`                                       | Investigate                          | Confirm they represent the same item before standardizing capitalization, then check for conflicting records. 

| P4 | The price column is stored as text.                                            | `df.info()`                                                 | Investigate, then correct the format | Identify conversion problems before making prices numeric, preserving the original values. 

| P5 | The date column is stored as text.                                             | `df.info()`                                                 | Investigate, then correct the format | Validate the date strings before converting them to datetime values. 

