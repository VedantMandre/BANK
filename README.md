# Send Payments Flow and Database Schema

This document provides a comprehensive overview of the "Send Payments" flow based on the Figma screens and the corresponding database schema. It includes details on the payment type selection, required fields in the payment section, beneficiary information, and all related database tables along with key considerations and potential future extensions.

---

## 1. Payment Type Selection

There are three distinct payment types shown:
- **Wire**
- **ACH**
- **Check**

---

## 2. Required Fields in Payment Section

### 2.1 Basic Payment Info

- **Copy - Funding Account (dropdown)**
  - **Maps to:** `DEBIT_ACCOUNT_NUMBER`
  
- **Payment Currency (dropdown)**
  - **Maps to:** `CURRENCY` (currently showing USD)
  
- **Payment Amount (input field)**
  - **Maps to:** `AMOUNT`
  - **Additional Info:** Displays current balance below the field
  
- **Value Date (date picker)**
  - **Maps to:** `VALUE_DATE`
  - **Format:** MM/DD/YYYY
  
- **Payment Frequency**
  - **Options:** Single / Recurring
  - **Maps to:** *New field needed* (not present in the current schema)
  
- **Payment Notes (text area)**
  - **Maps to:** `DETAILS_OF_PAYMENT_LINE1/2/3/4`

---

## 3. Beneficiary Information

### 3.1 Beneficiary Type Toggle

- **Options:**
  - Existing Beneficiary (with search)
  - New Beneficiary

### 3.2 Required Fields for New Beneficiary

- **Beneficiary Name (search/input)**
  - **Maps to:** `BENEF_CUST_NAME`
  
- **Bank Name**
  - **Maps to:** `BENEF_INSTIT_NAME`
  
- **Bank Account Number**
  - **Maps to:** `REC_CORR_ACCOUNT_NUMBER1`
  
- **BAN (Bank Account Number)**
  - **Maps to:** `IBAN`
  
- **Address Fields:**
  - **Street:** Maps to `BENEF_CUST_ADDRESS_LINE1`
  - **City:** Part of `BENEF_CUST_ADDRESS_LINE2`
  - **State:** Part of `BENEF_CUST_ADDRESS_LINE2`
  - **ZIP:** Part of `BENEF_CUST_ADDRESS_LINE3`

---

## 4. Core Payment Table

```sql
CREATE TABLE payments (
    payment_id BIGINT PRIMARY KEY,
    payment_type VARCHAR(10),  -- WIRE, ACH, CHECK
    funding_account_id VARCHAR(50),
    currency_code VARCHAR(3),
    amount DECIMAL(20,2),
    value_date DATE,
    payment_frequency VARCHAR(20),  -- SINGLE, RECURRING
    payment_notes TEXT,
    payment_status VARCHAR(20),  -- DRAFT, PENDING, COMPLETED, FAILED
    created_at TIMESTAMP,
    created_by VARCHAR(50),
    updated_at TIMESTAMP,
    updated_by VARCHAR(50)
);

6. Beneficiary Management
6.1 Beneficiaries Table
CREATE TABLE beneficiaries (
    beneficiary_id BIGINT PRIMARY KEY,
    beneficiary_name VARCHAR(100),
    beneficiary_type VARCHAR(20),  -- INDIVIDUAL, BUSINESS
    bank_name VARCHAR(100),
    bank_account_number VARCHAR(50),
    iban VARCHAR(34),
    is_active BOOLEAN,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);
6.2 Beneficiary Addresses Table
CREATE TABLE beneficiary_addresses (
    address_id BIGINT PRIMARY KEY,
    beneficiary_id BIGINT,
    street_address VARCHAR(200),
    city VARCHAR(100),
    state VARCHAR(50),
    zip_code VARCHAR(20),
    country VARCHAR(50),
    address_type VARCHAR(20),  -- PRIMARY, BILLING, etc.
    FOREIGN KEY (beneficiary_id) REFERENCES beneficiaries(beneficiary_id)
);
7. Payment-Beneficiary Relationship
CREATE TABLE payment_beneficiaries (
    payment_id BIGINT,
    beneficiary_id BIGINT,
    PRIMARY KEY (payment_id, beneficiary_id),
    FOREIGN KEY (payment_id) REFERENCES payments(payment_id),
    FOREIGN KEY (beneficiary_id) REFERENCES beneficiaries(beneficiary_id)
);
8. Payment Status History
CREATE TABLE payment_status_history (
    history_id BIGINT PRIMARY KEY,
    payment_id BIGINT,
    status VARCHAR(20),
    status_timestamp TIMESTAMP,
    status_reason TEXT,
    updated_by VARCHAR(50),
    FOREIGN KEY (payment_id) REFERENCES payments(payment_id)
);
9. Account Information
CREATE TABLE accounts (
    account_id VARCHAR(50) PRIMARY KEY,
    account_number VARCHAR(50),
    account_type VARCHAR(20),
    currency_code VARCHAR(3),
    current_balance DECIMAL(20,2),
    available_balance DECIMAL(20,2),
    is_active BOOLEAN,
    last_updated TIMESTAMP
);
10. Key Considerations
10.1 Data Types
VARCHAR: Use appropriate sizes for string fields.
DECIMAL: Use for monetary amounts.
TIMESTAMP: Use for audit and time-based fields.
10.2 Indexing Strategy
-- Example indexes
CREATE INDEX idx_payments_status ON payments(payment_status);
CREATE INDEX idx_payments_date ON payments(value_date);
CREATE INDEX idx_beneficiary_name ON beneficiaries(beneficiary_name);
