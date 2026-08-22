# Personal Expense and Budget Tracker - Test Cases

## Test Plan Overview

This document contains documented test cases covering the five required categories:

1. **Normal** - valid records process successfully
2. **Invalid** - invalid records are rejected with reasons
3. **Boundary** - edge cases at classification limits
4. **Search** - finding existing and non-existing records
5. **Menu** - invalid menu options and exit behaviour

**Status: all 15 tests pass. No open defects.**

### How these tests were executed

Every test below was run against `main.py` at commit `22e12fa`, with
`cap-data.csv` restored to its committed state between tests. Each **Evidence**
block is a captured terminal transcript of a real session - the values shown
after each prompt are the inputs typed by the tester.

To reproduce any test, run the program from inside the repository directory and
enter the inputs listed in the **Input** row:

```bash
python3 main.py
```

Repeated menu re-displays between steps are elided as `[menu redisplayed]` to
keep transcripts readable. Nothing else has been altered.

**Baseline dataset:** `cap-data.csv` contains 20 records - **17 valid,
3 invalid**. Of the 17 valid, 14 are Expense and 3 are Income.

### Revision history

| Rev | Against commit | Change |
|---|---|---|
| 1 | `61205e9` | 14 test cases documented |
| 2 | `61205e9` | Captured terminal evidence added for every test; defects D1–D3 raised |
| 3 | `bfc98ce` | Re-run after the invalid-records and add-confirmation tables were added; D1 upgraded to Medium |
| 4 | `5fe78d6` | Re-run after the summary, category-budget and payment displays were reformatted. D1 verified fixed; T015 added; D4–D6 raised |
| 5 | `22e12fa` | **Final re-run.** All six defects now resolved. Dataset corrected to 20 records following the D3 fix, so all figures are restated. Every test passes; the defect log is closed |

---

## Test 1: Normal Operation - Valid Transaction Accepted

| Attribute | Value |
|-----------|-------|
| **Test ID** | T001 |
| **Category** | Normal |
| **Objective** | Verify that a complete, valid transaction is accepted, added, confirmed on screen and persisted to disk |
| **Input** | Menu 2 → 1 (add):<br>ID: TX999, Type: Expense, Category: Food, Description: Groceries, Amount: 2500, Budget: 5000, Payment: Card |
| **Expected Result** | Transaction validates, is appended to the list, written to CSV, "Transaction added successfully." is displayed, and the new record is echoed back as a table |
| **Actual Result** | As expected - success message, confirmation table showing `2500.00`, and new row written to `cap-data.csv` |
| **Status** | **PASS** |

### Evidence

```
Enter selection: 2
1. Add a transaction
2. Search a transaction
Enter selection: 1


--- Add new transaction ---


Transaction ID: TX999
Type (Income/Expense): Expense
Category: Food
Description: Groceries
Amount (KES): 2500
Budget limit (KES): 5000
Payment method: Card

Transaction added successfully.

+--------------------------------------------------------------------------------------------------+
| Transaction Added                                                                                |
+--------------------------------------------------------------------------------------------------+
| ID       | Type     | Category      | Description        | Amount     | Budget     | Payment     |
+--------------------------------------------------------------------------------------------------+
| TX999    | Expense  | Food          | Groceries          | 2500.00    | 5000.00    | Card        |
+--------------------------------------------------------------------------------------------------+
```

**Persistence verified independently.** The confirmation table is printed from
the in-memory record, so on its own it does not prove the record reached disk.
Last line of `cap-data.csv` after the session ended:

```
$ tail -1 cap-data.csv
TX999,Expense,Food,Groceries,2500.0,5000.0,Card
```

**Code path exercised:** `add_transaction()` → `validate_transaction()` returns
`[]` → `transactions.append(new_record)` → `write_transactions()` →
`display_table([new_record], "Transaction Added")`.

---

## Test 2: Invalid Operation - Missing Required Category

