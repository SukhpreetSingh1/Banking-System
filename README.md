# Banking System — Backend Flow

## 1. Project Overview

This project is a **file-based Banking System** in which customer/account data is stored in a **binary file**.

The backend is responsible for:

- Creating a new bank account
- Storing account details
- Depositing money
- Withdrawing money
- Searching for an account
- Modifying account details
- Deleting an account
- Reading and writing customer data to a binary file

---

## 2. Backend Architecture

```text
                    ┌──────────────────────┐
                    │     Banking System   │
                    │      Backend         │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Account Class     │
                    ├──────────────────────┤
                    │ accountNumber        │
                    │ name                 │
                    │ deposit/balance      │
                    │ withdrawAmount       │
                    │ accountType          │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │  File Operations     │
                    ├──────────────────────┤
                    │ Read                 │
                    │ Write                │
                    │ Update               │
                    │ Delete               │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Binary File        │
                    │   accounts.dat       │
                    └──────────────────────┘
```

---

# 3. Account Data Model

Each customer account is represented by an `Account` object.

```text
Account
│
├── accountNumber
├── name
├── balance / deposit
├── withdrawAmount
└── accountType
```

The account object is converted into binary data when it is written to the file.

---

# 4. Main Backend Flow

```text
START
  │
  ▼
Initialize Banking System
  │
  ▼
Open / Create Binary File
  │
  ▼
Choose Backend Operation
  │
  ├───────────────┬────────────────┬────────────────┬─────────────────┐
  ▼               ▼                ▼                ▼                 ▼
CREATE          DEPOSIT          WITHDRAW         MODIFY            DELETE
  │               │                │                │                 │
  ▼               ▼                ▼                ▼                 ▼
Create          Find Account     Find Account     Find Account      Find Account
Account         │                │                │                 │
  │             ▼                ▼                ▼                 ▼
  ▼           Validate        Validate         Update Fields      Remove Record
Validate      Amount          Amount              │                 │
Data            │                │                ▼                 ▼
  │             ▼                ▼             Rewrite File      Rewrite File
  ▼           Update Balance Update Balance       │                 │
Write Record     │                │                └────────┬────────┘
  │             ▼                ▼                         │
  └─────────────┴────────────────┴─────────────────────────┘
                               │
                               ▼
                         Operation Complete
                               │
                               ▼
                              END
```

---

# 5. Create Account Flow

```text
START
  │
  ▼
Receive Account Details
  │
  ├── Account Number
  ├── Name
  ├── Initial Deposit
  └── Account Type
  │
  ▼
Validate Input
  │
  ├── Invalid ──► Reject Account
  │
  └── Valid
        │
        ▼
Check Account Number
        │
        ├── Already Exists ──► Reject Creation
        │
        └── Available
              │
              ▼
        Create Account Object
              │
              ▼
        Open Binary File
              │
              ▼
        Write Account Record
              │
              ▼
        Close File
              │
              ▼
        Account Created
```

---

# 6. Deposit Money Flow

```text
START
  │
  ▼
Receive Account Number
  │
  ▼
Search Binary File
  │
  ├── Account Not Found
  │       │
  │       ▼
  │   Operation Failed
  │
  └── Account Found
          │
          ▼
     Read Account Record
          │
          ▼
     Receive Deposit Amount
          │
          ▼
     Validate Amount
          │
          ├── Invalid ──► Reject Transaction
          │
          └── Valid
                │
                ▼
       balance = balance + amount
                │
                ▼
          Update Record
                │
                ▼
          Rewrite Binary File
                │
                ▼
        Deposit Successful
```

---

# 7. Withdraw Money Flow

```text
START
  │
  ▼
Receive Account Number
  │
  ▼
Search Binary File
  │
  ├── Account Not Found
  │       │
  │       ▼
  │   Operation Failed
  │
  └── Account Found
          │
          ▼
     Read Account Record
          │
          ▼
     Receive Withdrawal Amount
          │
          ▼
     Validate Amount
          │
          ├── Amount <= 0 ──► Reject Transaction
          │
          ├── Amount > Balance ──► Insufficient Balance
          │
          └── Valid
                │
                ▼
       balance = balance - amount
                │
                ▼
          Update Record
                │
                ▼
          Rewrite Binary File
                │
                ▼
        Withdrawal Successful
```

