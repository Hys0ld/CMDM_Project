# Customer Master Data Management

This project creates a **single Golden Customer Record** by combining customer data from three different business systems:

* Shopify
* Salesforce
* Zendesk

The goal is to identify which records belong to the same customer and create one standardized customer profile.

---

## Architecture

```text
Shopify ───────┐
               │
Salesforce ────┼──► Bronze ──► Silver ──► Golden Record
               │
Zendesk ───────┘
```

---

## 1. Bronze Layer

The raw customer data is loaded into Databricks Delta tables.

```text
bronze.customer_master_data_management
```

Sources include:

* Zendesk customer data
* Shopify customer data
* Salesforce leads

---

## 2. Silver Layer

The data is standardized before matching customers.

Main transformations include:

* Standardizing column names
* Converting text to lowercase
* Removing unnecessary spaces
* Standardizing state codes
* Cleaning phone numbers
* Separating phone extensions
* Handling duplicate email addresses

The three systems are then prepared for customer matching.

---

## 3. Customer Matching

The project uses two matching approaches.

### Deterministic Matching

The first step uses **email** as an exact match.

Customers with unique matching emails across the systems receive a confidence score of:

```text
1.0
```

This creates the first set of Golden Records.

---

### Fuzzy Matching

Customers that cannot be matched using an exact email match are compared using multiple attributes.

Matching factors include:

* Email
* First name
* Last name
* Phone number
* City
* State

The project uses:

* Levenshtein distance
* Soundex
* Exact matching

A weighted confidence score is calculated for each possible match.

Higher-confidence matches are selected as the best match for each customer.

---

## 4. Golden Record

The matched customer information from Shopify, Salesforce, and Zendesk is combined into one standardized record.

The Golden Record contains information such as:

* First name
* Last name
* Email
* Phone number
* Address
* City
* State
* ZIP code
* Source system IDs
* Confidence score
* Match method

Missing values are filled using information available from the other systems when possible.

---

## 5. Gold Layer

The final customer dimension is stored as:

```text
gold.customer_master_data_management.dim_customer_golden
```

A unique `golden_id` is generated using an MD5 hash of the customer's email and phone number.

A cross-reference table is also created:

```text
gold.customer_master_data_management.bridge_id_xref
```

This connects the Golden Customer ID to the original IDs from:

* Shopify
* Salesforce
* Zendesk

---

## 6. Mock Business Data

The project also creates sample business activity for testing the Golden Customer dimension.

### Sales

```text
gold.customer_master_data_management.fct_sales
```

Contains simulated Shopify transactions.

### Support

```text
gold.customer_master_data_management.fct_support
```

Contains simulated Zendesk support tickets.

This allows the Golden Customer dimension to be connected to business activity.

---

## Technologies

* Databricks
* PySpark
* Python
* Delta Lake
* Shopify API
* Salesforce API
* Zendesk data
* Levenshtein distance
* Soundex
* Data matching
* Master Data Management

---

## Project Goal

The main goal of this project is to demonstrate how multiple customer systems can be combined into a **single source of truth**.

```text
Multiple Systems
       │
       ▼
Standardization
       │
       ▼
Exact Matching
       │
       ▼
Fuzzy Matching
       │
       ▼
Golden Customer Record
       │
       ▼
Analytics
```
