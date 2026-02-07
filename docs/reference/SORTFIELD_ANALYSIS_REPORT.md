# SortField Pattern Analysis Report

**Date:** February 4, 2026  
**Tables Analyzed:**
- `raw_ext_sap.sap_floc_eastover`
- `raw_ext_sap.sap_floc_sumter`

## Executive Summary

Analysis of `sort_field` values from two SAP functional location tables reveals distinct patterns between Eastover and Sumter locations. Eastover has a much larger dataset with primarily numeric and alphanumeric equipment codes, while Sumter uses more descriptive alphanumeric names with spaces and special characters.

## Key Findings

### Eastover (`sap_floc_eastover`)

- **Total Rows Analyzed:** 5,000 (sample)
- **Non-null sortField Values:** 3,350
- **Unique sortField Values:** 3,251

**Pattern Distribution:**
- **Alphanumeric:** 1,628 unique values (50.1%)
  - Examples: `31200L16B`, `460LP602`, `471LP490`, `501LP1168A`
  - Pattern: Mix of numbers and letters, often with embedded letters (L, LP)
- **Numeric:** 1,576 unique values (48.5%)
  - Examples: `4710052317`, `441005075`, `321005076`, `4920051517`
  - Pattern: 9-10 digit numeric codes, likely equipment identifiers
- **With Special Characters:** 47 unique values (1.4%)
  - Examples: `312SRV039-01`, `471-8-1331-03`, `321-SP-BB`, `505.14`
  - Pattern: Hyphens, dots, or other separators

**Characteristics:**
- Most values are unique (only 3 values appear 3 times each)
- Highly structured codes suggesting systematic equipment numbering
- Numeric codes appear to follow a hierarchical pattern (e.g., `441005xxx` series)

### Sumter (`sap_floc_sumter`)

- **Total Rows Analyzed:** 1,067 (all rows)
- **Non-null sortField Values:** 83
- **Unique sortField Values:** 82

**Pattern Distribution:**
- **Alphanumeric:** 65 unique values (79.3%)
  - Examples: `1DUSTFAN`, `3BPACKAGING`, `1BPACKERLIDDER`, `2AREAMINSPECTOR`
  - Pattern: Descriptive names with numeric prefixes (area/line identifiers)
- **With Special Characters:** 10 unique values (12.2%)
  - Examples: `BALERROOM FAN HOUSES`, `#1SHEETERCRANE`, `BALEMASTER #1`, `CORESTRIP SHREDDER`
  - Pattern: Spaces, hash symbols, descriptive equipment names
- **Alphabetic:** 7 unique values (8.5%)
  - Examples: `CORESTRIPPER`, `BUILDINGS`, `BOILERROOM`, `MISCEQUIP`
  - Pattern: Pure text descriptions

**Characteristics:**
- More descriptive, human-readable naming convention
- Uses area/line prefixes (1A, 1B, 2A, 2B, 3A, 3B) followed by equipment type
- Some values contain spaces and special characters
- Most values are unique (only `BALERROOM FAN HOUSES` appears twice)

## Pattern Types Identified

### 1. Numeric Codes (Eastover: 48.5%)
- **Format:** 9-10 digit numbers
- **Examples:** `4710052317`, `441005075`, `321005076`
- **File Matching:** 
  - Extract numeric sequences from file names
  - May require exact match or leading zero handling
  - Could appear as part of longer numeric strings

### 2. Alphanumeric Codes (Both locations: 50.1% Eastover, 79.3% Sumter)
- **Eastover Format:** Numbers with embedded letters (e.g., `31200L16B`, `460LP602`)
- **Sumter Format:** Area prefix + descriptive name (e.g., `1DUSTFAN`, `2BWRAPPER`)
- **File Matching:**
  - Case-sensitive matching likely needed for Eastover codes
  - Sumter codes may match equipment descriptions in file names
  - Substring search may work for Sumter (e.g., "DUSTFAN" in filename)

### 3. Values with Special Characters (Eastover: 1.4%, Sumter: 12.2%)
- **Formats:** 
  - Hyphens: `312SRV039-01`, `471-8-1331-03`
  - Spaces: `BALERROOM FAN HOUSES`, `CORESTRIP SHREDDER`
  - Hash symbols: `#1SHEETERCRANE`
  - Dots: `505.14`