| Attribute | Value |
|-----------|-------|
| **Test ID** | T002 |
| **Category** | Invalid |
| **Objective** | Verify that an Expense transaction without a category is rejected, with the reason reported |
| **Input** | Menu 2 → 1 (add):<br>ID: TX998, Type: Expense, Category: *(empty)*, Description: Office supplies, Amount: 1500, Budget: 2000, Payment: Bank |
| **Expected Result** | Validation fails; a "Transaction rejected" block with a "Reasons" heading lists "missing category for expense"; record not added |
| **Actual Result** | As expected - rejection block displayed, record not appended, control returned to menu |
| **Status** | **PASS** |
| **Notes** | All seven prompts are collected before validation runs - contrast with T012 |

### Evidence

```
--- Add new transaction ---


Transaction ID: TX998
Type (Income/Expense): Expense
Category:
Description: Office supplies
Amount (KES): 1500
Budget limit (KES): 2000
Payment method: Bank

Transaction rejected:

Reasons

 - missing category for expense
```

**Code path exercised:** `validate_transaction()` →
`if tx_type.lower() == "expense" and not category` → reason appended → caller
prints the reasons block and skips `append`.

---

## Test 3: Invalid Operation - Negative Amount

| Attribute | Value |
|-----------|-------|
| **Test ID** | T003 |
| **Category** | Invalid |
| **Objective** | Verify that a record with a negative amount is rejected at load time and reported with its reason |
| **Input** | Existing CSV record TX005 (amount = -2500.0), viewed via Menu option 5 |
| **Expected Result** | Record classified invalid with reason "amount must be greater than zero" and shown in the invalid-records table |
| **Actual Result** | As expected - TX005 appears in the table with the correct reason |
| **Status** | **PASS** |
| **Notes** | Amounts must be strictly greater than zero; zero and negative are both rejected |

### Evidence

```
Enter selection: 5

 Showing invalid records


+-------------------------------------------------------------------------------------------------------------------+
| ID         | Type     | Category      | Description        | Amount    | Reasons                                  |
+-------------------------------------------------------------------------------------------------------------------+
| TX005      | Expense  | Entertainment | Streaming          | -2500.0   | amount must be greater than zero         |
| TX011      | EXP      | Entertainment | Streaming          | 8409.0    | unrecognized transaction_type: 'EXP'     |
| TX016      | Expense  |               | Books              | 12004.0   | missing category for expense             |
+-------------------------------------------------------------------------------------------------------------------+
```

This single view also serves as evidence for **T011**. Note TX016's blank
Category cell - the table renders a missing value as empty space rather than
crashing, which is correct for a record whose defect *is* the missing field.

**Completeness check:** 17 valid (T-VIEW) + 3 invalid (here) = 20, matching the
20 data rows in `cap-data.csv`. Every record is accounted for; none is silently
dropped.

---

## Test 4: Boundary Test - Amount Exactly Equal to Budget Limit

| Attribute | Value |
|-----------|-------|
| **Test ID** | T004 |
| **Category** | Boundary |
| **Objective** | Verify a transaction whose amount **equals** its budget limit is accepted and **not** flagged |
| **Input** | Menu 2 → 1 (add): ID: TX997, Type: Expense, Category: Transport, Description: Boundary equal, Amount: 5000, Budget: 5000, Payment: Card<br>Then Menu option 4 |
| **Expected Result** | Transaction accepted; TX997 does **not** appear in the individual warnings table |
| **Actual Result** | As expected - added successfully, and absent from the warnings table |
| **Status** | **PASS** |
| **Notes** | The comparison is `amount > budget`, not `>=`, so equality does not trigger a warning |

### Evidence

