# Lazim API Documentation

**Base URL**: `http://lazim-account.local`

This documentation provides details on working with the Lazim API, focusing on Tally Integration endpoints. It includes code examples in Bash and JavaScript. The API is not authenticated.

---

## Introduction

This documentation aims to provide all the information you need to work with our API.

As you scroll, you'll see code examples for working with the API in different programming languages in the dark area to the right (or as part of the content on mobile). You can switch the language used with the tabs at the top right (or from the nav menu at the top left on mobile).

---

## Authenticating Requests

This API is not authenticated.

---

## Tally Integration

### Retrieve Sales Vouchers

Fetches sales vouchers (invoices) within a specified date range for a given building. This endpoint integrates with Tally ERP by exporting invoice data in a structured format suitable for voucher import. Each voucher represents a sales transaction, including party details, income ledgers, and VAT breakdowns. Only unacknowledged vouchers (based on TallyAcknowledgement records) are included to avoid duplicates.

**Headers**  
- `Accept: application/json` (required - specifies JSON response format)  
- `Content-Type: application/json` (required for POST requests with body parameters)  
- `Authorization: Bearer {token}` (optional - if authentication is enabled for the API)

**Method**: GET  
**Path**: `/api/V1/getSalesVouchers`

#### Body Parameters
- `fromDate`: date (required) - The start date of the range in Y-m-d format. Filters invoices issued on or after this date. Example: `2025-01-01`  
- `toDate`: date (required) - The end date of the range in Y-m-d format. Filters invoices issued on or before this date. Example: `2025-09-17`  
- `building_id`: integer (required) - The unique identifier of the building to filter invoices for. Example: `1`

#### Example Request (Bash)
```bash
curl --request GET \
  --get "http://lazim-account.local/api/V1/getSalesVouchers" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --data "{
    \"fromDate\": \"2025-01-01\",
    \"toDate\": \"2025-09-17\",
    \"building_id\": 1
}"
```

#### Example Request (JavaScript)
```javascript
const url = new URL("http://lazim-account.local/api/V1/getSalesVouchers");
const headers = {
  "Content-Type": "application/json",
  "Accept": "application/json",
};
let body = {
  "fromDate": "2025-01-01",
  "toDate": "2025-09-17",
  "building_id": 1
};
fetch(url, { method: "GET", headers, body: JSON.stringify(body) }).then(response => response.json());
```

#### Example Responses
- **200 (Success)**:
  ```json
  {
    "result": "success",
    "total_vouchers": 3,
    "vouchers": [
      {
        "voucherDate": "Jan 11, 2025",
        "voucherNumber": "0225010008410981",
        "narration": "Reference: 0225010008410981",
        "voucherType": "Sales ERP Web",
        "voucherDetail": {
          "partyLedger": {
            "ledgerName": "DUBAI SILICON OASIS AUTHORITY",
            "transactionType": "Debit",
            "amount": 2266.5825,
            "reference_details": [
              {
                "reference_type": "New Ref",
                "reference_number": "0225010008410981",
                "reference_amount": 2266.5825
              }
            ]
          },
          "incomeLedger": [
            {
              "ledgerName": "Service Charges 2025 (Tax) - Gen & Res Fund",
              "transactionType": "Credit",
              "amount": "2158.65"
            },
            {
              "ledgerName": "VAT 5%",
              "transactionType": "Credit",
              "amount": 107.9325
            }
          ]
        }
      }
    ]
  }
  ```

- **422 (Validation Error)**:
  ```json
  {
    "result": "error",
    "errors": {
      "fromDate": ["The from date field is required."],
      "toDate": ["The to date must be a valid date."],
      "building_id": ["The building id field is required."]
    }
  }
  ```

- **500 (Server Error)**:
  ```json
  {
    "result": "error",
    "message": "An unexpected error occurred while fetching sales vouchers."
  }
  ```