---

# 8. Search Account Flow

Searching is used by deposit, withdrawal, modification, and deletion operations.

```text
START
  │
  ▼
Receive Account Number
  │
  ▼
Open accounts.dat
  │
  ▼
Read One Account Record
  │
  ▼
Compare Account Number
  │
  ├── Match ──────────► Return Account
  │
  └── No Match
          │
          ▼
     More Records?
       │
       ├── YES ──► Read Next Record
       │
       └── NO ───► Account Not Found
```

---

# 9. Modify Account Flow

```text
START
  │
  ▼
Receive Account Number
  │
  ▼
Open Original Binary File
  │
  ▼
Read Account Records
  │
  ▼
Find Matching Account
  │
  ├── Not Found ──► Operation Failed
  │
  └── Found
        │
        ▼
  Load Account Data
        │
        ▼
  Modify Allowed Fields
        │
        ▼
  Write Updated Record
        │
        ▼
  Save Updated Data
        │
        ▼
  Replace Original File
        │
        ▼
  Modification Complete
```

### Typical modification

```text
Old Account
     │
     ▼
Account Number ──► Usually used to identify the record
Name           ──► Can be modified
Account Type   ──► Can be modified
Balance        ──► Should normally be changed through
                  deposit/withdraw operations
     │
     ▼
Updated Account
```

---

# 10. Delete Account Flow

Because binary files do not normally support removing a record from the middle easily, deletion can be implemented using a **temporary binary file**.

```text
START
  │
  ▼
Receive Account Number
  │
  ▼
Open accounts.dat
  │
  ▼
Create temporary.dat
  │
  ▼
Read Account Record
  │
  ▼
Is Account Number Matching?
  │
  ├── YES ──► Skip Record
  │
  └── NO ───► Write Record to temporary.dat
                    │
                    ▼
              Read Next Record
                    │
                    ▼
               End of File?
               │          │
              NO         YES
               │          │
               └──────────┘
                          ▼
                  Close Both Files
                          │
                          ▼
                  Delete accounts.dat
                          │
                          ▼
                  Rename temporary.dat
                  to accounts.dat
                          │
                          ▼
                    Delete Complete
```

---

# 11. Binary File Structure

The binary file contains account records one after another.

```text
accounts.dat
│
├── Account Record 1
│     ├── Account Number
│     ├── Name
│     ├── Balance
│     ├── Withdrawal/transaction data
│     └── Account Type
│
├── Account Record 2
│     ├── Account Number
│     ├── Name
│     ├── Balance
│     ├── Withdrawal/transaction data
│     └── Account Type
│
├── Account Record 3
│     └── ...
│
└── Account Record N
```

The backend should treat the binary file as the **persistent storage layer**.

---

# 12. File Operations

| Operation | File Action |
|---|---|
| Create Account | Append new record |
| Search Account | Read records sequentially |
| Deposit | Find → modify → rewrite |
| Withdraw | Find → modify → rewrite |
| Modify | Find → modify → rewrite |
| Delete | Copy all records except target to temporary file |
| View Account | Read matching record |

---

# 13. Backend Components

```text
┌──────────────────────────────────────────┐
│              Banking Backend             │
├──────────────────────────────────────────┤
│                                          │
│  Account Class                           │
│       │                                  │
│       ├── Account Data                   │
│       └── Account Operations             │
│                                          │
│  Banking/File Functions                  │
│       │                                  │
│       ├── createAccount()                │
│       ├── findAccount()                  │
│       ├── deposit()                      │
│       ├── withdraw()                     │
│       ├── modifyAccount()                │
│       ├── deleteAccount()                │
│       └── displayAccount()               │
│                                          │
│                    │                     │
│                    ▼                     │
│             Binary File I/O              │
│                    │                     │
│                    ▼                     │
│              accounts.dat                │
│                                          │
└──────────────────────────────────────────┘
```