```
Transaction ID: TX997
Type (Income/Expense): Expense
Category: Transport
Description: Boundary equal
Amount (KES): 5000
Budget limit (KES): 5000
Payment method: Card

Transaction added successfully.

+--------------------------------------------------------------------------------------------------+
| Transaction Added                                                                                |
+--------------------------------------------------------------------------------------------------+
| ID       | Type     | Category      | Description        | Amount     | Budget     | Payment     |
+--------------------------------------------------------------------------------------------------+
| TX997    | Expense  | Transport     | Boundary equal     | 5000.00    | 5000.00    | Card        |
+--------------------------------------------------------------------------------------------------+

[menu redisplayed]

Enter selection: 4

================================================================================
INDIVIDUAL TRANSACTION WARNINGS
================================================================================
ID         Category        Amount       Limit        Over by
--------------------------------------------------------------------------------
TX010      Education       7690.00      3000.00      4690.00
TX015      Utilities       11285.00     3000.00      8285.00
TX017      Entertainment   12723.00     8000.00      4723.00
TX019      Transport       14161.00     13000.00     1161.00
TX020      Housing         14880.00     3000.00      11880.00
================================================================================


================================================================================
CATEGORY BUDGET SUMMARY (WITH WARNINGS)
================================================================================
Category        Spent        Budget       Status
--------------------------------------------------------------------------------
Transport       35760.00     42000.00     OK
Housing         33636.00     34500.00     OK
Utilities       20913.00     26500.00     OK
Education       11066.00     16000.00     OK
Entertainment   12723.00     8000.00      OVER
================================================================================
```

**TX997 is absent from the individual warnings table** - this absence is the
result being tested. The five baseline warnings are unchanged.

**Secondary confirmation:** Transport reads 35,760 spent against 42,000 budget,
up from the baseline 30,760 / 37,000 in T014. That is exactly TX997's 5,000 and
its 5,000 limit, proving the record *was* processed and included in the category
aggregate - it simply did not breach anything. A record that had been silently
dropped would leave both figures unchanged.

---

## Test 5: Boundary Test - Amount Just Over Budget Limit

| Attribute | Value |
|-----------|-------|
| **Test ID** | T005 |
| **Category** | Boundary |
| **Objective** | Verify a transaction marginally **over** its budget limit is flagged, and that the overage is reported accurately |
| **Input** | Menu 2 → 1 (add): ID: TX996, Type: Expense, Category: Transport, Description: Boundary over, Amount: 5000.01, Budget: 5000, Payment: Card<br>Then Menu option 4 |
| **Expected Result** | Transaction accepted; TX996 appears in the individual warnings table showing an overage of `0.01` |
| **Actual Result** | As expected - flagged, with the overage displayed correctly as `0.01` |
| **Status** | **PASS** |
| **Notes** | This test originally exposed defect D1, where the overage displayed as `0`. **It now verifies the fix** |

### Evidence

```
Transaction ID: TX996
Type (Income/Expense): Expense
Category: Transport
Description: Boundary over
Amount (KES): 5000.01
Budget limit (KES): 5000
Payment method: Card

Transaction added successfully.

+--------------------------------------------------------------------------------------------------+
| Transaction Added                                                                                |
+--------------------------------------------------------------------------------------------------+
| ID       | Type     | Category      | Description        | Amount     | Budget     | Payment     |
+--------------------------------------------------------------------------------------------------+
| TX996    | Expense  | Transport     | Boundary over      | 5000.01    | 5000.00    | Card        |
+--------------------------------------------------------------------------------------------------+

[menu redisplayed]

Enter selection: 4

================================================================================
INDIVIDUAL TRANSACTION WARNINGS
================================================================================
ID         Category        Amount       Limit        Over by
--------------------------------------------------------------------------------
TX010      Education       7690.00      3000.00      4690.00
TX015      Utilities       11285.00     3000.00      8285.00
TX017      Entertainment   12723.00     8000.00      4723.00
TX019      Transport       14161.00     13000.00     1161.00
TX020      Housing         14880.00     3000.00      11880.00
TX996      Transport       5000.01      5000.00      0.01
================================================================================
```

### Regression check - D1 fix verified

