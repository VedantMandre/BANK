# Payment System Schema and Column Analysis

This document outlines the necessary fields and their categorization for a payment system, based on the provided Figma landing page details and the underlying table schema (GCMS_FUNDS_TRANSFER_DETAIL). It includes both the frontend display fields and additional backend fields that, while not visible on the navigation page, are critical for processing and tracking payments.

---

## 1. Frontend Display Fields

These fields are expected to be shown on the navigation page based on the Figma design.

### 1.1 Payment Identification
- **FUNDS_TRANSFER_ID** (Primary Key)
- **TRANSACTION_ID**
- **TRANSACTION_REFERENCE_NUMBER**

### 1.2 Beneficiary Information
- **BENEF_CUST_NAME**  
  *Note: This may need to be parsed into first and last name.*
- **BENEF_CUST_ID**

### 1.3 Payment Type / Tags
- **TRANSACTION_CODE**
- **FUNDS_TYPE** (e.g., ACH, WIRE, etc.)
- **CATEGORY_PURPOSE**

### 1.4 Amount Details
- **AMOUNT**
- **CURRENCY**
- **USD_AMOUNT** (for standardization)

### 1.5 Status Information
- **TRANSACTION_STATUS_ID**
- **CREATION_DATE** (used as the submission date)

---

## 2. Critical Backend Fields (Not Displayed on Navigation Page)

These fields are necessary for transaction processing, auditing, and backend operations.

### 2.1 Transaction Processing
- **VALUE_DATE**
- **EXCHANGE_RATE**
- **BASE_RATE**
- **FX_EXCHANGE_RATE**
- **FX_ONLINE_INDICATOR**
- **TRACKING_STATUS_ID**
- **TRACKING_DETAIL_ID**

### 2.2 Extended Beneficiary Details
- **BENEF_CUST_ADDRESS_LINE1**
- **BENEF_CUST_ADDRESS_LINE2**
- **BENEF_CUST_ADDRESS_LINE3**
- **BENEF_CUST_COUNTRY_CODE**
- **BENEF_BANK_FUNDS_TYPE**
- **BENEF_INSTIT_ID**
- **BENEF_INSTIT_NAME**

### 2.3 Source / Routing Information
- **SOURCE_SYSTEM**
- **APPLICATION_ID**
- **IBAN**
- **BANK_SWIFT_ID**
- **UETR** (Unique End-to-end Transaction Reference)

### 2.4 Audit / Security Fields
- **CREATED_BY**
- **CREATION_DATE**
- **LAST_UPDATED_BY**
- **LAST_UPDATED_DATE**
- **USER_SESSION_ID**
- **VERSION**

### 2.5 Payment Details
- **DETAILS_OF_PAYMENT_LINE1**
- **DETAILS_OF_PAYMENT_LINE2**
- **DETAILS_OF_PAYMENT_LINE3**
- **DETAILS_OF_PAYMENT_LINE4**
- **DETAILS_OF_CHARGES**
- **PURPOSE_CODE**
- **BUSINESS_CATEGORY**

### 2.6 Account Information
- **DEBIT_ACCOUNT_NUMBER**
- **DEBIT_ACCOUNT_CURRENCY_CODE**
- **ACCOUNT_BALANCE**
- **ORIG_DEBIT_ACCOUNT_NUMBER**

---

## 3. Key Relationships and Structure

Understanding the relationships between the fields is crucial for proper data integration and processing.

### 3.1 Transaction Flow
- **FUNDS_TRANSFER_ID → TRANSACTION_ID**  
  *This establishes a unique relationship between funds transfer and the transaction identifier.*
- **TRANSACTION_STATUS_ID**  
  *Used for tracking the status of the payment.*
- **TRACKING_STATUS_ID → TRACKING_DETAIL_ID**  
  *Establishes the tracking mechanism for the payment process.*

### 3.2 Financial Flow
- **DEBIT_ACCOUNT_NUMBER → AMOUNT → EXCHANGE_RATE**  
  *Links account number to the payment amount and associated exchange rate.*
- **CURRENCY → USD_AMOUNT**  
  *Used for standardization of the payment amount.*

### 3.3 Audit Trail
- **CREATED_BY → CREATION_DATE**  
  *Captures who created the record and when.*
- **LAST_UPDATED_BY → LAST_UPDATED_DATE**  
  *Tracks who last updated the record and when.*
- **VERSION**  
  *Used for concurrency control and audit purposes.*

---

## 4. Additional Recommendations

### 4.1 Indexing
Consider creating indexes on the following fields to improve query performance:
- **TRANSACTION_STATUS_ID**
- **CREATION_DATE**
- **BENEF_CUST_ID**
- **FUNDS_TYPE**

### 4.2 Caching Strategies
Implement caching where necessary for:
- **Status Lookups**
- **Exchange Rates**
- **Customer Details**

---

## Summary

This document provides a structured overview of the necessary columns and their relationships for both the frontend display and backend processing of payment data. It is critical to collaborate with the microservices team (for account categorization and display requirements) and the payments team (for source system confirmation and data extraction details) to ensure that the data migration and integration are successful.