#### Response Fields
- `result`: string - The status of the request. Values: "success" or "error".  
- `total_vouchers`: integer - The total number of sales vouchers returned in the response.  
- `vouchers`: array - An array of voucher objects, each representing a sales invoice.  
  - `voucherDate`: string - The formatted date of the voucher (e.g., "Jan 11, 2025").  
  - `voucherNumber`: string - The unique reference number of the invoice (e.g., "0225010008410981").  
  - `narration`: string - A descriptive narration for the voucher, including the reference number.  
  - `voucherType`: string - The type of voucher for Tally import (e.g., "Sales ERP Web").  
  - `voucherDetail`: object - Detailed ledger entries for the voucher.  
    - `partyLedger`: object - The debtor/party ledger entry.  
      - `ledgerName`: string - The name of the customer/party (e.g., "DUBAI SILICON OASIS AUTHORITY").  
      - `transactionType`: string - The transaction type for the party ledger (always "Debit" for sales).  
      - `amount`: number - The total amount debited to the party (e.g., 2266.5825).  
      - `reference_details`: array - Array of reference details for the transaction.  
        - `reference_type`: string - The type of reference (e.g., "New Ref").  
        - `reference_number`: string - The reference number (matches voucherNumber).  
        - `reference_amount`: number - The amount associated with this reference.  
    - `incomeLedger`: array - Array of credit ledger entries for income and taxes.  
      - `ledgerName`: string - The name of the income ledger (e.g., "Service Charges 2025 (Tax) - Gen & Res Fund" or "VAT 5%").  
      - `transactionType`: string - The transaction type (always "Credit" for income ledgers).  
      - `amount`: number|string - The credited amount (e.g., "2158.65" or 107.9325).  
- `errors`: object (validation error only) - Key-value pairs of field names and their validation error messages.

---

### Retrieve Account Groups

Fetches all account groups for a specified building, including their hierarchical structure and types. This endpoint integrates with Tally ERP by exporting account group data in a format suitable for Tally's ledger group configuration. The response includes top-level account subtypes and their children, each linked to a parent group or type, facilitating proper accounting hierarchy setup in Tally.

**Headers**  
- `Accept: application/json` (required - specifies JSON response format)  
- `Content-Type: application/json` (required for POST requests with body parameters)  
- `Authorization: Bearer {token}` (optional - if authentication is enabled for the API)

**Method**: GET  
**Path**: `/api/V1/getGroups`

#### Body Parameters
- `building_id`: integer (required) - The unique identifier of the building to filter account groups for. Example: `1`

#### Example Request (Bash)
```bash
curl --request GET \
  --get "http://lazim-account.local/api/V1/getGroups" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --data "{
    \"building_id\": 1
}"
```

#### Example Request (JavaScript)
```javascript
const url = new URL("http://lazim-account.local/api/V1/getGroups");
const headers = {
  "Content-Type": "application/json",
  "Accept": "application/json",
};
let body = {
  "building_id": 1
};
fetch(url, { method: "GET", headers, body: JSON.stringify(body) }).then(response => response.json());
```

#### Example Responses
- **200 (Success)**:
  ```json
  {
    "result": "success",
    "total": 11,
    "data": [
      {
        "name": "Current Asset",
        "parent": "Assets",
        "type": "Assets"
      },
      {
        "name": "Fixed Assets",
        "parent": "Assets",
        "type": "Assets"
      },
      {
        "name": "Investments",
        "parent": "Assets",
        "type": "Assets"
      },
      {
        "name": "Misc. Expenses (Asset)",
        "parent": "Assets",
        "type": "Assets"
      },
      {
        "name": "Branch / Divisions",
        "parent": "Liabilities",
        "type": "Liabilities"
      },
      {
        "name": "Capital Account",
        "parent": "Liabilities",
        "type": "Liabilities"
      },
      {
        "name": "Current Liabilities",
        "parent": "Liabilities",
        "type": "Liabilities"
      },
      {
        "name": "Direct Income",
        "parent": "Income",
        "type": "Income"
      },
      {
        "name": "OSales Account",
        "parent": "Income",
        "type": "Income"
      },
      {
        "name": "Indirect Expenses",
        "parent": "Expenses",
        "type": "Expenses"
      },
      {
        "name": "Purchase Accounts",
        "parent": "Expenses",
        "type": "Expenses"
      }
    ]
  }
  ```

- **422 (Validation Error)**:
  ```json
  {
    "result": "error",
    "errors": {
      "building_id": ["The building id field is required."]
    }
  }
  ```