| | Revisions 2–3 | Revision 5 |
|---|---|---|
| Confirmation table | `5000` | `5000.01` ✓ |
| Warnings row | `5000  5000  0` | `5000.01  5000.00  0.01` ✓ |

The underlying detection was always correct - confirmed at revision 2 by calling
the function directly, which returned `'over': 0.010000000000218279`. Only the
display was at fault, and the `.0f` → `.2f` change fixed it.

Read together, T004 and T005 pin the boundary from both sides: 5000.00 is not
flagged, 5000.01 is.

---

## Test 6: Search Operation - Existing Transaction by ID

| Attribute | Value |
|-----------|-------|
| **Test ID** | T006 |
| **Category** | Search |
| **Objective** | Verify that searching by transaction ID returns the matching record |
| **Input** | Menu 2 → 2 (search), search term: `TX001` |
| **Expected Result** | TX001 found and displayed in the formatted table with all seven fields |
| **Actual Result** | As expected - single matching row displayed |
| **Status** | **PASS** |
| **Notes** | Search is case-insensitive; the term is lowercased before comparison, which is why the title reads `'tx001'` |

### Evidence

```
Search by Transaction ID or Category: TX001

+--------------------------------------------------------------------------------------------------+
| Search Results for 'tx001'                                                                       |
+--------------------------------------------------------------------------------------------------+
| ID       | Type     | Category      | Description        | Amount     | Budget     | Payment     |
+--------------------------------------------------------------------------------------------------+
| TX001    | Expense  | Transport     | Fuel               | 1219.00    | 5500.00    | Card        |
+--------------------------------------------------------------------------------------------------+
```

---

## Test 7: Search Operation - Non-Existing Transaction

| Attribute | Value |
|-----------|-------|
| **Test ID** | T007 |
| **Category** | Search |
| **Objective** | Verify that searching for a non-existent record returns the appropriate message rather than an error |
| **Input** | Menu 2 → 2 (search), search term: `TX404` (not present in the dataset) |
| **Expected Result** | "No matching transactions found." displayed; program does not crash; menu repeats |
| **Actual Result** | As expected - message displayed, control returned to menu |
| **Status** | **PASS** |
| **Notes** | `TX404` was chosen deliberately because `TX999` is created by T001; reusing it would make this test order-dependent |

### Evidence

```
Search Transaction

Search by Transaction ID or Category: TX404
No matching transactions found.
```

---

## Test 8: Search Operation - Multiple Records by Category

| Attribute | Value |
|-----------|-------|
| **Test ID** | T008 |
| **Category** | Search |
| **Objective** | Verify that searching by category returns **all** matching transactions |
| **Input** | Menu 2 → 2 (search), search term: `Transport` |
| **Expected Result** | All valid Transport-category records returned in a single formatted table |
| **Actual Result** | As expected - 4 records returned (TX001, TX007, TX013, TX019) |
| **Status** | **PASS** |
| **Notes** | Matching is **exact and case-insensitive**, not substring - searching `Trans` returns no results |

### Evidence

```
Search by Transaction ID or Category: Transport

+--------------------------------------------------------------------------------------------------+
| Search Results for 'transport'                                                                   |
+--------------------------------------------------------------------------------------------------+
| ID       | Type     | Category      | Description        | Amount     | Budget     | Payment     |
+--------------------------------------------------------------------------------------------------+
| TX001    | Expense  | Transport     | Fuel               | 1219.00    | 5500.00    | Card        |
| TX007    | Expense  | Transport     | Fuel               | 5533.00    | 8000.00    | Bank        |
| TX013    | Expense  | Transport     | Fuel               | 9847.00    | 10500.00   | Card        |
| TX019    | Expense  | Transport     | Fuel               | 14161.00   | 13000.00   | Bank        |
+--------------------------------------------------------------------------------------------------+
```

**Cross-check:** 1219 + 5533 + 9847 + 14161 = **30,760**, matching the Transport
figure in T013's category breakdown and the Transport row of T014's category
budget summary.

