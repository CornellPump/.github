# Python Style Guide — Copilot PR Review Instructions

Review ONLY the changed lines in this PR. For Python files, flag issues in the
following categories only. Do NOT comment on logic, architecture, performance,
or anything outside these categories.

---

## 1. Indentation and Line Length

All indentation must use 4 spaces. Tabs are never permitted.
Lines must not exceed 120 characters.

### ✅ Correct
```python
def getAssetById(assetId: int) -> Asset:
    if assetId is not None:
        return db.query(assetId)
```

### ❌ Incorrect — flag these
```python
def getAssetById(assetId: int) -> Asset:
  if assetId is not None:        # 2 spaces — flag this
      return db.query(assetId)
```

```python
def getAssetById(assetId: int) -> Asset:
	if assetId is not None:      # tab — flag this
		return db.query(assetId)
```

```python
# Line exceeds 120 characters — flag this
result = someFunction(argumentOne, argumentTwo, argumentThree, argumentFour, argumentFive, argumentSix, argumentSeven)
```

---

## 2. Naming Conventions

| Identifier | Convention | Example |
|---|---|---|
| Functions | camelCase | `calculateOperatingTime` |
| Variables | camelCase | `lastValidTime` |
| Classes | TitleCase (all words capitalised, no separators) | `AssetOperatingRecord` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_OPERATING_HOURS` |
| Test functions (in `unit_test.py` or `IntegrationTests/`) | `test_` prefix + snake_case | `test_calculate_operating_time` |

### ✅ Correct
```python
MAX_OPERATING_HOURS = 24
DEFAULT_SOOT_THRESHOLD = 0.85

class AssetOperatingRecord:
    def calculateOperatingTime(self, assetId: int) -> None:
        lastValidTime = getLastValidTime(assetId)
        operatingHours = compute(lastValidTime)
```

### ❌ Incorrect — flag these
```python
max_operating_hours = 24          # constant in snake_case — flag
DEFAULT_SOOT_THRESHOLD = 0.85     # correct
```

```python
class asset_operating_record:     # class in snake_case — flag
    ...

class assetOperatingRecord:       # class in camelCase — flag
    ...
```

```python
def calculate_operating_time(asset_id: int) -> None:   # function in snake_case — flag
    last_valid_time = getLastValidTime(asset_id)        # variable in snake_case — flag
    operatingHours = compute(last_valid_time)
```

```python
def CalculateOperatingTime(assetId: int) -> None:      # function in TitleCase — flag
    ...
```

Test functions in `unit_test.py` or anywhere under `IntegrationTests/` must be
prefixed with `test_` and written in snake_case. Regular camelCase rules do NOT
apply to test functions.

### ✅ Correct — test functions
```python
# in unit_test.py or IntegrationTests/
def test_calculate_operating_time_when_asset_stops():
    ...

def test_get_asset_by_id_returns_none_for_missing():
    ...
```

### ❌ Incorrect — flag these test functions
```python
def calculateOperatingTime():        # camelCase, missing test_ prefix — flag both
    ...

def test_calculateOperatingTime():   # test_ present but camelCase body — flag
    ...

def testCalculateOperatingTime():    # no underscore separator — flag
    ...
```

---

## 3. Docstrings

Every **public** function (not prefixed with `_`) must have a multi-line docstring
immediately after the `def` line. Single-line docstrings are not permitted.
Private/internal functions (prefixed with `_`) do not require docstrings.

Do NOT review or comment on the content of the docstring — only check that the
correct structure is present.

### Pattern
```python
def functionName(param: type) -> returnType:
    """
    Any text here.
    """
```

### ✅ Correct
```python
def calculateOperatingTimeAfterAssetStops(
    assetId: int, time: datetime, lastValidTimes: dict, operatingTimes: dict
) -> None:
    """
    Any description at all.
    """
    ...

def _computeInternalOffset(assetId: int) -> int:
    # No docstring required — private function
    ...
```

### ❌ Incorrect — flag these
```python
# Missing docstring entirely — flag this
def calculateOperatingTime(assetId: int) -> None:
    operatingTimes[assetId] = compute(assetId)
```

```python
# Single-line docstring — flag this
def getAssetById(assetId: int) -> Asset:
    """ Retrieve an asset by ID. """
    ...