- **500 (Server Error)**:
  ```json
  {
    "result": "error",
    "message": "An unexpected error occurred while fetching account groups."
  }
  ```

#### Response Fields
- `result`: string - The status of the request. Values: "success" or "error".  
- `total`: integer - The total number of account groups returned in the response.  
- `data`: array - An array of account group objects, each representing a group or subgroup.  
  - `name`: string - The name of the account group (e.g., "Current Asset").  
  - `parent`: string - The parent group or type of the account group (e.g., "Assets").  
  - `type`: string - The type of the account group, corresponding to the top-level accounting category (e.g., "Assets", "Liabilities", "Income", "Expenses").  
- `errors`: object (validation error only) - Key-value pairs of field names and their validation error messages.

---

### Retrieve Cost Categories

Fetches all cost categories for a specified building, filtered by the "Expenses" account type. This endpoint integrates with Tally ERP by exporting cost category data in a format suitable for Tally's cost category configuration. Each cost category represents an expense-related category used for tracking and allocating costs within the accounting system.

**Headers**  
- `Accept: application/json` (required - specifies JSON response format)  
- `Content-Type: application/json` (required for POST requests with body parameters)  
- `Authorization: Bearer {token}` (optional - if authentication is enabled for the API)

**Method**: GET  
**Path**: `/api/V1/getCostCategory`

#### Body Parameters
- `building_id`: integer (required) - The unique identifier of the building to filter cost categories for. Example: `1`

#### Example Request (Bash)
```bash
curl --request GET \
  --get "http://lazim-account.local/api/V1/getCostCategory" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --data "{
    \"building_id\": 1
}"
```

#### Example Request (JavaScript)
```javascript
const url = new URL("http://lazim-account.local/api/V1/getCostCategory");
const headers = {
  "Content-Type": "application/json",
  "Accept": "application/json",
};
let body = {
  "building_id": 1
};
fetch(url, { method: "GET", headers, body: JSON.stringify(body) }).then(response => response.json());
```

#### Example Responses
- **200 (Success)**:
  ```json
  {
    "result": "success",
    "total": 41,
    "data": [
      {
        "name": "3rd Party Requirement, Inspection And Certification"
      },
      {
        "name": "Additional Services"
      },
      {
        "name": "Adjustments"
      },
      {
        "name": "Bank Charges"
      },
      {
        "name": "Beaches"
      },
      {
        "name": "Civil & Architectural"
      },
      {
        "name": "Cleaning Services"
      },
      {
        "name": "Communication Charges / Postal Charges"
      },
      {
        "name": "Community Events"
      },
      {
        "name": "Community Improvement"
      },
      {
        "name": "Community Management Services"
      },
      {
        "name": "Concierge Services"
      },
      {
        "name": "Dewa Services"
      },
      {
        "name": "District Cooling Services"
      },
      {
        "name": "Fire Related Provisions"
      },
      {
        "name": "Gas Services"
      },
      {
        "name": "Government Entities Fees"
      },
      {
        "name": "Health, Safety & Environment Services"
      },
      {
        "name": "Hotel Operational Services"
      },
      {
        "name": "Infrastructure Maintenance"
      },
      {
        "name": "Insurance Services"
      },
      {
        "name": "IT SERVICES"
      },
      {
        "name": "Landscaping Services"
      },
      {
        "name": "Marine And Lakes Maintenance"
      },
      {
        "name": "Master Community Services"
      },
      {
        "name": "MEP Maintenance Services"
      },
      {
        "name": "Mosque Management"
      },
      {
        "name": "Pest Control Services"
      },
      {
        "name": "Professional Services"
      },
      {
        "name": "Recreation & Community Facilities"
      },
      {
        "name": "RECREATION AND COMMUNITY SERVICES"
      },
      {
        "name": "Reserved Fund"
      },
      {
        "name": "Revenue"
      },
      {
        "name": "Road Maintenance"
      },
      {
        "name": "Security Services"
      },
      {
        "name": "Sewerage Charges"
      },
      {
        "name": "Specialized System And Services"
      },
      {
        "name": "TELECOMMUNICATION"
      },
      {
        "name": "Unexpected & Contingency Cost"
      },
      {
        "name": "Waste Management Services"
      },
      {
        "name": "Water Treatment & Test Services"
      }
    ]
  }
  ```