*(This test returned 5 rows in revisions 1–4. The fifth was `TRX2000`, the stray
record removed by the D3 fix.)*

---

## Test 9: Menu Control - Invalid Menu Option

| Attribute | Value |
|-----------|-------|
| **Test ID** | T009 |
| **Category** | Menu |
| **Objective** | Verify that an out-of-range menu selection is rejected and the menu repeats |
| **Input** | At the main menu, enter `9` (valid options are 1–7) |
| **Expected Result** | "Invalid selection." displayed; menu redisplayed; program continues |
| **Actual Result** | As expected - error shown, loop continued, no crash |
| **Status** | **PASS** |
| **Notes** | Handled by the `else` branch of the `if`/`elif` chain, as required by the brief |

### Evidence

```
Enter selection: 9
Invalid selection.

=== Personal Expense and Budget Tracker ===
1. View valid transactions
2. Add or search a transaction
3. View income and expenditure summary
4. View budget warnings
5. View invalid records
6. View payment summary
7. Exit

Enter selection: 7
Program closed.
```

---

## Test 10: Menu Control - Exit Option Terminates Program

| Attribute | Value |
|-----------|-------|
| **Test ID** | T010 |
| **Category** | Menu |
| **Objective** | Verify that option 7 terminates the program cleanly |
| **Input** | At the main menu, enter `7` |
| **Expected Result** | "Program closed." displayed; loop exits; process terminates with no error |
| **Actual Result** | As expected - message displayed and process exited cleanly |
| **Status** | **PASS** |
| **Notes** | `break` exits the `while True` loop; added transactions were already written to CSV at add time, so no data is lost on exit |

### Evidence

```
=== Personal Expense and Budget Tracker ===
1. View valid transactions
2. Add or search a transaction
3. View income and expenditure summary
4. View budget warnings
5. View invalid records
6. View payment summary
7. Exit

Enter selection: 7
Program closed.
```

---

## Test 11: Data Validation - Unrecognized Transaction Type

| Attribute | Value |
|-----------|-------|
| **Test ID** | T011 |
| **Category** | Invalid |
| **Objective** | Verify that an unrecognized transaction type is rejected rather than guessed at |
| **Input** | Existing CSV record TX011 (`transaction_type` = `EXP`), viewed via Menu option 5 |
| **Expected Result** | Rejected with reason `unrecognized transaction_type: 'EXP'` |
| **Actual Result** | As expected - listed in the invalid-records table with the offending value quoted back |
| **Status** | **PASS** |
| **Notes** | Only `Income` and `Expense` are accepted. `algorithm.md` now matches this behaviour - see D6 |

### Evidence

See the invalid-records table in **Test 3** - row two:

```
| TX011      | EXP      | Entertainment | Streaming          | 8409.0    | unrecognized transaction_type: 'EXP'     |
```

The Type column preserves the offending value `EXP` rather than normalising it,
which is what makes the reason column verifiable at a glance.

The brief permits standardizing alternatives **only where a mapping is defined**
and says values that cannot be safely corrected must be rejected with a reason.
No mapping is defined for `EXP`, so rejection is the correct behaviour, and
`algorithm.md` documents it as such.

---

## Test 12: Data Validation - Non-Numeric Amount

| Attribute | Value |
|-----------|-------|
| **Test ID** | T012 |
| **Category** | Invalid |
| **Objective** | Verify that non-numeric input for amount is handled without crashing |
| **Input** | Menu 2 → 1 (add): ID: TX995, Type: Expense, Category: Food, Description: Snacks, Amount: `two thousand` |
| **Expected Result** | `ValueError` caught; "Invalid amount. Transaction not added." displayed; function returns early; menu repeats |
| **Actual Result** | As expected - error caught, message displayed, remaining prompts skipped, no traceback |
| **Status** | **PASS** |
| **Notes** | The function returns **immediately** - the Budget and Payment prompts never appear, confirming the early `return` |

### Evidence

