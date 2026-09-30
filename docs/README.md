# COBOL Account Management

This directory documents the COBOL account-management example in `src/cobol`. The program provides a menu for viewing an account balance, crediting the account, debiting the account, and exiting.

## Source Files

### `main.cob`

The `MainProgram` is the command-line entry point. It displays the menu and loops until the user selects exit. It dispatches the balance, credit, and debit choices to `Operations`, and reports invalid menu selections.

### `operations.cob`

The `Operations` program handles account actions based on the operation passed by `MainProgram`:

- `TOTAL`: reads the stored balance and displays it.
- `CREDIT`: accepts an amount, reads the balance, adds the amount, writes the updated balance, and displays it.
- `DEBIT`: accepts an amount, reads the balance, and subtracts and saves the amount only when the balance is at least that amount. Otherwise it displays an insufficient-funds message.

### `data.cob`

The `DataProgram` owns the balance storage. It supports `READ`, which copies the stored balance to the caller, and `WRITE`, which replaces the stored balance with the caller-provided value. The balance is initialized to `1000.00` when the program's storage is initialized.

## Account Rules and Scope

- The starting account balance is `1000.00`.
- Credits add the entered amount to the balance.
- Debits are allowed only when the current balance is greater than or equal to the requested amount; this prevents a debit that exceeds the available balance.
- A rejected debit does not write a changed balance.
- The code does not model student identity, enrollment, tuition, fees, or student-specific eligibility. It manages a single generic account balance; no additional student-account business rules are implemented.
- The operation logic contains no explicit validation for positive amounts or other amount limits.

## Data Flow

```mermaid
sequenceDiagram
	actor User
	participant Main as MainProgram
	participant Ops as Operations
	participant Data as DataProgram

	loop Until the user exits
		Main->>User: Display menu
		User->>Main: Select menu option
		alt View balance
			Main->>Ops: CALL TOTAL
			Ops->>Data: CALL READ
			Data-->>Ops: Return stored balance
			Ops-->>User: Display current balance
		else Credit account
			Main->>Ops: CALL CREDIT
			Ops->>User: Prompt for credit amount
			User-->>Ops: Enter amount
			Ops->>Data: CALL READ
			Data-->>Ops: Return stored balance
			Ops->>Ops: Add amount to balance
			Ops->>Data: CALL WRITE with updated balance
			Data-->>Ops: Save updated balance
			Ops-->>User: Display new balance
		else Debit account
			Main->>Ops: CALL DEBIT
			Ops->>User: Prompt for debit amount
			User-->>Ops: Enter amount
			Ops->>Data: CALL READ
			Data-->>Ops: Return stored balance
			alt Balance is at least the debit amount
				Ops->>Ops: Subtract amount from balance
				Ops->>Data: CALL WRITE with updated balance
				Data-->>Ops: Save updated balance
				Ops-->>User: Display new balance
			else Insufficient funds
				Ops-->>User: Display insufficient-funds message
			end
		else Invalid menu option
			Main-->>User: Display invalid-choice message
		else Exit
			Main->>Main: End menu loop
		end
	end
	Main-->>User: Display goodbye message
```