- **422 (Validation Error)**:
  ```json
  {
    "result": "error",
    "errors": {
      "building_id": ["The building id field is required."]
    }
  }
  ```

- **500 (Server Error)**:
  ```json
  {
    "result": "error",
    "message": "An unexpected error occurred while fetching cost categories."
  }
  ```

#### Response Fields
- `result`: string - The status of the request. Values: "success" or "error".  
- `total`: integer - The total number of cost categories returned in the response.  
- `data`: array - An array of cost category objects.  
  - `name`: string - The name of the cost category (e.g., "3rd Party Requirement, Inspection And Certification").  
- `errors`: object (validation error only) - Key-value pairs of field names and their validation error messages.

---

### Get All Cost Centres for a Building

**Method**: GET  
**Path**: `/api/V1/getCostCentre`

#### Body Parameters
- `building_id`: integer (required) - The unique identifier of the building. Example: `1`

(Example requests and responses similar to previous endpoints.)

---

### Retrieve Ledgers

Fetches all ledgers for a specified building, including internal accounts, bank details, customer (Sundry Debtors), and vendor (Sundry Creditors) information. This endpoint integrates with Tally ERP by exporting ledger data in a comprehensive format suitable for Tally's ledger master import. Ledgers are categorized under their parent groups, with optional fields for contact details, VAT registration, and banking information. Fake/generated bank details are used for internal accounts where real data is unavailable.

**Headers**  
- `Accept: application/json` (required - specifies JSON response format)  
- `Content-Type: application/json` (required for POST requests with body parameters)  
- `Authorization: Bearer {token}` (optional - if authentication is enabled for the API)

**Method**: GET  
**Path**: `/api/V1/getLedgers`

#### Body Parameters
- `building_id`: integer (required) - The unique identifier of the building to filter ledgers for. Example: `1`

(Example requests and responses similar to previous endpoints.)

---

### Store an Acknowledgement for a Master or Voucher

**Method**: POST  
**Path**: `/api/V1/acknowledgements`

(Details on parameters and responses not fully specified in the original; assume similar structure.)

---

### Retrieve Purchase Vouchers

Fetches purchase vouchers (bills) within a specified date range for a given building. This endpoint integrates with Tally ERP by exporting bill transaction data in a structured format suitable for voucher import. Each voucher represents a purchase transaction, including details of the vendor (party ledger), expense ledger, and associated cost categories and centers. Only unacknowledged vouchers (based on TallyAcknowledgement records) are included to prevent duplicates.

**Method**: GET  
**Path**: `/api/V1/getPurchaseVoucher`

#### Body Parameters
- `fromDate`: date (required) - Example: `2025-01-01`  
- `toDate`: date (required) - Example: `2025-09-17`  
- `building_id`: integer (required) - Example: `1`

(Example requests and responses similar to sales vouchers.)

---

### Retrieve Payment Vouchers

Fetches payment vouchers (outgoing payments) within a specified date range for a given building. This endpoint integrates with Tally ERP by exporting payment transaction data in a structured format suitable for voucher import. Each voucher represents a payment made to a vendor, including debit and credit ledger details. Note that the response may include duplicate vouchers due to multiple transactions referencing the same payment details. Only unacknowledged vouchers (based on TallyAcknowledgement records) are included to prevent duplicates.

**Method**: GET  
**Path**: `/api/V1/getPaymentVoucher`

#### Body Parameters
- `fromDate`: date (required) - Example: `2025-01-01`  
- `toDate`: date (required) - Example: `2025-09-17`  
- `building_id`: integer (required) - Example: `1`

(Example requests and responses similar.)

---

### Retrieve Receipt Vouchers

Fetches receipt vouchers (incoming payments/revenues) within a specified date range for a given building. This endpoint integrates with Tally ERP by exporting revenue transaction data in a structured format suitable for voucher import. Each voucher represents a payment received from a customer, including bank details, payment method information, and references to associated invoices. Only unacknowledged vouchers (based on TallyAcknowledgement records) are included to prevent duplicates. Duplicate vouchers in the response may occur due to multiple adjustments against the same reference.

**Method**: GET  
**Path**: `/api/V1/getReceiptVoucher`