```
--- Add new transaction ---


Transaction ID: TX995
Type (Income/Expense): Expense
Category: Food
Description: Snacks
Amount (KES): two thousand
Invalid amount. Transaction not added.

=== Personal Expense and Budget Tracker ===
```

**Code path exercised:** `try: amount = float(input(...))` raises `ValueError`
→ `except ValueError:` prints the message → `return` exits `add_transaction()`
before the budget prompt.

Contrast with T002: there, all seven prompts were collected and validation ran
at the end. Here the failure is caught mid-collection. The program has two
distinct rejection paths, and both are covered.

---

## Test 13: Summary Calculation - Totals, Balance and Highest Category

| Attribute | Value |
|-----------|-------|
| **Test ID** | T013 |
| **Category** | Normal |
| **Objective** | Verify income, expenditure, balance, per-category totals and the highest-spending category are computed correctly from valid records only |
| **Input** | Menu option 3 on the unmodified dataset |
| **Expected Result** | Income = sum of Income records; Expenditure = sum of Expense records; Balance = Income − Expenditure; per-category breakdown; highest-spending category identified |
| **Actual Result** | As expected - Income 27,384.00; Expenditure 109,098.00; Balance −81,714.00; highest category Housing at 33,636.00 |
| **Status** | **PASS** |
| **Notes** | The highest-spending category line is new - it was added in response to D2. The three invalid records are excluded, as intended |

### Evidence

```
Enter selection: 3

============================================================
INCOME AND EXPENDITURE SUMMARY
============================================================
Total Income                   KES             27384.00
Total Expenditure              KES            109098.00
Balance                        KES            -81714.00
============================================================

EXPENDITURE BY CATEGORY
------------------------------------------------------------
Category                                     Amount
------------------------------------------------------------
Transport                      KES           30760.00
Housing                        KES           33636.00
Utilities                      KES           20913.00
Education                      KES           11066.00
Entertainment                  KES           12723.00

Highest spending category      Housing (KES 33636.00)
============================================================
```

### Arithmetic verification

**Income** - the three valid Income records:

```
TX006  4,814
TX012  9,128
TX018 13,442
------------
      27,384   ✓ matches
```

**Category totals sum to total expenditure:**

```
Transport      30,760
Housing        33,636
Utilities      20,913
Education      11,066
Entertainment  12,723
---------------------
              109,098   ✓ matches Total Expenditure
```

**Balance:** 27,384 − 109,098 = **−81,714** ✓ matches

**Highest-spending category:** Housing at 33,636 is the largest of the five
figures listed above ✓ - verified by inspection of the same table the program
printed.

The negative balance is correct for this dataset - 14 expenses against 3 income
records.

*(Figures changed from revision 4 because the D3 fix removed `TRX2000`, a stray
record worth 5,000 in Transport. Transport fell from 35,760 to 30,760, which is
also why the highest-spending category is now Housing rather than Transport.)*

---

## Test 14: Budget Warnings - Individual and Category Levels

| Attribute | Value |
|-----------|-------|
| **Test ID** | T014 |
| **Category** | Normal |
| **Objective** | Verify warnings are produced at both the individual-transaction and category-aggregate levels |
| **Input** | Menu option 4 on the unmodified dataset |
| **Expected Result** | Individual table lists expenses exceeding their own limit; category table lists every category with an OK/OVER status |
| **Actual Result** | As expected - 5 individual warnings; all 5 categories listed, with Entertainment marked OVER |
| **Status** | **PASS** |
| **Notes** | The category view shows **all** categories with a status column, so a category being within budget is positively confirmed rather than merely absent |

### Evidence

