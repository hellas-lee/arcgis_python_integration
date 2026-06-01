# ArcGIS + Python Snippets for Planners

A quick-reference collection of Python expressions and scripts for common GIS data tasks — built for planners working in ArcGIS Pro. Covers Field Calculator expressions, scoring and normalization, lookup tables, and data export workflows.

All Field Calculator snippets use the **Python parser** unless marked as a standalone script.

---

## Contents

| Category | What's in it |
|---|---|
| [String operations](#string-operations) | Extract substrings, concatenate fields, parse delimiters |
| [Numeric & formatting](#numeric--formatting) | Zero-padding, type conversion |
| [Conditional logic](#conditional-logic) | Null handling, range binning, flag fields |
| [Lookup & value mapping](#lookup--value-mapping) | Borough codes, category-to-numeric dictionaries |
| [Scoring & normalization](#scoring--normalization) | Linear normalization, percentile tiers |
| [Export](#export) | Multi-layer Excel workbook, batch CSV export |

Full runnable code with context is in [`arcgis_snippets.ipynb`](arcgis_snippets.ipynb).

---

## String operations

### Extract text after a delimiter
Splits a string on a marker (e.g. `--`) and returns everything after it. Strips a trailing keyword. Null-safe.
```python
def get_corner(value):
    if value is None:
        return None
    marker = "--"
    if marker not in value:
        return None
    part = value.split(marker, 1)[1].strip()
    part = part.replace("corner", "").strip()
    return part

get_corner(!YourFieldName!)
```
> Adapt `marker` and `.replace()` to match your delimiter and unwanted keyword.

---

### Extract last N characters
Returns the rightmost X characters of a string field — useful for isolating codes or suffixes.
```python
str(!YourFieldName!)[-X:]
```
> Replace `X` with the character count, e.g. `[-4:]` for the last 4.

---

### Concatenate two fields into a string
Combines lat/lon (or any two fields) into a formatted string.
```python
str(!LAT!) + ", " + str(!LON!)
```

---

## Numeric & formatting

### Zero-pad single-digit integers
Formats integers below 10 with a leading zero — useful for district codes, IDs, or ranked fields.
```python
def format_district(num):
    if num < 10:
        return f"0{num}"
    else:
        return str(num)

format_district(!YourFieldName!)
```
> One-liner alternative: `str(!YourFieldName!).zfill(2)`

---

## Conditional logic

### Replace null with zero
Returns the field value if populated, 0 if null. Prevents downstream math errors.
```python
!A! if !A! is not None else 0
```

---

### Sum two fields, treating nulls as zero
Adds two fields safely — nulls contribute 0 rather than breaking the calculation.
```python
(!A! if !A! is not None else 0) + (!B! if !B! is not None else 0)
```
> Extend with additional `(!C! if !C! is not None else 0)` terms for more fields.

---

### Output 0 if any of N fields is zero
Flags a record as ineligible if any one of several fields is zero — useful for composite suitability scoring.
```python
0 if (!field1! == 0 or !field2! == 0 or !field3! == 0) else 1
```

---

### Flag if any category field is non-zero
Returns 1 if at least one of a set of category fields is populated.
```python
1 if sum([
    !cat1!, !cat2!, !cat3!, !cat4!, !cat5!,
    !cat6!, !cat7!, !cat8!, !cat9!, !cat10!
]) > 0 else 0
```

---

### Categorize numeric ranges into output values
Assigns a score or code based on which range a value falls in.
```python
def get_value(x):
    if x < 100:
        return 0
    elif 100 <= x < 500:
        return 10
    elif 500 <= x < 1000:
        return 3
    else:
        return 1

get_value(!YourFieldName!)
```
> Adjust thresholds and return values to match your classification scheme.

---

## Lookup & value mapping

### NYC borough name → borough code
Converts a borough name field to its standard numeric code (1–5). Returns null for unmatched values.
```python
def get_borocode(borough):
    codes = {
        "Manhattan": 1,
        "Bronx": 2,
        "Brooklyn": 3,
        "Queens": 4,
        "Staten Island": 5
    }
    return codes.get(borough, None)

get_borocode(!borough!)
```

---

### Map categorical text to numeric values
Converts a text category field to an ordered numeric value via a dictionary lookup.
```python
value_map = {
    "Value A": 1,
    "Value B": 2,
    "Value C": 3,
    "Value D": 4,
    "Value E": 5
}
value_map.get(!Category!, None)
```
> Works for any ordinal mapping: land use types, priority levels, road classifications, etc.

---

## Scoring & normalization

### Normalize a value to a 0–50 scale
Linear normalization between a min and max threshold, with hard floor (0) and ceiling (50).
```python
def normalize(x):
    if x < 100:
        return 0
    elif x > 800:
        return 50
    else:
        return 50 * ((x - 100) / 700)

normalize(!YourFieldName!)
```
> Adjust `100`, `800`, and `50` to set your min threshold, max threshold, and output scale.

---

### Assign percentile tiers across multiple score fields
*(Standalone script — not for Field Calculator)*

For each score field, computes 33rd and 67th percentile cutoffs, then writes a tier (1 = low, 2 = mid, 3 = high) to a new `_TIER` field. See [`arcgis_snippets.ipynb`](arcgis_snippets.ipynb) for the full script.

---

## Export

Both export scripts are standalone — run them in ArcGIS Pro's Notebook environment or any Python environment with `arcpy`. See [`arcgis_snippets.ipynb`](arcgis_snippets.ipynb) for full code.

- **Export multiple layers → single Excel workbook**: Each layer becomes its own sheet. Requires `pandas` and `openpyxl`.
- **Export multiple layers → individual CSVs**: Uses native `arcpy.conversion.TableToTable`. No extra dependencies.

---

## Notes

- Tested in **ArcGIS Pro** with the default `arcpy` Python environment.
- Standalone scripts can be run in the ArcGIS Pro Notebook tab or in any environment where `arcpy` is available.
- Field Calculator expressions go in the **Expression** box with **Python** selected as the parser. Multi-line `def` blocks go in the **Code Block** box above the expression line.