#### Body Parameters
- `fromDate`: date (required) - The start date of the range in Y-m-d format. Filters revenues dated on or after this date. Example: `2025-01-01`  
- `toDate`: date (required) - The end date of the range in Y-m-d format. Filters revenues dated on or before this date. Example: `2025-09-17`  
- `building_id`: integer (required) - The unique identifier of the building to filter revenues for. Example: `1`

#### Example Request (Bash)
```bash
curl --request GET \
  --get "http://lazim-account.local/api/V1/getReceiptVoucher" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --data "{
    \"fromDate\": \"2025-01-01\",
    \"toDate\": \"2025-09-17\",
    \"building_id\": 1
}"
```

#### Example Request (JavaScript)
```javascript
const url = new URL("http://lazim-account.local/api/V1/getReceiptVoucher");
const headers = {
  "Content-Type": "application/json",
  "Accept": "application/json",
};
let body = {
  "fromDate": "2025-01-01",
  "toDate": "2025-09-17",
  "building_id": 1
};
fetch(url, { method: "GET", headers, body: JSON.stringify(body) }).then(response => response.json());
```

#### Example Responses
- **200 (Success)**:
  ```json
  {
    "result": "success",
    "total_vouchers": 3,
    "vouchers": [
      {
        "voucherDate": "Jan 29, 2025",
        "voucherNumber": 25010000703687,
        "narration": "Reference: 25010000703687",
        "voucherType": "Receipt",
        "voucherDetail": {
          "credit": {
            "ledgerName": "DUBAI SILICON OASIS AUTHORITY",
            "transactionType": "Credit",
            "amount": 2266.58,
            "reference_details": [
              {
                "reference_type": "Agst Ref",
                "reference_number": null,
                "reference_amount": 2266.58
              }
            ]
          },
          "debit": {
            "ledgerName": "General Fund - Bank Account",
            "transactionType": "Debit",
            "amount": 2266.58,
            "payment_deatils": [
              {
                "transfer_method": "Virtual Account Transfer",
                "cheque_dd_number": "18624496690",
                "cheque_dd_date": "2025-04-15",
                "amount": 2266.58
              }
            ]
          }
        }
      }
    ]
  }
  ```

- **422 (Validation Error)**:
  ```json
  {
    "result": "error",
    "errors": {
      "fromDate": ["The from date field is required."],
      "toDate": ["The to date must be a valid date."],
      "building_id": ["The building id field is required."]
    }
  }
  ```

- **500 (Server Error)**:
  ```json
  {
    "result": "error",
    "message": "An unexpected error occurred while fetching receipt vouchers."
  }
  ```

#### Response Fields
- `result`: string - The status of the request. Values: "success" or "error".  
- `total_vouchers`: integer - The total number of receipt vouchers returned in the response (may include duplicates for multi-adjustment transactions).  
- `vouchers`: array - An array of voucher objects, each representing a revenue/payment received.  
  - `voucherDate`: string - The formatted date of the voucher (e.g., "Jan 29, 2025").  
  - `voucherNumber`: integer - The unique reference number of the revenue transaction (e.g., 25010000703687).  
  - `narration`: string - A descriptive narration for the voucher, including the reference number.  
  - `voucherType`: string - The type of voucher for Tally import (always "Receipt").  
  - `voucherDetail`: object - Detailed ledger entries for the voucher.  
    - `credit`: object - The credit ledger entry for the customer/party.  
      - `ledgerName`: string - The name of the customer/party (e.g., "DUBAI SILICON OASIS AUTHORITY"; may be empty if no customer is associated).  
      - `transactionType`: string - The transaction type for the credit ledger (always "Credit").  
      - `amount`: number - The amount credited to the customer (e.g., 2266.58).  
      - `reference_details`: array - Array of reference details linking to invoices (empty if no invoice adjustments).  
        - `reference_type`: string - The type of reference (e.g., "Agst Ref" for against invoice).  
        - `reference_number`: string - The reference number of the linked invoice (e.g., "INV-001"; may be null if not specified).  
        - `reference_amount`: number - The adjusted amount for this reference.  
    - `debit`: object - The debit ledger entry for the bank account.  
      - `ledgerName`: string - The name of the bank account ledger (e.g., "General Fund - Bank Account").  
      - `transactionType`: string - The transaction type for the debit ledger (always "Debit").  
      - `amount`: number - The amount debited from the bank (e.g., 2266.58).  
      - `payment_deatils`: array - Array of payment method details (always one entry per voucher).  
        - `transfer_method`: string - The method of transfer (e.g., "Virtual Account Transfer").  
        - `cheque_dd_number`: string - The cheque or DD number if applicable (e.g., "17313508863").  
        - `cheque_dd_date`: string - The date of the cheque or DD (e.g., "2025-01-29").  
        - `amount`: number - The payment amount (matches the voucher amount).  