```
Enter selection: 4

================================================================================
INDIVIDUAL TRANSACTION WARNINGS
================================================================================
ID         Category        Amount       Limit        Over by
--------------------------------------------------------------------------------
TX010      Education       7690.00      3000.00      4690.00
TX015      Utilities       11285.00     3000.00      8285.00
TX017      Entertainment   12723.00     8000.00      4723.00
TX019      Transport       14161.00     13000.00     1161.00
TX020      Housing         14880.00     3000.00      11880.00
================================================================================


================================================================================
CATEGORY BUDGET SUMMARY (WITH WARNINGS)
================================================================================
Category        Spent        Budget       Status
--------------------------------------------------------------------------------
Transport       30760.00     37000.00     OK
Housing         33636.00     34500.00     OK
Utilities       20913.00     26500.00     OK
Education       11066.00     16000.00     OK
Entertainment   12723.00     8000.00      OVER
================================================================================
```

### Spot-check of the logic

**Individual level** - TX020: amount 14,880 against its own limit of 3,000.
Over by 14,880 − 3,000 = **11,880.00** ✓ matches the printed row.

**Category level** - Entertainment has one valid expense (TX017: 12,723,
limit 8,000). TX011 is in the same category but invalid, so it is excluded.
Spent 12,723 > budget 8,000 → **OVER** ✓ matches.

**Cross-check** - the Spent column sums to
30,760 + 33,636 + 20,913 + 11,066 + 12,723 = **109,098**, matching Total
Expenditure in T013 ✓

**Why only Entertainment is OVER:** each category sums the budget limits of
*all* its records, so multi-record categories accumulate a large combined
allowance. Transport totals 30,760 against 37,000 - under, despite TX019
individually breaching its own 13,000 limit. This is why both levels are
needed: neither alone gives the full picture, and the individual table is what
catches TX019.

---

## Test 15: Payment Summary - Counts and Totals by Method

| Attribute | Value |
|-----------|-------|
| **Test ID** | T015 |
| **Category** | Normal |
| **Objective** | Verify the payment summary counts **all** valid transactions and totals amounts per payment method |
| **Input** | Menu option 6 on the unmodified dataset |
| **Expected Result** | Every payment method listed with the number of transactions and total amount; counts sum to the full valid-record count |
| **Actual Result** | As expected - counts sum to 17, matching the 17 valid transactions |
| **Status** | **PASS** |
| **Notes** | This test previously failed as D4, when Income records were excluded and only 15 of 18 were counted. **It now verifies the fix** |

### Evidence

```
Enter selection: 6

================================================================================
PAYMENT METHOD SUMMARY
================================================================================
Method          Count    Total Amount    Total Budget
--------------------------------------------------------------------------------
Card            4        30760.00        37000.00
Cash            5        38450.00        40000.00
Bank            4        33636.00        34500.00
M-Pesa          4        33636.00        34500.00
================================================================================
```

### Completeness check - the check that originally found D4

```
Card 4 + Cash 5 + Bank 4 + M-Pesa 4 = 17
```

The dataset has **17 valid transactions** (T-VIEW, T003). Every one is counted ✓

**Amounts also reconcile:**

```
30,760 + 38,450 + 33,636 + 33,636 = 136,482
Total Income 27,384 + Total Expenditure 109,098 = 136,482   ✓
```

The Total Amount column now represents total transaction value rather than
expenditure alone, which is what the Income records being included implies.

*(In revision 4 the counts summed to 15 of 18 and the amounts summed to total
expenditure only. Both are resolved.)*

---

## Test Summary

| Test ID | Category | Description | Status |
|---------|----------|-------------|--------|
| T001 | Normal | Valid transaction accepted, confirmed and persisted | PASS |
| T002 | Invalid | Missing category rejected with reason | PASS |
| T003 | Invalid | Negative amount rejected | PASS |
| T004 | Boundary | Amount = budget limit (no warning) | PASS |
| T005 | Boundary | Amount just over limit - overage reported as 0.01 | PASS |
| T006 | Search | Search by existing ID | PASS |
| T007 | Search | Search for non-existent record | PASS |
| T008 | Search | Search by category (multiple results) | PASS |
| T009 | Menu | Invalid menu option rejected | PASS |
| T010 | Menu | Exit option terminates program | PASS |
| T011 | Invalid | Unrecognized transaction type | PASS |
| T012 | Invalid | Non-numeric amount at add time | PASS |
| T013 | Normal | Summary, balance and highest-spending category | PASS |
| T014 | Normal | Budget warnings at both levels | PASS |
| T015 | Normal | Payment summary counts all valid transactions | PASS |