```

```python
# Docstring not immediately after def — flag this
def getAssetById(assetId: int) -> Asset:
    assetId = validate(assetId)
    """
    Retrieve an asset by ID.
    """
    ...
```

### Rules
- `"""` on its own line, content on the next line(s), closing `"""` on its own line
- Minimum structure: 3 lines total (`"""` / content / `"""`)
- Must appear on the line immediately after the `def` signature
- Do NOT flag or comment on what the docstring says

---

## 4. Type Hints

All parameters and return types must be annotated on every public and private function.
`-> None` is required when a function returns nothing.

### ✅ Correct
```python
def getAssetsByStatus(status: str, limit: int = 100) -> list[Asset]:
    ...

def updateOperatingTime(assetId: int, hours: float) -> None:
    ...
```

### ❌ Incorrect — flag these
```python
def getAssetsByStatus(status, limit=100):     # missing all annotations — flag
    ...

def updateOperatingTime(assetId: int, hours): # missing one param + return type — flag
    ...
```

---

## 5. String Formatting

Only f-strings are permitted for string interpolation.
`.format()` and `%`-style formatting must not be used.
Plain string concatenation for non-interpolated strings is fine.

### ✅ Correct
```python
message = f"Asset {assetId} stopped at {time}"
label = "No interpolation needed"           # plain string — fine
```

### ❌ Incorrect — flag these
```python
message = "Asset {} stopped at {}".format(assetId, time)   # .format() — flag
message = "Asset %s stopped at %s" % (assetId, time)       # % formatting — flag
message = "Asset " + str(assetId) + " stopped at " + str(time)  # concatenation with interpolation — flag
```

---

## 6. Function Length

If a function body exceeds 50 lines, flag it and suggest a logical split point based
on the structure of the code (e.g. distinct phases, separable sub-tasks, loops that
could become helpers).

### Output format for this check
- **Flag**: "This function is N lines — consider splitting"
- **Suggested split**: identify the line range and describe what the extracted function could be named and do
- Do NOT rewrite the function; suggest only

### Example flag
```
calculateOperatingTimeAfterAssetStops is 67 lines.
Consider extracting lines 34–67 (the loop that aggregates soot intervals)
into a helper function like accumulateSootIntervals(assetId, intervals) -> float.
```

---

## 7. List and Dict Comprehensions

If a `for` loop builds a list or dict and its entire body is a **single expression**
(no conditionals on the accumulation step, no multi-line logic), flag it and suggest
the equivalent comprehension. Do NOT flag multi-line or complex loops.

### ✅ Correct — comprehension already used
```python
activeIds = [asset.id for asset in assets if asset.isActive]
statusMap = {asset.id: asset.status for asset in assets}
```

### ✅ Correct — loop is too complex to flag
```python
results = []
for asset in assets:
    value = compute(asset)
    if value > threshold:
        log(asset)
        results.append(value)
```

### ❌ Flag this — single-expression loop body
```python
# Flag — suggest: activeIds = [asset.id for asset in assets]
activeIds = []
for asset in assets:
    activeIds.append(asset.id)
```

```python
# Flag — suggest: statusMap = {asset.id: asset.status for asset in assets}
statusMap = {}
for asset in assets:
    statusMap[asset.id] = asset.status
```

---

## 8. Comments

All inline and standalone comments must use `#` and be written as full sentences —
capitalised first word and a period at the end. Do NOT flag docstrings under this rule.

### ✅ Correct
```python
# Retrieve the last valid time before the asset stopped.
lastTime = lastValidTimes.get(assetId)

result = compute(assetId)  # Return early if the asset has no operating history.
```

### ❌ Incorrect — flag these
```python
# get last valid time          # not a full sentence, no capital, no period — flag
lastTime = lastValidTimes.get(assetId)

result = compute(assetId)  # return early    # lowercase, no period — flag

# TODO: fix this later         # not a full sentence — flag
```

### Rules
- Must start with `# ` (hash + space)
- First word capitalised
- Must end with a period
- Flag `#word` with no space after `#`
- Do NOT apply this rule to docstrings (`"""`)

---

## 9. Null / None Safety

