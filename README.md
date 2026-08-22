# Capstone Project: Personal Expense and Budget Tracker

A menu-driven Python program that helps an individual monitor income, expenses, budgets and payment methods. It loads a dataset of transactions, validates every record against a set of rules, rejects and explains bad records, and produces income/expenditure summaries, budget warnings and payment-method summaries.

The core program (`main.py`) is written in **core Python only**, with no Pandas, NumPy, charts or third-party libraries.

> **Group Members:**
>
> 1. Ansel Melly
> 2. Ivy Managene
> 3. Josiah Wandera
> 4. Lewis Munyi
> 5. Ruth Kwamboka
>

---

## Group members

-

---

## Table of contents

- [Problem definition](#problem-definition)
- [Objectives](#objectives)
- [Repository structure](#repository-structure)
- [Requirements](#requirements)
- [How to run](#how-to-run)
  - [Command-line version](#command-line-version-mainpy)
  - [GUI version (extra)](#gui-version-extra--gui_mainpy)
- [The menu](#the-menu)
- [Validation rules](#validation-rules)
- [Data standardization](#data-standardization)
- [Functions overview](#functions-overview)
- [Dataset](#dataset)
- [Extras we did](#extras-we-did)
- [Testing](#testing)
- [Git workflow](#git-workflow)
- [Group members](#group-members)
- [License](#license)

---

## Problem definition

A person needs a simple, reliable way to track personal finances. The supplied dataset contains deliberate anomalies, such as missing values, negative values, and unrecognized categories, that the program must detect and handle safely rather than crash on or silently accept.

The program must:

- Store the transaction records using an appropriate core-Python data structure.
- Validate every record and report rejected records with a reason.
- Calculate total income, total expenditure and the resulting balance.
- Calculate expense totals by category and identify the highest-spending category.
- Compare category expenditure against budget limits and warn when exceeded.
- Count transactions by payment method.
- Provide a repeating menu until the user chooses to exit.

## Objectives

- Translate a realistic data problem into a structured, modular, tested program.
- Practise algorithm design, validation and core Python (loops, decisions, functions, data structures) rather than advanced libraries.
- Cleanly separate calculation logic from display logic so the same functions can be reused (by both the CLI and the GUI).

---

## Repository structure

```
capstone-project/
├── README.md
├── main.py            # Command-line program (core Python only)
├── gui_main.py        # Tkinter GUI version (extra: reuses main.py logic)
├── cap-data.csv       # Transaction dataset (09_Expense_Tracker)
├── algorithm.md       # High-level algorithm summary
├── pseudocode.md      # Detailed, function-by-function pseudocode
├── tests.md           # 15 documented test cases with captured evidence
├── Budget tracker.ipynb  # Exploratory notebook (extra)
├── LICENSE
└── .gitignore
```

---

## Requirements

- **Python 3** (developed and tested on Python 3.14; any recent Python 3 works).
- **No third-party packages.** `main.py` uses only the standard-library `csv` module.
- **`cap-data.csv`** must be present in the same directory as the program, because the program reads (and, when a transaction is added, writes) that file by relative path.
- The GUI (`gui_main.py`) uses only `tkinter`, which ships with the standard CPython distribution. You can install it by running:

**macOS**

```bash
brew install python-tk
```

**Ubuntu linux**

```bash
sudo apt update && sudo apt install python3-tk
```

**Windows**
> Tkinter comes pre-installed in windows binaries unless it was unselected during installation. Check [this guide](https://www.pythonguis.com/installation/install-tkinter-windows/) for more info

To check your Python version:

```bash
python3 --version
```

---

## How to run

Clone the repository and change into it:

```bash
git clone http://github.com/lewis-munyi/capstone-project.git
cd capstone-project
```

### Command-line version (`main.py`)

This is the primary deliverable.

```bash
python3 main.py
```

You will be shown the menu; type a number (1–7) and press Enter. The menu repeats until you choose **7. Exit**.

> **Note on persistence:** adding a transaction (menu option 2 > 1) appends it to `cap-data.csv` immediately. If you add test records while trying the program, restore the dataset afterwards with `git restore cap-data.csv` so you don't accidentally commit test rows.

### GUI version (extra) `gui_main.py`

A **Tkinter** interface built on top of the same logic. It imports the validation and calculation functions directly from `main.py`, so both versions always stay in sync and are covered by the same tests.

```bash
python3 gui_main.py
```

The seven menu operations map to six tabs plus an Exit button:

| CLI menu option | GUI tab / control |
| --- | --- |
| 1. View valid transactions | **Valid Transactions** tab |
| 2. Add or search a transaction | **Add / Search** tab |
| 3. Income & expenditure summary | **Summary** tab |
| 4. View budget warnings | **Budget Warnings** tab |
| 5. View invalid records | **Invalid Records** tab |
| 6. View payment summary | **Payment Summary** tab |
| 7. Exit | **Exit** button (or close the window) |

Run `gui_main.py` from the same folder as `main.py` and `cap-data.csv`.

---

## The menu

```
=== Personal Expense and Budget Tracker ===
1. View valid transactions
2. Add or search a transaction
3. View income and expenditure summary
4. View budget warnings
5. View invalid records
6. View payment summary
7. Exit
```

The menu is implemented with a `while` loop, `input()`, an `if`/`elif`/`else` chain and `break`, as required. Each non-exit option calls a separate function, and an invalid selection displays an error and returns to the menu.

---

## Validation rules

A record is accepted only when all of the following hold true, otherwise it is rejected and reported with a reason:

- **Transaction type** is `Income` or `Expense`.
- **Amount** is numeric and greater than zero.
- **Category** is provided for `Expense` transactions.
- **Budget limit** is not negative.

Invalid records are excluded from all calculations and are viewable via menu option 5.

## Data standardization

Only recognized alternatives with a **defined mapping** are standardized; anything else is left unchanged so it is correctly rejected rather than guessed at. The transaction-type mapping normalizes common aliases (e.g. `income`, `inc`, `in` for `Income`; `expense`, `exp`, `ex` for `Expense`). A value such as `EXP` that is not in the map is passed through unchanged and flagged as an **unrecognized** transaction type.

---

## Functions overview

The code deliberately separates **calculation** functions (which return data and never print) from **display** functions (which format and print). This is what lets the CLI and GUI share identical logic.

| Function | Role |
| --- | --- |
| `load_transactions()` | Read `cap-data.csv` into a list of dictionaries and cast numeric fields |
| `validate_transaction(record)` | Apply the four validation rules to one record; return list of reasons |
| `validate_all(transactions)` | Split the dataset into valid and invalid records |
| `add_transaction(transactions)` | Prompt for, validate, append and persist a new record |
| `write_transactions(transactions)` | Write all records back to the CSV |
| `search_transaction(valid_records)` | Find records by ID or category (exact, case-insensitive) |
| `calculate_summary(valid_records)` | Compute total income, expenditure, balance and per-category totals |
| `highest_spending_category(valid_records)` | Identify the category with the highest expenditure |
| `check_individual_budget_warnings(valid_records)` | Flag expenses exceeding their own budget limit |
| `check_category_budget_summary(valid_records)` | Aggregate spend vs. budget per category with OK/OVER status |
| `calculate_payment_method_totals(valid_records)` | Count and total transactions per payment method |
| `display_*` functions | Format and print the results of the calculation functions |
| `main()` | The menu loop |

Full step-by-step pseudocode for each function is in [pseudocode.md](pseudocode.md), and a higher-level summary is in [algorithm.md](algorithm.md).

---

## Dataset

`cap-data.csv` contains **20 records** with
the following columns:

```
transaction_id, transaction_type, category, description,
amount_kes, budget_limit_kes, payment_method
```

Of the 20 records, **17 are valid and 3 are invalid** by design:

- `TX005` - negative amount.
- `TX011` - unrecognized transaction type (`EXP`).
- `TX016` - missing category on an expense.

These anomalies exercise the validation and error-reporting paths.

---

## Extras we did

Beyond the required command-line deliverable, we went further for practice:

- **Tkinter GUI (`gui_main.py`):** a full graphical version that reuses the exact same validation and calculation functions from `main.py`, so nothing is reimplemented and both stay consistent.
- **Add & persist:** new transactions can be added interactively and are written back to `cap-data.csv`.
- **Dual budget warnings:** warnings at both the individual-transaction level and the category-aggregate level.
- **Comprehensive test suite:** 15 documented test cases in [tests.md](tests.md) with captured terminal evidence and a resolved defect log.
- **Exploratory notebook:** `Budget tracker.ipynb` for experimentation.

---

## Testing

[tests.md](tests.md) documents **15 test cases** covering all five required
categories:

- **Normal** - valid records process, summarise and persist correctly.
- **Invalid** - missing category, negative amount, non-numeric amount and
  unrecognized type are each rejected with a reason.
- **Boundary** - amount exactly equal to vs. just over a budget limit.
- **Search** - existing item, non-existing item and multiple results.
- **Menu** - invalid option repeats the menu; Exit terminates cleanly.

All 15 tests pass and the defect log is closed. Each test includes a captured
terminal transcript and, where relevant, an arithmetic cross-check.

To reproduce, run `python3 main.py` and enter the inputs listed for each test.

---

## Git workflow

- Repository initialized at the start of development.
- Multiple meaningful commits with descriptive messages showing progressive
  development.
- Source, README, tests and algorithm documents are all tracked.
- The final version is tagged `v1.0`.
- No passwords, keys or unnecessary system files are committed.

Inspect the history:

```bash
git log --oneline
```

Public repository: <http://github.com/lewis-munyi/capstone-project.git>

---

## License

See [LICENSE](LICENSE).