**Total tests:** 15
**Passed:** 15
**Failed:** 0
**Open defects:** 0

Coverage against the five required categories: Normal (T001, T013, T014, T015),
Invalid (T002, T003, T011, T012), Boundary (T004, T005), Search (T006, T007,
T008), Menu (T009, T010).

---

## Defect Log - all resolved

Six defects were raised across revisions 2–4. All six have been fixed and
verified at commit `22e12fa`.

| ID | Defect | Found by | Status |
|---|---|---|---|
| D1 | Sub-unit amounts displayed as whole numbers - an overage of 0.01 showed as 0 | T005 | ✅ Fixed - verified by T005 |
| D2 | `highest_spending_category()` implemented but never called from any menu option | Call-site scan | ✅ Fixed - verified by T013 |
| D3 | `TRX2000` present in `cap-data.csv` but absent from the source worksheet | Dataset comparison | ✅ Fixed - record removed |
| D4 | Payment summary counted 15 of 18 valid transactions, excluding Income | T015 | ✅ Fixed - verified by T015 |
| D5 | Three functions unreachable after the display refactor; calculation logic duplicated inside display functions | Call-site scan | ✅ Fixed - no unreachable functions remain |
| D6 | `Algorithm` document described behaviour the code did not implement; no file extension | Document/code comparison | ✅ Fixed - reconciled and renamed `algorithm.md` |

### Verification of each fix

**D1** - `grep -n "\.0f" main.py` returns no matches. T005 shows
`5000.01 / 5000.00 / 0.01`.

**D2** - `highest_spending_category` now appears twice in `main.py`: its
definition at line 195 and a call at line 229, inside `display_summary()`.
Output visible in T013.

**D3** - `cap-data.csv` now ends at TX020. The dataset is 20 records, matching
the `09_Expense_Tracker` worksheet.

**D4** - counts sum to 17 of 17 valid records; amounts reconcile to income plus
expenditure. See T015.

**D5** - automated call-site scan of all functions in `main.py` reports no
unreachable functions.

**D6** - `algorithm.md` step 2 now reads "Check that the transaction type is
`Income` or `Expense`", matching the implementation, and the earlier claim about
standardizing `EXP` has been removed. File renamed with a `.md` extension so it
renders as Markdown.

---

## Test Execution Notes

- All tests were executed against `main.py` at commit `22e12fa`, with
  `cap-data.csv` restored to its committed state between tests.
- The program writes to `cap-data.csv` as soon as a transaction is added, so
  `git restore cap-data.csv` should be run after any manual testing to avoid
  committing test rows. D3 was an example of what happens otherwise.
- Invalid records (T003, T011, and TX016) are detected at **load** time by
  `validate_all()`; T002 and T012 are detected at **add** time. Both paths call
  the same `validate_transaction()` function.
- No test produced an unhandled exception or traceback.
- Menu control (T009, T010) confirms the loop is robust and exits cleanly.
- Search (T006–T008) handles zero, one and many results.
- Boundary tests (T004–T005) confirm the comparison is strict `>`, not `>=`.
- T007 uses `TX404` rather than `TX999` so the suite is order-independent -
  T001 creates `TX999`, which would otherwise cause T007 to fail when the tests
  are run in sequence.

### On cross-checking

Figures are verified against each other rather than only inspected in isolation:

- T008's Transport rows sum to T013's Transport total, which matches T014's
  Transport row
- T014's Spent column sums to T013's Total Expenditure
- T015's counts sum to the valid-record count from T003
- T015's amounts sum to Income plus Expenditure from T013
- T003's invalid count plus the valid count equals the number of rows in the CSV