- `errors`: object (validation error only) - Key-value pairs of field names and their validation error messages.

---

### Retrieve Contra Vouchers for a Given Building and Date Range

**Method**: GET  
**Path**: `/api/V1/getContraVouchers`

#### Body Parameters
- `fromDate`: date (required) - The start date in Y-m-d format. Example: `2024-01-01`  
- `toDate`: date (required) - The end date in Y-m-d format. Example: `2024-01-31`  
- `building_id`: integer (required) - The building ID. Example: `1`

#### Example Request (Bash)
```bash
curl --request GET \
  --get "http://lazim-account.local/api/V1/getContraVouchers" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --data "{
    \"fromDate\": \"2024-01-01\",
    \"toDate\": \"2024-01-31\",
    \"building_id\": 1
}"
```

#### Example Request (JavaScript)
```javascript
const url = new URL("http://lazim-account.local/api/V1/getContraVouchers");
const headers = {
  "Content-Type": "application/json",
  "Accept": "application/json",
};
let body = {
  "fromDate": "2024-01-01",
  "toDate": "2024-01-31",
  "building_id": 1
};
fetch(url, { method: "GET", headers, body: JSON.stringify(body) }).then(response => response.json());
```

#### Example Responses
- **200 (Success)**:
  ```json
  {
    "result": "success",
    "total_vouchers": 2,
    "vouchers": [
      // Array of contra vouchers
    ]
  }
  ```

- **422 (Validation Error)**:
  ```json
  {
    "result": "error",
    "errors": {
      "fromDate": ["The from date field is required."]
    }
  }
  ```

---

### Retrieve Credit Note Vouchers for a Given Building and Date Range

**Method**: GET  
**Path**: `/api/V1/getCreditNoteVouchers`

(Parameters and responses similar to contra vouchers.)

---

### Retrieve Debit Note Vouchers for a Given Building and Date Range

**Method**: GET  
**Path**: `/api/V1/getDebitNoteVouchers`

(Parameters and responses similar to contra vouchers.)

---

### Retrieve Journal Vouchers for a Given Building and Date Range

**Method**: GET  
**Path**: `/api/V1/getJournalVouchers`

(Parameters and responses similar to contra vouchers.)

---

### Retrieve Yearly Budgets for a Building

**Method**: GET  
**Path**: `/api/V1/getBudget`

#### Body Parameters
- `building_id`: integer (required) - The building ID. Example: `1`

#### Example Request (Bash)
```bash
curl --request GET \
  --get "http://lazim-account.local/api/V1/getBudget" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --data "{
    \"building_id\": 1
}"
```

#### Example Request (JavaScript)
```javascript
const url = new URL("http://lazim-account.local/api/V1/getBudget");
const headers = {
  "Content-Type": "application/json",
  "Accept": "application/json",
};
let body = {
  "building_id": 1
};
fetch(url, { method: "GET", headers, body: JSON.stringify(body) }).then(response => response.json());
```

#### Example Responses
- **200 (Success)**:
  ```json
  {
    "result": "success",
    "total": 1,
    "data": [
      // Array of budgets
    ]
  }
  ```

- **422 (Validation Error)**:
  ```json
  {
    "result": "error",
    "errors": {
      "building_id": ["The building id field is required."]
    }
  }
  ```

---

**Footer**  
- View Postman collection: [Link to Postman]({{ route("scribe.postman") }})  
- View OpenAPI spec: [Link to OpenAPI]({{ route("scribe.openapi") }})  
- Documentation powered by Scribe ✍  
- Last updated: September 17, 2025