Flag any location where a value that could be `None` is accessed without a prior guard.
Do NOT prescribe a specific guard style — flag the issue and leave the fix to the developer.

### Sources that can produce None — always treat as potentially None
- `dict.get(key)` — returns `None` if key is absent
- Direct dict access `dict[key]` — raises `KeyError`; flag if the key's presence is not guaranteed
- Function parameters or return types annotated as `Optional[X]` or `X | None`
- Any attribute on a model defined in `models.py` that is nullable (read `models.py` to determine this)

### ✅ Correct — guarded before use
```python
# dict.get()
lastTime = lastValidTimes.get(assetId)
if lastTime is not None:
    duration = currentTime - lastTime

# Optional parameter
def process(asset: Asset | None) -> None:
    if asset is None:
        return
    doWork(asset.id)

# Walrus operator — also acceptable
if operatingTime := operatingTimes.get(assetId):
    logTime(operatingTime)
```

### ❌ Incorrect — flag these
```python
# dict.get() result used without guard
lastTime = lastValidTimes.get(assetId)
duration = currentTime - lastTime          # lastTime could be None — flag

# Direct dict access without guaranteed key
status = assetMap[assetId].status          # assetMap[assetId] could raise or be None — flag

# Optional type used without guard
def process(asset: Asset | None) -> None:
    doWork(asset.id)                       # asset could be None — flag
```

### Checking model definitions
Before flagging null safety issues involving model attributes, read `models.py` to
determine whether that attribute is nullable. Only flag if the field is defined as
`Optional[X]`, `X | None`, or has `nullable=True` (or equivalent ORM annotation).

---

## 10. Inline SQL

SQL strings embedded in Python must be assigned to a SCREAMING_SNAKE_CASE constant
and written as a triple-quoted `"""` block. SQL keywords and clauses must follow the
formatting rules below. Do NOT flag SQL that is dynamically built via string
formatting — only flag static SQL string assignments.

### Rules
- Assigned to a module-level constant (`SCREAMING_SNAKE_CASE = """..."""`)
- Opening `"""` on the same line as the constant assignment; closing `"""` on its own line
- `SELECT`, `FROM`, `WHERE`, `JOIN`, `ON`, `GROUP BY`, `ORDER BY`, `HAVING`, `LIMIT` each on their own line, in UPPERCASE
- Each selected column on its own indented line
- Each `JOIN` on its own indented line
- Column aliases use backtick quoting: `` a.asset_id `assetId` ``
- Statement ends with `;` inside the closing `"""`

### ✅ Correct
```python
DEVICE_IDENTIFIER_SQL = """
SELECT
    a.asset_id `assetId`,
    d.identifier `deviceIdentifier`
FROM
    rpm.tmp_asset_ids tmp
    JOIN rpm.asset a ON (a.asset_id = tmp.asset_id)
    JOIN rpm.device d ON(d.device_id = a.device_id)
    JOIN rpm.asset_company ac ON(ac.asset_id = a.asset_id)
    JOIN rpm.company c ON(c.company_id = ac.company_id)
WHERE
    c.code = 'RAIN';
"""
```

### ❌ Incorrect — flag these
```python
# All on one line — flag
deviceIdentifierSql = "SELECT a.asset_id, d.identifier FROM rpm.asset a JOIN rpm.device d ON d.device_id = a.device_id"
```

```python
# Not assigned to a constant (camelCase variable) — flag
deviceIdentifierSql = """
SELECT
    a.asset_id `assetId`
FROM
    rpm.asset a;
"""
```

```python
# Keywords not on their own lines — flag
DEVICE_SQL = """
SELECT a.asset_id, d.identifier FROM rpm.asset a
JOIN rpm.device d ON d.device_id = a.device_id WHERE a.active = 1;
"""
```

---

## Output Format

For each issue found, respond with:

- **File**: `path/to/file.py`
- **Line**: 42
- **Category**: Indentation | Line Length | Naming | Test Naming | Docstring | Type Hints | String Formatting | Comments | Function Length | Comprehension | Inline SQL | Null Safety
- **Issue**: one-sentence description
- **Suggestion**: minimal fix or split point (for Function Length: describe the extracted function, do not rewrite)

If no issues are found in a category, output: `✅ No [Category] issues found.`