- **File Matching:**
  - Normalize file names (remove/replace special chars)
  - May require fuzzy matching for variations
  - Spaces in Sumter values suggest they may match descriptive file names

### 4. Alphabetic Codes (Sumter: 8.5%)
- **Format:** Pure text descriptions
- **Examples:** `CORESTRIPPER`, `BUILDINGS`, `BOILERROOM`
- **File Matching:**
  - May match equipment type names in file names
  - Case-insensitive matching recommended
  - Substring search could work

## File Matching Recommendations

### Strategy 1: Exact Match (Primary)
- **Best for:** Numeric codes, structured alphanumeric codes
- **Implementation:** Direct string comparison after normalization
- **Normalization:** 
  - Remove spaces
  - Convert to uppercase
  - Remove/replace special characters (hyphens, dots, hash)

### Strategy 2: Substring Search (Secondary)
- **Best for:** Sumter descriptive names, alphabetic codes
- **Implementation:** Check if sortField value appears anywhere in file name
- **Use cases:**
  - `DUSTFAN` matches `"Equipment_DUSTFAN_2024.pdf"`
  - `BALERROOM FAN HOUSES` matches `"BALERROOM_FAN_HOUSES_MAINTENANCE.pdf"`

### Strategy 3: Pattern Extraction (Advanced)
- **Best for:** Numeric codes embedded in file names
- **Implementation:** Use regex to extract numeric sequences
- **Example:** Extract `4710052317` from `"EQ_4710052317_REPORT_2024.xlsx"`

### Strategy 4: Fuzzy Matching (Fallback)
- **Best for:** Variations, typos, partial matches
- **Implementation:** Use similarity algorithms (Levenshtein, Jaro-Winkler)
- **Threshold:** 80-90% similarity for matches

## Implementation Considerations

### Case Sensitivity
- **Eastover:** Likely case-sensitive (codes like `31200L16B` use specific casing)
- **Sumter:** Case-insensitive recommended (descriptive names)

### Normalization Rules
1. Convert to uppercase for comparison
2. Remove spaces
3. Replace hyphens with underscores or remove
4. Remove hash symbols (#)
5. Handle dots (may be decimal separators or delimiters)

### Performance Optimization
- Index sortField values for fast lookups
- Cache unique values and their patterns
- Pre-compute normalized versions
- Use hash tables for exact match lookups

### Handling Multiple Matches
- If a file matches multiple sortFields, consider:
  - Using the most specific match (longest substring)
  - Using the most common sortField value
  - Creating multiple file-to-sortField relationships
  - Prompting user for selection

### Location-Specific Handling
- **Eastover:** Focus on exact numeric/alphanumeric code matching
- **Sumter:** Use substring search for descriptive names
- Consider different matching strategies per location

## Sample Matching Examples

### Eastover Examples
```
sortField: "4710052317"
File: "EQ_4710052317_MAINTENANCE_LOG.pdf" → MATCH ✓

sortField: "31200L16B"
File: "Equipment_31200L16B_Report.xlsx" → MATCH ✓

sortField: "471-8-1331-03"
File: "471_8_1331_03_DOCUMENT.pdf" → MATCH (after normalization) ✓
```

### Sumter Examples
```
sortField: "1DUSTFAN"
File: "1DUSTFAN_MAINTENANCE_2024.pdf" → MATCH ✓

sortField: "BALERROOM FAN HOUSES"
File: "BALERROOM_FAN_HOUSES.pdf" → MATCH (after normalization) ✓

sortField: "CORESTRIPPER"
File: "CoreStripper_Equipment_Manual.pdf" → MATCH (case-insensitive) ✓
```

## Next Steps

1. **Obtain Sample File Names:** Get actual file names from the system to validate matching strategies
2. **Test Matching Algorithms:** Implement and test each strategy with real file names
3. **Define Matching Rules:** Create location-specific matching rules based on file naming conventions
4. **Build Matching Service:** Implement a service that can match files to sortField values
5. **Validate Results:** Review matches manually to ensure accuracy

## Conclusion

The sortField values show distinct patterns between Eastover and Sumter locations:
- **Eastover** uses structured numeric/alphanumeric equipment codes suitable for exact matching
- **Sumter** uses descriptive names that may require substring or fuzzy matching

A hybrid approach combining exact matching (for Eastover) and substring/fuzzy matching (for Sumter) would likely yield the best results. The key is understanding the actual file naming conventions used in each location to fine-tune the matching strategy.