---

# 14. Overall Data Flow

```text
                 ┌──────────────┐
                 │    Request   │
                 │   Operation │
                 └──────┬───────┘
                        │
                        ▼
               ┌─────────────────┐
               │ Banking Logic   │
               └────────┬────────┘
                        │
                        ▼
               ┌─────────────────┐
               │ Find Account    │
               │ by Account No.  │
               └────────┬────────┘
                        │
                        ▼
               ┌─────────────────┐
               │ Read Binary     │
               │ Record          │
               └────────┬────────┘
                        │
                        ▼
               ┌─────────────────┐
               │ Validate         │
               │ Operation        │
               └────────┬────────┘
                        │
                        ▼
               ┌─────────────────┐
               │ Update Account  │
               │ Object          │
               └────────┬────────┘
                        │
                        ▼
               ┌─────────────────┐
               │ Write Updated   │
               │ Data            │
               └────────┬────────┘
                        │
                        ▼
               ┌─────────────────┐
               │ accounts.dat    │
               │ Persistent Data │
               └─────────────────┘
```

---

# 15. Important Backend Rules

1. **Account number should uniquely identify an account.**
2. Account creation should check whether the account number already exists.
3. Deposit amounts must be positive.
4. Withdrawal amounts must be positive.
5. Withdrawal must not allow the balance to become negative unless overdraft is explicitly implemented.
6. Every update must be persisted to the binary file.
7. Account deletion should preserve all other account records.
8. File streams should always be closed after the operation.
9. File errors should be handled instead of silently ignoring them.
10. Account balance should be maintained consistently; deposits and withdrawals should update it rather than independently storing conflicting values.

---

# 16. Complete Backend Lifecycle

```text
                 START
                   │
                   ▼
          Initialize Account
          / Banking Functions
                   │
                   ▼
            Open Binary File
                   │
                   ▼
          ┌─────────────────┐
          │ Select Operation│
          └────────┬────────┘
                   │
       ┌───────────┼────────────┐
       │           │            │
       ▼           ▼            ▼
    CREATE      TRANSACTION   ACCOUNT
       │        (Deposit/     MANAGEMENT
       │         Withdraw)    (Modify/Delete)
       │           │            │
       └───────────┼────────────┘
                   │
                   ▼
             Validate Data
                   │
                   ▼
             Read / Update
             Account Record
                   │
                   ▼
            Save to Binary File
                   │
                   ▼
             Close File
                   │
                   ▼
            Operation Done
                   │
                   ▼
                  END
```

---

## 17. Suggested Project Structure

```text
BankingSystem/
│
├── README.md
│
├── main.cpp
│
├── Account.h
├── Account.cpp
│
├── BankingSystem.h
├── BankingSystem.cpp
│
└── accounts.dat
```

### Responsibility of each file

```text
main.cpp
   │
   └── Starts the backend / calls banking operations

Account.h / Account.cpp
   │
   └── Account data and account-related functions

BankingSystem.h / BankingSystem.cpp
   │
   └── File handling and banking operations

accounts.dat
   │
   └── Persistent binary customer data
```

---

## Final Backend Concept

```text
             ACCOUNT OBJECT
                    │
                    ▼
          ┌──────────────────┐
          │ Banking Logic    │
          │                  │
          │ Create           │
          │ Search           │
          │ Deposit          │
          │ Withdraw         │
          │ Modify           │
          │ Delete           │
          └────────┬─────────┘
                   │
                   ▼
          ┌──────────────────┐
          │ Binary File I/O  │
          └────────┬─────────┘
                   │
                   ▼
             accounts.dat
                   │
                   ▼
          Persistent Accounts
```

**In short:** the backend follows a simple cycle:

`Request → Find Account → Validate → Modify Account Object → Write to Binary File → Save`
