# Capstone Project: Personal Expense and Budget Tracker

This is a menu-based Python program for tracking income, expenses, budgets and payment methods. It reads the transaction dataset, checks each record, keeps the valid ones, explains the invalid ones, and prints summaries for income, spending, budgets and payment methods.

The main program (`main.py`) uses **core Python only**. There is no Pandas, NumPy, charting library or other third-party package.

> **Group Members:**
>
> 1. Ansel Melly
> 2. Ivy Managene
> 3. Josiah Wandera
> 4. Lewis Munyi
> 5. Ruth Kwamboka
>

---

## Table of contents

- [Problem definition](#problem-definition)
- [Objectives](#objectives)
- [Repository structure](#repository-structure)
- [Requirements](#requirements)
- [How to run](#how-to-run)
  - [Command-line version](#command-line-version-mainpy)
  - [GUI version extra](#gui-version-extra)
- [The menu](#the-menu)
- [Validation rules](#validation-rules)
- [Transaction type handling](#transaction-type-handling)
- [Functions overview](#functions-overview)
- [Dataset](#dataset)
- [Extras we did](#extras-we-did)
- [Testing](#testing)
- [Git workflow](#git-workflow)
- [License](#license)

---

## Problem definition

A person needs a simple, reliable way to track personal finances. The supplied dataset contains deliberate anomalies, such as missing values, negative values, and unrecognized categories, that the program must detect and handle safely rather than crash on or silently accept.

The program needs to:

- Store the transaction records using an appropriate core-Python data structure.
- Validate every record and report rejected records with a reason.
- Calculate total income, total expenditure and the resulting balance.
- Calculate expense totals by category and identify the highest-spending category.
- Compare category expenditure against budget limits and warn when exceeded.
- Count transactions by payment method.
- Keep showing the menu until the user chooses to exit.

## Objectives

- Turn a realistic data problem into a clear, modular Python program.
- Practise algorithm design, validation and core Python basics like loops, decisions, functions and data structures.
- Keep calculation logic separate from display logic so the CLI and GUI can use the same functions.

---

## Repository structure

```text
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

- **Python 3**. This was developed and tested on Python 3.14, but any recent Python 3 version should work.
- **No third-party packages.** `main.py` only uses Python's built-in `csv` module.
- Keep **`cap-data.csv`** in the same folder as the program. The program reads from it, and when a transaction is added it writes back to it.
- The GUI (`gui_main.py`) uses `tkinter`, which normally comes with standard CPython. If it is missing, install it with:

### macOS

```bash
brew install python-tk
```

### Ubuntu linux

```bash
sudo apt update && sudo apt install python3-tk
```

### Windows

> Tkinter usually comes with the Windows Python installer unless it was left out during installation. This guide may help: <https://www.pythonguis.com/installation/install-tkinter-windows/>

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

Run the project by:

```bash
python3 main.py
```

The program shows a menu. Type a number from 1 to 7 and press Enter. It keeps looping until you choose **7. Exit**.

> **Note on saved data:** adding a transaction through menu option 2 > 1 writes it to `cap-data.csv` straight away. If you add test records, restore the file afterwards with `git restore cap-data.csv` so those rows are not committed by mistake.

### GUI version extra

A **Tkinter** interface is included as an extra. It imports the validation and calculation functions from `main.py`, so the CLI and GUI use the same logic.

```bash
python3 gui_main.py
```

The seven CLI menu options map to six GUI tabs plus an Exit button:

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

```text
=== Personal Expense and Budget Tracker ===
1. View valid transactions
2. Add or search a transaction
3. View income and expenditure summary
4. View budget warnings
5. View invalid records
6. View payment summary
7. Exit
```

The menu uses a `while` loop, `input()`, an `if`/`elif`/`else` chain and `break`. Each option calls its own function. If the user enters an invalid option, the program shows an error and returns to the menu.

---

## Validation rules

A record is accepted only if it passes these checks. If it fails, the program keeps it out of the valid records and shows the reason.

- **Transaction type** is `Income` or `Expense`.
- **Amount** is numeric and greater than zero.
- **Category** is provided for `Expense` transactions.
- **Budget limit** is not negative.

Invalid records are excluded from all calculations and are viewable via menu option 5.

## Transaction type handling

The program accepts `Income` and `Expense` as transaction types. Other values are left unchanged and rejected, so the program does not guess what a value like `EXP` was supposed to mean.

---

## Functions overview

The code keeps **calculation** functions separate from **display** functions. The calculation functions return data. The display functions print it. That is how the CLI and GUI can share the same logic.

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

```text
transaction_id, transaction_type, category, description,
amount_kes, budget_limit_kes, payment_method
```

Of the 20 records, **17 are valid and 3 are invalid**:

- `TX005` - negative amount.
- `TX011` - unrecognized transaction type (`EXP`).
- `TX016` - missing category on an expense.

These records are useful because they show that the validation and error messages work.

---

## Extras we did

Beyond the required command-line deliverable, we went further for practice:

- **Tkinter GUI (`gui_main.py`):** a graphical version that reuses the validation and calculation functions from `main.py`.
- **Add & persist:** new transactions can be added interactively and written back to `cap-data.csv`.
- **Dual budget warnings:** warnings are shown for individual transactions and for whole categories.
- **Test suite:** 15 documented test cases in [tests.md](tests.md), with terminal evidence and a closed defect log.
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

All 15 tests pass, and the defect log is closed. Each test includes a captured
terminal transcript and, where useful, an arithmetic check.

To reproduce, run `python3 main.py` and enter the inputs listed for each test.

---

## Git workflow

- The repository was initialized at the start of development.
- The commit history shows the project being built up in stages.
- Source files, README, tests and algorithm documents are tracked.
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
