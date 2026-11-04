# JSON Entry Management Instructions

## Objective
Parse a task text file containing Tier1 and Tier2 data and append new entries to the `log` array in `dsky.json`, maintaining strict alphabetical ordering by the `r1` field.

## Operational Constraints
- **Append Only:** Never delete or modify existing entries in `dsky.json`.
- **No Overwrites:** When updating the file, ensure all existing entries (including their original metadata like timestamps) are preserved.
- **Alphabetical Sort:** The `log` array must be sorted alphabetically by the `r1` value after every update.

## Input Data Format (Task File)
- Data is structured as:
  `tier1: [Value]`
  `tier2: [Value 1]`
  `[Value 2]`
- Each `tier1` may have one or more `tier2` values following it.

## Execution Logic
1. **Parse:**
   - Identify each `tier1` value.
   - Collect all subsequent `tier2` values until the next `tier1` marker is encountered.
2. **Transform:**
   - For every `tier2` value associated with a `tier1`, create a JSON object: `{"r1": "tier1_value tier2_value"}`.
3. **Integrate:**
   - Read the existing `log` array from `dsky.json`.
   - Merge the new objects into the existing array.
4. **Sort:**
   - Sort the resulting `log` array alphabetically based on the `r1` string.
5. **Commit:**
   - Write the updated JSON object back to `dsky.json` ensuring `constant` and `trail` fields remain unchanged.

## Verification
- Ensure no duplicate entries are created if the same Tier1/Tier2 pair exists.
- Verify that `r1` strings are constructed exactly as `[Tier1] [Tier2]` with a single space separator.
