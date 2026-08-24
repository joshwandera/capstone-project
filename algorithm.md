# FCC CAPSTONE PROJECT - ALGORITHM SUMMARY

## 1. Store the transaction records

- Read the records from `cap-data.csv` using Python's built-in `csv` module.
- Keep the records in a list of dictionaries.
- Convert the amount and budget-limit fields into numbers.

## 2. Define a function to validate transactions

- Check that the transaction type is `Income` or `Expense`.
- Check that the amount is a number and is greater than zero.
- Check that `Expense` transactions have a category.
- Check that the budget limit is not negative.

## 3. Define a function to view transactions

- Show valid or invalid transactions based on what the user selects.

## 4. Define a function to add a transaction

- Ask the user for the transaction details.
- Check the amount and budget limit.
- Add the new transaction to the list if it is valid.

## 5. Define a function to search for a transaction

- Ask the user for a transaction ID or category.
- Search through the transaction list.
- Show the matching transaction, or say that nothing was found.

## 6. Define a function to calculate the income and expenditure summary

- Calculate total income.
- Calculate total expenditure.
- Calculate the balance.
- Calculate category totals.
- Identify the highest-spending category.

## 7. Define a function to identify budget warnings

- Compare each transaction's amount with its own budget limit and flag it if it goes over.
- Calculate spending by category.
- Compare each category's spending with its budget.
- Show a warning when spending is above the budget.

## 8. Define a function to display invalid transactions

- Check each transaction using the validation function.
- Show the transactions that fail validation.

## 9. Define a function to display the payment summary

- Count transactions by payment method.
- Show the totals.

## 10. Define a function to exit the program

## 11. Ask the user to select an operation

- If the selection is invalid, show an error message and return to the menu.

## 12. Call the function for the selected operation

- Each non-exit menu option calls the matching function.

## 13. Repeat the menu and operation selection until the user selects Exit

## 14. Display a closing message

END
